
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Noite de Terror 3D</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#030407;color:white;font-family:Arial;overflow:hidden}
#game{position:fixed;inset:0}
canvas{display:block}
#hud{position:fixed;inset:0;pointer-events:none;text-shadow:2px 2px 4px black}
#time{position:absolute;right:20px;top:15px;font-size:28px;font-weight:bold}
#power{position:absolute;left:18px;top:18px;font-size:20px}
#usage{color:#7cff8b}
#message{position:absolute;top:45%;width:100%;text-align:center;font-size:25px}
#controls{position:fixed;bottom:15px;left:0;right:0;display:flex;justify-content:center;gap:8px;flex-wrap:wrap}
button{pointer-events:auto;padding:13px 16px;background:#252932;color:white;border:1px solid #777;border-radius:5px;font-weight:bold;cursor:pointer}
button:active{background:#7d2222}
#monitor{display:none;position:fixed;inset:5%;background:#07100e;border:5px solid #6c756f;z-index:4;padding:15px;text-align:center;box-shadow:0 0 80px black}
#monitor h2{margin:5px}
#camview{height:65%;position:relative;overflow:hidden;background:repeating-linear-gradient(0deg,#151a19,#151a19 2px,#080b0a 3px,#080b0a 5px)}
#camview canvas{width:100%;height:100%}
#camname{position:absolute;top:10px;left:10px;font-size:19px}
#cams{display:flex;justify-content:center;flex-wrap:wrap;gap:5px;margin-top:10px}
#scare{display:none;position:fixed;inset:0;background:#050505;z-index:10;align-items:center;justify-content:center;flex-direction:column}
#scareface{font-size:min(55vw,400px);filter:contrast(1.8);animation:shake .07s infinite}
@keyframes shake{0%{transform:translate(8px,-4px) rotate(3deg)}50%{transform:translate(-8px,5px) rotate(-3deg)}100%{transform:translate(2px,3px)}}
#start{position:fixed;inset:0;z-index:20;background:radial-gradient(#202020,#000 70%);display:flex;align-items:center;justify-content:center;text-align:center}
.card{max-width:600px;padding:25px}
h1{font-size:clamp(32px,6vw,65px);letter-spacing:3px}
#start button{background:#721b1b;font-size:20px;padding:18px 35px}
.small{color:#bbb;line-height:1.5}
</style>
</head>
<body>
<div id="game"></div>
<div id="hud">
 <div id="power">ENERGIA: <span id="battery">100</span>%<br><span id="usage">USO: ▮</span></div>
 <div id="time">12 AM</div>
 <div id="message"></div>
</div>
<div id="controls">
 <button onclick="door('left')">🚪 PORTA ESQUERDA</button>
 <button onclick="light('left')">💡 LUZ ESQUERDA</button>
 <button onclick="monitor()">📹 CÂMERAS</button>
 <button onclick="light('right')">💡 LUZ DIREITA</button>
 <button onclick="door('right')">🚪 PORTA DIREITA</button>
</div>
<div id="monitor">
 <h2>SISTEMA DE VIGILÂNCIA</h2>
 <div id="camview"><div id="camname">CAM 01 — PALCO</div></div>
 <div id="cams">
 <button onclick="cameraTo(0)">CAM 01 · PALCO</button>
 <button onclick="cameraTo(1)">CAM 02 · SALÃO</button>
 <button onclick="cameraTo(2)">CAM 03 · DEPÓSITO</button>
 <button onclick="cameraTo(3)">CAM 04 · CORREDOR</button>
 <button onclick="monitor()">FECHAR MONITOR</button>
 </div>
 <p class="small">Sinal instável • Áudio ativo • Câmeras noturnas</p>
</div>
<div id="scare"><div id="scareface">🐻</div><h1>VOCÊ FOI ENCONTRADO</h1><p id="endtext">Fim de jogo</p><button onclick="location.reload()">TENTAR NOVAMENTE</button></div>
<div id="start"><div class="card"><h1>NOITE DE TERROR</h1><p>Turno da madrugada — 12 AM às 6 AM</p><p class="small">Sobreviva usando portas, luzes e câmeras. Cada equipamento consome energia. Não deixe os visitantes chegarem até você.</p><button onclick="startGame()">INICIAR NOITE</button></div></div>
<script type="module">
import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';

const scene=new THREE.Scene();
scene.background=new THREE.Color(0x050607);
scene.fog=new THREE.FogExp2(0x070809,.055);
const camera=new THREE.PerspectiveCamera(72,innerWidth/innerHeight,.1,100);
camera.position.set(0,1.7,4.5);
const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,1.7));
renderer.shadowMap.enabled=true;
renderer.shadowMap.type=THREE.PCFSoftShadowMap;
renderer.toneMapping=THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure=1.05;
document.getElementById('game').appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0x7b8795,0x17110c,.55));
const lamp=new THREE.PointLight(0xffd6a0,18,12);
lamp.position.set(0,3,0);
scene.add(lamp);
const ceilingLight=new THREE.SpotLight(0xc8d9ff,35,18,Math.PI/3,.6);
ceilingLight.position.set(0,3.7,1);
ceilingLight.target.position.set(0,0,-4);
scene.add(ceilingLight,ceilingLight.target);

