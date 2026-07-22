#Cool Website
By: EDWARD
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Reflex Trainer</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            user-select: none;
        }

        body {
            background-color: #121214;
            color: #ffffff;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        .container {
            width: 100%;
            max-width: 600px;
            padding: 20px;
            text-align: center;
        }

        header {
            margin-bottom: 24px;
        }

        h1 {
            font-size: 2.5rem;
            font-weight: 700;
            margin-bottom: 8px;
        }

        p.subtitle {
            color: #a1a1aa;
            font-size: 1rem;
        }

        /* Test Box Styles */
        .test-box {
            width: 100%;
            height: 320px;
            border-radius: 16px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            cursor: pointer;
            transition: background-color 0.15s ease;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            margin-bottom: 24px;
        }

        /* State colors */
        .state-idle {
            background-color: #27272a;
        }
        .state-waiting {
            background-color: #dc2626; /* Red */
        }
        .state-ready {
            background-color: #16a34a; /* Green */
        }
        .state-result {
            background-color: #2563eb; /* Blue */
        }

        .test-box h2 {
            font-size: 2rem;
            margin-bottom: 8px;
            pointer-events: none;
        }

        .test-box p {
            font-size: 1.1rem;
            opacity: 0.9;
            pointer-events: none;
        }

        /* Stats Section */
        .stats {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 16px;
        }

        .stat-card {
            background-color: #18181b;
            border: 1px solid #27272a;
            padding: 16px;
            border-radius: 12px;
        }

        .stat-card span {
            display: block;
            font-size: 0.875rem;
            color: #a1a1aa;
            margin-bottom: 4px;
            text-transform: uppercase;
            letter-spacing: 0.05em;
        }

        .stat-card strong {
            font-size: 1.5rem;
            color: #ffffff;
        }
    </style>
</head>
<body>

    <div class="container">
        <header>
            <h1>Reflex Trainer</h1>
            <p class="subtitle">Test your reaction time and speed up your reflexes.</p>
        </header>

        <div id="test-box" class="test-box state-idle">
            <h2 id="main-text">Click to Start</h2>
            <p id="sub-text">When the red box turns green, click as fast as you can.</p>
        </div>

        <div class="stats">
            <div class="stat-card">
                <span>Last Time</span>
                <strong id="last-score">-- ms</strong>
            </div>
            <div class="stat-card">
                <span>Best Time</span>
                <strong id="best-score">-- ms</strong>
            </div>
        </div>
    </div>

    <script>
        const testBox = document.getElementById('test-box');
        const mainText = document.getElementById('main-text');
        const subText = document.getElementById('sub-text');
        const lastScoreEl = document.getElementById('last-score');
        const bestScoreEl = document.getElementById('best-score');

        // State variables
        let state = 'idle'; // idle | waiting | ready | result
        let timer = null;
        let startTime = 0;
        let bestScore = localStorage.getItem('bestScore') ? parseInt(localStorage.getItem('bestScore')) : null;

        if (bestScore) {
            bestScoreEl.textContent = `${bestScore} ms`;
        }

        // Helper to set UI states easily
        function setUIState(newState) {
            testBox.className = `test-box state-${newState}`;
            state = newState;
        }

        function handleInteraction() {
            if (state === 'idle' || state === 'result') {
                // Start waiting
                setUIState('waiting');
                mainText.textContent = 'Wait for Green...';
                subText.textContent = 'Hold on!';

                // Random delay between 2 and 5 seconds
                const randomDelay = Math.floor(Math.random() * 3000) + 2000;

                timer = setTimeout(() => {
                    setUIState('ready');
                    mainText.textContent = 'CLICK NOW!';
                    subText.textContent = '';
                    startTime = Date.now();
                }, randomDelay);

            } else if (state === 'waiting') {
                // Clicked too early!
                clearTimeout(timer);
                setUIState('idle');
                mainText.textContent = 'Too Early!';
                subText.textContent = 'You clicked before the box turned green. Click to try again.';

            } else if (state === 'ready') {
                // Successful reaction!
                const reactionTime = Date.now() - startTime;
                setUIState('result');
                
                mainText.textContent = `${reactionTime} ms`;
                subText.textContent = 'Click to try again.';

                // Update UI stats
                lastScoreEl.textContent = `${reactionTime} ms`;
                
                if (!bestScore || reactionTime < bestScore) {
                    bestScore = reactionTime;
                    localStorage.setItem('bestScore', bestScore);
                    bestScoreEl.textContent = `${bestScore} ms`;
                }
            }
        }

        // Handle both click and touch/spacebar inputs for better accuracy
        testBox.addEventListener('pointerdown', (e) => {
            e.preventDefault(); // Prevents double firing on mobile
            handleInteraction();
        });

        document.addEventListener('keydown', (e) => {
            if (e.code === 'Space') {
                e.preventDefault();
                handleInteraction();
            }
        });
    </script>
</body>
</html>
