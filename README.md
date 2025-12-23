
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>自然拼读法 - 卡片游戏</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .card {
            perspective: 1000px;
            cursor: pointer;
        }
        .card-inner {
            position: relative;
            width: 100%;
            height: 100%;
            transition: transform 0.6s;
            transform-style: preserve-3d;
        }
        .card.flipped .card-inner {
            transform: rotateY(180deg);
        }
        .card-front, .card-back {
            position: absolute;
            width: 100%;
            height: 100%;
            backface-visibility: hidden;
            border-radius: 1rem;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            padding: 1.5rem;
        }
        .card-front {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
        }
        .card-back {
            background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
            color: white;
            transform: rotateY(180deg);
        }
        .progress-bar {
            transition: width 0.3s ease;
        }
        .sound-toggle {
            position: fixed;
            top: 20px;
            right: 20px;
            z-index: 1000;
        }
        .word-hoverable {
            display: inline-block;
            padding: 2px 6px;
            margin: 0 2px;
            border-radius: 4px;
            transition: all 0.2s;
            cursor: pointer;
            position: relative;
        }
        .word-hoverable:hover {
            background-color: rgba(255, 255, 255, 0.3);
            transform: translateY(-2px);
        }
        .word-hoverable::after {
            content: '🔊';
            position: absolute;
            top: -20px;
            left: 50%;
            transform: translateX(-50%) scale(0);
            font-size: 12px;
            opacity: 0;
            transition: all 0.2s;
        }
        .word-hoverable:hover::after {
            transform: translateX(-50%) scale(1);
            opacity: 1;
        }
        .word-hoverable.speaking {
            background-color: rgba(255, 255, 255, 0.5);
            animation: pulse 0.5s ease-in-out;
        }
        @keyframes pulse {
            0%, 100% { transform: scale(1); }
            50% { transform: scale(1.1); }
        }
    </style>
