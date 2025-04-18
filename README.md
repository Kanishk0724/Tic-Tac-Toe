# Tic-Tac-Toe <br>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dodge Sphere by Kanishk</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background-color: #121212;
            font-family: Arial, sans-serif;
        }
        
        #game-container {
            position: relative;
            width: 100vw;
            height: 100vh;
        }
        
        canvas {
            display: block;
        }
        
        #score {
            position: absolute;
            top: 20px;
            left: 20px;
            color: white;
            font-size: 24px;
        }
        
        #game-over {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: rgba(0, 0, 0, 0.8);
            color: white;
            padding: 20px;
            border-radius: 10px;
            text-align: center;
            display: none;
        }
        
        button {
            background-color: #4CAF50;
            border: none;
            color: white;
            padding: 10px 20px;
            text-align: center;
            text-decoration: none;
            display: inline-block;
            font-size: 16px;
            margin: 10px 2px;
            cursor: pointer;
            border-radius: 5px;
        }
        
        #creator {
            position: absolute;
            bottom: 10px;
            left: 0;
            width: 100%;
            text-align: center;
            color: white;
            font-size: 14px;
        }

        #controls {
            position: absolute;
            bottom: 40px;
            left: 0;
            width: 100%;
            text-align: center;
            color: white;
            font-size: 14px;
        }
    </style>
