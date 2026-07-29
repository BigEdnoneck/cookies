<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JS Guitar Hero</title>
    <style>
        body {
            margin: 0;
            background: #111;
            color: #fff;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            overflow: hidden;
        }

#game-container {
            position: relative;
            width: 400px;
            height: 600px;
            background: #1a1a1a;
            border: 4px solid #333;
            box-shadow: 0 0 20px rgba(0,0,0,0.8);
            overflow: hidden;
        }

/* The fretboard lanes */
        .lane {
            position: absolute;
            top: 0;
            width: 80px;
            height: 100%;
            border-right: 1px solid rgba(255,255,255,0.1);
        }
        .lane:nth-child(1) { left: 0; background: rgba(255, 0, 0, 0.05); }
        .lane:nth-child(2) { left: 80px; background: rgba(0, 255, 0, 0.05); }
        .lane:nth-child(3) { left: 160px; background: rgba(0, 0, 255, 0.05); }
        .lane:nth-child(4) { left: 240px; background: rgba(255, 255, 0, 0.05); }
        .lane:nth-child(5) { left: 320px; background: rgba(255, 165, 0, 0.05); border-right: none; }

/* Hit target zone at the bottom */
        .hit-line {
            position: absolute;
            bottom: 80px;
            width: 100%;
            height: 10px;
            background: rgba(255, 255, 255, 0.3);
        }

 .target-key {
            position: absolute;
            bottom: 20px;
            width: 60px;
            height: 50px;
            margin-left: 10px;
            border-radius: 8px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: bold;
            font-size: 20px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.3);
        }
        #t-0 { left: 0; background: #ff4757; }
        #t-1 { left: 80px; background: #2ed573; }
        #t-2 { left: 160px; background: #1e90ff; }
        #t-3 { left: 240px; background: #ffa502; }
        #t-4 { left: 320px; background: #9b59b6; }

