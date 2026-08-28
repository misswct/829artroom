[Uploading code_artifact (6).html…]()
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>加入埔浸小新校園 - 小朋友創作動態展示系統</title>
    <!-- Tailwind CSS for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Fredoka:wght@400;600;700&family=Noto+Sans+TC:wght@400;500;700&display=swap" rel="stylesheet">
    <!-- PeerJS & QRCode for Direct P2P Network Synchronization -->
    <script src="https://unpkg.com/peerjs@1.5.2/dist/peerjs.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        brand: {
                            50: '#f0f9ff',
                            100: '#e0f2fe',
                            500: '#0ea5e9',
                            600: '#0284c7',
                            700: '#0369a1',
                        }
                    },
                    fontFamily: {
                        sans: ['Noto Sans TC', 'sans-serif'],
                        fun: ['Fredoka', 'Noto Sans TC', 'sans-serif']
                    }
                }
            }
        }
    </script>
    <style>
        /* Custom styles & canvas backgrounds */
        .bg-checkerboard {
            background-color: #f1f5f9;
            background-image: linear-gradient(45deg, #cbd5e1 25%, transparent 25%), linear-gradient(-45deg, #cbd5e1 25%, transparent 25%), linear-gradient(45deg, transparent 75%, #cbd5e1 75%), linear-gradient(-45deg, transparent 75%, #cbd5e1 75%);
            background-size: 20px 20px;
            background-position: 0 0, 0 10px, 10px -10px, -10px 0px;
        }

        .character-shadow {
            filter: drop-shadow(0px 8px 12px rgba(0, 0, 0, 0.25));
        }

        @keyframes bounce-gentle {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-8px); }
        }

        .animate-bounce-gentle {
            animation: bounce-gentle 2.5s infinite ease-in-out;
        }

        @keyframes character-jump {
            0% { transform: scale(1, 1) translateY(0); }
            20% { transform: scale(1.25, 0.75) translateY(0); }
            45% { transform: scale(0.85, 1.2) translateY(-45px); }
            70% { transform: scale(1.1, 0.9) translateY(-10px); }
            85% { transform: scale(0.95, 1.05) translateY(0); }
            100% { transform: scale(1, 1) translateY(0); }
        }

        .animate-jump {
            animation: character-jump 0.55s cubic-bezier(0.28, 0.84, 0.42, 1);
        }

        @keyframes float-sparkle {
            0% { opacity: 1; transform: translate(-50%, 0) scale(0.5) rotate(0deg); }
            100% { opacity: 0; transform: translate(calc(-50% + var(--dx)), -60px) scale(1.4) rotate(180deg); }
        }

        .sparkle-pop {
            position: absolute;
            pointer-events: none;
            font-size: 20px;
            animation: float-sparkle 0.8s ease-out forwards;
            z-index: 60;
        }

        /* Custom scrollbar */
        ::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f5f9;
        }
        ::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #94a3b8;
        }
    </style>
