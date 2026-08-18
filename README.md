game-thai-vowel
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
