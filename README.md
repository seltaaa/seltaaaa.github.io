<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Управление рукой</title>
<style>
  :root{
    --accent1:#6d5dfc;
    --accent2:#00d2b8;
    --bg:#eef1f7;
    --card:#ffffff;
    --ink:#1c2333;
    --muted:#6b7280;
    --log-bg:#0f1526;
    --log-line:#8fa3c7;
    --log-pos:#4fd1ff;
    --log-good:#4ade80;
    --log-bad:#fb7185;
  }
  *{ box-sizing:border-box; }
  body{
    font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,sans-serif;
    margin:0;
    background:linear-gradient(180deg,#f4f6fb,#e7ebf5 260px, var(--bg) 260px);
    color:var(--ink);
  }

  #topPanel{
    display:flex;
    gap:16px;
    padding:16px;
    flex-wrap:wrap;
    align-items:flex-start;
  }

  .card{
    background:var(--card);
    border-radius:16px;
    box-shadow:0 6px 24px rgba(20,25,50,0.08);
    padding:14px;
  }

  #camSection{ flex:0 0 auto; width:100%; max-width:480px; }
  #camBox{
    position:relative;
    width:100%;
    aspect-ratio:4/3;
    background:#000;
    border-radius:14px;
    overflow:hidden;
  }
  #camBox video{ display:none; }
  #camBox canvas{
    width:100%; height:100%;
    display:block;
    transform:scaleX(-1);
  }
  #camHint{
    position:absolute;
    inset:0;
    display:flex;
    align-items:center;
    justify-content:center;
    color:#9aa4c2;
    font-size:13px;
    text-align:center;
    padding:20px;
    pointer-events:none;
  }
  #camHint.hidden{ display:none; }

  #modeBar{
    display:flex;
    gap:6px;
    margin-top:10px;
    flex-wrap:wrap;
  }
  .modeBtnEl{
    flex:1 1 auto;
    border:none;
    padding:8px 10px;
    border-radius:10px;
    background:#eef0f8;
    color:var(--ink);
    font-size:12.5px;
    font-weight:600;
    cursor:pointer;
    transition:all .15s ease;
    white-space:nowrap;
  }
  .modeBtnEl.active{
    background:linear-gradient(135deg,var(--accent1),var(--accent2));
    color:#fff;
    box-shadow:0 4px 14px rgba(109,93,252,0.35);
  }

  #camButtons{ margin-top:10px; display:flex; gap:8px; flex-wrap:wrap; }
  #camButtons button{
    padding:9px 14px;
    border:none;
    border-radius:10px;
    font-size:13px;
    font-weight:600;
    cursor:pointer;
    background:var(--ink);
    color:#fff;
  }
  #camButtons button:disabled{ opacity:.5; cursor:default; }
  #camOffBtn{ background:#ef4444 !important; }

  #logSection{ flex:1; min-width:260px; display:flex; flex-direction:column; gap:10px; }
  #logSection h3{ margin:0; font-size:13px; color:var(--muted); font-weight:700; letter-spacing:.02em; text-transform:uppercase; }

  #statusRow{ display:flex; gap:8px; flex-wrap:wrap; margin-bottom:2px; }
  .statChip{
    background:var(--log-bg);
    color:#cfe0ff;
    font-family:ui-monospace,Menlo,Consolas,monospace;
    font-size:12px;
    padding:6px 10px;
    border-radius:8px;
    display:flex;
    gap:6px;
    align-items:center;
  }
  .statChip b{ color:#fff; }
  .dot{ width:7px; height:7px; border-radius:50%; background:var(--log-bad); }
  .dot.on{ background:var(--log-good); box-shadow:0 0 8px var(--log-good); }

  #logFeed{
    background:var(--log-bg);
    border-radius:12px;
    padding:10px 12px;
    height:260px;
    overflow-y:auto;
    font-family:ui-monospace,Menlo,Consolas,monospace;
    font-size:11.5px;
    line-height:1.65;
    scroll-behavior:smooth;
  }
  #logFeed::-webkit-scrollbar{ width:6px; }
  #logFeed::-webkit-scrollbar-thumb{ background:#2a3557; border-radius:4px; }
  .logLine{
    color:var(--log-line);
    white-space:pre;
    animation:fadeIn .25s ease;
  }
  .logLine .t{ color:#4a5578; margin-right:6px; }
  .logLine .pos{ color:var(--log-pos); }
  .logLine .good{ color:var(--log-good); }
  .logLine .bad{ color:var(--log-bad); }
  @keyframes fadeIn{ from{ opacity:0; transform:translateY(-2px);} to{ opacity:1; transform:none; } }

  main{ padding:0 16px 30px; max-width:900px; }
  main .card{ margin-bottom:14px; }
  main h1{ font-size:19px; margin:6px 0 14px; }
  main p{ font-size:14px; line-height:1.55; margin:0 0 10px; color:#2b3350; }
  main p:last-child{ margin-bottom:0; }
  main b{ color:var(--accent1); }

  #drawSection{ display:none; }
  #drawSection.visible{ display:block; }
  #drawTopRow{ display:flex; justify-content:space-between; align-items:center; margin-bottom:8px; }
  #drawTopRow button{
    border:none; padding:7px 12px; border-radius:8px; font-size:12.5px; font-weight:600;
    cursor:pointer; background:#eef0f8; color:var(--ink);
  }
  #drawCanvas{
    width:100%;
    aspect-ratio:16/9;
    background:#fff;
    border-radius:12px;
    border:2px dashed #d7dcec;
    touch-action:none;
  }

  #handCursor{
    position:fixed;
    width:26px; height:26px;
    margin-left:-13px; margin-top:-13px;
    border-radius:50%;
    border:2px solid var(--accent1);
    background:rgba(109,93,252,0.15);
    pointer-events:none;
    z-index:99999;
    display:none;
    transition:transform .05s linear, background .1s ease, border-color .1s ease;
  }
  #handCursor.pinching{
    background:rgba(0,210,184,0.35);
    border-color:var(--accent2);
    transform:scale(1.3);
  }
