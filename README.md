<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dino Jump Action & Payment Hub</title>
    <!-- QRCode.js Library for secure client-side QR generation -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <style>
        :root {
            --bg-color: #f7f7f7;
            --text-color: #333333;
            --accent-color: #28a745;
            --accent-hover: #218838;
            --dino-color: #535353;
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
            max-width: 800px;
            background: #ffffff;
            padding: 25px;
            border-radius: 12px;
            box-shadow: 0 8px 24px rgba(0,0,0,0.08);
            text-align: center;
        }

        h1 {
            font-size: 1.8rem;
            margin-bottom: 5px;
            color: var(--dino-color);
        }

        .instructions {
            font-size: 0.9rem;
            color: #666;
            margin-bottom: 20px;
        }

        /* Game Box Wrapper */
        .game-wrapper {
            position: relative;
            width: 100%;
            background: #fcfcfc;
            border: 2px solid #e0e0e0;
            border-radius: 8px;
            overflow: hidden;
            margin-bottom: 25px;
        }

        canvas {
            display: block;
            width: 100%;
            height: 250px;
            background-color: #fafafa;
        }

        /* Support / Payment Section */
        .payment-section {
            border-top: 1px solid #eee;
            padding-top: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .payment-section h2 {
            font-size: 1.3rem;
            margin-bottom: 15px;
        }

        .payment-controls {
            display: flex;
            gap: 10px;
            margin-bottom: 15px;
            flex-wrap: wrap;
            justify-content: center;
        }

        .amt-input {
            width: 120px;
            padding: 10px;
            border: 2px solid #ccc;
            border-radius: 6px;
            font-size: 1rem;
            outline: none;
            text-align: center;
            transition: border-color 0.2s;
        }

        .amt-input:focus {
            border-color: var(--accent-color);
        }

        .pay-btn {
            background-color: var(--accent-color);
            color: #fff;
            border: none;
            padding: 10px 20px;
            font-size: 1rem;
            font-weight: 600;
            border-radius: 6px;
            cursor: pointer;
            transition: background 0.2s;
        }

        .pay-btn:hover {
            background-color: var(--accent-hover);
        }

        /* QR Modal Container */
        .qr-container {
            display: none;
            flex-direction: column;
            align-items: center;
            margin-top: 15px;
            padding: 15px;
            background: #f9f9f9;
            border-radius: 8px;
            border: 1px dashed #bbb;
            animation: fadeIn 0.3s ease-in-out;
        }

        #qrcode {
            padding: 10px;
            background: #fff;
            border: 1px solid #ddd;
            border-radius: 4px;
        }

        .qr-note {
            font-size: 0.85rem;
            color: #555;
            margin-top: 10px;
            font-weight: 500;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
    </style>
</head>
<body>

<div class="container">
    <h1>Dino Jump Action</h1>
    <p class="instructions">Press <strong>Spacebar</strong>, <strong>Up Arrow</strong>, or <strong>Tap/Click inside the box</strong> to Jump and avoid the obstacles.</p>

    <!-- Frontend Game Platform -->
    <div class="game-wrapper" id="gameBox">
        <canvas id="gameCanvas" width="800" height="250"></canvas>
    </div>

    <!-- Payment Configuration Panel -->
    <div class="payment-section">
        <h2>Enjoyed the game? Support the Developer!</h2>
        <div class="payment-controls">
            <input type="number" id="payAmount" class="amt-input" value="10" min="1" placeholder="Amount (₹)">
            <button class="pay-btn" onclick="generatePaymentQR()">Generate Payment QR</button>
        </div>

        <div class="qr-container" id="qrWrapper">
            <div id="qrcode"></div>
            <p class="qr-note" id="qrMetadata">Scan via Google Pay, PhonePe, Paytm, or BHIM app</p>
        </div>
    </div>
</div>

<script>
    // --- GAME ENGINE CODING ---
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

    // Game variables
    let dino, obstacles, score, gameSpeed, isGameOver, isStarted, animationId;
    
    const gravity = 0.6;

    class Dino {
        constructor() {
            this.x = 50;
            this.y = canvas.height - 40;
            this.width = 30;
            this.height = 40;
            this.vy = 0;
            this.jumpForce = 12;
            this.isGrounded = true;
        }

        jump() {
            if (this.isGrounded) {
                this.vy = -this.jumpForce;
                this.isGrounded = false;
            }
        }

        update() {
            this.vy += gravity;
            this.y += this.vy;

            // Constrain to Ground Level
            if (this.y >= canvas.height - this.height - 10) {
                this.y = canvas.height - this.height - 10;
                this.vy = 0;
                this.isGrounded = true;
            }
        }

        draw() {
            ctx.fillStyle = '#535353';
            // Draw stylized retro block dino
            ctx.fillRect(this.x, this.y, this.width, this.height);
            // Eye
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(this.x + 20, this.y + 8, 4, 4);
        }
    }

    class Obstacle {
        constructor() {
            this.x = canvas.width;
            this.width = 15 + Math.random() * 20;
            this.height = 30 + Math.random() * 25;
            this.y = canvas.height - this.height - 10;
        }

        update() {
            this.x -= gameSpeed;
        }

        draw() {
            ctx.fillStyle = '#a71d2a'; // Contrast red obstacles
            ctx.fillRect(this.x, this.y, this.width, this.height);
        }
    }

    function init() {
        dino = new Dino();
        obstacles = [];
        score = 0;
        gameSpeed = 6;
        isGameOver = false;
        isStarted = false;
        drawStartScreen();
    }

    function drawStartScreen() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        drawGround();
        dino.draw();
        
        ctx.fillStyle = '#333';
        ctx.font = 'bold 20px sans-serif';
        ctx.fillText('PRESS JUMP TO RUN', canvas.width / 2 - 100, canvas.height / 2);
    }

    function drawGround() {
        ctx.strokeStyle = '#777';
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.moveTo(0, canvas.height - 10);
        ctx.lineTo(canvas.width, canvas.height - 10);
        ctx.stroke();
    }

    let spawnTimer = 0;
    function gameLoop() {
        if (isGameOver) return;

        ctx.clearRect(0, 0, canvas.width, canvas.height);
        drawGround();

        // Engine Updates
        dino.update();
        dino.draw();

        // Increment Difficulty slowly
        gameSpeed += 0.001;

        // Procedural obstacle generation
        spawnTimer++;
        if (spawnTimer > Math.max(50, 120 - gameSpeed * 2)) {
            if (Math.random() > 0.4) {
                obstacles.push(new Obstacle());
                spawnTimer = 0;
            }
        }

        // Processing Obstacles
        for (let i = obstacles.length - 1; i >= 0; i--) {
            obstacles[i].update();
            obstacles[i].draw();

            // Collision Check (AABB Algorithm)
            if (
                dino.x < obstacles[i].x + obstacles[i].width &&
                dino.x + dino.width > obstacles[i].x &&
                dino.y < obstacles[i].y + obstacles[i].height &&
                dino.y + dino.height > obstacles[i].y
            ) {
                endGame();
            }

            // Garbage Collection and Score Tracking
            if (obstacles[i].x + obstacles[i].width < 0) {
                obstacles.splice(i, 1);
                score += 10;
            }
        }

        // Live HUD Metrics
        ctx.fillStyle = '#333';
        ctx.font = '16px monospace';
        ctx.fillText(`SCORE: ${score}`, 20, 30);

        animationId = requestAnimationFrame(gameLoop);
    }

    function triggerJump() {
        if (!isStarted && !isGameOver) {
            isStarted = true;
            gameLoop();
        }
        if (isGameOver) {
            init();
        } else {
            dino.jump();
        }
    }

    function endGame() {
        isGameOver = true;
        cancelAnimationFrame(animationId);
        
        ctx.fillStyle = 'rgba(0,0,0,0.6)';
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        ctx.fillStyle = '#fff';
        ctx.font = 'bold 24px sans-serif';
        ctx.fillText('GAME OVER', canvas.width / 2 - 70, canvas.height / 2 - 10);
        
        ctx.font = '16px sans-serif';
        ctx.fillText(`Final Score: ${score}`, canvas.width / 2 - 50, canvas.height / 2 + 20);
        ctx.fillText('Tap / Press Space to Restart', canvas.width / 2 - 100, canvas.height / 2 + 50);
    }

    // Input Controllers
    window.addEventListener('keydown', (e) => {
        if (e.code === 'Space' || e.code === 'ArrowUp') {
            e.preventDefault(); // Stop window scrolling
            triggerJump();
        }
    });
    document.getElementById('gameBox').addEventListener('click', triggerJump);


    // --- BACKEND / PAYMENT ROUTING ---
    const targetUPI = "adey6547@axl";
    const payeeName = "Dino Jump Developer";
    let qrInstance = null;

    function generatePaymentQR() {
        const amtInput = document.getElementById('payAmount');
        const amount = parseFloat(amtInput.value) || 10;
        const qrWrapper = document.getElementById('qrWrapper');
        const qrDiv = document.getElementById('qrcode');
        const metadataText = document.getElementById('qrMetadata');

        // Clean out previous instance data
        qrDiv.innerHTML = "";

        // Standardized National Payments Corporation of India (NPCI) deep link schema
        const upiURI = `upi://pay?pa=${encodeURIComponent(targetUPI)}&pn=${encodeURIComponent(payeeName)}&am=${amount.toFixed(2)}&cu=INR&tn=Support%20Dino%20Game`;

        // Render clean structural vector QR code inside client engine
        if (qrInstance === null || typeof QRCode !== 'undefined') {
            qrInstance = new QRCode(qrDiv, {
                text: upiURI,
                width: 180,
                height: 180,
                colorDark : "#000000",
                colorLight : "#ffffff",
                correctLevel : QRCode.CorrectLevel.M
            });
        }

        metadataText.innerText = `Scan to pay ₹${amount.toFixed(2)} directly to ${targetUPI}`;
        qrWrapper.style.display = "flex";
    }

    // Spin engine live on context deployment
    init();
</script>

</body>
</html>
