<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>AllianceBoard // FRC Strategic Dashboard</title>
    <style>
        :root {
            --bg-main: #0a0c10;
            --bg-panel: #121620;
            --border-neon: #1f293d;
            --neon-blue: #00f0ff;
            --neon-red: #ff0055;
            --neon-green: #39ff14;
            --neon-orange: #ff9900;
            --text-main: #e2e8f0;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; }
        
        body {
            background-color: var(--bg-main);
            color: var(--text-main);
            font-family: 'Segoe UI', -apple-system, BlinkMacSystemFont, Roboto, sans-serif;
            height: 100vh;
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }

        header {
            background: var(--bg-panel);
            border-bottom: 2px solid var(--border-neon);
            padding: 10px 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            box-shadow: 0 4px 20px rgba(0,0,0,0.5);
            z-index: 10;
        }

        .brand {
            font-size: 18px;
            font-weight: bold;
            letter-spacing: 2px;
            color: var(--neon-blue);
            text-shadow: 0 0 8px rgba(0, 240, 255, 0.4);
        }

        .room-controls {
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .room-input {
            background: #1e2538;
            border: 1px solid #334155;
            color: #fff;
            padding: 6px 12px;
            border-radius: 6px;
            width: 100px;
            text-align: center;
            font-weight: bold;
            text-transform: uppercase;
        }

        .room-status {
            font-size: 13px;
            background: rgba(255, 153, 0, 0.1);
            padding: 6px 12px;
            border: 1px solid var(--neon-orange);
            border-radius: 4px;
            color: var(--neon-orange);
            font-weight: bold;
        }
        .room-status.connected {
            background: rgba(57, 255, 20, 0.1);
            border-color: var(--neon-green);
            color: var(--neon-green);
        }

        .dashboard-container {
            flex: 1;
            display: flex;
            position: relative;
            height: calc(100vh - 55px);
        }

        .sidebar {
            width: 240px;
            background: var(--bg-panel);
            border-right: 2px solid var(--border-neon);
            padding: 15px;
            display: flex;
            flex-direction: column;
            gap: 20px;
            z-index: 5;
            overflow-y: auto;
        }

        .panel-section h3 {
            font-size: 11px;
            text-transform: uppercase;
            letter-spacing: 1px;
            color: #64748b;
            margin-bottom: 10px;
            border-bottom: 1px solid var(--border-neon);
            padding-bottom: 5px;
        }

        .grid-2 { display: grid; grid-template-columns: repeat(2, 1fr); gap: 8px; }
        .color-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 6px; }

        .btn-tool {
            background: #1e2538;
            border: 1px solid #334155;
            color: var(--text-main);
            padding: 8px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 13px;
            transition: all 0.2s;
        }
        .btn-tool:hover { background: #2e3954; border-color: var(--neon-blue); }
        .btn-tool.active {
            background: rgba(0, 240, 255, 0.15);
            border-color: var(--neon-blue);
            color: var(--neon-blue);
        }

        .color-dot {
            height: 32px; border-radius: 6px; cursor: pointer; border: 2px solid transparent;
            display: flex; align-items: center; justify-content: center;
        }
        .color-dot.active { border-color: #fff; }

        .token-bank { display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; }
        .robot-token {
            padding: 10px; border-radius: 8px; font-size: 12px; font-weight: bold;
            text-align: center; cursor: grab; touch-action: none;
        }
        
        .token-blue { background: rgba(0, 150, 255, 0.2); border: 2px solid var(--neon-blue); color: #fff; }
        .token-red { background: rgba(255, 0, 85, 0.2); border: 2px solid var(--neon-red); color: #fff; }
        .token-note { background: rgba(255, 153, 0, 0.2); border: 2px solid var(--neon-orange); color: #fff; }

        .stage { flex: 1; display: flex; justify-content: center; align-items: center; padding: 20px; position: relative; overflow: hidden; }

        .canvas-container {
            position: relative; background: #0f131a; border: 3px solid #1e293b; border-radius: 12px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.7);
            background-image: 
                linear-gradient(rgba(255,255,255,0.02) 1px, transparent 1px),
                linear-gradient(90deg, rgba(255,255,255,0.02) 1px, transparent 1px),
                linear-gradient(90deg, transparent 49.8%, rgba(255,255,255,0.15) 49.8%, rgba(255,255,255,0.15) 50.2%, transparent 50.2%);
            background-size: 20px 20px, 20px 20px, 100% 100%;
        }

        canvas { display: block; border-radius: 10px; position: absolute; top: 0; left: 0; z-index: 1; }
        .object-layer { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; z-index: 2; }

        .placed-token {
            position: absolute; pointer-events: auto; transform: translate(-50%, -50%);
            width: 40px; height: 40px; border-radius: 50%; display: flex; align-items: center; justify-content: center;
            font-size: 11px; font-weight: bold; box-shadow: 0 4px 10px rgba(0,0,0,0.5); cursor: move; touch-action: none;
        }

        .info-overlay { position: absolute; bottom: 15px; right: 20px; background: rgba(18, 22, 32, 0.85); border: 1px solid var(--border-neon); padding: 6px 12px; border-radius: 6px; font-size: 11px; color: #64748b; z-index: 4; }
    </style>
</head>
<body>

    <header>
        <div class="brand">ALLIANCEBOARD // STRAT-FRC</div>
        <div class="room-controls">
            <input type="text" id="roomInput" class="room-input" placeholder="ODA KODU" maxlength="5">
            <button class="btn-tool" id="btnJoinRoom" style="padding: 6px 12px;">Odaya Gir</button>
            <button class="btn-tool" id="btnCreateRoom" style="padding: 6px 12px; border-color: var(--neon-green); color: var(--neon-green);">Oda Aç</button>
            <div class="room-status" id="roomStatus">LOKAL MOD</div>
        </div>
    </header>

    <div class="dashboard-container">
        <aside class="sidebar">
            <div class="panel-section">
                <h3>Çizim Modları</h3>
                <div class="grid-2">
                    <button class="btn-tool active" id="toolPencil">Kalem</button>
                    <button class="btn-tool" id="toolEraser">Silgi</button>
                </div>
            </div>
            <div class="panel-section">
                <h3>Neon Renkler</h3>
                <div class="color-grid">
                    <div class="color-dot active" style="background: var(--neon-blue);" data-color="#00f0ff"></div>
                    <div class="color-dot" style="background: var(--neon-red);" data-color="#ff0055"></div>
                    <div class="color-dot" style="background: var(--neon-green);" data-color="#39ff14"></div>
                    <div class="color-dot" style="background: var(--neon-orange);" data-color="#ff9900"></div>
                </div>
            </div>
            <div class="panel-section">
                <h3>Fırça Kalınlığı</h3>
                <div style="display:flex; gap:10px; align-items:center;">
                    <input type="range" id="brushSize" min="2" max="12" value="4" style="flex:1; accent-color: var(--neon-blue);">
                    <span id="brushSizeVal" style="font-size:12px; width:20px;">4px</span>
                </div>
            </div>
            <div class="panel-section">
                <h3>Robot & Parça Ekle</h3>
                <div class="token-bank">
                    <div class="robot-token token-blue" draggable="true" data-class="token-blue">B1</div>
                    <div class="robot-token token-blue" draggable="true" data-class="token-blue">B2</div>
                    <div class="robot-token token-red" draggable="true" data-class="token-red">R1</div>
                    <div class="robot-token token-red" draggable="true" data-class="token-red">R2</div>
                    <div class="robot-token token-note" draggable="true" data-class="token-note">🔴</div>
                    <div class="robot-token token-note" draggable="true" data-class="token-note">🔵</div>
                </div>
            </div>
            <div class="panel-section" style="margin-top:auto;">
                <button class="btn-tool" id="btnClearAll" style="width:100%; border-color:rgba(255,0,0,0.3); color:#f87171;">Taktik Tahtasını Temizle</button>
            </div>
        </aside>

        <main class="stage">
            <div class="canvas-container" id="canvasContainer" style="width: 800px; height: 400px;">
                <div style="position:absolute; width:25%; height:100%; border-right:2px dashed rgba(255,0,85,0.2); top:0; left:0; pointer-events:none;"></div>
                <div style="position:absolute; width:25%; height:100%; border-left:2px dashed rgba(0,150,255,0.2); top:0; right:0; pointer-events:none;"></div>
                <canvas id="paintCanvas" width="800" height="400"></canvas>
                <div class="object-layer" id="objectLayer"></div>
            </div>
            <div class="info-overlay">Anlık Senkronizasyon Aktif</div>
        </main>
    </div>

    <!-- Modüler Firebase v10 Entegrasyonu -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
        import { getDatabase, ref, push, onChildAdded, onValue, set, update, remove } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js";

        // Senin Sağladığın Canlı Veritabanı Bilgileri
        const firebaseConfig = {
            apiKey: "AIzaSyCfjOdqRN--atZVs0cQOD4sPMCS4TdWhaA",
            authDomain: "taktiktahta.firebaseapp.com",
            databaseURL: "https://taktiktahta-default-rtdb.firebaseio.com",
            projectId: "taktiktahta",
            storageBucket: "taktiktahta.firebasestorage.app",
            messagingSenderId: "512624589369",
            appId: "1:512624589369:web:7768cc3d6b67c865216be8"
        };

        // Firebase Başlatma
        const app = initializeApp(firebaseConfig);
        const database = getDatabase(app);

        // --- DOM ELEMENTLERİ VE DEĞİŞKENLER ---
        const canvas = document.getElementById('paintCanvas');
        const ctx = canvas.getContext('2d');
        const container = document.getElementById('canvasContainer');
        const objectLayer = document.getElementById('objectLayer');
        
        let isDrawing = false;
        let currentTool = 'pencil';
        let currentColor = '#00f0ff';
        let currentSize = 4;
        let lastPos = {x: 0, y: 0};

        let currentRoomId = null;
        let lineUnsubscribe = null;
        let tokensUnsubscribe = null;
        let clearUnsubscribe = null;

        ctx.lineCap = 'round';
        ctx.lineJoin = 'round';

        // --- CANLI ODA KONTROLÜ ---
        function switchRoom(roomId) {
            // Varsa eski odanın dinleyicilerini kapat (Abonelikten çık)
            if (lineUnsubscribe) lineUnsubscribe();
            if (tokensUnsubscribe) tokensUnsubscribe();
            if (clearUnsubscribe) clearUnsubscribe();
            
            currentRoomId = roomId;
            document.getElementById('roomStatus').innerText = `ODA: #${roomId}`;
            document.getElementById('roomStatus').classList.add('connected');
            document.getElementById('roomInput').value = roomId;
            
            // Ekranları yeni oda için sıfırla
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            objectLayer.innerHTML = '';

            const linesRef = ref(database, `rooms/${roomId}/lines`);
            const tokensRef = ref(database, `rooms/${roomId}/tokens`);
            const clearRef = ref(database, `rooms/${roomId}/clear`);

            // 1. Çizgileri Dinle
            lineUnsubscribe = onChildAdded(linesRef, (snapshot) => {
                const line = snapshot.val();
                drawLineLocal(line.x1, line.y1, line.x2, line.y2, line.color, line.size, line.tool);
            });

            // 2. Robot Konumlarını Dinle
            tokensUnsubscribe = onValue(tokensRef, (snapshot) => {
                updateTokensLocal(snapshot.val());
            });

            // 3. Temizleme Komutunu Dinle
            clearUnsubscribe = onValue(clearRef, (snapshot) => {
                if(snapshot.val()) {
                    ctx.clearRect(0, 0, canvas.width, canvas.height);
                    objectLayer.innerHTML = '';
                }
            });
        }

        document.getElementById('btnCreateRoom').addEventListener('click', () => {
            const randomId = Math.floor(1000 + Math.random() * 9000).toString();
            switchRoom(randomId);
        });

        document.getElementById('btnJoinRoom').addEventListener('click', () => {
            const inputId = document.getElementById('roomInput').value.trim().toUpperCase();
            if(inputId) switchRoom(inputId);
        });

        // --- ÇIZIM MOTORU FONKSİYONLARI ---
        function getMousePos(e) {
            const rect = canvas.getBoundingClientRect();
            const clientX = e.touches ? e.touches[0].clientX : e.clientX;
            const clientY = e.touches ? e.touches[0].clientY : e.clientY;
            return { x: clientX - rect.left, y: clientY - rect.top };
        }

        function startDrawing(e) {
            isDrawing = true;
            lastPos = getMousePos(e);
            e.preventDefault();
        }

        function draw(e) {
            if (!isDrawing) return;
            const currentPos = getMousePos(e);

            if (currentRoomId) {
                push(ref(database, `rooms/${currentRoomId}/lines`), {
                    x1: lastPos.x, y1: lastPos.y,
                    x2: currentPos.x, y2: currentPos.y,
                    color: currentColor, size: currentSize, tool: currentTool
                });
                set(ref(database, `rooms/${currentRoomId}/clear`), false);
            } else {
                drawLineLocal(lastPos.x, lastPos.y, currentPos.x, currentPos.y, currentColor, currentSize, currentTool);
            }

            lastPos = currentPos;
            e.preventDefault();
        }

        function drawLineLocal(x1, y1, x2, y2, color, size, tool) {
            ctx.beginPath();
            ctx.moveTo(x1, y1);
            ctx.lineWidth = size;
            
            if (tool === 'eraser') {
                ctx.globalCompositeOperation = 'destination-out';
            } else {
                ctx.globalCompositeOperation = 'source-over';
                ctx.strokeStyle = color;
                ctx.shadowBlur = 3;
                ctx.shadowColor = color;
            }
            
            ctx.lineTo(x2, y2);
            ctx.stroke();
            ctx.shadowBlur = 0;
        }

        canvas.addEventListener('mousedown', startDrawing);
        canvas.addEventListener('mousemove', draw);
        window.addEventListener('mouseup', () => isDrawing = false);
        canvas.addEventListener('touchstart', startDrawing, {passive: false});
        canvas.addEventListener('touchmove', draw, {passive: false});
        window.addEventListener('touchend', () => isDrawing = false);

        // --- DRAG & DROP & TOKEN KONTROLLERİ ---
        let draggedText = '';
        let draggedClass = '';

        document.querySelectorAll('.robot-token').forEach(token => {
            token.addEventListener('dragstart', () => {
                draggedText = token.innerText;
                draggedClass = token.getAttribute('data-class');
            });
            token.addEventListener('touchstart', () => {
                draggedText = token.innerText;
                draggedClass = token.getAttribute('data-class');
            }, {passive: true});
        });

        container.addEventListener('dragover', (e) => e.preventDefault());
        container.addEventListener('drop', (e) => {
            e.preventDefault();
            const rect = container.getBoundingClientRect();
            const x = e.clientX - rect.left;
            const y = e.clientY - rect.top;
            addTokenRequest(x, y, draggedText, draggedClass);
        });

        document.querySelectorAll('.robot-token').forEach(token => {
            token.addEventListener('touchend', () => {
                const rect = container.getBoundingClientRect();
                addTokenRequest(rect.width/2 + (Math.random()*40-20), rect.height/2 + (Math.random()*40-20), draggedText, draggedClass);
            });
        });

        function addTokenRequest(x, y, text, customClass) {
            const tokenId = 'tk_' + Date.now() + '_' + Math.floor(Math.random()*1000);
            if (currentRoomId) {
                set(ref(database, `rooms/${currentRoomId}/tokens/${tokenId}`), { x, y, text, customClass });
            } else {
                createLocalTokenElement(tokenId, x, y, text, customClass);
            }
        }

        function updateTokensLocal(tokensData) {
            objectLayer.innerHTML = '';
            if(!tokensData) return;

            Object.keys(tokensData).forEach(id => {
                const t = tokensData[id];
                createLocalTokenElement(id, t.x, t.y, t.text, t.customClass);
            });
        }

        function createLocalTokenElement(id, x, y, text, customClass) {
            const tokenEl = document.createElement('div');
            tokenEl.className = `placed-token ${customClass}`;
            tokenEl.innerText = text;
            tokenEl.style.left = `${x}px`;
            tokenEl.style.top = `${y}px`;

            tokenEl.addEventListener('pointerdown', (e) => {
                tokenEl.setPointerCapture(e.pointerId);
                e.stopPropagation();
                
                function onPointerMove(ev) {
                    const rect = container.getBoundingClientRect();
                    let nx = ev.clientX - rect.left;
                    let ny = ev.clientY - rect.top;

                    if(currentRoomId) {
                        update(ref(database, `rooms/${currentRoomId}/tokens/${id}`), { x: nx, y: ny });
                    } else {
                        tokenEl.style.left = `${nx}px`;
                        tokenEl.style.top = `${ny}px`;
                    }
                }

                function onPointerUp() {
                    tokenEl.removeEventListener('pointermove', onPointerMove);
                    tokenEl.removeEventListener('pointerup', onPointerUp);
                }

                tokenEl.addEventListener('pointermove', onPointerMove);
                tokenEl.addEventListener('pointerup', onPointerUp);
            });

            const removeAction = () => {
                if(currentRoomId) {
                    remove(ref(database, `rooms/${currentRoomId}/tokens/${id}`));
                } else {
                    tokenEl.remove();
                }
            };
            tokenEl.addEventListener('dblclick', removeAction);
            tokenEl.addEventListener('touchstart', (e) => {
                if (tokenEl.dataset.lastTouch && (Date.now() - tokenEl.dataset.lastTouch < 300)) removeAction();
                tokenEl.dataset.lastTouch = Date.now();
            });

            objectLayer.appendChild(tokenEl);
        }

        // --- ARAYÜZ ETKİLEŞİMLERİ ---
        document.getElementById('toolPencil').addEventListener('click', () => { currentTool = 'pencil'; toggleToolBtn('toolPencil'); });
        document.getElementById('toolEraser').addEventListener('click', () => { currentTool = 'eraser'; toggleToolBtn('toolEraser'); });
        
        function toggleToolBtn(id) {
            document.getElementById('toolPencil').classList.remove('active');
            document.getElementById('toolEraser').classList.remove('active');
            document.getElementById(id).classList.add('active');
        }

        document.querySelectorAll('.color-dot').forEach(dot => {
            dot.addEventListener('click', () => {
                document.querySelectorAll('.color-dot').forEach(d => d.classList.remove('active'));
                dot.classList.add('active');
                currentColor = dot.getAttribute('data-color');
                currentTool = 'pencil'; toggleToolBtn('toolPencil');
            });
        });

        const brushSlider = document.getElementById('brushSize');
        brushSlider.addEventListener('input', (e) => {
            currentSize = e.target.value;
            document.getElementById('brushSizeVal').innerText = currentSize + 'px';
        });

        document.getElementById('btnClearAll').addEventListener('click', () => {
            if(confirm('Tüm taktik tahtasını sıfırlamak istediğinden emin misin?')) {
                if (currentRoomId) {
                    remove(ref(database, `rooms/${currentRoomId}/lines`));
                    remove(ref(database, `rooms/${currentRoomId}/tokens`));
                    set(ref(database, `rooms/${currentRoomId}/clear`), true);
                } else {
                    ctx.clearRect(0, 0, canvas.width, canvas.height);
                    objectLayer.innerHTML = '';
                }
            }
        });
    </script>
</body>
</html>
