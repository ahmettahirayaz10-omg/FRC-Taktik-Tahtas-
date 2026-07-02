# FRC-Taktik-Tahtas-<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>AllianceBoard // FRC Strategic Dashboard</title>
    <style>
        /* Cyberpunk Renk Paleti ve Genel Ayarlar */
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

        /* Üst Bar Kontrolleri */
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

        .room-status {
            font-size: 12px;
            background: rgba(0, 240, 255, 0.1);
            padding: 4px 10px;
            border: 1px solid var(--neon-blue);
            border-radius: 4px;
            color: var(--neon-blue);
        }

        /* Ana Çalışma Alanı Layout */
        .dashboard-container {
            flex: 1;
            display: flex;
            position: relative;
            height: calc(100vh - 55px);
        }

        /* Sol Araç Paneli */
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

        /* Çizim Araçları Butonları */
        .tool-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 8px; }
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
            box-shadow: 0 0 10px rgba(0,240,255,0.2);
        }

        .color-dot {
            height: 32px; border-radius: 6px; cursor: pointer; border: 2px solid transparent;
            transition: transform 0.1s; display: flex; align-items: center; justify-content: center;
        }
        .color-dot:hover { transform: scale(1.1); }
        .color-dot.active { border-color: #fff; transform: scale(1.05); }

        /* Sürüklenebilir Robot Tokenları */
        .token-bank {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 10px;
        }

        .robot-token {
            padding: 10px;
            border-radius: 8px;
            font-size: 12px;
            font-weight: bold;
            text-align: center;
            cursor: grab;
            position: relative;
            touch-action: none; /* Mobil kaydırmayı engellemek için önemli */
        }
        .robot-token:active { cursor: grabbing; }
        
        .token-blue { background: rgba(0, 150, 255, 0.2); border: 2px solid var(--neon-blue); color: #hd; text-shadow: 0 0 5px var(--neon-blue); }
        .token-red { background: rgba(255, 0, 85, 0.2); border: 2px solid var(--neon-red); color: #fff; text-shadow: 0 0 5px var(--neon-red); }
        .token-note { background: rgba(255, 153, 0, 0.2); border: 2px solid var(--neon-orange); color: #fff; }

        /* Merkez Taktik Sahnesi */
        .stage {
            flex: 1;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
            position: relative;
            overflow: hidden;
        }

        .canvas-container {
            position: relative;
            background: #0f131a;
            border: 3px solid #1e293b;
            border-radius: 12px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.7);
            /* FRC Saha Oranı yakalanacak şekilde arka plan çizgileri */
            background-image: 
                linear-gradient(rgba(255,255,255,0.02) 1px, transparent 1px),
                linear-gradient(90deg, rgba(255,255,255,0.02) 1px, transparent 1px),
                /* Merkez Çizgisi */
                linear-gradient(90deg, transparent 49.8%, rgba(255,255,255,0.15) 49.8%, rgba(255,255,255,0.15) 50.2%, transparent 50.2%);
            background-size: 20px 20px, 20px 20px, 100% 100%;
        }

        /* Dinamik Çizim Katmanı */
        canvas {
            display: block;
            border-radius: 10px;
            position: absolute;
            top: 0; left: 0;
            z-index: 1;
        }

        /* Sahanın üzerine yerleşen hareketli objelerin container katmanı */
        .object-layer {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            pointer-events: none; /* Çizimi engellemesin */
            z-index: 2;
        }

        .placed-token {
            position: absolute;
            pointer-events: auto; /* Sadece token tıklanabilir olsun */
            transform: translate(-50%, -50%);
            width: 40px; height: 40px;
            border-radius: 50%;
            display: flex; align-items: center; justify-content: center;
            font-size: 11px; font-weight: bold;
            box-shadow: 0 4px 10px rgba(0,0,0,0.5);
            cursor: move;
            touch-action: none;
        }

        /* Sağ Alttaki Bilgi Paneli */
        .info-overlay {
            position: absolute; bottom: 15px; right: 20px;
            background: rgba(18, 22, 32, 0.85); border: 1px solid var(--border-neon);
            backdrop-filter: blur(5px); padding: 6px 12px; border-radius: 6px;
            font-size: 11px; color: #64748b; z-index: 4; pointer-events: none;
        }
    </style>
</head>
<body>

    <header>
        <div class="brand">ALLIANCEBOARD // STRAT-FRC</div>
        <div class="room-status" id="connectionStatus">LOCAL MODE (PHASE 1)</div>
    </header>

    <div class="dashboard-container">
        
        <!-- SOL PANEL -->
        <aside class="sidebar">
            <div class="panel-section">
                <h3>Çizim Modları</h3>
                <div class="tool-grid">
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
                <div style="display:flex; gap:10px; align-items:center; margin-top:5px;">
                    <input type="range" id="brushSize" min="2" max="12" value="4" style="flex:1; accent-color: var(--neon-blue);">
                    <span id="brushSizeVal" style="font-size:12px; width:20px;">4px</span>
                </div>
            </div>

            <div class="panel-section">
                <h3>Robot & Parça Ekle</h3>
                <p style="font-size:11px; color:#64748b; margin-bottom:8px;">Sahaya sürükle bırak yapın:</p>
                <div class="token-bank">
                    <div class="robot-token token-blue" draggable="true" data-type="R1">B1</div>
                    <div class="robot-token token-blue" draggable="true" data-type="R2">B2</div>
                    <div class="robot-token token-red" draggable="true" data-type="E1">R1</div>
                    <div class="robot-token token-red" draggable="true" data-type="E2">R2</div>
                    <div class="robot-token token-note" draggable="true" data-type="NOTE">🔴</div>
                    <div class="robot-token token-note" draggable="true" data-type="NOTE">🔵</div>
                </div>
            </div>

            <div class="panel-section" style="margin-top:auto;">
                <button class="btn-tool" id="btnClearAll" style="width:100%; border-color:rgba(255,0,0,0.3); color:#f87171;">Taktik Tahtasını Temizle</button>
            </div>
        </aside>

        <!-- MERKEZ SAHA ALANI -->
        <main class="stage">
            <div class="canvas-container" id="canvasContainer" style="width: 800px; height: 400px;">
                <!-- Vektörel FRC Bölge Çizgileri Mockup'ı -->
                <div style="position:absolute; width:25%; height:100%; border-right:2px dashed rgba(255,0,85,0.2); top:0; left:0; pointer-events:none;"></div>
                <div style="position:absolute; width:25%; height:100%; border-left:2px dashed rgba(0,150,255,0.2); top:0; right:0; pointer-events:none;"></div>
                
                <!-- Çizim Tuvali -->
                <canvas id="paintCanvas" width="800" height="400"></canvas>
                
                <!-- Sürüklenebilir Objeler Katmanı -->
                <div class="object-layer" id="objectLayer"></div>
            </div>

            <div class="info-overlay">Ekran Boyutu: 800x400 (Saha Ölçeği)</div>
        </main>

    </div>

    <script>
        const canvas = document.getElementById('paintCanvas');
        const ctx = canvas.getContext('2d');
        const container = document.getElementById('canvasContainer');
        const objectLayer = document.getElementById('objectLayer');
        
        let isDrawing = false;
        let currentTool = 'pencil'; // pencil, eraser
        let currentColor = '#00f0ff';
        let currentSize = 4;

        // --- ENTEGRE EDİLMİŞ GELİŞMİŞ ÇİZİM MOTORU ---
        ctx.lineCap = 'round';
        ctx.lineJoin = 'round';

        function getMousePos(e) {
            const rect = canvas.getBoundingClientRect();
            // Hem mouse hem de mobil touch event koordinatlarını doğru hesaplama yapısı
            const clientX = e.touches ? e.touches[0].clientX : e.clientX;
            const clientY = e.touches ? e.touches[0].clientY : e.clientY;
            return {
                x: clientX - rect.left,
                y: clientY - rect.top
            };
        }

        function startDrawing(e) {
            isDrawing = true;
            const pos = getMousePos(e);
            ctx.beginPath();
            ctx.moveTo(pos.x, pos.y);
            e.preventDefault();
        }

        function draw(e) {
            if (!isDrawing) return;
            const pos = getMousePos(e);
            
            ctx.lineWidth = currentSize;
            if (currentTool === 'eraser') {
                ctx.globalCompositeOperation = 'destination-out'; // Canvas piksellerini silme modu
            } else {
                ctx.globalCompositeOperation = 'source-over';
                ctx.strokeStyle = currentColor;
                ctx.shadowBlur = 4; // Siberpunk parlama efekti
                ctx.shadowColor = currentColor;
            }

            ctx.lineTo(pos.x, pos.y);
            ctx.stroke();
            e.preventDefault();
        }

        function stopDrawing() {
            isDrawing = false;
            ctx.shadowBlur = 0; // Performans için çizim bittiğinde parlamayı sıfırla
        }

        // Mouse Dinleyicileri
        canvas.addEventListener('mousedown', startDrawing);
        canvas.addEventListener('mousemove', draw);
        window.addEventListener('mouseup', stopDrawing);

        // Dokunmatik Ekran (Mobil/Tablet) Dinleyicileri
        canvas.addEventListener('touchstart', startDrawing, {passive: false});
        canvas.addEventListener('touchmove', draw, {passive: false});
        window.addEventListener('touchend', stopDrawing);

        // --- PANEL BUTON KONTROLLERİ ---
        document.getElementById('toolPencil').addEventListener('click', (e) => {
            currentTool = 'pencil';
            setActiveButton('toolPencil');
        });

        document.getElementById('toolEraser').addEventListener('click', (e) => {
            currentTool = 'eraser';
            setActiveButton('toolEraser');
        });

        function setActiveButton(id) {
            document.getElementById('toolPencil').classList.remove('active');
            document.getElementById('toolEraser').classList.remove('active');
            document.getElementById(id).classList.add('active');
        }

        // Renk Seçimi
        document.querySelectorAll('.color-dot').forEach(dot => {
            dot.addEventListener('click', (e) => {
                document.querySelectorAll('.color-dot').forEach(d => d.classList.remove('active'));
                dot.classList.add('active');
                currentColor = dot.getAttribute('data-color');
                currentTool = 'pencil';
                setActiveButton('toolPencil');
            });
        });

        // Kalınlık Ayarı
        const brushSlider = document.getElementById('brushSize');
        const brushValText = document.getElementById('brushSizeVal');
        brushSlider.addEventListener('input', (e) => {
            currentSize = e.target.value;
            brushValText.innerText = currentSize + 'px';
        });

        // Temizleme Sistemi
        document.getElementById('btnClearAll').addEventListener('click', () => {
            if(confirm('Tüm çizimleri ve rotaları temizlemek istediğinden emin misin?')) {
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                objectLayer.innerHTML = ''; // Robotları da kaldırır
            }
        });

        // --- MOBİL UYUMLU GELİŞMİŞ DRAG & DROP SİSTEMİ ---
        // Native HTML5 Drag-Drop mobilde çalışmadığı için özel pointer takibi yazdım
        let draggedType = null;
        let draggedText = '';
        let draggedClass = '';

        document.querySelectorAll('.robot-token').forEach(token => {
            token.addEventListener('dragstart', (e) => {
                draggedType = token.getAttribute('data-type');
                draggedText = token.innerText;
                draggedClass = token.className.replace('robot-token ', '');
            });

            // Mobil için touchstart yakalaması
            token.addEventListener('touchstart', (e) => {
                draggedType = token.getAttribute('data-type');
                draggedText = token.innerText;
                draggedClass = token.className.replace('robot-token ', '');
            }, {passive: true});
        });

        container.addEventListener('dragover', (e) => e.preventDefault());

        container.addEventListener('drop', (e) => {
            e.preventDefault();
            const rect = container.getBoundingClientRect();
            const x = e.clientX - rect.left;
            const y = e.clientY - rect.top;
            createPlacedToken(x, y, draggedText, draggedClass);
        });

        // Mobil için bırakma simülasyonu (ekranın ortasına veya son dokunulan yere atar)
        document.querySelectorAll('.robot-token').forEach(token => {
            token.addEventListener('touchend', (e) => {
                const rect = container.getBoundingClientRect();
                // Basitlik adına ekranın merkezine yakın bir noktaya veya rastgele fırlatırız
                const x = rect.width / 2 + (Math.random() * 60 - 30);
                const y = rect.height / 2 + (Math.random() * 60 - 30);
                createPlacedToken(x, y, draggedText, draggedClass);
            });
        });

        function createPlacedToken(x, y, text, customClass) {
            const tokenEl = document.createElement('div');
            tokenEl.className = `placed-token ${customClass}`;
            tokenEl.innerText = text;
            tokenEl.style.left = `${x}px`;
            tokenEl.style.top = `${y}px`;

            // Sahadaki tokenı tekrar hareket ettirme mekaniği
            tokenEl.addEventListener('pointerdown', (e) => {
                tokenEl.setPointerCapture(e.pointerId);
                
                function onPointerMove(ev) {
                    const rect = container.getBoundingClientRect();
                    let nx = ev.clientX - rect.left;
                    let ny = ev.clientY - rect.top;
                    
                    // Sınır koruması (Sahanın dışına kaçmasınlar)
                    if(nx < 0) nx = 0; if(nx > rect.width) nx = rect.width;
                    if(ny < 0) ny = 0; if(ny > rect.height) ny = rect.height;

                    tokenEl.style.left = `${nx}px`;
                    tokenEl.style.top = `${ny}px`;
                }

                function onPointerUp(ev) {
                    tokenEl.removeEventListener('pointermove', onPointerMove);
                    tokenEl.removeEventListener('pointerup', onPointerUp);
                }

                tokenEl.addEventListener('pointermove', onPointerMove);
                tokenEl.addEventListener('pointerup', onPointerUp);
            });

            // Çift tıklamayla sahada kalabalık yapan robotu geri silme özelliği
            tokenEl.addEventListener('dblclick', () => tokenEl.remove());
            tokenEl.addEventListener('touchstart', (e) => {
                // Mobil için çift dokunma silmesi kontrolü
                if (tokenEl.dataset.lastTouch && (Date.now() - tokenEl.dataset.lastTouch < 300)) {
                    tokenEl.remove();
                }
                tokenEl.dataset.lastTouch = Date.now();
            });

            objectLayer.appendChild(tokenEl);
        }
    </script>
</body>
</html>
