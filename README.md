```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pixel Heist</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            background: #0f0f1e;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            font-family: 'Courier New', monospace;
            overflow: hidden;
        }
        
        #gameContainer {
            position: relative;
            width: 400px;
            height: 400px;
        }
        
        canvas {
            border: 3px solid #4ecca3;
            image-rendering: pixelated;
            image-rendering: crisp-edges;
        }
        
        .screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(15, 15, 30, 0.95);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: #4ecca3;
            z-index: 10;
        }
        
        .screen.hidden {
            display: none;
        }
        
        h1 {
            font-size: 32px;
            margin-bottom: 20px;
            text-shadow: 2px 2px 0px #e74c3c;
            letter-spacing: 2px;
        }
        
        .subtitle {
            font-size: 14px;
            margin-bottom: 40px;
            color: #aaa;
        }
        
        button {
            background: #4ecca3;
            color: #0f0f1e;
            border: none;
            padding: 15px 40px;
            font-size: 18px;
            font-family: 'Courier New', monospace;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.2s;
            text-transform: uppercase;
            margin: 5px;
        }
        
        button:hover {
            background: #3db892;
            transform: scale(1.05);
        }
        
        button:active {
            transform: scale(0.95);
        }
        
        #hud {
            position: absolute;
            top: 10px;
            left: 10px;
            color: #fff;
            font-size: 14px;
            z-index: 5;
            pointer-events: none;
        }
        
        .hud-item {
            margin-bottom: 5px;
            text-shadow: 1px 1px 2px #000;
        }
        
        #sprintBar {
            width: 100px;
            height: 10px;
            background: #333;
            border: 1px solid #666;
            display: inline-block;
            vertical-align: middle;
            margin-left: 5px;
        }
        
        #sprintFill {
            height: 100%;
            background: #4ecca3;
            transition: width 0.1s;
        }
        
        .game-over-text {
            color: #e74c3c;
            font-size: 28px;
            margin-bottom: 20px;
        }
        
        .win-text {
            color: #2ecc71;
            font-size: 28px;
            margin-bottom: 20px;
        }
        
        .high-score {
            color: #ffd700;
            font-size: 16px;
            margin-bottom: 30px;
        }
        
        #adOverlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: #000;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: #fff;
            z-index: 20;
            font-size: 20px;
        }
        
        #adOverlay.hidden {
            display: none;
        }
        
        #adTimer {
            font-size: 48px;
            color: #4ecca3;
            margin-top: 20px;
        }
        
        .bonus-notification {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(78, 204, 163, 0.9);
            color: #0f0f1e;
            padding: 20px;
            border-radius: 10px;
            font-size: 18px;
            font-weight: bold;
            z-index: 15;
            animation: fadeInOut 3s ease-in-out;
        }
        
        @keyframes fadeInOut {
            0% { opacity: 0; transform: translate(-50%, -50%) scale(0.8); }
            20% { opacity: 1; transform: translate(-50%, -50%) scale(1); }
            80% { opacity: 1; transform: translate(-50%, -50%) scale(1); }
            100% { opacity: 0; transform: translate(-50%, -50%) scale(0.8); }
        }
        
        .hidden {
            display: none !important;
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <canvas id="game" width="400" height="400"></canvas>
        
        <div id="startScreen" class="screen">
            <h1>PIXEL HEIST</h1>
            <div class="high-score">🏆 Лучший рекорд: <span id="bestScore">0</span></div>
            <div class="subtitle">Собери алмазы и избегай охранников!</div>
            <button onclick="startGame()">Играть</button>
        </div>
        
        <div id="gameOverScreen" class="screen hidden">
            <div class="game-over-text">ПОЙМАН!</div>
            <div style="color: #fff; margin-bottom: 20px;">Счёт: <span id="finalScore">0</span></div>
            <button onclick="restartGame()">Заново</button>
            <button onclick="watchAdForBonus()" style="background: #9b59b6; margin-top: 10px;">📺 Посмотреть рекламу</button>
        </div>
        
        <div id="winScreen" class="screen hidden">
            <div class="win-text">УРОВЕНЬ ПРОЙДЕН!</div>
            <div style="color: #fff; margin-bottom: 20px;">Время: <span id="finalTime">0</span>с</div>
            <div style="color: #fff; margin-bottom: 20px;">Счёт: <span id="winScore">0</span></div>
            <button onclick="restartGame()">Ещё раз</button>
        </div>
        
        <div id="adOverlay" class="hidden">
            <div>📺 Идет реклама спонсора...</div>
            <div id="adTimer">5</div>
        </div>
        
        <div id="hud" class="hidden">
            <div class="hud-item">💎 <span id="diamondCount">0</span>/<span id="diamondTotal">0</span></div>
            <div class="hud-item">⚡ <div id="sprintBar"><div id="sprintFill" style="width: 100%"></div></div></div>
            <div class="hud-item" id="immunityHud" style="display: none;">🛡️ Бессмертие: <span id="immunityTimer">0</span>с</div>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('game');
        const ctx = canvas.getContext('2d');
        ctx.imageSmoothingEnabled = false;
        
        // Game constants
        const TILE_SIZE = 16;
        const GRID_WIDTH = 25;
        const GRID_HEIGHT = 25;
        const PLAYER_SPEED = 3;
        const SPRINT_SPEED = 5;
        const GUARD_SPEED = 2;
        const VISION_RANGE = 120;
        const VISION_ANGLE = Math.PI / 2;
        const SPRINT_DURATION = 3000; // ms
        
        // Level data
        const level = {
            grid: [
                [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
                [1,0,0,0,0,0,1,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,1],
                [1,0,1,1,1,0,1,0,1,1,1,1,1,0,1,0,1,1,1,1,1,1,1,0,1],
                [1,0,1,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,1,0,1],
                [1,0,1,0,1,1,1,1,1,1,1,0,1,1,1,1,1,1,1,1,1,0,1,0,1],
                [1,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,1],
                [1,1,1,0,1,0,1,1,1,0,1,1,1,1,1,0,1,1,1,0,1,1,1,1,1],
                [1,0,0,0,0,0,1,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,1],
                [1,0,1,1,1,1,1,0,1,1,1,1,1,0,1,1,1,1,1,1,1,1,1,0,1],
                [1,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
                [1,1,1,1,1,0,1,1,1,0,1,1,1,1,1,1,1,0,1,1,1,1,1,1,1],
                [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
                [1,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,1],
                [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
                [1,1,1,0,1,1,1,1,1,0,1,1,1,1,1,0,1,1,1,1,1,0,1,1,1],
                [1,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,1],
                [1,0,1,1,1,0,1,1,1,1,1,1,1,1,1,1,1,1,1,0,1,1,1,0,1],
                [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
                [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1]
            ],
            playerSpawn: { x: 1, y: 1 },
            exit: { x: 23, y: 17 },
            diamonds: [
                { x: 5, y: 3, collected: false },
                { x: 12, y: 5, collected: false },
                { x: 18, y: 9, collected: false },
                { x: 8, y: 13, collected: false }
            ],
            guards: [
                {
                    waypoints: [{ x: 3, y: 5 }, { x: 10, y: 5 }],
                    currentWaypoint: 0,
                    x: 3 * TILE_SIZE + TILE_SIZE/2,
                    y: 5 * TILE_SIZE + TILE_SIZE/2,
                    direction: { x: 1, y: 0 },
                    angle: 0,
                    waitTimer: 0
                },
                {
                    waypoints: [{ x: 15, y: 9 }, { x: 22, y: 9 }],
                    currentWaypoint: 0,
                    x: 15 * TILE_SIZE + TILE_SIZE/2,
                    y: 9 * TILE_SIZE + TILE_SIZE/2,
                    direction: { x: 1, y: 0 },
                    angle: 0,
                    waitTimer: 0
                },
                {
                    waypoints: [{ x: 5, y: 15 }, { x: 15, y: 15 }],
                    currentWaypoint: 0,
                    x: 5 * TILE_SIZE + TILE_SIZE/2,
                    y: 15 * TILE_SIZE + TILE_SIZE/2,
                    direction: { x: 1, y: 0 },
                    angle: 0,
                    waitTimer: 0
                }
            ]
        };
        
        // Game state
        let gameState = 'menu'; // menu, playing, gameover, win, paused
        let player = {
            x: 0,
            y: 0,
            vx: 0,
            vy: 0,
            sprintEnergy: SPRINT_DURATION,
            isSprinting: false,
            lastSprintTime: 0,
            immunity: 0, // Immunity timer in ms
            score: 0
        };
        let keys = {};
        let startTime = 0;
        let animationId = null;
        let adTimerInterval = null;
        
        // Audio context for sound effects
        const AudioContext = window.AudioContext || window.webkitAudioContext;
        const audioCtx = new AudioContext();
        
        function playSound(frequency, duration, type = 'sine') {
            const oscillator = audioCtx.createOscillator();
            const gainNode = audioCtx.createGain();
            
            oscillator.connect(gainNode);
            gainNode.connect(audioCtx.destination);
            
            oscillator.frequency.value = frequency;
            oscillator.type = type;
            
            gainNode.gain.setValueAtTime(0.3, audioCtx.currentTime);
            gainNode.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + duration);
            
            oscillator.start(audioCtx.currentTime);
            oscillator.stop(audioCtx.currentTime + duration);
        }
        
        // Load high score from localStorage
        function getHighScore() {
            return parseInt(localStorage.getItem('pixelHeistHighScore')) || 0;
        }
        
        function saveHighScore(score) {
            const currentHigh = getHighScore();
            if (score > currentHigh) {
                localStorage.setItem('pixelHeistHighScore', score);
            }
        }
        
        function updateHighScoreDisplay() {
            document.getElementById('bestScore').textContent = getHighScore();
        }
        
        function startGame() {
            document.getElementById('startScreen').classList.add('hidden');
            document.getElementById('hud').classList.remove('hidden');
            resetGame();
            gameState = 'playing';
            startTime = Date.now();
            gameLoop();
        }
        
        function restartGame() {
            document.getElementById('gameOverScreen').classList.add('hidden');
            document.getElementById('winScreen').classList.add('hidden');
            resetGame();
            gameState = 'playing';
            startTime = Date.now();
        }
        
        function resetGame() {
            player.x = level.playerSpawn.x * TILE_SIZE + TILE_SIZE/2;
            player.y = level.playerSpawn.y * TILE_SIZE + TILE_SIZE/2;
            player.vx = 0;
            player.vy = 0;
            player.sprintEnergy = SPRINT_DURATION;
            player.isSprinting = false;
            player.immunity = 0;
            player.score = 0;
            
            level.diamonds.forEach(d => d.collected = false);
            
            level.guards.forEach(guard => {
                guard.currentWaypoint = 0;
                guard.x = guard.waypoints[0].x * TILE_SIZE + TILE_SIZE/2;
                guard.y = guard.waypoints[0].y * TILE_SIZE + TILE_SIZE/2;
                guard.direction = { x: 1, y: 0 };
                guard.angle = 0;
                guard.waitTimer = 0;
            });
            
            updateHUD();
        }
        
        function updateHUD() {
            const collected = level.diamonds.filter(d => d.collected).length;
            document.getElementById('diamondCount').textContent = collected;
            document.getElementById('diamondTotal').textContent = level.diamonds.length;
            
            const sprintPercent = (player.sprintEnergy / SPRINT_DURATION) * 100;
            document.getElementById('sprintFill').style.width = sprintPercent + '%';
            
            // Update immunity display
            const immunityHud = document.getElementById('immunityHud');
            if (player.immunity > 0) {
                immunityHud.style.display = 'block';
                document.getElementById('immunityTimer').textContent = Math.ceil(player.immunity / 1000);
            } else {
                immunityHud.style.display = 'none';
            }
        }
        
        // Input handling
        window.addEventListener('keydown', (e) => {
            keys[e.key.toLowerCase()] = true;
            if (e.key === 'Shift') player.isSprinting = true;
        });
        
        window.addEventListener('keyup', (e) => {
            keys[e.key.toLowerCase()] = false;
            if (e.key === 'Shift') player.isSprinting = false;
        });
        
        function isWall(x, y) {
            const gridX = Math.floor(x / TILE_SIZE);
            const gridY = Math.floor(y / TILE_SIZE);
            
            if (gridX < 0 || gridX >= GRID_WIDTH || gridY < 0 || gridY >= GRID_HEIGHT) {
                return true;
            }
            
            return level.grid[gridY][gridX] === 1;
        }
        
        function updatePlayer(deltaTime) {
            let speed = PLAYER_SPEED;
            
            if (player.isSprinting && player.sprintEnergy > 0) {
                speed = SPRINT_SPEED;
                player.sprintEnergy -= deltaTime;
                if (player.sprintEnergy < 0) player.sprintEnergy = 0;
            } else if (!player.isSprinting && player.sprintEnergy < SPRINT_DURATION) {
                player.sprintEnergy += deltaTime * 0.3;
                if (player.sprintEnergy > SPRINT_DURATION) player.sprintEnergy = SPRINT_DURATION;
            }
            
            // Update immunity
            if (player.immunity > 0) {
                player.immunity -= deltaTime;
                if (player.immunity < 0) player.immunity = 0;
            }
            
            player.vx = 0;
            player.vy = 0;
            
            if (keys['w'] || keys['arrowup']) player.vy = -speed;
            if (keys['s'] || keys['arrowdown']) player.vy = speed;
            if (keys['a'] || keys['arrowleft']) player.vx = -speed;
            if (keys['d'] || keys['arrowright']) player.vx = speed;
            
            // Normalize diagonal movement
            if (player.vx !== 0 && player.vy !== 0) {
                player.vx *= 0.707;
                player.vy *= 0.707;
            }
            
            // Try to move X
            const newX = player.x + player.vx;
            if (!checkCollision(newX, player.y)) {
                player.x = newX;
            }
            
            // Try to move Y
            const newY = player.y + player.vy;
            if (!checkCollision(player.x, newY)) {
                player.y = newY;
            }
            
            // Keep player in bounds
            player.x = Math.max(4, Math.min(canvas.width - 4, player.x));
            player.y = Math.max(4, Math.min(canvas.height - 4, player.y));
        }
        
        function checkCollision(x, y) {
            const halfSize = 4; // Half of 8px hitbox
            
            // Check all four corners of the hitbox
            const points = [
                { x: x - halfSize, y: y - halfSize },
                { x: x + halfSize, y: y - halfSize },
                { x: x - halfSize, y: y + halfSize },
                { x: x + halfSize, y: y + halfSize }
            ];
            
            for (let point of points) {
                if (isWall(point.x, point.y)) {
                    return true;
                }
            }
            
            return false;
        }
        
        function updateGuards(deltaTime) {
            level.guards.forEach(guard => {
                if (guard.waitTimer > 0) {
                    guard.waitTimer -= deltaTime;
                    return;
                }
                
                const target = guard.waypoints[guard.currentWaypoint];
                const targetX = target.x * TILE_SIZE + TILE_SIZE/2;
                const targetY = target.y * TILE_SIZE + TILE_SIZE/2;
                
                const dx = targetX - guard.x;
                const dy = targetY - guard.y;
                const distance = Math.sqrt(dx * dx + dy * dy);
                
                if (distance < 2) {
                    // Reached waypoint
                    guard.currentWaypoint = (guard.currentWaypoint + 1) % guard.waypoints.length;
                    guard.waitTimer = 1000; // Wait 1 second
                    
                    // Update direction for next movement
                    const nextTarget = guard.waypoints[guard.currentWaypoint];
                    const nextDx = nextTarget.x * TILE_SIZE + TILE_SIZE/2 - guard.x;
                    const nextDy = nextTarget.y * TILE_SIZE + TILE_SIZE/2 - guard.y;
                    const nextDist = Math.sqrt(nextDx * nextDx + nextDy * nextDy);
                    
                    if (nextDist > 0) {
                        guard.direction = { x: nextDx / nextDist, y: nextDy / nextDist };
                        guard.angle = Math.atan2(nextDy, nextDx);
                    }
                } else {
                    // Move towards waypoint
                    const moveX = (dx / distance) * GUARD_SPEED;
                    const moveY = (dy / distance) * GUARD_SPEED;
                    
                    guard.x += moveX;
                    guard.y += moveY;
                    
                    guard.direction = { x: dx / distance, y: dy / distance };
                    guard.angle = Math.atan2(dy, dx);
                    
                    // Add slight random look around
                    if (Math.random() < 0.01) {
                        guard.angle += (Math.random() - 0.5) * 0.3;
                    }
                }
            });
        }
        
        function checkVision() {
            // If player has immunity, skip vision check
            if (player.immunity > 0) return;
            
            for (let guard of level.guards) {
                const dx = player.x - guard.x;
                const dy = player.y - guard.y;
                const distance = Math.sqrt(dx * dx + dy * dy);
                
                if (distance > VISION_RANGE) continue;
                
                const angleToPlayer = Math.atan2(dy, dx);
                let angleDiff = Math.abs(angleToPlayer - guard.angle);
                
                // Normalize angle difference
                if (angleDiff > Math.PI) {
                    angleDiff = 2 * Math.PI - angleDiff;
                }
                
                if (angleDiff > VISION_ANGLE / 2) continue;
                
                // Check for walls between guard and player
                if (!hasWallBetween(guard.x, guard.y, player.x, player.y)) {
                    gameOver();
                    return;
                }
            }
        }
        
        function hasWallBetween(x1, y1, x2, y2) {
            const steps = Math.ceil(Math.sqrt((x2-x1)**2 + (y2-y1)**2) / 4);
            
            for (let i = 0; i <= steps; i++) {
                const t = i / steps;
                const checkX = x1 + (x2 - x1) * t;
                const checkY = y1 + (y2 - y1) * t;
                
                if (isWall(checkX, checkY)) {
                    return true;
                }
            }
            
            return false;
        }
        
        function checkDiamonds() {
            level.diamonds.forEach(diamond => {
                if (diamond.collected) return;
                
                const diamondX = diamond.x * TILE_SIZE + TILE_SIZE/2;
                const diamondY = diamond.y * TILE_SIZE + TILE_SIZE/2;
                
                const dx = player.x - diamondX;
                const dy = player.y - diamondY;
                const distance = Math.sqrt(dx * dx + dy * dy);
                
                if (distance < 12) {
                    diamond.collected = true;
                    player.score += 100;
                    playSound(800, 0.1);
                    updateHUD();
                }
            });
        }
        
        function checkExit() {
            const allCollected = level.diamonds.every(d => d.collected);
            if (!allCollected) return;
            
            const exitX = level.exit.x * TILE_SIZE + TILE_SIZE/2;
            const exitY = level.exit.y * TILE_SIZE + TILE_SIZE/2;
            
            const dx = player.x - exitX;
            const dy = player.y - exitY;
            const distance = Math.sqrt(dx * dx + dy * dy);
            
            if (distance < 12) {
                winGame();
            }
        }
        
        function gameOver() {
            gameState = 'gameover';
            playSound(200, 0.3, 'sawtooth');
            
            // Save high score
            saveHighScore(player.score);
            
            document.getElementById('finalScore').textContent = player.score;
            document.getElementById('gameOverScreen').classList.remove('hidden');
        }
        
        function winGame() {
            gameState = 'win';
            const time = ((Date.now() - startTime) / 1000).toFixed(1);
            
            // Calculate final score based on time bonus
            const timeBonus = Math.max(0, 1000 - Math.floor(parseFloat(time) * 10));
            player.score += timeBonus;
            
            // Save high score
            saveHighScore(player.score);
            
            document.getElementById('finalTime').textContent = time;
            document.getElementById('winScore').textContent = player.score;
            playSound(600, 0.2);
            setTimeout(() => playSound(800, 0.2), 200);
            document.getElementById('winScreen').classList.remove('hidden');
        }
        
        // Rewarded Ad System
        function watchAdForBonus() {
            // Pause the game
            gameState = 'paused';
            
            // Show ad overlay
            const adOverlay = document.getElementById('adOverlay');
            adOverlay.classList.remove('hidden');
            
            let timeLeft = 5;
            document.getElementById('adTimer').textContent = timeLeft;
            
            // Start countdown
            adTimerInterval = setInterval(() => {
                timeLeft--;
                document.getElementById('adTimer').textContent = timeLeft;
                
                if (timeLeft <= 0) {
                    clearInterval(adTimerInterval);
                    finishAd();
                }
            }, 1000);
        }
        
        function finishAd() {
            // Hide ad overlay
            document.getElementById('adOverlay').classList.add('hidden');
            
            // Grant bonus: 5 seconds of immunity
            player.immunity = 5000;
            
            // Show bonus notification
            showBonusNotification('🛡️ +5 секунд бессмертия!');
            
            // Resume game
            gameState = 'playing';
            lastTime = performance.now();
            gameLoop();
        }
        
        function showBonusNotification(text) {
            const notification = document.createElement('div');
            notification.className = 'bonus-notification';
            notification.textContent = text;
            document.getElementById('gameContainer').appendChild(notification);
            
            setTimeout(() => {
                notification.remove();
            }, 3000);
        }
        
        function drawPixelSprite(x, y, pixels, colors, size = 16) {
            const pixelSize = size / pixels.length;
            
            for (let row = 0; row < pixels.length; row++) {
                for (let col = 0; col < pixels[row].length; col++) {
                    const colorKey = pixels[row][col];
                    if (colorKey !== '.' && colors[colorKey]) {
                        ctx.fillStyle = colors[colorKey];
                        ctx.fillRect(
                            x - size/2 + col * pixelSize,
                            y - size/2 + row * pixelSize,
                            pixelSize,
                            pixelSize
                        );
                    }
                }
            }
        }
        
        function render() {
            // Clear canvas
            ctx.fillStyle = '#1a1a2e';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            // Draw grid/walls
            for (let y = 0; y < level.grid.length; y++) {
                for (let x = 0; x < level.grid[y].length; x++) {
                    if (level.grid[y][x] === 1) {
                        ctx.fillStyle = '#444444';
                        ctx.fillRect(x * TILE_SIZE, y * TILE_SIZE, TILE_SIZE, TILE_SIZE);
                        
                        // Add some texture
                        ctx.fillStyle = '#555555';
                        ctx.fillRect(x * TILE_SIZE + 2, y * TILE_SIZE + 2, TILE_SIZE - 4, TILE_SIZE - 4);
                    }
                }
            }
            
            // Draw exit
            const exitX = level.exit.x * TILE_SIZE;
            const exitY = level.exit.y * TILE_SIZE;
            const allCollected = level.diamonds.every(d => d.collected);
            
            ctx.fillStyle = allCollected ? '#2ecc71' : '#1a6b3a';
            ctx.fillRect(exitX, exitY, TILE_SIZE, TILE_SIZE);
            
            // Exit door animation
            if (allCollected) {
                const pulse = Math.sin(Date.now() / 200) * 0.3 + 0.7;
                ctx.fillStyle = `rgba(46, 204, 113, ${pulse})`;
                ctx.fillRect(exitX + 2, exitY + 2, TILE_SIZE - 4, TILE_SIZE - 4);
            }
            
            // Draw diamonds
            const diamondPixels = [
                ['.', '.', 'C', 'C', '.', '.'],
                ['.', 'C', 'W', 'W', 'C', '.'],
                ['C', 'W', 'C', 'C', 'W', 'C'],
                ['C', 'C', 'W', 'W', 'C', 'C'],
                ['.', 'C', 'C', 'C', 'C', '.'],
                ['.', '.', 'C', 'C', '.', '.']
            ];
            
            const diamondColors = {
                'C': '#00d2ff',
                'W': '#ffffff'
            };
            
            level.diamonds.forEach(diamond => {
                if (diamond.collected) return;
                
                const x = diamond.x * TILE_SIZE + TILE_SIZE/2;
                const y = diamond.y * TILE_SIZE + TILE_SIZE/2;
                
                // Pulsing animation
                const scale = 1 + Math.sin(Date.now() / 500) * 0.1;
                
                ctx.save();
                ctx.translate(x, y);
                ctx.scale(scale, scale);
                drawPixelSprite(0, 0, diamondPixels, diamondColors, 12);
                ctx.restore();
            });
            
            // Draw guard vision cones
            level.guards.forEach(guard => {
                ctx.save();
                ctx.translate(guard.x, guard.y);
                ctx.rotate(guard.angle);
                
                // Vision cone gradient
                const gradient = ctx.createRadialGradient(0, 0, 0, 0, 0, VISION_RANGE);
                gradient.addColorStop(0, 'rgba(231, 76, 60, 0.4)');
                gradient.addColorStop(1, 'rgba(231, 76, 60, 0.1)');
                
                ctx.fillStyle = gradient;
                ctx.beginPath();
                ctx.moveTo(0, 0);
                ctx.arc(0, 0, VISION_RANGE, -VISION_ANGLE/2, VISION_ANGLE/2);
                ctx.closePath();
                ctx.fill();
                
                ctx.restore();
            });
            
            // Draw guards
            const guardPixels = [
                ['.', '.', 'R', 'R', '.', '.'],
                ['.', 'R', 'R', 'R', 'R', '.'],
                ['R', 'R', 'B', 'B', 'R', 'R'],
                ['R', 'R', 'R', 'R', 'R', 'R'],
                ['.', 'R', 'R', 'R', 'R', '.'],
                ['.', '.', 'R', 'R', '.', '.']
            ];
            
            const guardColors = {
                'R': '#e74c3c',
                'B': '#000000'
            };
            
            level.guards.forEach(guard => {
                drawPixelSprite(guard.x, guard.y, guardPixels, guardColors);
            });
            
            // Draw player
            const playerPixels = [
                ['.', '.', 'P', 'P', '.', '.'],
                ['.', 'P', 'P', 'P', 'P', '.'],
                ['.', 'P', 'E', 'E', 'P', '.'],
                ['.', 'P', 'P', 'P', 'P', '.'],
                ['.', '.', 'P', 'P', '.', '.'],
                ['.', 'P', '.', '.', 'P', '.']
            ];
            
            const playerColors = {
                'P': player.immunity > 0 ? '#ffd700' : '#4ecca3', // Gold when immune
                'E': '#ffffff'
            };
            
            // Add immunity shield effect
            if (player.immunity > 0) {
                ctx.save();
                ctx.strokeStyle = 'rgba(255, 215, 0, 0.6)';
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.arc(player.x, player.y, 12, 0, Math.PI * 2);
                ctx.stroke();
                ctx.restore();
            }
            
            drawPixelSprite(player.x, player.y, playerPixels, playerColors);
        }
        
        let lastTime = 0;
        
        function gameLoop(timestamp) {
            if (gameState !== 'playing') return;
            
            if (!lastTime) lastTime = timestamp;
            const deltaTime = timestamp - lastTime;
            lastTime = timestamp;
            
            updatePlayer(deltaTime);
            updateGuards(deltaTime);
            checkVision();
            checkDiamonds();
            checkExit();
            updateHUD();
            
            render();
            
            animationId = requestAnimationFrame(gameLoop);
        }
        
        // Initialize high score display
        updateHighScoreDisplay();
        
        // Initial render
        render();
    </script>
</body>
</html>
```
