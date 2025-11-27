<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Seni seviyorum -- Aýym</title>
<style>
  :root{
    --bg1:#0b1020;
    --bg2:#17082b;
    --accent1:#ff3d8a;
    --accent2:#ffb86b;
    --glow: 0 0 16px rgba(255,61,138,0.35);
  }
  *{box-sizing:border-box}
  html,body{height:100%; margin:0; font-family:Inter, Arial, sans-serif; -webkit-font-smoothing:antialiased}
  body{
    background: radial-gradient(1200px 800px at 10% 10%, rgba(255,61,138,0.06), transparent),
                radial-gradient(900px 600px at 90% 90%, rgba(255,184,107,0.04), transparent),
                linear-gradient(180deg,var(--bg1),var(--bg2));
    overflow:hidden;
    color: #fff;
    display:flex;
    align-items:center;
    justify-content:center;
  }

  /* Başlık */
  .title{
    position:absolute;
    top:32px;
    left:50%;
    transform:translateX(-50%);
    font-size:28px;
    letter-spacing:2px;
    color:var(--accent1);
    text-shadow: 0 0 10px rgba(255,61,138,0.6), 0 0 30px rgba(255,184,107,0.12);
    display:flex;
    gap:12px;
    align-items:center;
    z-index:5;
  }
  .title small{
    font-size:12px;
    color:#ffd6ea;
    opacity:0.9;
    transform: translateY(2px);
  }

  /* canvas kapsayıcı */
  #stage{
    position: absolute;
    inset: 0;
    width:100%;
    height:100%;
  }

  /* kontrol panel (opsiyonel küçük) */
  .hint{
    position:absolute;
    bottom:18px;
    left:50%;
    transform:translateX(-50%);
    background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(0,0,0,0.12));
    padding:8px 14px;
    border-radius:999px;
    font-size:13px;
    color:#ffdfe9;
    box-shadow: var(--glow);
    z-index:5;
  }

  /* ^ stil (canvas üzeri fallback) */
  .mz-float{
    position:absolute;
    pointer-events:none;
    font-weight:800;
    font-size:44px;
    transform:translate(-50%,-50%) scale(1);
    color:var(--accent1);
    text-shadow: 0 0 14px rgba(255,61,138,0.9), 0 0 28px rgba(255,184,107,0.6);
    will-change: transform, opacity;
    z-index:6;
  }

  @media (max-width:520px){
    .title{font-size:20px}
    .mz-float{font-size:36px}
  }
</style>
</head>
<body>
  <div class="title">💖 <strong>Seni Seviyorum</strong> <small>-- MZ Edition</small></div>
  <canvas id="stage"></canvas>
  <div class="hint">Dokun / Tıkla: Aýym çıkacak ✨</div>

<script>
/* -------------------------------------------
   Setup
--------------------------------------------*/
const canvas = document.getElementById('stage');
const ctx = canvas.getContext('2d', { alpha: true });

function resize() {
  canvas.width = innerWidth * devicePixelRatio;
  canvas.height = innerHeight * devicePixelRatio;
  canvas.style.width = innerWidth + 'px';
  canvas.style.height = innerHeight + 'px';
  ctx.setTransform(devicePixelRatio,0,0,devicePixelRatio,0,0);
}
addEventListener('resize', resize);
resize();

/* -------------------------------------------
   Utility
--------------------------------------------*/
function rand(min, max){ return Math.random()*(max-min)+min; }
function hsl(h,s,l){ return `hsl(${h},${s}%,${l}%)`; }
function easeOut(t){ return 1 - Math.pow(1-t, 3); }

/* -------------------------------------------
   Heart particle field (kalp formunda)
--------------------------------------------*/
const particles = [];
const PARTICLE_COUNT = Math.max(160, Math.floor((innerWidth*innerHeight)/25000)); // responsive

