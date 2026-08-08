<!DOCTYPE html>
<html>
<head>
<meta charset=utf-8>
<title>MicroOS</title>
<style>
*{margin:0;padding:0;box-sizing:border-box;font:14px monospace;user-select:none}
body{background:#1a1a2e;height:100vh;overflow:hidden}
#desktop{position:absolute;top:0;left:0;right:0;bottom:40px;padding:20px;display:grid;grid-template-columns:repeat(auto-fill,80px);gap:20px}
.icon{width:80px;text-align:center;color:#0f6;cursor:pointer;padding:8px 5px;border-radius:6px;transition:background .15s}
.icon:hover{background:rgba(0,255,100,0.12)}
.icon span{font-size:36px;display:block;margin-bottom:4px}
#taskbar{position:absolute;bottom:0;left:0;right:0;height:40px;background:#0a0a18;border-top:1px solid#333;display:flex;align-items:center;padding:0 10px;gap:8px;z-index:9999}
.task-btn{background:#2a2a3e;color:#0f6;padding:5px 12px;border-radius:4px;cursor:pointer;max-width:140px;overflow:hidden;text-overflow:ellipsis;white-space:nowrap;border:1px solid#333}
.task-btn:hover{background:#3a3a4e}
.task-btn.active{background:#0f6;color:#000}
.window{position:absolute;background:#15152a;border:1px solid#333;border-radius:6px;min-width:320px;min-height:200px;box-shadow:0 8px 24px rgba(0,0,0,.5);display:none;flex-direction:column;overflow:hidden}
.win-title{background:#0a0a18;color:#0f6;padding:8px 12px;display:flex;justify-content:space-between;align-items:center;border-bottom:1px solid#333;font-weight:bold}
.win-close{color:#f66;cursor:pointer;padding:0 6px;font-size:18px;line-height:1}
.win-close:hover{color:#faa}
.win-content{flex:1;padding:10px;overflow:auto}
.terminal{color:#0f6;height:100%;overflow:auto;white-space:pre-wrap;user-select:text;line-height:1.5}
.term-input{display:flex;align-items:center;margin-top:4px}
.term-input span{color:#0f6;margin-right:6px}
.term-input input{background:transparent;border:none;color:#0f6;outline:none;flex:1;font:14px monospace}
</style>
</head>
<body>

<div id=desktop>
  <div class=icon onclick="openApp('terminal')"><span>💻</span><div>终端</div></div>
  <div class=icon onclick="openApp('about')"><span>ℹ️</span><div>关于</div></div>
  <div class=icon onclick="openApp('game')"><span>🎮</span><div>射击游戏</div></div>
</div>

<div id=taskbar></div>
// ===== 射击游戏应用 =====
function initGame(win){
  var c=win.querySelector('canvas'),x=c.getContext('2d');
  var W=c.width=c.clientWidth,H=c.height=c.clientHeight,M=Math;
  var px=W/2,py=H-120,mx=W/2,my=H/2;
  var hp=2,sc=0,keys={},enemies=[],bullets=[],parts=[];

  function shoot(){
    var dx=mx-px,dy=my-py,len=M.sqrt(dx*dx+dy*dy);
    if(len===0)return;
    bullets.push({x:px,y:py,vx:dx/len*10,vy:dy/len*10,r:5});
  }

  win.onmousedown=e=>{if(e.target===c&&e.button===0)shoot()};
  win.onkeydown=e=>{keys[e.code]=true;if(e.code==='Space')shoot()};
  win.onkeyup=e=>keys[e.code]=false;
  win.onmousemove=e=>{mx=e.offsetX;my=e.offsetY};

  function spawn(){enemies.push({x:M.random()*W,y:-40,r:25+M.random()*20,s:0.3+M.random()*0.4});}
  spawn();spawn();

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
      var e=enemies[i];e.y+=e.s;
      if(e.y>H+50){enemies.splice(i,1);spawn();continue}
      x.beginPath();x.arc(e.x,e.y,e.r,0,7);x.fillStyle='#f22';x.fill();
      x.shadowColor='#f00';x.shadowBlur=12;x.fill();x.shadowBlur=0;
      if(M.hypot(px-e.x,py-e.y)<e.r+15){
        hp--;if(hp<=0){alert('游戏结束！得分:'+sc);location.reload()}
        enemies.splice(i,1);spawn();
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
          sc++;for(var p=0;p<8;p++)parts.push({x:e.x,y:e.y,vx:(M.random()-.5)*6,vy:(M.random()-.5)*6,l:15});
          enemies.splice(j,1);spawn();
          if(sc%50===0&&hp<5)hp++;
          hit=true;break;
        }
      }
      if(hit||b.x<0||b.x>W||b.y<0||b.y>H)bullets.splice(i,1);
    }

    // 粒子
    for(var i=parts.length-1;i>=0;i--){var p=parts[i];p.x+=p.vx;p.y+=p.vy;p.l--;x.globalAlpha=p.l/15;x.fillStyle='#ff0';x.fillRect(p.x,p.y,3,3);x.globalAlpha=1;if(p.l<=0)parts.splice(i,1);}

    // HUD
    x.fillStyle='#0f6';x.font='16px monospace';x.fillText(`❤️${hp} 🎯${sc}`,10,30);
  }
  loop();
}

// 修改 openApp 函数，加入 game 分支
var _openApp=openApp;
window.openApp=function(id){
  if(id==='game'){
    if(wins[id]){show(id);return}
    var win=createWindow('game','射击游戏',`<canvas style="width:100%;height:100%;display:block"></canvas>`);
    win.style.width='700px';win.style.height='500px';win.style.display='flex';
    setTimeout(()=>initGame(win),50);
    return;
  }
  _openApp(id);
}
<script>
var z=10,wins={};

function openApp(id){
  if(wins[id]){show(id);return}
  var cfg={terminal:'终端',about:'关于'}[id],html='';
  if(id==='terminal')html=`<div class="terminal" id="tout">MicroOS Terminal v1.3\n输入 help 查看命令\n\n$</div><div class="term-input"><span>$</span><input id="tin" autocomplete="off"></div>`;
  if(id==='about')html=`<div style="color:#0f6;line-height:1.8"><b>MicroOS v1.3</b><br>纯前端仿操作系统<br>零依赖 · 单文件<br><br>终端彩蛋：输入 secret</div>`;
  var w=document.createElement('div');w.className='window';
  w.innerHTML=`<div class="win-title"><span>${cfg}</span><span class="win-close" onclick="closeApp('${id}')">×</span></div><div class="win-content">${html}</div>`;
  w.style.left=80+Object.keys(wins).length*40+'px';
  w.style.top=60+Object.keys(wins).length*40+'px';
  w.style.zIndex=++z;document.body.appendChild(w);wins[id]={el:w,title:cfg};
  var b=document.createElement('div');b.className='task-btn active';b.textContent=cfg;b.dataset.id=id;
  b.onclick=()=>{wins[id].el.style.display==='none'?show(id):closeApp(id)};
  document.getElementById('taskbar').appendChild(b);
  show(id);
  if(id==='terminal')initTerm();
}

function show(id){wins[id].el.style.display='flex';wins[id].el.style.zIndex=++z;updateBtns()}
function closeApp(id){if(wins[id]){wins[id].el.style.display='none';updateBtns()}}
function updateBtns(){document.querySelectorAll('.task-btn').forEach(function(b){var id=b.dataset.id;b.classList.toggle('active',wins[id]&&wins[id].el.style.display!=='none')})}

function initTerm(){
  var out=document.getElementById('tout'),inp=document.getElementById('tin');
  inp.focus();
  inp.onkeydown=function(e){
    if(e.key!=='Enter')return;
    var cmd=inp.value.trim().toLowerCase();out.innerHTML+=cmd+'\n';inp.value='';
    var r='';
    if(cmd==='help')r='help   - 帮助\nclear  - 清屏\ntime   - 当前时间\nabout  - 关于系统\nsecret - 彩蛋';
    else if(cmd==='clear')r='__CLEAR__';
    else if(cmd==='time')r=new Date().toLocaleString();
    else if(cmd==='about'){openApp('about');r='打开关于...'}
    else if(cmd==='secret')r='🎉 致敬速通玩家！';
    else if(cmd==='')r='';
    else r='未知命令，输入 help';
    if(r==='__CLEAR__')out.innerHTML='';else out.innerHTML+=r+'\n\n$';
    out.scrollTop=out.scrollHeight;inp.focus();
  };
  wins.terminal.el.onclick=()=>inp.focus();
}
</script>
</body>
</html>
