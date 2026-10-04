<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>The Little Dumpling's Ultimate Name Bracket</title>
<style>
*{box-sizing:border-box}
body{margin:0;font-family:Arial,Helvetica,sans-serif;background:#f8f1e7;color:#2b2521}
header{text-align:center;padding:28px 16px 18px}
h1{margin:0;font-size:clamp(26px,4vw,44px);letter-spacing:.03em}
.subtitle{margin:8px 0 0;font-size:15px}
.instructions{max-width:900px;margin:0 auto 18px;text-align:center;font-size:14px}
.bracket-wrap{overflow-x:auto;padding:10px 18px 28px}
.bracket{min-width:1120px;max-width:1450px;margin:auto;display:grid;grid-template-columns:repeat(4,1fr);gap:26px}
.round{display:flex;flex-direction:column}
.round h2{text-align:center;font-size:14px;letter-spacing:.12em;text-transform:uppercase;margin:0 0 16px}
.games{flex:1;display:flex;flex-direction:column;justify-content:space-around;gap:18px}
.game{display:flex;flex-direction:column;gap:4px}
.slot{position:relative;display:flex;align-items:center;justify-content:space-between;background:white;border:2px solid #d8cdc1;border-radius:8px;padding:10px 12px;min-height:42px;cursor:pointer;transition:.15s}
.slot:hover{border-color:#8d6b58;transform:translateY(-1px)}
.slot.selected{background:#2b2521;color:white;border-color:#2b2521}
.slot.disabled{opacity:.45;cursor:not-allowed}
.seed{font-size:11px;opacity:.65;margin-right:8px}
.name{font-weight:700}
.pick{font-size:11px;opacity:.65}
.final .slot{min-height:48px}
.champion{border:3px solid #b48a45!important;background:#fffaf0!important;color:#2b2521!important}
.champion .pick{color:#b48a45}
.form{max-width:520px;margin:0 auto 40px;background:white;border:1px solid #ded3c7;border-radius:12px;padding:22px}
.form h2{margin-top:0}
label{display:block;font-size:13px;font-weight:700;margin:12px 0 6px}
input{width:100%;padding:12px;border:1px solid #cfc2b5;border-radius:7px;font-size:16px}
button{width:100%;margin-top:16px;padding:13px;border:0;border-radius:8px;background:#2b2521;color:white;font-size:16px;font-weight:700;cursor:pointer}
button:disabled{opacity:.5;cursor:not-allowed}
.status{text-align:center;margin-top:12px;font-size:13px}
.notice{font-size:12px;line-height:1.5;margin-top:10px;color:#665c55}
</style>
</head>
<body>
<header>
  <h1>THE LITTLE DUMPLING'S<br>ULTIMATE NAME BRACKET</h1>
  <p class="subtitle">16 names. 1 champion. Make your picks.</p>
</header>

<div class="instructions">
  Click one name in each matchup. Your selection advances automatically through the bracket.
  Complete all four rounds, then submit your bracket.
</div>

<div class="bracket-wrap">
<div class="bracket">
  <section class="round"><h2>Round 1</h2><div class="games" id="r1"></div></section>
  <section class="round"><h2>Round 2</h2><div class="games" id="r2"></div></section>
  <section class="round"><h2>Semifinals</h2><div class="games" id="r3"></div></section>
  <section class="round final"><h2>Final</h2><div class="games" id="r4"></div></section>
</div>
</div>

<div class="form">
  <h2>Submit your bracket</h2>
  <label for="player">Your name</label>
  <input id="player" placeholder="Enter your name">
  <button id="submit" disabled>Submit my bracket</button>
  <div class="status" id="status"></div>
  <div class="notice">One submission per person. Your name and completed picks are recorded when you submit.</div>
</div>

<script>
const rounds = [
  [
    [{seed:1,name:"Avery"},{seed:16,name:"Stump"}],
    [{seed:8,name:"Augusta"},{seed:9,name:"Bailey"}],
    [{seed:5,name:"Hieu"},{seed:12,name:"Callahan"}],
    [{seed:4,name:"Liam"},{seed:13,name:"Arlo"}],
    [{seed:6,name:"Logan"},{seed:11,name:"Caleb"}],
    [{seed:3,name:"Rhys"},{seed:14,name:"Jared Jr."}],
    [{seed:7,name:"Aiden"},{seed:10,name:"Lucas"}],
    [{seed:2,name:"Val"},{seed:15,name:"Hannala"}]
  ],
  Array(4).fill(null).map(()=>[null,null]),
  Array(2).fill(null).map(()=>[null,null]),
  Array(1).fill(null).map(()=>[null,null])
];

const picks = [[],[],[],[]];

function renderRound(r){
  const el=document.getElementById("r"+(r+1));
  el.innerHTML="";
  rounds[r].forEach((game,gi)=>{
    const div=document.createElement("div"); div.className="game";
    game.forEach((slot,si)=>{
      const b=document.createElement("div"); b.className="slot";
      if(!slot){ b.classList.add("disabled"); b.innerHTML='<span class="name">Waiting for your pick…</span>'; }
      else{
        const selected=picks[r][gi]===si;
        if(selected)b.classList.add("selected");
        if(r===3 && selected)b.classList.add("champion");
        b.innerHTML=`<span><span class="seed">${slot.seed??""}</span><span class="name">${slot.name}</span></span><span class="pick">${selected?"✓":""}</span>`;
        b.onclick=()=>choose(r,gi,si);
      }
      div.appendChild(b);
    });
    el.appendChild(div);
  });
}

function choose(r,gi,si){
  const slot=rounds[r][gi][si]; if(!slot)return;
  picks[r][gi]=si;
  // Advance selected name into next round.
  if(r<3){
    const nextGame=Math.floor(gi/2), nextSlot=gi%2;
    rounds[r+1][nextGame][nextSlot]={...slot};
    // Clear dependent later picks if necessary.
    for(let rr=r+1;rr<4;rr++){
      picks[rr]=picks[rr].map((v,i)=> (rr===r+1 && i===nextGame)?undefined:undefined);
    }
    // Rebuild all subsequent rounds from current picks.
    rebuildFrom(r+1);
  }
  renderAll();
}

function rebuildFrom(start){
  for(let r=start;r<4;r++){
    if(r>start){
      rounds[r].forEach(g=>g.forEach((_,si)=>rounds[r][si]=rounds[r][si]));
    }
    // If this round has enough source winners, populate slots from prior round.
    if(r>0){
      rounds[r].forEach((g,gi)=>{
        const a=picks[r-1][gi*2];
        const b=picks[r-1][gi*2+1];
        rounds[r][gi][0]=a!==undefined ? rounds[r-1][gi*2][a] : null;
        rounds[r][gi][1]=b!==undefined ? rounds[r-1][gi*2+1][b] : null;
      });
    }
    // Invalidate this and later picks if the selected slot no longer exists.
    if(r>=start) picks[r]=[];
  }
}
function renderAll(){for(let r=0;r<4;r++)renderRound(r); updateSubmit();}
function updateSubmit(){
  const complete=picks.every((arr,r)=>arr.length===rounds[r].length && arr.every(v=>v!==undefined));
  document.getElementById("submit").disabled=!complete;
}
document.getElementById("submit").onclick=()=>{
  const name=document.getElementById("player").value.trim();
  if(!name){alert("Please enter your name.");return;}
  const bracket=picks.map((arr,r)=>arr.map((si,gi)=>rounds[r][gi][si].name));
  const payload={name,submittedAt:new Date().toISOString(),round1:bracket[0],round2:bracket[1],semifinals:bracket[2],final:bracket[3]};
  // Replace this with your Google Apps Script endpoint when ready.
  console.log("SUBMISSION",payload);
  document.getElementById("status").textContent="Bracket ready! (The Google Sheet connection will be added next.)";
  document.getElementById("submit").disabled=true;
};
renderAll();
</script>
</body>
</html>