const mat=(color,roughness=.8,metalness=0)=>new THREE.MeshStandardMaterial({color,roughness,metalness});
const wallMat=mat(0x292a28),floorMat=mat(0x22201c),metal=mat(0x55595b,.45,.7);
function box(parent,x,y,z,w,h,d,material){
 const m=new THREE.Mesh(new THREE.BoxGeometry(w,h,d),material);
 m.position.set(x,y,z);m.castShadow=true;m.receiveShadow=true;parent.add(m);return m;
}
box(scene,0,-.15,0,8,.3,11,floorMat);
box(scene,0,4.1,0,8,.25,11,wallMat);
box(scene,-4,1.9,0,.2,4,11,wallMat);
box(scene,4,1.9,0,.2,4,11,wallMat);
box(scene,0,1.9,-5.5,8,4,.2,wallMat);
// Back wall details
for(let i=-3;i<=3;i++){
 box(scene,i,2.7,-5.35,.08,1.7,.06,mat(0x3b3029));
}
// Desk and equipment
box(scene,0,.85,2.4,3,.18,1.1,mat(0x30251e));
box(scene,0,1.1,2.2,.75,.48,.12,mat(0x111719));
box(scene,-.8,1.03,2.05,.4,.04,.3,mat(0x666b6b));
for(let x of [-1.5,1.5])box(scene,x,.4,2.65,.1,.8,.1,metal);
// Ventilation ducts
box(scene,-2.8,3.55,1.5,.75,.18,1.5,metal);
box(scene,2.8,3.55,1.5,.75,.18,1.5,metal);

const doors={left:false,right:false};
const lights={left:false,right:false};
const doorMeshes={};
for(const side of ['left','right']){
 const x=side==='left'?-3.15:3.15;
 const dm=box(scene,x,1.7,.05,.85,3.1,.2,mat(0x777b80,.4,.65));
 dm.visible=false;doorMeshes[side]=dm;
 box(scene,x,1.7,.18,1,.12,.12,mat(0x17191c));
 const glow=new THREE.PointLight(0xffffff,0,7);
 glow.position.set(x,1.7,-.7);scene.add(glow);
 lights[side]=false;
 lights[side+'_obj']=glow;
}

