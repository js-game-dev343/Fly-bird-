 <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0">
    <title>Flappy Bird Pro</title>
    <style>
        body { margin: 0; padding: 0; overflow: hidden; background-color: #1a1a1a; display: flex; justify-content: center; align-items: center; height: 100vh; font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif; touch-action: none; }
        #game-container { position: relative; user-select: none; }
        canvas { background-color: #70c5ce; display: block; border: 4px solid #fff; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.5); }
        
        /* UI Layer */
        #ui-layer { position: absolute; top: 0; left: 0; width: 100%; height: 100%; display: flex; flex-direction: column; justify-content: center; align-items: center; pointer-events: none; }
        #score-display { position: absolute; top: 30px; color: white; font-size: 50px; font-weight: 900; text-shadow: 4px 4px 0 #000; z-index: 10; }
        
        .panel { background: rgba(0,0,0,0.9); padding: 25px; border-radius: 20px; border: 4px solid #f1c40f; text-align: center; color: white; display: none; pointer-events: auto; width: 260px; box-shadow: 0 0 20px rgba(241, 196, 15, 0.3); }
        
        /* Score Row Layout */
        .score-row { display: flex; justify-content: space-around; margin: 20px 0; border-top: 1px solid #444; border-bottom: 1px solid #444; padding: 15px 0; }
        .score-box { display: flex; flex-direction: column; }
        .label { font-size: 14px; color: #aaa; text-transform: uppercase; letter-spacing: 1px; }
        .val { font-size: 28px; font-weight: bold; color: #fff; }
        .high-val { color: #f1c40f; }

        /* Button Layout */
        .button-stack { display: flex; flex-direction: column; gap: 12px; }
        .btn { padding: 14px; font-size: 18px; font-weight: bold; color: white; border-radius: 12px; cursor: pointer; border: none; transition: 0.1s; text-transform: uppercase; }
        .btn-retry { background: #e67e22; border-bottom: 4px solid #d35400; }
        .btn-ad { background: #9b59b6; border-bottom: 4px solid #8e44ad; }
        .btn:active { transform: translateY(3px); border-bottom-width: 1px; }
        
        #start-screen { display: block; }
    </style>
</head>
<body>

    <div id="game-container">
        <canvas id="gameCanvas"></canvas>
        <div id="ui-layer">
            <div id="score-display">0</div>
            
            <div id="start-screen" class="panel">
                <h1 style="margin: 0 0 20px 0; font-size: 32px; color: #f1c40f;">FLAPPY<br>BIRD</h1>
                <button class="btn btn-retry" style="width:100%" onclick="startGame(false)">PLAY NOW</button>
            </div>

            <div id="game-over-screen" class="panel">
                <h2 style="color: #e74c3c; margin: 0; font-size: 28px;">CRASHED!</h2>
                
                <div class="score-row">
                    <div class="score-box">
                        <span class="label">Score</span>
                        <span id="final-score" class="val">0</span>
                    </div>
                    <div class="score-box">
                        <span class="label">Best</span>
                        <span id="high-score" class="val high-val">0</span>
                    </div>
                </div>

                <div class="button-stack">
                    <button class="btn btn-retry" onclick="startGame(false)">RETRY</button>
                    <button id="ad-btn" class="btn btn-ad" onclick="handleAdRequest()">📺 REVIVE (AD)</button>
                </div>
            </div>
        </div>
    </div>

    <script>
        // ==========================================
        // ADMOB CONFIGURATION
        // ==========================================
        const ADMOB_CONFIG = {
            AD_UNIT_ID: "ca-app-pub-3893159799848007/7334521534", // PASTE UNIT ID HERE
        };

        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreDisplay = document.getElementById('score-display');
        const gameOverScreen = document.getElementById('game-over-screen');
        const startScreen = document.getElementById('start-screen');
        const adBtn = document.getElementById('ad-btn');

        function resize() {
            canvas.height = Math.min(window.innerHeight * 0.85, 600);
            canvas.width = Math.min(window.innerWidth * 0.95, 400);
        }
        window.addEventListener('resize', resize);
        resize();

        const GRAVITY = 0.25;
        const JUMP = -5.2;
        const PIPE_SPEED = 2.5;
        const PIPE_GAP = 160;
        const PIPE_WIDTH = 65;

        let bird, pipes, score, frame, gameRunning, hasRevived, highScore;
        highScore = localStorage.getItem('flappyHighScore') || 0;

        function init(keepProgress) {
            if (!keepProgress) {
                score = 0;
                pipes = [];
                hasRevived = false;
            }
            bird = { x: 50, y: canvas.height / 2, w: 38, h: 26, vel: 0, wing: 0 };
            frame = 0;
            gameRunning = true;
            startScreen.style.display = 'none';
            gameOverScreen.style.display = 'none';
            scoreDisplay.innerText = score;
            scoreDisplay.style.display = 'block';
        }

        function startGame(revive) {
            init(revive);
            if (revive) pipes = pipes.filter(p => p.x > 180 || p.x < -50);
            loop();
        }

        function die() {
            gameRunning = false;
            scoreDisplay.style.display = 'none';
            if (score > highScore) {
                highScore = score;
                localStorage.setItem('flappyHighScore', highScore);
            }
            document.getElementById('final-score').innerText = score;
            document.getElementById('high-score').innerText = highScore;
            gameOverScreen.style.display = 'block';
            adBtn.style.display = hasRevived ? 'none' : 'block';
        }

        // ==========================================
        // AD LOGIC
        // ==========================================
        function handleAdRequest() {
            // Placeholder for AdMob SDK: AdMob.showRewardedAd(...)
            console.log("Loading Ad Unit:", ADMOB_CONFIG.AD_UNIT_ID);
            alert("Watching Ad to continue...");
            setTimeout(() => {
                hasRevived = true;
                startGame(true);
            }, 1500); 
        }

        function jump() { if(gameRunning) bird.vel = JUMP; }
        window.addEventListener('keydown', (e) => { if(e.code === 'Space') jump(); });
        canvas.addEventListener('touchstart', (e) => { e.preventDefault(); jump(); }, {passive:false});
        canvas.addEventListener('mousedown', jump);

        function drawBird() {
            ctx.save();
            ctx.translate(bird.x + bird.w/2, bird.y + bird.h/2);
            ctx.rotate(Math.min(Math.PI/4, Math.max(-Math.PI/4, bird.vel * 0.1)));
            ctx.fillStyle = '#f1c40f';
            ctx.beginPath();
            ctx.ellipse(0, 0, bird.w/2, bird.h/2, 0, 0, Math.PI*2);
            ctx.fill(); ctx.stroke();
            bird.wing += 0.2;
            ctx.fillStyle = 'white';
            ctx.beginPath();
            ctx.ellipse(-8, Math.sin(bird.wing)*5, 10, 6, 0, 0, Math.PI*2);
            ctx.fill(); ctx.stroke();
            ctx.fillStyle = '#e74c3c';
            ctx.beginPath(); ctx.moveTo(12, -2); ctx.lineTo(22, 2); ctx.lineTo(12, 6); ctx.closePath(); ctx.fill(); ctx.stroke();
            ctx.fillStyle = 'black';
            ctx.beginPath(); ctx.arc(8, -4, 3, 0, Math.PI*2); ctx.fill();
            ctx.restore();
        }

        function drawPipe(p) {
            let grd = ctx.createLinearGradient(p.x, 0, p.x + PIPE_WIDTH, 0);
            grd.addColorStop(0, "#2ecc71"); grd.addColorStop(0.5, "#a2f3a2"); grd.addColorStop(1, "#27ae60");
            ctx.fillStyle = grd; ctx.lineWidth = 2;
            ctx.fillRect(p.x, 0, PIPE_WIDTH, p.top);
            ctx.strokeRect(p.x, 0, PIPE_WIDTH, p.top);
            ctx.fillRect(p.x-5, p.top-20, PIPE_WIDTH+10, 20);
            ctx.strokeRect(p.x-5, p.top-20, PIPE_WIDTH+10, 20);
            let bY = p.top + PIPE_GAP;
            ctx.fillRect(p.x, bY, PIPE_WIDTH, canvas.height-bY);
            ctx.strokeRect(p.x, bY, PIPE_WIDTH, canvas.height-bY);
            ctx.fillRect(p.x-5, bY, PIPE_WIDTH+10, 20);
            ctx.strokeRect(p.x-5, bY, PIPE_WIDTH+10, 20);
        }

        function loop() {
            if(!gameRunning) return;
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            bird.vel += GRAVITY;
            bird.y += bird.vel;
            if (frame % 100 === 0) {
                let h = Math.random() * (canvas.height - PIPE_GAP - 100) + 50;
                pipes.push({ x: canvas.width, top: h, passed: false });
            }
            pipes.forEach((p, i) => {
                p.x -= PIPE_SPEED;
                drawPipe(p);
                if (bird.x + bird.w - 5 > p.x && bird.x + 5 < p.x + PIPE_WIDTH) {
                    if (bird.y + 5 < p.top || bird.y + bird.h - 5 > p.top + PIPE_GAP) die();
                }
                if (p.x + PIPE_WIDTH < bird.x && !p.passed) {
                    score++; p.passed = true; scoreDisplay.innerText = score;
                }
                if (p.x < -PIPE_WIDTH) pipes.splice(i, 1);
            });
            if (bird.y + bird.h > canvas.height || bird.y < 0) die();
            drawBird();
            frame++;
            requestAnimationFrame(loop);
        }
    </script>
</body>
</html>
