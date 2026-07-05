<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Управление рукой</title>
  <style>
    * { box-sizing: border-box; }
    body {
      margin: 0;
      font-family: system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif;
      background: #f5f7fa;
      color: #1e293b;
    }

    #app {
      max-width: 1300px;
      margin: 0 auto;
      padding: 16px 20px 40px;
    }

    /* верхняя панель */
    .panel {
      display: flex;
      flex-wrap: wrap;
      gap: 20px 24px;
      background: white;
      padding: 18px 20px;
      border-radius: 20px;
      box-shadow: 0 8px 24px rgba(0,0,0,0.06);
      margin-bottom: 28px;
      align-items: flex-start;
    }

    .cam-block {
      flex: 0 0 340px;
      min-width: 280px;
    }

    .cam-box {
      width: 100%;
      aspect-ratio: 4 / 3;
      background: #0b1120;
      border-radius: 16px;
      overflow: hidden;
      position: relative;
      border: 1px solid #e2e8f0;
      box-shadow: 0 2px 8px rgba(0,0,0,0.03);
    }

    .cam-box video {
      display: none;
    }

    .cam-box canvas {
      display: block;
      width: 100%;
      height: 100%;
      object-fit: cover;
      transform: scaleX(-1);
      background: #0b1120;
    }

    .cam-actions {
      display: flex;
      flex-wrap: wrap;
      gap: 8px;
      margin-top: 12px;
    }

    .cam-actions button {
      background: #f1f5f9;
      border: 1px solid #d1d9e6;
      padding: 8px 16px;
      border-radius: 40px;
      font-weight: 500;
      font-size: 0.9rem;
      color: #1e293b;
      cursor: pointer;
      transition: 0.2s;
      box-shadow: 0 1px 2px rgba(0,0,0,0.02);
    }

    .cam-actions button:hover {
      background: #e2e8f0;
      border-color: #b9c4d4;
    }

    .cam-actions button:active {
      background: #cbd5e1;
      transform: scale(0.97);
    }

    .cam-actions button#camOffBtn {
      background: #fee2e2;
      border-color: #fecaca;
      color: #991b1b;
    }

    .cam-actions button#camOffBtn:hover {
      background: #fecaca;
    }

    .log-block {
      flex: 1;
      min-width: 200px;
    }

    .log-block h3 {
      margin: 0 0 6px 0;
      font-size: 0.9rem;
      font-weight: 600;
      letter-spacing: 0.3px;
      color: #475569;
    }

    .log-box {
      background: #f8fafc;
      border: 1px solid #e2e8f0;
      border-radius: 14px;
      padding: 14px 16px;
      font-family: 'JetBrains Mono', 'Fira Code', monospace;
      font-size: 0.8rem;
      line-height: 1.6;
      white-space: pre;
      overflow: auto;
      max-height: 280px;
      min-height: 130px;
      color: #0f172a;
      box-shadow: inset 0 1px 3px rgba(0,0,0,0.02);
    }

    /* основной контент */
    .info {
      background: white;
      border-radius: 20px;
      padding: 24px 28px;
      box-shadow: 0 4px 16px rgba(0,0,0,0.03);
      border: 1px solid #edf2f7;
    }

    .info h1 {
      font-size: 1.8rem;
      font-weight: 600;
      margin: 0 0 12px 0;
      letter-spacing: -0.3px;
    }

    .info p {
      margin: 8px 0;
      font-size: 1rem;
      line-height: 1.6;
      color: #334155;
    }

    .info p b {
      color: #0f172a;
      font-weight: 600;
    }

    .badge {
      display: inline-block;
      background: #dbeafe;
      color: #1e4a8a;
      padding: 2px 12px;
      border-radius: 30px;
      font-size: 0.8rem;
      font-weight: 500;
      margin-right: 6px;
    }

    @media (max-width: 720px) {
      .cam-block { flex: 1 1 100%; }
      .panel { padding: 16px; }
      .info { padding: 20px; }
    }
  </style>
</head>
<body>
<div id="app">

  <div class="panel">
    <div class="cam-block">
      <div class="cam-box">
        <video id="inputVideo" playsinline muted></video>
        <canvas id="camCanvas"></canvas>
      </div>
      <div class="cam-actions">
        <button id="camToggleBtn">📷 Включить камеру</button>
        <button id="camOffBtn" style="display:none;">⏻ Выключить</button>
        <button id="modeBtn">🔄 Сменить режим</button>
      </div>
    </div>

    <div class="log-block">
      <h3>📋 Логи и данные</h3>
      <div class="log-box" id="logText">Режим: Растягивание объекта
