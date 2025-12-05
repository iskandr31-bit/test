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
            touch-action: pan-y; 
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
        #pencil-button, #cancel-button, #shop-button, #fullscreen-button { 
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
        #fullscreen-button { background-color: #9900cc; color: white; }


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
        <button id="level-3-button" onclick="startGame('volcano')">Уровень 3: Вулкан (Сложность 3x)</button>
    </div>
    
    <div id="victory-screen">
        <h2>ПОБЕДА!</h2>
        <p id="victory-message">УРОВЕНЬ РАЗБЛОКИРОВАН!</p>
        <button onclick="goToMenu()">ПЕРЕЙТИ В МЕНЮ</button>
    </div>

    <div id="pencil-ui">
        <button id="shop-button" onclick="enterShop()">МАГАЗИН</button>
        <button id="pencil-button" onclick="enterDrawingMode()">КАРАНДАШ (Клик)</button>
        <button id="cancel-button" onclick="exitDrawingMode()">ОТМЕНА (X)</button>
        <button id="fullscreen-button" onclick="toggleFullScreen()">ПОЛНЫЙ ЭКРАН</button>
    </div>
    
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
            mapState: 'menu', // Сохраняет состояние карты, когда игрок в магазине
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
            level3Unlocked: false, 
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
        let gameOverScreen, finalScoreDisplay, menuScreen, victoryScreen, victoryMessage;
        let level1Button, level2Button, level3Button; 

        
        // --- ПЕРЕМЕННЫЕ ДЖОЙСТИКА ---
        let joystickBase, joystickKnob;
        let isDragging = false;
        let joystickAngle = 0; 
        let joystickDist = 0; 
        const JOYSTICK_RADIUS = 75; 
        let baseRect; 
        let joystickTouchId = null; 
        let enemySpawnInterval;

        // === ВСПОМОГАТЕЛЬНЫЕ ФУНКЦИИ ===
        
        function distance(x1, y1, x2, y2) {
            return Math.sqrt(Math.pow(x2 - x1, 2) + Math.pow(y2 - y1, 2));
        }
        
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
                localStorage.setItem('level3Unlocked', game.level3Unlocked); 
            } catch (e) {
                 console.error("Не удалось сохранить прогресс: ", e);
            }
        }

        function goToMenu() {
            game.state = 'menu';
            game.mapState = 'menu';
            if (enemySpawnInterval) clearInterval(enemySpawnInterval);
            victoryScreen.style.display = 'none';
            menuScreen.style.display = 'flex';
            updateMenuButtons();
        }

        function updateMenuButtons() {
            if (game.level2Unlocked) {
                 level2Button.disabled = false;
                 level2Button.textContent = 'Уровень 2: Пещера';
                 level2Button.style.backgroundColor = '#00ff00';
            } else {
                 level2Button.disabled = true;
                 level2Button.textContent = 'Уровень 2: Пещера (заблокирован)';
                 level2Button.style.backgroundColor = '#444';
            }
            if (game.level3Unlocked) {
                 level3Button.disabled = false;
                 level3Button.textContent = 'Уровень 3: Вулкан (Сложность 3x)';
                 level3Button.style.backgroundColor = '#cc6600';
            } else {
                 level3Button.disabled = true;
                 level3Button.textContent = 'Уровень 3: Вулкан (заблокирован)';
                 level3Button.style.backgroundColor = '#444';
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
            
            // Определяем текущий уровень (на основе значения в game.upgrades)
            const currentLevel = Math.round((game.upgrades[id] - 1) / upgrade.effect) + 1;
            
            if (currentLevel > upgrade.max) {
                 alert('Достигнут максимальный уровень улучшения!');
                 return;
            }

            // Рассчитываем динамическую стоимость
            let cost = upgrade.cost;
            for (let i = 1; i < currentLevel; i++) {
                 cost = Math.round(cost * 1.5);
            }

            if (game.coins >= cost) {
                game.coins -= cost;
                game.upgrades[id] += upgrade.effect;
                coinsDisplay.textContent = game.coins;
                updateShopList();
            } else {
                alert('Недостаточно монет! Требуется: ' + cost);
            }
        }

        function updateShopList() {
            shopCoinsDisplay.textContent = game.coins;
            upgradeList.innerHTML = '';
            UPGRADES.forEach(u => {
                const currentLevel = Math.round((game.upgrades[u.id] - 1) / u.effect) + 1;
                const nextLevel = currentLevel + 1;
                const maxReached = nextLevel > u.max;
                
                // Расчет стоимости следующего уровня
                let cost = u.cost;
                for (let i = 1; i < nextLevel; i++) {
                    cost = Math.round(cost * 1.5);
                }

                const div = document.createElement('div');
                div.className = 'upgrade-item';
                div.innerHTML = `
                    <span>${u.name} (Ур.${currentLevel}/${u.max})</span>
                    <span>Цена: ${maxReached ? '---' : cost} монет</span>
                    <button ${maxReached ? 'disabled' : ''} onclick="buyUpgrade('${u.id}')">
                        ${maxReached ? 'МАКС' : 'КУПИТЬ'}
                    </button>
                `;
                upgradeList.appendChild(div);
            });
        }

        function enterShop() {
            if (game.state !== 'menu' && game.state !== 'gameOver' && game.state !== 'victory') {
                game.state = 'shop';
                shopScreen.style.display = 'flex';
                updateShopList();
                pencilButton.style.display = 'none';
                cancelButton.style.display = 'none';
                shopButton.style.display = 'none';
                joystickBase.style.display = 'none';
            }
        }
        
        function exitShop() {
             game.state = game.mapState;
             shopScreen.style.display = 'none';
             if (game.hasPencil) pencilButton.style.display = 'block';
             shopButton.style.display = 'block';
             joystickBase.style.display = 'flex';
        }

        function spawnBoss() {
            if (boss) return; 

            const isLevel2 = (game.mapState === 'cave');
            const isLevel3 = (game.mapState === 'volcano');
            
            let maxHP = 500;
            if (isLevel2) maxHP = 1000;
            if (isLevel3) maxHP = 2000; 

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
                speed: isLevel3 ? 3 : 1.5, 
                nextShotTime: Date.now() + 2000,
                isLevel2: isLevel2,
                isLevel3: isLevel3
            };
        }
        
        function shootRocket(x, y, angle) {
            rockets.push({
                x, 
                y, 
                size: 20,
                vx: 5 * Math.cos(angle),
                vy: 5 * Math.sin(angle),
                isReversed: false 
            });
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
            if (game.state !== 'world' && game.state !== 'cave' && game.state !== 'volcano') return;

            let dx = 0;
            let dy = 0;
            
            const currentSpeed = PLAYER_SPEED * game.upgrades.speed; 
            let effectiveSpeed = currentSpeed; 

            // Управление Клавиатурой (ПК)
            if (keys['w'] || keys['W'] || keys['ArrowUp']) dy -= 1;
            if (keys['s'] || keys['S'] || keys['ArrowDown']) dx += 1;
            if (keys['a'] || keys['A'] || keys['ArrowLeft']) dx -= 1;
            if (keys['d'] || keys['D'] || keys['ArrowRight']) dy += 1;

            // Управление Джойстиком (Мобильные)
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
            
            // Блокировка выхода из пещеры/вулкана (если нужно)
            if (game.state === 'cave' || game.state === 'volcano') {
                const mapBoundary = 1000; 
                
                game.playerX = Math.max(-mapBoundary, Math.min(game.playerX, mapBoundary));
                game.playerY = Math.max(-mapBoundary, Math.min(game.playerY, mapBoundary));
            }


            cameraX = game.playerX - PLAYER_DRAW_X;
            cameraY = game.playerY - PLAYER_DRAW_Y;
        }

        function startGame(mapType) {
            
            if (mapType === 'cave' && !game.level2Unlocked) {
                 alert("Уровень 2 заблокирован! Победите Босса на Уровне 1.");
                 return;
            }
            if (mapType === 'volcano' && !game.level3Unlocked) {
                 alert("Уровень 3 заблокирован! Победите Босса на Уровне 2.");
                 return;
            }

            try {
                game.state = mapType;
                game.mapState = mapType;
                game.playerX = 0;
                game.playerY = 0;
                game.health = 100;
                game.kills = 0;
                
                game.invulnerabilityTimer = Date.now() + 1000; 

                enemies = []; 
                objects = [];
                bullets = [];
                coins = [];
                boss = null;
                rockets = [];
                
                // ГАРАНТИРОВАННО СКРЫВАЕМ МЕНЮ
                menuScreen.style.display = 'none'; 
                
                shopButton.style.display = 'block';
                joystickBase.style.display = 'flex';
                if (game.hasPencil) pencilButton.style.display = 'block';
                if (game.isDrawingMode) exitDrawingMode(); // Выключаем режим рисования при старте

                if (enemySpawnInterval) clearInterval(enemySpawnInterval);

                if (mapType === 'world') {
                     initGameObjects(); 
                     enemySpawnInterval = setInterval(() => {
                        if (game.state === 'world') createEnemy();
                     }, 1000); 
                } else if (mapType === 'cave') {
                     initCaveObjects();
                     enemySpawnInterval = setInterval(() => {
                        if (game.state === 'cave') createEnemy(undefined, undefined, undefined, 1.5); 
                     }, 650); 
                } else if (mapType === 'volcano') {
                     initVolcanoObjects(); 
                     enemySpawnInterval = setInterval(() => {
                        if (game.state === 'volcano') createEnemy(undefined, undefined, undefined, 3.0); 
                     }, 300); 
                }
            } catch (error) {
                 alert("Произошла ошибка при запуске игры: " + error.message);
                 game.state = 'menu';
            }
        }
        
        function initCaveObjects() {
            game.playerX = 0; 
            game.playerY = 0; 
            
            // Изначальное появление пещерных врагов
            for(let i=0; i<30; i++) { 
                createEnemy((Math.random() - 0.5) * 500, (Math.random() - 0.5) * 500, 'spider', 1.5);
            }
        }
        
        function initVolcanoObjects() {
             game.playerX = 0; 
             game.playerY = 0; 
             
             // Бассейны лавы
             for(let i=0; i<10; i++) {
                spawnObject((Math.random() - 0.5) * 1000, (Math.random() - 0.5) * 1000, 'lava_pool').size = 50; 
             }
             
             // Изначальное появление вулканических врагов
             for(let i=0; i<50; i++) { 
                createEnemy((Math.random() - 0.5) * 500, (Math.random() - 0.5) * 500, 'cockroach', 3.0);
             }
        }

        function enterDrawingMode() {
            if (game.hasPencil && game.state !== 'shop' && game.state !== 'menu') {
                game.isDrawingMode = true;
                pencilButton.style.display = 'none';
                cancelButton.style.display = 'block';
                CANVAS.classList.add('drawing-cursor');
                joystickBase.style.display = 'none';
            }
        }
        
        function exitDrawingMode() {
            game.isDrawingMode = false;
            if (game.hasPencil && game.state !== 'shop') {
                pencilButton.style.display = 'block';
            }
            cancelButton.style.display = 'none';
            CANVAS.classList.remove('drawing-cursor');
            joystickBase.style.display = 'flex';
        }

        // --- ОБРАБОТКА ВВОДА КЛАВИАТУРЫ ---
        document.addEventListener('keydown', (e) => {
            keys[e.key] = true;
            if ((game.state !== 'menu' && game.state !== 'shop') && e.key.toUpperCase() === 'E') {
                toggleShield();
            }
            if ((game.state !== 'menu' && game.state !== 'shop') && e.key.toUpperCase() === 'X' && game.isDrawingMode) {
                exitDrawingMode(); 
            }
        });
        document.addEventListener('keyup', (e) => {
            keys[e.key] = false;
        });

        // === 2. СНАРЯДЫ И ЛОГИКА МУЛЬТИТАЧА ===
        
        CANVAS.addEventListener('touchstart', (e) => {
            if (game.state !== 'world' && game.state !== 'cave' && game.state !== 'volcano') return;
            
            const rect = CANVAS.getBoundingClientRect();
            
            for (let i = 0; i < e.changedTouches.length; i++) {
                const touch = e.changedTouches[i];
                
                if (touch.identifier === joystickTouchId) {
                    continue; 
                }
                
                // Проверяем, что касание не находится в области джойстика
                const centerX = baseRect.left + JOYSTICK_RADIUS;
                const centerY = baseRect.top + JOYSTICK_RADIUS;
                const distFromBase = distance(touch.clientX, touch.clientY, centerX, centerY); 
                
                if (distFromBase > 100) { 
                    
                    const touchX = touch.clientX - rect.left;
                    const touchY = touch.clientY - rect.top;
                    
                    if (!game.isDrawingMode) {
                        const angle = Math.atan2(touchY - PLAYER_DRAW_Y, touchX - PLAYER_DRAW_X);
                        
                        bullets.push({
                            x: game.playerX,
                            y: game.playerY,
                            size: 5 * game.upgrades.bulletDamage,
                            vx: 10 * Math.cos(angle) * game.upgrades.bulletSpeed,
                            vy: 10 * Math.sin(angle) * game.upgrades.bulletSpeed
                        });
                    }
                }
            }
        });
        
        CANVAS.addEventListener('click', (e) => {
            if (game.state !== 'world' && game.state !== 'cave' && game.state !== 'volcano') return;
            if (e.target.tagName === 'BUTTON') return; 
            
            // Если клик произошел в области джойстика, игнорируем его
            const rect = CANVAS.getBoundingClientRect();
            const clickX = e.clientX;
            const clickY = e.clientY;
            
            if (joystickBase.style.display !== 'none') {
                const centerX = baseRect.left + JOYSTICK_RADIUS;
                const centerY = baseRect.top + JOYSTICK_RADIUS;
                if (distance(clickX, clickY, centerX, centerY) < JOYSTICK_RADIUS) {
                    return; 
                }
            }


            const mouseX = e.clientX - rect.left;
            const mouseY = e.clientY - rect.top;
            
            const worldClickX = mouseX + cameraX;
            const worldClickY = mouseY + cameraY;

            if (game.isDrawingMode) {
                let targetTree = null;
                const treeRadius = 60;
                
                for (let obj of objects) {
                    if ((obj.type === 'tree' || obj.type === 'green_tree') && distance(worldClickX, worldClickY, obj.x, obj.y) < treeRadius) {
                        targetTree = obj;
                        break;
                    }
                }
                
                if (targetTree) {
                    // "Покраска" дерева
                    targetTree.type = 'green_tree';
                    
                    game.wood += 2; 
                    spawnCoin(targetTree.x, targetTree.y); 
                    
                    // Убийство пауков
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
                 if (e.pointerType === 'mouse' || !e.pointerType) {
                    const angle = Math.atan2(mouseY - PLAYER_DRAW_Y, mouseX - PLAYER_DRAW_X);
                    
                    bullets.push({
                        x: game.playerX,
                        y: game.playerY,
                        size: 5 * game.upgrades.bulletDamage,
                        vx: 10 * Math.cos(angle) * game.upgrades.bulletSpeed,
                        vy: 10 * Math.sin(angle) * game.upgrades.bulletSpeed
                    });
                }
            }
        });
        
        function updateBullets() {
            for (let i = bullets.length - 1; i >= 0; i--) {
                const b = bullets[i];
                b.x += b.vx;
                b.y += b.vy;

                // Столкновение пули с ракетой
                for (let j = rockets.length - 1; j >= 0; j--) {
                    const r = rockets[j];
                    if (distance(r.x, r.y, b.x, b.y) < r.size/2 + b.size/2) {
                        bullets.splice(i, 1); 
                        r.vx *= -1; // Разворот ракеты
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

                const isPlayerInvulnerable = Date.now() < game.invulnerabilityTimer;

                // Столкновение с игроком (если ракета не развернута)
                if (!r.isReversed && !isPlayerInvulnerable && distance(game.playerX, game.playerY, r.x, r.y) < PLAYER_SIZE/2 + r.size/2) {
                    game.health -= 50; 
                    if (game.health <= 0) endGame();
                    rockets.splice(i, 1);
                    continue;
                }
                
                // Столкновение с боссом (если ракета развернута)
                if (r.isReversed && boss && distance(boss.x, boss.y, r.x, r.y) < boss.size/2 + r.size/2) {
                    boss.health -= 50; 
                    if (boss.health <= 0) {
                        const currentMap = game.mapState;
                        boss = null;
                        rockets = [];
                        
                        if (currentMap === 'world') {
                            game.level2Unlocked = true;
                            victoryMessage.textContent = 'УРОВЕНЬ 2 РАЗБЛОКИРОВАН!';
                        } else if (currentMap === 'cave') {
                            game.level3Unlocked = true; 
                            victoryMessage.textContent = 'УРОВЕНЬ 3 РАЗБЛОКИРОВАН!';
                        } else if (currentMap === 'volcano') {
                             victoryMessage.textContent = 'ПОЛНАЯ ПОБЕДА! ВЫ ПРОШЛИ ИГРУ!';
                        }

                        saveGameProgress(); 
                        game.state = 'victory';
                        victoryScreen.style.display = 'flex';
                        return; 
                    }
                    rockets.splice(i, 1);
                    continue;
                }

                 if (distance(game.playerX, game.playerY, r.x, r.y) > W + H + 500) {
                    rockets.splice(i, 1);
                }
            }
        }
        
        function spawnCoin(x, y) {
            coins.push({ x, y, size: 10, vy: -2, vx: (Math.random() - 0.5) * 1, isGravityApplied: true }); 
        }

        function checkCoinCollision() {
            for (let i = coins.length - 1; i >= 0; i--) {
                const c = coins[i];
                const distToPlayer = distance(game.playerX, game.playerY, c.x, c.y);

                if (distToPlayer < PLAYER_SIZE/2 + c.size/2 + 50) { 
                    // Притяжение монет
                    c.vx = (game.playerX - c.x) * 0.1;
                    c.vy = (game.playerY - c.y) * 0.1;
                    c.isGravityApplied = false;
                }
                
                if (distToPlayer < PLAYER_SIZE/2 + c.size/2 + 5) {
                    game.coins++;
                    coins.splice(i, 1);
                } else {
                    c.x += c.vx;
                    c.y += c.vy;
                    if (c.isGravityApplied) {
                        c.vy += 0.1; 
                    }
                }
            }
            coinsDisplay.textContent = game.coins;
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
                
                // Урон от лавы
                if ((obj.type === 'lava_pool') && distance(game.playerX, game.playerY, obj.x, obj.y) < PLAYER_SIZE/2 + obj.size/2) {
                    if (Date.now() % 60 === 0) { 
                        game.health -= 5;
                        if (game.health <= 0) endGame();
                    }
                }
                
                // Длительность паутины
                if (obj.type === 'web') {
                    obj.duration--;
                    if (obj.duration <= 0) {
                         objects.splice(i, 1);
                         continue;
                    }
                }
                
                // Удаление объектов, которые слишком далеко (только для "мира")
                if (game.mapState === 'world' && distance(game.playerX, game.playerY, obj.x, obj.y) > VIEW_RADIUS * 2) {
                    objects.splice(i, 1);
                }
            }
            
            // Появление новых деревьев в "мире"
            if (game.mapState === 'world') {
                if (objects.filter(o => o.type === 'tree' || o.type === 'green_tree').length < 15) {
                    const angle = Math.random() * Math.PI * 2;
                    const radius = W / 2 + Math.random() * 200; 
                    const x = game.playerX + radius * Math.cos(angle);
                    const y = game.playerY + radius * Math.sin(angle);
                    spawnObject(x, y, 'tree');
                }
            }
        }

        function createEnemy(x, y, type, multiplier = 1) {
            
            for (let i = 0; i < multiplier; i++) {
                if (x === undefined) {
                    // Появление врагов за пределами экрана
                    const angle = Math.random() * Math.PI * 2;
                    const radius = W / 2 + 100; 
                    x = game.playerX + radius * Math.cos(angle);
                    y = game.playerY + radius * Math.sin(angle);
                }
                
                const enemyType = type || (Math.random() < 0.3 ? 'spider' : 'cockroach');
                let speed = (enemyType === 'spider' ? 5 : 2) * multiplier; 
                let damage = (enemyType === 'spider' ? 30 : 5) * multiplier; 
                
                enemies.push({ x, y, size: 20, type: enemyType, speed, damage, state: 'idle' });
            }
        }

        function updateEnemies() {
            const isPlayerInvulnerable = Date.now() < game.invulnerabilityTimer;

            for (let i = enemies.length - 1; i >= 0; i--) {
                const e = enemies[i];
                const distToPlayer = distance(game.playerX, game.playerY, e.x, e.y);
                
                // Логика состояний
                if (boss) {
                    e.state = 'flee'; 
                } else if (distToPlayer < AGGRO_RADIUS) {
                    e.state = 'aggressive'; 
                } else {
                    e.state = 'idle';
                }

                // Движение врага
                if (e.state === 'aggressive') {
                    const angle = Math.atan2(game.playerY - e.y, game.playerX - e.x);
                    e.x += e.speed * Math.cos(angle);
                    e.y += e.speed * Math.sin(angle);
                } else if (e.state === 'flee') { 
                    const angle = Math.atan2(game.playerY - e.y, game.playerX - e.x);
                    e.x -= e.speed * Math.cos(angle); 
                    e.y -= e.speed * Math.sin(angle); 
                }

                // Столкновение с игроком
                if (distToPlayer < PLAYER_SIZE/2 + e.size/2) {
                    if (!boss) { 
                        if (!game.isShieldActive && !isPlayerInvulnerable) { 
                            game.health -= e.damage;
                            healthDisplay.textContent = Math.max(0, game.health);
                        }
                        
                        // Создание паутины при атаке паука
                        if (e.type === 'spider' && game.mapState === 'world' && Math.random() < 0.5) { 
                            spawnObject(e.x, e.y, 'web').size = 25; 
                        }

                        enemies.splice(i, 1);
                        if (game.health <= 0) { endGame(); return; }
                        continue; 
                    } 
                }
                
                // Столкновение с пулей
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
                
                // Удаление врагов, которые слишком далеко (только для "мира")
                if (game.mapState === 'world' && distToPlayer > VIEW_RADIUS * 2 + 500) {
                    enemies.splice(i, 1);
                }
            }
            
            // Появление босса
            if (!boss && game.kills >= BOSS_KILL_THRESHOLD && (game.state !== 'menu' && game.state !== 'shop')) {
                 spawnBoss();
            }
        }
        
        function updateBoss() {
            if (!boss) return;
            const now = Date.now();
            const distToPlayer = distance(game.playerX, game.playerY, boss.x, boss.y);

            // Движение босса
            const angle = Math.atan2(game.playerY - boss.y, game.playerX - boss.x);
            boss.x += boss.speed * Math.cos(angle);
            boss.y += boss.speed * Math.sin(angle);

            // Атака босса
            if (now >= boss.nextShotTime) {
                if (distToPlayer < VIEW_RADIUS + 200) { 
                    
                    if (boss.isLevel3) { // Тройная ракета для 3 уровня
                         shootRocket(boss.x, boss.y, angle);
                         shootRocket(boss.x, boss.y, angle + 0.2);
                         shootRocket(boss.x, boss.y, angle - 0.2);
                    }
                    else if (boss.isLevel2) { // Двойная ракета для 2 уровня
                        shootRocket(boss.x + 10, boss.y, angle + 0.1);
                        shootRocket(boss.x - 10, boss.y, angle - 0.1);
                    } else { // Одиночная ракета для 1 уровня
                        shootRocket(boss.x, boss.y, angle);
                    }
                }
                boss.nextShotTime = now + (boss.isLevel3 ? 1000 : 2000); 
            }
        }

        // === 3. ФУНКЦИИ ОТРИСОВКИ ===

        function drawBoss() {
            if (!boss) return;
            const screenX = boss.x - cameraX;
            const screenY = boss.y - cameraY;
            
            CTX.fillStyle = boss.isLevel3 ? '#ff0000' : (boss.isLevel2 ? '#8b008b' : 'darkred');
            CTX.beginPath();
            CTX.arc(screenX, screenY, boss.size, 0, Math.PI * 2);
            CTX.fill();
            
            // Полоса здоровья
            CTX.fillStyle = 'red';
            CTX.fillRect(screenX - boss.size, screenY - boss.size - 10, boss.size * 2, 5);
            CTX.fillStyle = 'lime';
            const healthWidth = (boss.health / boss.maxHealth) * boss.size * 2;
            CTX.fillRect(screenX - boss.size, screenY - boss.size - 10, healthWidth, 5);
        }

        function drawRockets() {
            for (let r of rockets) {
                const screenX = r.x - cameraX;
                const screenY = r.y - cameraY;
                CTX.fillStyle = r.isReversed ? 'yellow' : 'red';
                CTX.beginPath();
                CTX.arc(screenX, screenY, r.size/2, 0, Math.PI * 2);
                CTX.fill();
            }
        }
        
        // ОБНОВЛЕННАЯ ФУНКЦИЯ ОРИСОВКИ ВРАГОВ (ПИКСЕЛЬНЫЕ ТЕКСТУРЫ)
        function drawEnemyShape(e, screenX, screenY) {
            const halfSize = e.size / 2;
            
            // Таракан: простой коричневый квадрат
            if (e.type === 'cockroach') {
                CTX.fillStyle = '#8B4513'; // Коричневый
                CTX.fillRect(screenX - halfSize, screenY - halfSize, e.size, e.size);
                
            // Паук: темно-серый квадрат с белой точкой
            } else if (e.type === 'spider') {
                CTX.fillStyle = '#666'; // Темно-серый
                CTX.fillRect(screenX - halfSize, screenY - halfSize, e.size, e.size);
                
                // Глаз/метка для паука
                CTX.fillStyle = 'white';
                CTX.fillRect(screenX - 2, screenY - 2, 4, 4);
                
            } else {
                CTX.fillStyle = 'brown';
                CTX.fillRect(screenX - halfSize, screenY - halfSize, e.size, e.size);
            }
        }

        
        function drawPlayerEyes(centerX, centerY, targetX, targetY) {
            const angle = Math.atan2(targetY - centerY, targetX - centerX);
            const offset = 5; 
            const eyeX = centerX + offset * Math.cos(angle);
            const eyeY = centerY + offset * Math.sin(angle);

            CTX.fillStyle = 'black';
            CTX.beginPath();
            CTX.arc(eyeX - 4, eyeY, 4, 0, Math.PI * 2);
            CTX.arc(eyeX + 4, eyeY, 4, 0, Math.PI * 2);
            CTX.fill();
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
            } else if (obj.type === 'lava_pool') {
                CTX.fillStyle = '#ff4500'; 
                CTX.beginPath();
                CTX.arc(screenX, screenY, obj.size, 0, Math.PI * 2);
                CTX.fill();
            } else if (obj.type === 'web') { 
                CTX.fillStyle = 'rgba(150, 150, 150, 0.7)'; 
                CTX.fillRect(screenX - obj.size/2, screenY - obj.size/2, obj.size, obj.size);
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
            } else if (game.state === 'volcano') { 
                CTX.fillStyle = '#4d0000'; 
                CTX.fillRect(0, 0, W, H);
                CTX.fillStyle = '#800000'; 
                for(let i=0; i<W; i+=20) {
                     CTX.fillRect(i, 0, 10, H);
                }
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

            // Отрисовка пуль
            CTX.fillStyle = '#ffff00';
            for (let b of bullets) {
                const screenX = b.x - cameraX;
                const screenY = b.y - cameraY;
                CTX.fillRect(screenX - b.size/2, screenY - b.size/2, b.size, b.size);
            }
            
            drawRockets();

            // Отрисовка врагов
            for (let e of enemies) {
                const screenX = e.x - cameraX;
                const screenY = e.y - cameraY;
                drawEnemyShape(e, screenX, screenY);
            }
            
            drawBoss();

            // Отрисовка игрока
            const isInvulnerable = Date.now() < game.invulnerabilityTimer;

            CTX.fillStyle = isInvulnerable ? 'red' : '#00ffff'; 
            CTX.fillRect(PLAYER_DRAW_X - PLAYER_SIZE/2, PLAYER_DRAW_Y - PLAYER_SIZE/2, PLAYER_SIZE, PLAYER_SIZE);
            
            // Отрисовка глаз
            let nearestTarget = enemies.reduce((min, e) => {
                const dist = distance(game.playerX, game.playerY, e.x, e.y);
                return (dist < min.dist) ? { dist, x: e.x, y: e.y } : min;
            }, { dist: Infinity, x: PLAYER_DRAW_X + cameraX, y: PLAYER_DRAW_Y + cameraY - 100 });
            
            if (boss && distance(game.playerX, game.playerY, boss.x, boss.y) < nearestTarget.dist) {
                 nearestTarget.x = boss.x;
                 nearestTarget.y = boss.y;
            }

            drawPlayerEyes(PLAYER_DRAW_X, PLAYER_DRAW_Y, nearestTarget.x - cameraX, nearestTarget.y - cameraY);

            // Отрисовка щита
            if (game.isShieldActive || isInvulnerable) { 
                CTX.strokeStyle = isInvulnerable ? 'red' : '#00ffff';
                CTX.lineWidth = 3;
                CTX.beginPath();
                CTX.arc(PLAYER_DRAW_X, PLAYER_DRAW_Y, PLAYER_SIZE * 1.5, 0, Math.PI * 2);
                CTX.stroke();
            }
        }


        function initGameObjects() {
            objects = [];
            
            // Изначальное появление деревьев и врагов в "мире"
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
                
                // Появление карандаша
                if (i === pencilTreeIndex && !game.hasPencil) {
                    spawnObject(t.x + 30, t.y + 30, 'pencil');
                }
            }
        }
        
        function initDOM() {
            menuScreen = document.getElementById('menu-screen');
            victoryScreen = document.getElementById('victory-screen'); 
            victoryMessage = document.getElementById('victory-message');
            level1Button = document.getElementById('level-1-button');
            level2Button = document.getElementById('level-2-button');
            level3Button = document.getElementById('level-3-button'); 

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
            // Получаем размеры джойстика при инициализации
            baseRect = joystickBase.getBoundingClientRect(); 

            setupJoystick();
            
            // Скрытие элементов UI в меню
            document.getElementById('ui').style.display = 'flex'; 
            pencilButton.style.display = 'none';
            cancelButton.style.display = 'none';
            shopButton.style.display = 'none';
            joystickBase.style.display = 'none';
            
            // Загрузка прогресса
            const savedUnlock2 = localStorage.getItem('level2Unlocked');
            if (savedUnlock2 === 'true') {
                 game.level2Unlocked = true;
            }
            const savedUnlock3 = localStorage.getItem('level3Unlocked');
            if (savedUnlock3 === 'true') {
                 game.level3Unlocked = true;
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
            // --- PC MOUSE EVENTS ---
            joystickKnob.addEventListener('mousedown', (e) => {
                isDragging = true;
                e.preventDefault(); 
            });
            document.addEventListener('mousemove', (e) => {
                if (isDragging) handleMove(e.clientX, e.clientY);
            });
            document.addEventListener('mouseup', handleEnd);

            // --- MOBILE TOUCH EVENTS ---
            
            joystickKnob.addEventListener('touchstart', (e) => {
                if (joystickTouchId === null) {
                    isDragging = true;
                    joystickTouchId = e.changedTouches[0].identifier; 
                    
                    e.preventDefault(); 
                    handleMove(e.changedTouches[0].clientX, e.changedTouches[0].clientY);
                }
            }, { passive: false }); 
            
            document.addEventListener('touchmove', (e) => {
                if (isDragging) {
                    let dragTouch = Array.from(e.changedTouches).find(t => t.identifier === joystickTouchId);
                    
                    if (dragTouch) {
                        handleMove(dragTouch.clientX, dragTouch.clientY);
                    }
                }
            }, { passive: false }); 

            const handleTouchEnd = (e) => {
                let endedTouch = Array.from(e.changedTouches).find(t => t.identifier === joystickTouchId);
                if (endedTouch) {
                    joystickTouchId = null;
                    handleEnd(); 
                }
            };
            
            document.addEventListener('touchend', handleTouchEnd);
            document.addEventListener('touchcancel', handleTouchEnd);

            // Обновление размеров джойстика при изменении окна
            window.addEventListener('resize', () => {
                 baseRect = joystickBase.getBoundingClientRect();
            });
        }


        let gameLoopId;

        function gameLoop() {
            // !!! КРИТИЧЕСКАЯ ЗАЩИТА !!!
            try {
                if (game.state !== 'menu' && game.state !== 'shop' && game.state !== 'gameOver' && game.state !== 'victory') {
                    movePlayer();
                    updateBullets();
                    updateRockets(); 
                    updateEnemies();
                    updateBoss(); 
                    checkCoinCollision();
                    updateObjects();
                }
                
                draw();
                
                // Обновление UI
                scoreDisplay.textContent = game.score;
                woodDisplay.textContent = game.wood; 
                healthDisplay.textContent = Math.max(0, game.health);
                killsDisplay.textContent = game.kills; 
                
            } catch (error) {
                 // Если произошла ошибка во время цикла (обновление или отрисовка),
                 // мы сообщаем об этом пользователю и перезагружаем игру.
                 alert("Критическая ошибка: " + error.message + ". Пожалуйста, сообщите об этом. Игра будет перезагружена.");
                 window.location.reload(); 
                 return; 
            }

            gameLoopId = requestAnimationFrame(gameLoop);
        }
        
        document.addEventListener('DOMContentLoaded', () => {
             initDOM(); 
             gameLoop(); 
        });
    </script>

</body>
</html>
