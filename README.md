<!DOCTYPE html>
<html lang="zh">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Bouncing Ball</title>
<style>
*{margin:0;padding:0;box-sizing:border-box}
html,body{width:100%;height:100%;overflow:hidden;background:#0a0a0a}
canvas{display:block}
#errBox{display:none;position:fixed;top:50%;left:50%;transform:translate(-50%,-50%);color:#ff4444;font:16px/1.6 sans-serif;text-align:center;background:#1a1a1a;padding:24px 32px;border-radius:12px;border:1px solid #333;max-width:90vw;z-index:999}
#errBox a{color:#00ff66;text-decoration:none}
#errBox a:hover{text-decoration:underline}
</style>
</head>
<body>
<canvas id="game"></canvas>
<div id="errBox">
  <div id="errMsg">出错了</div>
  <div style="margin-top:12px;font-size:13px;color:#888">
    如有问题请联系：<a href="mailto:zzc.second055@simplelogin.com">zzc.second055@simplelogin.com</a>
  </div>
</div>
<script>
(function(){
  'use strict';

  // ---- 错误处理 ----
  function showError(msg){
    var box=document.getElementById('errBox');
    document.getElementById('errMsg').textContent=msg||'页面发生异常';
    box.style.display='block';
    // 隐藏canvas避免黑屏上叠文字难看
    var c=document.getElementById('game');
    if(c) c.style.display='none';
  }
  window.addEventListener('error',function(e){showError('渲染异常：'+e.message);});
  window.addEventListener('unhandledrejection',function(e){showError('网络/异步异常');});

  // ---- 重定向参数支持 ----
  // 用法：?redirect=https://目标地址 或 ?proxy=1&target=xxx
  var params=location.search?location.search.slice(1).split('&').reduce(function(o,p){
    var kv=p.split('=');o[kv[0]]=decodeURIComponent(kv[1]||'');return o;
  },{}):{};
  if(params.redirect){
    // 延迟跳转，确保页面至少渲染一帧
    setTimeout(function(){location.href=params.redirect;},100);
    return; // 不走后续逻辑
  }
  if(params.proxy && params.target){
    setTimeout(function(){location.href=params.target;},100);
    return;
  }

  // ---- 正常逻辑 ----
  var canvas=document.getElementById('game');
  var ctx=canvas.getContext('2d');
  var W,H;
  function resize(){
    W=canvas.width=window.innerWidth;
    H=canvas.height=window.innerHeight;
  }
  resize();
  window.addEventListener('resize',resize);
  window.addEventListener('orientationchange',function(){setTimeout(resize,50)});
  window.addEventListener('scroll',function(){W=window.innerWidth;H=window.innerHeight});

  // 代理前缀（可被外部注入覆盖）
  window.__PROXY_PREFIX__=window.__PROXY_PREFIX__||params.prefix||'';
  window.__WS_PATH__=window.__WS_PATH__||params.wspath||'/ws';

  var ball={x:200,y:200,r:20,vx:4,vy:3,color:'#00ff66'};

  function frame(){
    try{
      ctx.fillStyle='#0a0a0a';
      ctx.fillRect(0,0,W,H);
      ball.x+=ball.vx;
      ball.y+=ball.vy;
      if(ball.x-ball.r<0){ball.x=ball.r;ball.vx=Math.abs(ball.vx);}
      else if(ball.x+ball.r>W){ball.x=W-ball.r;ball.vx=-Math.abs(ball.vx);}
      if(ball.y-ball.r<0){ball.y=ball.r;ball.vy=Math.abs(ball.vy);}
      else if(ball.y+ball.r>H){ball.y=H-ball.r;ball.vy=-Math.abs(ball.vy);}
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
