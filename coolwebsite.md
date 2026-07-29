<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Reflex Trainer - 5-Dot Aim Game</title>
    <style>
        :root {
            --bg-color: #0f172a;
            --card-bg: #1e293b;
            --text-color: #f8fafc;
            --accent-neutral: #3b82f6;
            --accent-ready: #22c55e;
        }

         {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            user-select: none;
        }

        body {
            background-color: var(--bg-color);
            color: var(--text-color);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            padding: 20px;
        }

        .container {
            width: 100%;
            max-width: 600px;
            text-align: center;
            z-index: 10;
        }

        h1 {
            font-size: 2.5rem;
            margin-bottom: 8px;
            letter-spacing: 1px;
        }

        p.subtitle {
            color: #94a3b8;
            margin-bottom: 25px;
            font-size: 1rem;
        }

        .hud {
            display: flex;
            justify-content: space-between;
            background: var(--card-bg);
            padding: 15px 25px;
            border-radius: 12px;
            margin-bottom: 20px;
            font-weight: 600;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.3);
        }

        .hud span {
            color: #38bdf8;
        }

        /* Large Screen-Wide Target Field */
        .game-field {
            width: 100%;
            height: 400px;
            border-radius: 16px;
            position: relative;
            background-color: var(--card-bg);
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            margin-bottom: 24px;
            border: 2px solid #334155;
            overflow: hidden;
        }

        /* Start Screen Overlay */
        .start-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            background-color: #1e293b;
            z-index: 5;
            padding: 20px;
        }

        .start-screen h2 {
            font-size: 2.2rem;
            margin-bottom: 10px;
            pointer-events: none;
        }

        .start-screen p {
            font-size: 1.1rem;
            color: #94a3b8;
            pointer-events: none;
        }

        /* The Interactive Target Dot */
        .target-dot {
            width: 40px;
            height: 40px;
            border-radius: 50%;
            background-color: var(--accent-ready);
            box-shadow: 0 0 20px var(--accent-ready);
            border: 3px solid #ffffff;
            position: absolute;
            cursor: pointer;
            display: none;
            transform: translate(-50%, -50%);
        }

        /* End Results Card */
        .results-card {
            background: var(--card-bg);
            border-radius: 16px;
            padding: 30px;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.4);
            margin-top: 20px;
        }

        .results-card h2 {
            margin-bottom: 15px;
            color: #38bdf8;
            font-size: 1.8rem;
        }

        .history-list {
            list-style: none;
            margin: 20px 0;
            text-align: left;
        }

        .history-list li {
            padding: 10px 14px;
            border-bottom: 1px solid #334155;
            display: flex;
            justify-content: space-between;
            font-size: 1.1rem;
        }

        .btn-reset {
            background-color: var(--accent-neutral);
            color: white;
            border: none;
            padding: 12px 28px;
            font-size: 1rem;
            font-weight: bold;
            border-radius: 8px;
            cursor: pointer;
            transition: background-color 0.2s;
            box-shadow: 0 4px 6px rgba(0,0,0,0.2);
        }

        .btn-reset:hover {
            background-color: #2563eb;
        }
        
        .hidden {
            display: none !important;
        }
    </style>
