<!DOCTYPE html>
<canvas id=c></canvas>
<div id=ui>❤️5 🎯0</div>
<div id=tip>移动鼠标瞄准 · 点击射击</div>
<style>
*{margin:0;overflow:hidden;background:#000}
canvas{display:block}
#ui{position:fixed;top:6px;left:6px;color:#0f6;font:14px monospace}
#tip{position:fixed;top:55%;left:50%;transform:translateX(-50%);color:#ff0;font:14px monospace}
</style>
<script>
var c=document.getElementById('c'),x=c.getContext('2d');
c.width=innerWidth;c.height=innerHeight;
var W=c.width,H=c.height;
var ui=document.getElementById('ui'),tip=document.getElementById('tip');
var hp=5,sc=0,bullets=[],enemies=[],parts=[];
var px=W/2,py=H*0.7;  // 玩家位置固定偏下方
var mx=W/2,my=H/2;     // 鼠标位置
var shot=false;

// ===== 射击：从玩家位置往鼠标方向发射 =====
function shoot(){
  if(!shot){shot=true;tip.style.display='none'}
  // 向量：从玩家指向鼠标
  var dx=mx-px, dy=my-py;
  var len=Math.sqrt(dx*dx+dy*dy);
  if(len===0)return;
  // 归一化后乘速度
  var vx=dx/len*10, vy=dy/len*10;
  bullets.push({x:px,y:py,vx:vx,vy:vy,r:5});
}

document.addEventListener('mousedown',e=>{if(e.button===0)shoot()});
document.addEventListener('keydown',e=>{if(e.code==='Space')shoot()});
document.addEventListener('mousemove',e=>{mx=e.clientX;my=e.clientY});

// ===== 敌人：只在玩家正前方直线生成 =====
// "前方"=玩家到屏幕中心的连线方向
function spawn(){
  // 玩家朝向上方（屏幕中心方向）
  // 敌人生成在玩家"上方"区域（y < py），偏移不超过±20%
  var ex=px+(Math.random()-0.5)*W*0.4;  // 水平偏移有限
  var ey=py-(200+Math.random()*200);       // 在玩家上方200~400像素
  var r=25+Math.random()*20;
  enemies.push({x:ex,y:ey,r:r});
}
spawn();spawn();

// ===== 主循环 =====
function loop(){
  requestAnimationFrame(loop);
  x.fillStyle='#0a0a18';x.fillRect(0,0,W,H);
  x.fillStyle='#1a2a1a';x.fillRect(0,H*0.65,W,H*0.35);

  // 画瞄准线（从玩家到鼠标，虚线）
  x.strokeStyle='rgba(0,255,100,0.3)';x.setLineDash([5,5]);x.lineWidth=1;
  x.beginPath();x.moveTo(px,py);x.lineTo(mx,my);x.stroke();x.setLineDash([]);

  // 准星（跟着鼠标）
  x.strokeStyle='#0f6';x.lineWidth=2;x.beginPath();
  x.moveTo(mx-10,my);x.lineTo(mx+10,my);
  x.moveTo(mx,my-10);x.lineTo(mx,my+10);x.stroke();

  // 玩家（绿色三角形，底边朝鼠标方向）
  var angle=Math.atan2(my-py,mx-px);
  x.save();x.translate(px,py);x.rotate(angle);
  x.fillStyle='#0f6';x.beginPath();x.moveTo(20,0);x.lineTo(-12,-12);x.lineTo(-12,12);x.fill();x.restore();

  // 敌人
  for(var i=enemies.length-1;i>=0;i--){
    var e=enemies[i];
    // 敌人只往玩家直线追（从上往下走）
    var dx=px-e.x, dy=py-e.y;
    var d=Math.sqrt(dx*dx+dy*dy);
    e.x+=dx/d*0.5;e.y+=dy/d*0.5;

    // 画敌人
    x.beginPath();x.arc(e.x,e.y,e.r,0,7);x.fillStyle='#f22';x.fill();
    x.shadowColor='#f00';x.shadowBlur=15;x.fill();x.shadowBlur=0;

    // 碰触
    if(d<e.r+15){hp--;ui.innerHTML='❤️'+hp+' 🎯'+sc;enemies.splice(i,1);spawn();if(hp<=0){alert('Over! '+sc);location.reload()}}
  }

  // 子弹
  for(var i=bullets.length-1;i>=0;i--){
    var b=bullets[i];
    b.x+=b.vx;b.y+=b.vy;

    // 画子弹（带尾迹）
    x.beginPath();x.arc(b.x,b.y,b.r,0,7);x.fillStyle='#ff0';x.fill();

    // 命中检测
    var hit=false;
    for(var j=enemies.length-1;j>=0;j--){
      var e=enemies[j];
      var dx=b.x-e.x,dy=b.y-e.y;
      if(dx*dx+dy*dy<(e.r+b.r)*(e.r+b.r)){
        sc++;ui.innerHTML='❤️'+hp+' 🎯'+sc;
        for(var p=0;p<6;p++)parts.push({x:e.x,y:e.y,vx:(Math.random()-.5)*6,vy:(Math.random()-.5)*6,l:15});
        enemies.splice(j,1);spawn();hit=true;break;
      }
    }
    if(hit||b.x<0||b.x>W||b.y<0||b.y>H)bullets.splice(i,1);
  }

  // 粒子
  for(var i=parts.length-1;i>=0;i--){var p=parts[i];p.x+=p.vx;p.y+=p.vy;p.l--;x.globalAlpha=p.l/15;x.fillStyle='#ff0';x.fillRect(p.x,p.y,3,3);x.globalAlpha=1;if(p.l<=0)parts.splice(i,1);}
}
loop();
</script>      else if(ball.y+ball.r>H){ball.y=H-ball.r;ball.vy=-Math.abs(ball.vy);}
      ball.x=Math.max(ball.r,Math.min(W-ball.r,ball.x));
      ball.y=Math.max(ball.r,Math.min(H-ball.r,ball.y));
      ctx.beginPath();
      ctx.arc(ball.x,ball.y,ball.r,0,Math.PI*2);
      ctx.fillStyle=ball.color;
      ctx.fill();
      ctx.closePath();
      requestAnimationFrame(frame);
    }catch(err){
      showError('动画循环异常：'+err.message);
    }
  }
  requestAnimationFrame(frame);
})();
</script>
</body>
</html>
    // 画球
    ctx.beginPath();
    ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
    ctx.fillStyle = ball.color;
    ctx.fill();
    ctx.closePath();

    // 更新位置
    ball.x += ball.dx;
    ball.y += ball.dy;

    // 碰壁反弹
    if (ball.x + ball.radius > canvas.width || ball.x - ball.radius < 0) {
      ball.dx = -ball.dx;
    }
    if (ball.y + ball.radius > canvas.height || ball.y - ball.radius < 0) {
      ball.dy = -ball.dy;
    }

    requestAnimationFrame(draw);
  }

  draw();
</script>
</body>
</html>
