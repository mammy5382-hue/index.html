# game-thai-vowel
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>กิจกรรมสนุก: วาดสระใอไม้ม้วนบนอากาศ</title>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js" crossorigin="anonymous"></script>
    <style>
        body {
            font-family: 'Sarabun', sans-serif;
            text-align: center;
            background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
            margin: 0;
            padding: 15px;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }
        h1 {
            color: #d63031;
            margin-bottom: 5px;
            font-size: 24px;
        }
        p {
            color: #2d3436;
            margin-top: 0;
            font-size: 15px;
        }
        .app-container {
            position: relative;
            display: inline-block;
            margin-top: 10px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.2);
            border-radius: 20px;
            overflow: hidden;
            background: #000;
            max-width: 100%;
        }
        video {
            display: block;
            transform: scaleX(-1);
            width: 100%;
            max-width: 640px;
            height: auto;
        }
        canvas {
            position: absolute;
            left: 0;
            top: 0;
            transform: scaleX(-1);
            pointer-events: none;
            width: 100%;
            height: 100%;
        }
        .ui-overlay {
            position: absolute;
            top: 15px;
            left: 15px;
            right: 15px;
            display: flex;
            justify-content: space-between;
            pointer-events: none;
        }
        .badge {
            background: rgba(255, 255, 255, 0.9);
            padding: 8px 15px;
            border-radius: 20px;
            font-weight: bold;
            color: #2d3436;
            font-size: 14px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.1);
        }
        /* ไกด์ไลน์รูปสระ ใ- ลางๆ บนจอ */
        .target-guide {
            position: absolute;
            right: 30px;
            top: 80px;
            font-size: 80px;
            font-weight: bold;
            color: rgba(255, 255, 255, 0.4);
            pointer-events: none;
            text-shadow: 0 0 10px rgba(0,0,0,0.5);
        }
        .status-box {
            margin-top: 15px;
            font-size: 18px;
            font-weight: bold;
            color: #0984e3;
            background: white;
            padding: 10px 20px;
            border-radius: 30px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.05);
            display: inline-block;
        }
        .modal {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.7);
            justify-content: center;
            align-items: center;
            z-index: 1000;
            flex-direction: column;
        }
        .modal-content {
            background: white;
            padding: 20px;
            border-radius: 20px;
            text-align: center;
            max-width: 90%;
            box-shadow: 0 10px 30px rgba(0,0,0,0.3);
        }
        .modal-content img {
            max-width: 100%;
            border-radius: 12px;
            border: 4px solid #ff7675;
            margin-bottom: 15px;
        }
        .btn {
            background: #00b894;
            color: white;
            border: none;
            padding: 12px 25px;
            font-size: 16px;
            font-weight: bold;
            border-radius: 30px;
            cursor: pointer;
            box-shadow: 0 4px 10px rgba(0,184,148,0.4);
            transition: 0.2s;
            margin: 5px;
            text-decoration: none;
            display: inline-block;
        }
        .btn-retry {
            background: #dfe6e9;
            color: #2d3436;
            box-shadow: none;
        }
    </style>
