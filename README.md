<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Pixel Hunter: Boss Edition</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            background-color: #111;
            color: #fff;
            display: flex;
            flex-direction: column;
            align-items: center;
            margin: 0;
            padding: 20px;
            min-height: 100vh; 
        }

        #game-canvas {
            border: 3px solid #00ff00;
            cursor: crosshair;
            background-color: #333;
        }

        /* Общий UI */
        #ui {
            width: 800px;
            display: flex;
            justify-content: space-between;
            padding: 10px 0;
            font-size: 1.2em;
            margin-bottom: 5px;
        }
        .resource-display {
            padding: 5px 10px;
            background-color: #663300; 
            border-radius: 5px;
        }


        /* Кнопки Карандаша/Магазина */
        #pencil-ui {
            position: absolute;
            top: 20px;
            left: 20px;
            z-index: 50;
        }
        #pencil-button, #cancel-button, #shop-button, #fullscreen-button { /* ДОБАВЛЕНО: Кнопка Полный Экран */
            padding: 10px;
            border: none;
            cursor: pointer;
            font-weight: bold;
            font-size: 1em;
            width: 120px;
            text-align: center;
            margin-bottom: 5px;
        }
        #pencil-button { background-color: #008000; color: white; display: none; }
        #cancel-button { background-color: #cc0000; color: white; display: none; }
        #shop-button { background-color: #00bfff; color: black; }
        #fullscreen-button { background-color: #9900cc; color: white; } /* Стили для новой кнопки */


        /* Скрытые/Полноэкранные меню */
        #menu-screen, #shop-screen, #game-over-screen, #victory-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.9);
            color: #fff;
            display: none;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: 2em;
            text-align: center;
            z-index: 1000;
        }
        
        #victory-screen {
            background-color: rgba(0, 100, 0, 0.95);
        }

        #menu-screen button, #game-over-screen button, #victory-screen button {
            padding: 15px 30px;
            margin: 10px;
            font-size: 0.8em;
            cursor: pointer;
            background-color: #00ff00;
            border: none;
            border-radius: 5px;
        }
        
        #menu-screen button[disabled] {
             background-color: #444;
             color: #999;
             cursor: not-allowed;
        }

        .upgrade-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px 0;
            border-bottom: 1px solid #444;
            width: 90%;
            margin: 0 auto;
        }
        .upgrade-item button {
            padding: 5px 10px;
            background-color: #00ff00;
            color: black;
            border: none;
            cursor: pointer;
        }

        .drawing-cursor {
            cursor: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 32 32"><text x="16" y="16" font-size="20" fill="red" text-anchor="middle" dominant-baseline="middle">X</text></svg>') 16 16, crosshair !important;
        }

        /* === ДЖОЙСТИК СТИЛИ === */
        #joystick-base {
            position: fixed;
            bottom: 50px; 
            left: 50px;
            width: 150px;
            height: 150px;
            background-color: rgba(255, 255, 255, 0.2);
            border: 2px solid rgba(255, 255, 255, 0.3);
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 2000; 
            cursor: grab;
        }
        #joystick-knob {
            width: 60px;
            height: 60px;
            background-color: rgba(0, 255, 255, 0.7);
            border-radius: 50%;
            touch-action: none; 
            transform: translate(0, 0); 
        }
    </style>
