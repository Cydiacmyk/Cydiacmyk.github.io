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
</div>

<div id=taskbar></div>

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
