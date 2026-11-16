# beacon-run
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
<title>Beacon Run</title>
<style>
 body{margin:0;background:#111;color:#eee;font-family:system-ui,sans-serif;display:flex;justify-content:center;padding:10px;user-select:none;-webkit-user-select:none}
 #wrap{width:100%;max-width:360px}
 #hud{display:flex;justify-content:space-between;font-size:14px;margin-bottom:8px;color:#aaa}
 #hud b{color:#fff}
 canvas{width:100%;display:block;border:1px solid #444;border-radius:12px;touch-action:none}
 #pad{display:flex;gap:6px;margin-top:10px}
 button{flex:1;height:56px;font-size:20px;background:#2b2b33;color:#fff;border:1px solid #555;border-radius:10px;touch-action:none}
 button:active{background:#444}
 #jump{flex:1.6;font-size:16px}
 #rs{height:40px;font-size:14px;width:100%;margin-top:10px}
</style>
</head>
<body>
<div id="wrap">
 <div id="hud">
  <span>Lives: <b id="lv">3</b></span>
  <span id="msg">Tap the screen to start</span>
  <span>Cores: <b id="co">0</b></span>
 </div>
 <canvas id="c" width="360" height="480"></canvas>
 <div id="pad">
  <button id="bl">&#9664;</button>
  <button id="jump">Jump</button>
  <button id="br">&#9654;</button>
 </div>
 <button id="rs">Restart</button>
</div>
<script>
var cv=document.getElementById('c'),x=cv.getContext('2d');
var W=360,H=480,HZ=170,F=250,Z0=2.4,CH=2;
var LANES=[-1.3,0,1.3];
var ac=null,lane,px,py,vy,d,speed,L,obs,cores,lives,level,got,state,last=0;
function beep(f,t,w){try{if(!ac)ac=new (window.AudioContext||window.webkitAudioContext)();if(ac.state==='suspended')ac.resume();
 var o=ac.createOscillator(),g=ac.createGain();o.type=w||'square';o.frequency.value=f;g.gain.setValueAtTime(0.06,ac.currentTime);
 g.gain.exponentialRampToValueAtTime(0.0001,ac.currentTime+t);o.connect(g);g.connect(ac.destination);o.start();o.stop(ac.currentTime+t);}catch(e){}}
function $(i){return document.getElementById(i)}
function msg(t){$('msg').textContent=t}
function hud(){$('lv').textContent=lives;$('co').textContent=got}
function pr(X,Y,Z){var s=F/(Z+Z0);return [W/2+X*s,HZ+(CH-Y)*s]}
function poly(p,c){x.fillStyle=c;x.beginPath();x.moveTo(p[0][0],p[0][1]);for(var i=1;i<p.length;i++)x.lineTo(p[i][0],p[i][1]);x.closePath();x.fill()}
function box(X,Y,Z,w,h,dp,cf,ct,cs){
 var x0=X-w/2,x1=X+w/2,y1=Y+h,zb=Z+dp;
 if(x1<0)poly([pr(x1,Y,Z),pr(x1,Y,zb),pr(x1,y1,zb),pr(x1,y1,Z)],cs);
 if(x0>0)poly([pr(x0,Y,Z),pr(x0,Y,zb),pr(x0,y1,zb),pr(x0,y1,Z)],cs);
 if(y1<CH)poly([pr(x0,y1,Z),pr(x1,y1,Z),pr(x1,y1,zb),pr(x0,y1,zb)],ct);
 poly([pr(x0,Y,Z),pr(x1,Y,Z),pr(x1,y1,Z),pr(x0,y1,Z)],cf);
}
function build(){
 obs=[];cores=[];L=200+level*80;
 for(var z=20;z<L-12;z+=7+Math.random()*5){
  var n=Math.random()<0.4?2:1,tall=Math.random()<0.5,used=[];
  for(var k=0;k<n;k++){var ln=Math.floor(Math.random()*3);if(used.indexOf(ln)<0){used.push(ln);obs.push({x:LANES[ln],z:z,h:tall?1.8:0.8})}}
  if(used.length>=2&&tall){obs[obs.length-1].h=0.8}
  var cl=Math.floor(Math.random()*3);cores.push({x:LANES[cl],z:z+4,got:false});
 }
}
function startLevel(){lane=1;px=0;py=0;vy=0;d=0;speed=8+level*1.2;build()}
function newGame(){lives=3;level=1;got=0;startLevel();state='ready';hud();msg('Tap the screen to start')}
function begin(){
 if(state==='ready'){state='play';msg('Reach the beacon')}
 else if(state==='dead'){state='play';msg('Reach the beacon')}
 else if(state==='win'){level++;startLevel();state='play';msg('Reach the beacon')}
 else if(state==='over'){newGame();state='play';msg('Reach the beacon')}
}
function left(){begin();if(state==='play')lane=Math.max(0,lane-1)}
function right(){begin();if(state==='play')lane=Math.min(2,lane+1)}
function jump(){begin();if(state==='play'&&py===0){vy=6.8;beep(520,0.08)}}
function bind(id,f){$(id).addEventListener('pointerdown',function(e){e.preventDefault();f()})}
bind('bl',left);bind('br',right);bind('jump',jump);
cv.addEventListener('pointerdown',begin);
$('rs').onclick=function(){newGame()};
window.addEventListener('keydown',function(e){
 if(e.key==='ArrowLeft'){left()}else if(e.key==='ArrowRight'){right()}else if(e.key==='ArrowUp'||e.key===' '){jump()}else return;
 e.preventDefault()});
function hit(){
 lives--;hud();beep(150,0.35,'sawtooth');
 for(var i=obs.length-1;i>=0;i--){if(obs[i].z-d<12)obs.splice(i,1)}
 if(lives<=0){state='over';msg('Game over. Tap to retry')}else{state='dead';msg('Ouch. Tap to continue')}
}
function update(dt){
 d+=speed*dt;px+=(LANES[lane]-px)*Math.min(1,dt*14);
 if(py>0||vy>0){vy-=18*dt;py+=vy*dt;if(py<=0){py=0;vy=0}}
 for(var i=0;i<obs.length;i++){var o=obs[i],rz=o.z-d;
  if(rz<0.4&&rz+1>-0.4&&Math.abs(px-o.x)<0.6&&py<o.h-0.05){hit();return}}
 for(var j=0;j<cores.length;j++){var c=cores[j],cz=c.z-d;
  if(!c.got&&Math.abs(cz)<0.7&&Math.abs(px-c.x)<0.6&&py<1.5){c.got=true;got++;hud();beep(880,0.08)}}
 if(d>=L){state='win';msg('You made it. Tap for next level');beep(523,0.12);setTimeout(function(){beep(659,0.12)},130);setTimeout(function(){beep(784,0.2)},260)}
}
function draw(t){
 x.fillStyle='#1d2b4a';x.fillRect(0,0,W,HZ);
 x.fillStyle='#16161c';x.fillRect(0,HZ,W,H-HZ);
 poly([pr(-2.2,0,-1.5),pr(2.2,0,-1.5),pr(2.2,0,90),pr(-2.2,0,90)],'#2b2b33');
 var off=d%4;
 for(var z=-off;z<90;z+=4){for(var s=-1;s<=1;s+=2){var lx=0.65*s;
  poly([pr(lx-0.03,0,z),pr(lx+0.03,0,z),pr(lx+0.03,0,z+1.5),pr(lx-0.03,0,z+1.5)],'#6b6b75')}}
 var fz=L-d;
 if(fz<90&&fz>-1.4){poly([pr(-2.2,0,fz),pr(2.2,0,fz),pr(2.2,0,fz+1),pr(-2.2,0,fz+1)],'#EF9F27');
  box(-2.6,0,fz,0.6,6,0.6,'#EF9F27','#FAC775','#BA7517');box(2.6,0,fz,0.6,6,0.6,'#EF9F27','#FAC775','#BA7517')}
 var items=[];
 for(var i=0;i<obs.length;i++){var o=obs[i],rz=o.z-d;if(rz>-1.4&&rz<90)items.push({z:rz,k:0,o:o})}
 for(var j=0;j<cores.length;j++){var c=cores[j],cz=c.z-d;if(!c.got&&cz>-1.4&&cz<90)items.push({z:cz,k:1,o:c})}
 items.push({z:0,k:2});
 items.sort(function(a,b){return b.z-a.z});
 for(var m=0;m<items.length;m++){var it=items[m];
  if(it.k===0){var tl=it.o.h>1;box(it.o.x,0,it.z,0.9,it.o.h,1,tl?'#7F77DD':'#D85A30',tl?'#AFA9EC':'#F0997B',tl?'#534AB7':'#993C1D')}
  else if(it.k===1){var bob=0.9+Math.sin(t/200)*0.1,p=pr(it.o.x,bob,it.z),sz=0.28*F/(it.z+Z0);
   poly([[p[0],p[1]-sz],[p[0]+sz*0.7,p[1]],[p[0],p[1]+sz],[p[0]-sz*0.7,p[1]]],'#FAC775')}
  else{box(px,py,0,0.6,0.6,0.6,'#378ADD','#85B7EB','#185FA5')}
 }
 x.fillStyle='#aaa';x.font='12px system-ui';x.textAlign='left';
 x.fillText('Level '+level+'   '+Math.max(0,Math.round(L-d))+' m to go',8,18);
}
function loop(t){
 var dt=Math.min(0.05,(t-last)/1000||0);last=t;
 if(state==='play')update(dt);
 draw(t);requestAnimationFrame(loop);
}
newGame();requestAnimationFrame(loop);
</script>
</body>
</html>