</style>
</head>
<body>

  <div id="handCursor"></div>

  <div id="topPanel">
    <div id="camSection">
      <div class="card">
        <div id="camBox">
          <video id="inputVideo" playsinline muted></video>
          <canvas id="camCanvas"></canvas>
          <div id="camHint">Нажми «Включить камеру»</div>
        </div>
        <div id="modeBar">
          <button class="modeBtnEl active" data-mode="stretch">Растягивание</button>
          <button class="modeBtnEl" data-mode="frost">Запотевшее стекло</button>
          <button class="modeBtnEl" data-mode="draw">Рука-мышка / рисование</button>
        </div>
        <div id="camButtons">
          <button id="camToggleBtn">Включить камеру</button>
          <button id="camOffBtn" style="display:none;">Выключить камеру</button>
        </div>
      </div>
    </div>

    <div id="logSection">
      <h3>Логи и данные камеры</h3>
      <div class="card" style="padding:12px;">
        <div id="statusRow">
          <div class="statChip"><span class="dot" id="camDot"></span><span>Камера</span></div>
          <div class="statChip">Режим: <b id="statMode">Растягивание</b></div>
          <div class="statChip">FPS: <b id="statFps">-</b></div>
          <div class="statChip">Рук: <b id="statHands">0</b></div>
        </div>
        <div id="logFeed"></div>
      </div>
    </div>
  </div>

  <main>
    <div class="card">
      <h1>Режимы</h1>
      <p><b>Растягивание:</b> сожми большой и указательный палец на каждой руке — между четырьмя пальцами появится объект. Один раз щипни обеими руками, чтобы "схватить" его — дальше можно свободно двигать руками и растягивать, не удерживая пальцы идеально сжатыми. Чтобы отпустить — раскрой ладони пошире.</p>
      <p><b>Запотевшее стекло:</b> камера "запотевает", и тебя не видно. Проведи рукой перед камерой — в этом месте запотевание исчезнет.</p>
      <p><b>Рука-мышка / рисование:</b> указательный палец становится курсором на экране (как мышка). Щипни большим и указательным — это "клик". Если курсор наведён на поле для рисования ниже — щипок рисует линию; если на кнопку режима — щипок нажимает её.</p>
    </div>

    <div id="drawSection" class="card">
      <div id="drawTopRow">
        <h3 style="margin:0;font-size:13px;color:var(--muted);text-transform:uppercase;">Поле для рисования</h3>
        <button id="drawClearBtn">Очистить</button>
      </div>
      <canvas id="drawCanvas"></canvas>
    </div>
  </main>

