mkdir betting-app
cd betting-app
npm init -y
npm install express mongoose cors body-parser
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
const bodyParser = require("body-parser");

const app = express();
app.use(cors());
app.use(bodyParser.json());
app.use(express.static("public"));

mongoose.connect("mongodb://127.0.0.1:27017/bettingDB");

const UserSchema = new mongoose.Schema({
    username: String,
    balance: { type: Number, default: 1000 },
    bets: [
        {
            match: String,
            amount: Number,
            prediction: String,
            result: String
        }
    ]
});

const User = mongoose.model("User", UserSchema);

// Create or get user
app.post("/user", async (req, res) => {
    let user = await User.findOne({ username: req.body.username });
    if (!user) {
        user = new User({ username: req.body.username });
        await user.save();
    }
    res.json(user);
});

// Place bet
app.post("/bet", async (req, res) => {
    const { username, match, amount, prediction } = req.body;

    let user = await User.findOne({ username });

    if (!user || user.balance < amount)
        return res.json({ message: "Insufficient balance" });

    user.balance -= amount;
    user.bets.push({ match, amount, prediction, result: "Pending" });

    await user.save();
    res.json(user);
});

// Admin view all users
app.get("/admin/users", async (req, res) => {
    const users = await User.find();
    res.json(users);
});

// Auto update match results
app.post("/admin/update", async (req, res) => {
    const users = await User.find();

    for (let user of users) {
        for (let bet of user.bets) {
            if (bet.result === "Pending") {
                bet.result = "Won";
                user.balance += bet.amount * 2;
            }
        }
        await user.save();
    }

    res.json({ message: "Results Updated" });
});

app.listen(3000, () => console.log("Server running on port 3000"));
<!DOCTYPE html>
<html>
<head>
    <title>My Betting App</title>
</head>
<body>

<h2>Betting Platform (Demo)</h2>

<input id="username" placeholder="Enter username">
<button onclick="createUser()">Start</button>

<h3>Place Bet</h3>
<input id="match" placeholder="Match">
<input id="amount" type="number" placeholder="Amount">
<input id="prediction" placeholder="Prediction">
<button onclick="placeBet()">Bet</button>

<h3>User Info</h3>
<pre id="output"></pre>

<script>
let currentUser = "";

async function createUser() {
    currentUser = document.getElementById("username").value;

    const res = await fetch("/user", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ username: currentUser })
    });

    const data = await res.json();
    document.getElementById("output").innerText =
        JSON.stringify(data, null, 2);
}

async function placeBet() {
    const match = document.getElementById("match").value;
    const amount = document.getElementById("amount").value;
    const prediction = document.getElementById("prediction").value;

    const res = await fetch("/bet", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
            username: currentUser,
            match,
            amount,
            prediction
        })
    });

    const data = await res.json();
    document.getElementById("output").innerText =
        JSON.stringify(data, null, 2);
}
</script>

</body>
</html>

