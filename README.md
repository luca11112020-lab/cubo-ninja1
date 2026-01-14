[index.html.html](https://github.com/user-attachments/files/24624554/index.html.html)
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>Cubo Ninja Ultra</title>
    <style>
        body { margin: 0; background: #000; display: flex; justify-content: center; align-items: center; height: 100vh; color: white; font-family: sans-serif; overflow: hidden; }
        canvas { background: #1a1a2e; border: 5px solid #555; cursor: crosshair; display: block; }
    </style>
</head>
<body>

<canvas id="gameCanvas" width="800" height="400"></canvas>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

// Configurações Globais
let state = "MENU"; 
let gold = 0;
let score = 0;
let player = { x: 100, y: 300, w: 40, h: 40, vy: 0, g: 0.8, onG: false };
let enemies = [];
let spikes = [];

let weapons = [
    { name: "Adaga", reach: 70, color: "#ffffff", price: 0, owned: true },
    { name: "Katana", reach: 130, color: "#00ffff", price: 50, owned: false },
    { name: "Sabre", reach: 250, color: "#ff00ff", price: 150, owned: false }
];
let currentWeapon = weapons[0];

// Mouse
let mouseX = 0, mouseY = 0;
canvas.addEventListener("mousemove", (e) => {
    const rect = canvas.getBoundingClientRect();
    mouseX = e.clientX - rect.left;
    mouseY = e.clientY - rect.top;
});

// Clique
canvas.addEventListener("mousedown", () => {
    if (state === "MENU") {
        if (mouseX > 300 && mouseX < 500 && mouseY > 180 && mouseY < 230) state = "PLAYING";
        if (mouseX > 300 && mouseX < 500 && mouseY > 250 && mouseY < 300) state = "SHOP";
    } else if (state === "SHOP") {
        if (mouseX > 20 && mouseX < 120 && mouseY > 20 && mouseY < 60) state = "MENU";
        weapons.forEach((w, i) => {
            let y = 130 + i * 70;
            if (mouseX > 550 && mouseX < 700 && mouseY > y + 10 && mouseY < y + 50) {
                if (w.owned) currentWeapon = w;
                else if (gold >= w.price) { gold -= w.price; w.owned = true; currentWeapon = w; }
            }
        });
    } else if (state === "DEAD") {
        state = "MENU";
        resetGame();
    }
});

// Pulo
window.addEventListener("keydown", (e) => {
    if ((e.code === "Space" || e.code === "ArrowUp") && player.onG && state === "PLAYING") {
        player.vy = -16;
        player.onG = false;
    }
});

function resetGame() {
    player.y = 300; player.vy = 0;
    enemies = []; spikes = []; score = 0;
}

function gameLoop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    if (state === "MENU") {
        drawTxt("CUBO NINJA", 400, 100, 60, "#fff");
        drawTxt("MOEDAS: " + gold, 400, 150, 25, "#ffd700");
        drawBtn(300, 180, 200, 50, "JOGAR", "#2ecc71");
        drawBtn(300, 250, 200, 50, "LOJA", "#3498db");
    } 
    
    else if (state === "SHOP") {
        drawTxt("LOJA DE ARMAS", 400, 70, 40, "#fff");
        drawBtn(20, 20, 100, 40, "VOLTAR", "#555");
        weapons.forEach((w, i) => {
            let y = 130 + i * 70;
            ctx.fillStyle = "#222";
            ctx.fillRect(100, y, 620, 60);
            drawTxt(w.name + " (Alcance: " + w.reach + ")", 120, y + 38, 20, "#fff", "left");
            let btnTxt = w.owned ? "EQUIPAR" : "CUSTA " + w.price;
            if (currentWeapon === w) btnTxt = "USANDO";
            drawBtn(550, y + 10, 150, 40, btnTxt, w.owned ? "#444" : "#f1c40f");
        });
    }

    else if (state === "PLAYING") {
        // Lógica do Player
        player.y += player.vy;
        player.vy += player.g;
        if (player.y >= 340) { player.y = 340; player.vy = 0; player.onG = true; }

        // Spawn de Inimigos (Raro)
        if (Math.random() < 0.01) enemies.push({ x: 850, y: Math.random() * 200 + 100, r: 20 });
        
        // Spawn de Espinhos (Raro)
        if (Math.random() < 0.008) spikes.push({ x: 850, y: 340 });

        // Chão
        ctx.fillStyle = "#555";
        ctx.fillRect(0, 380, 800, 20);

        // Desenhar Espinhos
        ctx.fillStyle = "#ff4444";
        for (let i = spikes.length - 1; i >= 0; i--) {
            let s = spikes[i];
            s.x -= 5;
            ctx.beginPath();
            ctx.moveTo(s.x, 380);
            ctx.lineTo(s.x + 20, 340);
            ctx.lineTo(s.x + 40, 380);
            ctx.fill();
            
            // Colisão Espinho
            if (player.x + 35 > s.x && player.x < s.x + 35 && player.y + 35 > 340) {
                die();
            }
            if (s.x < -50) spikes.splice(i, 1);
        }

        // Inimigos
        ctx.fillStyle = "#f00";
        for (let i = enemies.length - 1; i >= 0; i--) {
            let e = enemies[i];
            e.x -= 4;
            ctx.beginPath();
            ctx.arc(e.x, e.y, e.r, 0, Math.PI * 2);
            ctx.fill();

            // Matar Monstro com a Espada
            let angle = Math.atan2(mouseY - (player.y + 20), mouseX - (player.x + 20));
            let tipX = player.x + 20 + Math.cos(angle) * currentWeapon.reach;
            let tipY = player.y + 20 + Math.sin(angle) * currentWeapon.reach;
            
            let distToSwordTip = Math.hypot(tipX - e.x, tipY - e.y);
            
            if (distToSwordTip < 30) {
                enemies.splice(i, 1);
                score += 10;
            } else if (Math.hypot(player.x + 20 - e.x, player.y + 20 - e.y) < 35) {
                die();
            }
            if (e.x < -50) enemies.splice(i, 1);
        }

        // Desenhar Personagem (Bloco Azul)
        ctx.fillStyle = "#00aaff";
        ctx.fillRect(player.x, player.y, player.w, player.h);

        // Desenhar Espada
        let angle = Math.atan2(mouseY - (player.y + 20), mouseX - (player.x + 20));
        ctx.strokeStyle = currentWeapon.color;
        ctx.lineWidth = 6;
        ctx.beginPath();
        ctx.moveTo(player.x + 20, player.y + 20);
        ctx.lineTo(player.x + 20 + Math.cos(angle) * currentWeapon.reach, player.y + 20 + Math.sin(angle) * currentWeapon.reach);
        ctx.stroke();

        drawTxt("PONTOS: " + score, 20, 40, 25, "#fff", "left");
    }

    else if (state === "DEAD") {
        drawTxt("FIM DE JOGO", 400, 180, 60, "#ff4444");
        drawTxt("CLIQUE PARA VOLTAR", 400, 240, 20, "#fff");
    }

    requestAnimationFrame(gameLoop);
}

function die() { gold += score; state = "DEAD"; }

function drawTxt(t, x, y, s, c, a = "center") {
    ctx.fillStyle = c; ctx.font = "bold " + s + "px Arial"; ctx.textAlign = a;
    ctx.fillText(t, x, y);
}

function drawBtn(x, y, w, h, t, c) {
    ctx.fillStyle = c; ctx.fillRect(x, y, w, h);
    ctx.fillStyle = "#000"; ctx.font = "bold 20px Arial"; ctx.textAlign = "center";
    ctx.fillText(t, x + w / 2, y + h / 2 + 8);
}

gameLoop();
</script>
</body>
</html>
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Cubo Ninja Mobile</title>
    <style>
        body { margin: 0; background: #111; display: flex; justify-content: center; align-items: center; height: 100vh; color: white; font-family: sans-serif; overflow: hidden; touch-action: none; }
        canvas { background: #1a1a2e; border: 2px solid #555; max-width: 100%; max-height: 100%; }
    </style>
</head>
<body>

<canvas id="gameCanvas"></canvas>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

// Ajuste de tela para Celular
function resize() {
    canvas.width = window.innerWidth > 800 ? 800 : window.innerWidth;
    canvas.height = window.innerHeight > 400 ? 400 : window.innerHeight;
}
window.addEventListener('resize', resize);
resize();

let state = "MENU", gold = 0, score = 0;
let player = { x: 50, y: 200, w: 35, h: 35, vy: 0, g: 0.8, onG: false };
let enemies = [], spikes = [];
let weapons = [
    { name: "Adaga", reach: 70, color: "#fff", price: 0, owned: true },
    { name: "Katana", reach: 130, color: "#0ff", price: 50, owned: false },
    { name: "Sabre", reach: 220, color: "#f0f", price: 150, owned: false }
];
let currentWeapon = weapons[0];
let swordAngle = 0;

// CONTROLES DE TOQUE (CELULAR) E MOUSE (PC)
function handleInput(x, y) {
    if (state === "MENU") {
        if (x > canvas.width/2 - 100 && x < canvas.width/2 + 100) {
            if (y > 180 && y < 230) state = "PLAYING";
            if (y > 250 && y < 300) state = "SHOP";
        }
    } else if (state === "SHOP") {
        if (x < 100 && y < 60) state = "MENU";
        weapons.forEach((w, i) => {
            let btnY = 130 + i * 70;
            if (x > canvas.width - 160 && y > btnY && y < btnY + 40) {
                if (w.owned) currentWeapon = w;
                else if (gold >= w.price) { gold -= w.price; w.owned = true; currentWeapon = w; }
            }
        });
    } else if (state === "PLAYING") {
        if (x < canvas.width / 2) { // Lado Esquerdo pula
            if (player.onG) { player.vy = -15; player.onG = false; }
        } else { // Lado Direito ataca
            swordAngle = Math.atan2(y - (player.y + 17), x - (player.x + 17));
            checkAttack(x, y);
        }
    } else if (state === "DEAD") {
        state = "MENU"; resetGame();
    }
}

canvas.addEventListener("touchstart", (e) => {
    e.preventDefault();
    const touch = e.touches[0];
    const rect = canvas.getBoundingClientRect();
    handleInput(touch.clientX - rect.left, touch.clientY - rect.top);
}, { passive: false });

canvas.addEventListener("mousedown", (e) => {
    const rect = canvas.getBoundingClientRect();
    handleInput(e.clientX - rect.left, e.clientY - rect.top);
});

window.addEventListener("keydown", (e) => {
    if ((e.code === "Space" || e.code === "ArrowUp") && player.onG && state === "PLAYING") {
        player.vy = -15; player.onG = false;
    }
});

function resetGame() { player.y = 200; player.vy = 0; enemies = []; spikes = []; score = 0; }

function checkAttack(tx, ty) {
    for (let i = enemies.length - 1; i >= 0; i--) {
        let e = enemies[i];
        let dToPlayer = Math.hypot((player.x + 17) - e.x, (player.y + 17) - e.y);
        if (dToPlayer < currentWeapon.reach + 20) {
            enemies.splice(i, 1);
            score += 10;
        }
    }
}

function gameLoop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    if (state === "MENU") {
        drawTxt("CUBO NINJA", canvas.width/2, 100, 40, "#fff");
        drawBtn(canvas.width/2 - 100, 180, 200, 50, "JOGAR", "#2ecc71");
        drawBtn(canvas.width/2 - 100, 250, 200, 50, "LOJA", "#3498db");
    } 
    else if (state === "SHOP") {
        drawTxt("LOJA", canvas.width/2, 60, 30, "#fff");
        drawBtn(10, 10, 80, 40, "SAIR", "#555");
        weapons.forEach((w, i) => {
            let y = 120 + i * 70;
            ctx.fillStyle = "#222"; ctx.fillRect(10, y, canvas.width-20, 50);
            drawTxt(w.name, 20, y + 30, 18, "#fff", "left");
            let txt = w.owned ? "USAR" : w.price + "G";
            if (currentWeapon === w) txt = "OK";
            drawBtn(canvas.width - 110, y + 5, 100, 40, txt, w.owned ? "#444" : "#f1c40f");
        });
    }
    else if (state === "PLAYING") {
        player.y += player.vy; player.vy += player.g;
        if (player.y >= canvas.height - 60) { player.y = canvas.height - 60; player.vy = 0; player.onG = true; }

        if (Math.random() < 0.01) enemies.push({ x: canvas.width + 50, y: Math.random() * (canvas.height-100) + 50 });
        if (Math.random() < 0.008) spikes.push({ x: canvas.width + 50 });

        // Chão e Obstáculos
        ctx.fillStyle = "#444"; ctx.fillRect(0, canvas.height - 25, canvas.width, 25);
        
        ctx.fillStyle = "#f44";
        for(let i=spikes.length-1; i>=0; i--) {
            spikes[i].x -= 4;
            ctx.beginPath(); ctx.moveTo(spikes[i].x, canvas.height-25);
            ctx.lineTo(spikes[i].x+15, canvas.height-55); ctx.lineTo(spikes[i].x+30, canvas.height-25); ctx.fill();
            if (player.x+30 > spikes[i].x && player.x < spikes[i].x+30 && player.y+35 > canvas.height-55) die();
            if (spikes[i].x < -50) spikes.splice(i,1);
        }

        ctx.fillStyle = "#f00";
        for(let i=enemies.length-1; i>=0; i--) {
            enemies[i].x -= 3;
            ctx.beginPath(); ctx.arc(enemies[i].x, enemies[i].y, 15, 0, Math.PI*2); ctx.fill();
            if (Math.hypot(player.x+17 - enemies[i].x, player.y+17 - enemies[i].y) < 30) die();
            if (enemies[i].x < -50) enemies.splice(i,1);
        }

        // Desenha Player e Espada
        ctx.fillStyle = "#0af"; ctx.fillRect(player.x, player.y, player.w, player.h);
        ctx.strokeStyle = currentWeapon.color; ctx.lineWidth = 5;
        ctx.beginPath(); ctx.moveTo(player.x+17, player.y+17);
        ctx.lineTo(player.x+17 + Math.cos(swordAngle)*currentWeapon.reach, player.y+17 + Math.sin(swordAngle)*currentWeapon.reach);
        ctx.stroke();

        drawTxt("PTS: " + score, 20, 30, 20, "#fff", "left");
        drawTxt("Pule (Esq) | Ataque (Dir)", canvas.width/2, canvas.height-10, 12, "#aaa");
    }
    else if (state === "DEAD") {
        drawTxt("MORREU! PTS: " + score, canvas.width/2, 180, 30, "#f44");
        drawTxt("TOQUE PARA VOLTAR", canvas.width/2, 230, 15, "#fff");
    }
    requestAnimationFrame(gameLoop);
}

function die() { gold += score; state = "DEAD"; }
function drawTxt(t, x, y, s, c, a="center") { ctx.fillStyle = c; ctx.font = s + "px Arial"; ctx.textAlign = a; ctx.fillText(t, x, y); }
function drawBtn(x, y, w, h, t, c) { ctx.fillStyle = c; ctx.fillRect(x, y, w, h); ctx.fillStyle = "#000"; ctx.textAlign="center"; ctx.fillText(t, x+w/2, y+h/2+5); }

gameLoop();
</script>
</body>
</html>
