# index.html87
小恐龍遊戲
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Dino Stable Edition</title>
    <style>
        body { margin: 0; background: #1a1a1a; display: flex; justify-content: center; align-items: center; height: 100vh; overflow: hidden; font-family: sans-serif; }
        #container { position: relative; }
        canvas { background: #87CEEB; display: block; border: 2px solid #555; image-rendering: pixelated; }
        #msg { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); color: #333; font-weight: bold; text-align: center; pointer-events: none; }
    </style>
</head>
<body>

<div id="container">
    <canvas id="game"></canvas>
    <div id="msg">載入中...</div>
</div>

<script>
/**
 * 1. 資源穩定載入器
 */
const config = {
    canvasW: 800,
    canvasH: 200,
    groundY: 160,
    assets: {
        dino: './assets/dino.png',
        cactus: './assets/cactus.png',
        bird: './assets/bird.png'
    }
};

const images = {};
let loadedCount = 0;
const assetKeys = Object.keys(config.assets);

// 核心：預載並確保穩定性
function loadAssets(callback) {
    assetKeys.forEach(key => {
        const img = new Image();
        img.src = config.assets[key];
        img.onload = () => {
            images[key] = img;
            checkAllLoaded(callback);
        };
        img.onerror = () => {
            console.warn(`資源 ${key} 讀取失敗，使用備援色塊`);
            images[key] = null; // 標記為空，後續繪圖會處理
            checkAllLoaded(callback);
        };
    });
}

function checkAllLoaded(callback) {
    loadedCount++;
    if (loadedCount === assetKeys.length) callback();
}

/**
 * 2. 遊戲主邏輯
 */
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const msg = document.getElementById('msg');
canvas.width = config.canvasW;
canvas.height = config.canvasH;

let score = 0, speed = 6, isOver = false;
let dino = { x: 50, y: config.groundY, w: 40, h: 40, dy: 0, jump: -12 };
let obs = [];

function init() {
    msg.innerText = "";
    loadAssets(() => requestAnimationFrame(update));
}

// 萬能輸入監聽
const handleAction = () => {
    if (isOver) return location.reload();
    if (dino.y >= config.groundY) dino.dy = dino.jump;
};
window.addEventListener('keydown', e => { if(e.code==='Space') handleAction(); });
canvas.addEventListener('touchstart', e => { e.preventDefault(); handleAction(); });

function update() {
    if (isOver) return;

    // 基礎物理與分數
    score++;
    speed += 0.001;
    dino.dy += 0.7; // 重力
    dino.y += dino.dy;
    if (dino.y > config.groundY) { dino.y = config.groundY; dino.dy = 0; }

    // 障礙物生成
    if (Math.random() < 0.02) {
        let isBird = Math.random() > 0.8;
        obs.push({ x: 800, y: isBird ? 80 : config.groundY + 10, w: 30, h: 30, type: isBird ? 'bird' : 'cactus' });
    }

    draw();
    requestAnimationFrame(update);
}

function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // 畫地面
    ctx.strokeStyle = "#555";
    ctx.beginPath(); ctx.moveTo(0, config.groundY + 40); ctx.lineTo(800, config.groundY + 40); ctx.stroke();

    // 畫恐龍 (穩定性檢查：有圖畫圖，沒圖畫色塊)
    if (images.dino) {
        ctx.drawImage(images.dino, dino.x, dino.y, dino.w, dino.h);
    } else {
        ctx.fillStyle = "#555";
        ctx.fillRect(dino.x, dino.y, dino.w, dino.h);
    }

    // 畫障礙物
    for (let i = obs.length - 1; i >= 0; i--) {
        let o = obs[i];
        o.x -= speed;

        const img = images[o.type];
        if (img) {
            ctx.drawImage(img, o.x, o.y, o.w, o.h);
        } else {
            ctx.fillStyle = o.type === 'bird' ? 'magenta' : 'green';
            ctx.fillRect(o.x, o.y, o.w, o.h);
        }

        // 碰撞偵測 (內縮 5 像素增加容錯感)
        if (o.x < dino.x + dino.w - 5 && o.x + o.w > dino.x + 5 && o.y < dino.y + dino.h - 5 && o.y + o.h > dino.y + 5) {
            gameOver();
        }
        if (o.x < -100) obs.splice(i, 1);
    }

    ctx.fillStyle = "#333";
    ctx.font = "20px monospace";
    ctx.fillText(Math.floor(score/10).toString().padStart(5, '0'), 10, 30);
}

function gameOver() {
    isOver = true;
    msg.innerHTML = "GAME OVER<br><small>點擊或空白鍵重啟</small>";
    msg.style.color = "red";
}

// 啟動
init();
</script>
</body>
</html>
