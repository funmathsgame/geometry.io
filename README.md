<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Wait For It</title>
  <style>
    *{ box-sizing:border-box }
    body{
      margin:0; font-family:system-ui,-apple-system,Segoe UI,Roboto;
      background:#0b0b0b; color:white;
      display:flex; align-items:center; justify-content:center; height:100vh;
    }
    .card{
      width:min(520px,92vw);
      background:#111;
      border-radius:24px;
      padding:32px;
      text-align:center;
      box-shadow:0 30px 80px #000;
      transition:transform .2s ease;
    }
    h1{ margin:0 0 10px; font-size:2.2rem }
    p{ opacity:.75 }
    .screen{ display:none }
    .screen.active{ display:block }
    .circle{
      width:200px; height:200px;
      border-radius:50%;
      margin:30px auto;
      background:#333;
      display:flex; align-items:center; justify-content:center;
      font-size:1.5rem; font-weight:700;
      user-select:none;
      cursor:pointer;
      transition:background .15s ease, transform .1s ease;
    }
    .circle:active{ transform:scale(.97) }
    .btn{
      border:none; border-radius:999px;
      padding:12px 22px;
      font-size:1rem;
      cursor:pointer;
      background:white; color:black;
    }
    .tiny{ font-size:.9rem; opacity:.6 }
  </style>
</head>
<body>

<div class="card">
  <div class="screen active" id="start">
    <h1>Wait For It</h1>
    <p>Click the circle only when it turns green.</p>
    <button class="btn" onclick="begin()">Start</button>
  </div>

  <div class="screen" id="game">
    <div class="circle" id="circle">Wait…</div>
    <p class="tiny">Don’t click early.</p>
  </div>

  <div class="screen" id="result">
    <h1 id="time">—</h1>
    <p id="msg"></p>
    <button class="btn" onclick="reset()">Try again</button>
  </div>
</div>

<script>
let startTime, timeout, ready=false;
const circle=document.getElementById('circle');

function show(id){
  document.querySelectorAll('.screen').forEach(s=>s.classList.remove('active'));
  document.getElementById(id).classList.add('active');
}

function begin(){
  show('game');
  ready=false;
  circle.style.background='#333';
  circle.textContent='Wait…';

  const delay=1000+Math.random()*3000;
  timeout=setTimeout(()=>{
    ready=true;
    startTime=performance.now();
    circle.style.background='#2ecc71';
    circle.textContent='CLICK';
  },delay);
}

circle.onclick=()=>{
  if(!ready){
    clearTimeout(timeout);
    end('Too early 😬','red');
    return;
  }
  const t=Math.round(performance.now()-startTime);
  let msg='';
  if(t<200) msg='Cheetah reflexes 🐆';
  else if(t<300) msg='Pretty fast 🦊';
  else if(t<450) msg='Not bad 🐕';
  else msg='Are you sleepy? 🦥';

  document.getElementById('time').textContent=t+' ms';
  document.getElementById('msg').textContent=msg;
  show('result');
}

function end(text,color){
  circle.style.background=color;
  document.getElementById('time').textContent=text;
  document.getElementById('msg').textContent='Try again.';
  show('result');
}

function reset(){ show('start'); }
</script>

</body>
</html>
