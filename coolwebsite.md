<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Reflex Trainer - 5-Round Game</title>
    <style>
        :root {
            --bg-color: #0f172a;
            --card-bg: #1e293b;
            --text-color: #f8fafc;
            --accent-wait: #eab308;
            --accent-ready: #22c55e;
            --accent-early: #ef4444;
            --accent-neutral: #3b82f6;
        }

        * {
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

        /* Game Interaction Box */
        .game-box {
            width: 100%;
            height: 320px;
            border-radius: 16px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            transition: background-color 0.15s ease, transform 0.1s ease;
            background-color: var(--card-bg);
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            padding: 20px;
            margin-bottom: 24px;
        }

        .game-box:active {
            transform: scale(0.99);
        }

        .game-box h2 {
            font-size: 2.2rem;
            pointer-events: none;
            margin-bottom: 15px;
        }

        .game-box p {
            font-size: 1.1rem;
            opacity: 0.9;
            pointer-events: none;
        }

        /* 5 Dots Indicator Container */
        .dots-container {
            display: flex;
            gap: 15px;
            margin-bottom: 20px;
            pointer-events: none;
        }

        .dot {
            width: 24px;
            height: 24px;
            border-radius: 50%;
            background-color: rgba(255, 255, 255, 0.1);
            border: 2px solid rgba(255, 255, 255, 0.2);
            transition: background-color 0.1s ease, box-shadow 0.1s ease;
        }

        /* Active dot styles during sequence */
        .dot.active {
            background-color: #ef4444; /* Red lights charging up */
            box-shadow: 0 0 15px #ef4444;
            border-color: #f87171;
        }

        /* Ready state for dots */
        .state-ready .dot {
            background-color: var(--accent-ready);
            box-shadow: 0 0 15px var(--accent-ready);
            border-color: #4ade80;
        }

        /* State color variations for the box background */
        .state-start { background-color: var(--accent-neutral); color: #ffffff; }
        .state-waiting { background-color: #1e293b; color: #ffffff; } /* Dark during countdown */
        .state-ready { background-color: var(--accent-ready); color: #000000; }
        .state-early { background-color: var(--accent-early); color: #ffffff; }
        .state-result { background-color: var(--accent-neutral); color: #ffffff; }

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
            transition: background-color 0.2s, transform 0.1s;
            box-shadow: 0 4px 6px rgba(0,0,0,0.2);
        }

        .btn-reset:hover {
            background-color: #2563eb;
        }
        
        .btn-reset:active {
            transform: scale(0.96);
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
            <div>Best: <span id="best-score">-- ms</span></div>
        </div>

        <!-- Interactive Game Box -->
        <div id="game-box" class="game-box state-start">
            <!-- 5 Dots Layout -->
            <div class="dots-container">
                <div class="dot"></div>
                <div class="dot"></div>
                <div class="dot"></div>
                <div class="dot"></div>
                <div class="dot"></div>
            </div>
            <h2 id="box-title">Click to Start</h2>
            <p id="box-desc">Complete all 5 rounds to see your score!</p>
        </div>

        <!-- End of Game Performance Breakdown -->
        <div id="results" class="results-card hidden">
            <h2>Game Complete!</h2>
            <p id="average-score" style="font-size: 1.3rem; font-weight: bold; margin-bottom: 10px;">Average: -- ms</p>
            <ul id="history-list" class="history-list"></ul>
            <button class="btn-reset" onclick="resetGame()">Play Again</button>
        </div>
    </div>

    <script>
        const gameBox = document.getElementById('game-box');
        const boxTitle = document.getElementById('box-title');
        const boxDesc = document.getElementById('box-desc');
        const currentRoundEl = document.getElementById('current-round');
        const bestScoreEl = document.getElementById('best-score');
        const resultsEl = document.getElementById('results');
        const historyList = document.getElementById('history-list');
        const averageScoreEl = document.getElementById('average-score');
        const dots = document.querySelectorAll('.dot');

        // Game Configuration variables
        let state = 'start'; // start | waiting | ready | early | result
        let currentRound = 0;
        let startTime = 0;
        let roundTimes = [];
        let dotTimers = []; // Keeps track of interval loops for lighting dots
        let greenLightTimer = null; 

        // Load personal best tracking from storage
        let bestScore = localStorage.getItem('bestScore') ? parseInt(localStorage.getItem('bestScore')) : null;
        if (bestScore) {
            bestScoreEl.textContent = `${bestScore} ms`;
        }

        // Primary interactive click handler
        gameBox.addEventListener('click', () => {
            if (state === 'start' || state === 'result' || state === 'early') {
                startRound();
            } else if (state === 'waiting') {
                triggerEarlyClick();
            } else if (state === 'ready') {
                handleSuccessfulClick();
            }
        });

        function setUIState(newState) {
            gameBox.className = `game-box state-${newState}`;
            state = newState;
        }

        function clearAllIntervals() {
            dotTimers.forEach(t => clearTimeout(t));
            dotTimers = [];
            clearTimeout(greenLightTimer);
        }

        function clearDotsColors() {
            dots.forEach(dot => dot.classList.remove('active'));
        }

        function startRound() {
            clearAllIntervals();
            clearDotsColors();
            
            if (currentRound >= 5) {
                currentRound = 0;
                roundTimes = [];
                resultsEl.classList.add('hidden');
            }

            currentRound++;
            currentRoundEl.textContent = currentRound;
            
            setUIState('waiting');
            boxTitle.textContent = "Hold on...";
            boxDesc.textContent = "Wait for the dots to fill and turn green!";

            // Light up the 5 dots sequentially (1 every 500ms)
            for (let i = 0; i < 5; i++) {
                let timerId = setTimeout(() => {
                    if (state === 'waiting') {
                        dots[i].classList.add('active');
                    }
                }, (i + 1) * 500);
                dotTimers.push(timerId);
            }

            // After all 5 dots light up, add a random delay between 1 to 3 seconds to turn green
            const totalSequenceTime = 2500; 
            const randomDelay = totalSequenceTime + (Math.random() * 2000 + 1000);

            greenLightTimer = setTimeout(() => {
                if (state === 'waiting') {
                    triggerGreenLight();
                }