// parametre: t ∈ [0, 2π]
function heartPoint(t){
  const x = 16*Math.pow(Math.sin(t),3);
  const y = 13*Math.cos(t) - 5*Math.cos(2*t) - 2*Math.cos(3*t) - Math.cos(4*t);
  return {x, y};
}

// initialize particles on heart outline with slight jitter
for(let i=0;i<PARTICLE_COUNT;i++){
  const t = Math.random()*Math.PI*2;
  const p = heartPoint(t);
  const cx = innerWidth/2;
  const cy = innerHeight/2 - 20;
  particles.push({
    x: cx + p.x*18 + rand(-6,6),
    y: cy - p.y*18 + rand(-6,6),
    baseX: cx + p.x*18,
    baseY: cy - p.y*18,
    size: rand(1.2,3.2),
    hue: rand(320,360),
    alpha: rand(0.6,1),
    phase: Math.random()*Math.PI*2,
    speed: rand(0.002,0.01)
  });
}

/* animate heart particles with soft breathing */
let time = 0;
function drawParticles(){
  particles.forEach(p=>{
    const pulse = 1 + Math.sin(time* p.speed * 20 + p.phase) * 0.12;
    const tx = p.baseX + Math.sin(time*0.002 + p.phase)*8;
    const ty = p.baseY + Math.cos(time*0.002 + p.phase)*6;
    p.x += (tx - p.x) * 0.06;
    p.y += (ty - p.y) * 0.06;

    ctx.beginPath();
    const g = ctx.createRadialGradient(p.x, p.y, 0, p.x, p.y, p.size*6);
    g.addColorStop(0, `hsla(${p.hue},100%,65%,${p.alpha})`);
    g.addColorStop(0.5, `hsla(${p.hue+20},95%,55%,${p.alpha*0.5})`);
    g.addColorStop(1, `rgba(0,0,0,0)`);
    ctx.fillStyle = g;
    ctx.arc(p.x, p.y, p.size * pulse, 0, Math.PI*2);
    ctx.fill();
  });
}

/* -------------------------------------------
   Floating Z letters in background (canvas-rendered)
--------------------------------------------*/
const zs = [];
const Z_COUNT = Math.max(6, Math.floor(innerWidth / 160));
for(let i=0;i<Z_COUNT;i++){
  zs.push({
    x: rand(0, innerWidth),
    y: rand(0, innerHeight),
    vx: rand(-0.15, 0.15),
    vy: rand(-0.08, 0.08),
    size: rand(28,80),
    hue: rand(200,280),
    alpha: rand(0.06,0.14),
    ang: rand(0,Math.PI*2),
    rotSpeed: rand(-0.002,0.002)
  });
}

function drawZs(){
  zs.forEach(z=>{
    z.x += z.vx;
    z.y += z.vy;
    z.ang += z.rotSpeed;
    if(z.x < -100) z.x = innerWidth + 100;
    if(z.x > innerWidth + 100) z.x = -100;
    if(z.y < -120) z.y = innerHeight + 120;
    if(z.y > innerHeight + 120) z.y = -120;

    ctx.save();
    ctx.translate(z.x, z.y);
    ctx.rotate(z.ang);
    ctx.globalAlpha = z.alpha;
    ctx.font = `${z.size}px "Arial Black", Gadget, sans-serif`;
    ctx.fillStyle = `hsl(${z.hue}, 90%, 60%)`;
    ctx.fillText('Z', -z.size/2, z.size/3);
    ctx.restore();
  });
}

/* -------------------------------------------
   Interactive Aýym explosions (on touch/click)
--------------------------------------------*/
const mzBursts = []; // each burst spawns particles + floating Aýym text

