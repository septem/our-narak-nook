<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Our Narak Nook | 我們的專屬小窩</title>
    <style>
        :root {
            --day-bg: linear-gradient(180deg, #FFEFBA 0%, #FFFFFF 100%);
            --night-bg: linear-gradient(180deg, #2C3E50 0%, #000000 100%);
            --text-color: #5D4037;
            --accent: #FF8A80;
        }

        body, html {
            margin: 0; padding: 0; width: 100%; height: 100%;
            font-family: 'Segoe UI', "Microsoft JhengHei", sans-serif;
            transition: 1s ease; overflow: hidden;
        }

        /* 遊戲場景 */
        #room {
            width: 100vw; height: 100vh;
            display: flex; justify-content: center; align-items: center;
            background: var(--day-bg);
            position: relative;
        }

        .night-mode #room { background: var(--night-bg); }
        .night-mode .status-text { color: #DDD !important; }

        /* 虛擬小人 */
        .character-container {
            display: flex; gap: 100px; z-index: 10;
        }

        .character {
            display: flex; flex-direction: column; align-items: center;
            transition: 0.5s;
        }

        .sprite {
            font-size: 80px;
            filter: drop-shadow(0 10px 10px rgba(0,0,0,0.1));
            animation: float 3s infinite ease-in-out;
        }

        @keyframes float {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-15px); }
        }

        .label {
            background: rgba(255,255,255,0.8);
            padding: 4px 12px; border-radius: 15px;
            margin-top: 10px; font-weight: bold; color: var(--text-color);
        }

        /* 底部選單 */
        #menu {
            position: fixed; bottom: 30px;
            background: rgba(255,255,255,0.9);
            padding: 20px 40px; border-radius: 50px;
            display: flex; gap: 20px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            backdrop-filter: blur(10px);
            left: 50%; transform: translateX(-50%);
            z-index: 100;
        }

        .btn {
            border: none; background: #FFF1F0;
            padding: 10px 20px; border-radius: 20px;
            cursor: pointer; transition: 0.3s;
            display: flex; flex-direction: column; align-items: center;
            color: #5D4037;
        }

        .btn:hover { background: var(--accent); color: white; transform: translateY(-5px); }
        .btn-icon { font-size: 24px; }
        .btn-text { font-size: 12px; margin-top: 4px; }

        /* 特效層 */
        #fx-canvas {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            pointer-events: none;
        }
    </style>
</head>
<body>

<div id="room">
    <canvas id="fx-canvas"></canvas>

    <div class="character-container">
        <div class="character" id="me">
            <div class="sprite" id="my-sprite">😊</div>
            <div class="label">我 (在線)</div>
            <div class="status-text" id="my-status" style="margin-top: 5px; color: #888;">正在發呆</div>
        </div>

        <div class="character" id="partner">
            <div class="sprite" id="partner-sprite">💤</div>
            <div class="label">另一半</div>
            <div class="status-text" id="partner-status" style="margin-top: 5px; color: #888;">正在睡覺</div>
        </div>
    </div>
</div>

<div id="menu">
    <button class="btn" onclick="updateStatus('awake', '☀️', '我醒了！')">
        <span class="btn-icon">☀️</span><span class="btn-text">起床</span>
    </button>
    <button class="btn" onclick="updateStatus('work', '👨‍💻', '認真工作中')">
        <span class="btn-icon">💻</span><span class="btn-text">工作</span>
    </button>
    <button class="btn" onclick="updateStatus('eat', '🍜', '填飽肚子中')">
        <span class="btn-icon">🍜</span><span class="btn-text">吃飯</span>
    </button>
    <button class="btn" onclick="updateStatus('sleep', '😴', '晚安安...')">
        <span class="btn-icon">🌙</span><span class="btn-text">睡覺</span>
    </button>
</div>

<script>
    const partnerData = {
        'awake': { sprite: '🐶', text: '太好了你醒了！' },
        'work': { sprite: '☕', text: '加油，我也在忙' },
        'eat': { sprite: '🍱', text: '看起來很好吃～' },
        'sleep': { sprite: '💤', text: '呼呼大睡中' }
    };

    function updateStatus(type, emoji, text) {
        // 更新我的狀態
        document.getElementById('my-sprite').innerText = emoji;
        document.getElementById('my-status').innerText = text;

        // 環境變化
        if (type === 'sleep') {
            document.body.classList.add('night-mode');
            startStarEffect();
        } else {
            document.body.classList.remove('night-mode');
            stopEffect();
        }

        // 模擬另一半的自動回饋 (實際開發需對接 Firebase)
        setTimeout(() => {
            const response = partnerData[type] || partnerData['awake'];
            document.getElementById('partner-sprite').innerText = response.sprite;
            document.getElementById('partner-status').innerText = response.text;
            
            // 彈跳特效
            const partner = document.getElementById('partner');
            partner.style.transform = 'scale(1.2)';
            setTimeout(() => partner.style.transform = 'scale(1)', 200);
        }, 1000);
    }

    // 簡單的 Canvas 星星特效
    const canvas = document.getElementById('fx-canvas');
    const ctx = canvas.getContext('2d');
    let animationId;

    function startStarEffect() {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
        const stars = Array.from({length: 50}, () => ({
            x: Math.random() * canvas.width,
            y: Math.random() * canvas.height,
            size: Math.random() * 2,
            blink: Math.random() * 0.05
        }));

        function animate() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = "white";
            stars.forEach(s => {
                ctx.globalAlpha = Math.abs(Math.sin(Date.now() * s.blink));
                ctx.beginPath();
                ctx.arc(s.x, s.y, s.size, 0, Math.PI * 2);
                ctx.fill();
            });
            animationId = requestAnimationFrame(animate);
        }
        animate();
    }

    function stopEffect() {
        cancelAnimationFrame(animationId);
        ctx.clearRect(0, 0, canvas.width, canvas.height);
    }

    window.onresize = () => {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
    };
</script>

</body>
</html>
