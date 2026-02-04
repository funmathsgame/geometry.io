<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Reaction Zoo</title>
  <style>
    :root{
      --bg:#0f1220; --card:#171a2e; --accent:#7cf3c0; --accent2:#6aa8ff; --text:#eef1ff; --muted:#9aa3ff55;
      --radius:18px;
    }
    [data-theme="light"]{ --bg:#f6f7fb; --card:#ffffff; --text:#111; --muted:#0002; }
    body{ margin:0; font-family:system-ui,-apple-system,Segoe UI,Roboto; background:radial-gradient(1200px 600px at 20% -10%,#1b2050,transparent),var(--bg); color:var(--text); }
    header{ display:flex; align-items:center; justify-content:space-between; padding:16px 20px; }
    .logo{ font-weight:800; letter-spacing:.4px; }
    .pill{ border-radius:999px; padding:8px 14px; background:linear-gradient(135deg,var(--accent),var(--accent2)); color:#08131a; border:none; cursor:pointer; }
    .wrap{ max-width:1200px; margin:auto; padding:16px; }
    .grid{ display:grid; grid-template-columns:repeat(auto-fit,minmax(220px,1fr)); gap:14px; }
    .card{ background:linear-gradient(180deg,#ffffff08,#00000008),var(--card); border-radius:var(--radius); padding:16px; box-shadow:0 10px 30px #0005; transition:transform .2s ease, box-shadow .2s ease; }
    .card:hover{ transform:translateY(-3px); box-shadow:0 16px 40px #0007; }
    h2{ margin:.2em 0 .6em; }
    .btn{ border-radius:14px; padding:10px 14px; border:1px solid var(--muted); background:#0000; color:var(--text); cursor:pointer; }
    .btn.primary{ background:linear-gradient(135deg,var(--accent),var(--accent2)); color:#08131a; border:none; }
    .row{ display:flex; gap:10px; flex-wrap:wrap; }
    .progress{ height:10px; background:#0004; border-radius:999px; overflow:hidden; }
    .bar{ height:100%; width:0%; background:linear-gradient(90deg,var(--accent),var(--accent2)); transition:width .2s linear; }
    .center{ text-align:center; }
    .big{ font-size:2.2rem; font-weight:800; }
    .hidden{ display:none; }
    canvas{ width:100%; height:140px; }
    /* confetti */
    .confetti{ position:fixed; inset:0; pointer-events:none; }
  </style>
</head>
<body data-theme="dark">
<header>
  <div class="logo">🦁 Reaction Zoo</div>
  <div class="row">
    <button class="btn" onclick="toggleTheme()">Theme</button>
    <button class="pill" onclick="dailyBonus()">Daily Bonus</button>
  </div>
</header>

<div class="wrap">
  <div class="grid">
    <div class="card">
      <h2>Play</h2>
      <p>Five reaction games • difficulties • streaks • rewards</p>
      <div class="row">
        <button class="btn primary" onclick="startGame('tap')">Tap</button>
        <button class="btn primary" onclick="startGame('color')">Color</button>
        <button class="btn primary" onclick="startGame('sound')">Sound</button>
        <button class="btn primary" onclick="startGame('math')">Math</button>
        <button class="btn primary" onclick="startGame('memory')">Memory</button>
      </div>
    </div>
    <div class="card">
      <h2>Stats</h2>
      <p id="rank">Rank: —</p>
      <p id="animal">Animal: —</p>
      <div class="progress"><div class="bar" id="xp"></div></div>
      <canvas id="graph"></canvas>
    </div>
    <div class="card">
      <h2>Leaderboards</h2>
      <ol id="lb"></ol>
      <button class="btn" onclick="share()">Share Score</button>
    </div>
    <div class="card">
      <h2>Customization</h2>
      <div class="row">
        <button class="btn" onclick="setTheme('dark')">Dark</button>
        <button class="btn" onclick="setTheme('light')">Light</button>
        <button class="btn" onclick="skin()">Unlock Skin</button>
      </div>
    </div>
  </div>

  <div class="card center" id="arena">
    <h2 id="title">Ready?</h2>
    <div class="big" id="prompt">Press Start</div>
    <div class="progress"><div class="bar" id="timebar"></div></div>
    <div class="row center">
      <button class="btn primary" onclick="go()">Start</button>
      <button class="btn" onclick="stop()">Stop</button>
    </div>
  </div>
</div>

<canvas class="confetti" id="fx"></canvas>

<script>
// ---------- persistence ----------
const S = JSON.parse(localStorage.getItem('rz')||'{}');
const save=()=>localStorage.setItem('rz',JSON.stringify(S));
S.name ||= 'Player'; S.xp ||= 0; S.best ||= 9999; S.hist ||= []; S.lb ||= [];

// ---------- ui ----------
const $=id=>document.getElementById(id);
function setTheme(t){ document.body.dataset.theme=t; }
function toggleTheme(){ setTheme(document.body.dataset.theme==='dark'?'light':'dark'); }

// ---------- game core ----------
let mode='tap', t0=0, running=false, timer;
const animals=[['Slow Sloth',600],['Curious Cat',450],['Swift Fox',350],['Cheetah',250]];
function rankFrom(ms){ return animals.find(a=>ms>a[1])?.[0]||'Falcon'; }
function go(){ if(running) return; running=true; $('prompt').textContent='Wait…'; $('timebar').style.width='0%';
  const wait=500+Math.random()*1500; setTimeout(()=>{ t0=performance.now(); $('prompt').textContent='GO!'; },wait);
  let p=0; timer=setInterval(()=>{ p=Math.min(100,p+5); $('timebar').style.width=p+'%'; if(p>=100) stop(); },50);
}
function stop(){ if(!running) return; running=false; clearInterval(timer);
  const t=performance.now()-t0; if(t>0&&t<2000){ finish(t); } $('prompt').textContent='Stopped'; }
function finish(ms){ S.best=Math.min(S.best,ms); S.hist.push(ms); S.xp+=Math.max(5,Math.round(200-ms/5)); S.lb.push({n:S.name,s:ms}); S.lb=S.lb.sort((a,b)=>a.s-b.s).slice(0,10); save(); draw(); confetti(); }
function startGame(m){ mode=m; $('title').textContent='Mode: '+m; }

// ---------- extras ----------
function dailyBonus(){ const k='d'+new Date().toDateString(); if(S[k]) return alert('Already claimed'); S[k]=1; S.xp+=100; save(); draw(); }
function skin(){ S.xp+=20; save(); draw(); }
function share(){ navigator.share?.({title:'Reaction Zoo',text:`My best: ${S.best}ms`}); }

// ---------- render ----------
function draw(){ $('rank').textContent='Rank: '+(S.best<9999?rankFrom(S.best):'—'); $('animal').textContent='Animal: '+(S.best<9999?rankFrom(S.best):'—'); $('xp').style.width=Math.min(100,S.xp%100)+'%';
  $('lb').innerHTML=S.lb.map(x=>`<li>${x.n} — ${x.s.toFixed(0)}ms</li>`).join(''); drawGraph(); }
function drawGraph(){ const c=$('graph'),g=c.getContext('2d'); c.width=c.clientWidth; c.height=c.clientHeight; g.clearRect(0,0,c.width,c.height);
  g.strokeStyle='#7cf3c0'; g.beginPath(); S.hist.slice(-20).forEach((v,i)=>{ const x=i*(c.width/20); const y=c.height-(v/800)*c.height; i?g.lineTo(x,y):g.moveTo(x,y); }); g.stroke(); }

// ---------- confetti ----------
function confetti(){ const c=$('fx'),g=c.getContext('2d'); c.width=innerWidth; c.height=innerHeight; let p=[...Array(120)].map(()=>({x:Math.random()*c.width,y:-20,vy:2+Math.random()*3}));
  const id=setInterval(()=>{ g.clearRect(0,0,c.width,c.height); p.forEach(o=>{ o.y+=o.vy; g.fillRect(o.x,o.y,3,6); }); if(p.every(o=>o.y>c.height)) clearInterval(id); },16);
}

// init
draw();
</script>
</body>
</html>