</head>
<body class="bg-slate-900 text-slate-800 font-sans min-h-screen flex flex-col overflow-x-hidden">

    <!-- Top Navigation Bar -->
    <header class="bg-slate-800 border-b border-slate-700 text-white z-30 flex-shrink-0 sticky top-0 shadow-md">
        <div class="max-w-7xl mx-auto px-4 py-3 flex items-center justify-between">
            <div class="flex items-center space-x-3">
                <div class="w-10 h-10 rounded-xl bg-gradient-to-tr from-sky-400 to-blue-600 flex items-center justify-center text-white text-xl font-bold shadow-lg">
                    <i class="fa-solid fa-graduation-cap"></i>
                </div>
                <div>
                    <h1 class="font-fun text-lg md:text-xl font-bold bg-gradient-to-r from-sky-300 via-cyan-200 to-blue-100 bg-clip-text text-transparent">
                        加入埔浸小新校園
                    </h1>
                    <p class="text-xs text-slate-400 hidden sm:block">兒童繪畫作品全螢幕互動展示系統</p>
                </div>
            </div>

            <!-- View Mode Tabs -->
            <div class="flex items-center space-x-2 bg-slate-900/60 p-1 rounded-xl border border-slate-700/60">
                <button id="btnTabDisplay" onclick="switchTab('display')" class="px-4 py-2 rounded-lg text-sm font-medium transition flex items-center space-x-2 bg-sky-500 text-slate-950 font-bold shadow-md">
                    <i class="fa-solid fa-tv"></i>
                    <span>動態展示場景</span>
                </button>
                <button id="btnTabAdmin" onclick="switchTab('admin')" class="px-4 py-2 rounded-lg text-sm font-medium transition flex items-center space-x-2 text-slate-300 hover:text-white hover:bg-slate-800">
                    <i class="fa-solid fa-sliders"></i>
                    <span>作品管理與去背上傳</span>
                </button>
            </div>

            <!-- Right Actions -->
            <div class="flex items-center space-x-3">
                <button onclick="toggleNetworkModal()" id="netStatusBtn" class="inline-flex items-center px-3 py-1.5 rounded-xl text-xs font-medium bg-sky-500/10 text-sky-400 border border-sky-500/20 hover:bg-sky-500/20 transition cursor-pointer">
                    <span id="syncDot" class="w-2 h-2 mr-1.5 rounded-full bg-slate-400"></span>
                    <span id="syncText">點對點網路同步中</span>
                </button>
                <button onclick="toggleFullScreen()" title="全螢幕播放" class="p-2 rounded-lg bg-slate-700 hover:bg-slate-600 text-white transition">
                    <i id="fullscreenIcon" class="fa-solid fa-expand text-lg"></i>
                </button>
            </div>
        </div>
    </header>

    <!-- Main Content Container -->
    <main class="flex-1 relative overflow-hidden flex flex-col">

        <!-- ================= SECTION 1: DISPLAY MODE (Campus Dynamic Stage) ================= -->
        <div id="panelDisplay" class="absolute inset-0 w-full h-full flex flex-col bg-slate-950">
            <!-- Stage Control Overlay (Top right floating tools) -->
            <div class="absolute top-4 right-4 z-20 flex items-center space-x-3 bg-slate-900/80 backdrop-blur-md p-2 rounded-2xl border border-slate-700/50 shadow-xl text-white">
                <div class="text-xs px-2 text-slate-300 font-medium">
                    <i class="fa-solid fa-user-group text-sky-400 mr-1"></i>
                    展示人數: <span id="displayCount" class="font-bold text-sky-300">0</span> 人
                </div>
                <div class="h-4 w-[1px] bg-slate-700"></div>
                <div class="text-xs px-2 text-slate-300 font-medium">
                    <i class="fa-regular fa-clock text-cyan-400 mr-1"></i>
                    更新倒數: <span id="timerCountdown" class="font-bold text-cyan-300">180</span>s
                </div>
                <div class="h-4 w-[1px] bg-slate-700"></div>
                <button onclick="triggerManualRotation()" class="px-2.5 py-1 bg-sky-500/20 hover:bg-sky-500/30 text-sky-300 text-xs rounded-lg transition border border-sky-500/30 flex items-center space-x-1">
                    <i class="fa-solid fa-rotate"></i>
                    <span>立即換下一批</span>
                </button>
            </div>

            <!-- Toast / Announcement Banner -->
            <div id="newArtworkBanner" class="hidden absolute top-4 left-1/2 -translate-x-1/2 z-30 bg-gradient-to-r from-sky-500 via-blue-500 to-indigo-500 text-white font-bold px-6 py-2.5 rounded-full shadow-2xl flex items-center space-x-3 border border-white/30 animate-bounce">
                <i class="fa-solid fa-wand-magic-sparkles text-xl text-sky-200"></i>
                <span id="bannerText">新作品剛剛加入了校園！</span>
            </div>

            <!-- Interactive Campus Playground Canvas Layer -->
            <div id="campusContainer" class="relative w-full h-full overflow-hidden select-none">
                <!-- Background Campus Image -->
                <div id="campusBgImage" class="absolute inset-0 bg-cover bg-center transition-all duration-700" style="background-image: url('https://images.unsplash.com/photo-1580582932707-520aed937b7b?auto=format&fit=crop&w=2000&q=80');">
                    <!-- Subtle overlay gradient for better contrast -->
                    <div class="absolute inset-0 bg-gradient-to-t from-black/40 via-transparent to-black/20"></div>
                </div>

                <!-- Floating Canvas Characters Container -->
                <div id="charactersStage" class="absolute inset-0 pointer-events-none"></div>

                <!-- Empty State Notice if no characters uploaded yet -->
                <div id="emptyDisplayState" class="hidden absolute inset-0 flex flex-col items-center justify-center text-center p-6 bg-slate-900/60 backdrop-blur-sm z-10">
                    <div class="w-20 h-20 rounded-full bg-slate-800 flex items-center justify-center text-slate-400 text-3xl mb-4 shadow-inner border border-slate-700">
                        <i class="fa-solid fa-palette"></i>
                    </div>
                    <h2 class="text-2xl font-bold text-white mb-2">校園舞台整備中</h2>
                    <p class="text-slate-300 max-w-md text-sm mb-6">目前還沒有作品或未啟用作品。請切換至「作品管理與去背上傳」頁面，拍攝並上傳小朋友的創作！</p>
                    <button onclick="switchTab('admin')" class="px-5 py-2.5 bg-sky-500 hover:bg-sky-600 text-slate-950 font-bold rounded-xl shadow-lg transition">
                        前往管理與上傳作品
                    </button>
                </div>
            </div>
        </div>

        <!-- ================= SECTION 2: ADMIN & UPLOAD CENTER ================= -->
        <div id="panelAdmin" class="hidden flex-1 w-full bg-slate-900 overflow-y-auto p-4 md:p-8">
            <div class="max-w-7xl mx-auto space-y-8">
                
                <!-- Section Header -->
                <div class="flex flex-col md:flex-row md:items-center justify-between gap-4 pb-6 border-b border-slate-800">
                    <div>
                        <h2 class="text-2xl md:text-3xl font-bold text-white font-fun">作品管理與智慧去背</h2>
                        <p class="text-slate-400 text-sm mt-1">拍照上傳小朋友的作品，去除背景並控制展示設定。</p>
                    </div>
                    
                    <!-- Quick Global Settings -->
                    <div class="flex flex-wrap items-center gap-3 bg-slate-800/80 p-3 rounded-2xl border border-slate-700">
                        <div class="flex items-center space-x-2 text-xs text-slate-300">
                            <label for="settingMinCount" class="font-medium text-slate-300">每批顯示數量:</label>
                            <select id="settingMinCount" onchange="updateSystemSettings()" class="bg-slate-900 border border-slate-700 text-white rounded-lg px-2 py-1 focus:outline-none focus:border-sky-500">
                                <option value="10" selected>最少 10 個</option>
                                <option value="15">最少 15 個</option>
                                <option value="20">最少 20 個</option>
                            </select>
                        </div>
                        <div class="h-4 w-[1px] bg-slate-700"></div>
                        <button onclick="toggleNetworkModal()" class="px-2.5 py-1 bg-sky-500/20 hover:bg-sky-500/30 text-sky-300 text-xs rounded-lg transition border border-sky-500/30 flex items-center space-x-1" title="裝置配對與房間連線">
                            <i class="fa-solid fa-network-wired text-sky-400"></i>
                            <span>跨裝置配對連線</span>
                        </button>
                    </div>
                </div>

                <!-- Two Column Layout: Left Upload/Chroma Remover, Right Artwork Gallery -->
                <div class="grid grid-cols-1 lg:grid-cols-12 gap-8">
                    
                    <!-- LEFT: Upload & Background Removal Workspace (5 cols) -->
                    <div class="lg:col-span-5 bg-slate-800/60 rounded-3xl p-6 border border-slate-700/60 flex flex-col space-y-5 shadow-xl">
                        <div class="flex items-center justify-between">
                            <h3 class="text-lg font-bold text-white flex items-center space-x-2">
                                <span class="w-8 h-8 rounded-lg bg-sky-500/20 text-sky-400 flex items-center justify-center text-sm">1</span>
                                <span>上傳與去背處理</span>
                            </h3>
                            <button onclick="resetUploadForm()" class="text-xs text-slate-400 hover:text-white transition">重置</button>
                        </div>

                        <!-- Image Input Dropzone -->
                        <div id="dropzone" onclick="triggerFileSelect()" class="border-2 border-dashed border-slate-600 hover:border-sky-500 transition rounded-2xl p-6 flex flex-col items-center justify-center text-center cursor-pointer bg-slate-900/40 relative">
                            <input type="file" id="imageInput" accept="image/*" class="hidden" onchange="handleFileSelect(event)">
                            <div id="uploadPlaceholder" class="space-y-3 pointer-events-none">
                                <div class="w-14 h-14 rounded-2xl bg-sky-500/10 text-sky-400 flex items-center justify-center text-2xl mx-auto">
                                    <i class="fa-solid fa-camera"></i>
                                </div>
                                <div class="text-slate-200 font-medium text-sm">點擊上傳或拍照作品</div>
                                <p class="text-xs text-slate-400">支援拖放圖片或拍照，建議以白色畫紙為背景</p>
                            </div>
                        </div>

                        <!-- Background Removal Studio (Hidden until image selected) -->
                        <div id="editorStudio" class="hidden space-y-4">
                            <!-- Canvas Editor Container -->
                            <div class="relative bg-checkerboard rounded-2xl overflow-hidden border border-slate-700 flex items-center justify-center min-h-[260px] max-h-[380px]">
                                <canvas id="processCanvas" class="max-w-full max-h-[360px] object-contain cursor-crosshair"></canvas>
                                <div class="absolute bottom-2 left-2 bg-slate-900/80 backdrop-blur-md px-2.5 py-1 rounded-md text-[11px] text-slate-300 pointer-events-none">
                                    提示: 點擊圖片點選要消除的背景顏色
                                </div>
                            </div>

                            <!-- Controls for Chroma Key Removal -->
                            <div class="space-y-3 bg-slate-900/60 p-4 rounded-xl border border-slate-700/60">
                                <div class="flex items-center justify-between text-xs font-medium text-slate-300">
                                    <span>去背容差 (Tolerance)</span>
                                    <span id="toleranceValue" class="text-sky-400 font-bold">35</span>
                                </div>
                                <input type="range" id="toleranceSlider" min="5" max="100" value="35" oninput="updateTolerance(this.value)" class="w-full accent-sky-500 cursor-pointer">
                                
                                <div class="flex items-center justify-between pt-2">
                                    <div class="flex items-center space-x-2">
                                        <span class="text-xs text-slate-400">目前去除底色:</span>
                                        <div id="selectedColorPreview" class="w-6 h-6 rounded-md border border-slate-500 shadow-sm" style="background-color: #ffffff;"></div>
                                    </div>
                                    <button onclick="autoDetectBackground()" class="text-xs bg-slate-700 hover:bg-slate-600 text-slate-200 px-3 py-1.5 rounded-lg transition flex items-center space-x-1">
                                        <i class="fa-solid fa-wand-magic-sparkles text-sky-400"></i>
                                        <span>自動白紙去背</span>
                                    </button>
                                </div>

                                <div class="pt-2 border-t border-slate-800 flex items-center justify-between">
                                    <label class="flex items-center space-x-2 cursor-pointer text-xs text-slate-300">
                                        <input type="checkbox" id="floodFillToggle" checked onchange="processBackgroundRemoval()" class="w-4 h-4 rounded accent-sky-500">
                                        <span class="font-medium text-sky-300">智慧連通去背（保護人物內部顏色）</span>
                                    </label>
                                </div>
                            </div>

                            <!-- Kid Meta Information Input -->
                            <div class="space-y-3">
                                <div>
                                    <label class="block text-xs font-medium text-slate-300 mb-1">創作者姓名 / 作品名稱</label>
                                    <input type="text" id="artworkName" placeholder="例如：小明 - 勇者騎士" class="w-full bg-slate-900 border border-slate-700 rounded-xl px-3.5 py-2.5 text-sm text-white placeholder-slate-500 focus:outline-none focus:border-sky-500">
                                </div>

                                <div class="grid grid-cols-2 gap-3">
                                    <div>
                                        <label class="block text-xs font-medium text-slate-300 mb-1">預設人物大小</label>
                                        <select id="artworkScale" class="w-full bg-slate-900 border border-slate-700 rounded-xl px-3 py-2 text-xs text-white focus:outline-none focus:border-sky-500">
                                            <option value="0.8">精小 (80%)</option>
                                            <option value="1.0" selected>標準 (100%)</option>
                                            <option value="1.3">特大 (130%)</option>
                                        </select>
                                    </div>
                                    <div class="flex items-end">
                                        <button onclick="saveArtworkToCloud()" id="btnSave" class="w-full bg-gradient-to-r from-sky-500 to-blue-600 hover:from-sky-600 hover:to-blue-700 text-slate-950 font-bold py-2 px-4 rounded-xl shadow-lg transition flex items-center justify-center space-x-2 text-sm">
                                            <i class="fa-solid fa-cloud-arrow-up"></i>
                                            <span>發佈至校園</span>
                                        </button>
                                    </div>
                                </div>
                            </div>
                        </div>

                    </div>

                    <!-- RIGHT: Artwork Gallery & Active Management (7 cols) -->
                    <div class="lg:col-span-7 space-y-4">
                        <div class="flex items-center justify-between bg-slate-800/40 p-4 rounded-2xl border border-slate-700/50">
                            <h3 class="text-lg font-bold text-white flex items-center space-x-2">
                                <span class="w-8 h-8 rounded-lg bg-sky-500/20 text-sky-400 flex items-center justify-center text-sm">2</span>
                                <span>已發佈作品庫</span>
                            </h3>
                            <span class="text-xs text-slate-400">總計: <strong id="totalArtworksCount" class="text-sky-400">0</strong> 個作品</span>
                        </div>

                        <!-- Artworks Grid Display -->
                        <div id="artworksGrid" class="grid grid-cols-2 sm:grid-cols-3 gap-4 min-h-[300px]">
                            <!-- Dynamic Cards loaded via JS -->
                        </div>
                    </div>

                </div>
            </div>
        </div>
    </main>

    <!-- Network Sync & Pairing Modal -->
    <div id="networkModal" class="hidden fixed inset-0 z-50 flex items-center justify-center bg-black/70 backdrop-blur-sm p-4">
        <div class="bg-slate-800 border border-slate-700 rounded-3xl p-6 max-w-md w-full text-white shadow-2xl space-y-5">
            <div class="flex items-center justify-between border-b border-slate-700 pb-3">
                <h3 class="text-lg font-bold flex items-center space-x-2 text-sky-400">
                    <i class="fa-solid fa-wifi"></i>
                    <span>跨裝置點對點網路同步</span>
                </h3>
                <button onclick="toggleNetworkModal()" class="text-slate-400 hover:text-white">
                    <i class="fa-solid fa-xmark text-lg"></i>
                </button>
            </div>
            
            <!-- Room Info & QR Code -->
            <div class="bg-slate-900/80 p-4 rounded-2xl border border-slate-700 flex flex-col items-center text-center space-y-3">
                <p class="text-xs text-slate-300">讓 iPad 掃描此 QR Code 即可自動加入大螢幕的同步房間：</p>
                <div id="qrcode" class="p-3 bg-white rounded-xl shadow-md"></div>
                <div class="text-xs text-slate-400 pt-1">
                    當前房間號碼: <strong id="displayRoomId" class="text-sky-300 font-mono text-sm select-all">---</strong>
                </div>
            </div>

            <!-- Join Room Manual Input -->
            <div class="space-y-2 pt-2 border-t border-slate-700/60">
                <label class="block text-xs text-slate-300 font-medium">連線至大螢幕的房間號碼 (在 iPad 上操作時輸入)：</label>
                <div class="flex items-center space-x-2">
                    <input type="text" id="targetRoomInput" placeholder="例如：8f3a9b" class="flex-1 bg-slate-900 border border-slate-700 rounded-xl px-3 py-2 text-xs font-mono text-sky-300 focus:outline-none focus:border-sky-500">
                    <button onclick="connectToTargetRoom()" class="px-4 py-2 bg-sky-500 hover:bg-sky-600 text-slate-950 font-bold text-xs rounded-xl transition">配對連線</button>
                </div>
            </div>

            <div class="text-[11px] text-slate-400 leading-relaxed bg-slate-900/40 p-3 rounded-xl">
                💡 <b>提示</b>：本系統採用直連 WebRTC 技術，同一網絡/WiFi下的 iPad 與大螢幕會直接傳輸數據，不經過雲端第三方資料庫。
            </div>
        </div>
    </div>

    <!-- Custom Notification Toast -->
    <div id="toast" class="fixed bottom-6 right-6 z-50 transform translate-y-20 opacity-0 transition-all duration-300 pointer-events-none bg-slate-800 text-white border border-slate-700 shadow-2xl px-4 py-3 rounded-xl flex items-center space-x-3">
        <i id="toastIcon" class="fa-solid fa-circle-check text-emerald-400 text-lg"></i>
        <span id="toastMessage" class="text-sm font-medium">提示訊息</span>
    </div>

    <script>
        // Global APP State Management
        const state = {
            artworks: [],
            activeDisplayedIds: [],
            minDisplayCount: 10,
            rotationInterval: 180, // 180 seconds
            timerCountdown: 180,
            timerId: null,
            currentBgUrl: 'https://images.unsplash.com/photo-1580582932707-520aed937b7b?auto=format&fit=crop&w=2000&q=80',
            editor: {
                originalImg: null,
                targetRgb: { r: 255, g: 255, b: 255 },
                tolerance: 35
            },
            charactersPhysics: []
        };

        // P2P Network PeerJS State
        let peer = null;
        let peerRoomId = '';
        let connectedPeers = [];
        const broadcast = typeof BroadcastChannel !== 'undefined' ? new BroadcastChannel('school_openday_local_sync') : null;

        if (broadcast) {
            broadcast.onmessage = (e) => {
                if (e.data && e.data.type === 'SYNC_PAYLOAD') {
                    applyReceivedPayload(e.data.payload, false);
                }
            };
        }

        function notifyLocalSync(payload) {
            if (broadcast) {
                try { broadcast.postMessage({ type: 'SYNC_PAYLOAD', payload }); } catch(err){}
            }
            broadcastToP2PPeers(payload);
        }

        window.switchTab = function(tabName) {
            const displayBtn = document.getElementById('btnTabDisplay');
            const adminBtn = document.getElementById('btnTabAdmin');
            const panelDisplay = document.getElementById('panelDisplay');
            const panelAdmin = document.getElementById('panelAdmin');

            if (tabName === 'display') {
                panelDisplay.classList.remove('hidden');
                panelAdmin.classList.add('hidden');
                
                displayBtn.className = "px-4 py-2 rounded-lg text-sm font-medium transition flex items-center space-x-2 bg-sky-500 text-slate-950 font-bold shadow-md";
                adminBtn.className = "px-4 py-2 rounded-lg text-sm font-medium transition flex items-center space-x-2 text-slate-300 hover:text-white hover:bg-slate-800";
                
                refreshDisplayCharacters();
            } else {
                panelDisplay.classList.add('hidden');
                panelAdmin.classList.remove('hidden');

                adminBtn.className = "px-4 py-2 rounded-lg text-sm font-medium transition flex items-center space-x-2 bg-sky-500 text-slate-950 font-bold shadow-md";
                displayBtn.className = "px-4 py-2 rounded-lg text-sm font-medium transition flex items-center space-x-2 text-slate-300 hover:text-white hover:bg-slate-800";
            }
        };

        window.toggleFullScreen = function() {
            if (!document.fullscreenElement) {
                document.documentElement.requestFullscreen().catch(err => {
                    console.warn(`Fullscreen error: ${err.message}`);
                });
                document.getElementById('fullscreenIcon').className = "fa-solid fa-compress text-lg";
            } else {
                if (document.exitFullscreen) {
                    document.exitFullscreen();
                    document.getElementById('fullscreenIcon').className = "fa-solid fa-expand text-lg";
                }
            }
        };

        window.showToast = function(msg, type = "success") {
            const toast = document.getElementById('toast');
            const toastMessage = document.getElementById('toastMessage');
            const toastIcon = document.getElementById('toastIcon');

            toastMessage.innerText = msg;
            if (type === "error") {
                toastIcon.className = "fa-solid fa-circle-xmark text-red-400 text-lg";
            } else if (type === "info") {
                toastIcon.className = "fa-solid fa-circle-info text-sky-400 text-lg";
            } else {
                toastIcon.className = "fa-solid fa-circle-check text-emerald-400 text-lg";
            }

            toast.classList.remove('translate-y-20', 'opacity-0');
            toast.classList.add('translate-y-0', 'opacity-100');

            setTimeout(() => {
                toast.classList.remove('translate-y-0', 'opacity-100');
                toast.classList.add('translate-y-20', 'opacity-0');
            }, 3000);
        };

        window.showBanner = function(msg) {
            const banner = document.getElementById('newArtworkBanner');
            const bannerText = document.getElementById('bannerText');
            bannerText.innerText = msg;
            banner.classList.remove('hidden');

            setTimeout(() => {
                banner.classList.add('hidden');
            }, 5000);
        };

        window.toggleNetworkModal = function() {
            const modal = document.getElementById('networkModal');
            modal.classList.toggle('hidden');
        };

        window.changeCampusBackground = function(bgUrl) {
            state.currentBgUrl = bgUrl;
            applyCampusBg(bgUrl);
            localStorage.setItem('school_openday_bg', bgUrl);
            notifyLocalSync({ type: 'BG_UPDATE', bgUrl });
        };

        window.handleBgUpload = function(event) {
            const file = event.target.files[0];
            if (!file) return;

            showToast("正在處理背景照片...", "info");

            const reader = new FileReader();
            reader.onload = function(e) {
                const img = new Image();
                img.onload = function() {
                    const canvas = document.createElement('canvas');
                    const maxW = 1280;
                    const maxH = 720;
                    let width = img.width;
                    let height = img.height;

                    if (width > maxW || height > maxH) {
                        if (width / height > maxW / maxH) {
                            height = Math.round((height * maxW) / width);
                            width = maxW;
                        } else {
                            width = Math.round((width * maxH) / height);
                            height = maxH;
                        }
                    }

                    canvas.width = width;
                    canvas.height = height;
                    const ctx = canvas.getContext('2d');
                    ctx.drawImage(img, 0, 0, width, height);

                    const compressedBase64 = canvas.toDataURL('image/jpeg', 0.7);
                    state.currentBgUrl = compressedBase64;
                    applyCampusBg(compressedBase64);
                    localStorage.setItem('school_openday_bg', compressedBase64);
                    notifyLocalSync({ type: 'BG_UPDATE', bgUrl: compressedBase64 });
                    showToast("新校園背景已成功更換！", "success");
                };
                img.src = e.target.result;
            };
            reader.readAsDataURL(file);
        };

        function applyCampusBg(bgUrl) {
            const bgEl = document.getElementById('campusBgImage');
            if (bgEl) {
                bgEl.style.backgroundImage = `url('${bgUrl}')`;
            }
            const selectEl = document.getElementById('settingBgTheme');
            if (selectEl) {
                for (let i = 0; i < selectEl.options.length; i++) {
                    if (selectEl.options[i].value === bgUrl) {
                        selectEl.selectedIndex = i;
                        break;
                    }
                }
            }
        }

        window.updateSystemSettings = function() {
            const minCount = parseInt(document.getElementById('settingMinCount').value, 10);
            state.minDisplayCount = minCount;
            refreshDisplayCharacters();
            showToast(`已更改每批顯示數量為最少 ${minCount} 個`, "info");
        };

        function startRotationTimer() {
            if (state.timerId) clearInterval(state.timerId);
            state.timerCountdown = state.rotationInterval;

            state.timerId = setInterval(() => {
                state.timerCountdown--;
                const countdownEl = document.getElementById('timerCountdown');
                if (countdownEl) countdownEl.innerText = state.timerCountdown;

                if (state.timerCountdown <= 0) {
                    state.timerCountdown = state.rotationInterval;
                    rotateArtworks();
                }
            }, 1000);
        }

        window.triggerManualRotation = function() {
            state.timerCountdown = state.rotationInterval;
            rotateArtworks();
            showToast("已手動換上一批作品展示", "info");
        };

        function rotateArtworks() {
            if (state.artworks.length <= state.minDisplayCount) {
                refreshDisplayCharacters();
                return;
            }
            const activeArtworks = state.artworks.filter(a => a.active !== false);
            if (activeArtworks.length > 0) {
                const shifted = activeArtworks.shift();
                activeArtworks.push(shifted);
            }
            refreshDisplayCharacters();
        }

        function updateSyncUI(connectedCount) {
            const syncText = document.getElementById('syncText');
            const syncDot = document.getElementById('syncDot');
            if (syncText) {
                if (connectedCount > 0) {
                    syncText.innerText = `網路已連線 (${connectedCount} 裝置)`;
                } else {
                    syncText.innerText = "網路房間就緒";
                }
            }
            if (syncDot) {
                syncDot.className = connectedCount > 0 ? "w-2 h-2 mr-1.5 rounded-full bg-emerald-400 animate-pulse" : "w-2 h-2 mr-1.5 rounded-full bg-sky-400";
            }
        }

        function loadLocalArtworks() {
            try {
                const saved = localStorage.getItem('school_openday_artworks');
                if (saved) {
                    state.artworks = JSON.parse(saved);
                } else {
                    state.artworks = [];
                }
            } catch(e) {
                state.artworks = [];
            }
        }

        function saveLocalArtworks() {
            try {
                localStorage.setItem('school_openday_artworks', JSON.stringify(state.artworks));
            } catch(e) {
                console.warn("Error saving local artworks:", e);
            }
        }

        /* Direct P2P Network Synchronization using PeerJS */
        function initNetworkSync() {
            loadLocalArtworks();

            const savedBg = localStorage.getItem('school_openday_bg');
            if (savedBg) {
                state.currentBgUrl = savedBg;
                applyCampusBg(savedBg);
            }

            renderGalleryGrid();
            refreshDisplayCharacters();

            // Check if URL has ?room=xxxx parameter
            const urlParams = new URLSearchParams(window.location.search);
            const roomParam = urlParams.get('room');

            peerRoomId = roomParam || ('ps_' + Math.random().toString(36).substr(2, 6));
            document.getElementById('displayRoomId').innerText = peerRoomId;

            // Generate QR Code for fast joining
            const joinUrl = window.location.origin + window.location.pathname + '?room=' + peerRoomId;
            const qrContainer = document.getElementById('qrcode');
            if (qrContainer) {
                qrContainer.innerHTML = '';
                new QRCode(qrContainer, {
                    text: joinUrl,
                    width: 140,
                    height: 140
                });
            }

            try {
                peer = new Peer(peerRoomId);
                
                peer.on('open', (id) => {
                    updateSyncUI(0);
                    // If room param was provided, connect to host room automatically
                    if (roomParam) {
                        connectToRoom(roomParam);
                    }
                });

                peer.on('connection', (conn) => {
                    setupP2PConnection(conn);
                });

                peer.on('error', (err) => {
                    console.warn("P2P Peer error:", err);
                    if (err.type === 'unavailable-id') {
                        // Id taken, join room instead
                        connectToRoom(peerRoomId);
                    }
                });
            } catch(err) {
                console.warn("P2P Init failed:", err);
            }
        }

        function setupP2PConnection(conn) {
            conn.on('open', () => {
                if (!connectedPeers.find(c => c.peer === conn.peer)) {
                    connectedPeers.push(conn);
                }
                updateSyncUI(connectedPeers.length);
                showToast("連線成功！裝置已配對", "success");

                // Send initial full state to newly connected peer
                conn.send({
                    type: 'FULL_SYNC',
                    artworks: state.artworks,
                    currentBgUrl: state.currentBgUrl
                });
            });

            conn.on('data', (data) => {
                applyReceivedPayload(data, true);
            });

            conn.on('close', () => {
                connectedPeers = connectedPeers.filter(c => c.peer !== conn.peer);
                updateSyncUI(connectedPeers.length);
            });
        }

        window.connectToTargetRoom = function() {
            const room = document.getElementById('targetRoomInput').value.trim();
            if (!room) {
                showToast("請輸入房間號碼", "error");
                return;
            }
            connectToRoom(room);
        };

        function connectToRoom(roomId) {
            if (!peer) return;
            showToast(`正在連線至房間 ${roomId}...`, "info");
            const conn = peer.connect(roomId);
            setupP2PConnection(conn);
        }

        function broadcastToP2PPeers(payload) {
            connectedPeers.forEach(conn => {
                if (conn.open) {
                    try { conn.send(payload); } catch(e){}
                }
            });
        }

        function applyReceivedPayload(payload, isFromP2P = false) {
            if (!payload) return;

            if (payload.type === 'FULL_SYNC') {
                state.artworks = payload.artworks || [];
                if (payload.currentBgUrl) {
                    state.currentBgUrl = payload.currentBgUrl;
                    applyCampusBg(payload.currentBgUrl);
                }
                saveLocalArtworks();
                renderGalleryGrid();
                refreshDisplayCharacters();
            } else if (payload.type === 'ARTWORK_ADD') {
                const exists = state.artworks.some(a => a.id === payload.artwork.id);
                if (!exists) {
                    state.artworks.unshift(payload.artwork);
                    saveLocalArtworks();
                    renderGalleryGrid();
                    refreshDisplayCharacters();
                    showBanner(`🎉 「${payload.artwork.name || '小朋友'}」的作品剛剛加入了校園舞台！`);
                    playFanfareSound();
                }
            } else if (payload.type === 'ARTWORK_UPDATE') {
                const found = state.artworks.find(a => a.id === payload.artwork.id);
                if (found) {
                    Object.assign(found, payload.artwork);
                    saveLocalArtworks();
                    renderGalleryGrid();
                    refreshDisplayCharacters();
                }
            } else if (payload.type === 'ARTWORK_DELETE') {
                state.artworks = state.artworks.filter(a => a.id !== payload.id);
                saveLocalArtworks();
                renderGalleryGrid();
                refreshDisplayCharacters();
            } else if (payload.type === 'BG_UPDATE') {
                state.currentBgUrl = payload.bgUrl;
                applyCampusBg(payload.bgUrl);
                localStorage.setItem('school_openday_bg', payload.bgUrl);
            }

            if (isFromP2P) {
                // Relay to BroadcastChannel
                if (broadcast) {
                    try { broadcast.postMessage({ type: 'SYNC_PAYLOAD', payload }); } catch(e){}
                }
            }
        }

        window.triggerFileSelect = function() {
            document.getElementById('imageInput').click();
        };

        window.handleFileSelect = function(event) {
            const file = event.target ? event.target.files[0] : event;
            if (!file || !(file instanceof Blob)) return;

            const reader = new FileReader();
            reader.onload = function(e) {
                const img = new Image();
                img.onload = function() {
                    state.editor.originalImg = img;
                    document.getElementById('uploadPlaceholder').classList.add('hidden');
                    document.getElementById('editorStudio').classList.remove('hidden');
                    
                    const tempCanvas = document.createElement('canvas');
                    tempCanvas.width = img.width;
                    tempCanvas.height = img.height;
                    const ctx = tempCanvas.getContext('2d');
                    ctx.drawImage(img, 0, 0);
                    const cornerPixel = ctx.getImageData(0, 0, 1, 1).data;
                    
                    state.editor.targetRgb = { r: cornerPixel[0], g: cornerPixel[1], b: cornerPixel[2] };
                    updateColorPreview();
                    processBackgroundRemoval();
                };
                img.src = e.target.result;
            };
            reader.readAsDataURL(file);
        };

        const processCanvas = document.getElementById('processCanvas');
        if (processCanvas) {
            processCanvas.addEventListener('click', function(e) {
                if (!state.editor.originalImg) return;
                const rect = processCanvas.getBoundingClientRect();
                const scaleX = processCanvas.width / rect.width;
                const scaleY = processCanvas.height / rect.height;
                const x = Math.floor((e.clientX - rect.left) * scaleX);
                const y = Math.floor((e.clientY - rect.top) * scaleY);

                const tempCanvas = document.createElement('canvas');
                tempCanvas.width = state.editor.originalImg.width;
                tempCanvas.height = state.editor.originalImg.height;
                const ctx = tempCanvas.getContext('2d');
                ctx.drawImage(state.editor.originalImg, 0, 0);
                
                const pixel = ctx.getImageData(x, y, 1, 1).data;
                state.editor.targetRgb = { r: pixel[0], g: pixel[1], b: pixel[2] };
                updateColorPreview();
                processBackgroundRemoval();
            });
        }

        window.updateTolerance = function(val) {
            state.editor.tolerance = parseInt(val, 10);
            document.getElementById('toleranceValue').innerText = val;
            processBackgroundRemoval();
        };

        window.autoDetectBackground = function() {
            state.editor.targetRgb = { r: 245, g: 245, b: 245 };
            state.editor.tolerance = 40;
            document.getElementById('toleranceSlider').value = 40;
            document.getElementById('toleranceValue').innerText = 40;
            updateColorPreview();
            processBackgroundRemoval();
        };

        function updateColorPreview() {
            const rgb = state.editor.targetRgb;
            document.getElementById('selectedColorPreview').style.backgroundColor = `rgb(${rgb.r}, ${rgb.g}, ${rgb.b})`;
        }

        function processBackgroundRemoval() {
            const img = state.editor.originalImg;
            if (!img) return;

            const maxDim = 600;
            let width = img.width;
            let height = img.height;
            if (width > maxDim || height > maxDim) {
                if (width > height) {
                    height = Math.round((height * maxDim) / width);
                    width = maxDim;
                } else {
                    width = Math.round((width * maxDim) / height);
                    height = maxDim;
                }
            }

            processCanvas.width = width;
            processCanvas.height = height;

            const ctx = processCanvas.getContext('2d');
            ctx.drawImage(img, 0, 0, width, height);

            const imageData = ctx.getImageData(0, 0, width, height);
            const data = imageData.data;
            const target = state.editor.targetRgb;
            const tol = state.editor.tolerance;
            const useFloodFill = document.getElementById('floodFillToggle') ? document.getElementById('floodFillToggle').checked : true;

            if (useFloodFill) {
                const visited = new Uint8Array(width * height);
                const queue = new Int32Array(width * height);
                let qHead = 0;
                let qTail = 0;

                function isBgColor(idx) {
                    const r = data[idx];
                    const g = data[idx + 1];
                    const b = data[idx + 2];
                    const dist = Math.sqrt(
                        (r - target.r) * (r - target.r) +
                        (g - target.g) * (g - target.g) +
                        (b - target.b) * (b - target.b)
                    );
                    return dist < tol;
                }

                for (let x = 0; x < width; x++) {
                    if (isBgColor(x * 4)) {
                        visited[x] = 1;
                        queue[qTail++] = x;
                    }
                    const bPos = (height - 1) * width + x;
                    if (!visited[bPos] && isBgColor(bPos * 4)) {
                        visited[bPos] = 1;
                        queue[qTail++] = bPos;
                    }
                }

                for (let y = 0; y < height; y++) {
                    const lPos = y * width;
                    if (!visited[lPos] && isBgColor(lPos * 4)) {
                        visited[lPos] = 1;
                        queue[qTail++] = lPos;
                    }
                    const rPos = y * width + (width - 1);
                    if (!visited[rPos] && isBgColor(rPos * 4)) {
                        visited[rPos] = 1;
                        queue[qTail++] = rPos;
                    }
                }

                while (qHead < qTail) {
                    const pos = queue[qHead++];
                    const px = pos % width;
                    const py = Math.floor(pos / width);

                    data[pos * 4 + 3] = 0;

                    const neighbors = [
                        px > 0 ? pos - 1 : -1,
                        px < width - 1 ? pos + 1 : -1,
                        py > 0 ? pos - width : -1,
                        py < height - 1 ? pos + width : -1
                    ];

                    for (let i = 0; i < 4; i++) {
                        const nPos = neighbors[i];
                        if (nPos >= 0 && !visited[nPos]) {
                            visited[nPos] = 1;
                            if (isBgColor(nPos * 4)) {
                                queue[qTail++] = nPos;
                            }
                        }
                    }
                }
            } else {
                for (let i = 0; i < data.length; i += 4) {
                    const r = data[i];
                    const g = data[i + 1];
                    const b = data[i + 2];

                    const dist = Math.sqrt(
                        (r - target.r) * (r - target.r) +
                        (g - target.g) * (g - target.g) +
                        (b - target.b) * (b - target.b)
                    );

                    if (dist < tol) {
                        data[i + 3] = 0;
                    } else if (dist < tol + 15) {
                        const alphaRatio = (dist - tol) / 15;
                        data[i + 3] = Math.floor(data[i + 3] * alphaRatio);
                    }
                }
            }

            ctx.putImageData(imageData, 0, 0);
        }

        window.saveArtworkToCloud = async function() {
            if (!state.editor.originalImg) {
                showToast("請先上傳或選擇圖片", "error");
                return;
            }

            const nameInput = document.getElementById('artworkName').value.trim() || "小創作者";
            const scaleSelect = parseFloat(document.getElementById('artworkScale').value) || 1.0;
            
            const tempCanvas = document.createElement('canvas');
            const maxDim = 450;
            let w = processCanvas.width;
            let h = processCanvas.height;
            if (w > maxDim || h > maxDim) {
                if (w > h) {
                    h = Math.round((h * maxDim) / w);
                    w = maxDim;
                } else {
                    w = Math.round((w * maxDim) / h);
                    h = maxDim;
                }
            }
            tempCanvas.width = w;
            tempCanvas.height = h;
            const tempCtx = tempCanvas.getContext('2d');
            tempCtx.drawImage(processCanvas, 0, 0, w, h);

            const processedBase64 = tempCanvas.toDataURL('image/png');

            const newDoc = {
                id: 'art_' + Date.now() + '_' + Math.random().toString(36).substr(2, 4),
                name: nameInput,
                scale: scaleSelect,
                imageData: processedBase64,
                createdAt: Date.now(),
                active: true
            };

            const btnSave = document.getElementById('btnSave');
            btnSave.disabled = true;
            btnSave.innerHTML = `<i class="fa-solid fa-spinner animate-spin"></i> <span>發佈中...</span>`;

            state.artworks.unshift(newDoc);
            saveLocalArtworks();
            renderGalleryGrid();
            refreshDisplayCharacters();

            notifyLocalSync({ type: 'ARTWORK_ADD', artwork: newDoc });

            showToast("作品已成功發佈並同步至展示舞台！", "success");

            btnSave.disabled = false;
            btnSave.innerHTML = `<i class="fa-solid fa-cloud-arrow-up"></i> <span>發佈至校園</span>`;
            resetUploadForm();
            switchTab('display');
        };

        window.resetUploadForm = function() {
            state.editor.originalImg = null;
            document.getElementById('imageInput').value = '';
            document.getElementById('artworkName').value = '';
            document.getElementById('uploadPlaceholder').classList.remove('hidden');
            document.getElementById('editorStudio').classList.add('hidden');
        };

        function renderGalleryGrid() {
            const grid = document.getElementById('artworksGrid');
            document.getElementById('totalArtworksCount').innerText = state.artworks.length;

            if (state.artworks.length === 0) {
                grid.innerHTML = `
                    <div class="col-span-full py-12 text-center text-slate-500 bg-slate-900/40 rounded-2xl border border-slate-800">
                        <i class="fa-solid fa-image text-3xl mb-2 text-slate-600"></i>
                        <p class="text-sm">尚未有作品。請在左側拍照或上傳小朋友的繪畫作品！</p>
                    </div>
                `;
                return;
            }

            grid.innerHTML = state.artworks.map((item) => `
                <div class="bg-slate-800 rounded-2xl p-3 border ${item.active ? 'border-slate-700' : 'border-red-900/40 opacity-60'} flex flex-col justify-between space-y-3 group shadow-md hover:border-slate-600 transition">
                    <div class="relative bg-checkerboard rounded-xl h-32 flex items-center justify-center overflow-hidden">
                        <img src="${item.imageData}" alt="${item.name}" class="max-h-28 max-w-full object-contain character-shadow transform group-hover:scale-105 transition duration-300">
                        ${!item.active ? '<span class="absolute top-2 left-2 bg-red-500 text-white text-[10px] px-2 py-0.5 rounded-full font-bold">已停用</span>' : ''}
                    </div>

                    <div class="space-y-2">
                        <div>
                            <label class="text-[10px] text-slate-400 block mb-0.5 font-medium">作品名稱 / 創作者</label>
                            <div class="relative">
                                <input type="text" value="${item.name || ''}" placeholder="輸入名稱" 
                                    onchange="updateArtworkName('${item.id}', this.value)"
                                    class="w-full bg-slate-900 border border-slate-700 focus:border-sky-500 rounded-lg px-2.5 py-1 text-xs text-white focus:outline-none transition pr-6">
                                <i class="fa-solid fa-pen text-[10px] text-slate-500 absolute right-2 top-2 pointer-events-none"></i>
                            </div>
                        </div>
                        
                        <div>
                            <label class="text-[10px] text-slate-400 block mb-0.5 font-medium">展示尺寸</label>
                            <select onchange="updateArtworkScale('${item.id}', parseFloat(this.value))" 
                                class="w-full bg-slate-900 border border-slate-700 focus:border-sky-500 rounded-lg px-2 py-1 text-xs text-sky-300 font-semibold focus:outline-none transition">
                                <option value="0.6" ${item.scale == 0.6 ? 'selected' : ''}>特小 (60%)</option>
                                <option value="0.8" ${item.scale == 0.8 ? 'selected' : ''}>精小 (80%)</option>
                                <option value="1.0" ${(!item.scale || item.scale == 1.0) ? 'selected' : ''}>標準 (100%)</option>
                                <option value="1.3" ${item.scale == 1.3 ? 'selected' : ''}>特大 (130%)</option>
                                <option value="1.6" ${item.scale == 1.6 ? 'selected' : ''}>超大 (160%)</option>
                            </select>
                        </div>
                    </div>

                    <div class="flex items-center justify-between pt-2 border-t border-slate-700/60 text-xs">
                        <button onclick="toggleArtworkActive('${item.id}', ${!item.active})" class="px-2.5 py-1 rounded-lg ${item.active ? 'bg-sky-500/20 text-sky-300 hover:bg-sky-500/30' : 'bg-slate-700 text-slate-300'} transition">
                            ${item.active ? '<i class="fa-solid fa-eye"></i> 展示中' : '<i class="fa-solid fa-eye-slash"></i> 已隱藏'}
                        </button>
                        <button onclick="deleteArtwork('${item.id}')" class="p-1.5 text-slate-400 hover:text-red-400 hover:bg-red-500/10 rounded-lg transition" title="刪除作品">
                            <i class="fa-solid fa-trash-can"></i>
                        </button>
                    </div>
                </div>
            `).join('');
        }

        window.updateArtworkName = function(id, newName) {
            const trimmed = newName.trim() || '無名作品';
            const found = state.artworks.find(a => a.id === id);
            if (found) {
                found.name = trimmed;
                saveLocalArtworks();
                renderGalleryGrid();
                refreshDisplayCharacters();
                notifyLocalSync({ type: 'ARTWORK_UPDATE', artwork: found });
            }
            showToast("作品名稱已更新", "success");
        };

        window.updateArtworkScale = function(id, newScale) {
            const found = state.artworks.find(a => a.id === id);
            if (found) {
                found.scale = newScale;
                saveLocalArtworks();
                renderGalleryGrid();
                refreshDisplayCharacters();
                notifyLocalSync({ type: 'ARTWORK_UPDATE', artwork: found });
            }
            showToast("作品尺寸已更新", "success");
        };

        window.toggleArtworkActive = function(id, newActive) {
            const found = state.artworks.find(a => a.id === id);
            if (found) {
                found.active = newActive;
                saveLocalArtworks();
                renderGalleryGrid();
                refreshDisplayCharacters();
                notifyLocalSync({ type: 'ARTWORK_UPDATE', artwork: found });
            }
        };

        window.deleteArtwork = function(id) {
            state.artworks = state.artworks.filter(a => a.id !== id);
            saveLocalArtworks();
            renderGalleryGrid();
            refreshDisplayCharacters();
            notifyLocalSync({ type: 'ARTWORK_DELETE', id });
            showToast("作品已從庫中刪除", "info");
        };

        function refreshDisplayCharacters() {
            const stage = document.getElementById('charactersStage');
            const emptyState = document.getElementById('emptyDisplayState');
            
            const availableArtworks = state.artworks.filter(a => a.active !== false);

            if (availableArtworks.length === 0) {
                stage.innerHTML = '';
                state.charactersPhysics = [];
                emptyState.classList.remove('hidden');
                document.getElementById('displayCount').innerText = '0';
                return;
            }

            emptyState.classList.add('hidden');

            const countToDisplay = Math.max(1, Math.min(availableArtworks.length, 12));
            const selectedSet = availableArtworks.slice(0, countToDisplay);
            document.getElementById('displayCount').innerText = selectedSet.length;

            stage.innerHTML = '';
            state.charactersPhysics = [];

            const stageRect = document.getElementById('campusContainer').getBoundingClientRect();
            const containerW = stageRect.width || window.innerWidth;
            const containerH = stageRect.height || window.innerHeight;

            selectedSet.forEach((item) => {
                const charEl = document.createElement('div');
                charEl.className = 'absolute cursor-grab active:cursor-grabbing transition-transform duration-100 select-none touch-none pointer-events-auto';
                
                const baseScale = item.scale || 1.0;
                const charSize = Math.floor(110 * baseScale);

                charEl.style.width = `${charSize}px`;
                charEl.style.height = `${charSize}px`;

                charEl.innerHTML = `
                    <div class="relative w-full h-full flex flex-col items-center pointer-events-none">
                        <img src="${item.imageData}" alt="${item.name}" class="w-full h-full object-contain character-shadow animate-bounce-gentle">
                        <div class="absolute -bottom-6 bg-slate-900/80 backdrop-blur-md text-sky-300 font-bold text-[11px] px-2.5 py-0.5 rounded-full border border-sky-500/30 whitespace-nowrap shadow-lg">
                            ${item.name || '小小畫家'}
                        </div>
                    </div>
                `;

                stage.appendChild(charEl);

                const posX = Math.random() * Math.max(10, containerW - charSize - 40) + 20;
                const posY = Math.random() * Math.max(10, containerH - charSize - 120) + 60;
                const vx = (Math.random() - 0.5) * 1.5;
                const vy = (Math.random() - 0.5) * 1.0;

                const physObj = {
                    element: charEl,
                    x: posX,
                    y: posY,
                    vx: vx === 0 ? 1 : vx,
                    vy: vy === 0 ? 1 : vy,
                    width: charSize,
                    height: charSize,
                    isDragging: false,
                    dragOffsetX: 0,
                    dragOffsetY: 0
                };

                setupDragEvents(charEl, physObj);
                state.charactersPhysics.push(physObj);
            });
        }

        // Animation Loop
        function animateCharacters() {
            const container = document.getElementById('campusContainer');
            if (container) {
                const rect = container.getBoundingClientRect();
                const containerW = rect.width || window.innerWidth;
                const containerH = rect.height || window.innerHeight;

                state.charactersPhysics.forEach((phys) => {
                    if (phys.isDragging) return;

                    phys.x += phys.vx;
                    phys.y += phys.vy;

                    if (phys.x <= 10) {
                        phys.x = 10;
                        phys.vx = Math.abs(phys.vx);
                    } else if (phys.x + phys.width >= containerW - 10) {
                        phys.x = containerW - phys.width - 10;
                        phys.vx = -Math.abs(phys.vx);
                    }

                    if (phys.y <= 40) {
                        phys.y = 40;
                        phys.vy = Math.abs(phys.vy);
                    } else if (phys.y + phys.height >= containerH - 60) {
                        phys.y = containerH - phys.height - 60;
                        phys.vy = -Math.abs(phys.vy);
                    }

                    phys.element.style.transform = `translate3d(${phys.x}px, ${phys.y}px, 0px)`;
                });
            }
            requestAnimationFrame(animateCharacters);
        }

        function setupDragEvents(el, physObj) {
            let lastMoveTime = Date.now();
            let lastX = physObj.x;
            let lastY = physObj.y;
            let startX = 0;
            let startY = 0;
            let totalMoved = 0;

            const onStart = (clientX, clientY) => {
                physObj.isDragging = true;
                startX = clientX;
                startY = clientY;
                totalMoved = 0;

                const container = document.getElementById('campusContainer');
                const rect = container.getBoundingClientRect();
                
                physObj.dragOffsetX = clientX - rect.left - physObj.x;
                physObj.dragOffsetY = clientY - rect.top - physObj.y;
                
                lastX = physObj.x;
                lastY = physObj.y;
                lastMoveTime = Date.now();

                el.classList.add('z-50');
                const img = el.querySelector('img');
                if (img) img.classList.remove('animate-bounce-gentle');
            };

            const onMove = (clientX, clientY) => {
                if (!physObj.isDragging) return;

                const moveDist = Math.hypot(clientX - startX, clientY - startY);
                totalMoved = Math.max(totalMoved, moveDist);

                const container = document.getElementById('campusContainer');
                const rect = container.getBoundingClientRect();

                let newX = clientX - rect.left - physObj.dragOffsetX;
                let newY = clientY - rect.top - physObj.dragOffsetY;

                const maxW = rect.width || window.innerWidth;
                const maxH = rect.height || window.innerHeight;
                newX = Math.max(0, Math.min(newX, maxW - physObj.width));
                newY = Math.max(20, Math.min(newY, maxH - physObj.height - 40));

                const now = Date.now();
                const dt = (now - lastMoveTime) / 1000;
                if (dt > 0.01) {
                    physObj.vx = (newX - lastX) / (dt * 60);
                    physObj.vy = (newY - lastY) / (dt * 60);
                    physObj.vx = Math.max(-8, Math.min(8, physObj.vx));
                    physObj.vy = Math.max(-8, Math.min(8, physObj.vy));
                    
                    lastX = newX;
                    lastY = newY;
                    lastMoveTime = now;
                }

                physObj.x = newX;
                physObj.y = newY;
                physObj.element.style.transform = `translate3d(${physObj.x}px, ${physObj.y}px, 0px)`;
            };

            const onEnd = () => {
                if (!physObj.isDragging) return;
                physObj.isDragging = false;
                el.classList.remove('z-50');
                const img = el.querySelector('img');
                if (img) img.classList.add('animate-bounce-gentle');

                if (totalMoved < 10) {
                    triggerCharacterJump(el, physObj);
                }

                if (Math.abs(physObj.vx) < 0.2) physObj.vx = (Math.random() - 0.5) * 1.5 || 1;
                if (Math.abs(physObj.vy) < 0.2) physObj.vy = (Math.random() - 0.5) * 1.5 || 1;
            };

            el.addEventListener('mousedown', (e) => {
                e.preventDefault();
                onStart(e.clientX, e.clientY);

                const handleMouseMove = (e) => onMove(e.clientX, e.clientY);
                const handleMouseUp = () => {
                    onEnd();
                    window.removeEventListener('mousemove', handleMouseMove);
                    window.removeEventListener('mouseup', handleMouseUp);
                };

                window.addEventListener('mousemove', handleMouseMove);
                window.addEventListener('mouseup', handleMouseUp);
            });

            el.addEventListener('touchstart', (e) => {
                if (e.touches.length > 0) {
                    onStart(e.touches[0].clientX, e.touches[0].clientY);
                }
            }, { passive: true });

            el.addEventListener('touchmove', (e) => {
                if (physObj.isDragging && e.touches.length > 0) {
                    onMove(e.touches[0].clientX, e.touches[0].clientY);
                }
            }, { passive: true });

            el.addEventListener('touchend', onEnd, { passive: true });
            el.addEventListener('touchcancel', onEnd, { passive: true });
        }

        function triggerCharacterJump(el, physObj) {
            const img = el.querySelector('img');
            if (img) {
                img.classList.remove('animate-bounce-gentle', 'animate-jump');
                void img.offsetWidth;
                img.classList.add('animate-jump');

                setTimeout(() => {
                    img.classList.remove('animate-jump');
                    img.classList.add('animate-bounce-gentle');
                }, 550);
            }

            physObj.vy = -4 - Math.random() * 2;
            physObj.vx += (Math.random() - 0.5) * 3;

            playBoingSound();
            spawnSparkles(physObj.x + physObj.width / 2, physObj.y);
        }

        function spawnSparkles(x, y) {
            const container = document.getElementById('campusContainer');
            if (!container) return;

            const icons = ['✨', '⭐', '❤️', '🎨', '🎉'];
            for (let i = 0; i < 5; i++) {
                const p = document.createElement('div');
                p.className = 'sparkle-pop';
                p.innerText = icons[Math.floor(Math.random() * icons.length)];
                p.style.left = `${x}px`;
                p.style.top = `${y}px`;
                const dx = (Math.random() - 0.5) * 80;
                p.style.setProperty('--dx', `${dx}px`);
                container.appendChild(p);

                setTimeout(() => p.remove(), 800);
            }
        }

        function playBoingSound() {
            try {
                const AudioCtx = window.AudioContext || window.webkitAudioContext;
                if (!AudioCtx) return;
                const ctx = new AudioCtx();
                const osc = ctx.createOscillator();
                const gain = ctx.createGain();

                osc.type = 'sine';
                const now = ctx.currentTime;
                osc.frequency.setValueAtTime(260, now);
                osc.frequency.exponentialRampToValueAtTime(680, now + 0.12);
                osc.frequency.exponentialRampToValueAtTime(320, now + 0.28);

                gain.gain.setValueAtTime(0.35, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.3);

                osc.connect(gain);
                gain.connect(ctx.destination);

                osc.start(now);
                osc.stop(now + 0.3);
            } catch (e) {}
        }

        function playFanfareSound() {
            try {
                const AudioCtx = window.AudioContext || window.webkitAudioContext;
                if (!AudioCtx) return;
                const ctx = new AudioCtx();
                const now = ctx.currentTime;

                const notes = [261.63, 329.63, 392.00, 523.25];
                notes.forEach((freq, i) => {
                    const osc = ctx.createOscillator();
                    const gain = ctx.createGain();
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(freq, now + i * 0.08);

                    gain.gain.setValueAtTime(0.2, now + i * 0.08);
                    gain.gain.exponentialRampToValueAtTime(0.001, now + i * 0.08 + 0.3);

                    osc.connect(gain);
                    gain.connect(ctx.destination);

                    osc.start(now + i * 0.08);
                    osc.stop(now + i * 0.08 + 0.3);
                });
            } catch (e) {}
        }

        // Initialize Application
        window.addEventListener('DOMContentLoaded', () => {
            initNetworkSync();
            requestAnimationFrame(animateCharacters);
            startRotationTimer();

            window.addEventListener('resize', () => {
                refreshDisplayCharacters();
            });
        });
    </script>
</body>
</html>
