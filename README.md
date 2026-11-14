
<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Minecraft Adventure</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#17251c;color:white;font-family:Arial;text-align:center}
header{background:#263d2c;padding:14px;font-size:23px;font-weight:bold}
.panel{background:#304b36;margin:12px auto;padding:14px;border-radius:14px;max-width:740px}
button{background:#579b45;color:white;border:0;border-bottom:4px solid #32642c;padding:12px 15px;margin:5px;border-radius:9px;font-size:15px;font-weight:bold}
button:active{transform:translateY(3px)}
.hidden{display:none!important}
.stats{display:flex;justify-content:center;gap:10px;flex-wrap:wrap}
.stat{background:#182d20;padding:9px;border-radius:9px}
#game{width:min(96vw,700px);height:auto;background:#91cce5;border:5px solid #263d2c;border-radius:8px;touch-action:none;image-rendering:pixelated}
.controls{display:flex;justify-content:center;gap:5px;flex-wrap:wrap}
#portal{font-size:45px;animation:pulse 1s infinite alternate}
@keyframes pulse{to{filter:brightness(1.8);transform:scale(1.08)}}
.small{font-size:13px;color:#d3e4d4}
</style>
</head>
<body>

<header>⛏️ MINECRAFT ADVENTURE</header>

<div id="home">
 <div class="panel">
  <h2>🌍 Обычный мир</h2>
  <div class="stats">
   <div class="stat">🪙 <span id="coins">0</span></div>
   <div class="stat">💚 <span id="emeralds">0</span></div>
   <div class="stat">⛏️ <span id="pickaxe">Обычная</span></div>
  </div>
  <p>Добывай блоки и сражайся с монстрами!</p>
  <button onclick="mine()">⛏️ ДОБЫТЬ БЛОК</button>
  <p id="message">Добудь первый блок, чтобы встретить зомби!</p>
  <button id="zombieStart" class="hidden" onclick="startZombie()">🧟 СРАЗИТЬСЯ С ЗОМБИ</button>
 </div>

 <div class="panel">
  <h2>🎒 Инвентарь</h2>
  <div id="inventory">⚔️ Обычный меч<br>⛏️ Обычная кирка</div>
 </div>

 <div class="panel">
  <h2>🛒 Магазин</h2>
  <button onclick="upgradeSword()">⚔️ Улучшить меч — 50 монет</button>
 </div>

 <div class="panel">
  <h2>🌀 Портал в босс-файт</h2>
  <div id="locked">🔒 Найди изумруд с зомби!</div>
  <div id="portalSection" class="hidden">
   <div id="portal">🌀</div>
   <button onclick="startHerobrine()">ВОЙТИ К ХЕРОБРИНУ</button>
  </div>
 </div>
</div>

<div id="battle" class="hidden">
 <div class="panel">
  <h2 id="battleTitle">БОЙ</h2>
  <div class="stats">
   <div class="stat">❤️ Ты: <span id="playerHP">100</span></div>
   <div class="stat">👹 Враг: <span id="enemyHP">100</span></div>
  </div>

  <canvas id="game" width="700" height="430"></canvas>

  <p class="small">A/D — движение, W — прыжок, J — удар, K — блок, L — уклонение</p>

  <div class="controls">
   <button id="left">⬅️</button>
   <button id="right">➡️</button>
   <button onclick="jump()">⬆️ Прыжок</button>
   <button onclick="attack()">⚔️ Удар</button>
   <button onclick="block()">🛡️ Блок</button>
   <button onclick="dodge()">💨 Уклонение</button>
  </div>

  <h3 id="battleMessage">Победи врага!</h3>
  <button onclick="goHome()">🏠 Домой</button>
 </div>
</div>

<script>
let coins=0,emeralds=0,swordLevel=1,pickaxeOwned=false;
let portalUnlocked=false,zombieReady=false;
let enemyType="zombie";
let playerHP=100,enemyHP=100;
let running=false,gameOver=false;
let blocking=false,dodging=false;
let keys={};

let player={
 x:100,y:280,vy:0,onGround:true,
 attackAnim:0,walk:0,hurt:0,attackCooldown:0
};

let enemy={
 x:470,y:280,attackTimer:0,
 attackAnim:0,walk:0,hurt:0
};

const canvas=document.getElementById("game");
const ctx=canvas.getContext("2d");

function ui(){
 document.getElementById("coins").textContent=coins;
 document.getElementById("emeralds").textContent=emeralds;
 document.getElementById("pickaxe").textContent=pickaxeOwned?"ЛЕГЕНДАРНАЯ":"Обычная";

 document.getElementById("inventory").innerHTML=
 "⚔️ Меч: "+(swordLevel>1?"Улучшенный уровень "+swordLevel:"Обычный")+
 "<br>⛏️ Кирка: "+(pickaxeOwned?"🌟 Легендарная":"Обычная")+
 "<br>💚 Изумруды: "+emeralds;

 document.getElementById("locked").classList.toggle("hidden",portalUnlocked);
 document.getElementById("portalSection").classList.toggle("hidden",!portalUnlocked);
}

function mine(){
 if(zombieReady){
  document.getElementById("message").textContent="Сначала победи зомби!";
  return;
 }

 let reward=pickaxeOwned?10:5;
 coins+=reward;
 zombieReady=true;

 document.getElementById("zombieStart").classList.remove("hidden");
 document.getElementById("message").textContent=
 "🧟 ЗОМБИ ПОЯВИЛСЯ! +"+reward+" монет. Иди сражаться!";

 ui();
}

function upgradeSword(){
 if(coins<50){
  document.getElementById("message").textContent="Нужно 50 монет!";
  return;
 }
 coins-=50;
 swordLevel++;
 document.getElementById("message").textContent="Меч улучшен!";
 ui();
}

function setupBattle(type){
 enemyType=type;
 document.getElementById("home").classList.add("hidden");
 document.getElementById("battle").classList.remove("hidden");

 playerHP=100;
 enemyHP=type==="herobrine"?300:100;

 player={
  x:90,y:280,vy:0,onGround:true,
  attackAnim:0,walk:0,hurt:0,attackCooldown:0
 };

 enemy={
  x:490,y:280,attackTimer:0,
  attackAnim:0,walk:0,hurt:0
 };

 gameOver=false;
 blocking=false;
 dodging=false;
 keys={};

 document.getElementById("battleTitle").textContent=
 type==="zombie"?"🧟 БОЙ С ЗОМБИ":"👁️ БОЙ С ХЕРОБРИНОМ";

 document.getElementById("battleMessage").textContent="Победи врага!";
 updateHP();

 running=true;
 requestAnimationFrame(loop);
}

function startZombie(){
 if(zombieReady)setupBattle("zombie");
}

function startHerobrine(){
 if(portalUnlocked)setupBattle("herobrine");
}

function goHome(){
 running=false;
 keys={};
 document.getElementById("battle").classList.add("hidden");
 document.getElementById("home").classList.remove("hidden");
 ui();
}

function updateHP(){
 document.getElementById("playerHP").textContent=Math.max(0,playerHP);
 document.getElementById("enemyHP").textContent=Math.max(0,enemyHP);
}

function attack(){
 if(gameOver||player.attackCooldown>0)return;

 player.attackAnim=15;
 player.attackCooldown=25;

 let distance=Math.abs(player.x-enemy.x);

 if(distance<155){
  let damage=20+(swordLevel-1)*10;
  enemyHP-=damage;
  enemy.hurt=12;
  enemy.x+=player.x<enemy.x?22:-22;

  document.getElementById("battleMessage").textContent=
   "💥 Удар! Ты нанёс "+damage+" урона!";

  if(enemyHP<=0)win();
  updateHP();
 }else{
  document.getElementById("battleMessage").textContent="Подойди ближе к врагу!";
 }
}

function jump(){
 if(!gameOver&&player.onGround){
  player.vy=-15;
  player.onGround=false;
 }
}

function block(){
 if(gameOver)return;
 blocking=true;
 document.getElementById("battleMessage").textContent="🛡️ Ты защищаешься!";
 setTimeout(()=>blocking=false,900);
}

function dodge(){
 if(gameOver)return;
 dodging=true;
 player.x+=player.x<enemy.x?-95:95;
 player.x=Math.max(5,Math.min(600,player.x));
 document.getElementById("battleMessage").textContent="💨 Уклонение!";
 setTimeout(()=>dodging=false,550);
}

function win(){
 gameOver=true;
 running=false;

 if(enemyType==="zombie"){
  coins+=25;
  let msg="🧟 Зомби побеждён! +25 монет.";

  if(Math.random()<0.16){
   emeralds++;
   portalUnlocked=true;
   msg+=" 💚 ВЫПАЛ ИЗУМРУД! Портал открыт!";
  }else{
   msg+=" Изумруд не выпал. Попробуй ещё!";
  }

  zombieReady=false;
  document.getElementById("zombieStart").classList.add("hidden");
  document.getElementById("battleMessage").textContent=msg;

 }else{
  coins+=300;
  emeralds+=5;
  let msg="🏆 Херобрин побеждён! +300 монет и +5 изумрудов!";

  if(Math.random()<0.10){
   pickaxeOwned=true;
   msg+=" 🌟 ЛЕГЕНДАРНАЯ КИРКА!";
  }else{
   msg+=" Кирка не выпала. Попробуй снова!";
  }

  document.getElementById("battleMessage").textContent=msg;
 }

 ui();
}

function lose(){
 gameOver=true;
 running=false;
 document.getElementById("battleMessage").textContent=
 "💀 Ты проиграл! Вернись домой и попробуй снова.";
}

/* Большие пиксельные персонажи с анимациями */
function drawPerson(x,y,type,walk,attackFrame,hurt,flip){
 ctx.save();

 // Увеличиваем модели
 ctx.translate(x,y);
 ctx.scale(1.45,1.45);

 // Направление персонажа
 if(flip) {
  ctx.translate(60,0);
  ctx.scale(-1,1);
 }

 // Моргание при получении урона
 if(hurt>0){
  ctx.globalAlpha=0.65;
 }

 let zombie=type==="zombie";
 let herobrine=type==="herobrine";

 let head=zombie?"#438d45":herobrine?"#171717":"#e5b78c";
 let body=zombie?"#286b39":herobrine?"#111111":"#2869b1";
 let legs=zombie?"#244b30":herobrine?"#111111":"#244d91";

 let swing=Math.sin(walk*0.16)*5;
 let armSwing=attackFrame>0?Math.sin((15-attackFrame)/15*Math.PI)*35:swing;

 // Ноги
 ctx.fillStyle=legs;
 ctx.fillRect(10,91+swing,17,30);
 ctx.fillRect(35,91-swing,17,30);

 // Обувь
 ctx.fillStyle=herobrine?"#050505":zombie?"#193e27":"#182b53";
 ctx.fillRect(7,115+swing,22,9);
 ctx.fillRect(33,115-swing,22,9);

 // Дальняя рука
 ctx.fillStyle=body;
 ctx.fillRect(1,47-swing,12,43);

 // Тело
 ctx.fillStyle=body;
 ctx.fillRect(10,43,42,52);

 // Детали одежды
 if(zombie){
  ctx.fillStyle="#3c8644";
  ctx.fillRect(17,52,9,17);
  ctx.fillRect(36,52,9,17);
 }else if(!herobrine){
  ctx.fillStyle="#1e4e91";
  ctx.fillRect(18,50,8,23);
 }

 // Ближняя рука с анимацией замаха
 ctx.save();
 ctx.translate(51,49);
 ctx.rotate(attackFrame>0?-armSwing*0.035:swing*0.025);
 ctx.fillStyle=body;
 ctx.fillRect(-1,0,14,43);
 ctx.fillStyle=head;
 ctx.fillRect(0,36,13,12);

 // Меч во время атаки
 if(!zombie&&!herobrine&&attackFrame>0){
  ctx.fillStyle="#d9e2ed";
  ctx.fillRect(5,43,8,39);
  ctx.fillStyle="#8c6035";
  ctx.fillRect(-1,45,20,7);
  ctx.fillStyle="#fff";
  ctx.fillRect(7,43,4,27);
 }
 ctx.restore();

 // Голова
 ctx.fillStyle=herobrine?"#171717":"#333";
 ctx.fillRect(10,0,42,43);
 ctx.fillStyle=head;
 ctx.fillRect(10,6,42,34);

 // Лицо зомби
 if(zombie){
  ctx.fillStyle="#fff";
  ctx.fillRect(16,17,9,8);
  ctx.fillRect(37,17,9,8);
  ctx.fillStyle="#111";
  ctx.fillRect(19,19,4,5);
  ctx.fillRect(40,19,4,5);
  ctx.fillStyle="#16351c";
  ctx.fillRect(23,31,18,5);
  ctx.fillRect(28,28,8,4);
 }

 // Лицо Херобрина
 else if(herobrine){
  ctx.fillStyle="#fff";
  ctx.fillRect(16,17,9,6);
  ctx.fillRect(37,17,9,6);
 }

 // Лицо игрока
 else{
  ctx.fillStyle="#222";
  ctx.fillRect(19,18,6,6);
  ctx.fillRect(38,18,6,6);
  ctx.fillStyle="#9a5b45";
  ctx.fillRect(27,31,10,3);
 }

 ctx.restore();
}

function draw(){
 ctx.clearRect(0,0,700,430);

 // Фон
 let gradient=ctx.createLinearGradient(0,0,0,430);

 if(enemyType==="zombie"){
  gradient.addColorStop(0,"#8ed1ed");
  gradient.addColorStop(1,"#b4e0ed");
 }else{
  gradient.addColorStop(0,"#58477e");
  gradient.addColorStop(1,"#29213f");
 }

 ctx.fillStyle=gradient;
 ctx.fillRect(0,0,700,430);

 // Облака или тёмные блоки
 if(enemyType==="zombie"){
  ctx.fillStyle="#d5eff7";
  ctx.fillRect(60,65,90,25);
  ctx.fillRect(105,45,65,25);
  ctx.fillRect(460,95,100,25);
 }else{
  ctx.fillStyle="#7962a1";
  for(let i=0;i<7;i++){
   ctx.fillRect(i*110,170,65,35);
  }
 }

 // Земля
 ctx.fillStyle=enemyType==="zombie"?"#528c42":"#423b54";
 ctx.fillRect(0,385,700,45);
 ctx.fillStyle="#80633e";
 ctx.fillRect(0,385,700,9);

 // Тени
 ctx.fillStyle="rgba(0,0,0,.25)";
 ctx.fillRect(player.x+5,410,75,8);
 ctx.fillRect(enemy.x+5,410,75,8);

 // Защита
 if(blocking){
  ctx.strokeStyle="#50dfff";
  ctx.lineWidth=6;
  ctx.beginPath();
  ctx.arc(player.x+43,player.y+75,67,0,Math.PI*2);
  ctx.stroke();
 }

 // Вспышка удара
 if(player.attackAnim>0){
  ctx.fillStyle="#fff5a6";
  ctx.fillRect(player.x+65,player.y+55,18,18);
 }

 drawPerson(player.x,player.y,"player",player.walk,player.attackAnim,player.hurt,player.x>enemy.x);
 drawPerson(enemy.x,enemy.y,enemyType,enemy.walk,enemy.attackAnim,enemy.hurt,enemy.x<player.x);

 ctx.fillStyle="white";
 ctx.font="bold 18px Arial";
 ctx.fillText("ТЫ",player.x+15,player.y-12);
 ctx.fillText(enemyType==="zombie"?"ЗОМБИ":"ХЕРОБРИН",enemy.x+2,enemy.y-12);
}

function loop(){
 if(!running)return;

 if(!gameOver){
  let moving=false;

  if(keys.left||keys.a){
   player.x-=4;
   moving=true;
  }

  if(keys.right||keys.d){
   player.x+=4;
   moving=true;
  }

  if(moving)player.walk+=1;
  player.x=Math.max(5,Math.min(610,player.x));

  // Гравитация и прыжок
  player.y+=player.vy;
  player.vy+=0.7;

  if(player.y>=280){
   player.y=280;
   player.vy=0;
   player.onGround=true;
  }

  // Таймеры анимаций
  if(player.attackAnim>0)player.attackAnim--;
  if(player.attackCooldown>0)player.attackCooldown--;
  if(player.hurt>0)player.hurt--;
  if(enemy.hurt>0)enemy.hurt--;

  // Враг приближается
  let distance=Math.abs(player.x-enemy.x);

  if(distance>100){
   enemy.x+=player.x>enemy.x?1.5:-1.5;
   enemy.walk+=1;
  }

  // Атака врага
  enemy.attackTimer++;

  let interval=enemyType==="zombie"?60:72;

  if(enemy.attackTimer>interval){
   enemy.attackTimer=0;

   if(distance<155&&!dodging){
    enemy.attackAnim=14;

    let damage=enemyType==="zombie"?8:12;
    playerHP-=blocking?Math.ceil(damage*0.25):damage;

    player.hurt=10;

    document.getElementById("battleMessage").textContent=
     blocking?"🛡️ Ты заблокировал удар!":"💥 Враг атаковал тебя!";

    updateHP();

    if(playerHP<=0)lose();
   }
  }

  if(enemy.attackAnim>0)enemy.attackAnim--;
 }

 draw();

 if(running)requestAnimationFrame(loop);
}

function setKey(id,key){
 let b=document.getElementById(id);

 b.addEventListener("pointerdown",e=>{
  e.preventDefault();
  keys[key]=true;
 });

 ["pointerup","pointerleave","pointercancel"].forEach(ev=>{
  b.addEventListener(ev,()=>keys[key]=false);
 });
}

setKey("left","left");
setKey("right","right");

document.addEventListener("keydown",e=>{
 let k=e.key.toLowerCase();

 if([" ","arrowleft","arrowright","arrowup"].includes(k))e.preventDefault();

 keys[k]=true;

 if(k==="w"||k===" ")jump();
 if(k==="j")attack();
 if(k==="k")block();
 if(k==="l")dodge();
});

document.addEventListener("keyup",e=>{
 keys[e.key.toLowerCase()]=false;
});

ui();
</script>
</body>
</html>