// Animatrônico original, montado com formas 3D
function makeBot(color,kind=0){
 const g=new THREE.Group();
 const bodyMat=mat(color,.65,.15),dark=mat(0x151514,.4,.4),eye=mat(0xff2020,.25);
 const body=new THREE.Mesh(new THREE.SphereGeometry(.55,20,16),bodyMat);
 body.position.y=1.05;body.scale.set(.85,1.1,.6);body.castShadow=true;g.add(body);
 const head=new THREE.Mesh(new THREE.SphereGeometry(.48,24,20),bodyMat);
 head.position.set(0,1.95,0);head.castShadow=true;g.add(head);
 for(let x of [-.31,.31]){
  const ear=new THREE.Mesh(new THREE.SphereGeometry(.18,16,12),bodyMat);
  ear.position.set(x,2.35,0);g.add(ear);
  const e=new THREE.Mesh(new THREE.SphereGeometry(.075,12,12),eye);
  e.position.set(x*.75,2.02,.42);g.add(e);
  const pupil=new THREE.Mesh(new THREE.SphereGeometry(.025,8,8),mat(0xffffff));
  pupil.position.set(x*.75,2.02,.48);g.add(pupil);
 }
 const muzzle=new THREE.Mesh(new THREE.SphereGeometry(.22,16,12),dark);
 muzzle.position.set(0,1.82,.4);muzzle.scale.set(1,.7,.55);g.add(muzzle);
 const jaw=box(g,0,1.62,.38,.32,.08,.16,dark);
 for(let x of [-.34,.34]){
  const arm=new THREE.Mesh(new THREE.CylinderGeometry(.13,.17,.85,12),bodyMat);
  arm.position.set(x,.95,0);arm.rotation.z=x<0?-.15:.15;g.add(arm);
  const leg=new THREE.Mesh(new THREE.CylinderGeometry(.17,.2,.65,12),dark);
  leg.position.set(x*.6,.3,0);g.add(leg);
 }
 g.userData.kind=kind;
 return g;
}
const bots=[
 {name:'O URSO',obj:makeBot(0x55433b,0),pos:0,side:'left',rate:8},
 {name:'A RAPOSA',obj:makeBot(0x77716a,1),pos:0,side:'right',rate:10}
];
bots[1].obj.scale.set(.9,1.05,.9);
scene.add(...bots.map(b=>b.obj));
function placeBot(b){
 const spots=[
  [0,0,-4.6],[-2.3,0,-3.4],[2.3,0,-3.4],
  [-3,0,-1.7],[3,0,-1.7],[-3.1,0,.25],[3.1,0,.25]
 ];
 const p=spots[Math.min(b.pos,spots.length-1)];
 b.obj.position.set(...p);
 b.obj.rotation.y=(p[0]<0?-.15:.15);
 b.obj.visible=b.pos>0;
}
bots.forEach(placeBot);

