<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Backshot Roulette</title>
<style>
body{
  margin:0;
  background:#111;
  color:white;
  font-family:sans-serif;
  text-align:center;
}
.player{
  border:2px solid white;
  margin:10px;
  padding:10px;
}
button{
  margin:4px;
  padding:8px 12px;
  font-size:16px;
}
.flash-red{background:#800;}
.flash-green{background:#064;}
#log{min-height:60px;margin-top:10px;}
</style>
</head>
<body>

<h2 id="info"></h2>

<div class="player">
<h3>A</h3>
❤️ <span id="hpA"></span>
<div id="itemsA"></div>
</div>

<div class="player">
<h3>B</h3>
❤️ <span id="hpB"></span>
<div id="itemsB"></div>
</div>

<button onclick="shoot('self')">自分に撃つ</button>
<button onclick="shoot('other')">相手に撃つ</button>

<div id="log"></div>

<script>
/* ===== 基本 ===== */
const maxHP=[0,3,5,6];
const itemsPerRound=[0,3,4,5];
let round=1;
let turn=Math.random()<0.5?"A":"B";

let players={
  A:{hp:3,items:[],cuffed:false},
  B:{hp:3,items:[],cuffed:false}
};

let bullets=[];
let sawed=false;

/* ===== アイテム ===== */
const ITEMS=[
"虫眼鏡","ビール","タバコ","手錠",
"ノコギリ","使い捨て携帯","インバータ",
"アドレナリン","期限切れ薬"
];

function randomItem(){
  if(Math.random()<0.08) return "タバコ";
  return ITEMS.filter(i!=="タバコ")[Math.floor(Math.random()*8)];
}

/* ===== ラウンド開始 ===== */
function startRound(){
  ["A","B"].forEach(p=>{
    players[p].hp=maxHP[round];
    players[p].items=[];
    players[p].cuffed=false;
  });
  reload();
  updateUI();
}

/* ===== 装填 ===== */
function reload(){
  bullets=[];
  let total=Math.floor(Math.random()*6)+3;
  let real=Math.floor(total/2);
  let blank=total-real;
  while(Math.abs(real-blank)>2){
    real--; blank++;
  }
  bullets=[...Array(real).fill("real"),...Array(blank).fill("blank")]
    .sort(()=>Math.random()-0.5);

  ["A","B"].forEach(p=>{
    for(let i=0;i<itemsPerRound[round];i++){
      if(players[p].items.length<8)
        players[p].items.push(randomItem());
    }
  });

  log(`再装填：実弾${real} 空弾${blank}`);
}

/* ===== 発砲 ===== */
function shoot(target){
  if(bullets.length===0) return;
  let shooter=turn;
  let victim=(target==="self")?shooter:(shooter==="A"?"B":"A");

  log(`${shooter} → ${victim}`);
  setTimeout(()=>{
    let b=bullets.shift();
    if(b==="real"){
      document.body.className="flash-red";
      players[victim].hp-=sawed?2:1;
    }else{
      document.body.className="flash-green";
    }
    sawed=false;
    setTimeout(()=>{
      document.body.className="";
      afterShot(b,shooter,victim);
    },500);
  },4000);
}

function afterShot(b,shooter,victim){
  if(players[victim].hp<=0){
    round++;
    if(round>3){log("ゲーム終了");return;}
    startRound();
    return;
  }
  if(bullets.length===0) reload();
  if(!(b==="blank" && shooter===victim))
    turn=shooter==="A"?"B":"A";
  updateUI();
}

/* ===== アイテム使用 ===== */
function useItem(p,i){
  if(turn!==p) return;
  let item=players[p].items[i];
  if(!confirm(item+" を使用しますか？"))return;

  let other=p==="A"?"B":"A";
  log(p+" は "+item+" を使用");

  switch(item){
    case "虫眼鏡":
      alert("現在の弾："+bullets[0]);
      break;

    case "ビール":
      let out=bullets.shift();
      alert(out==="real"?"実弾が出ました":"空弾が出ました");
      if(bullets.length===0) reload();
      break;

    case "タバコ":
      players[p].hp=Math.min(players[p].hp+1,maxHP[round]);
      break;

    case "手錠":
      if(players[other].cuffed) break;
      players[other].cuffed=true;
      break;

    case "ノコギリ":
      sawed=true;
      break;

    case "使い捨て携帯":
      if(bullets.length>1){
        let n=Math.floor(Math.random()*(bullets.length-1))+1;
        alert((n+1)+"発目は "+bullets[n]);
      }
      break;

    case "インバータ":
      bullets=bullets.map(b=>b==="real"?"blank":"real");
      break;

    case "アドレナリン":
      let list=players[other].items.filter(i=>i!=="アドレナリン");
      if(list.length){
        let stolen=list[0];
        alert("奪った："+stolen);
      }
      break;

    case "期限切れ薬":
      if(Math.random()<0.5)
        players[p].hp=Math.min(players[p].hp+2,maxHP[round]);
      else
        players[p].hp--;
      break;
  }

  players[p].items.splice(i,1);
  updateUI();
}

/* ===== UI ===== */
function updateUI(){
  info.textContent=`R${round} 手番:${turn}`;
  hpA.textContent=players.A.hp;
  hpB.textContent=players.B.hp;

  itemsA.innerHTML="";
  players.A.items.forEach((it,i)=>{
    let b=document.createElement("button");
    b.textContent=it;
    b.onclick=()=>useItem("A",i);
    itemsA.appendChild(b);
  });

  itemsB.innerHTML="";
  players.B.items.forEach((it,i)=>{
    let b=document.createElement("button");
    b.textContent=it;
    b.onclick=()=>useItem("B",i);
    itemsB.appendChild(b);
  });
}

function log(t){document.getElementById("log").textContent=t;}

startRound();
</script>
</body>
</html>