</head>
<body>
    <div id="game-container">
        <canvas id="gameCanvas"></canvas>
        <div id="score">Score: 0</div>
        <div id="game-over">
            <h2>Game Over!</h2>
            <p id="final-score">Score: 0</p>
            <button id="restart">Play Again</button>
        </div>
        <div id="controls">Use arrow keys or WASD to move</div>
        <div id="creator">Created by Kanishk</div>
    </div>

    <script>
        // Game Setup
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const gameOverElement = document.getElementById('game-over');
        const finalScoreElement = document.getElementById('final-score');
        const restartButton = document.getElementById('restart');
        
        // Set canvas to window size
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
        
        let gameRunning = true;
        let score = 0;
        let obstacles = [];
        let spawnRate = 100; // Lower is faster
        let spawnCounter = 0;
        let pulseValue = 0;
        let pulseDirection = 1;
        
        // Player properties
        const player = {
            x: canvas.width / 2,
            y: canvas.height / 2,
            radius: 25, // Slightly larger radius
            baseColor: '#00FFFF', // Bright cyan color
            outlineColor: '#FF00FF', // Magenta outline
            innerColor: '#FFFFFF', // White inner circle
            speed: 5,
            dx: 0,
            dy: 0
        };
        
        // Controls setup
        const keys = {
            ArrowUp: false,
            ArrowDown: false,
            ArrowLeft: false,
            ArrowRight: false,
            w: false,
            a: false,
            s: false,
            d: false
        };
        
        // Event listeners for keyboard controls
        document.addEventListener('keydown', function(e) {
            if (keys.hasOwnProperty(e.key)) {
                keys[e.key] = true;
            }
        });
        
        document.addEventListener('keyup', function(e) {
            if (keys.hasOwnProperty(e.key)) {
                keys[e.key] = false;
            }
        });
        
        // Window resize handler
        window.addEventListener('resize', function() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        });
        
        // Restart button click handler
        restartButton.addEventListener('click', restartGame);
        
        // Create a new obstacle
        function createObstacle() {
            const side = Math.floor(Math.random() * 4); // 0: top, 1: right, 2: bottom, 3: left
            let x, y;
            
            // Set initial position based on which side it's coming from
            switch(side) {
                case 0: // top
                    x = Math.random() * canvas.width;
                    y = -30;
                    break;
                case 1: // right
                    x = canvas.width + 30;
                    y = Math.random() * canvas.height;
                    break;
                case 2: // bottom
                    x = Math.random() * canvas.width;
                    y = canvas.height + 30;
                    break;
                case 3: // left
                    x = -30;
                    y = Math.random() * canvas.height;
                    break;
            }
            
            // Calculate direction towards player
            const angle = Math.atan2(player.y - y, player.x - x);
            const speed = Math.random() * 2 + 1;
            
            obstacles.push({
                x: x,
                y: y,
                radius: Math.random() * 10 + 10,
                color: getRandomColor(),
                dx: Math.cos(angle) * speed,
                dy: Math.sin(angle) * speed
            });
        }
        
        // Generate random color
        function getRandomColor() {
            const colors = ['#FF5252', '#FF4081', '#E040FB', '#7C4DFF', '#536DFE', '#448AFF', '#40C4FF', '#18FFFF', '#64FFDA', '#69F0AE', '#B2FF59', '#EEFF41', '#FFFF00', '#FFD740', '#FFAB40', '#FF6E40'];
            return colors[Math.floor(Math.random() * colors.length)];
        }
        
        // Check collision between two circles
        function checkCollision(x1, y1, r1, x2, y2, r2) {
            const dx = x2 - x1;
            const dy = y2 - y1;
            const distance = Math.sqrt(dx * dx + dy * dy);
            return distance < r1 + r2;
        }
        
        // Game over function
        function gameOver() {
            gameRunning = false;
            finalScoreElement.textContent = `Score: ${score}`;
            gameOverElement.style.display = 'block';
        }
        
        // Restart game function
        function restartGame() {
            gameRunning = true;
            score = 0;
            obstacles = [];
            player.x = canvas.width / 2;
            player.y = canvas.height / 2;
            gameOverElement.style.display = 'none';
            requestAnimationFrame(gameLoop);
        }
        
        // Update player position based on key presses
        function updatePlayerPosition() {
            // Reset velocity
            player.dx = 0;
            player.dy = 0;
            
            // Update based on key presses
            if (keys.ArrowUp || keys.w) player.dy = -player.speed;
            if (keys.ArrowDown || keys.s) player.dy = player.speed;
            if (keys.ArrowLeft || keys.a) player.dx = -player.speed;
            if (keys.ArrowRight || keys.d) player.dx = player.speed;
            
            // Update position
            player.x += player.dx;
            player.y += player.dy;
            
            // Keep player within canvas bounds
            if (player.x - player.radius < 0) player.x = player.radius;
            if (player.x + player.radius > canvas.width) player.x = canvas.width - player.radius;
            if (player.y - player.radius < 0) player.y = player.radius;
            if (player.y + player.radius > canvas.height) player.y = canvas.height - player.radius;
        }
        
        // Update pulse effect value
        function updatePulse() {
            pulseValue += 0.05 * pulseDirection;
            if (pulseValue >= 1) {
                pulseValue = 1;
                pulseDirection = -1;
            } else if (pulseValue <= 0) {
                pulseValue = 0;
                pulseDirection = 1;
            }
        }
        
        // Draw player with enhanced visibility
        function drawPlayer() {
            // Update the pulse effect
            updatePulse();
            
            // Draw outer glow effect
            const glowRadius = player.radius + 5 + pulseValue * 5;
            const gradient = ctx.createRadialGradient(
                player.x, player.y, player.radius,
                player.x, player.y, glowRadius
            );
            gradient.addColorStop(0, player.outlineColor);
            gradient.addColorStop(1, 'rgba(255, 0, 255, 0)');
            
            ctx.beginPath();
            ctx.arc(player.x, player.y, glowRadius, 0, Math.PI * 2);
            ctx.fillStyle = gradient;
            ctx.fill();
            ctx.closePath();
            
            // Draw outer circle
            ctx.beginPath();
            ctx.arc(player.x, player.y, player.radius, 0, Math.PI * 2);
            ctx.fillStyle = player.baseColor;
            ctx.fill();
            ctx.closePath();
            
            // Draw inner circle
            ctx.beginPath();
            ctx.arc(player.x, player.y, player.radius * 0.6, 0, Math.PI * 2);
            ctx.fillStyle = player.innerColor;
            ctx.fill();
            ctx.closePath();
            
            // Draw "YOU" text inside player sphere
            ctx.font = "bold 12px Arial";
            ctx.fillStyle = "#000000";
            ctx.textAlign = "center";
            ctx.textBaseline = "middle";
            ctx.fillText("YOU", player.x, player.y);
            
            // Draw movement direction indicator
            if (player.dx !== 0 || player.dy !== 0) {
                const angle = Math.atan2(player.dy, player.dx);
                const indicatorLength = player.radius * 1.5;
                const endX = player.x + Math.cos(angle) * indicatorLength;
                const endY = player.y + Math.sin(angle) * indicatorLength;
                
                ctx.beginPath();
                ctx.moveTo(player.x, player.y);
                ctx.lineTo(endX, endY);
                ctx.strokeStyle = "#FFFF00";
                ctx.lineWidth = 3;
                ctx.stroke();
                ctx.closePath();
                
                // Draw arrowhead
                const arrowSize = 8;
                const arrowAngle = 0.5;
                ctx.beginPath();
                ctx.moveTo(endX, endY);
                ctx.lineTo(
                    endX - arrowSize * Math.cos(angle - arrowAngle),
                    endY - arrowSize * Math.sin(angle - arrowAngle)
                );
                ctx.lineTo(
                    endX - arrowSize * Math.cos(angle + arrowAngle),
                    endY - arrowSize * Math.sin(angle + arrowAngle)
                );
                ctx.closePath();
                ctx.fillStyle = "#FFFF00";
                ctx.fill();
            }
        }
        
        // Update and draw obstacles
        function updateObstacles() {
            for (let i = 0; i < obstacles.length; i++) {
                const obstacle = obstacles[i];
                
                // Move obstacle
                obstacle.x += obstacle.dx;
                obstacle.y += obstacle.dy;
                
                // Draw obstacle
                ctx.beginPath();
                ctx.arc(obstacle.x, obstacle.y, obstacle.radius, 0, Math.PI * 2);
                ctx.fillStyle = obstacle.color;
                ctx.fill();
                ctx.closePath();
                
                // Check collision with player
                if (checkCollision(player.x, player.y, player.radius * 0.9, obstacle.x, obstacle.y, obstacle.radius)) {
                    gameOver();
                    return;
                }
                
                // Remove obstacles that are off-screen
                if (obstacle.x < -50 || obstacle.x > canvas.width + 50 || 
                    obstacle.y < -50 || obstacle.y > canvas.height + 50) {
                    obstacles.splice(i, 1);
                    i--;
                }
            }
        }
        
        // Main game loop
        function gameLoop() {
            if (!gameRunning) return;
            
            // Clear canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Spawn obstacles
            spawnCounter++;
            if (spawnCounter >= spawnRate) {
                createObstacle();
                spawnCounter = 0;
                // Increase difficulty
                if (spawnRate > 30) spawnRate--;
            }
            
            // Update player
            updatePlayerPosition();
            drawPlayer();
            
            // Update obstacles
            updateObstacles();
            
            // Update score
            score++;
            scoreElement.textContent = `Score: ${score}`;
            
            // Continue game loop
            requestAnimationFrame(gameLoop);
        }
        
        // Start the game
        restartGame();
    </script>
</body>
</html>
