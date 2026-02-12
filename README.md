{
  "name": "maddison-bet-backend",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "bcryptjs": "^2.4.3",
    "cors": "^2.8.5",
    "dotenv": "^16.0.0",
    "express": "^4.18.2",
    "jsonwebtoken": "^9.0.0",
    "mongoose": "^7.0.0",
    "socket.io": "^4.7.0"
  }
}
require("dotenv").config();
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
const http = require("http");
const socketIo = require("socket.io");

const app = express();
const server = http.createServer(app);
const io = socketIo(server, {
  cors: { origin: "*" }
});

app.use(cors());
app.use(express.json());

mongoose.connect(process.env.MONGO_URI)
  .then(() => console.log("Database Connected"))
  .catch(err => console.log(err));

app.use("/api/auth", require("./routes/auth"));
app.use("/api/bet", require("./routes/bet"));

io.on("connection", (socket) => {
  console.log("User Connected");

  setInterval(() => {
    const randomOdds = (Math.random() * 3 + 1).toFixed(2);
    socket.emit("updateOdds", randomOdds);
  }, 5000);
});

server.listen(5000, () => {
  console.log("MADDISON BET Server Running on port 5000");
});
const mongoose = require("mongoose");

const UserSchema = new mongoose.Schema({
  username: String,
  email: String,
  password: String,
  balance: { type: Number, default: 0 }
});

module.exports = mongoose.model("User", UserSchema);
const router = require("express").Router();
const bcrypt = require("bcryptjs");
const jwt = require("jsonwebtoken");
const User = require("../models/User");

router.post("/register", async (req, res) => {
  const hashed = await bcrypt.hash(req.body.password, 10);

  const user = new User({
    username: req.body.username,
    email: req.body.email,
    password: hashed
  });

  await user.save();
  res.json({ message: "User Registered" });
});

router.post("/login", async (req, res) => {
  const user = await User.findOne({ email: req.body.email });
  if (!user) return res.status(400).json({ error: "User not found" });

  const valid = await bcrypt.compare(req.body.password, user.password);
  if (!valid) return res.status(400).json({ error: "Wrong password" });

  const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET);
  res.json({ token });
});

module.exports = router;
const router = require("express").Router();
const User = require("../models/User");

router.post("/place", async (req, res) => {
  const { userId, amount, odds } = req.body;

  const user = await User.findById(userId);

  if (user.balance < amount)
    return res.status(400).json({ error: "Insufficient balance" });

  user.balance -= amount;
  await user.save();

  res.json({
    message: "Bet placed",
    potentialWin: amount * odds
  });
});

module.exports = router;
import React, { useEffect, useState } from "react";
import io from "socket.io-client";

const socket = io("http://localhost:5000");

function App() {
  const [odds, setOdds] = useState(1.0);

  useEffect(() => {
    socket.on("updateOdds", (newOdds) => {
      setOdds(newOdds);
    });
  }, []);

  return (
    <div>
      <h1>MADDISON BET</h1>
      <h2>Live Football Odds: {odds}</h2>
    </div>
  );
}

export default App;
const axios = require("axios");

async function fetchMatches() {
  const response = await axios.get("SPORTS_API_URL");
  return response.data;
}
mongodump --uri="your_mongo_uri" --out=/backup/