// Camera feeds are stylized views of the actual animatronics.
let camIndex=0,monitorOpen=false,running=false,gameOver=false;
let battery=100,elapsed=0,lastTick=0,lastMove=0;
const totalNight=240; // 4 minutos de jogo = 6 horas
function cameraTo(n){
 camIndex=n;
 document.getElementById('camname').textContent=[
  'CAM 01 — PALCO','CAM 02 — SALÃO','CAM 03 — DEPÓSITO','CAM 04 — CORREDOR'
 ][n];
 const cv=document.getElementById('camview');
 cv.style.background=[
  'radial-gradient(#38332b,#080908 75%)',
  'radial-gradient(#242725,#030404 75%)',
  'radial-gradient(#373b3d,#050606 75%)',
  'radial-gradient(#292b31,#020303 75%)'
 ][n];
 cv.querySelectorAll('.camBot').forEach(e=>e.remove());
 const count=bots.filter(b=>b.pos===n || (n===3&&b.pos>=3)).length;
 if(count){
  const b=bots.find(b=>b.pos===n || (n===3&&b.pos>=3));
  const face=document.createElement('div');face.className='camBot';
  face.textContent=b.name==='O URSO'?'🐻':'🦊';
  face.style.cssText='font-size:100px;position:absolute;top:25%;left:35%;filter:grayscale(1) contrast(2);text-shadow:0 0 20px #aaa';
  cv.appendChild(face);
 }
}
function monitor(){
 if(!running||gameOver)return;
 monitorOpen=!monitorOpen;
 document.getElementById('monitor').style.display=monitorOpen?'block':'none';
 camera.position.set(0,1.7,monitorOpen?4.5:4.5);
}
window.monitor=monitor;window.cameraTo=cameraTo;
function door(s){
 if(!running||gameOver)return;
 doors[s]=!doors[s];doorMeshes[s].visible=doors[s];
 doorMeshes[s].position.y=doors[s]?1.7:5;
}
function light(s){
 if(!running||gameOver)return;
 lights[s]=!lights[s];
 lights[s+'_obj'].intensity=lights[s]?35:0;
}
window.door=door;window.light=light;
function startGame(){
 document.getElementById('start').style.display='none';
 running=true;elapsed=0;battery=100;
 bots.forEach(b=>{b.pos=0;placeBot(b)});
}
window.startGame=startGame;
function endGame(win,reason){
 gameOver=true;running=false;
 if(monitorOpen)document.getElementById('monitor').style.display='none';
 if(win){
  document.getElementById('scareface').textContent='🌅';
  document.getElementById('scare').style.background='#000';
  document.getElementById('scareface').style.animation='none';
  document.getElementById('endtext').textContent='6 AM — VOCÊ SOBREVIVEU!';
 }else{
  document.getElementById('scareface').textContent='🤖';
  document.getElementById('endtext').textContent=reason||'O animatrônico entrou no escritório.';
 }
 document.getElementById('scare').style.display='flex';
}
let lastFrame=performance.now();
function animate(now){
 requestAnimationFrame(animate);
 const dt=Math.min((now-lastFrame)/1000,.05);lastFrame=now;
 if(running&&!gameOver){
  elapsed+=dt;
  if(elapsed>=totalNight){endGame(true);return}
  const hour=Math.floor(elapsed/40);
  document.getElementById('time').textContent=(hour===0?'12':hour)+' AM';
  const drain=.035+(doors.left?.12:0)+(doors.right?.12:0)+(lights.left?.12:0)+(lights.right?.12:0)+(monitorOpen?.08:0);
  battery=Math.max(0,battery-drain*dt*10);
  document.getElementById('battery').textContent=Math.ceil(battery);
  document.getElementById('usage').textContent='USO: '+('▮'.repeat(1+(doors.left?1:0)+(doors.right?1:0)+(monitorOpen?1:0)));
  if(battery<=0){
   doors.left=doors.right=false;
   doorMeshes.left.visible=doorMeshes.right.visible=false;
   lights.left=lights.right=false;
   lights.left_obj.intensity=lights.right_obj.intensity=0;
   lamp.intensity=0;ceilingLight.intensity=0;
   document.getElementById('message').textContent='A ENERGIA ACABOU...';
  }
  if(now-lastMove>6500){
   lastMove=now;
   bots.forEach(b=>{
    if(Math.random()<.72){
     b.pos=Math.min(b.pos+1,5);
     placeBot(b);
    }
   });
  }
  bots.forEach(b=>{
   if(b.pos>=5){
    const blocked=doors[b.side]&&battery>0;
    if(blocked){b.pos=2;placeBot(b)}
    else if(Math.random()<.025)endGame(false,b.name+' chegou até você!');
   }
  });
 }
 // Animatrônicos podem ser vistos no escritório quando muito próximos.
 bots.forEach(b=>{
  if(b.pos>=4&&!monitorOpen&&!gameOver){
   b.obj.visible=true;
   b.obj.position.x=b.side==='left'?-2.3:2.3;
   b.obj.position.z=-.4;
   b.obj.position.y=Math.sin(now*.003)*.025;
   b.obj.rotation.y=Math.sin(now*.001)*.1;
  }else if(b.pos<4)placeBot(b);
 });
 camera.lookAt(0,1.4,-1);
 renderer.render(scene,camera);
}
requestAnimationFrame(animate);
addEventListener('resize',()=>{
 camera.aspect=innerWidth/innerHeight;camera.updateProjectionMatrix();
 renderer.setSize(innerWidth,innerHeight);
});
cameraTo(0);
</script>
</body>
</html>
