
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>G-Monitor Telemetry Pro v3.2</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; }
        body { 
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; 
            background: #0a0a0c; 
            color: #fff; 
            display: flex; 
            flex-direction: column; 
            height: 100dvh; 
            overflow: hidden; 
        }

        /* ヘッダー情報 */
        .header {
            background: #141721;
            padding: 6px 10px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-bottom: 2px solid #232838;
            font-size: 0.75rem;
            font-family: monospace;
            flex-shrink: 0;
        }
        .header-item { display: flex; align-items: center; gap: 4px; color: #00e5ff; }

        /* カスタム設定バー */
        .config-bar {
            background: #1e2436;
            padding: 6px 10px;
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 6px;
            border-bottom: 1px solid #2a354d;
            font-size: 0.75rem;
            flex-shrink: 0;
        }
        .config-group { display: flex; align-items: center; justify-content: space-between; gap: 4px; }
        .config-group select, .config-group label {
            background: #141721;
            color: #00ff66;
            border: 1px solid #00e5ff;
            border-radius: 4px;
            padding: 3px 6px;
            font-weight: bold;
            outline: none;
            font-size: 0.75rem;
            flex: 1;
            text-align: center;
            cursor: pointer;
        }
        .config-group label.upload-lbl {
            background: #2563eb;
            color: #fff;
            border-color: #3b82f6;
        }
        .config-group button.clear-btn {
            background: #334155;
            color: #cbd5e1;
            border: 1px solid #475569;
            border-radius: 4px;
            padding: 3px 6px;
            font-weight: bold;
            font-size: 0.75rem;
            cursor: pointer;
            flex: 1;
        }
        #car-image-input { display: none; }

        /* キャンバスエリア */
        #canvas-container { 
            position: relative; 
            flex: 1; 
            min-height: 180px;
            background: radial-gradient(circle, #161922 0%, #0d0f14 100%);
            display: flex;
            justify-content: center;
            align-items: center;
            overflow: hidden; 
        }
        canvas { display: block; width: 100%; height: 100%; }

        /* ダッシュボード（数値パネル） */
        .dash-panel { 
            background: #141721; 
            padding: 6px 10px; 
            display: grid; 
            grid-template-columns: repeat(3, 1fr); 
            gap: 6px; 
            border-top: 2px solid #232838; 
            flex-shrink: 0;
        }
        .val-box { 
            background: #1e2436; 
            padding: 4px; 
            border-radius: 6px; 
            text-align: center; 
            border: 1px solid #2a354d;
        }
        .val-label { font-size: 0.55rem; color: #8892b0; margin-bottom: 1px; }
        .val-num { font-size: 0.95rem; font-weight: bold; font-family: monospace; color: #fff; }
        .val-num.green { color: #00ff66; }
        .val-num.yellow { color: #f1c40f; }

        /* 操作ボタン群 */
        .control-panel {
            background: #141721;
            padding: 6px 10px 10px 10px;
            display: flex;
            gap: 6px;
            flex-shrink: 0;
        }
        button.action-btn { 
            flex: 1;
            border: none; 
            border-radius: 6px; 
            color: white; 
            padding: 8px; 
            font-size: 0.8rem; 
            font-weight: bold; 
            display: flex; 
            align-items: center; 
            justify-content: center; 
            gap: 4px; 
            cursor: pointer; 
            background: #1e2436;
            border: 1px solid #00e5ff;
            transition: all 0.2s;
        }
        button.action-btn:active { background: #00e5ff; color: #000; }
        button.reset-btn { border-color: #ff3366; }
        button.reset-btn:active { background: #ff3366; color: #fff; }
    </style>
</head>
<body>

    <!-- ステータスヘッダー -->
    <div class="header">
        <div class="header-item"><i class="fas fa-satellite-dish"></i> <span id="gps-speed">0.0 km/h</span></div>
        <div class="header-item" style="color: #ffaa00;"><i class="fas fa-circle" id="status-dot"></i> <span id="status-text">タップして開始</span></div>
        <div class="header-item"><i class="fas fa-compass"></i> <span id="max-g-label">最大: 0.00G</span></div>
    </div>

    <!-- カスタム設定バー -->
    <div class="config-bar">
        <div class="config-group">
            <select id="vehicle-select">
                <option value="crown">👑 クラウンスポーツ</option>
                <option value="truck">🚛 大型トラック</option>
                <option value="sedan">🚗 セダン</option>
                <option value="minivan">🚐 ミニバン</option>
                <option value="kei">🚙 軽自動車</option>
            </select>
        </div>
        <div class="config-group">
            <select id="angle-select">
                <option value="rear">🔙 リヤビュー (後方)</option>
                <option value="front">🔜 フロントビュー (前方)</option>
                <option value="side">↔️ サイドビュー (真横)</option>
            </select>
        </div>
        <div class="config-group" style="grid-column: span 2;">
            <label for="car-image-input" class="upload-lbl"><i class="fas fa-camera"></i> 愛車の写メ</label>
            <input type="file" id="car-image-input" accept="image/*">
            <button id="btn-clear-img" class="clear-btn"><i class="fas fa-undo"></i> 標準に戻す</button>
        </div>
    </div>

    <!-- Gサークル描画エリア -->
    <div id="canvas-container">
        <canvas id="gCanvas"></canvas>
    </div>

    <!-- リアルタイム数値ダッシュボード -->
    <div class="dash-panel">
        <div class="val-box">
            <div class="val-label">現在のG</div>
            <div id="curr-g" class="val-num green">0.00</div>
        </div>
        <div class="val-box">
            <div class="val-label">前後G (加減速)</div>
            <div id="y-g" class="val-num yellow">0.00</div>
        </div>
        <div class="val-box">
            <div class="val-label">左右G (旋回)</div>
            <div id="x-g" class="val-num yellow">0.00</div>
        </div>
    </div>

    <!-- 操作ボタン -->
    <div class="control-panel">
        <button id="btn-calib" class="action-btn"><i class="fas fa-play"></i> センサー有効化</button>
        <button id="btn-reset" class="action-btn reset-btn"><i class="fas fa-trash-alt"></i> ピークリセット</button>
    </div>

    <script>
        const canvas = document.getElementById('gCanvas');
        const ctx = canvas.getContext('2d');
        const container = document.getElementById('canvas-container');

        const currGEl = document.getElementById('curr-g');
        const yGEl = document.getElementById('y-g');
        const xGEl = document.getElementById('x-g');
        const gpsSpeedEl = document.getElementById('gps-speed');
        const statusText = document.getElementById('status-text');
        const statusDot = document.getElementById('status-dot');
        const maxGLabel = document.getElementById('max-g-label');
        const vehicleSelect = document.getElementById('vehicle-select');
        const angleSelect = document.getElementById('angle-select');
        const carImageInput = document.getElementById('car-image-input');
        const btnClearImg = document.getElementById('btn-clear-img');

        const btnCalib = document.getElementById('btn-calib');
        const btnReset = document.getElementById('btn-reset');

        let width, height, centerX, centerY, radius;

        let rawX = 0, rawY = 0, rawZ = 0;
        let offsetX = 0, offsetY = 0;
        const gSens = 9.80665;

        let peakG = 0;
        const history = [];
        const maxHistory = 15;

        let customCarImage = null;

        // 車種×アングルごとの専用アイコン（文字・矢印を完全に排除し、車の向き・見え方に特化）
        const vehicleIcons = {
            crown: { rear: '🚘', front: '🏎️', side: '🚙' }, // リヤ、フロント、サイドそれぞれの見え方に合わせたアイコン
            truck: { rear: '🚛', front: '🚚', side: '🚛' },
            sedan: { rear: '🚘', front: '🚗', side: '🚗' },
            minivan: { rear: '🚐', front: '🚙', side: '🚐' },
            kei: { rear: '🛺', front: '🚙', side: '🛺' }
        };

        carImageInput.addEventListener('change', (e) => {
            const file = e.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = (event) => {
                    const img = new Image();
                    img.onload = () => {
                        customCarImage = img;
                    };
                    img.src = event.target.result;
                };
                reader.readAsDataURL(file);
            }
        });

        btnClearImg.addEventListener('click', () => {
            customCarImage = null;
            carImageInput.value = '';
        });

        function resizeCanvas() {
            width = container.clientWidth || 300;
            height = container.clientHeight || 200;
            canvas.width = width;
            canvas.height = height;
            centerX = width / 2;
            centerY = height / 2;
            radius = Math.min(centerX, centerY) - 25;
        }

        window.addEventListener('resize', resizeCanvas);
        setTimeout(resizeCanvas, 100);

        function initSensors() {
            window.addEventListener('devicemotion', (e) => {
                const acc = e.accelerationIncludingGravity;
                if (acc && (acc.x !== null || acc.y !== null)) {
                    rawX = acc.x || 0;
                    rawY = acc.y || 0;
                    rawZ = acc.z || 0;
                    statusText.textContent = '計測中';
                    statusDot.style.color = '#00ff66';
                }
            });

            if (navigator.geolocation) {
                navigator.geolocation.watchPosition(
                    (pos) => {
                        const speedKmh = (pos.coords.speed || 0) * 3.6;
                        gpsSpeedEl.textContent = `${speedKmh.toFixed(1)} km/h`;
                    },
                    (err) => {},
                    { enableHighAccuracy: true, maximumAge: 0 }
                );
            }
        }

        btnCalib.addEventListener('click', async () => {
            if (typeof DeviceMotionEvent !== 'undefined' && typeof DeviceMotionEvent.requestPermission === 'function') {
                try {
                    const response = await DeviceMotionEvent.requestPermission();
                    if (response !== 'granted') {
                        alert('センサーのアクセスが拒否されました');
                        return;
                    }
                } catch(e) {}
            }

            offsetX = rawX;
            offsetY = rawY;
            peakG = 0;
            history.length = 0;
            statusText.textContent = '補正完了・計測中';
            statusDot.style.color = '#00ff66';
            setTimeout(() => {
                if(statusText.textContent === '補正完了・計測中') statusText.textContent = '計測中';
            }, 1500);
        });

        btnReset.addEventListener('click', () => {
            peakG = 0;
            maxGLabel.textContent = `最大: 0.00G`;
            history.length = 0;
        });

        function render() {
            if (canvas.width !== container.clientWidth || canvas.height !== container.clientHeight) {
                resizeCanvas();
            }

            ctx.clearRect(0, 0, width, height);

            [0.5, 1.0, 1.5].forEach(g => {
                const r = radius * (g / 1.0);
                ctx.beginPath();
                ctx.arc(centerX, centerY, r, 0, Math.PI * 2);
                ctx.strokeStyle = g === 1.0 ? '#2a354d' : '#1a2233';
                ctx.lineWidth = g === 1.0 ? 2 : 1;
                ctx.setLineDash(g === 1.0 ? [] : [4, 4]);
                ctx.stroke();
            });
            ctx.setLineDash([]);

            ctx.beginPath();
            ctx.moveTo(centerX, 15); ctx.lineTo(centerX, height - 15);
            ctx.moveTo(15, centerY); ctx.lineTo(width - 15, centerY);
            ctx.strokeStyle = '#222b3d';
            ctx.lineWidth = 1.5;
            ctx.stroke();

            ctx.fillStyle = '#4a5568';
            ctx.font = '11px monospace';
            ctx.fillText('前 (減速)', centerX - 25, 12);
            ctx.fillText('後 (加速)', centerX - 25, height - 5);
            ctx.fillText('左', 5, centerY + 4);
            ctx.fillText('右', width - 15, centerY + 4);

            const gx = (rawX - offsetX) / gSens;
            const gy = (rawY - offsetY) / gSens;
            const currentG = Math.sqrt(gx * gx + gy * gy);

            if (currentG > peakG) {
                peakG = currentG;
                maxGLabel.textContent = `最大: ${peakG.toFixed(2)}G`;
            }

            currGEl.textContent = currentG.toFixed(2);
            yGEl.textContent = gy.toFixed(2);
            xGEl.textContent = gx.toFixed(2);

            const posX = centerX + (gx * radius);
            const posY = centerY - (gy * radius);

            history.push({ x: posX, y: posY });
            if (history.length > maxHistory) history.shift();

            if (history.length > 1) {
                ctx.beginPath();
                ctx.moveTo(history[0].x, history[0].y);
                for (let i = 1; i < history.length; i++) {
                    ctx.lineTo(history[i].x, history[i].y);
                }
                ctx.strokeStyle = 'rgba(0, 229, 255, 0.6)';
                ctx.lineWidth = 3;
                ctx.stroke();
            }

            ctx.save();
            ctx.globalAlpha = 1.0;

            if (customCarImage) {
                const imgSize = 40;
                ctx.beginPath();
                ctx.arc(posX, posY, imgSize / 2, 0, Math.PI * 2);
                ctx.closePath();
                ctx.clip();
                ctx.drawImage(customCarImage, posX - imgSize / 2, posY - imgSize / 2, imgSize, imgSize);
                
                ctx.strokeStyle = '#00ff66';
                ctx.lineWidth = 2;
                ctx.stroke();
            } else {
                const selectedVehicle = vehicleSelect.value;
                const selectedAngle = angleSelect.value;
                const iconStr = vehicleIcons[selectedVehicle][selectedAngle] || '🚗';

                ctx.font = '24px sans-serif';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.shadowColor = '#00ff66';
                ctx.shadowBlur = 10;
                ctx.fillText(iconStr, posX, posY);
            }
            ctx.restore();

            requestAnimationFrame(render);
        }

        initSensors();
        resizeCanvas();
        render();
    </script>
</body>
</html>

