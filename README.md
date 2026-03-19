<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1">
<title>Raspadinha Premiada</title>

<style>
body{
margin:0;
font-family:Arial;
background:linear-gradient(135deg,#070b2a,#1a0d2f);
color:white;
padding:20px;
}

h1{
color:#f4c430;
}

.cards{
display:grid;
grid-template-columns:repeat(2,1fr);
gap:15px;
margin-top:20px;
}

.card{
background:rgba(255,255,255,0.05);
border-radius:20px;
overflow:hidden;
position:relative;
height:160px;
}

.content{
position:absolute;
inset:0;
display:flex;
align-items:center;
justify-content:center;
text-align:center;
padding:15px;
z-index:1;
font-weight:bold;
}

canvas{
position:absolute;
inset:0;
width:100%;
height:100%;
z-index:2;
touch-action:none;
}

.progress{
margin-top:20px;
}

.bar{
height:10px;
background:#333;
border-radius:10px;
overflow:hidden;
}

.fill{
height:100%;
width:0%;
background:gold;
}

#msg{
margin-top:20px;
font-size:18px;
}

.confetti{
position:fixed;
top:-10px;
width:10px;
height:10px;
background:gold;
animation:fall linear forwards;
}

@keyframes fall{
to{
transform:translateY(100vh) rotate(720deg);
}
}
</style>
</head>

<body>

<h1>Raspadinha • 6 Tentativas</h1>
<p>Raspe todas as 6. Na última você ganha um brinde 🎁</p>

<div class="progress">
<div id="text">0 / 6</div>
<div class="bar"><div class="fill" id="fill"></div></div>
</div>

<div class="cards" id="cards"></div>

<div id="msg"></div>

<script>
const cardsContainer = document.getElementById("cards");
const fill = document.getElementById("fill");
const text = document.getElementById("text");
const msg = document.getElementById("msg");

let count = 0;
const total = 6;

const data = Array.from({length:6},(_,i)=>(
i===5
? {text:"🎁 Você ganhou um BRINDE!",win:true}
: {text:"😅 Tente novamente... você está quase!",win:false}
));

function update(){
text.innerText = count + " / " + total;
fill.style.width = (count/total*100)+"%";
}

function confetti(){
for(let i=0;i<80;i++){
let c=document.createElement("div");
c.className="confetti";
c.style.left=Math.random()*100+"vw";
c.style.background=["gold","white","pink"][Math.floor(Math.random()*3)];
c.style.animationDuration=(Math.random()*2+2)+"s";
document.body.appendChild(c);
setTimeout(()=>c.remove(),4000);
}
}

data.forEach((item,index)=>{
let card=document.createElement("div");
card.className="card";

card.innerHTML=`
<div class="content">${item.text}</div>
<canvas></canvas>
`;

cardsContainer.appendChild(card);

let canvas=card.querySelector("canvas");
let ctx=canvas.getContext("2d");

function resize(){
canvas.width=card.offsetWidth;
canvas.height=card.offsetHeight;

ctx.fillStyle="#999";
ctx.fillRect(0,0,canvas.width,canvas.height);

ctx.fillStyle="white";
ctx.font="bold 18px Arial";
ctx.textAlign="center";
ctx.fillText("RASPE",canvas.width/2,canvas.height/2);
}

resize();
window.addEventListener("resize",resize);

let drawing=false;
let revealed=false;

function scratch(x,y){
ctx.globalCompositeOperation="destination-out";
ctx.beginPath();
ctx.arc(x,y,20,0,Math.PI*2);
ctx.fill();
}

function getPos(e){
let r=canvas.getBoundingClientRect();
if(e.touches){
return{
x:e.touches[0].clientX-r.left,
y:e.touches[0].clientY-r.top
}
}
return{
x:e.clientX-r.left,
y:e.clientY-r.top
}
}

function check(){
if(revealed)return;

let data=ctx.getImageData(0,0,canvas.width,canvas.height).data;
let clear=0;

for(let i=3;i<data.length;i+=4){
if(data[i]===0)clear++;
}

if(clear/(canvas.width*canvas.height)>0.3){
revealed=true;
canvas.style.pointerEvents="none";
count++;
update();

if(item.win){
msg.innerHTML="🎉 Parabéns! Você ganhou um brinde!";
confetti();
}else{
msg.innerHTML="Continue... falta "+(total-count);
}
}
}

canvas.addEventListener("mousedown",e=>{
drawing=true;
let p=getPos(e);
scratch(p.x,p.y);
});

canvas.addEventListener("mousemove",e=>{
if(!drawing)return;
let p=getPos(e);
scratch(p.x,p.y);
});

window.addEventListener("mouseup",()=>{
drawing=false;
check();
});

canvas.addEventListener("touchstart",e=>{
drawing=true;
let p=getPos(e);
scratch(p.x,p.y);
e.preventDefault();
},{passive:false});

canvas.addEventListener("touchmove",e=>{
if(!drawing)return;
let p=getPos(e);
scratch(p.x,p.y);
e.preventDefault();
},{passive:false});

canvas.addEventListener("touchend",()=>{
drawing=false;
check();
});

});

update();
</script>

</body>
</html>
