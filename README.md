<!DOCTYPE html>
<html>
<head>
    <title>Simple Bible App</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            text-align: center;
            padding: 20px;
        }

        h1 {
            color: darkblue;
        }

        input {
            padding: 8px;
            width: 200px;
        }

        button {
            padding: 8px 12px;
            background-color: darkblue;
            color: white;
            border: none;
            cursor: pointer;
        }

        button:hover {
            background-color: navy;
        }

        #result {
            margin-top: 20px;
            font-size: 18px;
            background: white;
            padding: 15px;
            border-radius: 5px;
        }
    </style>
</head>
<body>

    <h1>📖 Simple Bible Website</h1>
    <p>Enter a Bible reference (example: John 3:16)</p>

    <input type="text" id="reference" placeholder="John 3:16">
    <button onclick="getVerse()">Search</button>

    <div id="result"></div>

    <script>
        async function getVerse() {
            const ref = document.getElementById("reference").value;
            const resultDiv = document.getElementById("result");

            if (!ref) {
                resultDiv.innerHTML = "Please enter a Bible reference.";
                return;
            }

            try {
                const response = await fetch(`https://bible-api.com/${ref}`);
                const data = await response.json();

                if (data.text) {
                    resultDiv.innerHTML = `
                        <strong>${data.reference}</strong><br><br>
                        ${data.text}
                    `;
                } else {
                    resultDiv.innerHTML = "Verse not found.";
                }
            } catch (error) {
                resultDiv.innerHTML = "Error fetching verse.";
            }
        }
    </script>

</body>
</html>