<script src="https://cdn.jsdelivr.net/npm/@mediapipe/drawing_utils/drawing_utils.js" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js" crossorigin="anonymous"></script>
<script>
(function(){
  const camToggleBtn = document.getElementById('camToggleBtn');
  const camOffBtn = document.getElementById('camOffBtn');
  const camHint = document.getElementById('camHint');
  const camDot = document.getElementById('camDot');
  const modeBar = document.getElementById('modeBar');
  const videoEl = document.getElementById('inputVideo');
  const camCanvas = document.getElementById('camCanvas');
  const camCtx = camCanvas.getContext('2d');

  const statMode = document.getElementById('statMode');
  const statFps = document.getElementById('statFps');
  const statHands = document.getElementById('statHands');
  const logFeed = document.getElementById('logFeed');

  const drawSection = document.getElementById('drawSection');
  const drawCanvas = document.getElementById('drawCanvas');
  const drawClearBtn = document.getElementById('drawClearBtn');
  const handCursor = document.getElementById('handCursor');

  let hands = null;
  let mediaStream = null;
  let running = false;
  let rafId = null;
  let latestResults = null;

  let lastTs = performance.now();
  let fpsFrames = 0, fpsAccum = 0, fps = 0;

  let mode = 'stretch';
  const MODE_LABELS = { stretch:'Растягивание', frost:'Запотевшее стекло', draw:'Рука-мышка' };
  let frostCanvas = null;
  let frostCtx = null;

  let detectBusy = false;

  let lastQuad = null;

  const PINCH_ON = 0.65;
  const PINCH_OFF = 1.0;
  const pinchStates = {};
  function updatePinch(label, dist, scale){
    const prev = !!pinchStates[label];
    const next = prev ? (dist < scale * PINCH_OFF) : (dist < scale * PINCH_ON);
    pinchStates[label] = next;
    return next;
  }

  let drawCtx = null;
  let isPenDown = false;
  let penIsDrawing = false;
  let lastPenPoint = null;

  function setActiveModeButton(){
    [...modeBar.querySelectorAll('.modeBtnEl')].forEach(b=>{
      b.classList.toggle('active', b.dataset.mode === mode);
    });
    drawSection.classList.toggle('visible', mode === 'draw');
    handCursor.style.display = (mode === 'draw' && running) ? 'block' : 'none';
    statMode.textContent = MODE_LABELS[mode];
  }

  modeBar.addEventListener('click', (e)=>{
    const btn = e.target.closest('.modeBtnEl');
    if(!btn) return;
    setMode(btn.dataset.mode);
  });

  function setMode(newMode){
    mode = newMode;
    if(mode === 'frost'){ setupFrost(); }
    lastQuad = null;
    penIsDrawing = false;
    lastPenPoint = null;
    setActiveModeButton();
    appendLog('Режим переключён: ' + MODE_LABELS[mode], 'good');
  }
  setActiveModeButton();

  function resizeDrawCanvas(){
    const rect = drawCanvas.getBoundingClientRect();
    if(rect.width > 0 && (drawCanvas.width !== Math.round(rect.width) || drawCanvas.height !== Math.round(rect.height))){
      const prev = drawCtx ? drawCtx.getImageData(0,0,drawCanvas.width,drawCanvas.height) : null;
      drawCanvas.width = Math.round(rect.width);
      drawCanvas.height = Math.round(rect.height);
      drawCtx = drawCanvas.getContext('2d');
      drawCtx.lineCap = 'round';
      drawCtx.lineJoin = 'round';
    }
  }
  window.addEventListener('resize', resizeDrawCanvas);
  drawClearBtn.addEventListener('click', ()=>{
    resizeDrawCanvas();
    if(drawCtx) drawCtx.clearRect(0,0,drawCanvas.width,drawCanvas.height);
    appendLog('Поле для рисования очищено', 'good');
  });

  let lastLogPush = 0;
  const LOG_THROTTLE_MS = 160;
  const MAX_LOG_LINES = 140;
  function appendLog(html, cls){
    const line = document.createElement('div');
    line.className = 'logLine' + (cls ? ' ' + cls : '');
    const t = new Date();
    const ts = String(t.getHours()).padStart(2,'0') + ':' + String(t.getMinutes()).padStart(2,'0') + ':' + String(t.getSeconds()).padStart(2,'0') + '.' + String(t.getMilliseconds()).padStart(3,'0');
    line.innerHTML = '<span class="t">' + ts + '</span>' + html;
    logFeed.appendChild(line);
    while(logFeed.children.length > MAX_LOG_LINES){ logFeed.removeChild(logFeed.firstChild); }
    logFeed.scrollTop = logFeed.scrollHeight;
  }
  function appendTrackingLog(text){
    const now = performance.now();
    if(now - lastLogPush < LOG_THROTTLE_MS) return;
    lastLogPush = now;
    appendLog(text);
  }

  function convexHull(points){
    const pts = points.slice().sort((a,b)=> a.x - b.x || a.y - b.y);
    const cross = (o,a,b)=> (a.x-o.x)*(b.y-o.y) - (a.y-o.y)*(b.x-o.x);
    const lower = [];
    for(const p of pts){
      while(lower.length>=2 && cross(lower[lower.length-2], lower[lower.length-1], p) <= 0) lower.pop();
      lower.push(p);
    }
    const upper = [];
    for(let i=pts.length-1;i>=0;i--){
      const p = pts[i];
      while(upper.length>=2 && cross(upper[upper.length-2], upper[upper.length-1], p) <= 0) upper.pop();
      upper.push(p);
    }
    lower.pop(); upper.pop();
    return lower.concat(upper);
  }

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
    idxs.forEach(i=>{ sx += lm[i].x; sy += lm[i].y; });
    return { x:(sx/idxs.length)*w, y:(sy/idxs.length)*h };
  }

  function setupFrost(){
    const vw = videoEl.videoWidth || 640, vh = videoEl.videoHeight || 480;
    frostCanvas = document.createElement('canvas');
    frostCanvas.width = vw; frostCanvas.height = vh;
    frostCtx = frostCanvas.getContext('2d');
    frostCtx.fillStyle = 'rgba(225,230,235,0.96)';
    frostCtx.fillRect(0, 0, vw, vh);
  }

  function wipeFrost(x, y, radius){
    if(!frostCtx) return;
    frostCtx.save();
    frostCtx.globalCompositeOperation = 'destination-out';
    const grad = frostCtx.createRadialGradient(x, y, 0, x, y, radius);
    grad.addColorStop(0, 'rgba(0,0,0,1)');
    grad.addColorStop(1, 'rgba(0,0,0,0)');
    frostCtx.fillStyle = grad;
    frostCtx.beginPath();
    frostCtx.arc(x, y, radius, 0, Math.PI*2);
    frostCtx.fill();
    frostCtx.restore();
  }

  function drawStretchObject(ctx, pts, now){
    const cx = pts.reduce((s,p)=>s+p.x,0)/pts.length;
    const cy = pts.reduce((s,p)=>s+p.y,0)/pts.length;
    const hue = (now/20) % 360;

    ctx.save();
    ctx.shadowColor = `hsla(${hue},90%,60%,0.9)`;
    ctx.shadowBlur = 28;

    const grad = ctx.createLinearGradient(pts[0].x, pts[0].y, pts[2%pts.length].x, pts[2%pts.length].y);
    grad.addColorStop(0, `hsla(${hue},90%,65%,0.55)`);
    grad.addColorStop(0.5, `hsla(${(hue+60)%360},90%,60%,0.4)`);
    grad.addColorStop(1, `hsla(${(hue+120)%360},90%,65%,0.55)`);

    ctx.beginPath();
    ctx.moveTo(pts[0].x, pts[0].y);
    for(let i=1;i<pts.length;i++) ctx.lineTo(pts[i].x, pts[i].y);
    ctx.closePath();
    ctx.fillStyle = grad;
    ctx.fill();

    ctx.lineWidth = 2.5;
    ctx.strokeStyle = `hsla(${hue},95%,80%,0.95)`;
    ctx.setLineDash([10,6]);
    ctx.lineDashOffset = -(now/15) % 16;
    ctx.stroke();
    ctx.setLineDash([]);
    ctx.restore();

    const pulse = 4.5 + Math.sin(now/150) * 1.5;
    pts.forEach(p=>{
      ctx.save();
      ctx.shadowColor = `hsla(${hue},95%,70%,0.9)`;
      ctx.shadowBlur = 12;
      ctx.beginPath();
      ctx.arc(p.x, p.y, pulse, 0, Math.PI*2);
      ctx.fillStyle = '#ffffff';
      ctx.fill();
      ctx.restore();
    });

    ctx.save();
    ctx.globalAlpha = 0.5 + Math.sin(now/120)*0.2;
    ctx.beginPath();
    ctx.arc(cx, cy, 3, 0, Math.PI*2);
    ctx.fillStyle = `hsla(${hue},100%,85%,1)`;
    ctx.fill();
    ctx.restore();
  }

  camToggleBtn.addEventListener('click', startHandControl);
  camOffBtn.addEventListener('click', stopHandControl);

  async function ensureHands(){
    if(hands) return;
    if(typeof Hands === 'undefined'){
      throw new Error('Библиотека распознавания рук не загрузилась (проверь интернет / доступ к cdn.jsdelivr.net).');
    }
    hands = new Hands({ locateFile:(f)=>`https://cdn.jsdelivr.net/npm/@mediapipe/hands/${f}` });
    hands.setOptions({ maxNumHands:2, modelComplexity:0, minDetectionConfidence:0.5, minTrackingConfidence:0.5 });
    hands.onResults((r)=>{ latestResults = r; });
  }

  async function startHandControl(){
    camToggleBtn.disabled = true;
    try{
      await ensureHands();
      mediaStream = await navigator.mediaDevices.getUserMedia({
        video:{ width:{ideal:640}, height:{ideal:480}, facingMode:'user' },
        audio:false
      });
      videoEl.srcObject = mediaStream;
      await videoEl.play();

      camToggleBtn.style.display = 'none';
      camOffBtn.style.display = 'inline-block';
      camHint.classList.add('hidden');
      camDot.classList.add('on');
      running = true;
      lastTs = performance.now();
      resizeDrawCanvas();
      setActiveModeButton();
      appendLog('Камера включена', 'good');
      loop();
    } catch(err){
      console.error(err);
      alert('Не удалось включить камеру: ' + (err.message || err.name));
      camToggleBtn.disabled = false;
      appendLog('Ошибка запуска камеры: ' + (err.message || err.name), 'bad');
    }
  }

  function stopHandControl(){
    running = false;
    if(rafId) cancelAnimationFrame(rafId);
    if(mediaStream){ mediaStream.getTracks().forEach(t=>t.stop()); mediaStream = null; }
    camToggleBtn.style.display = 'inline-block';
    camToggleBtn.disabled = false;
    camOffBtn.style.display = 'none';
    camHint.classList.remove('hidden');
    camDot.classList.remove('on');
    lastQuad = null;
    penIsDrawing = false;
    lastPenPoint = null;
    handCursor.style.display = 'none';
    statFps.textContent = '-';
    statHands.textContent = '0';
    appendLog('Камера выключена', 'bad');
  }

  async function loop(){
    if(!running) return;

    const now = performance.now();
    const dt = (now - lastTs) / 1000;
    lastTs = now;
    fpsFrames++; fpsAccum += dt;
    if(fpsAccum > 0.5){
      fps = Math.round(fpsFrames / fpsAccum);
      fpsFrames = 0; fpsAccum = 0;
      statFps.textContent = fps;
    }

    const vw = videoEl.videoWidth, vh = videoEl.videoHeight;
    if(vw && vh){
      if(camCanvas.width !== vw || camCanvas.height !== vh){
        camCanvas.width = vw; camCanvas.height = vh;
      }
      camCtx.clearRect(0,0,vw,vh);
      camCtx.drawImage(videoEl, 0, 0, vw, vh);
    }

    if(!detectBusy && vw && vh){
      detectBusy = true;
      try{
        await hands.send({image: videoEl});
      } catch(e){
        console.error('hand detection error', e);
      }
      detectBusy = false;
    }

    processFrame(vw, vh, now);

    rafId = requestAnimationFrame(loop);
  }

  function processFrame(vw, vh, now){
    const list = (latestResults && latestResults.multiHandLandmarks) ? latestResults.multiHandLandmarks : [];
    const handedness = (latestResults && latestResults.multiHandedness) ? latestResults.multiHandedness : [];
    statHands.textContent = list.length;

    if(!vw || !vh){ return; }

    list.forEach(lm=>{
      if(window.drawConnectors && window.HAND_CONNECTIONS){
        window.drawConnectors(camCtx, lm, window.HAND_CONNECTIONS, { color:'#00d2b8', lineWidth:2 });
        window.drawLandmarks(camCtx, lm, { color:'#6d5dfc', fillColor:'#6d5dfc', radius:2 });
      }
    });

    if(list.length === 0){
      appendTrackingLog('<span class="bad">Рука не найдена</span>');
    } else {
      list.forEach((lm, i)=>{
        const c = handCenter(lm, vw, vh);
        const label = (handedness[i] && handedness[i].label) ? handedness[i].label : ('#'+i);
        appendTrackingLog('Рука ' + label + ' <span class="pos">X:' + Math.round(c.x) + ' Y:' + Math.round(c.y) + '</span>');
      });
    }

    if(mode === 'stretch'){
      processStretch(list, handedness, vw, vh, now);
    } else if(mode === 'frost'){
      processFrost(list, vw, vh);
    } else if(mode === 'draw'){
      processHandCursor(list, vw, vh, now);
    }
  }

  function processStretch(list, handedness, vw, vh, now){
    if(list.length >= 2){
      const lmA = list[0], lmB = list[1];
      const labelA = (handedness[0] && handedness[0].label) ? handedness[0].label : 'A';
      const labelB = (handedness[1] && handedness[1].label) ? handedness[1].label : 'B';
      const scaleA = handScale(lmA, vw, vh), scaleB = handScale(lmB, vw, vh);
      const distA = pinchDist(lmA, vw, vh), distB = pinchDist(lmB, vw, vh);
      const pinchA = updatePinch(labelA, distA, scaleA);
      const pinchB = updatePinch(labelB, distB, scaleB);

      appendTrackingLog('Щипок ' + labelA + ': <span class="' + (pinchA?'good':'bad') + '">' + (pinchA?'да':'нет') + '</span> · ' + labelB + ': <span class="' + (pinchB?'good':'bad') + '">' + (pinchB?'да':'нет') + '</span>');

      if(pinchA && pinchB){
        const thumbA = { x: lmA[4].x*vw, y: lmA[4].y*vh };
        const indexA = { x: lmA[8].x*vw, y: lmA[8].y*vh };
        const thumbB = { x: lmB[4].x*vw, y: lmB[4].y*vh };
        const indexB = { x: lmB[8].x*vw, y: lmB[8].y*vh };
        const rawPts = [thumbA, indexA, thumbB, indexB];
        let pts = convexHull(rawPts);

        if(pts.length < 3 && lastQuad){
          pts = lastQuad;
        } else {
          lastQuad = pts;
        }

        drawStretchObject(camCtx, pts, now);

        const minX = Math.min(...pts.map(p=>p.x)), maxX = Math.max(...pts.map(p=>p.x));
        const minY = Math.min(...pts.map(p=>p.y)), maxY = Math.max(...pts.map(p=>p.y));
        appendTrackingLog('Ширина: <span class="pos">' + Math.round(maxX-minX) + 'px</span> Высота: <span class="pos">' + Math.round(maxY-minY) + 'px</span>');
      } else {
        lastQuad = null;
      }
    } else {
      lastQuad = null;
      Object.keys(pinchStates).forEach(k=>delete pinchStates[k]);
    }
  }

  function processFrost(list, vw, vh){
    if(!frostCanvas) setupFrost();
    const wipeRadius = 55;
    list.forEach(lm=>{
      const c = handCenter(lm, vw, vh);
      wipeFrost(c.x, c.y, wipeRadius);
    });
    camCtx.drawImage(frostCanvas, 0, 0);
  }

  function processHandCursor(list, vw, vh, now){
    if(list.length === 0){
      handCursor.style.display = 'none';
      penIsDrawing = false;
      lastPenPoint = null;
      return;
    }
    handCursor.style.display = 'block';
    const lm = list[0];
    const label = (latestResults.multiHandedness[0] && latestResults.multiHandedness[0].label) ? latestResults.multiHandedness[0].label : 'A';
    const tip = lm[8];

    const mirroredNX = 1 - tip.x;
    const pageX = mirroredNX * window.innerWidth;
    const pageY = tip.y * window.innerHeight;
    handCursor.style.left = pageX + 'px';
    handCursor.style.top = pageY + 'px';

    const scale = handScale(lm, vw, vh);
    const dist = pinchDist(lm, vw, vh);
    const pinching = updatePinch(label, dist, scale);
    handCursor.classList.toggle('pinching', pinching);

    appendTrackingLog('Курсор <span class="pos">X:' + Math.round(pageX) + ' Y:' + Math.round(pageY) + '</span> · щипок: <span class="' + (pinching?'good':'bad') + '">' + (pinching?'да':'нет') + '</span>');

    resizeDrawCanvas();
    const padRect = drawCanvas.getBoundingClientRect();
    const overPad = pageX >= padRect.left && pageX <= padRect.right && pageY >= padRect.top && pageY <= padRect.bottom;

    if(pinching && !isPenDown){
      if(overPad && drawCtx){
        penIsDrawing = true;
        lastPenPoint = { x: pageX - padRect.left, y: pageY - padRect.top };
      } else {
        penIsDrawing = false;
        const el = document.elementFromPoint(pageX, pageY);
        if(el && el.closest('button')){ el.closest('button').click(); }
      }
    } else if(pinching && isPenDown && penIsDrawing && overPad && drawCtx){
      const p = { x: pageX - padRect.left, y: pageY - padRect.top };
      drawCtx.strokeStyle = '#6d5dfc';
      drawCtx.lineWidth = 3;
      drawCtx.beginPath();
      drawCtx.moveTo(lastPenPoint.x, lastPenPoint.y);
      drawCtx.lineTo(p.x, p.y);
      drawCtx.stroke();
      lastPenPoint = p;
    } else if(!pinching){
      penIsDrawing = false;
      lastPenPoint = null;
    }
    isPenDown = pinching;
  }
})();
</script>
</body>
</html>
