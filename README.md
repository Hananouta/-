# -<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>算数シューティング！引き算編</title>
    <style>
        body { text-align: center; background: #000; color: #fff; margin: 0; overflow: hidden; }
        #game-canvas { background: #000; border: 4px solid #fff; display: block; margin: 10px auto; }
        #ui { font-size: 1.5em; font-weight: bold; }
    </style>
</head>
<body>
    <h1>算数シューティング！(ひきざん)</h1>
    <div id="ui">← →:移動, Space:ビーム | 残り:<span id="time">180</span>秒 | スコア:<span id="score">0</span></div>
    <div id="question" style="font-size: 3em; color: #fff; margin: 10px;">計算中...</div>
    <canvas id="game-canvas" width="600" height="500"></canvas>

    <script>
        const canvas = document.getElementById('game-canvas');
        const ctx = canvas.getContext('2d');
        let score = 0, timeLeft = 180, currentAnswer = 0;
        let shipX = 280;
        let enemies = [], beams = [];
        let lastCorrectSpawnTime = 0;
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();

        function playSound(type) {
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.connect(gain); gain.connect(audioCtx.destination);
            osc.frequency.value = type === 'hit' ? 880 : 220;
            gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.3);
            osc.start(); osc.stop(audioCtx.currentTime + 0.3);
        }

        // 引き算用の問題生成（10以下）
        function generateProblem() {
            let a = Math.floor(Math.random() * 9) + 1; // 引かれる数 1~9
            let b = Math.floor(Math.random() * a) + 0;     // 引く数 0~a
            currentAnswer = a - b;
            document.getElementById('question').innerText = `${a} - ${b} = ?`;
            lastCorrectSpawnTime = Date.now();
        }

        function drawShip(x) {
            ctx.fillStyle = '#fff';
            ctx.beginPath();
            ctx.moveTo(x + 20, 450); ctx.lineTo(x, 490); ctx.lineTo(x + 40, 490);
            ctx.closePath(); ctx.fill();
        }

        document.addEventListener('keydown', (e) => {
            if (e.key === 'ArrowLeft') shipX = Math.max(0, shipX - 30);
            if (e.key === 'ArrowRight') shipX = Math.min(560, shipX + 30);
            if (e.key === ' ') beams.push({ x: shipX + 20, y: 450 });
        });

        function update() {
            ctx.clearRect(0, 0, 600, 500);
            drawShip(shipX);

            if (Date.now() - lastCorrectSpawnTime > 2000) {
                let newX = Math.random() * 550;
                enemies.push({ val: currentAnswer, x: newX, y: 0 });
                lastCorrectSpawnTime = Date.now();
            }

            if (Math.random() < 0.01) { 
                let newX = Math.random() * 550;
                if (enemies.every(en => Math.abs(en.x - newX) > 60)) {
                    // 間違った答えを0~10の範囲で選ぶ
                    let wrongVal = Math.floor(Math.random() * 11);
                    enemies.push({ val: wrongVal, x: newX, y: 0 });
                }
            }

            beams.forEach((b, i) => {
                b.y -= 10;
                ctx.fillStyle = '#ff0'; ctx.fillRect(b.x - 2, b.y, 4, 15);
                enemies.forEach((en, j) => {
                    if (b.y < en.y + 40 && b.x > en.x && b.x < en.x + 40) {
                        if (en.val === currentAnswer) {
                            score += 100; playSound('hit'); generateProblem();
                        } else {
                            score = Math.max(0, score - 50); playSound('miss');
                        }
                        enemies.splice(j, 1); beams.splice(i, 1);
                    }
                });
                if (b.y < 0) beams.splice(i, 1);
            });

            enemies.forEach((en, i) => {
                en.y += 0.7;
                ctx.fillStyle = '#fff';
                ctx.font = 'bold 50px Arial';
                ctx.fillText(en.val, en.x, en.y);
                if (en.y > 500) enemies.splice(i, 1);
            });

            document.getElementById('score').innerText = score;
            requestAnimationFrame(update);
        }

        setInterval(() => { if (--timeLeft <= 0) alert("終了！最終スコア: " + score); document.getElementById('time').innerText = timeLeft; }, 1000);

        generateProblem();
        update();
    </script>
</body>
</html>
