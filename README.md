<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>2048游戏</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.7.2/css/all.min.css" rel="stylesheet">
    
    <!-- Tailwind配置 -->
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#faa31b',
                        secondary: '#8f7a66',
                        background: '#faf8ef',
                        textDark: '#776e65',
                        textLight: '#f9f6f2',
                        cellEmpty: '#cdc1b4',
                        cell2: '#eee4da',
                        cell4: '#ede0c8',
                        cell8: '#f2b179',
                        cell16: '#f59563',
                        cell32: '#f67c5f',
                        cell64: '#f65e3b',
                        cell128: '#edcf72',
                        cell256: '#edcc61',
                        cell512: '#edc850',
                        cell1024: '#edc53f',
                        cell2048: '#edc22e',
                        cellSuper: '#3c3a32'
                    },
                    fontFamily: {
                        inter: ['Inter', 'sans-serif'],
                    },
                }
            }
        }
    </script>
    
    <!-- 自定义工具类 -->
    <style type="text/tailwindcss">
        @layer utilities {
            .content-auto {
                content-visibility: auto;
            }
            .tile-2 { background-color: #eee4da; color: #776e65; }
            .tile-4 { background-color: #ede0c8; color: #776e65; }
            .tile-8 { background-color: #f2b179; color: #f9f6f2; }
            .tile-16 { background-color: #f59563; color: #f9f6f2; }
            .tile-32 { background-color: #f67c5f; color: #f9f6f2; }
            .tile-64 { background-color: #f65e3b; color: #f9f6f2; }
            .tile-128 { background-color: #edcf72; color: #f9f6f2; }
            .tile-256 { background-color: #edcc61; color: #f9f6f2; }
            .tile-512 { background-color: #edc850; color: #f9f6f2; }
            .tile-1024 { background-color: #edc53f; color: #f9f6f2; }
            .tile-2048 { background-color: #edc22e; color: #f9f6f2; }
            .tile-super { background-color: #3c3a32; color: #f9f6f2; }
        }
    </style>
    
    <!-- 动画和基础样式 -->
    <style>
        /* 全局重置 */
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            -webkit-tap-highlight-color: transparent;
        }
        
        /* 为旧版浏览器添加CSS Grid前缀 */
        .grid-container {
            position: relative; /* 关键修复：确保父容器为相对定位 */
            display: -ms-grid;
            display: -webkit-grid;
            display: grid;
            -ms-grid-columns: (1fr)[4];
            grid-template-columns: repeat(4, 1fr);
        }
        
        .tile {
            position: absolute; /* 方块使用绝对定位 */
            -webkit-transform: translate3d(0, 0, 0);
            transform: translate3d(0, 0, 0);
            will-change: transform;
        }
        
        /* 动画关键帧 */
        @-webkit-keyframes appear {
            0% { -webkit-transform: scale(0); opacity: 0; }
            70% { -webkit-transform: scale(1.1); }
            100% { -webkit-transform: scale(1); opacity: 1; }
        }
        
        @keyframes appear {
            0% { transform: scale(0); opacity: 0; }
            70% { transform: scale(1.1); }
            100% { transform: scale(1); opacity: 1; }
        }
        
        @-webkit-keyframes merge {
            0% { -webkit-transform: scale(1); }
            50% { -webkit-transform: scale(1.2); }
            100% { -webkit-transform: scale(1); }
        }
        
        @keyframes merge {
            0% { transform: scale(1); }
            50% { transform: scale(1.2); }
            100% { transform: scale(1); }
        }
        
        .tile.appear {
            -webkit-animation: appear 200ms ease-in-out;
            animation: appear 200ms ease-in-out;
            -webkit-animation-fill-mode: backwards;
            animation-fill-mode: backwards;
        }
        
        .tile.merge {
            -webkit-animation: merge 200ms ease-in-out;
            animation: merge 200ms ease-in-out;
            -webkit-animation-fill-mode: backwards;
            animation-fill-mode: backwards;
        }
        
        /* 防止触摸事件默认行为 */
        body {
            touch-action: manipulation;
            overscroll-behavior: none;
        }
        
        /* 确保容器有最小高度 */
        .game-wrapper {
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }
        
        /* 优化字体渲染 */
        body {
            -webkit-font-smoothing: antialiased;
            -moz-osx-font-smoothing: grayscale;
        }
    </style>
</head>
<body class="bg-background font-inter text-textDark overflow-hidden">
    <!-- 游戏包装器 -->
    <div class="game-wrapper">
        <!-- 游戏容器 -->
        <div class="max-w-md w-full mx-auto">
            <!-- 标题和描述 -->
            <div class="flex justify-between items-end mb-4">
                <div>
                    <h1 class="text-[clamp(2.5rem,5vw,3.5rem)] font-bold text-secondary mb-1">2048</h1>
                    <p class="text-sm text-textDark/70 max-w-xs">通过方向键或滑动来合并相同数字的方块，尝试得到2048分！</p>
                </div>
                <div class="flex flex-col gap-2">
                    <div class="bg-secondary rounded-md text-center px-3 py-2">
                        <div class="text-xs text-textLight/70">得分</div>
                        <div id="score" class="text-xl font-bold text-textLight">0</div>
                    </div>
                    <div class="bg-secondary rounded-md text-center px-3 py-2">
                        <div class="text-xs text-textLight/70">最高分</div>
                        <div id="best-score" class="text-xl font-bold text-textLight">0</div>
                    </div>
                </div>
            </div>
            
            <!-- 控制按钮 -->
            <div class="flex justify-between items-center mb-4">
                <button id="new-game" class="bg-primary hover:bg-primary/90 text-textLight font-bold py-2 px-4 rounded-md transition-all duration-200 transform hover:scale-105 active:scale-95">
                    <i class="fa-solid fa-refresh mr-1"></i> 新游戏
                </button>
                <div class="flex gap-2">
                    <button id="bg-music-toggle" class="bg-secondary hover:bg-secondary/90 text-textLight font-bold py-2 px-3 rounded-md transition-all duration-200 transform hover:scale-105 active:scale-95">
                        <i class="fa-solid fa-music"></i>
                    </button>
                    <button id="sound-effect-toggle" class="bg-secondary hover:bg-secondary/90 text-textLight font-bold py-2 px-3 rounded-md transition-all duration-200 transform hover:scale-105 active:scale-95">
                        <i class="fa-solid fa-volume-up"></i>
                    </button>
                </div>
            </div>
            
            <!-- 游戏网格 -->
            <div class="relative bg-cellEmpty rounded-lg p-2 sm:p-3">
                <!-- 网格背景 -->
                <div id="grid-container" class="grid-container grid-cols-4 gap-2 sm:gap-3">
                    <!-- 单元格将通过JS动态生成 -->
                </div>
                
                <!-- 游戏结束/胜利遮罩 -->
                <div id="game-message" class="absolute inset-0 bg-black/70 rounded-lg flex flex-col items-center justify-center hidden">
                    <p id="game-message-text" class="text-2xl font-bold text-textLight mb-4"></p>
                    <button id="game-message-button" class="bg-primary hover:bg-primary/90 text-textLight font-bold py-2 px-4 rounded-md transition-all duration-200 transform hover:scale-105 active:scale-95">
                        再来一局
                    </button>
                </div>
            </div>
            
            <!-- 游戏说明 -->
            <div class="mt-6 text-sm text-textDark/70">
                <h3 class="font-bold mb-2">游戏说明:</h3>
                <ul class="list-disc list-inside space-y-1">
                    <li>使用 <span class="font-bold">方向键</span> 或 <span class="font-bold">滑动屏幕</span> 移动方块</li>
                    <li>相同数字的方块相撞时会 <span class="font-bold">合并成为它们的和</span></li>
                    <li>每次移动后会随机生成一个新的 <span class="font-bold">2</span> 或 <span class="font-bold">4</span></li>
                    <li>创建出 <span class="font-bold">2048</span> 方块即可获胜!</li>
                </ul>
            </div>
        </div>
    </div>
    
    <!-- 音频元素 -->
    <audio id="bg-music" loop>
        <source src="https://example.com/bg-music.mp3" type="audio/mpeg">
    </audio>
    <audio id="move-sound">
        <source src="https://example.com/move.mp3" type="audio/mpeg">
    </audio>
    <audio id="merge-sound">
        <source src="https://example.com/merge.mp3" type="audio/mpeg">
    </audio>
    <audio id="win-sound">
        <source src="https://example.com/win.mp3" type="audio/mpeg">
    </audio>
    <audio id="game-over-sound">
        <source src="https://example.com/game-over.mp3" type="audio/mpeg">
    </audio>
    
    <script>
        // 为旧版浏览器添加CSS Grid前缀支持
        document.addEventListener('DOMContentLoaded', function() {
            var style = document.createElement('style');
            style.textContent = `
                .grid-container {
                    position: relative;
                    display: -ms-grid;
                    display: -webkit-grid;
                    display: grid;
                    -ms-grid-columns: (1fr)[4];
                    grid-template-columns: repeat(4, 1fr);
                }
                .tile {
                    position: absolute;
                    -webkit-transform: translate3d(0, 0, 0);
                    transform: translate3d(0, 0, 0);
                }
            `;
            document.head.appendChild(style);
        });
        
        // 音频控制
        var bgMusic = document.getElementById('bg-music');
        var moveSound = document.getElementById('move-sound');
        var mergeSound = document.getElementById('merge-sound');
        var winSound = document.getElementById('win-sound');
        var gameOverSound = document.getElementById('game-over-sound');
        
        var bgMusicEnabled = true;
        var soundEffectEnabled = true;
        
        // 游戏配置
        var config = {
            gridSize: 4,
            startTiles: 2,
            winningValue: 2048,
            animationDuration: 200 // 动画持续时间（毫秒）
        };
        
        // 游戏状态变量
        var grid = [];
        var score = 0;
        var bestScore = localStorage.getItem('2048-best-score') || 0;
        var gameOver = false;
        var gameWon = false;
        var canMove = true;
        
        // 触摸事件变量
        var touchStartX = 0;
        var touchStartY = 0;
        var touchEndX = 0;
        var touchEndY = 0;
        
        // 设备检测
        var isMobile = /iPhone|iPad|iPod|Android/i.test(navigator.userAgent);
        
        // 初始化游戏
        function initGame() {
            // 重置游戏状态
            grid = createEmptyGrid();
            score = 0;
            gameOver = false;
            gameWon = false;
            canMove = true;
            
            // 更新UI
            updateScore();
            updateBestScore();
            renderGrid();
            addStartTiles();
            hideGameMessage();
            
            // 绑定事件
            bindEvents();
        }
        
        // 创建空网格
        function createEmptyGrid() {
            var grid = [];
            for (var x = 0; x < config.gridSize; x++) {
                grid[x] = [];
                for (var y = 0; y < config.gridSize; y++) {
                    grid[x][y] = null;
                }
            }
            return grid;
        }
        
        // 添加初始方块
        function addStartTiles() {
            for (var i = 0; i < config.startTiles; i++) {
                addRandomTile();
            }
        }
        
        // 添加随机方块
        function addRandomTile() {
            if (hasEmptyCells()) {
                // 随机选择位置
                var emptyCells = getEmptyCells();
                var cell = emptyCells[Math.floor(Math.random() * emptyCells.length)];
                
                // 90%概率生成2，10%概率生成4
                var value = Math.random() < 0.9 ? 2 : 4;
                
                // 添加方块到网格
                grid[cell.x][cell.y] = {
                    value: value,
                    merged: false,
                    new: true
                };
                
                // 更新UI
                renderTile(cell.x, cell.y, value, true);
                
                return true;
            }
            return false;
        }
        
        // 获取所有空单元格
        function getEmptyCells() {
            var emptyCells = [];
            for (var x = 0; x < config.gridSize; x++) {
                for (var y = 0; y < config.gridSize; y++) {
                    if (grid[x][y] === null) {
                        emptyCells.push({ x: x, y: y });
                    }
                }
            }
            return emptyCells;
        }
        
        // 检查是否有空单元格
        function hasEmptyCells() {
            for (var x = 0; x < config.gridSize; x++) {
                for (var y = 0; y < config.gridSize; y++) {
                    if (grid[x][y] === null) {
                        return true;
                    }
                }
            }
            return false;
        }
        
        // 渲染整个网格
        function renderGrid() {
            var gridContainer = document.getElementById('grid-container');
            gridContainer.innerHTML = '';
            
            // 计算单元格大小
            var containerWidth = gridContainer.offsetWidth;
            var cellSize = (containerWidth - (config.gridSize - 1) * 10) / config.gridSize;
            
            // 创建单元格和方块
            for (var x = 0; x < config.gridSize; x++) {
                for (var y = 0; y < config.gridSize; y++) {
                    // 创建单元格背景
                    var cell = document.createElement('div');
                    cell.className = 'bg-cellEmpty/50 rounded-md';
                    cell.style.width = cellSize + 'px';
                    cell.style.height = cellSize + 'px';
                    gridContainer.appendChild(cell);
                    
                    // 如果有方块，渲染方块
                    if (grid[x][y]) {
                        renderTile(x, y, grid[x][y].value, grid[x][y].new);
                        grid[x][y].new = false;
                    }
                }
            }
        }
        
        // 渲染单个方块
        function renderTile(x, y, value, isNew) {
            var gridContainer = document.getElementById('grid-container');
            var index = x * config.gridSize + y;
            
            // 如果方块已存在，移除它
            var existingTile = gridContainer.querySelector('.tile[data-index="' + index + '"]');
            if (existingTile) {
                existingTile.remove();
            }
            
            // 创建新方块
            var tile = document.createElement('div');
            tile.className = 'tile absolute rounded-md font-bold flex items-center justify-center transition-all duration-200 ease-in-out';
            
            // 添加值类
            var valueClass = 'tile-' + value;
            if (value >= 2048) {
                valueClass = 'tile-super';
            }
            tile.classList.add(valueClass);
            
            // 设置位置和大小
            var cellSize = (gridContainer.offsetWidth - (config.gridSize - 1) * 10) / config.gridSize;
            var xPos = y * (cellSize + 10);
            var yPos = x * (cellSize + 10);
            
            tile.style.width = cellSize + 'px';
            tile.style.height = cellSize + 'px';
            tile.style.left = xPos + 'px';
            tile.style.top = yPos + 'px';
            
            // 设置数据属性
            tile.setAttribute('data-index', index);
            tile.setAttribute('data-value', value);
            
            // 设置字体大小（根据值调整）
            var fontSize = Math.max(14, cellSize * 0.4);
            if (value >= 1024) {
                fontSize *= 0.7;
            } else if (value >= 128) {
                fontSize *= 0.85;
            }
            tile.style.fontSize = fontSize + 'px';
            
            // 添加内容
            tile.textContent = value;
            
            // 添加动画类
            if (isNew) {
                tile.classList.add('appear');
            }
            
            // 添加到容器
            gridContainer.appendChild(tile);
        }
        
        // 更新方块位置（用于移动动画）
        function updateTilePosition(x, y, value, merged) {
            var gridContainer = document.getElementById('grid-container');
            var index = x * config.gridSize + y;
            
            // 查找现有方块
            var tile = gridContainer.querySelector('.tile[data-value="' + value + '"][data-merged="false"]');
            
            if (tile) {
                // 设置新位置
                var cellSize = (gridContainer.offsetWidth - (config.gridSize - 1) * 10) / config.gridSize;
                var xPos = y * (cellSize + 10);
                var yPos = x * (cellSize + 10);
                
                // 使用CSS transform代替left/top以获得更好的性能
                tile.style.transform = 'translate(' + xPos + 'px, ' + yPos + 'px)';
                
                // 如果是合并的方块，添加合并动画
                if (merged) {
                    // 移除旧方块
                    tile.setAttribute('data-merged', 'true');
                    
                    // 创建新的合并后方块
                    setTimeout(function() {
                        renderTile(x, y, value, false);
                        var newTile = gridContainer.querySelector('.tile[data-index="' + index + '"]');
                        newTile.classList.add('merge');
                        
                        // 播放合并音效
                        if (soundEffectEnabled) {
                            playSound(mergeSound);
                        }
                    }, config.animationDuration / 2);
                }
            }
        }
        
        // 播放音效（处理iOS自动播放限制）
        function playSound(audioElement) {
            if (audioElement.paused) {
                audioElement.currentTime = 0;
                audioElement.play().catch(e => console.log('播放音效失败:', e));
            } else {
                audioElement.currentTime = 0;
            }
        }
        
        // 移动方块
        function move(direction) {
            if (!canMove || gameOver || gameWon) return false;
            
            var moved = false;
            canMove = false;
            
            // 重置所有方块的合并状态
            for (var x = 0; x < config.gridSize; x++) {
                for (var y = 0; y < config.gridSize; y++) {
                    if (grid[x][y]) {
                        grid[x][y].merged = false;
                    }
                }
            }
            
            // 根据方向处理移动
            switch(direction) {
                case 'up':
                    moved = moveUp();
                    break;
                case 'down':
                    moved = moveDown();
                    break;
                case 'left':
                    moved = moveLeft();
                    break;
                case 'right':
                    moved = moveRight();
                    break;
            }
            
            // 如果有移动，添加新方块并检查游戏状态
            if (moved) {
                // 播放移动音效
                if (soundEffectEnabled) {
                    playSound(moveSound);
                }
                
                // 添加新方块
                setTimeout(function() {
                    addRandomTile();
                    
                    // 检查游戏状态
                    checkGameState();
                    
                    // 允许再次移动
                    canMove = true;
                }, config.animationDuration);
            } else {
                // 允许再次移动
                canMove = true;
            }
            
            return moved;
        }
        
        // 向上移动
        function moveUp() {
            var moved = false;
            
            for (var y = 0; y < config.gridSize; y++) {
                for (var x = 1; x < config.gridSize; x++) {
                    if (grid[x][y]) {
                        var newX = x;
                        
                        // 找到可以移动到的最高位置
                        for (var i = x - 1; i >= 0; i--) {
                            if (grid[i][y] === null) {
                                newX = i;
                            } else if (grid[i][y].value === grid[x][y].value && !grid[i][y].merged) {
                                newX = i;
                                break;
                            } else {
                                break;
                            }
                        }
                        
                        // 如果有移动或合并
                        if (newX !== x) {
                            // 如果是合并
                            if (grid[newX][y] && grid[newX][y].value === grid[x][y].value) {
                                grid[newX][y].value *= 2;
                                grid[newX][y].merged = true;
                                score += grid[newX][y].value;
                                updateScore();
                                
                                // 检查是否达到胜利条件
                                if (grid[newX][y].value >= config.winningValue && !gameWon) {
                                    gameWon = true;
                                    showGameMessage('恭喜你获胜了！');
                                    
                                    // 播放胜利音效
                                    if (soundEffectEnabled) {
                                        playSound(winSound);
                                    }
                                }
                            } else {
                                grid[newX][y] = grid[x][y];
                            }
                            
                            grid[x][y] = null;
                            updateTilePosition(newX, y, grid[newX][y].value, grid[newX][y].merged);
                            moved = true;
                        }
                    }
                }
            }
            
            return moved;
        }
        
        // 向下移动
        function moveDown() {
            var moved = false;
            
            for (var y = 0; y < config.gridSize; y++) {
                for (var x = config.gridSize - 2; x >= 0; x--) {
                    if (grid[x][y]) {
                        var newX = x;
                        
                        // 找到可以移动到的最低位置
                        for (var i = x + 1; i < config.gridSize; i++) {
                            if (grid[i][y] === null) {
                                newX = i;
                            } else if (grid[i][y].value === grid[x][y].value && !grid[i][y].merged) {
                                newX = i;
                                break;
                            } else {
                                break;
                            }
                        }
                        
                        // 如果有移动或合并
                        if (newX !== x) {
                            // 如果是合并
                            if (grid[newX][y] && grid[newX][y].value === grid[x][y].value) {
                                grid[newX][y].value *= 2;
                                grid[newX][y].merged = true;
                                score += grid[newX][y].value;
                                updateScore();
                                
                                // 检查是否达到胜利条件
                                if (grid[newX][y].value >= config.winningValue && !gameWon) {
                                    gameWon = true;
                                    showGameMessage('恭喜你获胜了！');
                                    
                                    // 播放胜利音效
                                    if (soundEffectEnabled) {
                                        playSound(winSound);
                                    }
                                }
                            } else {
                                grid[newX][y] = grid[x][y];
                            }
                            
                            grid[x][y] = null;
                            updateTilePosition(newX, y, grid[newX][y].value, grid[newX][y].merged);
                            moved = true;
                        }
                    }
                }
            }
            
            return moved;
        }
        
        // 向左移动
        function moveLeft() {
            var moved = false;
            
            for (var x = 0; x < config.gridSize; x++) {
                for (var y = 1; y < config.gridSize; y++) {
                    if (grid[x][y]) {
                        var newY = y;
                        
                        // 找到可以移动到的最左位置
                        for (var i = y - 1; i >= 0; i--) {
                            if (grid[x][i] === null) {
                                newY = i;
                            } else if (grid[x][i].value === grid[x][y].value && !grid[x][i].merged) {
                                newY = i;
                                break;
                            } else {
                                break;
                            }
                        }
                        
                        // 如果有移动或合并
                        if (newY !== y) {
                            // 如果是合并
                            if (grid[x][newY] && grid[x][newY].value === grid[x][y].value) {
                                grid[x][newY].value *= 2;
                                grid[x][newY].merged = true;
                                score += grid[x][newY].value;
                                updateScore();
                                
                                // 检查是否达到胜利条件
                                if (grid[x][newY].value >= config.winningValue && !gameWon) {
                                    gameWon = true;
                                    showGameMessage('恭喜你获胜了！');
                                    
                                    // 播放胜利音效
                                    if (soundEffectEnabled) {
                                        playSound(winSound);
                                    }
                                }
                            } else {
                                grid[x][newY] = grid[x][y];
                            }
                            
                            grid[x][y] = null;
                            updateTilePosition(x, newY, grid[x][newY].value, grid[x][newY].merged);
                            moved = true;
                        }
                    }
                }
            }
            
            return moved;
        }
        
        // 向右移动
        function moveRight() {
            var moved = false;
            
            for (var x = 0; x < config.gridSize; x++) {
                for (var y = config.gridSize - 2; y >= 0; y--) {
                    if (grid[x][y]) {
                        var newY = y;
                        
                        // 找到可以移动到的最右位置
                        for (var i = y + 1; i < config.gridSize; i++) {
                            if (grid[x][i] === null) {
                                newY = i;
                            } else if (grid[x][i].value === grid[x][y].value && !grid[x][i].merged) {
                                newY = i;
                                break;
                            } else {
                                break;
                            }
                        }
                        
                        // 如果有移动或合并
                        if (newY !== y) {
                            // 如果是合并
                            if (grid[x][newY] && grid[x][newY].value === grid[x][y].value) {
                                grid[x][newY].value *= 2;
                                grid[x][newY].merged = true;
                                score += grid[x][newY].value;
                                updateScore();
                                
                                // 检查是否达到胜利条件
                                if (grid[x][newY].value >= config.winningValue && !gameWon) {
                                    gameWon = true;
                                    showGameMessage('恭喜你获胜了！');
                                    
                                    // 播放胜利音效
                                    if (soundEffectEnabled) {
                                        playSound(winSound);
                                    }
                                }
                            } else {
                                grid[x][newY] = grid[x][y];
                            }
                            
                            grid[x][y] = null;
                            updateTilePosition(x, newY, grid[x][newY].value, grid[x][newY].merged);
                            moved = true;
                        }
                    }
                }
            }
            
            return moved;
        }
        
        // 更新分数
        function updateScore() {
            document.getElementById('score').textContent = score;
            
            // 更新最高分
            if (score > bestScore) {
                bestScore = score;
                localStorage.setItem('2048-best-score', bestScore);
                updateBestScore();
            }
        }
        
        // 更新最高分
        function updateBestScore() {
            document.getElementById('best-score').textContent = bestScore;
        }
        
        // 显示游戏消息
        function showGameMessage(text) {
            var messageElement = document.getElementById('game-message');
            var messageText = document.getElementById('game-message-text');
            
            messageText.textContent = text;
            messageElement.classList.remove('hidden');
            
            // 添加淡入动画
            messageElement.style.animation = 'none';
            messageElement.offsetHeight; // 触发重绘
            messageElement.style.animation = 'fadeIn 0.3s ease-in-out forwards';
            
            // 如果是游戏结束，播放游戏结束音效
            if (text === '游戏结束！') {
                if (soundEffectEnabled) {
                    playSound(gameOverSound);
                }
            }
        }
        
        // 隐藏游戏消息
        function hideGameMessage() {
            document.getElementById('game-message').classList.add('hidden');
        }
        
        // 检查游戏状态
        function checkGameState() {
            // 检查是否获胜
            if (!gameWon) {
                for (var x = 0; x < config.gridSize; x++) {
                    for (var y = 0; y < config.gridSize; y++) {
                        if (grid[x][y] && grid[x][y].value >= config.winningValue) {
                            gameWon = true;
                            showGameMessage('恭喜你获胜了！');
                            
                            // 播放胜利音效
                            if (soundEffectEnabled) {
                                playSound(winSound);
                            }
                            return;
                        }
                    }
                }
            }
            
            // 检查是否游戏结束
            if (!hasEmptyCells()) {
                // 检查是否还能移动
                for (var x = 0; x < config.gridSize; x++) {
                    for (var y = 0; y < config.gridSize; y++) {
                        var value = grid[x][y].value;
                        
                        // 检查上下左右是否有相同值的方块
                        if ((x > 0 && grid[x-1][y] && grid[x-1][y].value === value) ||
                            (x < config.gridSize - 1 && grid[x+1][y] && grid[x+1][y].value === value) ||
                            (y > 0 && grid[x][y-1] && grid[x][y-1].value === value) ||
                            (y < config.gridSize - 1 && grid[x][y+1] && grid[x][y+1].value === value)) {
                            return; // 还能移动，游戏未结束
                        }
                    }
                }
                
                // 不能移动，游戏结束
                gameOver = true;
                showGameMessage('游戏结束！');
            }
        }
        
        // 绑定事件
        function bindEvents() {
            // 新游戏按钮
            document.getElementById('new-game').addEventListener('click', function() {
                initGame();
                
                // 确保音乐在用户交互后播放
                if (bgMusicEnabled && bgMusic.paused) {
                    bgMusic.play().catch(e => console.log('需要用户交互后才能播放音乐:', e));
                }
            });
            
            // 背景音乐切换
            document.getElementById('bg-music-toggle').addEventListener('click', function() {
                bgMusicEnabled = !bgMusicEnabled;
                
                if (bgMusicEnabled) {
                    this.innerHTML = '<i class="fa-solid fa-music"></i>';
                    if (bgMusic.paused) {
                        bgMusic.play().catch(e => console.log('需要用户交互后才能播放音乐:', e));
                    }
                } else {
                    this.innerHTML = '<i class="fa-solid fa-music-slash"></i>';
                    bgMusic.pause();
                }
            });
            
            // 音效切换
            document.getElementById('sound-effect-toggle').addEventListener('click', function() {
                soundEffectEnabled = !soundEffectEnabled;
                
                if
