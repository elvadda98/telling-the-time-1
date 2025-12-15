<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Speak the Time – Pronunciation Practice</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
  margin:0;
  font-family:system-ui, Arial, sans-serif;
  background:linear-gradient(135deg,#0b132b,#1c2541);
  color:#fff;
  min-height:100vh;
  display:flex;
  justify-content:center;
  align-items:center;
}

.container{
  width:min(900px,95%);
  background:#1c2541;
  border-radius:18px;
  padding:24px;
  box-shadow:0 25px 60px rgba(0,0,0,.4);
}

h1{
  margin-top:0;
  font-size:24px;
}

.time{
  font-size:64px;
  font-weight:800;
  text-align:center;
  margin:20px 0;
  letter-spacing:2px;
}

.buttons{
  display:flex;
  gap:12px;
  flex-wrap:wrap;
  justify-content:center;
}

button{
  padding:12px 18px;
  font-size:16px;
  font-weight:700;
  border-radius:14px;
  border:none;
  cursor:pointer;
  background:#64d404;
  color:#081b00;
}

button.secondary{
  background:#3a86ff;
  color:#fff;
}

button.danger{
  background:#ff595e;
  color:#fff;
}

.status{
  margin-top:16px;
  padding:14px;
  border-radius:12px;
  background:#0b132b;
  min-height:52px;
}

input{
  width:100%;
  margin-top:12px;
  padding:12px;
  border-radius:12px;
  border:none;
  font-size:16px;
}

.answers{
  margin-top:14px;
  padding:12px;
  background:#0b132b;
  border-radius:12px;
  font-family:monospace;
  font-size:13px;
  display:none;
  white-space:pre-wrap;
}
</style>
</head>

<body>
<div class="container">
  <h1>🕒 Speak the Time</h1>
  <p>Say the time using <b>o’clock / past / half past / to</b>.  
  If correct, a new time appears automatically.</p>

  <div id="time" class="time">--:--</div>

  <div class="buttons">
    <button id="start">🎤 Start</button>
    <button id="stop" class="secondary">⏹ Stop</button>
    <button id="new" class="secondary">🔄 New time</button>
    <button id="toggle" class="secondary">👀 Show answers</button>
  </div>

  <input id="manual" placeholder="No mic? Type your answer here (e.g. 20 to twelve)" />

  <div id="status" class="status">Ready.</div>
  <div id="answers" class="answers"></div>
</div>

<script>
// ==================== DATA ====================
const HOUR_WORD = {
  1:"one",2:"two",3:"three",4:"four",5:"five",6:"six",
  7:"seven",8:"eight",9:"nine",10:"ten",11:"eleven",12:"twelve"
};

const MIN_WORD = {
  5:"five",10:"ten",15:"quarter",20:"twenty",25:"twenty five"
};

const ALLOWED_MINUTES = [0,5,10,15,20,25,30,35,40,45,50,55];

// ==================== HELPERS ====================
const norm = s =>
  s.toLowerCase()
   .replace(/’|'/g,"")
   .replace(/-/g," ")
   .replace(/[^\w\s]/g,"")
   .replace(/\s+/g," ")
   .trim();

const uniq = arr => [...new Set(arr.map(norm))];

const pad = n => String(n).padStart(2,"0");

// ==================== TIME LOGIC ====================
function randomTime(){
  return {
    h:Math.floor(Math.random()*12)+1,
    m:ALLOWED_MINUTES[Math.floor(Math.random()*ALLOWED_MINUTES.length)]
  };
}

function acceptedAnswers(h,m){
  let out=[];
  const hw=HOUR_WORD[h], hn=String(h);

  const add=c=>{
    out.push(c,`its ${c}`,`it is ${c}`,`it's ${c}`);
  };

  if(m===0){
    add(`${hw} oclock`);
    add(`${hn} oclock`);
    add(hw); add(hn);
    return uniq(out);
  }

  if(m===30){
    add(`half past ${hw}`);
    add(`half past ${hn}`);
    add(`30 past ${hw}`);
    add(`30 past ${hn}`);
    add(`30 minutes past ${hw}`);
    add(`30 minutes past ${hn}`);
    return uniq(out);
  }

  if(m<30){
    const mw=MIN_WORD[m], mn=String(m);
    add(`${mw} past ${hw}`);
    add(`${mw} past ${hn}`);
    add(`${mn} past ${hw}`);
    add(`${mn} past ${hn}`);
    add(`${mn} minutes past ${hw}`);
    add(`${mn} minutes past ${hn}`);
    if(m===15){
      add(`quarter past ${hw}`);
      add(`quarter past ${hn}`);
      add(`15 past ${hw}`);
      add(`15 past ${hn}`);
    }
    return uniq(out);
  }

  const next=(h%12)+1;
  const rw=60-m, rws=MIN_WORD[rw], rn=String(rw);
  const nw=HOUR_WORD[next], nn=String(next);

  add(`${rws} to ${nw}`);
  add(`${rws} to ${nn}`);
  add(`${rn} to ${nw}`);
  add(`${rn} to ${nn}`);
  add(`${rn} minutes to ${nw}`);
  add(`${rn} minutes to ${nn}`);

  if(rw===15){
    add(`quarter to ${nw}`);
    add(`quarter to ${nn}`);
    add(`15 to ${nw}`);
    add(`15 to ${nn}`);
  }

  return uniq(out);
}

// ==================== GAME ====================
let current=randomTime();
let answers=[];
let score=0;

const timeEl=document.getElementById("time");
const statusEl=document.getElementById("status");
const answersEl=document.getElementById("answers");
const manual=document.getElementById("manual");

function render(){
  timeEl.textContent=`${current.h}:${pad(current.m)}`;
  answers=acceptedAnswers(current.h,current.m);
  answersEl.textContent=answers.join("\n");
}

function next(){
  current=randomTime();
  manual.value="";
  statusEl.textContent="New time. Speak now.";
  render();
}

function check(input){
  if(answers.includes(norm(input))){
    statusEl.textContent="✅ Correct!";
    setTimeout(next,600);
  }else{
    statusEl.textContent=`❌ I heard: "${input}"`;
  }
}

// ==================== SPEECH ====================
const SR=window.SpeechRecognition||window.webkitSpeechRecognition;
let rec=null;

if(SR){
  rec=new SR();
  rec.lang="en-GB";
  rec.onresult=e=>check(e.results[0][0].transcript);
}

document.getElementById("start").onclick=()=>rec?.start();
document.getElementById("stop").onclick=()=>rec?.stop();
document.getElementById("new").onclick=next;

manual.onkeydown=e=>{
  if(e.key==="Enter") check(manual.value);
};

document.getElementById("toggle").onclick=()=>{
  answersEl.style.display =
    answersEl.style.display==="none"?"block":"none";
};

render();
</script>
</body>
</html>