function spawnAýym(x,y){
  // floating Aýym DOM-element (for crisp text + CSS glow)
  const el = document.createElement('div');
  el.className = 'ayym-float';
  el.style.left = x + 'px';
  el.style.top = y + 'px';
  el.style.opacity = '1';
  el.textContent = 'Aýym';
  document.body.appendChild(el);

  // animate with CSS-like timeline via JS
  const start = performance.now();
  const duration = 1200;
  const float = {el, start, duration};

  // particle burst (canvas)
  const burstParticles = [];
  const count = 28 + Math.floor(rand(0,18));
  for(let i=0;i<count;i++){
    const angle = rand(0, Math.PI*2);
    const speed = rand(1.8,5.6);
    burstParticles.push({
      x,y,
      vx: Math.cos(angle)*speed,
      vy: Math.sin(angle)*speed - rand(0,1.6),
      life: rand(600,1200),
      start: performance.now(),
      size: rand(2,6),
      hue: rand(300,360),
      alpha: 1,
      gravity: 0.04
    });
  }
  mzBursts.push({float, parts: burstParticles});
}

/* Input handling -- support mobile touch and desktop click */
function pointerToXY(e){
  if(e.touches && e.touches[0]) {
    return {x: e.touches[0].clientX, y: e.touches[0].clientY};
  } else if (e.changedTouches && e.changedTouches[0]) {
    return {x: e.changedTouches[0].clientX, y: e.changedTouches[0].clientY};
  } else {
    return {x: e.clientX, y: e.clientY};
  }
}

function handlePointer(e){
  const p = pointerToXY(e);
  spawnAýym(p.x, p.y);
}
window.addEventListener('click', handlePointer);
window.addEventListener('touchstart', (e) => { handlePointer(e); });

// draw mz bursts
function drawMzBursts(now){
  for(let i = mzBursts.length-1; i>=0; i--){
    const burst = mzBursts[i];
    // canvas particles
    for(let j=burst.parts.length-1;j>=0;j--){
      const part = burst.parts[j];
      const t = now - part.start;
      if(t > part.life){
        burst.parts.splice(j,1);
        continue;
      }
      // physics
      part.vy += part.gravity;
      part.x += part.vx;
      part.y += part.vy;
      const lifeRatio = 1 - (t/part.life);
      ctx.beginPath();
      const g = ctx.createRadialGradient(part.x, part.y, 0, part.x, part.y, part.size*6);
      g.addColorStop(0, `hsla(${part.hue},90%,60%,${0.9*lifeRatio})`);
      g.addColorStop(1, `rgba(0,0,0,0)`);
      ctx.fillStyle = g;
      ctx.arc(part.x, part.y, part.size * lifeRatio, 0, Math.PI*2);
      ctx.fill();
    }

    // floating DOM Aýym animation
    const tFloat = now - burst.float.start;
    const d = burst.float.duration;
    const el = burst.float.el;
    if(tFloat >= d){
      el.remove();
      mzBursts.splice(i,1);
      continue;
    } else {
      const pr = easeOut(tFloat/d);
      el.style.transform = `translate(-50%,-50%) translateY(${-70 * pr}px) scale(${1 + 0.6*pr})`;
      el.style.opacity = `${1 - pr}`;
    }
  }
}

/* -------------------------------------------
   Main render loop
--------------------------------------------*/
function render(now){
  time = now;
  // clear with slight transparent fill for motion trails
  ctx.clearRect(0,0,innerWidth,innerHeight);

  // subtle star background
  ctx.save();
  for(let s=0;s<120;s++){
    const sx = (Math.sin((now/1000 + s)*1.3 + s)*0.5+0.5)*innerWidth;
    const sy = (Math.cos((now/1200 + s)*0.7 + s)*0.5+0.5)*innerHeight;
    ctx.globalAlpha = 0.02;
    ctx.fillStyle = 'white';
    ctx.fillRect(sx, sy, 1,1);
  }
  ctx.restore();

  drawZs();
  drawParticles();
  drawMzBursts(now);

  requestAnimationFrame(render);
}

requestAnimationFrame(render);

/* -------------------------------------------
   auto demo pulses: click center occasionally (optional)
--------------------------------------------*/
let demoTimer = setInterval(()=>{
  spawnMZ(innerWidth/2 + rand(-80,80), innerHeight/2 + rand(-80,80));
}, 4200);

</script>
</body>
</html>
