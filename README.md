<!DOCTYPE html>
<html>
<head>
<meta charset=utf-8>
<title>极简射击</title>
<style>
*{margin:0;overflow:hidden;background:#000}
canvas{display:block}
#ui{position:fixed;top:6px;left:6px;color:#0f6;font:16px monospace;z-index:9}
#tip{position:fixed;top:55%;left:50%;transform:translateX(-50%);color:#ff0;font:14px monospace}
</style>
</head>
<body>
<canvas id=c></canvas>
<div id=ui>❤️5 🎯0</div>
<div id=tip>移动鼠标瞄准 · 点击射击 · WASD移动</div>
<script>
var c=document.getElementById('c'),x=c.getContext('2d');
c.width=innerWidth;c.height=innerHeight;
var W=c.width,H=c.height,M=Math;
var ui=document.getElementById('tip');
var px=W/2,py=H-120,mx=W/2,my=H/2;
var hp=2,sc=0,shot=false;
var keys={},enemies=[],bullets=[],parts=[];

function shoot(){
  if(!shot){shot=true;ui.style.display='none'}
  var dx=mx-px,dy=my-py,len=M.sqrt(dx*dx+dy*dy);
  if(len===0)return;
  bullets.push({x:px,y:py,vx:dx/len*10,vy:dy/len*10,r:5});
}

onmousedown=e=>{if(e.button===0)shoot()};
onkeydown=e=>{keys[e.code]=true;if(e.code==='Space')shoot()};
onkeyup=e=>keys[e.code]=false;
onmousemove=e=>{mx=e.clientX;my=e.clientY};

function spawn(){
  enemies.push({x:M.random()*W,y:-40,r:25+M.random()*20,s:0.3+M.random()*0.4});
}
spawn();

function loop(){
  requestAnimationFrame(loop);
  x.fillStyle='#0a0a12';x.fillRect(0,0,W,H);
  x.fillStyle='#050';x.fillRect(0,H*0.8,W,H*0.2);

  var a=M.atan2(my-py,mx-px);
  var spd=4;
  if(keys.KeyW){px+=M.cos(a)*spd;py+=M.sin(a)*spd}
  if(keys.KeyS){px-=M.cos(a)*spd;py-=M.sin(a)*spd}
  if(keys.KeyA){px+=M.cos(a-M.PI/2)*spd;py+=M.sin(a-M.PI/2)*spd}
  if(keys.KeyD){px+=M.cos(a+M.PI/2)*spd;py+=M.sin(a+M.PI/2)*spd}
  px=M.max(20,M.min(W-20,px));py=M.max(20,M.min(H-20,py));

  // 玩家
  x.save();x.translate(px,py);x.rotate(a);
  x.fillStyle='#0f6';x.beginPath();x.moveTo(18,0);x.lineTo(-10,-10);x.lineTo(-10,10);x.fill();x.restore();

  // 准星
  x.strokeStyle='#0f6';x.lineWidth=2;x.beginPath();
  x.moveTo(mx-10,my);x.lineTo(mx+10,my);x.moveTo(mx,my-10);x.lineTo(mx,my+10);x.stroke();

  // 敌人
  for(var i=enemies.length-1;i>=0;i--){
    var e=enemies[i];
    e.y+=e.s;
    if(e.y>H+50){enemies.splice(i,1);spawn();continue}
    x.beginPath();x.arc(e.x,e.y,e.r,0,7);x.fillStyle='#f22';x.fill();
    x.shadowColor='#f00';x.shadowBlur=12;x.fill();x.shadowBlur=0;
    if(M.hypot(px-e.x,py-e.y)<e.r+15){
      hp--;updateUI();
      enemies.splice(i,1);spawn();
      if(hp<=0){alert('游戏结束！得分:'+sc);location.reload()}
    }
  }

  // 子弹
  for(var i=bullets.length-1;i>=0;i--){
    var b=bullets[i];b.x+=b.vx;b.y+=b.vy;
    x.beginPath();x.arc(b.x,b.y,b.r,0,7);x.fillStyle='#ff0';x.fill();
    var hit=false;
    for(var j=enemies.length-1;j>=0;j--){
      var e=enemies[j];
      if(M.hypot(b.x-e.x,b.y-e.y)<e.r+b.r){
        sc++;updateUI();
        for(var p=0;p<8;p++)parts.push({x:e.x,y:e.y,vx:(M.random()-.5)*6,vy:(M.random()-.5)*6,l:15});
        enemies.splice(j,1);spawn();
        // 每50分回1血
        if(sc%50===0&&hp<5)hp++;
        hit=true;break;
      }
    }
    if(hit||b.x<0||b.x>W||b.y<0||b.y>H)bullets.splice(i,1);
  }

  // 粒子
  for(var i=parts.length-1;i>=0;i--){var p=parts[i];p.x+=p.vx;p.y+=p.vy;p.l--;x.globalAlpha=p.l/15;x.fillStyle='#ff0';x.fillRect(p.x,p.y,3,3);x.globalAlpha=1;if(p.l<=0)parts.splice(i,1);}
}
function updateUI(){document.getElementById('ui').innerHTML='❤️'+hp+' 🎯'+sc}
loop();
</script>
</body>
</html>