</head>
<body>

    <h1>🌟 วาดสระใอไม้ม้วนบนอากาศ 🌟</h1>
    <p>ยกนิ้วชี้ขึ้นมา วาดตามตัวอย่างเป้าหมายด้านขวา</p>

    <div class="app-container">
        <video id="webcam" autoplay playsinline width="640" height="480"></video>
        <canvas id="output_canvas" width="640" height="480"></canvas>
        <div class="target-guide">ใ-</div>
        <div class="ui-overlay">
            <div class="badge" id="stepBadge">เป้าหมาย: วาดสระใอ</div>
            <div class="badge" id="timerBadge" style="display: none;">ถ่ายรูปใน: 3</div>
        </div>
    </div>

    <div class="status-box" id="statusText">กำลังเตรียมกล้อง...</div>

    <div class="modal" id="resultModal">
        <div class="modal-content">
            <h2 style="color: #00b894; margin-top: 0;">🎉 เก่งมาก! วาดสำเร็จแล้ว</h2>
            <canvas id="photoCanvas" style="display:none;"></canvas>
            <img id="resultImage" alt="รูปถ่ายผลงาน">
            <div>
                <a id="downloadBtn" class="btn" download="สระใอไม้ม้วนของฉัน.png">📥 บันทึกรูปภาพ</a>
                <button class="btn btn-retry" onclick="resetActivity()">🔄 เล่นอีกครั้ง</button>
            </div>
        </div>
    </div>

    <script>
        const videoElement = document.getElementById('webcam');
        const canvasElement = document.getElementById('output_canvas');
        const canvasCtx = canvasElement.getContext('2d');
        const statusText = document.getElementById('statusText');
        const stepBadge = document.getElementById('stepBadge');
        const timerBadge = document.getElementById('timerBadge');
        const resultModal = document.getElementById('resultModal');
        const resultImage = document.getElementById('resultImage');
        const downloadBtn = document.getElementById('downloadBtn');

        let drawingPoints = [];
        let stars = []; // เก็บตำแหน่งดาวเอฟเฟกต์
        let gameState = 'DRAWING';
        let countdownTimer = 3;
        let countdownInterval = null;

        // ระบบเสียงเอฟเฟกต์สังเคราะห์ (Web Audio API)
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        
        function playSound(type) {
            if (audioCtx.state === 'suspended') { audioCtx.resume(); }
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.connect(gain);
            gain.connect(audioCtx.destination);
            
            let now = audioCtx.currentTime;
            if (type === 'pop') {
                osc.frequency.setValueAtTime(400, now);
                osc.frequency.exponentialRampToValueAtTime(800, now + 0.1);
                gain.gain.setValueAtTime(0.3, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.1);
                osc.start(now);
                osc.stop(now + 0.1);
            } else if (type === 'success') {
                // เสียงดนตรีสำเร็จสั้นๆ
                [523.25, 659.25, 783.99, 1046.50].forEach((freq, i) => {
                    let o = audioCtx.createOscillator();
                    let g = audioCtx.createGain();
                    o.connect(g); g.connect(audioCtx.destination);
                    o.frequency.setValueAtTime(freq, now + i * 0.1);
                    g.gain.setValueAtTime(0.2, now + i * 0.1);
                    g.gain.exponentialRampToValueAtTime(0.01, now + i * 0.1 + 0.2);
                    o.start(now + i * 0.1);
                    o.stop(now + i * 0.1 + 0.2);
                });
            }
        }

        function onResults(results) {
            canvasCtx.save();
            canvasCtx.clearRect(0, 0, canvasElement.width, canvasElement.height);
            canvasCtx.drawImage(results.image, 0, 0, canvasElement.width, canvasElement.height);

            // วาดเอฟเฟกต์ดาวกระจายหากวาดสำเร็จแล้ว
            if (gameState === 'SUCCESS' || gameState === 'COUNTDOWN' || gameState === 'DONE') {
                drawStars();
            }

            if (gameState === 'DONE') {
                canvasCtx.restore();
                return;
            }

            if (results.multiHandLandmarks && results.multiHandLandmarks.length > 0 && gameState === 'DRAWING') {
                statusText.innerText = "กำลังวาดเส้น... วนหัวแล้วลากขึ้นเลย!";
                statusText.style.color = "#e17055";

                const landmarks = results.multiHandLandmarks[0];
                const indexFingerTip = landmarks[8];
                
                const x = indexFingerTip.x * canvasElement.width;
                const y = indexFingerTip.y * canvasElement.height;

                drawingPoints.push({ x, y });
                if (drawingPoints.length > 100) drawingPoints.shift();

                checkDrawingCompletion();
            } else if (gameState === 'DRAWING') {
                statusText.innerText = "ยกนิ้วชี้ขึ้นมาหน้ากล้องเพื่อเริ่มวาด";
                statusText.style.color = "#0984e3";
            }

            // วาดเส้นร่องรอยนิ้วมือ
            if (drawingPoints.length > 1) {
                canvasCtx.beginPath();
                canvasCtx.strokeStyle = '#ff7675';
                canvasCtx.lineWidth = 12;
                canvasCtx.lineCap = 'round';
                canvasCtx.lineJoin = 'round';

                for (let i = 0; i < drawingPoints.length; i++) {
                    if (i === 0) canvasCtx.moveTo(drawingPoints[i].x, drawingPoints[i].y);
                    else canvasCtx.lineTo(drawingPoints[i].x, drawingPoints[i].y);
                }
                canvasCtx.stroke();
            }

            canvasCtx.restore();
        }

        function checkDrawingCompletion() {
            if (drawingPoints.length > 40 && gameState === 'DRAWING') {
                let startY = drawingPoints[0].y;
                let endY = drawingPoints[drawingPoints.length - 1].y;
                
                if (startY - endY > 50) { 
                    triggerSuccessSequence();
                }
            }
        }

        function triggerSuccessSequence() {
            gameState = 'SUCCESS';
            playSound('success');
            createStars();

            stepBadge.innerText = "⭐ สำเร็จแล้ว!";
            timerBadge.style.display = "block";
            statusText.innerText = "เก่งมาก! ทำท่าสวยๆ เตรียมถ่ายรูป...";
            statusText.style.color = "#00b894";

            setTimeout(() => {
                gameState = 'COUNTDOWN';
                countdownTimer = 3;
                timerBadge.innerText = `ถ่ายรูปใน: ${countdownTimer}`;

                countdownInterval = setInterval(() => {
                    countdownTimer--;
                    timerBadge.innerText = `ถ่ายรูปใน: ${countdownTimer}`;
                    playSound('pop');
                    
                    if (countdownTimer <= 0) {
                        clearInterval(countdownInterval);
                        timerBadge.style.display = "none";
                        captureSnapshot();
                    }
                }, 1000);
            }, 1000);
        }

        // สร้างตำแหน่งดาวกระจาย
        function createStars() {
            stars = [];
            for (let i = 0; i < 20; i++) {
                stars.push({
                    x: Math.random() * canvasElement.width,
                    y: Math.random() * canvasElement.height,
                    size: Math.random() * 20 + 10,
                    angle: Math.random() * Math.PI * 2
                });
            }
        }

        // วาดรูปดาวบนแคนวาส
        function drawStars() {
            stars.forEach(star => {
                canvasCtx.save();
                canvasCtx.translate(star.x, star.y);
                canvasCtx.rotate(star.angle);
                canvasCtx.beginPath();
                canvasCtx.fillStyle = '#f1c40f';
                let spikes = 5;
                let outerRadius = star.size;
                let innerRadius = star.size / 2;
                for (let i = 0; i < spikes * 2; i++) {
                    let r = (i % 2 === 0) ? outerRadius : innerRadius;
                    let a = (i * Math.PI) / spikes;
                    let x = Math.cos(a) * r;
                    let y = Math.sin(a) * r;
                    if (i === 0) canvasCtx.moveTo(x, y);
                    else canvasCtx.lineTo(x, y);
                }
                canvasCtx.closePath();
                canvasCtx.fill();
                canvasCtx.restore();
            });
        }

        function captureSnapshot() {
            gameState = 'DONE';
            
            const photoCanvas = document.getElementById('photoCanvas');
            photoCanvas.width = videoElement.videoWidth || 640;
            photoCanvas.height = videoElement.videoHeight || 480;
            const pCtx = photoCanvas.getContext('2d');

            pCtx.save();
            pCtx.scale(-1, 1);
            pCtx.drawImage(videoElement, -photoCanvas.width, 0, photoCanvas.width, photoCanvas.height);
            pCtx.restore();

            // ตกแต่งกรอบรูปและคำศัพท์
            pCtx.lineWidth = 20;
            pCtx.strokeStyle = '#ff7675';
            pCtx.strokeRect(0, 0, photoCanvas.width, photoCanvas.height);

            pCtx.fillStyle = 'rgba(255, 255, 255, 0.9)';
            pCtx.fillRect(20, 20, photoCanvas.width - 40, 70);

            pCtx.font = 'bold 26px "Sarabun", sans-serif';
            pCtx.fillStyle = '#d63031';
            pCtx.textAlign = 'center';
            pCtx.fillText("⭐ สระใอไม้ม้วน: ใบ, ใส, ให้, ไปไหน ⭐", photoCanvas.width / 2, 65);

            const dataUrl = photoCanvas.toDataURL('image/png');
            resultImage.src = dataUrl;
            downloadBtn.href = dataUrl;
            resultModal.style.display = 'flex';
        }

        function resetActivity() {
            resultModal.style.display = 'none';
            drawingPoints = [];
            stars = [];
            gameState = 'DRAWING';
            stepBadge.innerText = "เป้าหมาย: วาดสระใอ";
            statusText.innerText = "พร้อมแล้ว! ยกนิ้วชี้ขึ้นมาวาดบนอากาศได้เลย";
        }

        const hands = new Hands({
            locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`
        });

        hands.setOptions({
            maxNumHands: 1,
            modelComplexity: 1,
            minDetectionConfidence: 0.6,
            minTrackingConfidence: 0.6
        });

        hands.onResults(onResults);

        const camera = new Camera(videoElement, {
            onFrame: async () => {
                await hands.send({ image: videoElement });
            },
            width: 640,
            height: 480
        });

        camera.start()
            .then(() => {
                statusText.innerText = "พร้อมแล้ว! ยกนิ้วชี้ขึ้นมาวาดบนอากาศได้เลย";
            })
            .catch((err) => {
                statusText.innerText = "ไม่สามารถเปิดกล้องได้ กรุณาอนุญาตการใช้งานกล้อง";
                console.error(err);
            });
    </script>
</body>
</html>
