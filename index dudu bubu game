<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dudu Bubu Game</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { width: 100vw; height: 100vh; font-family: Arial; background: #333; }
        
        canvas { display: block; width: 100%; height: 100%; background: #87ceeb; }
        
        .overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.8); display: flex; justify-content: center; align-items: center; z-index: 100; }
        
        .box { background: white; padding: 20px; border-radius: 10px; text-align: center; max-width: 300px; }
        .box h1 { color: #667eea; margin-bottom: 10px; }
        .box input { width: 100%; padding: 10px; margin: 10px 0; border: 1px solid #ccc; border-radius: 5px; }
        .box button { width: 100%; padding: 10px; background: #667eea; color: white; border: none; border-radius: 5px; margin-top: 10px; cursor: pointer; font-weight: bold; }
        .box button:hover { background: #5568d3; }
        
        .hidden { display: none !important; }
        
        .hud { position: fixed; top: 10px; left: 10px; right: 10px; display: flex; justify-content: space-between; color: white; font-weight: bold; z-index: 50; text-shadow: 2px 2px 4px rgba(0,0,0,0.7); }
        .hud-item { background: rgba(0,0,0,0.6); padding: 10px; border-radius: 5px; }
    </style>
</head>
<body>
    <canvas id="canvas"></canvas>
    
    <!-- LOGIN SCREEN -->
    <div id="loginScreen" class="overlay">
        <div class="box">
            <h1>🎮 DUDU BUBU</h1>
            <input type="text" id="username" placeholder="Enter username">
            <input type="tel" id="phone" placeholder="Enter phone">
            <button onclick="startGame()">PLAY</button>
        </div>
    </div>

    <!-- PAUSE SCREEN -->
    <div id="pauseScreen" class="overlay hidden">
        <div class="box">
            <h1>⏸ PAUSED</h1>
            <p id="pauseText"></p>
            <button onclick="resumeGame()">RESUME</button>
            <button onclick="quitGame()">QUIT</button>
        </div>
    </div>

    <!-- GAME OVER SCREEN -->
    <div id="gameOverScreen" class="overlay hidden">
        <div class="box">
            <h1>😵 GAME OVER</h1>
            <p id="gameOverText"></p>
            <button onclick="restartGame()">PLAY AGAIN</button>
        </div>
    </div>

    <!-- HUD -->
    <div class="hud">
        <div class="hud-item">Score: <span id="scoreText">0</span></div>
        <div class="hud-item">Level: <span id="levelText">1</span></div>
        <div class="hud-item">Lives: <span id="livesText">❤️❤️❤️</span></div>
    </div>

    <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');

        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        let gameState = {
            username: '',
            phone: '',
            score: 0,
            level: 1,
            lives: 3,
            gameRunning: false,
            gamePaused: false
        };

        let gameObjects = {
            player: null,
            enemies: [],
            items: [],
            ground: canvas.height - 50
        };

        // PLAYER
        class Player {
            constructor() {
                this.x = 50;
                this.y = gameObjects.ground - 40;
                this.width = 30;
                this.height = 40;
                this.velocityX = 0;
                this.velocityY = 0;
                this.isJumping = false;
            }

            update() {
                // Gravity
                this.velocityY += 0.6;
                
                // Movement
                this.x += this.velocityX;
                this.y += this.velocityY;

                // Friction
                this.velocityX *= 0.95;

                // Boundaries
                if (this.x < 0) this.x = 0;
                if (this.x + this.width > canvas.width) this.x = canvas.width - this.width;

                // Ground collision
                if (this.y + this.height >= gameObjects.ground) {
                    this.y = gameObjects.ground - this.height;
                    this.velocityY = 0;
                    this.isJumping = false;
                }
            }

            jump() {
                if (!this.isJumping) {
                    this.velocityY = -12;
                    this.isJumping = true;
                }
            }

            draw() {
                ctx.fillStyle = '#00ff00';
                ctx.fillRect(this.x, this.y, this.width, this.height);

                // Eyes
                ctx.fillStyle = '#000';
                ctx.fillRect(this.x + 8, this.y + 10, 5, 5);
                ctx.fillRect(this.x + 17, this.y + 10, 5, 5);
            }
        }

        // ENEMY
        class Enemy {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.width = 25;
                this.height = 25;
                this.speed = 2;
                this.direction = 1;
                this.range = 150;
                this.startX = x;
            }

            update() {
                this.x += this.speed * this.direction;

                if (Math.abs(this.x - this.startX) > this.range) {
                    this.direction *= -1;
                }
            }

            draw() {
                ctx.fillStyle = '#ff6b6b';
                ctx.fillRect(this.x, this.y, this.width, this.height);
            }

            getBox() {
                return { x: this.x, y: this.y, w: this.width, h: this.height };
            }
        }

        // COLLECTIBLE
        class Item {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.width = 20;
                this.height = 20;
                this.rotation = 0;
            }

            update() {
                this.rotation += 0.05;
                this.y += Math.sin(this.rotation) * 0.3;
            }

            draw() {
                ctx.save();
                ctx.translate(this.x + this.width / 2, this.y + this.height / 2);
                ctx.rotate(this.rotation);
                ctx.fillStyle = '#FFD700';
                ctx.beginPath();
                ctx.arc(0, 0, 10, 0, Math.PI * 2);
                ctx.fill();
                ctx.restore();
            }

            getBox() {
                return { x: this.x, y: this.y, w: this.width, h: this.height };
            }
        }

        // COLLISION DETECTION
        function checkCollision(box1, box2) {
            return box1.x < box2.x + box2.w &&
                   box1.x + box1.w > box2.x &&
                   box1.y < box2.y + box2.h &&
                   box1.y + box1.h > box2.y;
        }

        function getPlayerBox() {
            return {
                x: gameObjects.player.x,
                y: gameObjects.player.y,
                w: gameObjects.player.width,
                h: gameObjects.player.height
            };
        }

        // CREATE LEVEL
        function createLevel() {
            gameObjects.player = new Player();
            gameObjects.enemies = [];
            gameObjects.items = [];

            // Enemies
            for (let i = 0; i < 2 + gameState.level; i++) {
                gameObjects.enemies.push(new Enemy(150 + i * 300, gameObjects.ground - 50));
            }

            // Items
            for (let i = 0; i < 8; i++) {
                gameObjects.items.push(new Item(100 + i * 80, gameObjects.ground - 100));
            }
        }

        // UPDATE GAME
        function update() {
            if (!gameState.gameRunning || gameState.gamePaused) return;

            // Update objects
            gameObjects.player.update();
            gameObjects.enemies.forEach(e => e.update());
            gameObjects.items.forEach(i => i.update());

            // Check collisions with enemies
            gameObjects.enemies.forEach(enemy => {
                if (checkCollision(getPlayerBox(), enemy.getBox())) {
                    gameState.lives--;
                    if (gameState.lives <= 0) {
                        gameOver();
                        return;
                    }
                    gameObjects.player = new Player();
                    updateHUD();
                }
            });

            // Check collisions with items
            gameObjects.items = gameObjects.items.filter(item => {
                if (checkCollision(getPlayerBox(), item.getBox())) {
                    gameState.score += 10;
                    return false;
                }
                return true;
            });

            // Level complete
            if (gameObjects.items.length === 0) {
                levelComplete();
            }

            updateHUD();
        }

        // DRAW GAME
        function draw() {
            // Background
            const grad = ctx.createLinearGradient(0, 0, 0, canvas.height);
            grad.addColorStop(0, '#87ceeb');
            grad.addColorStop(1, '#e0f6ff');
            ctx.fillStyle = grad;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Ground
            ctx.fillStyle = '#8B7355';
            ctx.fillRect(0, gameObjects.ground, canvas.width, canvas.height - gameObjects.ground);

            // Draw objects
            gameObjects.player.draw();
            gameObjects.enemies.forEach(e => e.draw());
            gameObjects.items.forEach(i => i.draw());
        }

        // GAME LOOP
        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        // START GAME
        function startGame() {
            const username = document.getElementById('username').value.trim();
            const phone = document.getElementById('phone').value.trim();

            if (!username || !phone) {
                alert('Please enter username and phone');
                return;
            }

            gameState.username = username;
            gameState.phone = phone;
            gameState.score = 0;
            gameState.level = 1;
            gameState.lives = 3;
            gameState.gameRunning = true;
            gameState.gamePaused = false;

            document.getElementById('loginScreen').classList.add('hidden');
            document.getElementById('gameOverScreen').classList.add('hidden');
            document.getElementById('pauseScreen').classList.add('hidden');

            createLevel();
            updateHUD();
            gameLoop();
        }

        // PAUSE GAME
        function pauseGame() {
            gameState.gamePaused = true;
            document.getElementById('pauseScreen').classList.remove('hidden');
            document.getElementById('pauseText').textContent = `Score: ${gameState.score} | Level: ${gameState.level}`;
        }

        function resumeGame() {
            gameState.gamePaused = false;
            document.getElementById('pauseScreen').classList.add('hidden');
        }

        // GAME OVER
        function gameOver() {
            gameState.gameRunning = false;
            document.getElementById('gameOverScreen').classList.remove('hidden');
            document.getElementById('gameOverText').textContent = `Score: ${gameState.score}\nLevel: ${gameState.level}`;
        }

        // LEVEL COMPLETE
        function levelComplete() {
            gameState.level++;
            gameState.score += 100;
            createLevel();
        }

        // RESTART GAME
        function restartGame() {
            startGame();
        }

        // QUIT GAME
        function quitGame() {
            gameState.gameRunning = false;
            gameState.gamePaused = false;
            document.getElementById('loginScreen').classList.remove('hidden');
            document.getElementById('pauseScreen').classList.add('hidden');
            document.getElementById('username').value = '';
            document.getElementById('phone').value = '';
        }

        // UPDATE HUD
        function updateHUD() {
            document.getElementById('scoreText').textContent = gameState.score;
            document.getElementById('levelText').textContent = gameState.level;
            
            let hearts = '';
            for (let i = 0; i < gameState.lives; i++) hearts += '❤️';
            document.getElementById('livesText').textContent = hearts;
        }

        // KEYBOARD CONTROLS
        const keys = {};

        document.addEventListener('keydown', (e) => {
            keys[e.key] = true;

            if (!gameState.gameRunning) return;

            if (e.key === 'ArrowLeft' || e.key === 'a') {
                gameObjects.player.velocityX = -5;
            }
            if (e.key === 'ArrowRight' || e.key === 'd') {
                gameObjects.player.velocityX = 5;
            }
            if (e.key === ' ') {
                gameObjects.player.jump();
                e.preventDefault();
            }
            if (e.key === 'p' || e.key === 'P') {
                pauseGame();
            }
        });

        document.addEventListener('keyup', (e) => {
            keys[e.key] = false;

            if (e.key === 'ArrowLeft' || e.key === 'ArrowRight' || e.key === 'a' || e.key === 'd') {
                gameObjects.player.velocityX *= 0.8;
            }
        });

        // WINDOW RESIZE
        window.addEventListener('resize', () => {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
            gameObjects.ground = canvas.height - 50;
        });
    </script>
</body>
</html>
