<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Reflex Trainer - Full Screen Aim Game</title>
    <style>
        :root {
            --bg-color: #0f172a;
            --card-bg: #1e293b;
            --text-color: #f8fafc;
            --accent-neutral: #3b82f6;
            --accent-ready: #22c55e;
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
            justify-content: flex-start;
            width: 100vw;
            height: 100vh;
            overflow: hidden;
            padding: 15px;
        }

        .container {
            width: 100%;
            height: 100%;
            max-width: 1400px;
            display: flex;
            flex-direction: column;
            z-index: 10;
        }

        header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 12px;
        }

        h1 {
            font-size: 1.8rem;
            letter-spacing: 1px;
        }

        p.subtitle {
            color: #94a3b8;
            font-size: 0.9rem;
        }

        .hud {
            display: flex;
            gap: 20px;
            background: var(--card-bg);
            padding: 10px 20px;
            border-radius: 12px;
            font-weight: 600;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.3);
        }

        .hud span {
            color: #38bdf8;
        }

        /* Full Screen Dynamic Target Field */
        .game-field {
            flex-grow: 1;
            width: 100%;
            border-radius: 16px;
            position: relative;
            background-color: var(--card-bg);
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            margin-bottom: 15px;
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
            background-color: rgba(30, 41, 59, 0.95);
            z-index: 5;
            padding: 20px;
            text-align: center;
        }

        .start-screen h2 {
            font-size: 2.5rem;
            margin-bottom: 10px;
            pointer-events: none;
        }

        .start-screen p {
            font-size: 1.2rem;
            color: #94a3b8;
            pointer-events: none;
            max-width: 500px;
        }

        /* The Interactive Target Dot */
        .target-dot {
            border-radius: 50%;
            background-color: var(--accent-ready);
            box-shadow: 0 0 20px var(--accent-ready);
            border: 3px solid #ffffff;
            position: absolute;
            cursor: pointer;
            display: none;
            transform: translate(-50%, -50%);
            transition: background-color 0.1s, box-shadow 0.1s;
        }

        /* End Results Card Overlay */
        .results-card {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: var(--card-bg);
            border-radius: 16px;
            padding: 30px;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.6);
            border: 2px solid #334155;
            z-index: 20;
            width: 90%;
            max-width: 450px;
            text-align: center;
        }

        .results-card h2 {
            margin-bottom: 15px;
            color: #38bdf8;
            font-size: 2rem;
        }

        .history-list {
            list-style: none;
            margin: 20px 0;
            text-align: left;
            max-height: 200px;
            overflow-y: auto;
        }

        .history-list li {
            padding: 8px 12px;
            border-bottom: 1px solid #334155;
            display: flex;
            justify-content: space-between;
            font-size: 1rem;
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
            width: 100%;
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
    <header>
        <div>
            <h1>⚡ Reflex Trainer</h1>
            <p class="subtitle">By: EDWARD</p>
        </div>
        <div class="hud">
            <div>Level: <span id="current-level">1</span> / 5</div>
            <div>Dot: <span id="dot-count">0</span> / <span id="dot-target">5</span></div>
            <div>Best: <span id="best-score">-- ms</span></div>
        </div>
    </header>
    
    <!-- Full Screen Target Field -->
    <div id="game-field" class="game-field">
        <!-- Start / Overlay Panel -->
        <div id="start-screen" class="start-screen">
            <h2 id="screen-title">Click to Start</h2>
            <p id="screen-desc">Level 1: Pop 5 large dots across the entire screen as fast as you can!</p>
        </div>
        
        <!-- Clickable Interactive Target Dot -->
        <div id="target-dot" class="target-dot"></div>

        <!-- End of Game Performance Breakdown -->
        <div id="results" class="results-card hidden">
            <h2>Challenge Complete!</h2>
            <p id="average-score" style="font-size: 1.2rem; font-weight: bold; margin-bottom: 10px;">Total Average: -- ms</p>
            <ul id="history-list" class="history-list"></ul>
            <button class="btn-reset" onclick="resetGame()">Play Again</button>
        </div>
    </div>
</div>

<script>
    const startScreen = document.getElementById('start-screen');
    const screenTitle = document.getElementById('screen-title');
    const screenDesc = document.getElementById('screen-desc');
    const targetDot = document.getElementById('target-dot');
    const gameField = document.getElementById('game-field');
    
    const currentLevelEl = document.getElementById('current-level');
    const dotCountEl = document.getElementById('dot-count');
    const dotTargetEl = document.getElementById('dot-target');
    const bestScoreEl = document.getElementById('best-score');
    const resultsEl = document.getElementById('results');
    const historyList = document.getElementById('history-list');
    const averageScoreEl = document.getElementById('average-score');

    // Game Configuration per Level (Gets progressively harder)
    const levels = [
        { level: 1, dots: 5, size: 60, name: "Easy (Large Dots)" },
        { level: 2, dots: 8, size: 45, name: "Medium (More Targets)" },
        { level: 3, dots: 10, size: 35, name: "Hard (Smaller Dots)" },
        { level: 4, dots: 12, size: 25, name: "Expert (Tiny Dots)" },
        { level: 5, dots: 15, size: 20, name: "Master (Micro Targets)" }
    ];

    let currentLevelIndex = 0;
    let activeDotsPopped = 0;
    let levelStartTime = 0;
    let levelTimes = [];

    // Load personal best tracking from storage
    let bestScore = localStorage.getItem('bestScoreAimFull') ? parseInt(localStorage.getItem('bestScoreAimFull')) : null;
    if (bestScore) {
        bestScoreEl.textContent = `${bestScore} ms`;
    }

    // Clicking the start overlay triggers the level
    startScreen.addEventListener('click', () => {
        startLevel();
    });

    // Action when a dot is clicked
    targetDot.addEventListener('mousedown', (e) => {
        e.stopPropagation(); 
        activeDotsPopped++;
        dotCountEl.textContent = activeDotsPopped;

        const currentLevelData = levels[currentLevelIndex];
        if (activeDotsPopped < currentLevelData.dots) {
            moveDotRandomly();
        } else {
            endLevel();
        }
    });

    function startLevel() {
        startScreen.classList.add('hidden');
        
        const currentLevelData = levels[currentLevelIndex];
        currentLevelEl.textContent = currentLevelData.level;
        dotTargetEl.textContent = currentLevelData.dots;
        activeDotsPopped = 0;
        dotCountEl.textContent = activeDotsPopped;

        // Resize dot based on difficulty level
        targetDot.style.width = `${currentLevelData.size}px`;
        targetDot.style.height = `${currentLevelData.size}px`;

        // Start clock tracker for the current level
        levelStartTime = window.performance.now();
        moveDotRandomly();
        targetDot.style.display = 'block';
    }

    function moveDotRandomly() {
        const fieldWidth = gameField.clientWidth;
        const fieldHeight = gameField.clientHeight;
        const currentLevelData = levels[currentLevelIndex];
        
        // Keep dots safely inside bounds based on their size
        const padding = currentLevelData.size + 10; 
        const randomX = Math.floor(Math.random() * (fieldWidth - padding * 2)) + padding;
        const randomY = Math.floor(Math.random() * (fieldHeight - padding * 2)) + padding;

        targetDot.style.left = `${randomX}px`;
        targetDot.style.top = `${randomY}px`;
    }

    function endLevel() {
        const levelEndTime = window.performance.now();
        const timeElapsed = Math.round(levelEndTime - levelStartTime);
        levelTimes.push({ level: levels[currentLevelIndex].name, time: timeElapsed });

        targetDot.style.display = 'none';
        currentLevelIndex++;

        if (currentLevelIndex < levels.length) {
            startScreen.classList.remove('hidden');
            screenTitle.textContent = `Level ${currentLevelIndex} Complete!`;
            screenDesc.textContent = `Time: ${timeElapsed}ms. Click anywhere to start Level ${currentLevelIndex + 1}: ${levels[currentLevelIndex].name}`;
        } else {
            processFinalResults();
        }
    }

    function processFinalResults() {
        const total = levelTimes.reduce((acc, curr) => acc + curr.time, 0);
        const average = Math.round(total / levelTimes.length);
        
        averageScoreEl.textContent = `Average Time per Level: ${average} ms`;
        historyList.innerHTML = "";

        levelTimes.forEach((item, index) => {
            const li = document.createElement('li');
            li.innerHTML = `<span>Lvl ${index + 1} (${item.level}):</span> <strong>${item.time} ms</strong>`;
            historyList.appendChild(li);
        });

        if (!bestScore || average < bestScore) {
            bestScore = average;
            localStorage.setItem('bestScoreAimFull', bestScore);
            bestScoreEl.textContent = `${bestScore} ms`;
        }

        resultsEl.classList.remove('hidden');
    }

    function resetGame() {
        currentLevelIndex = 0;
        activeDotsPopped = 0;
        levelTimes = [];
        currentLevelEl.textContent = "1";
        dotCountEl.textContent = "0";
        resultsEl.classList.add('hidden');
        targetDot.style.display = 'none';
        screenTitle.textContent = "Click to Start";
        screenDesc.textContent = "Level 1: Pop 5 large dots across the entire screen as fast as you can!";
        startScreen.classList.remove('hidden');
    }
</script>
</body>
</html>
