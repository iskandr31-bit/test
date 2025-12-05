<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Pixel Hunter: Expansion Pack</title>
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
        
        #menu-list {
            display: grid;
            grid-template-columns: repeat(4, 1fr); 
            gap: 10px;
            max-width: 900px; 
            margin: 20px auto;
        }
        
        .level-button {
             padding: 10px 15px;
             font-size: 0.7em;
             background-color: #008000;
             color: white;
             border: none;
             cursor: pointer;
             border-radius: 5px;
        }
        .level-button[disabled] {
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
        <h1>PIXEL HUNTER: EXPANSION PACK</h1>
        <p>Выберите биом для старта:</p>
        <div id="menu-list">
            </div>
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
        const BOSS_KILL_THRESHOLD = 50; 
        
        // --- СПИСОК ВСЕХ 20 БИОМОВ ---
        const BIOMES = [
            { id: 'world', name: '1. Лес (Пауки, Тараканы)', multiplier: 1.0, enemies: ['spider', 'cockroach'] },
            { id: 'cave', name: '2. Пещера (Усиленные Враги)', multiplier: 1.5, enemies: ['spider', 'cockroach'] },
            { id: 'volcano', name: '3. Вулкан (Лавовые Тараканы)', multiplier: 3.0, enemies: ['lava_roach'] },
            { id: 'swamp', name: '4. Болото (Ядовитые Муравьи)', multiplier: 2.0, enemies: ['poison_ant', 'cockroach'] },
            { id: 'winter', name: '5. Зима (Ледяные Пауки)', multiplier: 2.5, enemies: ['ice_spider', 'spider'] },
            { id: 'desert', name: '6. Пустыня (Быстрые Мухи)', multiplier: 3.5, enemies: ['fast_fly', 'cockroach'] },
            { id: 'forest_dark', name: '7. Темный Лес (Двойной Урон)', multiplier: 4.0, enemies: ['spider', 'cockroach'] },
            { id: 'canyon', name: '8. Каньон (Высокая Скорость)', multiplier: 4.5, enemies: ['fast_fly', 'poison_ant'] },
            { id: 'river', name: '9. Река (Мокрые Враги)', multiplier: 5.0, enemies: ['spider', 'cockroach'] },
            { id: 'mountain', name: '10. Горы (Лава + Лед)', multiplier: 6.0, enemies: ['lava_roach', 'ice_spider'] },
            { id: 'deep_cave', name: '11. Глубокая пещера (Пещера 2.0)', multiplier: 7.0, enemies: ['spider', 'cockroach'] },
            { id: 'ruins', name: '12. Руины (Мухи + Паутина)', multiplier: 8.0, enemies: ['fast_fly', 'spider'] },
            { id: 'city', name: '13. Город (Робо-Тараканы)', multiplier: 9.0, enemies: ['cockroach'] },
            { id: 'jungle', name: '14. Джунгли (Яд + Скорость)', multiplier: 10.0, enemies: ['poison_ant', 'fast_fly'] },
            { id: 'crystal', name: '15. Кристалл (Прочные Враги)', multiplier: 11.0, enemies: ['spider', 'lava_roach'] },
            { id: 'space', name: '16. Космос (Все Враги)', multiplier: 12.0, enemies: ['spider', 'cockroach', 'ice_spider', 'poison_ant'] },
            { id: 'ocean', name: '17. Океан (Лед + Яд)', multiplier: 13.0, enemies: ['ice_spider', 'poison_ant'] },
            { id: 'moon', name: '18. Луна (Медленные и Толстые)', multiplier: 14.0, enemies: ['cockroach', 'lava_roach'] },
            { id: 'hell', name: '19. Ад (Лава 2.0)', multiplier: 15.0, enemies: ['lava_roach', 'fast_fly'] },
            { id: 'paradise', name: '20. Рай (Финальное испытание)', multiplier: 20.0, enemies: ['spider', 'cockroach', 'ice_spider', 'poison_ant', 'lava_roach', 'fast_fly'] }
        ];

        
        // === ПЕРЕМЕННЫЕ ИГРЫ ===
        let game = {
            state: 'menu', 
            mapState: 'menu', 
            currentLevel: 'world', 
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
            upgrades: {
                speed: 1, 
                bulletSpeed: 1, 
                bulletDamage: 1, 
            },
            unlockedLevels: 1, // Изначально разблокирован только первый уровень
            invulnerabilityTimer: 0, 
            burnNextTick: 0,
            poisonNextTick: 0, 
            slowedTimer: 0 // Таймер замедления
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
        
        // --- ЭФФЕКТЫ ---
        let burnEffect = { active: false, endTime: 0, damagePerSecond: 1, duration: 5000 };
        let poisonEffect = { active: false, endTime: 0, damagePerSecond: 2, duration: 3000 };
        let slowEffect = { active: false, endTime: 0, duration: 3000 };
        
        // --- DOM ЭЛЕМЕНТЫ ---
        let scoreDisplay, coinsDisplay, healthDisplay, shieldDisplay, woodDisplay, killsDisplay;
        let pencilButton, cancelButton, shopButton, shopScreen, upgradeList, shopCoinsDisplay;
        let gameOverScreen, finalScoreDisplay, menuScreen, victoryScreen, victoryMessage, menuList;
        
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
                localStorage.setItem('unlockedLevels', game.unlockedLevels);
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
            menuList.innerHTML = '';
            BIOMES.forEach((biome, index) => {
                const levelNum = index + 1;
                const button = document.createElement('button');
                button.className = 'level-button';
                button.textContent = biome.name;
                button.onclick = () => startGame(biome.id);
                
                if (levelNum > game.unlockedLevels) {
                    button.disabled = true;
                    button.textContent += ' (Забл.)';
                    button.style.backgroundColor = '#444';
                } else if (levelNum === BIOMES.length) {
                    button.style.backgroundColor = '#cc00cc'; // Финальный уровень
                } else if (biome.multiplier >= 4.0) {
                    button.style.backgroundColor = '#cc6600'; // Сложные уровни
                }
                
                menuList.appendChild(button);
            });
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
            
            const currentLevel = Math.round((game.upgrades[id] - 1) / upgrade.effect) + 1;
            
            if (currentLevel > upgrade.max) {
                 alert('Достигнут максимальный уровень улучшения!');
                 return;
            }

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

            const biomeData = BIOMES.find(b => b.id === game.mapState);
            const maxHP = 500 * biomeData.multiplier; 

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
                damage: 50 * biomeData.multiplier,
                speed: 1.5 * Math.sqrt(biomeData.multiplier), 
                nextShotTime: Date.now() + 2000,
                isLevel: biomeData.id 
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
            if (game.state === 'menu' || game.state === 'shop' || game.state === 'gameOver' || game.state === 'victory') return;

            let dx = 0;
            let dy = 0;
            
            let currentSpeed = PLAYER_SPEED * game.upgrades.speed; 
            let effectiveSpeed = currentSpeed; 
            
            // Эффект замедления
            if (slowEffect.active) {
                effectiveSpeed *= 0.4;
            }

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
            
            // Ограничение карты
            const mapBoundary = 1000 * Math.sqrt(BIOMES.find(b => b.id === game.mapState).multiplier); 
            game.playerX = Math.max(-mapBoundary, Math.min(game.playerX, mapBoundary));
            game.playerY = Math.max(-mapBoundary, Math.min(game.playerY, mapBoundary));


            cameraX = game.playerX - PLAYER_DRAW_X;
            cameraY = game.playerY - PLAYER_DRAW_Y;
        }

        function startGame(mapType) {
            
            const levelIndex = BIOMES.findIndex(b => b.id === mapType);
            if (levelIndex + 1 > game.unlockedLevels) {
                 alert("Этот уровень заблокирован!");
                 return;
            }

            try {
                game.state = mapType;
                game.mapState = mapType;
                game.currentLevel = mapType; 
                game.playerX = 0;
                game.playerY = 0;
                game.health = 100;
                game.kills = 0;
                
                game.invulnerabilityTimer = Date.now() + 1000; 
                burnEffect.active = false; 
                poisonEffect.active = false; 
                slowEffect.active = false; 

                enemies = []; 
                objects = [];
                bullets = [];
                coins = [];
                boss = null;
                rockets = [];
                
                menuScreen.style.display = 'none'; 
                
                shopButton.style.display = 'block';
                joystickBase.style.display = 'flex';
                if (game.hasPencil) pencilButton.style.display = 'block';
                if (game.isDrawingMode) exitDrawingMode(); 

                if (enemySpawnInterval) clearInterval(enemySpawnInterval);
                
                initMapObjects(mapType);
                
                const biomeData = BIOMES[levelIndex];
                const spawnRate = 1000 / biomeData.multiplier; 
                
                enemySpawnInterval = setInterval(() => {
                    if (game.state === biomeData.id) {
                        const enemyType = biomeData.enemies[Math.floor(Math.random() * biomeData.enemies.length)];
                        createEnemy(undefined, undefined, enemyType, biomeData.multiplier);
                    }
                }, spawnRate); 
                
            } catch (error) {
                 alert("Произошла ошибка при запуске игры: " + error.message);
                 game.state = 'menu';
            }
        }
        
        function initMapObjects(mapType) {
            const biomeData = BIOMES.find(b => b.id === mapType);
            const initialEnemyCount = 30 * biomeData.multiplier;
            const range = 500; 

            if (mapType === 'world') {
                 // Деревья и карандаш только в "мире"
                 let initialTrees = [];
                 for(let i=0; i<15; i++) {
                     const angle = Math.random() * Math.PI * 2;
                     const radius = 1000 + Math.random() * 500;
                     initialTrees.push({ x: radius * Math.cos(angle), y: radius * Math.sin(angle) });
                 }
                 const pencilTreeIndex = Math.floor(Math.random() * initialTrees.length);
                 for (let i = 0; i < initialTrees.length; i++) {
                     const t = initialTrees[i];
                     const treeObj = spawnObject(t.x, t.y, 'tree');
                     if (i === pencilTreeIndex && !game.hasPencil) {
                         spawnObject(t.x + 30, t.y + 30, 'pencil');
                     }
                 }
            } else if (mapType === 'volcano' || mapType === 'hell') {
                 // Лавовые бассейны
                 for(let i=0; i<10; i++) {
                    spawnObject((Math.random() - 0.5) * range, (Math.random() - 0.5) * range, 'lava_pool').size = 50; 
                 }
            } else if (mapType === 'winter' || mapType === 'ocean') {
                 // Ледяные препятствия
                 for(let i=0; i<10; i++) {
                    spawnObject((Math.random() - 0.5) * range, (Math.random() - 0.5) * range, 'ice_patch').size = 40; 
                 }
            }

            // Изначальное появление врагов
            for(let i=0; i<initialEnemyCount; i++) { 
                const enemyType = biomeData.enemies[Math.floor(Math.random() * biomeData.enemies.length)];
                createEnemy((Math.random() - 0.5) * range, (Math.random() - 0.5) * range, enemyType, biomeData.multiplier);
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

        // === 2. СНАРЯДЫ И ЛОГИКА МУЛЬТИТАЧА (ИСПРАВЛЕНА) ===
        
        CANVAS.addEventListener('touchstart', (e) => {
            if (game.state === 'menu' || game.state === 'shop' || game.state === 'gameOver' || game.state === 'victory') return;
            
            const rect = CANVAS.getBoundingClientRect();
            let bulletFired = false; 
            
            for (let i = 0; i < e.changedTouches.length; i++) {
                const touch = e.changedTouches[i];
                
                // Пропускаем касание, если оно связано с джойстиком или пуля уже выпущена
                if (touch.identifier === joystickTouchId || bulletFired) { 
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
                        bulletFired = true; // Выпущена только одна пуля
                    }
                }
            }
        });
        
        CANVAS.addEventListener('click', (e) => {
            if (game.state === 'menu' || game.state === 'shop' || game.state === 'gameOver' || game.state === 'victory') return;
            if (e.target.tagName === 'BUTTON') return; 
            
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
                    targetTree.type = 'green_tree';
                    game.wood += 2; 
                    spawnCoin(targetTree.x, targetTree.y); 
                    
                    const killRadius = 150;
                    for (let i = enemies.length - 1; i >= 0; i--) {
                        const e = enemies[i];
                        if (distance(targetTree.x, targetTree.y, e.x, e.y) < killRadius && (e.type === 'spider' || e.type === 'ice_spider')) {
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
                        const biomeData = BIOMES.find(b => b.id === game.mapState);
                        const nextLevelIndex = BIOMES.findIndex(b => b.id === game.mapState) + 1;
                        
                        boss = null;
                        rockets = [];
                        
                        if (nextLevelIndex > game.unlockedLevels) {
                            game.unlockedLevels = nextLevelIndex;
                            saveGameProgress(); 
                            victoryMessage.textContent = `УРОВЕНЬ ${nextLevelIndex} РАЗБЛОКИРОВАН!`;
                        } else {
                            victoryMessage.textContent = `УРОВЕНЬ ${nextLevelIndex} УЖЕ РАЗБЛОКИРОВАН!`;
                        }
                        
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
                
                // Эффект ледяной ловушки (замедление)
                if ((obj.type === 'ice_patch') && distance(game.playerX, game.playerY, obj.x, obj.y) < PLAYER_SIZE/2 + obj.size/2) {
                    slowEffect.active = true;
                    slowEffect.endTime = Date.now() + slowEffect.duration;
                }
                
                // Длительность паутины
                if (obj.type === 'web') {
                    obj.duration--;
                    if (obj.duration <= 0) {
                         objects.splice(i, 1);
                         continue;
                    }
                }
            }
        }

        function createEnemy(x, y, type, multiplier = 1) {
            
            for (let i = 0; i < 1; i++) { // Создаем 1 врага, множитель влияет на характеристики
                if (x === undefined) {
                    const angle = Math.random() * Math.PI * 2;
                    const radius = W / 2 + 100; 
                    x = game.playerX + radius * Math.cos(angle);
                    y = game.playerY + radius * Math.sin(angle);
                }
                
                const baseSpeed = (type === 'spider' || type === 'ice_spider') ? 5 : ((type === 'lava_roach') ? 0.5 : 2);
                const baseDamage = (type === 'spider') ? 30 : ((type === 'lava_roach') ? 6 : ((type === 'poison_ant') ? 10 : 5));
                const speedMultiplier = (type === 'fast_fly') ? 3 : 1; // Мухи всегда быстрые
                
                enemies.push({ 
                    x, 
                    y, 
                    size: 20, 
                    type, 
                    speed: baseSpeed * multiplier * speedMultiplier, 
                    damage: baseDamage * multiplier, 
                    state: 'idle' 
                });
            }
        }

        function applyBurnEffect(damage) {
            if (!game.isShieldActive && !(Date.now() < game.invulnerabilityTimer)) {
                game.health -= damage;
                burnEffect.active = true;
                burnEffect.endTime = Date.now() + burnEffect.duration;
                game.burnNextTick = Date.now() + 1000;
                if (game.health <= 0) endGame();
            }
        }
        
        function applyPoisonEffect(damage) {
            if (!game.isShieldActive && !(Date.now() < game.invulnerabilityTimer)) {
                game.health -= damage;
                poisonEffect.active = true;
                poisonEffect.endTime = Date.now() + poisonEffect.duration;
                game.poisonNextTick = Date.now() + 1000;
                if (game.health <= 0) endGame();
            }
        }
        
        function applySlowEffect() {
            if (!game.isShieldActive && !(Date.now() < game.invulnerabilityTimer)) {
                 slowEffect.active = true;
                 slowEffect.endTime = Date.now() + slowEffect.duration;
            }
        }

        function updateEnemies() {
            const isPlayerInvulnerable = Date.now() < game.invulnerabilityTimer;

            for (let i = enemies.length - 1; i >= 0; i--) {
                const e = enemies[i];
                const distToPlayer = distance(game.playerX, game.playerY, e.x, e.y);
                
                if (boss) {
                    e.state = 'flee'; 
                } else if (distToPlayer < AGGRO_RADIUS) {
                    e.state = 'aggressive'; 
                } else {
                    e.state = 'idle';
                }

                if (e.state === 'aggressive' || e.state === 'flee') {
                    const angle = Math.atan2(game.playerY - e.y, game.playerX - e.x);
                    const moveSign = e.state === 'aggressive' ? 1 : -1;
                    e.x += e.speed * Math.cos(angle) * moveSign;
                    e.y += e.speed * Math.sin(angle) * moveSign;
                }

                // Столкновение с игроком
                if (distToPlayer < PLAYER_SIZE/2 + e.size/2) {
                    if (!boss) { 
                        if (!game.isShieldActive && !isPlayerInvulnerable) { 
                            if (e.type === 'lava_roach') {
                                applyBurnEffect(e.damage);
                            } else if (e.type === 'poison_ant') {
                                applyPoisonEffect(e.damage);
                            } else if (e.type === 'ice_spider') {
                                applySlowEffect();
                                game.health -= e.damage;
                            } else {
                                game.health -= e.damage;
                            }
                            
                            if (e.type === 'spider' && game.mapState === 'world' && Math.random() < 0.5) { 
                                spawnObject(e.x, e.y, 'web').size = 25; 
                            }
                            
                            healthDisplay.textContent = Math.max(0, game.health);

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
            }
        }
        
        function updateBoss() {
            if (!boss) return;
            const now = Date.now();
            const distToPlayer = distance(game.playerX, game.playerY, boss.x, boss.y);

            const angle = Math.atan2(game.playerY - boss.y, game.playerX - boss.x);
            boss.x += boss.speed * Math.cos(angle);
            boss.y += boss.speed * Math.sin(angle);

            if (now >= boss.nextShotTime) {
                if (distToPlayer < VIEW_RADIUS + 200) { 
                    const multiplier = BIOMES.find(b => b.id === boss.isLevel).multiplier;
                    
                    if (multiplier >= 15) { // 3 ракеты
                         shootRocket(boss.x, boss.y, angle);
                         shootRocket(boss.x, boss.y, angle + 0.2);
                         shootRocket(boss.x, boss.y, angle - 0.2);
                    }
                    else if (multiplier >= 4.0) { // 2 ракеты
                        shootRocket(boss.x + 10, boss.y, angle + 0.1);
                        shootRocket(boss.x - 10, boss.y, angle - 0.1);
                    } else { // 1 ракета
                        shootRocket(boss.x, boss.y, angle);
                    }
                }
                boss.nextShotTime = now + (2000 / Math.sqrt(multiplier)); 
            }
        }
        
        function updatePlayerEffects() {
            const now = Date.now();
            
            // --- Эффект Горения ---
            if (burnEffect.active) {
                if (now >= burnEffect.endTime) {
                    burnEffect.active = false;
                } else if (now >= game.burnNextTick) {
                    game.health -= burnEffect.damagePerSecond; 
                    game.burnNextTick = now + 1000;
                    if (game.health <= 0) endGame();
                }
            }
            
            // --- Эффект Отравления ---
            if (poisonEffect.active) {
                if (now >= poisonEffect.endTime) {
                    poisonEffect.active = false;
                } else if (now >= game.poisonNextTick) {
                    game.health -= poisonEffect.damagePerSecond; 
                    game.poisonNextTick = now + 1000;
                    if (game.health <= 0) endGame();
                }
            }
            
            // --- Эффект Замедления ---
            if (slowEffect.active) {
                if (now >= slowEffect.endTime) {
                    slowEffect.active = false;
                }
            }

            healthDisplay.textContent = Math.max(0, game.health);
        }

        // === 3. ФУНКЦИИ ОТРИСОВКИ ===

        function drawBoss() {
            if (!boss) return;
            const screenX = boss.x - cameraX;
            const screenY = boss.y - cameraY;
            
            CTX.fillStyle = 'darkred';
            CTX.beginPath();
            CTX.arc(screenX, screenY, boss.size, 0, Math.PI * 2);
            CTX.fill();
            
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
            
            switch (e.type) {
                case 'cockroach':
                    CTX.fillStyle = '#8B4513'; // Коричневый
                    CTX.fillRect(screenX - halfSize, screenY - halfSize, e.size, e.size);
                    break;
                case 'spider':
                    CTX.fillStyle = '#666'; // Темно-серый
                    CTX.fillRect(screenX - halfSize, screenY - halfSize, e.size, e.size);
                    CTX.fillStyle = 'white';
                    CTX.fillRect(screenX - 2, screenY - 2, 4, 4);
                    break;
                case 'lava_roach':
                    CTX.fillStyle = '#ff6600'; // Ярко-оранжевый (Лава)
                    CTX.fillRect(screenX - halfSize, screenY - halfSize, e.size, e.size);
                    if (Date.now() % 300 < 150) { // Эффект огня
                        CTX.fillStyle = 'red';
                        CTX.fillRect(screenX - halfSize - 2, screenY - halfSize - 2, e.size + 4, e.size + 4);
                    }
                    break;
                case 'ice_spider':
                    CTX.fillStyle = '#00ffff'; // Голубой (Лед)
                    CTX.fillRect(screenX - halfSize, screenY - halfSize, e.size, e.size);
                    CTX.fillStyle = 'white';
                    CTX.fillRect(screenX - 2, screenY - 2, 4, 4);
                    break;
                case 'poison_ant':
                    CTX.fillStyle = '#006400'; // Темно-зеленый (Яд)
                    CTX.fillRect(screenX - halfSize, screenY - halfSize, e.size, e.size);
                    break;
                case 'fast_fly':
                    CTX.fillStyle = '#999'; // Серый (Муха)
                    CTX.beginPath();
                    CTX.arc(screenX, screenY, halfSize, 0, Math.PI * 2);
                    CTX.fill();
                    break;
                default:
                    CTX.fillStyle = 'brown';
                    CTX.fillRect(screenX - halfSize, screenY - halfSize, e.size, e.size);
                    break;
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
            } else if (obj.type === 'ice_patch') {
                CTX.fillStyle = 'rgba(0, 191, 255, 0.7)'; 
                CTX.beginPath();
                CTX.arc(screenX, screenY, obj.size, 0, Math.PI * 2);
                CTX.fill();
            } else if (obj.type === 'web') { 
                CTX.fillStyle = 'rgba(150, 150, 150, 0.7)'; 
                CTX.fillRect(screenX - obj.size/2, screenY - obj.size/2, obj.size, obj.size);
            }
        }
        
        function drawBackground(mapType) {
            switch (mapType) {
                case 'world':
                    CTX.fillStyle = '#333'; 
                    CTX.fillRect(0, 0, W, H);
                    CTX.fillStyle = '#555'; 
                    const segmentSize = 30;
                    let roadOffset = (game.playerY * 0.5) % segmentSize;
                    for (let i = -1; i < H / segmentSize + 1; i++) {
                        const yPos = i * segmentSize + roadOffset;
                        CTX.fillRect(W / 2 - 2, yPos, 4, 20); 
                    }
                    break;
                case 'cave':
                case 'deep_cave':
                    CTX.fillStyle = '#221100'; 
                    CTX.fillRect(0, 0, W, H);
                    break;
                case 'volcano':
                case 'hell':
                    CTX.fillStyle = '#4d0000'; 
                    CTX.fillRect(0, 0, W, H);
                    CTX.fillStyle = '#800000'; 
                    for(let i=0; i<W; i+=20) CTX.fillRect(i, 0, 10, H);
                    break;
                case 'swamp':
                case 'jungle':
                    CTX.fillStyle = '#224422'; 
                    CTX.fillRect(0, 0, W, H);
                    break;
                case 'winter':
                case 'ocean':
                    CTX.fillStyle = '#87cefa'; 
                    CTX.fillRect(0, 0, W, H);
                    break;
                case 'desert':
                case 'canyon':
                case 'moon':
                    CTX.fillStyle = '#f5deb3'; 
                    CTX.fillRect(0, 0, W, H);
                    break;
                default:
                    CTX.fillStyle = '#333'; 
                    CTX.fillRect(0, 0, W, H);
                    break;
            }
        }

        function draw() {
            if (game.state === 'menu' || game.state === 'shop' || game.state === 'gameOver' || game.state === 'victory') {
                 CTX.clearRect(0, 0, W, H);
                 return;
            }

            drawBackground(game.mapState);

            for(let obj of objects) {
                const screenX = obj.x - cameraX;
                const screenY = obj.y - cameraY;
                drawObjectShape(obj, screenX, screenY);
            }
            
            CTX.fillStyle = 'gold';
            for(let c of coins) {
                const screenX = c.x - cameraX;
                const screenY = c.y - cameraY;
                CTX.fillRect(screenX - c.size/2, screenY - c.size/2, c.size, c.size);
            }

            CTX.fillStyle = '#ffff00';
            for (let b of bullets) {
                const screenX = b.x - cameraX;
                const screenY = b.y - cameraY;
                CTX.fillRect(screenX - b.size/2, screenY - b.size/2, b.size, b.size);
            }
            
            drawRockets();

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
            
            // Отрисовка эффекта горения/отравления
            if (burnEffect.active) {
                CTX.strokeStyle = 'red';
                CTX.lineWidth = 2;
                CTX.beginPath();
                CTX.arc(PLAYER_DRAW_X, PLAYER_DRAW_Y, PLAYER_SIZE + 5, 0, Math.PI * 2);
                CTX.stroke();
                
                if (Date.now() % 200 < 100) {
                    CTX.fillStyle = 'rgba(255, 100, 0, 0.5)';
                    CTX.beginPath();
                    CTX.arc(PLAYER_DRAW_X, PLAYER_DRAW_Y, PLAYER_SIZE * 1.5, 0, Math.PI * 2);
                    CTX.fill();
                }
            }
            if (poisonEffect.active) {
                CTX.strokeStyle = 'lime';
                CTX.lineWidth = 2;
                CTX.beginPath();
                CTX.arc(PLAYER_DRAW_X, PLAYER_DRAW_Y, PLAYER_SIZE + 5, 0, Math.PI * 2);
                CTX.stroke();
            }
            // Отрисовка эффекта замедления
            if (slowEffect.active) {
                CTX.strokeStyle = 'white';
                CTX.lineWidth = 2;
                CTX.beginPath();
                CTX.arc(PLAYER_DRAW_X, PLAYER_DRAW_Y, PLAYER_SIZE * 1.6, 0, Math.PI * 2);
                CTX.stroke();
            }
        }


        function initDOM() {
            menuScreen = document.getElementById('menu-screen');
            victoryScreen = document.getElementById('victory-screen'); 
            victoryMessage = document.getElementById('victory-message');
            menuList = document.getElementById('menu-list');

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
            joystickBase.style.display = 'none';
            
            const savedLevels = localStorage.getItem('unlockedLevels');
            if (savedLevels) {
                 game.unlockedLevels = parseInt(savedLevels) || 1;
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
            try {
                if (game.state !== 'menu' && game.state !== 'shop' && game.state !== 'gameOver' && game.state !== 'victory') {
                    movePlayer();
                    updateBullets();
                    updateRockets(); 
                    updateEnemies();
                    updateBoss(); 
                    checkCoinCollision();
                    updateObjects();
                    updatePlayerEffects(); // Обновление эффектов горения/отравления/замедления
                }
                
                draw();
                
                scoreDisplay.textContent = game.score;
                woodDisplay.textContent = game.wood; 
                killsDisplay.textContent = game.kills; 
                
            } catch (error) {
                 alert("Критическая ошибка: " + error.message + ". Игра будет перезагружена.");
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
