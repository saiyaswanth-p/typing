<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Typing Speed Test with History & Modes</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 0;
            padding: 0;
            background: #f4f4f4;
            color: #333;
            transition: 0.3s;
        }
        .container {
            margin: 50px auto;
            width: 80%;
            max-width: 800px;
            background: white;
            box-shadow: 0 0 15px #ccc;
            padding: 30px;
            border-radius: 10px;
        }
        textarea, input {
            width: 100%;
            font-size: 16px;
            padding: 10px;
            margin: 10px 0;
            border: 1px solid #ccc;
            border-radius: 5px;
            resize: none;
        }
        button {
            padding: 10px 20px;
            color: white;
            background: #28a745;
            border: none;
            cursor: pointer;
            transition: 0.3s;
            margin: 5px;
        }
        button:hover {
            background: #218838;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
        }
        .result, .history, .wpm {
            margin-top: 20px;
            font-size: 18px;
        }
        .timer {
            font-size: 20px;
            color: #dc3545;
        }
        .history {
            max-height: 200px;
            overflow-y: auto;
            background: #f0f0f0;
            padding: 10px;
            border-radius: 5px;
        }
        .dark-mode {
            background: #333;
            color: #f0f0f0;
        }
        .dark-mode .container {
            background: #444;
            color: #fff;
        }
        .dark-mode .history {
            background: #555;
        }
        .accuracy.high {
            color: #28a745;
        }
        .accuracy.medium {
            color: #ffc107;
        }
        .accuracy.low {
            color: #dc3545;
        }
    </style>
</head>
<body>

<div class="container">
    <h2>Typing Speed Test with History & Modes</h2>

    <button onclick="toggleDarkMode()">Toggle Dark Mode 🌙</button>

    <div>
        <label>
            <input type="radio" name="mode" value="timed" checked> Timed
        </label>
        <label>
            <input type="radio" name="mode" value="untimed"> Untimed
        </label>

        <label>
            <select id="durationSelect">
                <option value="30">30s</option>
                <option value="60" selected>60s</option>
                <option value="120">120s</option>
            </select>
        </label>
    </div>

    <br>
    
    <button onclick="loadRandomText()">Random Text</button>
    <button onclick="setCustomText()">Use Custom Text</button>

    <p id="sampleText">Click "Random Text" or "Use Custom Text" to start!</p>

    <textarea id="userInput" placeholder="Start typing here..." oninput="trackInput()" disabled></textarea>

    <button onclick="calculateSpeed()">Calculate Speed</button>
    <button onclick="resetTest()">Start Again</button>
    <button onclick="exportHistory()">Export History</button>
    
    <div class="result" id="results"></div>
    <div class="wpm" id="wpmDisplay">WPM: 0</div>
    <p class="timer" id="timer">Time Left: 60s</p>

    <h3>Saved Results:</h3>
    <div class="history" id="history"></div>
</div>

<script>
    const sampleTexts = [
        "The sun sets over the horizon, painting the sky with hues of orange and pink.",
        "She sells seashells by the seashore, which sparkle in the morning light.",
        "A journey of a thousand miles begins with a single step.",
        "Technology is best when it brings people together.",
        "Creativity is intelligence having fun.",
        "The only limit to our realization of tomorrow is our doubts of today."
    ];

    let startTime, endTime, timerInterval, timeLeft;
    let timerRunning = false;
    let history = JSON.parse(localStorage.getItem('history')) || [];
    let mistakeTracker = [];

    function toggleDarkMode() {
        document.body.classList.toggle("dark-mode");
    }

    function loadRandomText() {
        const randomIndex = Math.floor(Math.random() * sampleTexts.length);
        document.getElementById("sampleText").innerText = sampleTexts[randomIndex];
        document.getElementById("userInput").disabled = false;
        resetTest();
    }

    function setCustomText() {
        const customText = prompt("Enter your own sample text:");
        if (customText) {
            document.getElementById("sampleText").innerText = customText;
            document.getElementById("userInput").disabled = false;
            resetTest();
        }
    }

    function startTimer() {
        if (timerRunning) return;

        const mode = document.querySelector('input[name="mode"]:checked').value;
        const duration = parseInt(document.getElementById("durationSelect").value);
        timeLeft = duration;

        if (mode === "untimed") return;

        timerRunning = true;
        startTime = new Date().getTime();

        timerInterval = setInterval(() => {
            timeLeft--;
            document.getElementById("timer").textContent = `Time Left: ${timeLeft}s`;

            if (timeLeft <= 0) {
                clearInterval(timerInterval);
                document.getElementById("userInput").disabled = true;
                calculateSpeed();
            }
        }, 1000);
    }

    function trackInput() {
        const sampleText = document.getElementById("sampleText").innerText;
        const userInput = document.getElementById("userInput").value;

        if (userInput.length === 1 && !timerRunning) {
            startTimer();
        }

        const words = (userInput.match(/\S+/g) || []).length;
        const duration = parseInt(document.getElementById("durationSelect").value);
        const wpm = Math.round((words / (duration - timeLeft)) * 60);
        document.getElementById("wpmDisplay").textContent = `WPM: ${wpm}`;
    }

    function calculateSpeed() {
        clearInterval(timerInterval);
        
        const sampleText = document.getElementById("sampleText").innerText;
        const userInput = document.getElementById("userInput").value;

        const totalTime = (new Date().getTime() - startTime) / 1000;
        const words = (userInput.match(/\S+/g) || []).length;
        const wpm = Math.round((words / totalTime) * 60);

        const accuracy = (userInput === sampleText) ? "100%" : `${(userInput.length / sampleText.length * 100).toFixed(2)}%`;

        const result = `Speed: ${wpm} WPM | Accuracy: ${accuracy}`;
        
        history.push(result);
        localStorage.setItem('history', JSON.stringify(history));
        displayHistory();
        
        document.getElementById("results").innerHTML = result;
    }

    function displayHistory() {
        const historyDiv = document.getElementById("history");
        historyDiv.innerHTML = history.map((res, i) => `<p>${i + 1}. ${res}</p>`).join('');
    }

    function exportHistory() {
        const content = history.join("\n");
        const blob = new Blob([content], { type: "text/plain" });
        const anchor = document.createElement("a");
        anchor.href = URL.createObjectURL(blob);
        anchor.download = "typing_test_history.txt";
        anchor.click();
    }

    function resetTest() {
        clearInterval(timerInterval);
        document.getElementById("userInput").value = "";
        document.getElementById("timer").textContent = "Time Left: 60s";
        document.getElementById("wpmDisplay").textContent = "WPM: 0";
        timerRunning = false;
    }

    displayHistory();
</script>

</body>
</html>
