# You-are-
<!doctype html>
<html lang="vi">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Chúc 20/10 — Trái Tim Bay Bay</title>
<style>
  :root{
    --bg:#0b1020;
    --accent:#ff6b9a;
    --soft:#ffdce6;
  }
  html,body{height:100%;margin:0;font-family: "Segoe UI", Roboto, "Helvetica Neue", Arial;}
  body{
    background: radial-gradient(1200px 600px at 10% 10%, rgba(255,107,154,0.05), transparent),
                linear-gradient(180deg, #071226 0%, var(--bg) 100%);
    color:var(--soft); overflow:hidden;
    display:flex;align-items:center;justify-content:center;
  }
  .stage{width:100%;height:100%;position:relative;display:flex;align-items:center;justify-content:center;flex-direction:column;}
  /* Heart container */
  .heart-wrap{
    position:relative;
    width:320px;height:320px;
    display:flex;align-items:center;justify-content:center;
  }
  /* SVG heart */
  .heart-svg{position:absolute;left:0;top:0;width:100%;height:100%;pointer-events:none;filter:drop-shadow(0 8px 24px rgba(255,107,154,0.15));}
  /* center message */
  .center-msg{
    position:absolute; width:220px; text-align:center; transform:translateY(18px);
    font-weight:600; font-size:18px; letter-spacing:0.6px; color:#fff9;
    text-shadow:0 1px 0 rgba(0,0,0,0.6);
  }
  .center-msg .big{font-size:22px;font-weight:700;color:var(--soft);display:block}
  /* bottom orbit text area hidden (used by canvas) */
  canvas{position:absolute;left:0;top:0;width:100%;height:100%;}
  /* controls (hidden on phone) */
  .hint{position:absolute;bottom:28px;font-size:13px;color:#aac;opacity:0.9}
  /* little spark styles when they gather */
  .spark{
    position:absolute;width:6px;height:6px;border-radius:50%;
    background:var(--accent); box-shadow:0 0 8px rgba(255,107,154,0.85);
    pointer-events:none; will-change:transform,opacity;
    mix-blend-mode:screen;
  }
  /* responsive */
  @media (max-width:420px){
    .heart-wrap{width:260px;height:260px}
    .center-msg{width:180px}
  }
</style>
</head>
<body>
<div class="stage">
  <div class="heart-wrap" id="heartWrap">
    <svg class="heart-svg" viewBox="0 0 100 100" preserveAspectRatio="xMidYMid meet">
      <defs>
        <linearGradient id="g1" x1="0" x2="1">
          <stop offset="0%" stop-color="#ff9bb8"/>
          <stop offset="100%" stop-color="#ff6b9a"/>
        </linearGradient>
        <filter id="glow" x="-50%" y="-50%" width="200%" height="200%">
          <feGaussianBlur stdDeviation="3.5" result="blur"/>
          <feMerge><feMergeNode in="blur"/><feMergeNode in="SourceGraphic"/></feMerge>
        </filter>
      </defs>
      <!-- Heart path -->
      <path id="heartPath" d="M50 86 L18 56 C6 44,10 28,28 26 C38 24,46 34,50 44 C54 34,62 24,72 26 C90 28,94 44,82 56 Z"
            fill="url(#g1)" filter="url(#glow)"/>
      <!-- outline -->
      <path d="M50 86 L18 56 C6 44,10 28,28 26 C38 24,46 34,50 44 C54 34,62 24,72 26 C90 28,94 44,82 56 Z"
            fill="none" stroke="rgba(255,255,255,0.06)" stroke-width="0.6"/>
    </svg>

    <canvas id="c"></canvas>

    <div class="center-msg">
      <span class="big">Chúc mừng 20/10</span>
      <span>Hồn nhiên như hoa, dịu dàng như nắng</span>
    </div>
  </div>

  <div class="hint">Chạm/nhấn để tái kích hoạt hiệu ứng</div>
</div>

<script>
/*
  Mô tả: hệ thống particle chữ + sao.
  1) Khởi tạo mảng "phrases" (lời chúc).
  2) Các chữ (small particles) chạy vòng tròn ở đáy trái tim (orbit).
  3) Sau một thời gian, chúng tách ra, bay lên theo đường cong vào tâm trái tim, chuyển thành "sao" nhỏ và hòa nhập vào trung tâm.
*/

const phrases = [
  "Mong em luôn tươi thắm 20/10",
  "Vạn sự an lành, nhan sắc diễm lệ",
  "Tâm nhàn thân thảnh, phúc cùng an khang",
  "Hồng nhan vô giá, thanh xuân bất lão",
  "Ngọc tình trường tồn, hoa lòng nở rộ"
];

const canvas = document.getElementById('c');
const ctx = canvas.getContext('2d');
const wrap = document.getElementById('heartWrap');

let DPR = window.devicePixelRatio || 1;
function resize(){
  const r = wrap.getBoundingClientRect();
  canvas.width = Math.floor(r.width * DPR);
  canvas.height = Math.floor(r.height * DPR);
  canvas.style.width = r.width + 'px';
  canvas.style.height = r.height + 'px';
  ctx.setTransform(DPR,0,0,DPR,0,0);
}
resize();
window.addEventListener('resize', ()=>{ resize(); init(); });

/* Orbit & heart geometry in canvas coordinates */
function heartCenter(){ const r = wrap.getBoundingClientRect(); return {x: r.width/2, y: r.height/2}; }
function orbitCenter(){ const r = wrap.getBoundingClientRect(); return {x: r.width/2, y: r.height*0.78}; }
function orbitRadius(){ const r = wrap.getBoundingClientRect(); return Math.min(r.width, r.height)*0.18; }

/* Pre-render phrase images to draw quickly */
function makeTextSprite(text){
  const f = 14; // px
  const tmp = document.createElement('canvas');
  const tctx = tmp.getContext('2d');
  tctx.font = `600 ${f}px system-ui, Roboto, "Segoe UI"`;
  const w = Math.ceil(tctx.measureText(text).width + 12);
  tmp.width = w; tmp.height = f+10;
  tctx.font = `600 ${f}px system-ui, Roboto, "Segoe UI"`;
  tctx.textBaseline = 'top';
  // glow
  tctx.fillStyle = "rgba(255,255,255,0.95)";
  // outline glow
  tctx.shadowColor = 'rgba(255,107,154,0.65)';
  tctx.shadowBlur = 8;
  tctx.fillText(text,6,0);
  return tmp;
}

const sprites = phrases.map(p => makeTextSprite(p));

/* Particle model */
class Particle {
  constructor(textSprite, angle){
    this.sprite = textSprite;
    this.angle = angle; // orbit angle (rad)
    this.phase = 'orbit'; // orbit -> launch -> gather
    this.orbitSpeed = 0.8 + Math.random()*0.6; // deg/sec
    this.orbitRadius = orbitRadius();
    const oc = orbitCenter();
    this.x = oc.x + Math.cos(angle)*this.orbitRadius;
    this.y = oc.y + Math.sin(angle)*this.orbitRadius;
    this.target = heartCenter();
    this.vx = 0; this.vy = 0;
    this.life = 0;
    this.alpha = 1;
    this.size = 1;
    // for star conversion
    this.isStar = false;
  }
  update(dt){
    this.life += dt;
    if(this.phase === 'orbit'){
      // rotate
      this.angle += dt * 0.9 * (this.orbitSpeed*0.6);
      const oc = orbitCenter();
      this.x = oc.x + Math.cos(this.angle)*this.orbitRadius;
      this.y = oc.y + Math.sin(this.angle)*this.orbitRadius;
    } else if(this.phase === 'launch'){
      // accelerate upward and towards heart
      const target = this.target;
      const dx = target.x - this.x;
      const dy = target.y - this.y;
      const dist = Math.max(8, Math.hypot(dx,dy));
      const ax = dx / (dist*0.08);
      const ay = dy / (dist*0.08) - 50 * (0.6 + Math.random()*0.8); // give upward thrust
      this.vx += ax * dt * 60;
      this.vy += ay * dt * 60;
      this.vx *= 0.98; this.vy *= 0.98;
      this.x += this.vx*dt*60;
      this.y += this.vy*dt*60;
      // fade and shrink when close
      if(dist < 18){
        this.phase = 'gather';
        this.isStar = true;
        this.size = 0.9;
      }
    } else if(this.phase === 'gather'){
      // gently settle to center then vanish
      const target = this.target;
      this.x += (target.x - this.x)*0.14*dt*60;
      this.y += (target.y - this.y)*0.14*dt*60;
      this.alpha -= 0.01 * dt * 60;
      if(this.alpha < 0) this.alpha = 0;
    }
  }
  draw(ctx){
    ctx.save();
    ctx.globalAlpha = this.alpha;
    if(this.isStar){
      // draw small star dot
      ctx.beginPath();
      ctx.arc(this.x, this.y, 2.4*this.size, 0, Math.PI*2);
      ctx.fillStyle = "rgba(255,107,154,"+ (0.9*this.alpha) +")";
      ctx.fill();
      ctx.closePath();
    } else {
      // draw the text sprite centered
      const s = this.sprite;
      const w = s.width; const h = s.height;
      ctx.drawImage(s, this.x - w/2, this.y - h/2 - 6);
    }
    ctx.restore();
  }
}

/* System state */
let particles = [];
function init(){
  particles = [];
  const count = phrases.length;
  const oc = orbitCenter();
  const r = orbitRadius();
  for(let i=0;i<count;i++){
    const angle = (i / count) * Math.PI*2 + (Math.random()*0.3);
    const p = new Particle(sprites[i%sprites.length], angle);
    particles.push(p);
  }
}
init();

/* Time loop */
let last = performance.now();
let launchTimer = 0;
let cycle = 0;

function step(now){
  const dt = Math.min(0.06, (now - last)/1000);
  last = now;
  launchTimer += dt;
  // every 3.2s launch one phrase, until all launched
  if(launchTimer > 2.2 && cycle < particles.length){
    particles[cycle].phase = 'launch';
    launchTimer = 0;
    cycle++;
  }
  // after all launched and 2s later, reset
  const allGone = particles.every(p => p.alpha <= 0 || p.phase === 'gather' && p.alpha < 0.02);
  if(cycle >= particles.length && allGone){
    // create glow sparks briefly
    for(let i=0;i<15;i++) createSpark();
    setTimeout(()=>{ resetAnimation(); }, 900);
  }

  updateParticles(dt);
  render();
  requestAnimationFrame(step);
}

function updateParticles(dt){
  for(const p of particles) p.update(dt);
  // update sparks
  for(let i=sparks.length-1;i>=0;i--){
    sparks[i].tick(dt);
    if(sparks[i].dead) sparks.splice(i,1);
  }
}

/* Render */
function render(){
  ctx.clearRect(0,0,canvas.width/DPR,canvas.height/DPR);
  // slight vignette
  const hw = canvas.width/DPR, hh = canvas.height/DPR;
  // draw characters / particles
  for(const p of particles) p.draw(ctx);
  // draw sparks
  for(const s of sparks) s.draw(ctx);
}

function resetAnimation(){
  // gently reinitialize orbit positions & bring back alphas
  init();
  last = performance.now();
  launchTimer = 0; cycle = 0;
}

/* Sparks: decorative stars that appear when gathering */
const sparks = [];
class Spark {
  constructor(x,y){
    this.x = x; this.y = y;
    this.vx = (Math.random()-0.5)*80;
    this.vy = (Math.random()-1.5)*80;
    this.life = 0.7 + Math.random()*0.8;
    this.age = 0;
    this.dead = false;
    this.size = 2 + Math.random()*2;
  }
  tick(dt){
    this.age += dt;
    if(this.age > this.life) { this.dead=true; return; }
    this.x += this.vx * dt;
    this.y += this.vy * dt;
    this.vy += 120 * dt; // gravity
  }
  draw(ctx){
    ctx.save();
    const a = Math.max(0, 1 - this.age/this.life);
    ctx.globalAlpha = a;
    ctx.beginPath();
    ctx.arc(this.x, this.y, this.size * a, 0, Math.PI*2);
    ctx.fillStyle = "rgba(255,220,230,0.95)";
    ctx.fill();
    ctx.restore();
  }
}
function createSpark(){
  const c = heartCenter();
  for(let i=0;i<10;i++){
    sparks.push(new Spark(c.x + (Math.random()-0.5)*50, c.y + (Math.random()-0.5)*40));
  }
}

/* Interaction to replay */
canvas.addEventListener('click', ()=>{ resetAnimation(); });
canvas.addEventListener('touchstart', (e)=>{ e.preventDefault(); resetAnimation(); }, {passive:false});

/* start */
requestAnimationFrame(step);

/* Accessibility: adapt DPR when display changes */
</script>
</body>
</html>