FPS: —
Рука не найдена</div>
    </div>
  </div>

  <div class="info">
    <h1>🧠 Режимы управления</h1>
    <p><span class="badge">Растягивание</span> <b>Сожми большой и указательный палец</b> на каждой руке (получится по «щипку»). Между четырьмя пальцами появится объект. Двигай руками — меняй его ширину и длину.</p>
    <p><span class="badge">Запотевшее стекло</span> Камера «запотевает», и тебя не видно. Проведи рукой перед камерой — в этом месте запотевание исчезнет, и станет видно видео.</p>
  </div>
</div>

<!-- MediaPipe -->
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/drawing_utils/drawing_utils.js" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js" crossorigin="anonymous"></script>

<script>
  (function(){
    'use strict';

    // --- DOM refs ---
    const camToggleBtn = document.getElementById('camToggleBtn');
    const camOffBtn = document.getElementById('camOffBtn');
    const modeBtn = document.getElementById('modeBtn');
    const logText = document.getElementById('logText');
    const videoEl = document.getElementById('inputVideo');
    const camCanvas = document.getElementById('camCanvas');
    const camCtx = camCanvas.getContext('2d');

    // --- state ---
    let hands = null;
    let mediaStream = null;
    let running = false;
    let rafId = null;
    let latestResults = null;

    let lastTs = performance.now();
    let fpsFrames = 0, fpsAccum = 0, fps = 0;

    const PINCH_RATIO = 0.55;

    // 'stretch' | 'frost'
    let mode = 'stretch';
    let frostCanvas = null;
    let frostCtx = null;

    let detectBusy = false;

    // для стабильности формы (1 кадр коллинеарности)
    let lastQuad = null;

    // --- convex hull (монотонная цепочка) ---
    function convexHull(points){
      if(points.length < 3) return points.slice();
      const pts = points.slice().sort((a,b)=> a.x - b.x || a.y - b.y);
      const cross = (o,a,b)=> (a.x-o.x)*(b.y-o.y) - (a.y-o.y)*(b.x-o.x);
      const lower = [];
      for(const p of pts){
        while(lower.length >= 2 && cross(lower[lower.length-2], lower[lower.length-1], p) <= 0) lower.pop();
        lower.push(p);
      }
      const upper = [];
      for(let i=pts.length-1; i>=0; i--){
        const p = pts[i];
        while(upper.length >= 2 && cross(upper[upper.length-2], upper[upper.length-1], p) <= 0) upper.pop();
        upper.push(p);
      }
      lower.pop(); upper.pop();
      return lower.concat(upper);
    }

    // --- утилиты для рук ---
    function handScale(lm, w, h){
      const a = lm[0], b = lm[9];
      return Math.hypot((a.x-b.x)*w, (a.y-b.y)*h);
    }
    function pinchDist(lm, w, h){
      const t = lm[4], i = lm[8];
      return Math.hypot((t.x-i.x)*w, (t.y-i.y)*h);
    }
    function handCenter(lm, w, h){
      const idxs = [0,5,9,13,17];
      let sx=0, sy=0;
      for(let idx of idxs){ sx += lm[idx].x; sy += lm[idx].y; }
      return { x: (sx/idxs.length)*w, y: (sy/idxs.length)*h };
    }

    // --- frost ---
    function setupFrost(){
      const vw = videoEl.videoWidth || 320, vh = videoEl.videoHeight || 240;
      if(!frostCanvas || frostCanvas.width !== vw || frostCanvas.height !== vh){
        frostCanvas = document.createElement('canvas');
        frostCanvas.width = vw; frostCanvas.height = vh;
        frostCtx = frostCanvas.getContext('2d');
      }
      frostCtx.fillStyle = 'rgba(225,230,235,0.96)';
      frostCtx.fillRect(0, 0, vw, vh);
    }

    function wipeFrost(x, y, radius){
      if(!frostCtx) return;
      frostCtx.save();
      frostCtx.globalCompositeOperation = 'destination-out';
      const grad = frostCtx.createRadialGradient(x, y, 0, x, y, radius);
      grad.addColorStop(0, 'rgba(0,0,0,1)');
      grad.addColorStop(0.7, 'rgba(0,0,0,0.85)');
      grad.addColorStop(1, 'rgba(0,0,0,0)');
      frostCtx.fillStyle = grad;
      frostCtx.beginPath();
      frostCtx.arc(x, y, radius, 0, Math.PI*2);
      frostCtx.fill();
      frostCtx.restore();
    }

    // --- инициализация MediaPipe Hands ---
    async function ensureHands(){
      if(hands) return;
      if(typeof Hands === 'undefined'){
        throw new Error('Библиотека распознавания рук не загрузилась (проверь интернет / доступ к cdn.jsdelivr.net).');
      }
      hands = new Hands({
        locateFile: (f) => `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${f}`
      });
      hands.setOptions({
        maxNumHands: 2,
        modelComplexity: 0,
        minDetectionConfidence: 0.5,
        minTrackingConfidence: 0.5
      });
      hands.onResults((r) => { latestResults = r; });
    }

    // --- камера ---
    async function startHandControl(){
      camToggleBtn.disabled = true;
      try{
        await ensureHands();
        mediaStream = await navigator.mediaDevices.getUserMedia({
          video: { width: { ideal: 320 }, height: { ideal: 240 }, facingMode: 'user' },
          audio: false
        });
        videoEl.srcObject = mediaStream;
        await videoEl.play();

        camToggleBtn.style.display = 'none';
        camOffBtn.style.display = 'inline-block';
        running = true;
        lastTs = performance.now();
        if(frostCanvas) setupFrost(); // обновить размеры если надо
        loop();
      } catch(err){
        console.error(err);
        alert('Не удалось включить камеру: ' + (err.message || err.name));
        camToggleBtn.disabled = false;
      }
    }

    function stopHandControl(){
      running = false;
      if(rafId) cancelAnimationFrame(rafId);
      if(mediaStream){ mediaStream.getTracks().forEach(t=>t.stop()); mediaStream = null; }
      camToggleBtn.style.display = 'inline-block';
      camToggleBtn.disabled = false;
      camOffBtn.style.display = 'none';
      lastQuad = null;
      logText.textContent = 'Режим: ' + (mode==='stretch' ? 'Растягивание объекта' : 'Запотевшее стекло') + '\nFPS: —\nРука не найдена';
    }

    // --- главный цикл ---
    async function loop(){
      if(!running) return;

      const now = performance.now();
      const dt = (now - lastTs) / 1000;
      lastTs = now;
      fpsFrames++; fpsAccum += dt;
      if(fpsAccum > 0.5){
        fps = Math.round(fpsFrames / fpsAccum);
        fpsFrames = 0; fpsAccum = 0;
      }

      const vw = videoEl.videoWidth, vh = videoEl.videoHeight;
      if(vw && vh){
        if(camCanvas.width !== vw || camCanvas.height !== vh){
          camCanvas.width = vw; camCanvas.height = vh;
        }
        camCtx.clearRect(0,0,vw,vh);
        camCtx.drawImage(videoEl, 0, 0, vw, vh);
      }

      // детекция
      if(!detectBusy && vw && vh){
        detectBusy = true;
        try{
          await hands.send({ image: videoEl });
        } catch(e){ console.warn('detect error', e); }
        detectBusy = false;
      }

      processFrame(vw, vh);

      rafId = requestAnimationFrame(loop);
    }

    // --- обработка кадра ---
    function processFrame(vw, vh){
      const list = (latestResults && latestResults.multiHandLandmarks) ? latestResults.multiHandLandmarks : [];
      const modeLabel = mode === 'stretch' ? 'Растягивание объекта' : 'Запотевшее стекло';
      const lines = ['Режим: ' + modeLabel, 'FPS: ' + (fps || '—'), 'Обнаружено рук: ' + list.length];

      if(!vw || !vh){
        logText.textContent = lines.join('\n');
        return;
      }

      // скелеты
      list.forEach(lm => {
        if(window.drawConnectors && window.HAND_CONNECTIONS){
          window.drawConnectors(camCtx, lm, window.HAND_CONNECTIONS, { color: '#00AAFF', lineWidth: 2 });
          window.drawLandmarks(camCtx, lm, { color: '#FF0000', fillColor: '#FF0000', radius: 2 });
        }
      });

      if(mode === 'stretch'){
        if(list.length >= 2){
          const lmA = list[0], lmB = list[1];
          const scaleA = handScale(lmA, vw, vh), scaleB = handScale(lmB, vw, vh);
          const pinchA = pinchDist(lmA, vw, vh) < scaleA * PINCH_RATIO;
          const pinchB = pinchDist(lmB, vw, vh) < scaleB * PINCH_RATIO;

          lines.push('Щипок рука 1: ' + (pinchA ? '✅ да' : '❌ нет'));
          lines.push('Щипок рука 2: ' + (pinchB ? '✅ да' : '❌ нет'));

          if(pinchA && pinchB){
            const thumbA = { x: lmA[4].x*vw, y: lmA[4].y*vh };
            const indexA = { x: lmA[8].x*vw, y: lmA[8].y*vh };
            const thumbB = { x: lmB[4].x*vw, y: lmB[4].y*vh };
            const indexB = { x: lmB[8].x*vw, y: lmB[8].y*vh };
            const rawPts = [thumbA, indexA, thumbB, indexB];
            let pts = convexHull(rawPts);

            // если вырожденный кадр — используем последний корректный
            if(pts.length < 3 && lastQuad){
              pts = lastQuad;
            } else if(pts.length >= 3){
              lastQuad = pts;
            }

            if(pts.length >= 3){
              camCtx.save();
              camCtx.beginPath();
              camCtx.moveTo(pts[0].x, pts[0].y);
              for(let i=1; i<pts.length; i++) camCtx.lineTo(pts[i].x, pts[i].y);
              camCtx.closePath();
              camCtx.fillStyle = 'rgba(60,150,255,0.72)';
              camCtx.fill();
              camCtx.strokeStyle = 'rgba(255,255,255,0.9)';
              camCtx.lineWidth = 2.5;
              camCtx.stroke();
              camCtx.restore();

              pts.forEach(p => {
                camCtx.beginPath();
                camCtx.arc(p.x, p.y, 6, 0, Math.PI*2);
                camCtx.fillStyle = '#FFFFFF';
                camCtx.shadowColor = 'rgba(0,0,0,0.3)';
                camCtx.shadowBlur = 6;
                camCtx.fill();
                camCtx.shadowBlur = 0;
              });

              const minX = Math.min(...pts.map(p=>p.x)), maxX = Math.max(...pts.map(p=>p.x));
              const minY = Math.min(...pts.map(p=>p.y)), maxY = Math.max(...pts.map(p=>p.y));
              lines.push('Ширина: ' + Math.round(maxX-minX) + 'px');
              lines.push('Высота: ' + Math.round(maxY-minY) + 'px');
            } else {
              lines.push('Объект не построен (вырожденные точки)');
            }
          } else {
            lastQuad = null;
            lines.push('✋ Сожми обе руки, чтобы получить объект');
          }
        } else {
          lastQuad = null;
          lines.push('👐 Нужно 2 руки в кадре');
        }
      } else {
        // frost
        if(!frostCanvas) setupFrost();

        const wipeRadius = 55;
        list.forEach(lm => {
          const c = handCenter(lm, vw, vh);
          wipeFrost(c.x, c.y, wipeRadius);
          lines.push('🧽 Протирание: X:' + Math.round(c.x) + ' Y:' + Math.round(c.y));
        });
        if(list.length === 0) lines.push('🖐️ Рука не найдена');

        camCtx.drawImage(frostCanvas, 0, 0);
      }

      logText.textContent = lines.join('\n');
    }

    // --- события кнопок ---
    camToggleBtn.addEventListener('click', startHandControl);
    camOffBtn.addEventListener('click', stopHandControl);

    modeBtn.addEventListener('click', ()=>{
      mode = (mode === 'stretch') ? 'frost' : 'stretch';
      if(mode === 'frost') setupFrost();
      lastQuad = null;
      // обновим лог
      const lines = [
        'Режим: ' + (mode==='stretch' ? 'Растягивание объекта' : 'Запотевшее стекло'),
        'FPS: ' + (fps || '—'),
        'Рука не найдена'
      ];
      logText.textContent = lines.join('\n');
    });

  })();
</script>
</body>
</html>
