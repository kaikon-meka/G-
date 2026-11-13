<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>G-Monitor Telemetry Pro v5.1 (Voice Naming Edition)</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; -webkit-user-select: none; }
        body { 
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; 
            background: #0a0a0c; 
            color: #fff; 
            display: flex; 
            flex-direction: column; 
            height: 100dvh; 
            overflow: hidden; 
        }

        .app-layout {
            display: flex;
            flex-direction: column;
            height: 100dvh;
            width: 100vw;
            overflow: hidden;
        }

        .right-sidebar { display: contents; }
        .header { order: 1; }
        .config-bar { order: 2; }
        #canvas-container { order: 3; }
        .dash-panel { order: 4; }
        .control-panel { order: 5; }

        /* ヘッダー */
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
        .config-group button.clear-btn, .config-group button.poyon-btn, .config-group label.song-lbl {
            background: #334155;
            color: #cbd5e1;
            border: 1px solid #475569;
            border-radius: 4px;
            padding: 3px 6px;
            font-weight: bold;
            font-size: 0.75rem;
            cursor: pointer;
            flex: 1;
            text-align: center;
        }
        .config-group button.poyon-btn {
            background: #ff758c;
            color: #fff;
            border-color: #ffb199;
            animation: pulse-btn 1.5s infinite;
        }
        .config-group label.song-lbl {
            background: #8b5cf6;
            color: #fff;
            border-color: #a78bfa;
        }
        @keyframes pulse-btn {
            0% { transform: scale(1); }
            50% { transform: scale(1.03); box-shadow: 0 0 10px #ff758c; }
            100% { transform: scale(1); }
        }
        #car-image-input, #song-file-input { display: none; }

        /* キャンバスエリア */
        #canvas-container { 
            position: relative; 
            flex: 1; 
            width: 100%;
            min-height: 150px;
            background: radial-gradient(circle, #161922 0%, #0d0f14 100%);
            display: flex;
            justify-content: center;
            align-items: center;
            overflow: hidden; 
            touch-action: none;
        }
        canvas { display: block; width: 100%; height: 100%; }

        /* 録画中バッジ & ご機嫌バッジ */
        #rec-badge {
            position: absolute;
            top: 10px;
            right: 12px;
            background: rgba(220, 38, 38, 0.85);
            color: #fff;
            padding: 4px 10px;
            border-radius: 20px;
            font-size: 0.75rem;
            font-weight: bold;
            font-family: monospace;
            display: none;
            align-items: center;
            gap: 6px;
            border: 1px solid #ff8a8a;
            box-shadow: 0 0 12px rgba(239, 68, 68, 0.7);
            z-index: 10;
        }
        #mood-badge {
            position: absolute;
            top: 10px;
            left: 12px;
            background: rgba(20, 23, 33, 0.85);
            color: #ffb6c1;
            padding: 4px 10px;
            border-radius: 20px;
            font-size: 0.72rem;
            font-weight: bold;
            display: none;
            align-items: center;
            gap: 4px;
            border: 1px solid #ff758c;
            z-index: 10;
            cursor: pointer;
        }
        .rec-dot {
            width: 8px;
            height: 8px;
            background: #fff;
            border-radius: 50%;
            animation: blink 0.8s infinite;
        }
        @keyframes blink {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.2; }
        }

        /* ダッシュボード */
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
            display: grid;
            grid-template-columns: 1fr 1fr 1.3fr;
            gap: 6px;
            flex-shrink: 0;
        }
        button.action-btn { 
            border: none; 
            border-radius: 6px; 
            color: white; 
            padding: 8px 4px; 
            font-size: 0.72rem; 
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
        
        /* ログボタン */
        button.log-btn {
            background: #2b1d2e;
            border-color: #ff758c;
            color: #ffb6c1;
        }
        button.log-btn.locked {
            background: #181c26;
            border-color: #334155;
            color: #64748b;
            opacity: 0.75;
        }
        button.log-btn.recording {
            background: #dc2626;
            border-color: #fca5a5;
            color: #fff;
            animation: pulse-btn 1s infinite;
        }

        /* ==========================================================
           🎙️ 音声ヒアリング＆名前確認モーダル
           ========================================================== */
        #voice-name-modal {
            position: fixed;
            inset: 0;
            background: rgba(5, 7, 12, 0.92);
            backdrop-filter: blur(8px);
            display: none;
            justify-content: center;
            align-items: center;
            z-index: 1000;
            padding: 16px;
        }
        .voice-card {
            background: linear-gradient(145deg, #1b2030, #121622);
            border: 2px solid #ff758c;
            border-radius: 20px;
            width: 100%;
            max-width: 360px;
            padding: 20px;
            text-align: center;
            box-shadow: 0 0 30px rgba(255, 117, 140, 0.5);
            animation: popIn 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }
        .mic-circle {
            width: 72px;
            height: 72px;
            border-radius: 50%;
            background: #ff758c;
            color: #fff;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 2rem;
            margin: 0 auto 14px auto;
            box-shadow: 0 0 20px rgba(255, 117, 140, 0.8);
            animation: micPulse 1.2s infinite;
        }
        @keyframes micPulse {
            0% { transform: scale(1); box-shadow: 0 0 0 0 rgba(255, 117, 140, 0.7); }
            70% { transform: scale(1.08); box-shadow: 0 0 0 18px rgba(255, 117, 140, 0); }
            100% { transform: scale(1); box-shadow: 0 0 0 0 rgba(255, 117, 140, 0); }
        }
        .voice-title {
            font-size: 1.05rem;
            font-weight: bold;
            color: #ffb6c1;
            margin-bottom: 6px;
        }
        .voice-status {
            font-size: 0.82rem;
            color: #00e5ff;
            margin-bottom: 14px;
            min-height: 20px;
        }
        .voice-btn-row {
            display: flex;
            gap: 8px;
            justify-content: center;
        }
        .voice-sub-btn {
            background: #252b40;
            color: #cbd5e1;
            border: 1px solid #475569;
            border-radius: 8px;
            padding: 8px 12px;
            font-size: 0.75rem;
            font-weight: bold;
            cursor: pointer;
            flex: 1;
        }
        .voice-sub-btn.retry {
            background: #ff758c;
            color: #fff;
            border-color: #ffb199;
        }

        /* ==========================================================
           🌸 ぽよんの運転採点＆ふれあいモーダル
           ========================================================== */
        #review-modal {
            position: fixed;
            inset: 0;
            background: rgba(5, 7, 12, 0.90);
            backdrop-filter: blur(6px);
            display: none;
            justify-content: center;
            align-items: center;
            z-index: 999;
            padding: 12px;
        }
        .review-card {
            background: linear-gradient(145deg, #1b2030, #121622);
            border: 2px solid #ff758c;
            border-radius: 16px;
            width: 100%;
            max-width: 440px;
            max-height: 92dvh;
            display: flex;
            flex-direction: column;
            box-shadow: 0 0 25px rgba(255, 117, 140, 0.45);
            overflow: hidden;
            animation: popIn 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }
        @keyframes popIn {
            0% { transform: scale(0.8); opacity: 0; }
            100% { transform: scale(1); opacity: 1; }
        }
        .review-header {
            background: #ff758c;
            color: #fff;
            padding: 8px 14px;
            font-weight: bold;
            font-size: 0.9rem;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .review-body {
            padding: 12px;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .score-pet-grid {
            display: grid;
            grid-template-columns: 110px 1fr;
            gap: 10px;
            background: #141721;
            border: 1px solid #2a354d;
            border-radius: 12px;
            padding: 8px;
            align-items: center;
        }
        .score-box {
            text-align: center;
            border-right: 1px dashed #2a354d;
            padding-right: 6px;
        }
        .score-lbl { font-size: 0.65rem; color: #8892b0; }
        .score-val { font-size: 2.0rem; font-weight: 900; font-family: monospace; color: #00ff66; line-height: 1.1; }
        .score-val.bad { color: #ff3366; }
        .score-unit { font-size: 0.75rem; color: #cbd5e1; }

        .pet-area {
            display: flex;
            flex-direction: column;
            align-items: center;
            position: relative;
        }
        #petCanvas {
            width: 140px;
            height: 110px;
            background: radial-gradient(circle, #252b40 0%, #141721 100%);
            border-radius: 12px;
            border: 1px solid #ff758c;
            cursor: pointer;
            touch-action: none;
        }
        .pet-hint {
            font-size: 0.65rem;
            color: #ffb6c1;
            margin-top: 4px;
            text-align: center;
            font-weight: bold;
        }
        .mood-bar-wrap {
            width: 100%;
            background: #0d1017;
            height: 8px;
            border-radius: 4px;
            margin-top: 4px;
            overflow: hidden;
            border: 1px solid #334155;
        }
        .mood-bar-fill {
            height: 100%;
            width: 100%;
            background: linear-gradient(90deg, #ff3366, #ff758c, #00ff66);
            transition: width 0.15s;
        }

        .poyon-speech-box {
            background: #252b40;
            border-left: 4px solid #ff758c;
            border-radius: 8px;
            padding: 10px;
        }
        .poyon-speech-title {
            font-size: 0.7rem;
            color: #ffb6c1;
            font-weight: bold;
            margin-bottom: 4px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .speak-again-btn {
            background: #ff758c;
            color: #fff;
            border: none;
            border-radius: 4px;
            padding: 2px 8px;
            font-size: 0.65rem;
            cursor: pointer;
            font-weight: bold;
        }
        .poyon-speech-text {
            font-size: 0.84rem;
            line-height: 1.45;
            color: #fff;
        }
        .review-stats {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 6px;
        }
        .stat-pill {
            background: #141721;
            border: 1px solid #2a354d;
            border-radius: 8px;
            padding: 5px;
            text-align: center;
        }
        .stat-pill-lbl { font-size: 0.58rem; color: #8892b0; }
        .stat-pill-val { font-size: 0.85rem; font-weight: bold; font-family: monospace; color: #00e5ff; margin-top: 1px; }

        .log-list-title {
            font-size: 0.72rem;
            color: #cbd5e1;
            font-weight: bold;
            border-bottom: 1px solid #2a354d;
            padding-bottom: 3px;
        }
        .log-list {
            background: #0d1017;
            border: 1px solid #232838;
            border-radius: 8px;
            padding: 6px;
            max-height: 110px;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            gap: 4px;
            font-family: monospace;
            font-size: 0.72rem;
        }
        .log-item {
            display: flex;
            gap: 6px;
            align-items: flex-start;
            line-height: 1.3;
            border-bottom: 1px dashed #1e2436;
            padding-bottom: 3px;
        }
        .log-time { color: #00e5ff; flex-shrink: 0; }
        .log-msg { color: #e2e8f0; }
        .log-msg.danger { color: #ff6b81; font-weight: bold; }
        .log-msg.sport { color: #facc15; }

        .review-footer {
            padding: 8px 14px;
            background: #141721;
            border-top: 1px solid #232838;
            display: flex;
            justify-content: center;
        }
        .close-review-btn {
            background: #00e5ff;
            color: #000;
            border: none;
            border-radius: 8px;
            padding: 8px 24px;
            font-weight: bold;
            font-size: 0.85rem;
            cursor: pointer;
            width: 100%;
        }

        /* 🚗 車載横画面（ランドスケープ）専用 */
        @media (orientation: landscape) {
            .app-layout {
                display: grid;
                grid-template-columns: 1fr 340px;
                grid-template-rows: 1fr;
                height: 100dvh;
                width: 100vw;
            }
            #canvas-container { grid-column: 1; grid-row: 1; height: 100% !important; width: 100% !important; order: unset; }
            .right-sidebar {
                grid-column: 2; grid-row: 1;
                display: flex; flex-direction: column;
                background: #141721; border-left: 2px solid #232838;
                height: 100dvh; overflow-y: auto;
            }
            .header, .config-bar, .dash-panel, .control-panel { order: unset; }
            .header { padding: 8px 10px; font-size: 0.7rem; }
            .config-bar { padding: 8px 10px; grid-template-columns: 1fr; gap: 6px; }
            .dash-panel { padding: 8px 10px; grid-template-columns: repeat(3, 1fr); gap: 4px; }
            .val-box { padding: 4px; }
            .val-num { font-size: 0.85rem; }
            .control-panel { padding: 8px 10px 12px 10px; grid-template-columns: 1fr 1fr; }
            button.log-btn { grid-column: span 2; padding: 8px; font-size: 0.8rem; }
            button.action-btn { padding: 6px; font-size: 0.72rem; }
        }
    </style>
</head>
<body>

    <div class="app-layout">
        <div id="canvas-container">
            <canvas id="gCanvas"></canvas>
            <div id="mood-badge" title="タップでぽよんと遊ぶ！">🌸 ご機嫌: 100%</div>
            <div id="rec-badge">
                <div class="rec-dot"></div>
                <span id="rec-timer">REC 05:00</span>
            </div>
        </div>

        <div class="right-sidebar">
            <div class="header">
                <div class="header-item"><i class="fas fa-satellite-dish"></i> <span id="gps-speed">0.0 km/h</span></div>
                <div class="header-item" style="color: #ffaa00;"><i class="fas fa-circle" id="status-dot"></i> <span id="status-text">起動中...</span></div>
                <div class="header-item"><i class="fas fa-compass"></i> <span id="max-g-label">最大: 0.00G</span></div>
            </div>

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
                <div class="config-group">
                    <label for="car-image-input" class="upload-lbl"><i class="fas fa-camera"></i> 愛車の写め</label>
                    <input type="file" id="car-image-input" accept="image/*">
                    <button id="btn-poyon" class="poyon-btn"><i class="fas fa-heart"></i> 🌸 ぽよん召喚</button>
                </div>
                <div class="config-group">
                    <label for="song-file-input" class="song-lbl"><i class="fas fa-music"></i> 🎵 曲をセット</label>
                    <input type="file" id="song-file-input" accept="audio/*">
                    <button id="btn-clear-img" class="clear-btn"><i class="fas fa-undo"></i> 標準に戻す</button>
                </div>
            </div>

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

            <div class="control-panel">
                <button id="btn-calib" class="action-btn"><i class="fas fa-crosshairs"></i> ゼロ調整</button>
                <button id="btn-reset" class="action-btn reset-btn"><i class="fas fa-trash-alt"></i> リセット</button>
                <button id="btn-log" class="action-btn log-btn locked"><i class="fas fa-lock"></i> ぽよん召喚で解放</button>
            </div>
        </div>
    </div>

    <!-- 🎙️ 音声ヒアリング＆名前確認モーダル -->
    <div id="voice-name-modal">
        <div class="voice-card">
            <div class="mic-circle"><i class="fas fa-microphone"></i></div>
            <div class="voice-title">👋😃 私ぽよん！キミの名前は？</div>
            <div id="voice-status-text" class="voice-status">🎤 マイクに向かってお名前を教えてね！</div>
            <div class="voice-btn-row">
                <button id="btn-voice-retry" class="voice-sub-btn retry"><i class="fas fa-redo"></i> もう一回聞く</button>
                <button id="btn-voice-manual" class="voice-sub-btn"><i class="fas fa-keyboard"></i> 手入力にする</button>
            </div>
        </div>
    </div>

    <!-- 🌸 ぽよんの運転採点＆ふれあいモーダル -->
    <div id="review-modal">
        <div class="review-card">
            <div class="review-header">
                <span>🌸 ぽよんの採点＆お気持ち</span>
                <span id="review-duration-badge" style="font-size:0.72rem; background:rgba(0,0,0,0.25); padding:2px 8px; border-radius:10px;">計測: 0分00秒</span>
            </div>
            <div class="review-body">
                <div class="score-pet-grid">
                    <div class="score-box">
                        <div class="score-lbl">運転スコア</div>
                        <div id="rev-score-val" class="score-val">100</div>
                        <div class="score-unit">/ 100点</div>
                    </div>
                    <div class="pet-area">
                        <canvas id="petCanvas" width="140" height="110"></canvas>
                        <div class="mood-bar-wrap">
                            <div id="mood-bar-fill" class="mood-bar-fill"></div>
                        </div>
                        <div id="pet-hint-text" class="pet-hint">👆タップで弾む / スリスリでなでなで💕</div>
                    </div>
                </div>

                <div class="poyon-speech-box">
                    <div class="poyon-speech-title">
                        <span>💬 ぽよんからのお気持ち</span>
                        <button id="btn-speak-again" class="speak-again-btn"><i class="fas fa-volume-up"></i> もう一度喋る</button>
                    </div>
                    <div id="poyon-comment" class="poyon-speech-text">
                        ふんふん〜♪ 今日もいいドライブだったね！
                    </div>
                </div>

                <div class="review-stats">
                    <div class="stat-pill">
                        <div class="stat-pill-lbl">セッション最大G</div>
                        <div id="rev-max-g" class="stat-pill-val">0.00 G</div>
                    </div>
                    <div class="stat-pill">
                        <div class="stat-pill-lbl">激しいG回数</div>
                        <div id="rev-event-cnt" class="stat-pill-val" style="color:#facc15;">0 回</div>
                    </div>
                    <div class="stat-pill">
                        <div class="stat-pill-lbl">白目絶叫回数</div>
                        <div id="rev-scream-cnt" class="stat-pill-val" style="color:#ff6b81;">0 回</div>
                    </div>
                </div>

                <div class="log-list-title">📜 タイムライン記録</div>
                <div id="log-list-container" class="log-list"></div>
            </div>
            <div class="review-footer">
                <button id="btn-close-review" class="close-review-btn">確認して閉じる ✨</button>
            </div>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gCanvas');
        const ctx = canvas.getContext('2d');
        const container = document.getElementById('canvas-container');

        const petCanvas = document.getElementById('petCanvas');
        const petCtx = petCanvas.getContext('2d');

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
        const songFileInput = document.getElementById('song-file-input');
        const btnPoyon = document.getElementById('btn-poyon');
        const btnClearImg = document.getElementById('btn-clear-img');

        const btnCalib = document.getElementById('btn-calib');
        const btnReset = document.getElementById('btn-reset');
        const btnLog = document.getElementById('btn-log');
        const recBadge = document.getElementById('rec-badge');
        const recTimerEl = document.getElementById('rec-timer');
        const moodBadge = document.getElementById('mood-badge');

        const voiceNameModal = document.getElementById('voice-name-modal');
        const voiceStatusText = document.getElementById('voice-status-text');
        const btnVoiceRetry = document.getElementById('btn-voice-retry');
        const btnVoiceManual = document.getElementById('btn-voice-manual');

        const reviewModal = document.getElementById('review-modal');
        const btnCloseReview = document.getElementById('btn-close-review');
        const btnSpeakAgain = document.getElementById('btn-speak-again');
        const poyonCommentEl = document.getElementById('poyon-comment');
        const revScoreValEl = document.getElementById('rev-score-val');
        const moodBarFillEl = document.getElementById('mood-bar-fill');
        const petHintTextEl = document.getElementById('pet-hint-text');
        const revMaxGEl = document.getElementById('rev-max-g');
        const revEventCntEl = document.getElementById('rev-event-cnt');
        const revScreamCntEl = document.getElementById('rev-scream-cnt');
        const logListContainer = document.getElementById('log-list-container');
        const reviewDurationBadge = document.getElementById('review-duration-badge');

        let width = 300, height = 300, centerX = 150, centerY = 150, radius = 100;
        let rawX = 0, rawY = 0, rawZ = 0;
        let offsetX = 0, offsetY = 0;
        const gSens = 9.80665;
        let isInitialized = false;

        let smoothGx = 0, smoothGy = 0;
        const smoothing = 0.25; 
        const deadzone = 0.03;  
        const invertX = -1, invertY = -1; 

        let peakG = 0;
        const history = [];
        const maxHistory = 15;

        let customCarImage = null;
        let isPoyonActive = false;
        let openingStartTime = null;
        const openingDuration = 3200;

        // 🌸 ユーザー名記憶システム（localStorage対応）
        let poyonUserName = localStorage.getItem('poyon_user_name') || '';

        const themeAudio = new Audio();
        themeAudio.loop = true;
        themeAudio.volume = 1.0;

        // ==========================================================
        // 🌸 ぽよんのご機嫌・ふれあいアニメーションステート
        // ==========================================================
        let poyonMood = 100;
        let bounceVelocity = 0;
        let bounceScaleX = 1.0;
        let bounceScaleY = 1.0;
        let isBeingPetted = false;
        let petTiltAngle = 0; 
        let petHearts = [];   
        let lastPetX = 0, lastPetY = 0;

        function updateMoodUI() {
            moodBarFillEl.style.width = `${poyonMood}%`;
            if (poyonMood < 50) {
                moodBadge.style.display = isPoyonActive ? 'flex' : 'none';
                moodBadge.textContent = `💢 ぷんぷん (${Math.floor(poyonMood)}%)`;
                moodBadge.style.color = '#ff6b81';
                moodBadge.style.borderColor = '#ff3366';
            } else {
                moodBadge.style.display = isPoyonActive ? 'flex' : 'none';
                moodBadge.textContent = `🌸 ご機嫌: ${Math.floor(poyonMood)}%`;
                moodBadge.style.color = '#ffb6c1';
                moodBadge.style.borderColor = '#ff758c';
            }
        }

        function spawnHeart(x, y, isAngry = false) {
            petHearts.push({
                x: x + (Math.random() - 0.5) * 30,
                y: y - 10,
                vy: -1.2 - Math.random() * 1.0,
                alpha: 1.0,
                char: isAngry ? '💢' : (Math.random() > 0.3 ? '💕' : '✨')
            });
        }

        // ログ関連
        let isLogging = false;
        let logStartTime = 0;
        const maxLogDuration = 5 * 60 * 1000;
        let sessionMaxG = 0;
        let sessionEvents = [];
        let lastEventRecordTime = 0;
        let screamCount = 0;
        let sportCount = 0;
        let harshPenalty = 0;

        // ==========================================================
        // 🗣️ オーディオ・ダッキング付き ぽよんロリボイス合成（完了コールバック対応）
        // ==========================================================
        function speakPoyonVoice(text, onEndCallback = null) {
            if (!('speechSynthesis' in window)) {
                if (onEndCallback) onEndCallback();
                return;
            }
            window.speechSynthesis.cancel();

            const utterance = new SpeechSynthesisUtterance(text);
            utterance.lang = 'ja-JP';
            utterance.pitch = 1.85;
            utterance.rate = 1.18;

            const voices = window.speechSynthesis.getVoices();
            const jpVoice = voices.find(v => v.lang.includes('ja') && (v.name.includes('Google') || v.name.includes('Natural') || v.name.includes('Female') || v.name.includes('Kyoko')));
            if (jpVoice) utterance.voice = jpVoice;

            utterance.onstart = () => {
                if (!themeAudio.paused) {
                    themeAudio.volume = 0.15;
                }
            };

            const finishSpeech = () => {
                themeAudio.volume = 1.0;
                if (onEndCallback) onEndCallback();
            };
            utterance.onend = finishSpeech;
            utterance.onerror = finishSpeech;

            window.speechSynthesis.speak(utterance);
        }

        if ('speechSynthesis' in window) {
            window.speechSynthesis.onvoiceschanged = () => { window.speechSynthesis.getVoices(); };
        }

        // ==========================================================
        // 🎙️ 音声認識（Web Speech API）による名前登録システム
        // ==========================================================
        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        let activeRecognition = null;

        function completeSummonSequence() {
            voiceNameModal.style.display = 'none';
            isPoyonActive = true;
            customCarImage = null;
            openingStartTime = Date.now();
            updateLogButtonState();
            updateMoodUI();

            if (themeAudio.src) {
                themeAudio.volume = 1.0;
                themeAudio.currentTime = 0;
                themeAudio.play().catch(err => console.log("Audio play blocked", err));
            }
        }

        function saveUserNameAndGreet(rawName) {
            // 「〜です」「〜だよ」などの語尾や句読点を自然にクリーニング
            let cleaned = rawName.replace(/(です|だよ|だ|といいます|と申します|。|、|\s)+$/g, '').trim();
            if (!cleaned) cleaned = "ボス";

            poyonUserName = cleaned;
            localStorage.setItem('poyon_user_name', poyonUserName);
            voiceStatusText.textContent = `✨「${poyonUserName}」だね！覚えたよ！`;

            speakPoyonVoice(`わーい！よろしくね、${poyonUserName}！`, () => {
                completeSummonSequence();
            });
        }

        function fallbackToManualInput() {
            if (activeRecognition) {
                try { activeRecognition.abort(); } catch(e) {}
            }
            voiceNameModal.style.display = 'none';
            let inputName = prompt("👋😃 声が聞き取れなかったみたい！キミの名前を教えてね：", "ボス");
            if (inputName && inputName.trim() !== "") {
                poyonUserName = inputName.trim();
            } else {
                poyonUserName = "ボス";
            }
            localStorage.setItem('poyon_user_name', poyonUserName);
            speakPoyonVoice(`よろしくね、${poyonUserName}！`, () => {
                completeSummonSequence();
            });
        }

        function startVoiceNameListening() {
            if (!SpeechRecognition) {
                fallbackToManualInput();
                return;
            }

            if (activeRecognition) {
                try { activeRecognition.abort(); } catch(e) {}
            }

            const recognition = new SpeechRecognition();
            activeRecognition = recognition;
            recognition.lang = 'ja-JP';
            recognition.interimResults = false;
            recognition.maxAlternatives = 1;

            voiceStatusText.textContent = '🔴 聴き取り中…「〇〇です！」と喋ってね！';

            recognition.onresult = (event) => {
                const spokenText = event.results[0][0].transcript;
                saveUserNameAndGreet(spokenText);
            };

            recognition.onerror = (event) => {
                console.log("Speech recognition error:", event.error);
                voiceStatusText.textContent = '💦 うまく聞き取れなかったよ…もう一回か手入力してね！';
            };

            recognition.onnomatch = () => {
                voiceStatusText.textContent = '💦 声が小さかったかも？「もう一回聞く」を押してね！';
            };

            try {
                recognition.start();
            } catch (e) {
                fallbackToManualInput();
            }
        }

        btnVoiceRetry.addEventListener('click', () => {
            startVoiceNameListening();
        });

        btnVoiceManual.addEventListener('click', () => {
            fallbackToManualInput();
        });

        // ==========================================================
        // 🌸 ぽよん召喚ボタン
        // ==========================================================
        btnPoyon.addEventListener('click', () => {
            if (!poyonUserName || poyonUserName.trim() === "") {
                voiceNameModal.style.display = 'flex';
                voiceStatusText.textContent = '🗣️ ぽよんがお話し中…終わったら喋ってね！';
                // ぽよんが質問を喋り終わったら自動でマイクON！
                speakPoyonVoice("私ぽよん！キミの名前は？マイクに向かって教えてね！", () => {
                    startVoiceNameListening();
                });
            } else {
                // 2回目以降はすでに名前を知っているので即召喚＆挨拶！
                speakPoyonVoice(`おかえり、${poyonUserName}！今日も安全運転で行こうね！`);
                completeSummonSequence();
            }
        });

        function updateLogButtonState() {
            if (isLogging) {
                btnLog.classList.remove('locked');
                btnLog.classList.add('recording');
                btnLog.innerHTML = '<i class="fas fa-stop"></i> ⏹ 記録停止';
            } else if (isPoyonActive) {
                btnLog.classList.remove('locked', 'recording');
                btnLog.innerHTML = '<i class="fas fa-circle"></i> 🔴 ログ開始 (5分)';
            } else {
                btnLog.classList.remove('recording');
                btnLog.classList.add('locked');
                btnLog.innerHTML = '<i class="fas fa-lock"></i> ぽよん召喚で解放';
            }
        }

        function formatSec(ms) {
            const totalSec = Math.max(0, Math.floor(ms / 1000));
            const m = String(Math.floor(totalSec / 60)).padStart(2, '0');
            const s = String(totalSec % 60).padStart(2, '0');
            return `${m}:${s}`;
        }

        function startEventLogging() {
            if (!isPoyonActive) {
                alert('🌸 先に「ぽよん召喚」を押して、ぽよんを助手席に乗せてね！');
                return;
            }
            isLogging = true;
            logStartTime = Date.now();
            sessionMaxG = 0;
            sessionEvents = [];
            screamCount = 0;
            sportCount = 0;
            harshPenalty = 0;
            lastEventRecordTime = 0;

            updateLogButtonState();
            recBadge.style.display = 'flex';
            recTimerEl.textContent = 'REC 05:00';

            sessionEvents.push({
                time: '00:00',
                msg: `🚀 計測スタート！${poyonUserName}の助手席でぽよんシートベルト装着完了！`,
                type: 'normal'
            });
        }

        function stopEventLogging(isAutoTimeout = false) {
            if (!isLogging) return;
            isLogging = false;
            const elapsed = Math.min(maxLogDuration, Date.now() - logStartTime);

            updateLogButtonState();
            recBadge.style.display = 'none';

            sessionEvents.push({
                time: formatSec(elapsed),
                msg: isAutoTimeout ? '⏰ 5分経過！自動で計測を完了したよ！' : '🏁 手動ストップで計測完了！',
                type: 'normal'
            });

            showPoyonReviewModal(elapsed);
        }

        function recordDrivingEvent(currentG, gx, gy, elapsed) {
            if (currentG > sessionMaxG) sessionMaxG = currentG;

            if (currentG > 0.65) {
                harshPenalty += (currentG - 0.65) * 0.4;
            }

            const now = Date.now();
            if (now - lastEventRecordTime < 1800) return;

            const timeStr = formatSec(elapsed);

            if (currentG >= 0.80) {
                lastEventRecordTime = now;
                screamCount++;
                let dir = Math.abs(gx) > Math.abs(gy) ? (gx > 0 ? '猛烈な右旋回G' : '猛烈な左旋回G') : (gy > 0 ? 'ロケット急加速G' : 'フルブレーキング減速G');
                sessionEvents.push({
                    time: timeStr,
                    msg: `😱 【${currentG.toFixed(2)}G】${dir}！ぽよん白目剥いて絶叫！！`,
                    type: 'danger'
                });
            } else if (currentG >= 0.40) {
                lastEventRecordTime = now;
                sportCount++;
                let dir = Math.abs(gx) > Math.abs(gy) ? (gx > 0 ? '鋭い右コーナリング' : '鋭い左コーナリング') : (gy > 0 ? 'グッとくる加速' : '強めのブレーキング');
                sessionEvents.push({
                    time: timeStr,
                    msg: `💦 【${currentG.toFixed(2)}G】${dir}を検知！`,
                    type: 'sport'
                });
            }
        }

        function calculateDriveScore() {
            let score = 100;
            score -= screamCount * 18; 
            score -= sportCount * 3;   
            score -= Math.floor(harshPenalty);
            if (sessionMaxG > 1.2) score -= 15;
            return Math.max(5, Math.min(100, Math.round(score)));
        }

        function showPoyonReviewModal(elapsed) {
            const totalSec = Math.floor(elapsed / 1000);
            const mins = Math.floor(totalSec / 60);
            const secs = totalSec % 60;
            reviewDurationBadge.textContent = `計測: ${mins}分${String(secs).padStart(2, '0')}秒`;

            const score = calculateDriveScore();
            revScoreValEl.textContent = score;
            if (score < 50) {
                revScoreValEl.classList.add('bad');
                poyonMood = Math.max(15, score); 
            } else {
                revScoreValEl.classList.remove('bad');
                poyonMood = Math.min(100, Math.max(70, score));
            }
            updateMoodUI();

            revMaxGEl.textContent = `${sessionMaxG.toFixed(2)} G`;
            revEventCntEl.textContent = `${sportCount + screamCount} 回`;
            revScreamCntEl.textContent = `${screamCount} 回`;

            let comment = '';
            if (score < 50) {
                petHintTextEl.textContent = '💢 おこ！指でスリスリなでなでしてご機嫌直して！';
                comment = `ぷんぷん！${poyonUserName}の運転荒すぎて ${score}点 だよー！最大 ${sessionMaxG.toFixed(2)}G なんて怖かったんだからっ！もう怒った！ちゃんとなでなでして遊んでくれなきゃ許さないんだからね！`;
            } else if (score < 80) {
                petHintTextEl.textContent = '👆タップで弾む / スリスリでなでなで💕';
                comment = `うーん、今日は ${score}点 かなっ！途中の ${sessionMaxG.toFixed(2)}G でちょっとドキドキしちゃったけど、まあまあ楽しかったよ！ご褒美になでなでして、${poyonUserName}〜♪`;
            } else {
                petHintTextEl.textContent = '🌸 ご機嫌マックス！スリスリすると喜ぶよ♪';
                comment = `わーい！すごいっ、${score}点 だよ${poyonUserName}！とっても優しい運転で、ぽよんすっごく快適だったぁ！大好きだよ、なでなでしてっ💕`;
            }

            poyonCommentEl.textContent = comment;

            logListContainer.innerHTML = '';
            sessionEvents.forEach(ev => {
                const div = document.createElement('div');
                div.className = 'log-item';
                div.innerHTML = `<span class="log-time">[${ev.time}]</span><span class="log-msg ${ev.type}">${ev.msg}</span>`;
                logListContainer.appendChild(div);
            });

            reviewModal.style.display = 'flex';
            speakPoyonVoice(comment);
        }

        // ==========================================================
        // 🐱 タップでポヨンポヨン＆スリスリなでなで操作
        // ==========================================================
        function triggerBounce(x, y) {
            bounceVelocity = 0.35; 
            spawnHeart(x, y, poyonMood < 50);
            if (poyonMood < 50) {
                poyonMood = Math.min(100, poyonMood + 4);
                updateMoodUI();
                checkForgiven();
            }
        }

        function triggerPetting(x, y) {
            isBeingPetted = true;
            if (Math.random() < 0.25) {
                spawnHeart(x, y, false);
            }
            if (poyonMood < 100) {
                const wasAngry = poyonMood < 50;
                poyonMood = Math.min(100, poyonMood + 1.8);
                updateMoodUI();
                if (wasAngry && poyonMood >= 50) {
                    checkForgiven();
                }
            }
        }

        function checkForgiven() {
            if (poyonMood >= 50 && revScoreValEl.classList.contains('bad')) {
                petHintTextEl.textContent = '💕 仲直り完了！スリスリ気持ちいい〜♪';
                const forgiveMsg = `もぉ〜、しょうがないなぁ、${poyonUserName}！そんなに優しくなでなでされたら怒れないよ〜っ。許してあげるっ💕`;
                poyonCommentEl.textContent = forgiveMsg;
                speakPoyonVoice(forgiveMsg);
                revScoreValEl.classList.remove('bad');
            }
        }

        petCanvas.addEventListener('pointerdown', (e) => {
            const rect = petCanvas.getBoundingClientRect();
            lastPetX = e.clientX - rect.left;
            lastPetY = e.clientY - rect.top;
            triggerBounce(lastPetX, lastPetY);
            isBeingPetted = true;
        });

        petCanvas.addEventListener('pointermove', (e) => {
            if (!isBeingPetted) return;
            const rect = petCanvas.getBoundingClientRect();
            const x = e.clientX - rect.left;
            const y = e.clientY - rect.top;
            const dist = Math.hypot(x - lastPetX, y - lastPetY);
            if (dist > 3) {
                triggerPetting(x, y);
                lastPetX = x;
                lastPetY = y;
            }
        });

        window.addEventListener('pointerup', () => {
            isBeingPetted = false;
        });

        canvas.addEventListener('pointerdown', (e) => {
            if (!isPoyonActive) return;
            const rect = canvas.getBoundingClientRect();
            lastPetX = e.clientX - rect.left;
            lastPetY = e.clientY - rect.top;
            triggerBounce(lastPetX, lastPetY);
            isBeingPetted = true;
        });

        canvas.addEventListener('pointermove', (e) => {
            if (!isPoyonActive || !isBeingPetted) return;
            const rect = canvas.getBoundingClientRect();
            const x = e.clientX - rect.left;
            const y = e.clientY - rect.top;
            if (Math.hypot(x - lastPetX, y - lastPetY) > 4) {
                triggerPetting(x, y);
                lastPetX = x;
                lastPetY = y;
            }
        });

        moodBadge.addEventListener('click', () => {
            if (isPoyonActive) {
                reviewModal.style.display = 'flex';
            }
        });

        btnLog.addEventListener('click', () => {
            if (isLogging) {
                stopEventLogging(false);
            } else {
                startEventLogging();
            }
        });

        btnCloseReview.addEventListener('click', () => {
            if ('speechSynthesis' in window) window.speechSynthesis.cancel();
            themeAudio.volume = 1.0;
            reviewModal.style.display = 'none';
        });

        btnSpeakAgain.addEventListener('click', () => {
            speakPoyonVoice(poyonCommentEl.textContent);
        });

        songFileInput.addEventListener('change', (e) => {
            const file = e.target.files[0];
            if (file) {
                const fileURL = URL.createObjectURL(file);
                themeAudio.src = fileURL;
                themeAudio.volume = 1.0;
                alert('🎵 ぽよんのテーマソングがセットされました！「🌸 ぽよん召喚」を押してね！');
            }
        });

        const vehicleIcons = {
            crown: { rear: '🚘', front: '🏎️', side: '🚙' },
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
                        isPoyonActive = false;
                        openingStartTime = null;
                        if (isLogging) stopEventLogging(false);
                        updateLogButtonState();
                        updateMoodUI();
                    };
                    img.src = event.target.result;
                };
                reader.readAsDataURL(file);
            }
        });

        btnClearImg.addEventListener('click', () => {
            customCarImage = null;
            isPoyonActive = false;
            openingStartTime = null;
            if (isLogging) stopEventLogging(false);
            updateLogButtonState();
            updateMoodUI();
            carImageInput.value = '';
            songFileInput.value = '';
            themeAudio.pause();
            themeAudio.currentTime = 0;
            themeAudio.volume = 1.0;
        });

        function resizeCanvas() {
            const rectW = container.clientWidth;
            const rectH = container.clientHeight;
            if (rectW > 0 && rectH > 0) {
                width = rectW;
                height = rectH;
                if (canvas.width !== width || canvas.height !== height) {
                    canvas.width = width;
                    canvas.height = height;
                }
                centerX = width / 2;
                centerY = height / 2;
                radius = Math.max(20, Math.min(centerX, centerY) - 25);
            }
        }

        window.addEventListener('resize', resizeCanvas);
        setTimeout(resizeCanvas, 50);

        function initSensors() {
            window.addEventListener('devicemotion', (e) => {
                const acc = e.accelerationIncludingGravity;
                if (acc && (acc.x !== null || acc.y !== null)) {
                    rawX = Number(acc.x) || 0;
                    rawY = Number(acc.y) || 0;
                    rawZ = Number(acc.z) || 0;

                    if (!isInitialized) {
                        offsetX = rawX;
                        offsetY = rawY;
                        isInitialized = true;
                    }

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
            smoothGx = 0;
            smoothGy = 0;
            peakG = 0;
            history.length = 0;
            statusText.textContent = 'ゼロ調整完了';
            statusDot.style.color = '#00ff66';
            setTimeout(() => {
                if(statusText.textContent === 'ゼロ調整完了') statusText.textContent = '計測中';
            }, 1500);
        });

        btnReset.addEventListener('click', () => {
            peakG = 0;
            maxGLabel.textContent = `最大: 0.00G`;
            history.length = 0;
        });

        // ==========================================================
        // 🎨 ぽよん描画エンジン
        // ==========================================================
        function drawDynamicPoyon(pCtx, posX, posY, currentG, customScale = 0.45, forceWink = false, isPetMode = false) {
            if (isNaN(posX) || isNaN(posY)) return;

            pCtx.save();
            pCtx.translate(posX, posY);

            const sx = customScale * bounceScaleX;
            const sy = customScale * bounceScaleY;
            pCtx.rotate(petTiltAngle);
            pCtx.scale(sx, sy);

            const chinUpOffset = isBeingPetted ? -6 : 0;
            pCtx.translate(-65, -70);

            const mainRadius = 45;
            const puffRadius = 10;
            const numPuffs = 12;
            const mainX = 65;
            const mainY = 70;

            pCtx.fillStyle = (poyonMood < 50 && !isBeingPetted) ? '#ff9aa2' : '#ffb6c1';
            for (let i = 0; i < numPuffs; i++) {
                const angle = (i / numPuffs) * Math.PI * 2;
                pCtx.beginPath();
                pCtx.arc(mainX + Math.cos(angle) * mainRadius, mainY + Math.sin(angle) * mainRadius, puffRadius, 0, Math.PI * 2);
                pCtx.fill();
            }

            pCtx.beginPath();
            pCtx.arc(mainX, mainY, mainRadius + puffRadius - 1, 0, Math.PI * 2);
            pCtx.fill();

            pCtx.strokeStyle = (poyonMood < 50 && !isBeingPetted) ? '#ff4d6d' : '#ff758c';
            pCtx.lineWidth = 5;
            pCtx.lineCap = 'round';
            for (let i = 0; i < numPuffs; i++) {
                const angle = (i / numPuffs) * Math.PI * 2;
                pCtx.beginPath();
                pCtx.arc(mainX + Math.cos(angle) * mainRadius, mainY + Math.sin(angle) * mainRadius, puffRadius, 0, Math.PI * 2);
                pCtx.stroke();
            }
            pCtx.beginPath();
            pCtx.arc(mainX, mainY, mainRadius + 1, 0, Math.PI * 2);
            pCtx.fill();

            pCtx.fillStyle = isBeingPetted ? '#ff4d6d' : '#ff8da1';
            pCtx.beginPath();
            pCtx.arc(43, 77 + chinUpOffset, isBeingPetted ? 10 : 8, 0, Math.PI * 2);
            pCtx.arc(87, 77 + chinUpOffset, isBeingPetted ? 10 : 8, 0, Math.PI * 2);
            pCtx.fill();

            pCtx.fillStyle = '#a8e6cf';
            pCtx.strokeStyle = '#7fd8b2';
            pCtx.lineWidth = 2.5;
            pCtx.beginPath();
            pCtx.ellipse(65, 31 + chinUpOffset * 0.5, 30, 14, 0, 0, Math.PI * 2);
            pCtx.fill();
            pCtx.stroke();

            pCtx.fillStyle = '#ff758c';
            pCtx.beginPath();
            pCtx.arc(65, 31 + chinUpOffset * 0.5, 7, 0, Math.PI * 2);
            pCtx.fill();

            pCtx.strokeStyle = '#4a2e35';
            pCtx.fillStyle = '#4a2e35';

            if (isBeingPetted) {
                pCtx.lineWidth = 4;
                pCtx.lineCap = 'round';
                pCtx.beginPath();
                pCtx.arc(50, 65 + chinUpOffset, 7, Math.PI, 0);
                pCtx.stroke();
                pCtx.beginPath();
                pCtx.arc(80, 65 + chinUpOffset, 7, Math.PI, 0);
                pCtx.stroke();

                pCtx.lineWidth = 3;
                pCtx.beginPath();
                pCtx.arc(60, 74 + chinUpOffset, 5, 0, Math.PI);
                pCtx.arc(70, 74 + chinUpOffset, 5, 0, Math.PI);
                pCtx.stroke();

            } else if (poyonMood < 50 && !forceWink && currentG < 0.8) {
                pCtx.lineWidth = 4;
                pCtx.lineCap = 'round';
                pCtx.beginPath();
                pCtx.moveTo(42, 54); pCtx.lineTo(56, 60);
                pCtx.moveTo(88, 54); pCtx.lineTo(74, 60);
                pCtx.stroke();

                pCtx.beginPath();
                pCtx.arc(50, 66, 5, 0, Math.PI * 2);
                pCtx.arc(80, 66, 5, 0, Math.PI * 2);
                pCtx.fill();

                pCtx.lineWidth = 3.5;
                pCtx.beginPath();
                pCtx.moveTo(56, 79);
                pCtx.quadraticCurveTo(65, 72, 74, 79);
                pCtx.stroke();

            } else if (forceWink) {
                pCtx.beginPath();
                pCtx.arc(50, 65, 6, 0, Math.PI * 2);
                pCtx.fill();
                pCtx.fillStyle = '#fff';
                pCtx.beginPath();
                pCtx.arc(48, 63, 2.5, 0, Math.PI * 2);
                pCtx.fill();
                pCtx.lineWidth = 4;
                pCtx.lineCap = 'round';
                pCtx.beginPath();
                pCtx.arc(80, 66, 7, Math.PI, 0);
                pCtx.stroke();
                pCtx.fillStyle = '#ff5277';
                pCtx.beginPath();
                pCtx.arc(65, 73, 10, 0, Math.PI);
                pCtx.fill();
                pCtx.stroke();

            } else if (currentG < 0.3) {
                pCtx.beginPath();
                pCtx.arc(50, 65, 6, 0, Math.PI * 2);
                pCtx.arc(80, 65, 6, 0, Math.PI * 2);
                pCtx.fill();
                pCtx.fillStyle = '#fff';
                pCtx.beginPath();
                pCtx.arc(48, 63, 2.5, 0, Math.PI * 2);
                pCtx.arc(78, 63, 2.5, 0, Math.PI * 2);
                pCtx.fill();
                pCtx.strokeStyle = '#4a2e35';
                pCtx.lineWidth = 3;
                pCtx.beginPath();
                pCtx.moveTo(58, 73);
                pCtx.quadraticCurveTo(65, 79, 72, 73);
                pCtx.stroke();

            } else if (currentG < 0.8) {
                pCtx.lineWidth = 4;
                pCtx.lineCap = 'round';
                pCtx.beginPath();
                pCtx.moveTo(43, 58); pCtx.lineTo(53, 65); pCtx.lineTo(43, 72);
                pCtx.stroke();
                pCtx.beginPath();
                pCtx.moveTo(87, 58); pCtx.lineTo(77, 65); pCtx.lineTo(87, 72);
                pCtx.stroke();
                pCtx.fillStyle = '#4a2e35';
                pCtx.beginPath();
                pCtx.arc(65, 76, 6, 0, Math.PI * 2);
                pCtx.fill();

            } else {
                pCtx.strokeStyle = '#d32f2f';
                pCtx.lineWidth = 4;
                pCtx.beginPath();
                pCtx.moveTo(43, 58); pCtx.lineTo(55, 70);
                pCtx.moveTo(55, 58); pCtx.lineTo(43, 70);
                pCtx.stroke();
                pCtx.beginPath();
                pCtx.moveTo(75, 58); pCtx.lineTo(87, 70);
                pCtx.moveTo(87, 58); pCtx.lineTo(75, 70);
                pCtx.stroke();
                pCtx.strokeStyle = '#4a2e35';
                pCtx.lineWidth = 3;
                pCtx.beginPath();
                pCtx.arc(65, 72, 8, 0, Math.PI);
                pCtx.stroke();
                pCtx.fillStyle = '#ff6b81';
                pCtx.beginPath();
                pCtx.arc(68, 77, 5, 0, Math.PI * 2);
                pCtx.fill();
            }
            pCtx.restore();
        }

        function updatePetPhysics() {
            if (Math.abs(bounceVelocity) > 0.001 || Math.abs(bounceScaleX - 1.0) > 0.001) {
                bounceScaleX += bounceVelocity;
                bounceScaleY = 2.0 - bounceScaleX; 
                bounceVelocity += (1.0 - bounceScaleX) * 0.25; 
                bounceVelocity *= 0.82; 
            } else {
                bounceScaleX = 1.0;
                bounceScaleY = 1.0;
            }

            const targetTilt = isBeingPetted ? Math.sin(Date.now() * 0.008) * 0.14 : 0;
            petTiltAngle += (targetTilt - petTiltAngle) * 0.15;
        }

        function drawHearts(targetCtx) {
            for (let i = petHearts.length - 1; i >= 0; i--) {
                const h = petHearts[i];
                h.y += h.vy;
                h.alpha -= 0.025;
                if (h.alpha <= 0) {
                    petHearts.splice(i, 1);
                    continue;
                }
                targetCtx.save();
                targetCtx.globalAlpha = h.alpha;
                targetCtx.font = '16px sans-serif';
                targetCtx.fillText(h.char, h.x, h.y);
                targetCtx.restore();
            }
        }

        function drawOpeningAnimation(elapsed) {
            const progress = Math.max(0, Math.min(1, elapsed / openingDuration));
            ctx.fillStyle = '#090b10';
            ctx.fillRect(0, 0, width, height);

            const maxRing = Math.max(50, Math.max(width, height) * 0.7);
            const ringRadius = Math.max(1, (elapsed * 0.25) % maxRing);
            ctx.beginPath();
            ctx.arc(centerX, centerY, ringRadius, 0, Math.PI * 2);
            ctx.strokeStyle = `rgba(255, 117, 140, ${Math.max(0, 1 - ringRadius / maxRing)})`;
            ctx.lineWidth = 3;
            ctx.stroke();

            if (progress < 0.35) {
                const alpha = Math.min(1, progress / 0.15);
                ctx.save();
                ctx.fillStyle = `rgba(0, 229, 255, ${alpha})`;
                ctx.font = 'bold 14px monospace';
                ctx.textAlign = 'center';
                ctx.fillText('SYSTEM INITIALIZING...', centerX, centerY - 12);
                ctx.fillStyle = `rgba(255, 117, 140, ${alpha})`;
                ctx.font = 'bold 16px monospace';
                ctx.fillText(`HELLO, ${poyonUserName || 'BOSS'}!!`, centerX, centerY + 14);
                ctx.restore();
            } else if (progress < 0.65) {
                const t = (progress - 0.35) / 0.30;
                const dropY = centerY - (1 - Math.pow(t, 0.5)) * (height * 0.6) - Math.abs(Math.sin(t * Math.PI * 2)) * (1 - t) * 40;
                const bounceScale = 1.1 + Math.sin(t * Math.PI * 3) * 0.15 * (1 - t);
                drawDynamicPoyon(ctx, centerX, dropY, 0, bounceScale, false);
            } else {
                const t = (progress - 0.65) / 0.35;
                const currentScale = t < 0.7 ? 1.25 : 1.25 - ((t - 0.7) / 0.3) * (1.25 - 0.45);
                for (let i = 0; i < 8; i++) {
                    const angle = (i / 8) * Math.PI * 2 + elapsed * 0.005;
                    const dist = 65 + Math.sin(elapsed * 0.01 + i) * 15;
                    ctx.fillStyle = i % 2 === 0 ? '#ffd700' : '#ff758c';
                    ctx.beginPath();
                    ctx.arc(centerX + Math.cos(angle) * dist, centerY + Math.sin(angle) * dist, 4, 0, Math.PI * 2);
                    ctx.fill();
                }
                drawDynamicPoyon(ctx, centerX, centerY, 0, currentScale, true);
                if (t < 0.85) {
                    ctx.save();
                    ctx.font = 'bold 22px "Segoe UI", sans-serif';
                    ctx.textAlign = 'center';
                    ctx.shadowColor = '#ff758c';
                    ctx.shadowBlur = 15;
                    ctx.fillStyle = '#fff';
                    ctx.fillText(`🌸 ${poyonUserName}、よろしくね！ 🌸`, centerX, centerY + 75);
                    ctx.restore();
                }
            }
        }

        function render() {
            try {
                updatePetPhysics();

                if (reviewModal.style.display === 'flex') {
                    petCtx.clearRect(0, 0, 140, 110);
                    drawDynamicPoyon(petCtx, 70, 62, 0, 0.75, false, true);
                    drawHearts(petCtx);
                }

                if (canvas.width !== container.clientWidth || canvas.height !== container.clientHeight) {
                    resizeCanvas();
                }

                ctx.clearRect(0, 0, width, height);

                if (openingStartTime !== null) {
                    const elapsed = Date.now() - openingStartTime;
                    if (elapsed < openingDuration) {
                        drawOpeningAnimation(elapsed);
                        requestAnimationFrame(render);
                        return;
                    } else {
                        openingStartTime = null;
                    }
                }

                [0.5, 1.0, 1.5].forEach(g => {
                    const r = Math.max(5, radius * (g / 1.0));
                    ctx.beginPath();
                    ctx.arc(centerX, centerY, r, 0, Math.PI * 2);
                    ctx.strokeStyle = g === 1.0 ? '#2a354d' : '#1a2233';
                    ctx.lineWidth = g === 1.0 ? 2 : 1;
                    ctx.setLineDash(g === 1.0 ? [] : [4, 4]);
                    ctx.stroke();
                });
                ctx.setLineDash([]);

                ctx.beginPath();
                ctx.moveTo(centerX, 15); ctx.lineTo(centerX, Math.max(20, height - 15));
                ctx.moveTo(15, centerY); ctx.lineTo(Math.max(20, width - 15), centerY);
                ctx.strokeStyle = '#222b3d';
                ctx.lineWidth = 1.5;
                ctx.stroke();

                ctx.fillStyle = '#4a5568';
                ctx.font = '11px monospace';
                ctx.fillText('前 (減速)', centerX - 25, 12);
                ctx.fillText('後 (加速)', centerX - 25, height - 5);
                ctx.fillText('左', 5, centerY + 4);
                ctx.fillText('右', width - 15, centerY + 4);

                const targetGx = ((rawX - offsetX) / gSens) || 0;
                const targetGy = ((rawY - offsetY) / gSens) || 0;

                smoothGx += (targetGx - smoothGx) * smoothing;
                smoothGy += (targetGy - smoothGy) * smoothing;

                const filteredGx = Math.abs(smoothGx) < deadzone ? 0 : smoothGx;
                const filteredGy = Math.abs(smoothGy) < deadzone ? 0 : smoothGy;

                const gx = (filteredGx * invertX) || 0;
                const gy = (filteredGy * invertY) || 0;
                const currentG = Math.sqrt(gx * gx + gy * gy) || 0;

                if (currentG > peakG) {
                    peakG = currentG;
                    maxGLabel.textContent = `最大: ${peakG.toFixed(2)}G`;
                }

                currGEl.textContent = currentG.toFixed(2);
                yGEl.textContent = gy.toFixed(2);
                xGEl.textContent = gx.toFixed(2);

                if (isLogging) {
                    const logElapsed = Date.now() - logStartTime;
                    if (logElapsed >= maxLogDuration) {
                        stopEventLogging(true);
                    } else {
                        const remaining = maxLogDuration - logElapsed;
                        recTimerEl.textContent = `REC ${formatSec(remaining)}`;
                        recordDrivingEvent(currentG, gx, gy, logElapsed);
                    }
                }

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
                if (isPoyonActive) {
                    drawDynamicPoyon(ctx, posX, posY, currentG, 0.45, false);
                    drawHearts(ctx);
                } else if (customCarImage) {
                    const imgSize = 48;
                    ctx.beginPath();
                    ctx.arc(posX, posY, imgSize / 2 + 2, 0, Math.PI * 2);
                    ctx.closePath();
                    ctx.clip();
                    ctx.drawImage(customCarImage, posX - imgSize / 2, posY - imgSize / 2, imgSize, imgSize);
                    ctx.strokeStyle = '#ff758c';
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
            } catch (e) {
                console.error("Render error caught:", e);
            }

            requestAnimationFrame(render);
        }

        initSensors();
        resizeCanvas();
        requestAnimationFrame(render);
    </script>
</body>
</html>

