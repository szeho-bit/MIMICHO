<八個方向大冒險 html
<html lang="zh-HK">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>八個方向 RPG 大冒險</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Chakra+Petch:wght@600;700&family=Noto+Sans+HK:wght@400;700;900&display=swap" rel="stylesheet">
    <style>
        /* 霓虹與像素遊戲風格設定 */
        body {
            font-family: 'Noto Sans HK', sans-serif;
            background-color: #050510;
            color: #fff;
            overflow: hidden;
            user-select: none;
        }

        .pixel-font {
            font-family: 'Chakra Petch', 'Noto Sans HK', sans-serif;
        }

        /* 發光特效 */
        .neon-text-green {
            color: #4ade80;
            text-shadow: 0 0 5px #4ade80, 0 0 15px #4ade80, 0 0 30px #22c55e;
        }
        
        .neon-text-blue {
            color: #38bdf8;
            text-shadow: 0 0 5px #38bdf8, 0 0 15px #38bdf8;
        }

        .neon-border-purple {
            border: 4px solid #a855f7;
            box-shadow: 0 0 15px #a855f7, inset 0 0 15px #a855f7;
        }

        /* 遊戲按鈕 */
        .game-btn {
            background: linear-gradient(180deg, #1e1b4b 0%, #0f172a 100%);
            border: 3px solid #38bdf8;
            box-shadow: 0 6px 0 #0284c7, 0 0 15px rgba(56, 189, 248, 0.5);
            transition: all 0.1s;
            position: relative;
            overflow: hidden;
        }
        
        .game-btn:hover {
            background: linear-gradient(180deg, #312e81 0%, #1e293b 100%);
            box-shadow: 0 6px 0 #0284c7, 0 0 25px #38bdf8;
            transform: translateY(-2px);
        }

        .game-btn:active {
            box-shadow: 0 0 0 #0284c7;
            transform: translateY(6px);
        }

        .game-btn.correct {
            border-color: #4ade80;
            box-shadow: 0 6px 0 #16a34a, 0 0 25px #4ade80;
            background: linear-gradient(180deg, #064e3b 0%, #065f46 100%);
        }

        .game-btn.wrong {
            border-color: #f43f5e;
            box-shadow: 0 6px 0 #e11d48, 0 0 25px #f43f5e;
            background: linear-gradient(180deg, #881337 0%, #9f1239 100%);
        }

        /* 地圖節點與路徑 */
        .map-node {
            width: 30px;
            height: 30px;
            background-color: #1e293b;
            border: 3px solid #64748b;
            border-radius: 50%;
            position: absolute;
            transform: translate(-50%, 50%);
            z-index: 10;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: bold;
            color: #94a3b8;
        }

        .map-node.active {
            background-color: #facc15;
            border-color: #fb923c;
            color: #000;
            box-shadow: 0 0 20px #facc15;
            animation: pulse 1.5s infinite;
        }

        .map-node.completed {
            background-color: #4ade80;
            border-color: #16a34a;
            color: #000;
        }

        @keyframes pulse {
            0% { transform: translate(-50%, 50%) scale(1); }
            50% { transform: translate(-50%, 50%) scale(1.2); }
            100% { transform: translate(-50%, 50%) scale(1); }
        }

        /* 角色 */
        #player {
            position: absolute;
            font-size: 50px;
            transform: translate(-50%, 20%);
            z-index: 20;
            transition: all 1s cubic-bezier(0.34, 1.56, 0.64, 1); /* Q彈移動特效 */
            filter: drop-shadow(0 0 10px #facc15);
        }

        .moving-up {
            animation: jump 0.5s ease-in-out infinite;
        }

        @keyframes jump {
            0%, 100% { margin-bottom: 0; }
            50% { margin-bottom: 15px; }
        }

        /* 畫面震動 (錯誤時) */
        .shake {
            animation: shake 0.4s cubic-bezier(.36,.07,.19,.97) both;
        }

        @keyframes shake {
            10%, 90% { transform: translate3d(-2px, 0, 0); }
            20%, 80% { transform: translate3d(4px, 0, 0); }
            30%, 50%, 70% { transform: translate3d(-6px, 0, 0); }
            40%, 60% { transform: translate3d(6px, 0, 0); }
        }

        /* 掃描線特效 (Retro Scanline) */
        .scanlines {
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: linear-gradient(rgba(18, 16, 16, 0) 50%, rgba(0, 0, 0, 0.25) 50%), linear-gradient(90deg, rgba(255, 0, 0, 0.06), rgba(0, 255, 0, 0.02), rgba(0, 0, 255, 0.06));
            background-size: 100% 4px, 3px 100%;
            z-index: 999;
            pointer-events: none;
        }
        
        /* 碎紙花畫布 */
        #confetti-canvas {
            position: fixed;
            top: 0; left: 0;
            width: 100%; height: 100%;
            pointer-events: none;
            z-index: 1000;
        }
    </style>
</head>
<body class="flex flex-col h-screen relative">
    <div class="scanlines"></div>
    <canvas id="confetti-canvas"></canvas>

    <!-- 頂部 HUD (平視顯示器) -->
    <header class="flex justify-between items-center p-6 z-50">
        <!-- 左上角：發光指南針 -->
        <div class="flex items-center gap-3">
            <div class="text-6xl animate-bounce">🧭</div>
            <div class="pixel-font text-5xl font-black neon-text-green tracking-widest">北方 ↑</div>
        </div>
        
        <!-- 右上角：坐標進度 (更新為8個方向的圖示) -->
        <div class="bg-gray-900 border-2 border-blue-400 p-2 rounded-xl shadow-[0_0_15px_#38bdf8] flex items-center gap-3">
            <div class="pixel-font text-xl text-blue-200 px-2">方向坐標收集</div>
            <div class="relative w-16 h-16">
                <svg viewBox="0 0 100 100" class="w-full h-full" id="compass-icon">
                    <circle cx="50" cy="50" r="35" fill="#1e293b" stroke="#38bdf8" stroke-width="2" stroke-dasharray="4,4">
                        <animateTransform attributeName="transform" type="rotate" from="0 50 50" to="360 50 50" dur="20s" repeatCount="indefinite"/>
                    </circle>
                    <!-- 8個方向的箭頭 (會根據進度亮起) -->
                    <g id="dir-N"  transform="translate(50,15)"><polygon points="0,-10 6,0 -6,0" fill="#475569"/></g>
                    <g id="dir-NE" transform="translate(75,25) rotate(45)"><polygon points="0,-10 6,0 -6,0" fill="#475569"/></g>
                    <g id="dir-E"  transform="translate(85,50) rotate(90)"><polygon points="0,-10 6,0 -6,0" fill="#475569"/></g>
                    <g id="dir-SE" transform="translate(75,75) rotate(135)"><polygon points="0,-10 6,0 -6,0" fill="#475569"/></g>
                    <g id="dir-S"  transform="translate(50,85) rotate(180)"><polygon points="0,-10 6,0 -6,0" fill="#475569"/></g>
                    <g id="dir-SW" transform="translate(25,75) rotate(225)"><polygon points="0,-10 6,0 -6,0" fill="#475569"/></g>
                    <g id="dir-W"  transform="translate(15,50) rotate(270)"><polygon points="0,-10 6,0 -6,0" fill="#475569"/></g>
                    <g id="dir-NW" transform="translate(25,25) rotate(315)"><polygon points="0,-10 6,0 -6,0" fill="#475569"/></g>
                    <circle cx="50" cy="50" r="14" fill="#0f172a" stroke="#facc15" stroke-width="2"/>
                    <text x="50" y="56" font-size="18" fill="#facc15" text-anchor="middle" font-weight="bold" class="pixel-font" id="level-display">1</text>
                </svg>
            </div>
        </div>
    </header>

    <!-- 地圖與角色區域 (RPG 畫面) -->
    <main class="flex-grow relative mx-6 mb-4 mt-2 neon-border-purple rounded-2xl bg-gray-900 overflow-hidden" id="map-container">
        <!-- 網格背景 -->
        <div class="absolute inset-0 opacity-20" style="background-image: radial-gradient(#38bdf8 1px, transparent 1px); background-size: 40px 40px;"></div>
        
        <!-- 連接路徑 (SVG) -->
        <svg class="absolute inset-0 w-full h-full" id="path-svg">
            <path id="route-path" fill="none" stroke="#475569" stroke-width="6" stroke-dasharray="10,10" />
        </svg>

        <!-- 城堡 (終點) -->
        <div id="castle" class="absolute text-7xl z-10" style="bottom: 85%; left: 85%; transform: translate(-50%, 50%); filter: drop-shadow(0 0 20px #a855f7);">🏰</div>
        <div class="absolute text-xl font-bold text-purple-300 pixel-font" style="bottom: 95%; left: 85%; transform: translate(-50%, 0);">遊戲大師城堡</div>

        <!-- 起點 -->
        <div class="absolute text-xl font-bold text-gray-400 pixel-font" style="bottom: 2%; left: 10%; transform: translate(-50%, 0);">新手村</div>

        <!-- 角色 -->
        <div id="player">🧑‍🎓</div>
        
        <!-- 節點由 JS 自動生成 -->
    </main>

    <!-- 底部問答區域 -->
    <footer class="bg-gray-950 border-t-4 border-pink-500 p-4 z-50 min-h-[380px] flex flex-col justify-center relative shadow-[0_-10px_20px_rgba(236,72,153,0.2)]">
        <!-- 題目與動畫 -->
        <div class="text-center mb-4 flex flex-col items-center w-full">
            <div id="question-visual-container" class="hidden mb-3 w-full max-w-lg h-32 rounded-xl border-2 border-blue-500 overflow-hidden shadow-[0_0_15px_rgba(56,189,248,0.3)] bg-gray-800">
                <!-- 動畫 SVG 會由 JavaScript 注入在這裡 -->
            </div>
            <h2 id="question-text" class="text-2xl font-bold text-white pixel-font leading-snug">載入中...</h2>
        </div>

        <!-- 選項按鈕 -->
        <div id="options-container" class="grid grid-cols-2 gap-3 w-full max-w-4xl mx-auto">
            <!-- Buttons generated by JS -->
        </div>

        <!-- 回饋訊息 (正確/錯誤) -->
        <div id="feedback-panel" class="absolute inset-0 bg-gray-950/95 flex flex-col items-center justify-center hidden z-20">
            <h3 id="feedback-title" class="text-5xl font-black mb-4 pixel-font"></h3>
            <p id="feedback-desc" class="text-2xl text-gray-300 mb-8"></p>
            <button id="next-btn" class="px-8 py-4 bg-yellow-500 hover:bg-yellow-400 text-black font-bold rounded-full text-2xl pixel-font shadow-[0_0_20px_#facc15] transition-transform hover:scale-110">繼續前進</button>
        </div>
    </footer>

    <script>
        // 遊戲設定與資料
        const TOTAL_LEVELS = 8;
        let currentLevel = 1;
        let isAnimating = false;

        // 地圖節點坐標 (相對於 map-container 的 bottom 和 left 百分比)
        const pathPoints = [
            { level: 1, bottom: 10, left: 10 },  // 新手村
            { level: 2, bottom: 25, left: 30 },
            { level: 3, bottom: 35, left: 15 },
            { level: 4, bottom: 50, left: 40 },
            { level: 5, bottom: 60, left: 65 },
            { level: 6, bottom: 75, left: 50 },
            { level: 7, bottom: 80, left: 70 },
            { level: 8, bottom: 85, left: 85 }   // 城堡
        ];

        // 題目資料庫 (八個方向：由淺入深，加入生活情境)
        const questions = [
            {
                q: "【關卡 1：新手出發】太陽每天早上都會從哪一個方向升起？",
                options: ["A. 遊戲村", "B. 東方", "C. 西方", "D. 北方"],
                answer: 1,
                correctMsg: "正確！向著日出的方向出發，往北方前進！",
                wrongMsg: "再試一次！提示：太陽從「東」邊升起喔。"
            },
            {
                q: "【關卡 2：裝備指南針】指南針上，紅色的指針通常會永遠指向哪一個方向？",
                options: ["A. 南方", "B. 終點方向", "C. 北方", "D. 東方"],
                answer: 2,
                correctMsg: "太棒了！指南針的紅針指著北方，繼續向北走！",
                wrongMsg: "哎呀走錯了！看看畫面左上角的發光提示？"
            },
            {
                q: "【關卡 3：混合方向解鎖】在「東」和「北」之間的方向，我們稱為什麼？",
                options: ["A. 東北方", "B. 東南方", "C. 西北方", "D. 西南方"],
                answer: 0,
                correctMsg: "邏輯完全正確！你已經掌握了進階方向！",
                wrongMsg: "邏輯錯誤！記得公式：先說（東/西），再說（南/北）。"
            },
            {
                q: "【關卡 4：系統語言辨識】在國際地圖上，英文縮寫「SW」代表哪一個方向？",
                options: ["A. 東南 (Southeast)", "B. 東北 (Northeast)", "C. 西北 (Northwest)", "D. 西南 (Southwest)"],
                answer: 3,
                correctMsg: "解碼成功！S代表South，W代表West，往北方繼續前進！",
                wrongMsg: "代碼錯誤！S 是 South(南)，W 是 West(西) 喔。"
            },
            {
                q: "【關卡 5：情境考驗 1】強烈颱風正從香港的「東南方」吹來，為了躲避強風，我們應該站在大樓的哪一面最安全？",
                options: ["A. 東南面", "B. 正南面", "C. 西北面", "D. 東北面"],
                answer: 2,
                correctMsg: "危機解除！躲在風向的「相反」方向最安全！",
                wrongMsg: "警告！你迎向颱風了！應該躲在風向的「相反」方向才對。",
                visual: `<svg viewBox="0 0 300 120" class="w-full h-full">
                            <defs>
                                <marker id="arrow" viewBox="0 0 10 10" refX="5" refY="5" markerWidth="5" markerHeight="5" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="#38bdf8"/></marker>
                            </defs>
                            <rect x="120" y="30" width="60" height="80" fill="#334155" stroke="#94a3b8" stroke-width="3"/>
                            <text x="150" y="75" fill="white" font-size="18" text-anchor="middle" class="pixel-font">大樓</text>
                            <path d="M 260 110 L 190 60" stroke="#38bdf8" stroke-width="4" stroke-dasharray="10,10" fill="none" marker-end="url(#arrow)">
                                <animate attributeName="stroke-dashoffset" from="40" to="0" dur="0.5s" repeatCount="indefinite"/>
                            </path>
                            <path d="M 230 120 L 160 70" stroke="#38bdf8" stroke-width="4" stroke-dasharray="10,10" fill="none" marker-end="url(#arrow)">
                                <animate attributeName="stroke-dashoffset" from="40" to="0" dur="0.5s" repeatCount="indefinite"/>
                            </path>
                            <text x="240" y="90" fill="#38bdf8" font-size="16" font-weight="bold" class="pixel-font">東南風 🌪️</text>
                            <circle cx="100" cy="40" r="20" fill="rgba(74, 222, 128, 0.2)" stroke="#4ade80" stroke-width="2" stroke-dasharray="2,2"/>
                            <text x="100" y="45" fill="#4ade80" font-size="14" text-anchor="middle" font-weight="bold">安全區?</text>
                        </svg>`
            },
            {
                q: "【關卡 6：情境考驗 2】你站在操場面向「北方」，這時你向「右後方」轉身，你現在面向哪裡？",
                options: ["A. 東南方", "B. 西南方", "C. 東北方", "D. 西北方"],
                answer: 0,
                correctMsg: "空間感超強！面向北時，右邊是東，後面是南，所以右後方是東南！",
                wrongMsg: "轉暈了嗎？先想想面向北時，右邊是什麼？後面是什麼？",
                visual: `<svg viewBox="0 0 300 120" class="w-full h-full">
                            <defs>
                                <marker id="arrow-yellow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" fill="#facc15"/></marker>
                            </defs>
                            <circle cx="150" cy="60" r="20" fill="#3b82f6"/>
                            <text x="150" y="65" fill="white" font-size="14" text-anchor="middle">你</text>
                            <g>
                                <path d="M 150 60 L 150 15" stroke="#facc15" stroke-width="4" marker-end="url(#arrow-yellow)"/>
                                <animateTransform attributeName="transform" type="rotate" values="0 150 60; 135 150 60; 135 150 60" keyTimes="0; 0.5; 1" dur="2s" repeatCount="indefinite"/>
                            </g>
                            <text x="150" y="12" fill="#4ade80" font-size="14" text-anchor="middle" font-weight="bold">北</text>
                            <path d="M 150 30 A 30 30 0 0 1 171 81" fill="none" stroke="#64748b" stroke-width="2" stroke-dasharray="4,4"/>
                            <text x="195" y="90" fill="#f43f5e" font-size="14" font-weight="bold">右後方?</text>
                        </svg>`
            },
            {
                q: "【關卡 7：地圖實戰】地圖上通常「上方」代表北。如果「圖書館」在「學校」的左下方，代表圖書館在學校的哪一個方向？",
                options: ["A. 東南方", "B. 西南方", "C. 西北方", "D. 東北方"],
                answer: 1,
                correctMsg: "地圖大師！左邊是西，下方是南，所以是西南方！即將到達終點！",
                wrongMsg: "看錯地圖囉！地圖的左邊是西，下方是南喔。",
                visual: `<svg viewBox="0 0 300 120" class="w-full h-full">
                            <defs>
                                <marker id="arrow-pink" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" fill="#f472b6"/></marker>
                            </defs>
                            <text x="30" y="30" fill="#4ade80" font-size="16" font-weight="bold" class="pixel-font">北 ↑</text>
                            <rect x="160" y="20" width="60" height="40" fill="#3b82f6" rx="5"/>
                            <text x="190" y="45" fill="white" font-size="16" text-anchor="middle">🏫 學校</text>
                            <rect x="80" y="70" width="70" height="40" fill="#a855f7" rx="5"/>
                            <text x="115" y="95" fill="white" font-size="16" text-anchor="middle">📚 圖書館</text>
                            <path d="M 160 40 L 120 70" stroke="#f472b6" stroke-width="3" stroke-dasharray="5,5" marker-end="url(#arrow-pink)"/>
                        </svg>`
            },
            {
                q: "【關卡 8：最終魔王關】尋寶謎題：你從寶箱位置出發，先向「北」走1步，再向「東」走1步。這時候，寶箱在你的哪一個方向？",
                options: ["A. 西北方", "B. 東南方", "C. 西南方", "D. 東北方"],
                answer: 2,
                correctMsg: "破關成功！你往東北走，所以起點就在你的西南方！",
                wrongMsg: "最後一關不能大意！你在寶箱的東北方，那寶箱在你的哪裡呢？（相反方向）",
                visual: `<svg viewBox="0 0 300 120" class="w-full h-full">
                            <defs>
                                <marker id="arrow-red" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" fill="#f43f5e"/></marker>
                                <marker id="arrow-green" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" fill="#4ade80"/></marker>
                            </defs>
                            <rect x="100" y="80" width="30" height="20" fill="#facc15" rx="3"/>
                            <text x="115" y="75" fill="#facc15" font-size="14" text-anchor="middle" font-weight="bold">寶箱</text>
                            <path d="M 115 80 L 115 40 L 175 40" stroke="#4ade80" stroke-width="4" stroke-dasharray="5,5" fill="none" marker-end="url(#arrow-green)">
                                <animate attributeName="stroke-dashoffset" from="30" to="0" dur="1s" repeatCount="indefinite"/>
                            </path>
                            <circle cx="185" cy="40" r="12" fill="#38bdf8"/>
                            <text x="185" y="44" fill="white" font-size="12" text-anchor="middle">你</text>
                            <path d="M 175 50 L 125 75" stroke="#f43f5e" stroke-width="3" marker-end="url(#arrow-red)" fill="none"/>
                            <text x="160" y="85" fill="#f43f5e" font-size="14" font-weight="bold">哪個方向?</text>
                        </svg>`
            }
        ];

        // 初始設定 DOM
        const player = document.getElementById('player');
        const levelDisplay = document.getElementById('level-display');
        const questionText = document.getElementById('question-text');
        const optionsContainer = document.getElementById('options-container');
        const mapContainer = document.getElementById('map-container');
        const pathSvg = document.getElementById('route-path');
        const feedbackPanel = document.getElementById('feedback-panel');
        const feedbackTitle = document.getElementById('feedback-title');
        const feedbackDesc = document.getElementById('feedback-desc');
        const nextBtn = document.getElementById('next-btn');

        // 建立地圖節點與繪製路徑
        function initMap() {
            let pathD = "";
            pathPoints.forEach((pt, index) => {
                // 產生節點 HTML
                const node = document.createElement('div');
                node.className = `map-node pixel-font text-sm ${index === 0 ? 'active' : ''}`;
                node.id = `node-${pt.level}`;
                node.style.bottom = `${pt.bottom}%`;
                node.style.left = `${pt.left}%`;
                node.innerText = pt.level;
                mapContainer.appendChild(node);

                // SVG 路徑計算 (需要將 % 轉為實際坐標或使用 relative svg)
                // 為了簡化，SVG path 在 resize 時可能不準確，這裡採用簡易相對百分比畫法
                // SVG viewBox 預設為 100x100 方便對應百分比
            });

            // 設定 SVG viewBox 配合百分比
            document.getElementById('path-svg').setAttribute('viewBox', '0 0 100 100');
            document.getElementById('path-svg').setAttribute('preserveAspectRatio', 'none');
            
            // 構建 SVG 路徑 d 屬性 (注意 SVG 的 Y 軸是從上到下，bottom % 需要轉換為 top %)
            pathPoints.forEach((pt, index) => {
                const x = pt.left;
                const y = 100 - pt.bottom;
                if (index === 0) {
                    pathD += `M ${x} ${y} `;
                } else {
                    pathD += `L ${x} ${y} `;
                }
            });
            pathSvg.setAttribute('d', pathD);

            // 設定玩家初始位置
            updatePlayerPosition(1);
        }

        function updatePlayerPosition(level) {
            const pt = pathPoints[level - 1];
            player.style.bottom = `${pt.bottom}%`;
            player.style.left = `${pt.left}%`;
            
            // 更新節點樣式
            document.querySelectorAll('.map-node').forEach(node => {
                const nodeLevel = parseInt(node.id.split('-')[1]);
                if (nodeLevel < level) {
                    node.className = 'map-node pixel-font text-sm completed';
                    node.innerText = '✓';
                } else if (nodeLevel === level) {
                    node.className = 'map-node pixel-font text-sm active';
                    node.innerText = nodeLevel;
                } else {
                    node.className = 'map-node pixel-font text-sm';
                    node.innerText = nodeLevel;
                }
            });
        }

        // 載入題目
        function loadQuestion() {
            if (currentLevel > TOTAL_LEVELS) {
                gameWin();
                return;
            }

            const qData = questions[currentLevel - 1];
            levelDisplay.innerText = currentLevel;
            questionText.innerHTML = qData.q.replace(/「(.*?)」/g, '<span class="text-yellow-400">「$1」</span>'); // 關鍵字高亮
            
            // 更新方向坐標圖示 (點亮對應進度的指南針箭頭)
            const dirs = ['N', 'NE', 'E', 'SE', 'S', 'SW', 'W', 'NW'];
            dirs.forEach((dir, index) => {
                const arrow = document.querySelector(`#dir-${dir} polygon`);
                if (arrow) {
                    if (index < currentLevel) {
                        arrow.setAttribute('fill', '#facc15');
                        arrow.style.filter = 'drop-shadow(0 0 5px #facc15)';
                    } else {
                        arrow.setAttribute('fill', '#475569');
                        arrow.style.filter = 'none';
                    }
                }
            });

            // 更新視覺動畫區域 (若該題有 visual 屬性則顯示)
            const visualContainer = document.getElementById('question-visual-container');
            if (qData.visual) {
                visualContainer.innerHTML = qData.visual;
                visualContainer.classList.remove('hidden');
            } else {
                visualContainer.innerHTML = '';
                visualContainer.classList.add('hidden');
            }
            
            optionsContainer.innerHTML = '';
            qData.options.forEach((opt, index) => {
                const btn = document.createElement('button');
                // 稍微縮小按鈕大小以騰出空間給圖片
                btn.className = 'game-btn text-white text-xl font-bold py-3 px-5 rounded-xl pixel-font text-left w-full';
                btn.innerText = opt;
                btn.onclick = () => checkAnswer(index, btn);
                optionsContainer.appendChild(btn);
            });
            
            feedbackPanel.classList.add('hidden');
        }

        // 檢查答案
        function checkAnswer(selectedIndex, btnElement) {
            if (isAnimating) return;
            isAnimating = true;

            const qData = questions[currentLevel - 1];
            const isCorrect = selectedIndex === qData.answer;

            // 停用所有按鈕
            document.querySelectorAll('.game-btn').forEach(b => b.disabled = true);

            if (isCorrect) {
                // 答對處理
                btnElement.classList.add('correct');
                
                setTimeout(() => {
                    feedbackTitle.innerText = "MISSION SUCCESS";
                    feedbackTitle.className = "text-6xl font-black mb-4 pixel-font neon-text-green";
                    feedbackDesc.innerText = qData.correctMsg;
                    nextBtn.innerText = currentLevel === TOTAL_LEVELS ? "領取大師證書" : "進發下一關";
                    nextBtn.onclick = () => {
                        currentLevel++;
                        player.classList.add('moving-up');
                        updatePlayerPosition(currentLevel > TOTAL_LEVELS ? TOTAL_LEVELS : currentLevel);
                        
                        setTimeout(() => {
                            player.classList.remove('moving-up');
                            loadQuestion();
                            isAnimating = false;
                        }, 1000);
                    };
                    feedbackPanel.classList.remove('hidden');
                }, 500);

            } else {
                // 答錯處理
                btnElement.classList.add('wrong');
                document.body.classList.add('shake');
                
                setTimeout(() => {
                    document.body.classList.remove('shake');
                    feedbackTitle.innerText = "SYSTEM ERROR";
                    feedbackTitle.className = "text-6xl font-black mb-4 pixel-font text-red-500 drop-shadow-[0_0_15px_#f43f5e]";
                    feedbackDesc.innerText = qData.wrongMsg;
                    nextBtn.innerText = "重新定位";
                    nextBtn.onclick = () => {
                        btnElement.classList.remove('wrong');
                        document.querySelectorAll('.game-btn').forEach(b => b.disabled = false);
                        feedbackPanel.classList.add('hidden');
                        isAnimating = false;
                    };
                    feedbackPanel.classList.remove('hidden');
                }, 500);
            }
        }

        // 遊戲通關
        function gameWin() {
            document.getElementById('question-visual-container').classList.add('hidden');
            questionText.innerText = "🎊 恭喜！你已到達遊戲大師城堡！";
            optionsContainer.innerHTML = '<div class="col-span-2 text-center text-xl text-yellow-300 mt-4 pixel-font">你已經完全精通了八個方向的知識，無論是地圖還是生活情境都難不倒你！</div>';
            player.innerText = "🦸‍♂️";
            player.style.transform = "translate(-50%, 20%) scale(1.5)";
            fireConfetti();
        }

        // 碎紙花特效 (簡易版)
        function fireConfetti() {
            const canvas = document.getElementById('confetti-canvas');
            const ctx = canvas.getContext('2d');
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;

            const particles = [];
            const colors = ['#facc15', '#4ade80', '#38bdf8', '#f472b6', '#a855f7'];

            for (let i = 0; i < 150; i++) {
                particles.push({
                    x: Math.random() * canvas.width,
                    y: Math.random() * canvas.height - canvas.height,
                    size: Math.random() * 10 + 5,
                    color: colors[Math.floor(Math.random() * colors.length)],
                    speedY: Math.random() * 3 + 2,
                    speedX: Math.random() * 2 - 1,
                    rotation: Math.random() * 360,
                    rotationSpeed: Math.random() * 10 - 5
                });
            }

            function animateConfetti() {
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                particles.forEach(p => {
                    ctx.save();
                    ctx.translate(p.x, p.y);
                    ctx.rotate(p.rotation * Math.PI / 180);
                    ctx.fillStyle = p.color;
                    ctx.fillRect(-p.size/2, -p.size/2, p.size, p.size);
                    ctx.restore();

                    p.y += p.speedY;
                    p.x += p.speedX;
                    p.rotation += p.rotationSpeed;

                    if (p.y > canvas.height) {
                        p.y = -10;
                        p.x = Math.random() * canvas.width;
                    }
                });
                requestAnimationFrame(animateConfetti);
            }
            animateConfetti();
        }

        // 啟動遊戲
        window.onload = () => {
            initMap();
            loadQuestion();
        };

        // 視窗大小改變時重繪地圖避免錯位
        window.onresize = () => {
            const canvas = document.getElementById('confetti-canvas');
            if(canvas.width > 0) {
                canvas.width = window.innerWidth;
                canvas.height = window.innerHeight;
            }
        };
    </script>
</body>
</html>
