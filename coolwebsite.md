#Cool Website
By: EDWARD
<!DOCTYPE html>
<html lang="en">
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
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
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
      margin-bottom: 10px;
      letter-spacing: 1px;
    }

    p.subtitle {
      color: #94a3b8;
      margin-bottom: 25px;
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

    .game-box {
      height: 320px;
      border-radius: 16px;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      user-select: none;
      transition: background-color 0.2s ease, transform 0.1s ease;
      background-color: var(--card-bg);
      box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.4);
      padding: 20px;
    }

    .game-box:active {
      transform: scale(0.98);
    }

    .game-box h2 {
      font-size: 2rem;
      pointer-events: none;
    }

    .game-box p {
      font-size: 1.1rem;
      margin-top: 10px;
      opacity: 0.9;
      pointer-events: none;
    }

    /* States */
    .state-start { background-color: var(--accent-neutral); }
    .state-waiting { background-color: var(--accent-wait); color: #000; }
    .state-ready { background-color: var(--accent-ready); color: #000; }
    .state-early { background-color: var(--accent-early); }

    .results {
      display: none;
      background: var(--card-bg);
      border-radius: 16px;
      padding: 30px;
      margin-top: 20px;
      box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.4);
    }

    .results h2 {
      margin-bottom: 15px;
      color: #38bdf8;
    }

    .history-list {
      list-style: none;
      margin: 15px 0;
      text-align: left;
    }

    .history-list li {
      padding: 8px 12px;
      border-bottom: 1px solid #334155;
      display: flex;
      justify-content: space-between;
    }

    .btn-reset {
      background-color: var(--accent-neutral);
      color: white;
      border: none;
      padding: 12px 24px;
      font-size: 1rem;
      font-weight: bold;
      border-radius: 8px;
      cursor: pointer;
      margin-top: 15px;
      transition: background-color 0.2s;
    }

    .btn-reset:hover {
      background-color: #2563eb;
    }
  </style>
</head>
<body>

  <div class="container">
    <h1>⚡ Reflex Trainer</h1>
    <p class="subtitle">Complete 5 rounds to test your average reaction time!</p>

    <div class="hud">
      <div>Round: <span id="current-round">0</span> / 5</div>
      <div>Last Score: <span id="last-score">--</span></div>
    </div>

    <div id="game-box" class="game-box state-start">
      <h2 id="box-title">Click to Start</h2>
      <p id="box-desc">Tests will run over 5 points/rounds</p>
    </div>

    <div id="results" class="results">
      <h2>Game Complete! 🏆</h2>
      <p>Average Reaction Time: <strong id="avg-score" style="font-size: 1.5rem; color: #22c55e;">0 ms</strong></p>
      
      <ul id="history-list" class="history-list"></ul>

      <button class="btn-reset" onclick="resetGame()">Play Again</button>
    </div>
  </div>

  <script>
    const TOTAL_ROUNDS = 5;
    let currentRound = 0;
    let scores = [];
    let timer = null;
    let startTime = 0;
    let state = 'IDLE'; // IDLE, WAITING, READY, EARLY, FINISHED

    const gameBox = document.getElementById('game-box');
    const boxTitle = document.getElementById('box-title');
    const boxDesc = document.getElementById('box-desc');
    const roundDisplay = document.getElementById('current-round');
    const lastScoreDisplay = document.getElementById('last-score');
    const resultsDiv = document.getElementById('results');
    const historyList = document.getElementById('history-list');
    const avgScoreDisplay = document.getElementById('avg-score');

    function setBoxState(className, title, desc) {
      gameBox.className = 'game-box ' + className;
      boxTitle.textContent = title;
      boxDesc.textContent = desc;
    }

    function startRound() {
      state = 'WAITING';
      setBoxState('state-waiting', 'Wait for Green...', 'Click as fast as you can when the box turns green!');
      
      // Random delay between 1.5s and 4s
      const delay = Math.floor(Math.random() * 2500) + 1500;
      
      timer = setTimeout(() => {
        state = 'READY';
        startTime = Date.now();
        setBoxState('state-ready', 'CLICK NOW!', 'GO GO GO!');
      }, delay);
    }

    function handleBoxClick() {
      if (state === 'IDLE') {
        currentRound = 1;
        scores = [];
        roundDisplay.textContent = currentRound;
        resultsDiv.style.display = 'none';
        startRound();
      } else if (state === 'WAITING') {
        // Clicked too early!
        clearTimeout(timer);
        state = 'EARLY';
        setBoxState('state-early', 'Too Soon!', 'You clicked before the box turned green. Click to retry round.');
      } else if (state === 'EARLY') {
        // Retry the current round
        startRound();
      } else if (state === 'READY') {
        // Valid click
        const reactionTime = Date.now() - startTime;
        scores.push(reactionTime);
        lastScoreDisplay.textContent = `${reactionTime} ms`;

        if (currentRound < TOTAL_ROUNDS) {
          currentRound++;
          roundDisplay.textContent = currentRound;
          startRound();
        } else {
          finishGame();
        }
      }
    }

    function finishGame() {
      state = 'FINISHED';
      gameBox.style.display = 'none';
      resultsDiv.style.display = 'block';

      // Calculate Average
      const total = scores.reduce((sum, val) => sum + val, 0);
      const avg = Math.round(total / scores.length);

      avgScoreDisplay.textContent = `${avg} ms`;

      // Render Breakdown
      historyList.innerHTML = '';
      scores.forEach((score, index) => {
        const li = document.createElement('li');
        li.innerHTML = `<span>Round ${index + 1}</span> <strong>${score} ms</strong>`;
        historyList.appendChild(li);
      });
    }

    function resetGame() {
      state = 'IDLE';
      currentRound = 0;
      roundDisplay.textContent = '0';
      lastScoreDisplay.textContent = '--';
      gameBox.style.display = 'flex';
      resultsDiv.style.display = 'none';
      setBoxState('state-start', 'Click to Start', 'Tests will run over 5 points/rounds');
    }

    gameBox.addEventListener('click', handleBoxClick);
  </script>
</body>
</html>
