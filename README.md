<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <title>Cyber Snake | 赛博贪吃蛇</title>
    <link href="https://cdn.staticfile.org/tailwindcss/2.2.19/tailwind.min.css" rel="stylesheet">
    <link href="https://cdn.staticfile.org/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&family=Noto+Sans+SC&display=swap');
        body { 
            margin:0; 
            font-family: 'Noto Sans SC', sans-serif;
            background-color: #000;
            overflow: hidden;
        }
        canvas { position: fixed; top:0; left:0; }
        .glass-panel { 
            background: rgba(50,20,80,0.85); 
            backdrop-filter: blur(10px);
            border: 2px solid #0ff;
            box-shadow: 0 0 20px rgba(0,255,255,0.5);
        }
        .cyber-btn {
            background: linear-gradient(45deg, #8b5cf6, #3b82f6);
            color: white;
            text-shadow: 0 0 5px #fff;
            border: 1px solid #0ff;
            transition: all 0.3s;
            text-transform: uppercase;
            letter-spacing: 1px;
        }
        .cyber-btn:hover {
            background: linear-gradient(45deg, #3b82f6, #8b5cf6);
            box-shadow: 0 0 15px rgba(0,255,255,0.8);
            transform: translateY(-2px);
        }
        .cyber-text {
            font-family: 'Press Start 2P', cursive;
            color: #0ff;
            text-shadow: 2px 2px 0px #ff00ff, -2px -2px 0px #00ffff;
        }
        .grid-bg {
            background-image: linear-gradient(rgba(66, 0, 128, 0.5) 1px, transparent 1px),
                              linear-gradient(90deg, rgba(66, 0, 128, 0.5) 1px, transparent 1px);
            background-size: 20px 20px;
            background-position: center center;
        }
        .neon-border {
            border: 2px solid #0ff;
            box-shadow: 0 0 10px #0ff, inset 0 0 10px #0ff;
        }
        .neon-text {
            color: #fff;
            text-shadow: 0 0 5px #0ff, 0 0 10px #0ff;
        }
        .sound-btn {
            background: rgba(20, 10, 40, 0.7);
            color: #0ff;
            border: 1px solid #0ff;
            transition: all 0.3s;
            width: 50px;
            height: 50px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            box-shadow: 0 0 10px rgba(0,255,255,0.5);
        }
        .sound-btn:hover {
            background: rgba(40, 20, 80, 0.9);
            box-shadow: 0 0 15px rgba(0,255,255,0.8);
        }
        .sound-btn i {
            font-size: 1.5rem;
            text-shadow: 0 0 5px #0ff;
        }
        .volume-slider {
            -webkit-appearance: none;
            width: 80px;
            height: 8px;
            background: linear-gradient(90deg, #8b5cf6, #3b82f6);
            border-radius: 4px;
            outline: none;
            border: 1px solid #0ff;
            box-shadow: 0 0 5px rgba(0,255,255,0.5);
        }
        .volume-slider::-webkit-slider-thumb {
            -webkit-appearance: none;
            appearance: none;
            width: 16px;
            height: 16px;
            border-radius: 50%;
            background: #0ff;
            cursor: pointer;
            box-shadow: 0 0 8px rgba(0,255,255,0.8);
        }
    </style>
</head>
<body class="grid-bg">
    <!-- 场景容器 -->
    <div id="scene" class="fixed top-0 left-0 w-full h-full"></div>
    
    <!-- 顶部得分显示 -->
    <div class="fixed top-4 left-1/2 transform -translate-x-1/2 glass-panel px-6 py-3 rounded-full z-10">
        <h2 class="cyber-text text-2xl">得分: <span id="score">0</span></h2>
    </div>

    <!-- 右上角音乐控制 -->
    <div class="fixed top-4 right-4 glass-panel rounded-lg p-4 z-10 flex items-center space-x-3">
        <button id="toggleMusic" class="sound-btn">
            <i class="fas fa-volume-up"></i>
        </button>
        <input type="range" id="volumeSlider" class="volume-slider" min="0" max="1" step="0.1" value="0.5">
    </div>

    <!-- 左侧控制面板 -->
    <div class="fixed left-4 top-1/4 glass-panel rounded-lg p-5 shadow-lg z-10 neon-border">
        <div class="w-64">
            <h2 class="neon-text text-xl mb-4 font-bold">控制说明</h2>
            <div class="text-white mb-4 space-y-3">
                <p class="flex items-center"><i class="fas fa-arrows-alt text-purple-400 mr-3 text-xl"></i>←→↑↓ 控制方向</p>
                <p class="flex items-center"><i class="fas fa-pause text-purple-400 mr-3 text-xl"></i>空格键 暂停/继续</p>
                <p class="flex items-center"><i class="fas fa-bolt text-yellow-400 mr-3 text-xl"></i>加速时注意避免撞墙!</p>
                <p class="flex items-center"><i class="fas fa-volume-up text-purple-400 mr-3 text-xl"></i>右上角可控制音量</p>
            </div>
            <div class="mt-4 border-t border-purple-500 pt-4">
                <button id="restart" class="cyber-btn px-4 py-2 rounded-md w-full flex justify-center items-center">
                    <i class="fas fa-redo mr-2"></i>重新开始
                </button>
            </div>
        </div>
    </div>

    <!-- 游戏结束弹窗 -->
    <div id="gameOver" class="fixed inset-0 hidden items-center justify-center bg-black/70 z-20">
        <div class="glass-panel p-8 rounded-lg text-center neon-border transform scale-110">
            <h2 class="cyber-text text-3xl mb-6">游戏结束</h2>
            <p class="neon-text text-2xl mb-6">最终得分: <span id="finalScore" class="text-yellow-300 text-3xl">0</span></p>
            <button onclick="startGame()" class="cyber-btn px-6 py-3 rounded-md">
                <i class="fas fa-play mr-2"></i>再来一局
            </button>
        </div>
    </div>

    <!-- 添加一个开始游戏的弹窗，让用户点击以激活音频 -->
    <div id="startPrompt" class="fixed inset-0 flex items-center justify-center bg-black/80 z-30">
        <div class="glass-panel p-8 rounded-lg text-center neon-border transform scale-110">
            <h2 class="cyber-text text-3xl mb-6">赛博贪吃蛇</h2>
            <p class="text-white mb-6">点击开始游戏并启用声音</p>
            <button id="startGameBtn" class="cyber-btn px-6 py-3 rounded-md">
                <i class="fas fa-play mr-2"></i>开始游戏
            </button>
        </div>
    </div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/gsap.min.js"></script>
<script>
    let scene, camera, renderer, snake = [], food, score = 0;
    let direction = { x: 1, y: 0 };
    let nextDirection = { x: 1, y: 0 };
    let isPaused = false;
    let lastUpdateTime = 0;
    let updateInterval = 150; // 毫秒
    const CELL_SIZE = 20;
    const GRID_WIDTH = 30;
    const GRID_HEIGHT = 20;
    let particles = [];
    let isMuted = false;
    
    // 计算游戏区域
    const GAME_WIDTH = GRID_WIDTH * CELL_SIZE;
    const GAME_HEIGHT = GRID_HEIGHT * CELL_SIZE;

    // 使用经过测试的可用音频源
    const AUDIO_URLS = {
        // 背景音乐：8-bit电子风格
        background: "https://vgmsite.com/soundtracks/mega-man-11-original-soundtrack/ehlhxgwm/1-01.%20Title%20Theme.mp3",
        // 吃食物音效：简短的电子音
        eat: "https://vgmsite.com/soundtracks/mega-man-11-original-soundtrack/vhxfpxtn/1-28.%20Get%20Weapon.mp3",
        // 游戏结束音效：电子爆炸声
        gameOver: "https://vgmsite.com/soundtracks/mega-man-11-original-soundtrack/ojqmqxtn/1-29.%20Game%20Over.mp3"
    };

    let bgMusic, eatSound, gameOverSound;

    const volumeSlider = document.getElementById('volumeSlider');
    const toggleMusic = document.getElementById('toggleMusic');
    const startPrompt = document.getElementById('startPrompt');
    const startGameBtn = document.getElementById('startGameBtn');

    // 初始化游戏
    function init() {
        // 创建2D画布
        const canvas = document.createElement('canvas');
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
        document.getElementById('scene').appendChild(canvas);
        const ctx = canvas.getContext('2d');
        
        // 创建初始蛇
        createSnake();
        spawnFood();
        
        // 事件监听
        document.addEventListener('keydown', handleKeyPress);
        window.addEventListener('resize', () => {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        });
        document.getElementById('restart').addEventListener('click', startGame);
        
        // 音频控制
        initAudio();
        
        // 处理开始游戏弹窗
        startGameBtn.addEventListener('click', () => {
            startPrompt.classList.add('hidden');
            startPrompt.classList.remove('flex');
            initAudio();
            playBackgroundMusic();
            startGame();
        });

        // 动画循环
        function gameLoop() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // 绘制网格背景
            drawGrid(ctx);
            
            // 游戏区域居中
            const offsetX = (canvas.width - GAME_WIDTH) / 2;
            const offsetY = (canvas.height - GAME_HEIGHT) / 2;
            
            // 绘制游戏边界
            drawBorder(ctx, offsetX, offsetY);
            
            // 更新和绘制蛇
            if (!isPaused) {
                const now = Date.now();
                if (now - lastUpdateTime >= updateInterval) {
                    lastUpdateTime = now;
                    direction = { ...nextDirection };
                    updateSnake();
                }
            }
            
            // 绘制蛇
            drawSnake(ctx, offsetX, offsetY);
            
            // 绘制食物
            drawFood(ctx, offsetX, offsetY);
            
            // 绘制粒子效果
            updateAndDrawParticles(ctx, offsetX, offsetY);
            
            // 添加霓虹扫描线效果
            drawScanlines(ctx);
            
            requestAnimationFrame(gameLoop);
        }
        
        gameLoop();
    }
    
    // 初始化音频
    function initAudio() {
        // 创建音频对象
        bgMusic = new Audio(AUDIO_URLS.background);
        eatSound = new Audio(AUDIO_URLS.eat);
        gameOverSound = new Audio(AUDIO_URLS.gameOver);
        
        // 设置背景音乐循环
        bgMusic.loop = true;
        
        // 设置音量
        const volume = 0.3; // 默认音量设为30%
        bgMusic.volume = volume;
        eatSound.volume = volume;
        gameOverSound.volume = volume;
        
        // 预加载音频
        bgMusic.load();
        eatSound.load();
        gameOverSound.load();
        
        // 设置音量滑块初始值
        document.getElementById('volumeSlider').value = volume;
        
        // 音量控制
        document.getElementById('volumeSlider').addEventListener('input', (e) => {
            const volume = e.target.value;
            bgMusic.volume = volume;
            eatSound.volume = volume;
            gameOverSound.volume = volume;
        });
        
        // 音乐开关控制
        document.getElementById('toggleMusic').addEventListener('click', () => {
            isMuted = !isMuted;
            if (isMuted) {
                bgMusic.pause();
                document.getElementById('toggleMusic').innerHTML = '<i class="fas fa-volume-mute"></i>';
            } else {
                playBackgroundMusic();
                document.getElementById('toggleMusic').innerHTML = '<i class="fas fa-volume-up"></i>';
            }
        });
    }
    
    // 播放背景音乐
    function playBackgroundMusic() {
        if (!isMuted && bgMusic) {
            try {
                bgMusic.play().catch(error => {
                    console.log("播放背景音乐失败:", error);
                });
            } catch (error) {
                console.log("播放背景音乐时发生错误:", error);
            }
        }
    }
    
    // 播放吃食物音效
    function playEatSound() {
        if (!isMuted && eatSound) {
            try {
                eatSound.currentTime = 0;
                eatSound.play().catch(error => {
                    console.log("播放食物音效失败:", error);
                });
            } catch (error) {
                console.log("播放食物音效时发生错误:", error);
            }
        }
    }
    
    // 播放游戏结束音效
    function playGameOverSound() {
        if (!isMuted && gameOverSound) {
            try {
                bgMusic.pause();
                gameOverSound.currentTime = 0;
                gameOverSound.play().catch(error => {
                    console.log("播放结束音效失败:", error);
                });
            } catch (error) {
                console.log("播放结束音效时发生错误:", error);
            }
        }
    }
    
    function drawGrid(ctx) {
        // 绘制背景网格
        ctx.fillStyle = '#000';
        ctx.fillRect(0, 0, ctx.canvas.width, ctx.canvas.height);
        
        const offsetX = (ctx.canvas.width - GAME_WIDTH) / 2;
        const offsetY = (ctx.canvas.height - GAME_HEIGHT) / 2;
        
        // 绘制网格线
        ctx.strokeStyle = 'rgba(60, 20, 120, 0.3)';
        ctx.lineWidth = 1;
        
        // 垂直线
        for (let x = 0; x <= GRID_WIDTH; x++) {
            ctx.beginPath();
            ctx.moveTo(offsetX + x * CELL_SIZE, offsetY);
            ctx.lineTo(offsetX + x * CELL_SIZE, offsetY + GAME_HEIGHT);
            ctx.stroke();
        }
        
        // 水平线
        for (let y = 0; y <= GRID_HEIGHT; y++) {
            ctx.beginPath();
            ctx.moveTo(offsetX, offsetY + y * CELL_SIZE);
            ctx.lineTo(offsetX + GAME_WIDTH, offsetY + y * CELL_SIZE);
            ctx.stroke();
        }
    }
    
    function drawBorder(ctx, offsetX, offsetY) {
        // 游戏区域外发光边框
        ctx.strokeStyle = '#0ff';
        ctx.lineWidth = 3;
        ctx.shadowColor = '#0ff';
        ctx.shadowBlur = 15;
        ctx.beginPath();
        ctx.rect(offsetX - 5, offsetY - 5, GAME_WIDTH + 10, GAME_HEIGHT + 10);
        ctx.stroke();
        ctx.shadowBlur = 0;
    }
    
    function createSnake() {
        snake = [
            { x: 5, y: 10 },
            { x: 4, y: 10 },
            { x: 3, y: 10 }
        ];
    }

    function spawnFood() {
        // 随机放置食物，避开蛇身
        let validPosition = false;
        let foodPos;
        
        while (!validPosition) {
            foodPos = {
                x: Math.floor(Math.random() * GRID_WIDTH),
                y: Math.floor(Math.random() * GRID_HEIGHT)
            };
            
            validPosition = true;
            
            // 检查食物是否与蛇身重叠
            for (const segment of snake) {
                if (segment.x === foodPos.x && segment.y === foodPos.y) {
                    validPosition = false;
                    break;
                }
            }
        }
        
        food = foodPos;
    }
    
    function createParticles(x, y, offsetX, offsetY) {
        // 创建吃到食物时的粒子效果
        const realX = offsetX + x * CELL_SIZE + CELL_SIZE/2;
        const realY = offsetY + y * CELL_SIZE + CELL_SIZE/2;
        
        for (let i = 0; i < 20; i++) {
            const angle = Math.random() * Math.PI * 2;
            const speed = 1 + Math.random() * 3;
            const size = 2 + Math.random() * 4;
            const life = 30 + Math.random() * 30;
            
            particles.push({
                x: realX,
                y: realY,
                vx: Math.cos(angle) * speed,
                vy: Math.sin(angle) * speed,
                color: `hsl(${Math.random() * 60 + 180}, 100%, 50%)`,
                size,
                life,
                maxLife: life
            });
        }
    }
    
    function updateAndDrawParticles(ctx, offsetX, offsetY) {
        for (let i = particles.length - 1; i >= 0; i--) {
            const p = particles[i];
            p.x += p.vx;
            p.y += p.vy;
            p.life--;
            
            if (p.life <= 0) {
                particles.splice(i, 1);
                continue;
            }
            
            const alpha = p.life / p.maxLife;
            ctx.fillStyle = p.color;
            ctx.globalAlpha = alpha;
            ctx.beginPath();
            ctx.arc(p.x, p.y, p.size * alpha, 0, Math.PI * 2);
            ctx.fill();
        }
        
        ctx.globalAlpha = 1;
    }
    
    function drawScanlines(ctx) {
        // 霓虹扫描线效果
        ctx.fillStyle = 'rgba(120, 0, 255, 0.03)';
        const scanLineHeight = 4;
        
        for (let y = 0; y < ctx.canvas.height; y += scanLineHeight * 2) {
            ctx.fillRect(0, y, ctx.canvas.width, scanLineHeight);
        }
    }

    function updateSnake() {
        // 移动蛇头
        const head = { ...snake[0] };
        head.x += direction.x;
        head.y += direction.y;
        
        // 边界检测
        if (head.x < 0 || head.x >= GRID_WIDTH || head.y < 0 || head.y >= GRID_HEIGHT) {
            gameOver();
            return;
        }
        
        // 自身碰撞检测
        for (let i = 0; i < snake.length; i++) {
            if (snake[i].x === head.x && snake[i].y === head.y) {
                gameOver();
                return;
            }
        }
        
        // 添加新的头部
        snake.unshift(head);
        
        // 检查是否吃到食物
        if (head.x === food.x && head.y === food.y) {
            // 吃到食物，增加分数
            score++;
            document.getElementById('score').textContent = score;
            
            // 生成粒子效果
            const offsetX = (document.getElementById('scene').firstChild.width - GAME_WIDTH) / 2;
            const offsetY = (document.getElementById('scene').firstChild.height - GAME_HEIGHT) / 2;
            createParticles(head.x, head.y, offsetX, offsetY);
            
            // 播放音效
            playEatSound();
            
            // 放置新的食物
            spawnFood();
            
            // 加快速度
            if (updateInterval > 50) {
                updateInterval -= 5;
            }
        } else {
            // 如果没吃到食物，移除尾部
            snake.pop();
        }
    }
    
    function drawSnake(ctx, offsetX, offsetY) {
        // 绘制蛇身
        for (let i = 0; i < snake.length; i++) {
            const segment = snake[i];
            
            // 渐变色蛇身
            const hue = (i * 5 + Date.now() / 50) % 360;
            
            ctx.fillStyle = i === 0 
                ? '#50ffff' // 蛇头
                : `hsl(${hue}, 100%, 60%)`; // 蛇身
                
            const x = offsetX + segment.x * CELL_SIZE;
            const y = offsetY + segment.y * CELL_SIZE;
            
            // 发光效果
            ctx.shadowColor = i === 0 ? '#00ffff' : `hsl(${hue}, 100%, 70%)`;
            ctx.shadowBlur = 10;
            
            // 蛇的形状
            if (i === 0) {
                // 蛇头为圆形
                ctx.beginPath();
                ctx.arc(
                    x + CELL_SIZE/2, 
                    y + CELL_SIZE/2, 
                    CELL_SIZE/2 - 1, 
                    0, 
                    Math.PI * 2
                );
                ctx.fill();
                
                // 眼睛
                ctx.fillStyle = '#000';
                ctx.shadowBlur = 0;
                
                // 根据方向确定眼睛位置
                const eyeOffsetX = direction.x * 3;
                const eyeOffsetY = direction.y * 3;
                
                // 左眼
                ctx.beginPath();
                ctx.arc(
                    x + CELL_SIZE/2 - 4 + eyeOffsetX, 
                    y + CELL_SIZE/2 - 4 + eyeOffsetY, 
                    2, 
                    0, 
                    Math.PI * 2
                );
                ctx.fill();
                
                // 右眼
                ctx.beginPath();
                ctx.arc(
                    x + CELL_SIZE/2 + 4 + eyeOffsetX, 
                    y + CELL_SIZE/2 - 4 + eyeOffsetY, 
                    2, 
                    0, 
                    Math.PI * 2
                );
                ctx.fill();
            } else {
                // 蛇身为圆角矩形
                const radius = 5;
                
                ctx.beginPath();
                ctx.moveTo(x + radius, y);
                ctx.lineTo(x + CELL_SIZE - radius, y);
                ctx.quadraticCurveTo(x + CELL_SIZE, y, x + CELL_SIZE, y + radius);
                ctx.lineTo(x + CELL_SIZE, y + CELL_SIZE - radius);
                ctx.quadraticCurveTo(x + CELL_SIZE, y + CELL_SIZE, x + CELL_SIZE - radius, y + CELL_SIZE);
                ctx.lineTo(x + radius, y + CELL_SIZE);
                ctx.quadraticCurveTo(x, y + CELL_SIZE, x, y + CELL_SIZE - radius);
                ctx.lineTo(x, y + radius);
                ctx.quadraticCurveTo(x, y, x + radius, y);
                ctx.closePath();
                ctx.fill();
            }
        }
        
        ctx.shadowBlur = 0;
    }
    
    function drawFood(ctx, offsetX, offsetY) {
        // 绘制食物
        const x = offsetX + food.x * CELL_SIZE;
        const y = offsetY + food.y * CELL_SIZE;
        
        // 脉动效果
        const pulseSize = Math.sin(Date.now() / 200) * 2 + 8;
        
        // 外层光环
        ctx.fillStyle = 'rgba(255, 0, 255, 0.3)';
        ctx.shadowColor = '#ff00ff';
        ctx.shadowBlur = 15;
        ctx.beginPath();
        ctx.arc(
            x + CELL_SIZE/2, 
            y + CELL_SIZE/2, 
            CELL_SIZE/2 + pulseSize, 
            0, 
            Math.PI * 2
        );
        ctx.fill();
        
        // 内部
        ctx.fillStyle = '#ff00ff';
        ctx.beginPath();
        ctx.arc(
            x + CELL_SIZE/2, 
            y + CELL_SIZE/2, 
            CELL_SIZE/3, 
            0, 
            Math.PI * 2
        );
        ctx.fill();
        
        // 高光
        ctx.fillStyle = '#ffffff';
        ctx.beginPath();
        ctx.arc(
            x + CELL_SIZE/2 - 2, 
            y + CELL_SIZE/2 - 2, 
            2, 
            0, 
            Math.PI * 2
        );
        ctx.fill();
        
        ctx.shadowBlur = 0;
    }

    function gameOver() {
        // 播放游戏结束音效
        playGameOverSound();
        
        document.getElementById('gameOver').classList.remove('hidden');
        document.getElementById('gameOver').classList.add('flex');
        document.getElementById('finalScore').textContent = score;
        isPaused = true;
    }

    function startGame() {
        score = 0;
        updateInterval = 150;
        document.getElementById('score').textContent = 0;
        document.getElementById('gameOver').classList.add('hidden');
        document.getElementById('gameOver').classList.remove('flex');
        
        // 重置蛇
        createSnake();
        direction = { x: 1, y: 0 };
        nextDirection = { x: 1, y: 0 };
        isPaused = false;
        particles = [];
        spawnFood();
        
        // 确保没有静音时播放背景音乐
        playBackgroundMusic();
    }

    function handleKeyPress(e) {
        if(e.key === ' ') {
            isPaused = !isPaused;
            // 暂停/继续音乐
            if (!isMuted) {
                if (isPaused) {
                    bgMusic.pause();
                } else {
                    bgMusic.play();
                }
            }
            return;
        }
        
        if(isPaused) return;
        
        // 使用nextDirection来避免在同一帧内多次改变方向
        switch(e.key) {
            case 'ArrowLeft': 
                if(direction.x !== 1) nextDirection = { x: -1, y: 0 }; 
                break;
            case 'ArrowRight': 
                if(direction.x !== -1) nextDirection = { x: 1, y: 0 }; 
                break;
            case 'ArrowUp': 
                if(direction.y !== 1) nextDirection = { x: 0, y: -1 }; 
                break;
            case 'ArrowDown': 
                if(direction.y !== -1) nextDirection = { x: 0, y: 1 }; 
                break;
        }
    }

    // 初始化游戏
    init();
</script>
</body>
</html>