</head>
<body>

    <div class="container">
        <h1>⚡ Reflex Trainer</h1>
        <p class="subtitle">By: EDWARD</p>

        <div class="hud">
            <div>Round: <span id="current-round">0</span> / 5</div>
            <div>Dot: <span id="dot-count">0</span> / 5</div>
            <div>Best: <span id="best-score">-- ms</span></div>
        </div>

        <!-- Target Field -->
        <div id="game-field" class="game-field">
            <!-- Start / Overlay Panel -->
            <div id="start-screen" class="start-screen">
                <h2 id="screen-title">Click to Start</h2>
                <p id="screen-desc">Pop 5 random dots as fast as you can each round!</p>
            </div>
            
            <!-- Clickable Interactive Target Dot -->
            <div id="target-dot" class="target-dot"></div>
        </div>

        <!-- End of Game Performance Breakdown -->
        <div id="results" class="results-card hidden">
            <h2>Game Complete!</h2>
            <p id="average-score" style="font-size: 1.3rem; font-weight: bold; margin-bottom: 10px;">Average Round: -- ms</p>
            <ul id="history-list" class="history-list"></ul>
            <button class="btn-reset" onclick="resetGame()">Play Again</button>
        </div>
    </div>

    <script>
        const startScreen = document.getElementById('start-screen');
        const screenTitle = document.getElementById('screen-title');
        const screenDesc = document.getElementById('screen-desc');
        const targetDot = document.getElementById('target-dot');
        const gameField = document.getElementById('game-field');
        
        const currentRoundEl = document.getElementById('current-round');
        const dotCountEl = document.getElementById('dot-count');
        const bestScoreEl = document.getElementById('best-score');
        const resultsEl = document.getElementById('results');
        const historyList = document.getElementById('history-list');
        const averageScoreEl = document.getElementById('average-score');

        // State Tracking Data
        let currentRound = 0;
        let activeDotsPopped = 0;
        let roundStartTime = 0;
        let roundTimes = [];

        // Load personal best tracking from storage
        let bestScore = localStorage.getItem('bestScoreAim') ? parseInt(localStorage.getItem('bestScoreAim')) : null;
        if (bestScore) {
            bestScoreEl.textContent = `${bestScore} ms`;
        }

        // Clicking the start overlay triggers the round
        startScreen.addEventListener('click', () => {
            startRound();
        });

        // Action when a dot is clicked
        targetDot.addEventListener('mousedown', (e) => {
            e.stopPropagation(); // Stop click from filtering to background
            activeDotsPopped++;
            dotCountEl.textContent = activeDotsPopped;

            if (activeDotsPopped < 5) {
                moveDotRandomly();
            } else {
                endRound();
            }
        });

        function startRound() {
            startScreen.classList.add('hidden');
            
            if (currentRound >= 5) {
                currentRound = 0;
                roundTimes = [];
                resultsEl.classList.add('hidden');
            }

            currentRound++;
            activeDotsPopped = 0;
            
            currentRoundEl.textContent = currentRound;
            dotCountEl.textContent = activeDotsPopped;

            // Start clock tracker for the current round
            roundStartTime = window.performance.now();
            moveDotRandomly();
            targetDot.style.display = 'block';
        }

        function moveDotRandomly() {
            const fieldWidth = gameField.clientWidth;
            const fieldHeight = gameField.clientHeight;
            
            // Keep dots safely 40px away from the edges
            const padding = 40; 
            const randomX = Math.floor(Math.random() * (fieldWidth - padding * 2)) + padding;
            const randomY = Math.floor(Math.random() * (fieldHeight - padding * 2)) + padding;

            targetDot.style.left = `${randomX}px`;
            targetDot.style.top = `${randomY}px`;
        }

        function endRound() {
            const roundEndTime = window.performance.now();
            const timeElapsed = Math.round(roundEndTime - roundStartTime);
            roundTimes.push(timeElapsed);

            targetDot.style.display = 'none';
            startScreen.classList.remove('hidden');

            if (currentRound < 5) {
                screenTitle.textContent = `Round ${currentRound} Complete!`;
                screenDesc.textContent = `Time: ${timeElapsed}ms. Click here to start Round ${currentRound + 1}.`;
            } else {
                processFinalResults();
            }
        }

        function processFinalResults() {
            screenTitle.textContent = "Finished!";
            screenDesc.textContent = "See your summary stats down below.";

            const total = roundTimes.reduce((acc, curr) => acc + curr, 0);
            const average = Math.round(total / roundTimes.length);
            
            averageScoreEl.textContent = `Average Round: ${average} ms`;
            historyList.innerHTML = "";

            roundTimes.forEach((time, index) => {
                const li = document.createElement('li');
                li.innerHTML = <span>Round ${index + 1}:</span> <strong>${time} ms</strong>;
                historyList.appendChild(li);});
                if (!bestScore || average < bestScore) {bestScore = average;localStorage.setItem('bestScoreAim', bestScore);
                bestScoreEl.textContent = ${bestScore} ms;
                }resultsEl.classList.remove('hidden');
                }function resetGame() {currentRound = 0;
                activeDotsPopped = 0;
                roundTimes = [];
                currentRoundEl.textContent = "0";
                dotCountEl.textContent = "0";
                resultsEl.classList.add('hidden');
                targetDot.style.display = 'none';
                screenTitle.textContent = "Click to Start";
                screenDesc.textContent = "Pop 5 random dots as fast as you can each round!";
                startScreen.classList.remove('hidden');}