</head>
<body class="bg-gradient-to-br from-purple-100 to-pink-100 min-h-screen p-4">
    <!-- 声音开关 -->
    <button onclick="toggleSound()" class="sound-toggle px-4 py-2 bg-white rounded-full shadow-lg hover:shadow-xl transition" id="sound-btn">
        <span id="sound-icon">🔊</span> <span id="sound-text">声音开</span>
    </button>

    <div class="max-w-6xl mx-auto">
        <!-- 标题 -->
        <div class="text-center mb-8">
            <h1 class="text-4xl font-bold text-purple-800 mb-2">🎮 自然拼读法卡片游戏</h1>
            <p class="text-gray-600">选择学习模式开始！鼠标悬停单词可听发音 🎧</p>
        </div>

        <!-- 模式选择 -->
        <div class="bg-white rounded-2xl shadow-lg p-6 mb-6">
            <h2 class="text-xl font-bold text-gray-800 mb-4">学习模式：</h2>
            <div class="flex flex-wrap gap-3">
                <button onclick="setMode('single')" id="mode-single" class="px-6 py-3 bg-indigo-500 text-white rounded-lg hover:bg-indigo-600 transition">📱 单卡模式</button>
                <button onclick="setMode('grid')" id="mode-grid" class="px-6 py-3 bg-gray-400 text-white rounded-lg hover:bg-gray-500 transition">📋 网格模式</button>
            </div>
        </div>

        <!-- 类别选择 -->
        <div class="bg-white rounded-2xl shadow-lg p-6 mb-6">
            <h2 class="text-xl font-bold text-gray-800 mb-4">选择类别：</h2>
            <div class="flex flex-wrap gap-3">
                <button onclick="setCategory('consonants')" class="px-6 py-3 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition">声母基础</button>
                <button onclick="setCategory('consonant-combos')" class="px-6 py-3 bg-green-500 text-white rounded-lg hover:bg-green-600 transition">声母组合</button>
                <button onclick="setCategory('vowels')" class="px-6 py-3 bg-yellow-500 text-white rounded-lg hover:bg-yellow-600 transition">韵母单字母</button>
                <button onclick="setCategory('vowel-combos')" class="px-6 py-3 bg-pink-500 text-white rounded-lg hover:bg-pink-600 transition">韵母双字母</button>
                <button onclick="setCategory('endings')" class="px-6 py-3 bg-purple-500 text-white rounded-lg hover:bg-purple-600 transition">韵母组合</button>
                <button onclick="setCategory('all')" class="px-6 py-3 bg-red-500 text-white rounded-lg hover:bg-red-600 transition">全部混合</button>
            </div>
        </div>

        <!-- 进度条 -->
        <div class="bg-white rounded-2xl shadow-lg p-6 mb-6">
            <div class="flex justify-between items-center mb-2">
                <span class="text-sm font-semibold text-gray-700">学习进度</span>
                <span class="text-sm font-semibold text-gray-700"><span id="current">0</span> / <span id="total">0</span></span>
            </div>
            <div class="w-full bg-gray-200 rounded-full h-4">
                <div id="progress" class="progress-bar bg-gradient-to-r from-green-400 to-blue-500 h-4 rounded-full" style="width: 0%"></div>
            </div>
        </div>

        <!-- 单卡模式容器 -->
        <div id="single-mode" class="hidden">
            <div class="bg-white rounded-2xl shadow-2xl p-8 mb-6">
                <div class="card h-96 max-w-2xl mx-auto" id="single-card" onclick="toggleCard(event)">
                    <div class="card-inner">
                        <div class="card-front">
                            <div class="text-8xl font-bold mb-4" id="single-front">a</div>
                            <div class="text-sm opacity-75">点击卡片查看详情</div>
                        </div>
                        <div class="card-back">
                            <div class="text-3xl font-bold mb-4" id="single-title">a</div>
                            <div class="text-lg text-center leading-relaxed" id="single-back"></div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- 导航按钮 -->
            <div class="flex justify-center items-center gap-4 mb-6">
                <button onclick="prevCard()" class="px-8 py-4 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition text-lg font-semibold disabled:bg-gray-300 disabled:cursor-not-allowed" id="prev-btn">
                    ⬅️ 上一张
                </button>
                <button onclick="nextCard()" class="px-8 py-4 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition text-lg font-semibold disabled:bg-gray-300 disabled:cursor-not-allowed" id="next-btn">
                    下一张 ➡️
                </button>
            </div>

            <!-- 快速跳转 -->
            <div class="text-center">
                <button onclick="randomCard()" class="px-6 py-3 bg-orange-500 text-white rounded-lg hover:bg-orange-600 transition">
                    🎲 随机卡片
                </button>
            </div>
        </div>

        <!-- 网格模式容器 -->
        <div id="grid-mode" class="hidden">
            <div id="card-container" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            </div>
        </div>

        <!-- 重置按钮 -->
        <div class="text-center mt-8">
            <button onclick="resetProgress()" class="px-8 py-4 bg-gray-700 text-white rounded-lg hover:bg-gray-800 transition text-lg font-semibold">🔄 重置进度</button>
        </div>
    </div>

    <script>
        const phonicsData = {
            consonants: [
                { title: 'c', front: 'c', back: '/k/ cat, cup, come\n/s/ city, cent, cycle' },
                { title: 'j', front: 'j', back: '/吱/ Jack, jump, juice' },
                { title: 'q', front: 'q', back: '/k/ question, queen, quick' },
                { title: 'r', front: 'r', back: '/弱/ red, run, rice' },
                { title: 'v', front: 'v', back: '上牙放在下唇上，然后震动\nvan, very, five' },
                { title: 'x', front: 'x', back: '/ks/ box, six, fox' },
                { title: 'z', front: 'z', back: '震动 zoo, zero, zip' }
            ],
            'consonant-combos': [
                { title: 'tr', front: 'tr', back: '/戳/ tree, train, truck' },
                { title: 'dr', front: 'dr', back: '/捉/ dress, drink, dragon' },
                { title: 'th', front: 'th', back: '/咬舌的s/ think, thank, three\n/咬舌的z/ this, that, the' },
                { title: 'ph', front: 'ph', back: '/f/ phone, photo, elephant' },
                { title: 'gh', front: 'gh', back: '/f/ laugh, cough, enough\n不发音 night, light, high' },
                { title: 'wh', front: 'wh', back: '/w/ what, when, white' },
                { title: 'kn', front: 'kn', back: '/n/ knee, know, knife' },
                { title: 'wr', front: 'wr', back: '/弱/ write, wrong, wrap' },
                { title: 'ck', front: 'ck', back: '/k/ back, black, clock' }
            ],
            vowels: [
                { title: 'a', front: 'a', back: '/哎/ bag, cat\n/A/ cake, name, make' },
                { title: 'e', front: 'e', back: '/哎/ bed, red, pen' },
                { title: 'i', front: 'i', back: '/A/ big, sit, fish\n/I/ bike, like, time' },
                { title: 'o', front: 'o', back: '/奥/ dog, hot, box\n/欧/ nose, home, hope' },
                { title: 'u', front: 'u', back: '/啊/ cup, bus, run\n/优/ cute, use, usually' },
                { title: 'y', front: 'y', back: '/A/ city\n/I/ my, fly, sky' }
            ],
            'vowel-combos': [
                { title: 'au/aw', front: 'au/aw', back: '/奥/ autumn, because, draw, saw' },
                { title: 'ai/ay', front: 'ai/ay', back: '/A/ rain, wait, day, play' },
                { title: 'ee', front: 'ee', back: '/衣/ see, tree, bee' },
                { title: 'ea', front: 'ea', back: '/衣/ sea, tea, read' },
                { title: 'ie', front: 'ie', back: '/衣/ piece, field' },
                { title: 'oa', front: 'oa', back: '/欧/ boat, coat, road' },
                { title: 'oo', front: 'oo', back: '/乌/ book, look, good, moon, food, zoo' },
                { title: 'ou/ow', front: 'ou/ow', back: '/奥/ house, mouse, cow, now' },
                { title: 'oi/oy', front: 'oi/oy', back: '/哦A/ oil, coin, boy, toy' },
                { title: 'ar', front: 'ar', back: '/阿/ car, star, park' },
                { title: 'or', front: 'or', back: '/哦/ for, short, horse' },
                { title: 'er/ir/ur', front: 'er/ir/ur', back: '/饿/ her, tiger, bird, girl, turn, nurse' }
            ],
            endings: [
                { title: 'ing', front: 'ing', back: '/嘤/' },
                { title: 'ful', front: 'ful', back: '/fao/ beautiful, wonderful, full' },
                { title: 'all', front: 'all', back: '/奥/ ball, tall, wall' },
                { title: 'tion', front: 'tion', back: '/神/ station, nation' },
                { title: 'sion', front: 'sion', back: '/神/ 或 /任/ television, decision' },
                { title: 'ture', front: 'ture', back: '/车/ picture, nature' }
            ]
        };

        let currentCategory = 'all';
        let currentMode = 'single';
        let currentIndex = 0;
        let flippedCards = new Set();
        let currentCards = [];
        let soundEnabled = true;
        let isSpeaking = false;

        // 音频上下文
        const audioContext = new (window.AudioContext || window.webkitAudioContext)();
        
        // Web Speech API
        const synth = window.speechSynthesis;
        let voices = [];

        // 加载语音列表
        function loadVoices() {
            voices = synth.getVoices();
            // 优先使用英语语音
            voices.sort((a, b) => {
                if (a.lang.startsWith('en') && !b.lang.startsWith('en')) return -1;
                if (!a.lang.startsWith('en') && b.lang.startsWith('en')) return 1;
                return 0;
            });
        }

        if (synth.onvoiceschanged !== undefined) {
            synth.onvoiceschanged = loadVoices;
        }
        loadVoices();

        // 播放单词发音
        function speakWord(word, element) {
        if (!soundEnabled) return;
        
        // 清理单词（移除标点符号）
        const cleanWord = word.replace(/[,\.]/g, '').trim();
        if (!cleanWord || /[\u4e00-\u9fa5]/.test(cleanWord) || cleanWord.startsWith('/')) return;

        // 立即停止当前播放
        if (isSpeaking) {
            synth.cancel();
        }

        // 添加视觉反馈
        if (element) {
            element.classList.add('speaking');
            setTimeout(() => element.classList.remove('speaking'), 500);
        }

        // 检查 Web Speech API 支持
        if (synth && voices.length > 0) {
            isSpeaking = true;
            const utterance = new SpeechSynthesisUtterance(cleanWord);
            
            // 选择英语语音
            const enVoice = voices.find(voice => voice.lang.startsWith('en-US')) || 
                        voices.find(voice => voice.lang.startsWith('en')) ||
                        voices[0];
            
            if (enVoice) {
                utterance.voice = enVoice;
            }
            
            utterance.rate = 0.8;
            utterance.pitch = 1;
            utterance.volume = 1;
            
            utterance.onend = () => {
                isSpeaking = false;
            };
            
            utterance.onerror = () => {
                isSpeaking = false;
            };
            
            synth.speak(utterance);
        }
    }

        // 将文本转换为可交互的单词元素
        function makeWordsHoverable(text) {
            // 分割文本，保留换行符
            const lines = text.split('\n');
            return lines.map(line => {
                // 分割每一行的单词，保留标点符号
                const parts = line.split(/(\s+|,\s*)/);
                return parts.map(part => {
                    // 检查是否为英文单词
                    if (/^[a-zA-Z]+$/.test(part.trim())) {
                        return `<span class="word-hoverable" onmouseenter="speakWord('${part.trim()}', this)">${part}</span>`;
                    }
                    return part;
                }).join('');
            }).join('<br>');
        }

        // 播放翻转音效
        function playFlipSound() {
            if (!soundEnabled) return;
            const oscillator = audioContext.createOscillator();
            const gainNode = audioContext.createGain();
            
            oscillator.connect(gainNode);
            gainNode.connect(audioContext.destination);
            
            oscillator.frequency.value = 800;
            oscillator.type = 'sine';
            
            gainNode.gain.setValueAtTime(0.3, audioContext.currentTime);
            gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.1);
            
            oscillator.start(audioContext.currentTime);
            oscillator.stop(audioContext.currentTime + 0.1);
        }

        // 播放切换卡片音效
        function playNavigateSound() {
            if (!soundEnabled) return;
            const oscillator = audioContext.createOscillator();
            const gainNode = audioContext.createGain();
            
            oscillator.connect(gainNode);
            gainNode.connect(audioContext.destination);
            
            oscillator.frequency.value = 600;
            oscillator.type = 'triangle';
            
            gainNode.gain.setValueAtTime(0.2, audioContext.currentTime);
            gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.15);
            
            oscillator.start(audioContext.currentTime);
            oscillator.stop(audioContext.currentTime + 0.15);
        }

        // 播放完成音效
        function playSuccessSound() {
            if (!soundEnabled) return;
            const oscillator = audioContext.createOscillator();
            const gainNode = audioContext.createGain();
            
            oscillator.connect(gainNode);
            gainNode.connect(audioContext.destination);
            
            oscillator.frequency.setValueAtTime(523, audioContext.currentTime);
            oscillator.frequency.setValueAtTime(659, audioContext.currentTime + 0.1);
            oscillator.frequency.setValueAtTime(784, audioContext.currentTime + 0.2);
            oscillator.type = 'sine';
            
            gainNode.gain.setValueAtTime(0.3, audioContext.currentTime);
            gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.3);
            
            oscillator.start(audioContext.currentTime);
            oscillator.stop(audioContext.currentTime + 0.3);
        }

        // 切换声音
        function toggleSound() {
            soundEnabled = !soundEnabled;
            document.getElementById('sound-icon').textContent = soundEnabled ? '🔊' : '🔇';
            document.getElementById('sound-text').textContent = soundEnabled ? '声音开' : '声音关';
            
            if (soundEnabled) {
                playNavigateSound();
            }
        }

        function setMode(mode) {
            currentMode = mode;
            playNavigateSound();
            
            document.getElementById('mode-single').className = mode === 'single' 
                ? 'px-6 py-3 bg-indigo-500 text-white rounded-lg hover:bg-indigo-600 transition'
                : 'px-6 py-3 bg-gray-400 text-white rounded-lg hover:bg-gray-500 transition';
            document.getElementById('mode-grid').className = mode === 'grid' 
                ? 'px-6 py-3 bg-indigo-500 text-white rounded-lg hover:bg-indigo-600 transition'
                : 'px-6 py-3 bg-gray-400 text-white rounded-lg hover:bg-gray-500 transition';
            
            if (mode === 'single') {
                document.getElementById('single-mode').classList.remove('hidden');
                document.getElementById('grid-mode').classList.add('hidden');
                updateSingleCard();
            } else {
                document.getElementById('single-mode').classList.add('hidden');
                document.getElementById('grid-mode').classList.remove('hidden');
                renderCards();
            }
        }

        function setCategory(category) {
            currentCategory = category;
            currentIndex = 0;
            playNavigateSound();
            
            if (currentCategory === 'all') {
                currentCards = Object.values(phonicsData).flat();
            } else {
                currentCards = phonicsData[currentCategory] || [];
            }
            
            if (currentMode === 'single') {
                updateSingleCard();
            } else {
                flippedCards.clear();
                renderCards();
            }
            updateProgress();
        }

        function updateSingleCard() {
            if (currentCategory === 'all') {
                currentCards = Object.values(phonicsData).flat();
            } else {
                currentCards = phonicsData[currentCategory] || [];
            }

            if (currentCards.length === 0) return;

            const card = currentCards[currentIndex];
            document.getElementById('single-front').textContent = card.front;
            document.getElementById('single-title').textContent = card.title;
            document.getElementById('single-back').innerHTML = makeWordsHoverable(card.back);
            
            // 重置翻转状态
            document.getElementById('single-card').classList.remove('flipped');
            
            // 更新按钮状态
            document.getElementById('prev-btn').disabled = currentIndex === 0;
            document.getElementById('next-btn').disabled = currentIndex === currentCards.length - 1;
            
            updateProgress();
        }

        function toggleCard(event) {
            // 如果点击的是单词，不翻转卡片
            if (event.target.classList.contains('word-hoverable')) {
                return;
            }
            
            const card = document.getElementById('single-card');
            card.classList.toggle('flipped');
            playFlipSound();
            
            const cardId = `${currentCategory}-${currentIndex}`;
            if (card.classList.contains('flipped')) {
                flippedCards.add(cardId);
            }
            updateProgress();
        }

        function nextCard() {
            if (currentIndex < currentCards.length - 1) {
                currentIndex++;
                playNavigateSound();
                updateSingleCard();
            }
        }

        function prevCard() {
            if (currentIndex > 0) {
                currentIndex--;
                playNavigateSound();
                updateSingleCard();
            }
        }

        function randomCard() {
            currentIndex = Math.floor(Math.random() * currentCards.length);
            playNavigateSound();
            updateSingleCard();
        }

        function renderCards() {
            const container = document.getElementById('card-container');
            container.innerHTML = '';
            
            currentCards.forEach((card, index) => {
                const cardDiv = document.createElement('div');
                cardDiv.className = 'card h-64';
                cardDiv.innerHTML = `
                    <div class="card-inner">
                        <div class="card-front">
                            <div class="text-6xl font-bold mb-4">${card.front}</div>
                            <div class="text-sm opacity-75">点击查看详情</div>
                        </div>
                        <div class="card-back">
                            <div class="text-2xl font-bold mb-4">${card.title}</div>
                            <div class="text-base text-center leading-relaxed">${makeWordsHoverable(card.back)}</div>
                        </div>
                    </div>
                `;
                cardDiv.onclick = (e) => {
                    // 如果点击的是单词，不翻转卡片
                    if (e.target.classList.contains('word-hoverable')) {
                        return;
                    }
                    flipCard(cardDiv, index);
                };
                container.appendChild(cardDiv);
            });

            document.getElementById('total').textContent = currentCards.length;
        }

        function flipCard(cardDiv, index) {
            cardDiv.classList.toggle('flipped');
            playFlipSound();
            
            const cardId = `${currentCategory}-${index}`;
            
            if (cardDiv.classList.contains('flipped')) {
                flippedCards.add(cardId);
            } else {
                flippedCards.delete(cardId);
            }
            
            updateProgress();
        }

        function updateProgress() {
            const total = currentCards.length;
            const current = flippedCards.size;
            const percentage = total > 0 ? (current / total * 100) : 0;
            
            document.getElementById('current').textContent = current;
            document.getElementById('total').textContent = total;
            document.getElementById('progress').style.width = percentage + '%';
            
            // 当完成所有卡片时播放成功音效
            if (current === total && total > 0) {
                playSuccessSound();
            }
        }

        function resetProgress() {
            flippedCards.clear();
            currentIndex = 0;
            playNavigateSound();
            
            document.querySelectorAll('.card').forEach(card => {
                card.classList.remove('flipped');
            });
            if (currentMode === 'single') {
                updateSingleCard();
            }
            updateProgress();
        }

        // 初始化
        setCategory('all');
        setMode('single');
    </script>
</body>
</html>
