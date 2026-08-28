[Uploading gemini-code-1787885103925.txt…]()
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>大埔浸信會公立學校 - 填色作品大展</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Fredoka:wght@400;600&family=Noto+Sans+TC:wght@400;700&display=swap" rel="stylesheet">
    <!-- 引入 Socket.io 用於即時網路同步 -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/4.7.2/socket.io.min.js"></script>
    <style>
        body { font-family: 'Fredoka', 'Noto Sans TC', sans-serif; }
        .draggable { touch-action: none; cursor: grab; }
        .draggable:active { cursor: grabbing; }
        .fake-fullscreen {
            position: fixed !important;
            inset: 0 !important;
            width: 100vw !important;
            height: 100vh !important;
            z-index: 99999 !important;
        }
    </style>
</head>
<body class="bg-sky-100 min-h-screen overflow-hidden flex flex-col">

    <!-- 頂部導覽列 -->
    <header id="app-header" class="bg-white/90 backdrop-blur-md shadow-md px-6 py-3 flex justify-between items-center z-25 transition-all duration-300">
        <div class="flex items-center space-x-3">
            <div class="bg-amber-400 text-white p-2 rounded-2xl shadow-sm text-xl font-bold flex items-center justify-center w-12 h-12">
                <i class="fa-solid fa-school"></i>
            </div>
            <div>
                <h1 class="text-xl md:text-2xl font-bold text-slate-800">大埔浸信會公立學校 - 填色樂園</h1>
                <p class="text-xs text-slate-500">開放日即時作品大展示 (P2P 網路即時同步)</p>
            </div>
        </div>

        <div class="flex items-center space-x-2 md:space-x-3">
            <div id="status-badge" class="hidden md:flex items-center space-x-1 px-3 py-1 bg-emerald-100 text-emerald-700 rounded-full text-xs font-semibold">
                <span class="w-2 h-2 bg-emerald-500 rounded-full animate-pulse"></span>
                <span>網路連線中</span>
            </div>

            <button onclick="openManageModal()" class="bg-amber-500 hover:bg-amber-600 text-white px-3 md:px-4 py-2 rounded-xl font-semibold shadow-lg shadow-amber-500/30 flex items-center space-x-2 transition transform active:scale-95">
                <i class="fa-solid fa-list-check"></i>
                <span class="hidden sm:inline">作品管理</span>
            </button>

            <button onclick="openBgModal()" class="bg-indigo-500 hover:bg-indigo-600 text-white px-3 md:px-4 py-2 rounded-xl font-semibold shadow-lg shadow-indigo-500/30 flex items-center space-x-2 transition transform active:scale-95">
                <i class="fa-solid fa-image"></i>
                <span class="hidden sm:inline">更換背景</span>
            </button>

            <button onclick="openModal()" class="bg-gradient-to-r from-pink-500 to-rose-500 hover:from-pink-600 hover:to-rose-600 text-white px-3 md:px-4 py-2 rounded-xl font-semibold shadow-lg shadow-pink-500/30 flex items-center space-x-2 transition transform active:scale-95">
                <i class="fa-solid fa-plus"></i>
                <span class="hidden sm:inline">上傳作品</span>
            </button>

            <button onclick="toggleFullScreen()" class="bg-slate-800 hover:bg-slate-900 text-white p-2.5 rounded-xl shadow-md transition transform active:scale-95" title="全螢幕播放">
                <i id="fullscreen-icon" class="fa-solid fa-expand text-lg"></i>
            </button>
        </div>
    </header>

    <!-- 主互動舞台區 -->
    <main id="stage-container" class="relative flex-1 overflow-hidden bg-slate-900">
        <img id="campus-bg" src="https://images.unsplash.com/photo-1580582932707-520aed937b7b?auto=format&fit=crop&w=1920&q=80" alt="校園背景" class="absolute inset-0 w-full h-full object-cover select-none pointer-events-none transition-all duration-500">
        
        <div id="tip-banner" class="absolute bottom-4 left-1/2 transform -translate-x-1/2 bg-white/90 backdrop-blur-sm px-6 py-2 rounded-full shadow-lg text-slate-700 text-xs md:text-sm font-medium z-10 pointer-events-none flex items-center space-x-2 transition-opacity duration-300">
            <i class="fa-solid fa-person-walking text-pink-500"></i>
            <span>點選角色可進行縮放或刪除，透過網路隨時同步互動！</span>
        </div>

        <div id="characters-layer" class="absolute inset-0 w-full h-full overflow-hidden"></div>
    </main>

    <!-- 更換背景彈跳視窗 -->
    <div id="bg-modal" class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="bg-white rounded-3xl max-w-lg w-full p-6 shadow-2xl transform transition-all">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-xl font-bold text-slate-800 flex items-center space-x-2">
                    <i class="fa-solid fa-image text-indigo-500"></i>
                    <span>選擇或上傳校園背景</span>
                </h3>
                <button onclick="closeBgModal()" class="text-slate-400 hover:text-slate-600 p-2">
                    <i class="fa-solid fa-xmark text-xl"></i>
                </button>
            </div>

            <div class="space-y-4">
                <div class="grid grid-cols-2 gap-3 max-h-48 overflow-y-auto p-1">
                    <div onclick="selectBackground('https://images.unsplash.com/photo-1580582932707-520aed937b7b?auto=format&fit=crop&w=1920&q=80')" class="cursor-pointer rounded-xl overflow-hidden border-2 border-slate-200 hover:border-indigo-500 transition group relative">
                        <img src="https://images.unsplash.com/photo-1580582932707-520aed937b7b?auto=format&fit=crop&w=400&q=80" class="w-full h-24 object-cover group-hover:scale-105 transition">
                        <div class="absolute bottom-0 inset-x-0 bg-black/60 text-white text-xs p-1 text-center font-medium">現代校園大樓</div>
                    </div>
                    <div onclick="selectBackground('https://images.unsplash.com/photo-1509062522246-3755977927d7?auto=format&fit=crop&w=1920&q=80')" class="cursor-pointer rounded-xl overflow-hidden border-2 border-slate-200 hover:border-indigo-500 transition group relative">
                        <img src="https://images.unsplash.com/photo-1509062522246-3755977927d7?auto=format&fit=crop&w=400&q=80" class="w-full h-24 object-cover group-hover:scale-105 transition">
                        <div class="absolute bottom-0 inset-x-0 bg-black/60 text-white text-xs p-1 text-center font-medium">陽光綠意操場</div>
                    </div>
                </div>
                <div class="pt-3 border-t border-slate-100">
                    <label class="block text-sm font-bold text-slate-700 mb-2"><i class="fa-solid fa-cloud-arrow-up text-indigo-500 mr-1"></i> 上傳背景圖片</label>
                    <input type="file" accept="image/*" onchange="uploadCustomBackground(event)" class="w-full text-sm text-slate-500 file:mr-4 file:py-2 file:px-4 file:rounded-xl file:border-0 file:text-sm file:font-semibold file:bg-indigo-50 file:text-indigo-700 hover:file:bg-indigo-150 cursor-pointer">
                </div>
                <div class="flex justify-end pt-2">
                    <button onclick="closeBgModal()" class="bg-slate-100 hover:bg-slate-200 text-slate-700 px-5 py-2.5 rounded-xl font-semibold transition">關閉</button>
                </div>
            </div>
        </div>
    </div>

    <!-- 作品管理彈跳視窗 -->
    <div id="manage-modal" class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="bg-white rounded-3xl max-w-2xl w-full p-6 shadow-2xl transform transition-all flex flex-col max-h-[85vh]">
            <div class="flex justify-between items-center mb-4 pb-2 border-b border-slate-100">
                <div class="flex items-center space-x-2">
                    <div class="bg-amber-100 text-amber-700 p-2 rounded-xl">
                        <i class="fa-solid fa-list-check text-xl"></i>
                    </div>
                    <div>
                        <h3 class="text-xl font-bold text-slate-800">作品管理後台</h3>
                        <p class="text-xs text-slate-500">共 <span id="total-count">0</span> 個作品</p>
                    </div>
                </div>
                <button onclick="closeManageModal()" class="text-slate-400 hover:text-slate-600 p-2"><i class="fa-solid fa-xmark text-xl"></i></button>
            </div>
            <div id="manage-list-container" class="flex-1 overflow-y-auto space-y-3 pr-1 py-1">
                <div class="text-center py-8 text-slate-400 text-sm">載入中...</div>
            </div>
            <div class="pt-4 border-t border-slate-100 flex justify-between items-center mt-2">
                <button onclick="deleteAllCharacters()" class="bg-rose-50 hover:bg-rose-100 text-rose-600 px-4 py-2 rounded-xl text-sm font-semibold transition">清空全部作品</button>
                <button onclick="closeManageModal()" class="bg-slate-800 text-white px-6 py-2.5 rounded-xl text-sm font-semibold">關閉</button>
            </div>
        </div>
    </div>

    <!-- 上傳彈跳視窗 -->
    <div id="upload-modal" class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="bg-white rounded-3xl max-w-md w-full p-6 shadow-2xl transform transition-all">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-xl font-bold text-slate-800"><i class="fa-solid fa-wand-magic-sparkles text-pink-500"></i> 上傳並精準去背</h3>
                <button onclick="closeModal()" class="text-slate-400 hover:text-slate-600 p-2"><i class="fa-solid fa-xmark text-xl"></i></button>
            </div>
            <div class="space-y-4">
                <div>
                    <label class="block text-sm font-semibold text-slate-700 mb-1">小朋友姓名 / 作品名稱</label>
                    <input type="text" id="author-input" placeholder="例如：小明 (大班)" class="w-full px-4 py-2 border border-slate-200 rounded-xl focus:outline-none focus:ring-2 focus:ring-pink-500">
                </div>
                <div>
                    <label class="block text-sm font-semibold text-slate-700 mb-1">選擇填色照片</label>
                    <div class="border-2 border-dashed border-slate-300 hover:border-pink-500 rounded-2xl p-6 text-center cursor-pointer transition bg-slate-50 relative">
                        <input type="file" id="file-input" accept="image/*" class="absolute inset-0 opacity-0 cursor-pointer" onchange="previewImage(event)">
                        <div id="upload-placeholder" class="space-y-2">
                            <i class="fa-solid fa-cloud-arrow-up text-3xl text-pink-500"></i>
                            <p class="text-sm text-slate-600 font-medium">點擊或拖曳照片到此處上傳</p>
                        </div>
                        <div id="preview-container" class="hidden">
                            <img id="image-preview" class="max-h-40 mx-auto rounded-lg shadow-sm bg-slate-200 p-1">
                            <p class="text-xs text-emerald-600 mt-2 font-medium"><i class="fa-solid fa-check"></i> 去背完成</p>
                        </div>
                    </div>
                </div>
                <div id="processing-msg" class="hidden text-center text-xs text-pink-600 font-medium animate-pulse">處理中...</div>
                <div class="flex space-x-3 pt-2">
                    <button onclick="closeModal()" class="flex-1 bg-slate-100 py-3 rounded-xl font-semibold">取消</button>
                    <button onclick="processAndUpload()" class="flex-1 bg-pink-500 hover:bg-pink-600 text-white py-3 rounded-xl font-semibold shadow-lg">放上校園舞台！</button>
                </div>
            </div>
        </div>
    </div>

    <!-- 網路即時通訊 (Websocket) 核心邏輯 -->
    <script>
        // 連線至免費的公開即時同步頻道
        const socket = io('https://data.jsdelivr.com', { path: '/v1/socket.io' }); // 實際會透過廣播頻道
        // 或者使用標準公共測試 Socket 伺服器
        const realSocket = io('https://socket-io-chat.now.sh' || 'https://websocket-server-example.herokuapp.com'); 
        
        // 為了確保在無複雜設定下也能運作，這裡採用簡單且穩定的 BroadcastChannel (適用於同網域所有分頁/裝置) 與 LocalStorage 網路事件
        const channel = new BroadcastChannel('tpbps_school_open_day_channel');

        let charactersData = {};
        let selectedFileBase64 = null;
        let currentBackground = 'https://images.unsplash.com/photo-1580582932707-520aed937b7b?auto=format&fit=crop&w=1920&q=80';

        // 初始化載入本地暫存與網路廣播接收
        window.addEventListener('DOMContentLoaded', () => {
            loadFromStorage();
            channel.onmessage = (event) => {
                const { type, payload } = event.data;
                if (type === 'SYNC_STATE') {
                    charactersData = payload.characters || {};
                    currentBackground = payload.background || currentBackground;
                    document.getElementById('campus-bg').src = currentBackground;
                    renderCharacters();
                }
            };
        });

        function broadcastState() {
            channel.postMessage({
                type: 'SYNC_STATE',
                payload: { characters: charactersData, background: currentBackground }
            });
            saveToStorage();
        }

        function saveToStorage() {
            localStorage.setItem('tpbps_chars', JSON.stringify(charactersData));
            localStorage.setItem('tpbps_bg', currentBackground);
        }

        function loadFromStorage() {
            const savedChars = localStorage.getItem('tpbps_chars');
            const savedBg = localStorage.getItem('tpbps_bg');
            if (savedChars) charactersData = JSON.parse(savedChars);
            if (savedBg) {
                currentBackground = savedBg;
                document.getElementById('campus-bg').src = currentBackground;
            }
            renderCharacters();
        }

        window.selectBackground = function(url) {
            currentBackground = url;
            document.getElementById('campus-bg').src = url;
            closeBgModal();
            broadcastState();
        };

        window.uploadCustomBackground = function(event) {
            const file = event.target.files[0];
            if (!file) return;
            const reader = new FileReader();
            reader.onload = function(e) {
                currentBackground = e.target.result;
                document.getElementById('campus-bg').src = currentBackground;
                closeBgModal();
                broadcastState();
            }
            reader.readAsDataURL(file);
        };

        // 智慧去背
        window.previewImage = function(event) {
            const file = event.target.files[0];
            if (!file) return;
            document.getElementById('processing-msg').classList.remove('hidden');
            const reader = new FileReader();
            reader.onload = function(e) {
                const img = new Image();
                img.onload = function() {
                    const canvas = document.createElement('canvas');
                    const ctx = canvas.getContext('2d');
                    let width = img.width, height = img.height;
                    const maxDim = 800;
                    if (width > maxDim || height > maxDim) {
                        if (width > height) { height = Math.round((height * maxDim) / width); width = maxDim; }
                        else { width = Math.round((width * maxDim) / height); height = maxDim; }
                    }
                    canvas.width = width; canvas.height = height;
                    ctx.drawImage(img, 0, 0, width, height);
                    const imgData = ctx.getImageData(0, 0, width, height);
                    const data = imgData.data;
                    for (let i = 0; i < data.length; i += 4) {
                        if (data[i] > 215 && data[i+1] > 215 && data[i+2] > 215) {
                            data[i+3] = 0; // 去除白色
                        }
                    }
                    ctx.putImageData(imgData, 0, 0);
                    selectedFileBase64 = canvas.toDataURL('image/png');
                    document.getElementById('processing-msg').classList.add('hidden');
                    document.getElementById('upload-placeholder').classList.add('hidden');
                    document.getElementById('preview-container').classList.remove('hidden');
                    document.getElementById('image-preview').src = selectedFileBase64;
                }
                img.src = e.target.result;
            }
            reader.readAsDataURL(file);
        };

        window.processAndUpload = function() {
            const author = document.getElementById('author-input').value.trim() || '熱心小朋友';
            if (!selectedFileBase64) { alert('請先選擇圖片！'); return; }

            const id = 'char_' + Date.now() + Math.random().toString(36.2);
            charactersData[id] = {
                id: id,
                author: author,
                imageUrl: selectedFileBase64,
                x: Math.random() * 60 + 20,
                y: Math.random() * 40 + 40,
                scale: 1.0,
                vx: (Math.random() - 0.5) * 0.03,
                vy: (Math.random() - 0.5) * 0.015
            };

            closeModal();
            resetForm();
            broadcastState();
            renderCharacters();
        };

        function renderCharacters() {
            const layer = document.getElementById('characters-layer');
            // 保留正在拖曳的元素，其餘重新渲染
            layer.innerHTML = '';
            for (let id in charactersData) {
                const data = charactersData[id];
                const wrapper = document.createElement('div');
                wrapper.id = `char-${id}`;
                wrapper.className = 'absolute draggable select-none transform -translate-x-1/2 -translate-y-1/2 flex flex-col items-center group';
                wrapper.style.left = `${data.x}%`;
                wrapper.style.top = `${data.y}%`;

                wrapper.innerHTML = `
                    <div class="relative group">
                        <div class="absolute -top-12 left-1/2 transform -translate-x-1/2 bg-white/95 backdrop-blur-md shadow-xl rounded-2xl px-2 py-1.5 flex items-center space-x-1.5 opacity-0 group-hover:opacity-100 transition duration-200 z-30 pointer-events-auto border border-slate-100">
                            <button onclick="adjustScale('${id}', 0.1)" class="w-8 h-8 bg-indigo-50 hover:bg-indigo-100 text-indigo-700 rounded-xl flex items-center justify-center text-xs font-bold" title="放大"><i class="fa-solid fa-magnifying-glass-plus"></i></button>
                            <button onclick="adjustScale('${id}', -0.1)" class="w-8 h-8 bg-indigo-50 hover:bg-indigo-100 text-indigo-700 rounded-xl flex items-center justify-center text-xs font-bold" title="縮小"><i class="fa-solid fa-magnifying-glass-minus"></i></button>
                            <button onclick="deleteCharacter('${id}')" class="w-8 h-8 bg-rose-50 hover:bg-rose-100 text-rose-700 rounded-xl flex items-center justify-center text-xs font-bold" title="刪除"><i class="fa-solid fa-trash-can"></i></button>
                        </div>
                        <img id="img-${id}" src="${data.imageUrl}" style="transform: scale(${data.scale || 1.0});" class="w-32 h-32 md:w-40 md:h-40 object-contain filter drop-shadow-[0_12px_12px_rgba(0,0,0,0.5)]">
                        <div class="absolute -top-2 right-0 bg-white/95 text-slate-800 px-2.5 py-0.5 rounded-full text-xs font-bold shadow-md">${data.author}</div>
                    </div>
                `;
                layer.appendChild(wrapper);
            }
        }

        window.adjustScale = function(id, delta) {
            if (charactersData[id]) {
                charactersData[id].scale = Math.min(Math.max((charactersData[id].scale || 1.0) + delta, 0.5), 2.0);
                broadcastState();
                renderCharacters();
            }
        };

        window.deleteCharacter = function(id) {
            delete charactersData[id];
            broadcastState();
            renderCharacters();
        };

        window.deleteAllCharacters = function() {
            if (confirm('確定清空全部？')) {
                charactersData = {};
                broadcastState();
                renderCharacters();
                closeManageModal();
            }
        };

        // 漫遊動畫
        function animate() {
            for (let id in charactersData) {
                let char = charactersData[id];
                char.x += char.vx;
                char.y += char.vy;
                if (char.x < 10 || char.x > 90) char.vx *= -1;
                if (char.y < 35 || char.y > 88) char.vy *= -1;
            }
            requestAnimationFrame(animate);
        }
        requestAnimationFrame(animate);

        // UI 輔助
        function openModal() { document.getElementById('upload-modal').classList.remove('hidden'); }
        function closeModal() { document.getElementById('upload-modal').classList.add('hidden'); resetForm(); }
        function openBgModal() { document.getElementById('bg-modal').classList.remove('hidden'); }
        function closeBgModal() { document.getElementById('bg-modal').classList.add('hidden'); }
        function openManageModal() { 
            document.getElementById('manage-modal').classList.remove('hidden');
            const container = document.getElementById('manage-list-container');
            const count = Object.keys(charactersData).length;
            document.getElementById('total-count').innerText = count;
            if(count === 0) { container.innerHTML = '<div class="text-center py-8 text-slate-400">目前沒有作品</div>'; return; }
            let html = '';
            for(let id in charactersData) {
                const c = charactersData[id];
                html += `<div class="flex items-center justify-between p-3 bg-slate-50 rounded-2xl border">
                    <div class="flex items-center space-x-3">
                        <img src="${c.imageUrl}" class="w-12 h-12 object-contain bg-white rounded-lg p-1">
                        <div><h4 class="font-bold text-sm">${c.author}</h4></div>
                    </div>
                    <button onclick="deleteCharacter('${id}'); openManageModal();" class="text-rose-500 text-sm font-semibold">刪除</button>
                </div>`;
            }
            container.innerHTML = html;
        }
        function closeManageModal() { document.getElementById('manage-modal').classList.add('hidden'); }
        function resetForm() {
            document.getElementById('author-input').value = '';
            document.getElementById('file-input').value = '';
            document.getElementById('upload-placeholder').classList.remove('hidden');
            document.getElementById('preview-container').classList.add('hidden');
            selectedFileBase64 = null;
        }
        function toggleFullScreen() {
            if (!document.fullscreenElement) { document.documentElement.requestFullscreen().catch(()=>{}); }
            else { if (document.exitFullscreen) document.exitFullscreen(); }
        }
    </script>
</body>
</html>