/* Falling Notes */
        .note {
            position: absolute;
            width: 60px;
            height: 20px;
            margin-left: 10px;
            border-radius: 4px;
        }
        .note-0 { background: #ff4757; }
        .note-1 { background: #2ed573; }
        .note-2 { background: #1e90ff; }
        .note-3 { background: #ffa502; }
        .note-4 { background: #9b59b6; }

/* UI Overlays */
        #ui {
            position: absolute;
            top: 20px;
            left: 20px;
            right: 20px;
            display: flex;
            justify-content: space-between;
            font-size: 18px;
            font-weight: bold;
            z-index: 10;
            pointer-events: none;
        }

#menu, #game-over {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.85);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 20;
        }

button {
            padding: 12px 24px;
            font-size: 18px;
            font-weight: bold;
            background: #2ed573;
            color: #fff;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition: background 0.2s;
        }
        button:hover { background: #26af5f; }

.hidden { display: none !important; }
        
#feedback {
            position: absolute;
            top: 40%;
            width: 100%;
            text-align: center;
            font-size: 24px;
            font-weight: bold;
            transition: opacity 0.3s;
            pointer-events: none;
        }
    </style>
</head>
<body>

<div id="game-container">
        <!-- UI Overlay -->
        <div id="ui">
            <div>Score: <span id="score">0</span></div>
            <div>Streak: <span id="streak">0</span>x</div>
        </div>

        <div id="feedback"></div>

<!-- Fretboard Lanes -->
<div class="lane"></div>
        <div class="lane"></div>
        <div class="lane"></div>
        <div class="lane"></div>
        <div class="lane"></div>

        <div class="hit-line"></div>
<!-- Target Keys indicators -->
<div class="target-key" id="t-0">A</div>
        <div class="target-key" id="t-1">S</div>
        <div class="target-key" id="t-2">D</div>
        <div class="target-key" id="t-3">F</div>
        <div class="target-key" id="t-4">G</div>

        <!-- Main Menu Overlay -->
<div id="menu">
            <h1>JS Guitar Hero</h1>
            <p>Keys: <strong>A S D F G</strong></p>
            <button onclick="startGame()">Play Game</button>
        </div>
        <!-- Game Over Overlay -->
<div id="game-over" class="hidden">
            <h1>Song Finished!</h1>
            <p>Final Score: <span id="final-score">0</span></p>
            <button onclick="startGame()">Play Again</button>
        </div>
    </div>

<script>
    const lanes = [0, 1, 2, 3, 4];
    const keyMap = { 'KeyA': 0, 'KeyS': 1, 'KeyD': 2, 'KeyF': 3, 'KeyG': 4 };
    
    let score = 0;
    let streak = 0;
    let notes = [];
    let gameActive = false;
    let songTimer = 0;
    let noteSpawnTimer = 0;
    const noteSpeed = 4; // Pixels per frame

    // Procedural track generation pattern
    const songNotes = [];
    function generateSong() {
        for (let i = 100; i < 3000; i += 40) {
            if (Math.random() > 0.3) {
                let lane = Math.floor(Math.random() * 5);
                songNotes.push({ time: i, lane: lane, hit: false });
            }
        }
    }

    const container = document.getElementById('game-container');
    const scoreEl = document.getElementById('score');
    const streakEl = document.getElementById('streak');
    const menuEl = document.getElementById('menu');
    const gameOverEl = document.getElementById('game-over');
    const feedbackEl = document.getElementById('feedback');

    function startGame() {
        menuEl.classList.add('hidden');
        gameOverEl.classList.add('hidden');
        container.querySelectorAll('.note').forEach(n => n.remove());
        
        score = 0;
        streak = 0;
        songTimer = 0;
        notes = [];
        generateSong();
        scoreEl.innerText = score;
        streakEl.innerText = streak;
        gameActive = true;
        
        requestAnimationFrame(gameLoop);
    }

    function showFeedback(text, color) {
        feedbackEl.innerText = text;
        feedbackEl.style.color = color;
        feedbackEl.style.opacity = 1;
        setTimeout(() => { feedbackEl.style.opacity = 0; }, 300);
    }

    window.addEventListener('keydown', (e) => {
        if (!gameActive) return;
        if (keyMap.hasOwnProperty(e.code)) {
            let lane = keyMap[e.code];
            checkHit(lane);
        }
    });

    function checkHit(lane) {
        let hitZoneY = 500; // Hit line position from top
        let threshold = 35; // Pixel tolerance

        for (let note of notes) {
            if (note.lane === lane && !note.hit) {
                let distance = Math.abs(note.y - hitZoneY);
                if (distance < threshold) {
                    note.hit = true;
                    note.element.remove();
                    
                    if (distance < 15) {
                        score += 100;
                        showFeedback("PERFECT!", "#2ed573");
                    } else {
                        score += 50;
                        showFeedback("GOOD", "#1e90ff");
                    }
                    streak++;
                    scoreEl.innerText = score;
                    streakEl.innerText = streak;
                    return;
                }
            }
        }
        // Miss hit penalty
        streak = 0;
        streakEl.innerText = streak;
        showFeedback("MISS", "#ff4757");
    }

    function gameLoop() {
        if (!gameActive) return;

        songTimer++;

        // Spawn notes from song sequence
        songNotes.forEach(n => {
            if (n.time === songTimer) {
                let el = document.createElement('div');
                el.classList.add('note', `note-${n.lane}`);
                el.style.left = (n.lane * 80) + 'px';
                el.style.top = '-20px';
                container.appendChild(el);
                
                notes.push({ lane: n.lane, y: -20, element: el, hit: false });
            }
        });

        // Update note positions
        for (let i = notes.length - 1; i >= 0; i--) {
            let note = notes[i];
            if (note.hit) continue;

            note.y += noteSpeed;
            note.element.style.top = note.y + 'px';

            // Check if note missed bottom boundary
            if (note.y > 600) {
                note.element.remove();
                notes.splice(i, 1);
                streak = 0;
                streakEl.innerText = streak;
            }
        }

        // End condition check
        if (songTimer > 3200 && notes.length === 0) {
            gameActive = false;
            document.getElementById('final-score').innerText = score;
            gameOverEl.classList.remove('hidden');
            return;
        }

        requestAnimationFrame(gameLoop);
    }
</script>

</body>
</html>