</head>
<body>

    <div id="menu-screen">
        <h1>PIXEL HUNTER</h1>
        <p>Выберите уровень для старта:</p>
        <button id="level-1-button" onclick="startGame('world')">Уровень 1: Лес</button>
        <button id="level-2-button" onclick="startGame('cave')">Уровень 2: Пещера</button>
    </div>
    
    <div id="victory-screen">
        <h2>ПОБЕДА!</h2>
        <p>2 УРОВЕНЬ РАЗРЕШЕН!</p>
        <button onclick="goToMenu()">ПЕРЕЙТИ В МЕНЮ</button>
    </div>

    <div id="pencil-ui">
        <button id="shop-button" onclick="enterShop()">МАГАЗИН</button>
        <button id="pencil-button" onclick="enterDrawingMode()">КАРАНДАШ (Клик)</button>
        <button id="cancel-button" onclick="exitDrawingMode()">ОТМЕНА (X)</button>
        <button id="fullscreen-button" onclick="toggleFullScreen()">ПОЛНЫЙ ЭКРАН</button> </div>
    
    <div id="ui">
        <span>Счёт: <span id="score-display">0</span></span>
        <span>Убийств: <span id="kills-display">0</span></span>
        <span>Монеты: <span id="coins-display">0</span></span>
        <span class="resource-display">Древесина: <span id="wood-display">0</span></span> 
        <span>Здоровье: <span id="health-display">100</span></span>
        <span>Щит: <span id="shield-display">OFF (E)</span></span>
    </div>

    <canvas id="game-canvas" width="800" height="500"></canvas>

    <div id="joystick-base">
        <div id="joystick-knob"></div>
    </div>
    <div id="shop-screen">
        <h3>МАГАЗИН УЛУЧШЕНИЙ</h3>
        <div id="upgrade-list">
            </div>
        <p>Ваши монеты: <span id="shop-coins-display">0</span></p>
        <button onclick="exitShop()" style="margin-top: 15px; background-color: #444; color: white;">ЗАКРЫТЬ</button>
    </div>

    <div id="game-over-screen">
        <p>ИГРА ОКОНЧЕНА</p>
        <p>Ваш финальный счёт: <span id="final-score">0</span></p>
        <button onclick="window.location.reload()">Начать Снова</button>
    </div>

    <script>
        // === КОНСТАНТЫ ИГРЫ ===
        const CANVAS = document.getElementById('game-canvas');
        const CTX = CANVAS.getContext('2d');
        const W = CANVAS.width;
        const H = CANVAS.height;
        const PLAYER_SIZE = 20;
        const PLAYER_SPEED = 4;
        const AGGRO_RADIUS = 200;
        const VIEW_RADIUS = 500;
        const MIN_INITIAL_SPAWN_DIST = 1000; 
        const MAX_INITIAL_SPAWN_DIST = 1500; 

        const BOSS_KILL_THRESHOLD = 50; 
        
        // === ПЕРЕМЕННЫЕ ИГРЫ ===
        let game = {
            state: 'menu', 
            score: 0,
            health: 100,
            coins: 0,
            wood: 0, 
            kills: 0, 
            isShieldActive: false,
            playerX: 0, 
            playerY: 0,
            hasPencil: false,
            isDrawingMode: false,
            level2Unlocked: false, 
            invulnerabilityTimer: 0, 
            upgrades: {
                speed: 1, 
                bulletSpeed: 1, 
                bulletDamage: 1, 
            }
        };
        
        const PLAYER_DRAW_X = W / 2;
        const PLAYER_DRAW_Y = H / 2;
        let cameraX = 0;
        let cameraY = 0;

        let enemies = [];
        let bullets = [];
        let coins = [];
        let objects = []; 
        let keys = {};
        
        let boss = null; 
        let rockets = []; 
        
        // --- DOM ЭЛЕМЕНТЫ ---
        let scoreDisplay, coinsDisplay, healthDisplay, shieldDisplay, woodDisplay, killsDisplay;
        let pencilButton, cancelButton, shopButton, shopScreen, upgradeList, shopCoinsDisplay;
        let gameOverScreen, finalScoreDisplay, menuScreen, victoryScreen;
        let level1Button, level2Button;

        
        // --- ПЕРЕМЕННЫЕ ДЖОЙСТИКА ---
        let joystickBase, joystickKnob;
        let isDragging = false;
        let joystickAngle = 0; 
        let joystickDist = 0; 
        const JOYSTICK_RADIUS = 75; 
        let baseRect; 

        // === ВСПОМОГАТЕЛЬНЫЕ ФУНКЦИИ ===
        
        function distance(x1, y1, x2, y2) {
            return Math.sqrt(Math.pow(x2 - x1, 2) + Math.pow(y2 - y1, 2));
        }
        
        // НОВАЯ ФУНКЦИЯ: Управление полным экраном
        function toggleFullScreen() {
             const doc = document.documentElement;
             if (document.fullscreenElement) {
                 document.exitFullscreen();
             } else {
                 doc.requestFullscreen().catch(err => {
                     console.error(`Ошибка при попытке перехода в полноэкранный режим: ${err.message}`);
                 });
             }
        }
        
        function saveGameProgress() {
            try {
                localStorage.setItem('level2Unlocked', game.level2Unlocked);
            } catch (e) {
                 console.error("Не удалось сохранить прогресс: ", e);
            }
        }

        function goToMenu() {
            game.state = 'menu';
            if (enemySpawnInterval) clearInterval(enemySpawnInterval);
            victoryScreen.style.display = 'none';
            menuScreen.style.display = 'flex';
            updateMenuButtons();
        }

        function updateMenuButtons() {
            if (game.level2Unlocked) {
                 level2Button.disabled = false;
                 level2Button.textContent = 'Уровень 2: Пещера (УСИЛЕН)';
            } else {
                 level2Button.disabled = true;
                 level2Button.textContent = 'Уровень 2: Пещера (заблокирован)';
            }
        }

        function endGame() {
            game.state = 'gameOver';
            if (enemySpawnInterval) clearInterval(enemySpawnInterval);
            finalScoreDisplay.textContent = game.score;
            gameOverScreen.style.display = 'flex';
        }

        const UPGRADES = [
            { id: 'speed', name: 'Скорость Игрока', cost: 10, effect: 0.2, max: 5 },
            { id: 'bulletSpeed', name: 'Скорость Пули', cost: 15, effect: 0.3, max: 4 },
            { id: 'bulletDamage', name: 'Урон Пули', cost: 20, effect: 0.5, max: 3 }
        ];

        function buyUpgrade(id) {
            const upgrade = UPGRADES.find(u => u.id === id);
            
            if (game.upgrades[id] >= upgrade.max) {
                 alert('Достигнут максимальный уровень улучшения!');
                 return;
            }

            if (game.coins >= upgrade.cost) {
                game.coins -= upgrade.cost;
                game.upgrades[id] += upgrade.effect;
                upgrade.cost = Math.round(upgrade.cost * 1.5);
                coinsDisplay.textContent = game.coins;
                updateShopList();
            } else {
                alert('Недостаточно монет!');
            }
        }

        function updateShopList() {
            shopCoinsDisplay.textContent = game.coins;
            upgradeList.innerHTML = '';
            UPGRADES.forEach(u => {
                const level = Math.round((game.upgrades[u.id] - 1) / u.effect) + 1;
                const maxReached = game.upgrades[u.id] >= u.max;

                const div = document.createElement('div');
                div.className = 'upgrade-item';
                div.innerHTML = `
                    <span>${u.name} (Ур.${level})</span>
                    <span>Цена: ${u.cost} монет</span>
                    <button ${maxReached ? 'disabled' : ''} onclick="buyUpgrade('${u.id}')">
                        ${maxReached ? 'МАКС' : 'КУПИТЬ'}
                    </button>
                `;
                upgradeList.appendChild(div);
            });
        }

        function enterShop() {
            if (game.state === 'world' || game.state === 'cave') {
                game.state = 'shop';
                shopScreen.style.display = 'flex';
                updateShopList();
                pencilButton.style.display = 'none';
                cancelButton.style.display = 'none';
                shopButton.style.display = 'none';
            }
        }
        
        function exitShop() {
             game.state = game.mapState;
             shopScreen.style.display = 'none';
             if (game.hasPencil) pencilButton.style.display = 'block';
             shopButton.style.display = 'block';
        }

        function spawnBoss() {
            if (boss) return; 

            const isLevel2 = (game.mapState === 'cave');
            const maxHP = isLevel2 ? 1000 : 500; 
            
            const angle = Math.random() * Math.PI * 2;
            const radius = W / 2 + 300; 
            const x = game.playerX + radius * Math.cos(angle);
            const y = game.playerY + radius * Math.sin(angle);

            boss = {
                x, 
                y, 
                size: 50, 
                health: maxHP, 
                maxHealth: maxHP,
                damage: 50,
                speed: 1.5,
                nextShotTime: Date.now() + 2000,
                isLevel2: isLevel2 
            };
        }


        // === 1. УПРАВЛЕНИЕ ИГРОКОМ И КАРТОЙ ===
        
        function toggleShield() {
            if (!game.isShieldActive && game.coins >= 5) {
                game.isShieldActive = true;
                game.coins -= 5;
                shieldDisplay.textContent = 'ON';
                shieldDisplay.style.color = '#00ffff';
                setTimeout(() => {
                    game.isShieldActive = false;
                    shieldDisplay.textContent = 'OFF (E)';
                    shieldDisplay.style.color = '#fff';
                }, 5000); 
            } else if (!game.isShieldActive) {
                 shieldDisplay.textContent = 'LOW COINS!';
                 setTimeout(() => { shieldDisplay.textContent = 'OFF (E)'; }, 1000);
            }
        }

        function movePlayer() {
            if (game.state !== 'world' && game.state !== 'cave') return;

            let dx = 0;
            let dy = 0;
            
            const currentSpeed = PLAYER_SPEED * game.upgrades.speed; 
            let effectiveSpeed = currentSpeed; 

            if (keys['w'] || keys['W'] || keys['ArrowUp']) dy -= 1;
            if (keys['s'] || keys['S'] || keys['ArrowDown']) dy += 1;
            if (keys['a'] || keys['A'] || keys['ArrowLeft']) dx -= 1;
            if (keys['d'] || keys['D'] || keys['ArrowRight']) dx += 1;

            if (joystickDist > 0.1) { 
                dx = joystickDist * Math.cos(joystickAngle);
                dy = joystickDist * Math.sin(joystickAngle);
            }
            
            if (dx === 0 && dy === 0) {
                 return;
            }

            for (let obj of objects) {
                if (obj.type === 'web' && distance(game.playerX, game.playerY, obj.x, obj.y) < PLAYER_SIZE/2 + obj.size/2) {
                    effectiveSpeed *= 0.4; 
                    break;
                }
            }

            const magnitude = distance(0, 0, dx, dy);
            const moveFactor = effectiveSpeed / magnitude;
            
            game.playerX += dx * moveFactor;
            game.playerY += dy * moveFactor;
            
            cameraX = game.playerX - PLAYER_DRAW_X;
            cameraY = game.playerY - PLAYER_DRAW_Y;
            
            // Телепорт из пещеры
            if (game.state === 'cave' && game.playerY > H) {
                 teleportToWorld();
            }
        }

        function startGame(mapType) {
            
            if (mapType === 'cave' && !game.level2Unlocked) {
                 alert("Уровень 2 заблокирован! Победите Босса на Уровне 1.");
                 return;
            }

            game.state = mapType;
            game.mapState = mapType;
            game.playerX = 0;
            game.playerY = 0;
            game.health = 100;
            game.kills = 0;
            
            // Установка таймера неуязвимости на 1 секунду
            game.invulnerabilityTimer = Date.now() + 1000; 

            enemies = []; 
            objects = [];
            bullets = [];
            coins = [];
            boss = null;
            rockets = [];
            
            shopButton.style.display = 'block';
            if (game.hasPencil) pencilButton.style.display = 'block';

            menuScreen.style.display = 'none';

            if (mapType === 'world') {
                 initGameObjects(); 
                 if (enemySpawnInterval) clearInterval(enemySpawnInterval);
                 enemySpawnInterval = setInterval(() => {
                    if (game.state === 'world') createEnemy();
                 }, 1000); 
            } else if (mapType === 'cave') {
                 initCaveObjects();
                 if (enemySpawnInterval) clearInterval(enemySpawnInterval);
                 enemySpawnInterval = setInterval(() => {
                    if (game.state === 'cave') createEnemy(undefined, undefined, undefined, 1.5); 
                 }, 650); 
            }
        }
        
        function initCaveObjects() {
            game.playerX = W / 2;
            game.playerY = H - 50; 
            
            // Враги спавнятся только в верхней 50% экрана
            for(let i=0; i<30; i++) { 
                createEnemy(Math.random() * W, Math.random() * H * 0.5, 'spider');
            }
        }


        function teleportToCave(passageX, passageY) {
            if (!game.level2Unlocked) {
                 alert("Уровень 2 заблокирован! Победите Босса на Уровне 1.");
                 return;
            }
            
            game.state = 'cave';
            game.mapState = 'cave';
            initCaveObjects();
        }
        
        function teleportToWorld() {
            game.state = 'world';
            game.mapState = 'world';
            game.playerX = 0;
            game.playerY = 0;
            enemies = []; 
            objects = [];
            
            initGameObjects(); 
        }

        function enterDrawingMode() {
            if (game.hasPencil && game.state !== 'shop' && game.state !== 'menu') {
                game.isDrawingMode = true;
                pencilButton.style.display = 'none';
                cancelButton.style.display = 'block';
                CANVAS.classList.add('drawing-cursor');
            }
        }
        
        function exitDrawingMode() {
            game.isDrawingMode = false;
            if (game.hasPencil && game.state !== 'shop') {
                pencilButton.style.display = 'block';
            }
            cancelButton.style.display = 'none';
            CANVAS.classList.remove('drawing-cursor');
        }

        // --- ОБРАБОТКА ВВОДА КЛАВИАТУРЫ ---
        document.addEventListener('keydown', (e) => {
            keys[e.key] = true;
            if ((game.state === 'world' || game.state === 'cave') && e.key.toUpperCase() === 'E') {
                toggleShield();
            }
            if ((game.state === 'world' || game.state === 'cave') && e.key.toUpperCase() === 'X' && game.isDrawingMode) {
                exitDrawingMode(); 
            }
        });
        document.addEventListener('keyup', (e) => {
            keys[e.key] = false;
        });

        // === 2. СНАРЯДЫ И ЛОГИКА КЛИКА ===

        CANVAS.addEventListener('click', (e) => {
            if (game.state !== 'world' && game.state !== 'cave') return;

            // ГЛАВНОЕ ИСПРАВЛЕНИЕ: Мы проверяем, что нажатие произошло не на элементы джойстика
            if (e.target.id === 'joystick-base' || e.target.id === 'joystick-knob') return; 
            
            // Если игрок нажал на кнопку "Полный экран" или другую кнопку в UI, мы выходим
            if (e.target.tagName === 'BUTTON') return; 

            const rect = CANVAS.getBoundingClientRect();
            const mouseX = e.clientX - rect.left;
            const mouseY = e.clientY - rect.top;
            
            const worldClickX = mouseX + cameraX;
            const worldClickY = mouseY + cameraY;


            if (game.isDrawingMode) {
                let targetTree = null;
                const treeRadius = 60;
                
                for (let obj of objects) {
                    if (obj.type === 'tree' && distance(worldClickX, worldClickY, obj.x, obj.y) < treeRadius) {
                        targetTree = obj;
                        break;
                    }
                }
                
                if (targetTree) {
                    targetTree.type = 'green_tree';
                    
                    game.wood += 2; 
                    spawnCoin(targetTree.x, targetTree.y); 
                    
                    const killRadius = 150;
                    for (let i = enemies.length - 1; i >= 0; i--) {
                        const e = enemies[i];
                        if (distance(targetTree.x, targetTree.y, e.x, e.y) < killRadius && e.type === 'spider') {
                            game.kills++; 
                            spawnCoin(e.x, e.y);
                            enemies.splice(i, 1);
                        }
                    }
                    exitDrawingMode();
                }

            } else {
                // РЕЖИМ СТРЕЛЬБЫ
                const angle = Math.atan2(mouseY - PLAYER_DRAW_Y, mouseX - PLAYER_DRAW_X);
                
                bullets.push({
                    x: game.playerX,
                    y: game.playerY,
                    size: 5 * game.upgrades.bulletDamage,
                    vx: 10 * Math.cos(angle) * game.upgrades.bulletSpeed,
                    vy: 10 * Math.sin(angle) * game.upgrades.bulletSpeed
                });
            }
        });

        function updateBullets() {
            for (let i = bullets.length - 1; i >= 0; i--) {
                const b = bullets[i];
                b.x += b.vx;
                b.y += b.vy;

                // Столкновение пули с ракетой Босса
                for (let j = rockets.length - 1; j >= 0; j--) {
                    const r = rockets[j];
                    if (distance(r.x, r.y, b.x, b.y) < r.size/2 + b.size/2) {
                        
                        bullets.splice(i, 1); 

                        // РАЗВОРОТ РАКЕТЫ
                        r.vx *= -1;
                        r.vy *= -1;
                        r.isReversed = true; 
                        
                        i--; 
                        break; 
                    }
                }

                if (distance(game.playerX, game.playerY, b.x, b.y) > W + H + 500) {
                    bullets.splice(i, 1);
                }
            }
        }
        
        function updateRockets() {
            for (let i = rockets.length - 1; i >= 0; i--) {
                const r = rockets[i];
                r.x += r.vx;
                r.y += r.vy;

                // 1. Урон Игроку (если не развернута)
                // Проверка неуязвимости игрока!
                const isPlayerInvulnerable = Date.now() < game.invulnerabilityTimer;

                if (!r.isReversed && !isPlayerInvulnerable && distance(game.playerX, game.playerY, r.x, r.y) < PLAYER_SIZE/2 + r.size/2) {
                    game.health -= 50; 
                    if (game.health <= 0) endGame();
                    rockets.splice(i, 1);
                    continue;
                }
                
                // 2. Урон Боссу (если развернута)
                if (r.isReversed && boss && distance(boss.x, boss.y, r.x, r.y) < boss.size/2 + r.size/2) {
                    boss.health -= 50; 
                    if (boss.health <= 0) {
                        // ПОБЕДА НАД БОССОМ
                        boss = null;
                        rockets = [];
                        
                        if (game.mapState === 'world') {
                            game.level2Unlocked = true;
                            saveGameProgress(); 
                            // Показываем экран победы
                            game.state = 'victory';
                            victoryScreen.style.display = 'flex';
                            return; 
                        } else {
                            // Если победа на 2 уровне
                             alert("ПОБЕДА! ВЫ ПРОШЛИ 2 УРОВЕНЬ!");
                             goToMenu();
                             return;
                        }
                    }
                    rockets.splice(i, 1);
                    continue;
                }

                // 3. Удаление, если улетела слишком далеко
                 if (distance(game.playerX, game.playerY, r.x, r.y) > W + H + 500) {
                    rockets.splice(i, 1);
                }
            }
        }


        // === 3. МОНЕТЫ И ПРЕДМЕТЫ СРЕДЫ ===

        function spawnCoin(x, y) {
            coins.push({ x, y, size: 10, color: 'gold' });
        }
        
        function checkCoinCollision() {
            for (let i = coins.length - 1; i >= 0; i--) {
                const c = coins[i];
                if (distance(game.playerX, game.playerY, c.x, c.y) < PLAYER_SIZE/2 + c.size/2) {
                    game.coins += 1;
                    coinsDisplay.textContent = game.coins;
                    coins.splice(i, 1);
                }
            }
        }
        
        function spawnObject(x, y, type) {
            const newObj = { x, y, size: 30, type, duration: type === 'web' ? 500 : 0 }; 
            objects.push(newObj);
            return newObj;
        }

        function updateObjects() {
            for (let i = objects.length - 1; i >= 0; i--) {
                const obj = objects[i];
                
                if (obj.type === 'pencil' && distance(game.playerX, game.playerY, obj.x, obj.y) < PLAYER_SIZE/2 + obj.size/2) {
                    game.hasPencil = true;
                    pencilButton.style.display = 'block';
                    objects.splice(i, 1);
                    continue;
                }
                
                if (obj.type === 'black_passage' && distance(game.playerX, game.playerY, obj.x, obj.y) < PLAYER_SIZE/2 + obj.size/2) {
                    teleportToCave(obj.x, obj.y);
                    return;
                }
                
                if (obj.type === 'web') {
                    obj.duration--;
                    if (obj.duration <= 0) {
                         objects.splice(i, 1);
                         continue;
                    }
                }
                
                if (game.mapState === 'world' && distance(game.playerX, game.playerY, obj.x, obj.y) > VIEW_RADIUS * 2) {
                    objects.splice(i, 1);
                }
            }
            
            if (game.mapState === 'world') {
                if (objects.filter(o => o.type === 'tree' || o.type === 'green_tree').length < 15) {
                    const angle = Math.random() * Math.PI * 2;
                    const radius = W / 2 + Math.random() * 200; 
                    const x = game.playerX + radius * Math.cos(angle);
                    const y = game.playerY + radius * Math.sin(angle);
                    spawnObject(x, y, 'tree');
                }
                
                if (!objects.some(o => o.type === 'black_passage') && Math.random() < 0.015) { 
                    const angle = Math.random() * Math.PI * 2;
                    const radius = W; 
                    const x = game.playerX + radius * Math.cos(angle);
                    const y = game.playerY + radius * Math.sin(angle);
                    
                    spawnObject(x, y, 'black_passage').size = 40; 
                }
            }
        }


        // === 4. ВРАГИ И БОСС ===
        
        function createEnemy(x, y, type, multiplier = 1) {
            
            for (let i = 0; i < multiplier; i++) {
                if (x === undefined) {
                    const angle = Math.random() * Math.PI * 2;
                    const radius = W / 2 + 100; 
                    x = game.playerX + radius * Math.cos(angle);
                    y = game.playerY + radius * Math.sin(angle);
                }
                
                const enemyType = type || (Math.random() < 0.3 ? 'spider' : 'cockroach');
                let speed = enemyType === 'spider' ? 5 : 2;
                let damage = enemyType === 'spider' ? 30 : 5; 
                
                enemies.push({ x, y, size: 20, type: enemyType, speed, damage, state: 'idle' });
            }
        }

        let enemySpawnInterval; 

        function updateEnemies() {
            const isPlayerInvulnerable = Date.now() < game.invulnerabilityTimer;

            for (let i = enemies.length - 1; i >= 0; i--) {
                const e = enemies[i];
                const distToPlayer = distance(game.playerX, game.playerY, e.x, e.y);
                
                // 1. Определение состояния (Агрессия или Бегство)
                if (boss) {
                    e.state = 'flee'; 
                } else if (distToPlayer < AGGRO_RADIUS) {
                    e.state = 'aggressive'; 
                } else {
                    e.state = 'idle';
                }

                // 2. Движение
                if (e.state === 'aggressive') {
                    const angle = Math.atan2(game.playerY - e.y, game.playerX - e.x);
                    e.x += e.speed * Math.cos(angle);
                    e.y += e.speed * Math.sin(angle);
                } else if (e.state === 'flee') { 
                    const angle = Math.atan2(game.playerY - e.y, game.playerX - e.x);
                    e.x -= e.speed * Math.cos(angle); 
                    e.y -= e.speed * Math.sin(angle); 
                }

                // 3. Столкновение с игроком (Только если нет босса!)
                if (distToPlayer < PLAYER_SIZE/2 + e.size/2) {
                    if (!boss) { 
                        // Проверка неуязвимости
                        if (!game.isShieldActive && !isPlayerInvulnerable) { 
                            game.health -= e.damage;
                            healthDisplay.textContent = Math.max(0, game.health);
                        }
                        
                        if (e.type === 'spider' && game.mapState === 'world' && Math.random() < 0.5) { 
                            spawnObject(e.x, e.y, 'web').size = 25; 
                        }

                        enemies.splice(i, 1);
                        if (game.health <= 0) { endGame(); return; }
                        continue; 
                    } 
                }
                
                // 4. Столкновение с пулей
                for (let j = bullets.length - 1; j >= 0; j--) {
                    const b = bullets[j];
                    if (distance(e.x, e.y, b.x, b.y) < e.size/2) {
                        game.score += 10;
                        game.kills++; 
                        spawnCoin(e.x, e.y); 
                        enemies.splice(i, 1);
                        bullets.splice(j, 1);
                        break;
                    }
                }
                
                if (game.mapState === 'world' && distToPlayer > VIEW_RADIUS * 2 + 500) {
                    enemies.splice(i, 1);
                }
            }
            
            // ПРОВЕРКА СПАВНА БОССА
            if (!boss && game.kills >= BOSS_KILL_THRESHOLD && (game.state === 'world' || game.state === 'cave')) {
                 spawnBoss();
            }
        }
        
        function updateBoss() {
            if (!boss) return;
            const now = Date.now();
            const distToPlayer = distance(game.playerX, game.playerY, boss.x, boss.y);

            // Движение 
            const angle = Math.atan2(game.playerY - boss.y, game.playerX - boss.x);
            boss.x += boss.speed * Math.cos(angle);
            boss.y += boss.speed * Math.sin(angle);

            // Атака ракетами
            if (now >= boss.nextShotTime) {
                if (distToPlayer < VIEW_RADIUS + 200) { 
                    
                    // УРОВЕНЬ 2: ДВОЙНОЙ ВЫСТРЕЛ
                    if (boss.isLevel2) { 
                        // Выстрел немного смещенный влево/вправо (две "пушки")
                        shootRocket(boss.x + 10, boss.y, angle + 0.1);
                        shootRocket(boss.x - 10, boss.y, angle - 0.1);
                    } else {
                        shootRocket(boss.x, boss.y, angle);
                    }
                }
                boss.nextShotTime = now + 2000; 
            }
        }
        
        function shootRocket(startX, startY, angle) {
            
            rockets.push({
                x: startX,
                y: startY,
                size: 15,
                vx: 4 * Math.cos(angle), 
                vy: 4 * Math.sin(angle),
                isReversed: false,
                damage: 50 
            });
        }


        // === 5. ОТРИСОВКА (DRAW) ===

        function drawPlayerEyes(px, py, targetX, targetY) {
            const eyeSize = 5;
            const pupilSize = 2;
            const pupilRange = 1.5;

            const angle = Math.atan2(targetY - py, targetX - px);
            
            for (let i = -1; i <= 1; i += 2) {
                let eyeX = px + 4;
                let eyeY = py + 4 * i;
                let pupilX = eyeX + pupilRange * Math.cos(angle);
                let pupilY = eyeY + pupilRange * Math.sin(angle);
                
                CTX.fillStyle = 'white';
                CTX.fillRect(eyeX - eyeSize/2, eyeY - eyeSize/2, eyeSize, eyeSize);

                CTX.fillStyle = 'black';
                CTX.fillRect(pupilX - pupilSize/2, pupilY - pupilSize/2, pupilSize, pupilSize);
            }
        }

        function drawEnemyShape(e, screenX, screenY) {
            if (e.type === 'spider') {
                CTX.fillStyle = e.state === 'flee' ? '#4d004d' : '#800080'; 
                const s = 5; 
                CTX.fillRect(screenX - s, screenY - s, s * 2, s * 2);
                CTX.fillRect(screenX - 2*s, screenY - s/2, s, s);
                CTX.fillRect(screenX + s, screenY - s/2, s, s);
                CTX.fillRect(screenX - 2*s, screenY + s/2, s, s);
                CTX.fillRect(screenX + s, screenY + s/2, s, s);
            } 
            else if (e.type === 'cockroach') {
                CTX.fillStyle = e.state === 'flee' ? '#b30000' : '#ff0000';
                const s = 6; 
                CTX.fillRect(screenX - s/2, screenY - s, s, s);
                CTX.fillRect(screenX - s, screenY, s * 2, s * 2);
                CTX.fillRect(screenX - s/2, screenY + 2*s, s, s);
            }
        }
        
        function drawBoss() {
            if (!boss) return;
            const screenX = boss.x - cameraX;
            const screenY = boss.y - cameraY;
            const s = boss.size / 2;

            // Тело Босса (Огромный Черный Паук)
            CTX.fillStyle = 'black';
            CTX.beginPath();
            CTX.arc(screenX, screenY, s, 0, Math.PI * 2);
            CTX.fill();
            
            // Если это босс 2 уровня, добавим красные детали
            if (boss.isLevel2) {
                 CTX.fillStyle = '#cc0000';
                 CTX.fillRect(screenX - 5, screenY - 5, 10, 10);
            }

            // Полоса здоровья
            const hpBarWidth = 100;
            const hpBarHeight = 10;
            const hpRatio = boss.health / boss.maxHealth;
            
            CTX.fillStyle = 'red';
            CTX.fillRect(screenX - hpBarWidth/2, screenY - s - 20, hpBarWidth, hpBarHeight);
            
            CTX.fillStyle = 'lime';
            CTX.fillRect(screenX - hpBarWidth/2, screenY - s - 20, hpBarWidth * hpRatio, hpBarHeight);
            
            CTX.strokeStyle = '#fff';
            CTX.strokeRect(screenX - hpBarWidth/2, screenY - s - 20, hpBarWidth, hpBarHeight);
        }

        function drawObjectShape(obj, screenX, screenY) {
            if (obj.type === 'tree' || obj.type === 'green_tree') {
                CTX.fillStyle = obj.type === 'green_tree' ? 'green' : 'black';
                CTX.fillRect(screenX - 2, screenY - 30, 4, 60);
                CTX.fillRect(screenX - 10, screenY - 15, 20, 2);
            } else if (obj.type === 'pencil') {
                CTX.fillStyle = '#00ff00';
                CTX.fillRect(screenX - 10, screenY - 5, 20, 10);
                CTX.fillStyle = '#663300'; 
                CTX.fillRect(screenX + 10, screenY - 2.5, 5, 5);
            } else if (obj.type === 'black_passage') {
                CTX.fillStyle = 'black';
                CTX.fillRect(screenX - 20, screenY - 20, 40, 40);
                CTX.strokeStyle = '#fff';
                CTX.lineWidth = 2;
                CTX.strokeRect(screenX - 20, screenY - 20, 40, 40);
            } else if (obj.type === 'web') { 
                CTX.fillStyle = 'rgba(150, 150, 150, 0.7)'; 
                CTX.fillRect(screenX - obj.size/2, screenY - obj.size/2, obj.size, obj.size);
            }
        }
        
        function drawRockets() {
            for(let r of rockets) {
                const screenX = r.x - cameraX;
                const screenY = r.y - cameraY;
                
                CTX.fillStyle = r.isReversed ? 'yellow' : 'red'; 
                CTX.fillRect(screenX - r.size/2, screenY - r.size/2, r.size, r.size);
            }
        }

        function draw() {
            if (game.state === 'menu' || game.state === 'shop' || game.state === 'gameOver' || game.state === 'victory') {
                 CTX.clearRect(0, 0, W, H);
                 return;
            }

            // Отрисовка фона
            if (game.state === 'world') {
                 CTX.fillStyle = '#333'; 
                 CTX.fillRect(0, 0, W, H);
                 CTX.fillStyle = '#555'; 
                 const segmentSize = 30;
                 let roadOffset = (game.playerY * 0.5) % segmentSize;
                 for (let i = -1; i < H / segmentSize + 1; i++) {
                     const yPos = i * segmentSize + roadOffset;
                     CTX.fillRect(W / 2 - 2, yPos, 4, 20); 
                 }
            } else if (game.state === 'cave') {
                CTX.fillStyle = '#221100'; 
                CTX.fillRect(0, 0, W, H);
                
                CTX.fillStyle = '#333';
                CTX.fillRect(W / 2 - 50, H - 20, 100, 20); 
            }

            // Отрисовка объектов
            for(let obj of objects) {
                const screenX = obj.x - cameraX;
                const screenY = obj.y - cameraY;
                drawObjectShape(obj, screenX, screenY);
            }
            
            // Отрисовка монет
            CTX.fillStyle = 'gold';
            for(let c of coins) {
                const screenX = c.x - cameraX;
                const screenY = c.y - cameraY;
                CTX.fillRect(screenX - c.size/2, screenY - c.size/2, c.size, c.size);
            }

            // Отрисовка пуль игрока
            CTX.fillStyle = '#ffff00';
            for (let b of bullets) {
                const screenX = b.x - cameraX;
                const screenY = b.y - cameraY;
                CTX.fillRect(screenX - b.size/2, screenY - b.size/2, b.size, b.size);
            }
            
            // Отрисовка ракет босса
            drawRockets();

            // Отрисовка врагов
            for (let e of enemies) {
                const screenX = e.x - cameraX;
                const screenY = e.y - cameraY;
                drawEnemyShape(e, screenX, screenY);
            }
            
            // Отрисовка Босса
            drawBoss();

            // Отрисовка игрока
            const isInvulnerable = Date.now() < game.invulnerabilityTimer;

            CTX.fillStyle = isInvulnerable ? 'red' : '#00ffff'; 
            CTX.fillRect(PLAYER_DRAW_X - PLAYER_SIZE/2, PLAYER_DRAW_Y - PLAYER_SIZE/2, PLAYER_SIZE, PLAYER_SIZE);
            
            let nearestTarget = enemies.reduce((min, e) => {
                const dist = distance(game.playerX, game.playerY, e.x, e.y);
                return (dist < min.dist) ? { dist, x: e.x, y: e.y } : min;
            }, { dist: Infinity, x: PLAYER_DRAW_X, y: PLAYER_DRAW_Y - 100 });
            
            if (boss && distance(game.playerX, game.playerY, boss.x, boss.y) < nearestTarget.dist) {
                 nearestTarget.x = boss.x;
                 nearestTarget.y = boss.y;
            } else if (!boss && nearestTarget.dist === Infinity) {
                 nearestTarget.x = PLAYER_DRAW_X;
                 nearestTarget.y = PLAYER_DRAW_Y - 100;
            }

            drawPlayerEyes(PLAYER_DRAW_X, PLAYER_DRAW_Y, nearestTarget.x - cameraX, nearestTarget.y - cameraY);

            if (game.isShieldActive || isInvulnerable) { 
                CTX.strokeStyle = isInvulnerable ? 'red' : '#00ffff';
                CTX.lineWidth = 3;
                CTX.beginPath();
                CTX.arc(PLAYER_DRAW_X, PLAYER_DRAW_Y, PLAYER_SIZE * 1.5, 0, Math.PI * 2);
                CTX.stroke();
            }
        }

        // === 6. ИНИЦИАЛИЗАЦИЯ И ЗАПУСК ===

        function initGameObjects() {
            objects = [];
            
            let initialTrees = [];
            for(let i=0; i<15; i++) {
                const angle = Math.random() * Math.PI * 2;
                const radius = MIN_INITIAL_SPAWN_DIST + Math.random() * (MAX_INITIAL_SPAWN_DIST - MIN_INITIAL_SPAWN_DIST);
                
                initialTrees.push({
                    x: radius * Math.cos(angle),
                    y: radius * Math.sin(angle)
                });
            }
            
            const pencilTreeIndex = Math.floor(Math.random() * initialTrees.length);

            for (let i = 0; i < initialTrees.length; i++) {
                const t = initialTrees[i];
                const treeObj = spawnObject(t.x, t.y, 'tree');
                
                for (let j = 0; j < 3; j++) {
                    createEnemy(treeObj.x + (Math.random() - 0.5) * 50, treeObj.y + (Math.random() - 0.5) * 50, 'spider');
                }
                
                if (i === pencilTreeIndex) {
                    spawnObject(t.x + 30, t.y + 30, 'pencil');
                }
            }
        }
        
        // --- ИНИЦИАЛИЗАЦИЯ DOM ЭЛЕМЕНТОВ ---
        function initDOM() {
            menuScreen = document.getElementById('menu-screen');
            victoryScreen = document.getElementById('victory-screen'); 
            level1Button = document.getElementById('level-1-button');
            level2Button = document.getElementById('level-2-button'); 

            scoreDisplay = document.getElementById('score-display');
            coinsDisplay = document.getElementById('coins-display');
            healthDisplay = document.getElementById('health-display');
            shieldDisplay = document.getElementById('shield-display');
            woodDisplay = document.getElementById('wood-display'); 
            killsDisplay = document.getElementById('kills-display'); 
            
            pencilButton = document.getElementById('pencil-button');
            cancelButton = document.getElementById('cancel-button');
            shopButton = document.getElementById('shop-button');
            shopScreen = document.getElementById('shop-screen');
            upgradeList = document.getElementById('upgrade-list');
            shopCoinsDisplay = document.getElementById('shop-coins-display');
            gameOverScreen = document.getElementById('game-over-screen');
            finalScoreDisplay = document.getElementById('final-score');
            
            joystickBase = document.getElementById('joystick-base');
            joystickKnob = document.getElementById('joystick-knob');
            baseRect = joystickBase.getBoundingClientRect();

            setupJoystick();
            
            document.getElementById('ui').style.display = 'flex'; 
            pencilButton.style.display = 'none';
            cancelButton.style.display = 'none';
            shopButton.style.display = 'none';
            
            // Загрузка статуса разблокировки
            const savedUnlock = localStorage.getItem('level2Unlocked');
            if (savedUnlock === 'true') {
                 game.level2Unlocked = true;
            }

            updateMenuButtons(); 
            menuScreen.style.display = 'flex';
        }

        // --- ЛОГИКА ДЖОЙСТИКА ---
        function handleMove(clientX, clientY) {
            const centerX = baseRect.left + JOYSTICK_RADIUS;
            const centerY = baseRect.top + JOYSTICK_RADIUS;

            const dx = clientX - centerX;
            const dy = clientY - centerY;
            const dist = distance(0, 0, dx, dy);

            joystickAngle = Math.atan2(dy, dx); 

            if (dist > JOYSTICK_RADIUS) {
                const fixedX = JOYSTICK_RADIUS * Math.cos(joystickAngle);
                const fixedY = JOYSTICK_RADIUS * Math.sin(joystickAngle);
                joystickKnob.style.transform = `translate(${fixedX}px, ${fixedY}px)`;
                joystickDist = 1;
            } else {
                joystickKnob.style.transform = `translate(${dx}px, ${dy}px)`;
                joystickDist = dist / JOYSTICK_RADIUS; 
            }
        }

        function handleEnd() {
            isDragging = false;
            joystickDist = 0;
            joystickKnob.style.transform = `translate(0, 0)`; 
        }

        function setupJoystick() {
            joystickKnob.addEventListener('mousedown', (e) => {
                isDragging = true;
                e.preventDefault(); 
            });
            document.addEventListener('mousemove', (e) => {
                if (isDragging) handleMove(e.clientX, e.clientY);
            });
            document.addEventListener('mouseup', handleEnd);

            joystickKnob.addEventListener('touchstart', (e) => {
                isDragging = true;
                e.preventDefault(); 
                handleMove(e.touches[0].clientX, e.touches[0].clientY);
            });
            document.addEventListener('touchmove', (e) => {
                if (isDragging && e.touches.length > 0) handleMove(e.touches[0].clientX, e.touches[0].clientY);
            });
            document.addEventListener('touchend', handleEnd);
            document.addEventListener('touchcancel', handleEnd);

            window.addEventListener('resize', () => {
                 baseRect = joystickBase.getBoundingClientRect();
            });
        }


        let gameLoopId;

        function gameLoop() {
            
            if (game.state === 'world' || game.state === 'cave') {
                movePlayer();
                updateBullets();
                updateRockets(); 
                updateEnemies();
                updateBoss(); 
                checkCoinCollision();
                updateObjects();
            }
            
            draw();
            
            scoreDisplay.textContent = game.score;
            woodDisplay.textContent = game.wood; 
            healthDisplay.textContent = Math.max(0, game.health);
            killsDisplay.textContent = game.kills; 

            gameLoopId = requestAnimationFrame(gameLoop);
        }
        
        document.addEventListener('DOMContentLoaded', () => {
             initDOM(); 
             gameLoop(); 
        });
    </script>

</body>
</html>
