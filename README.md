# Cydiacmyk.github.io
<!DOCTYPE html>
<html lang="zh">
<head>
<meta charset="UTF-8">
<title>Bouncing Green Ball</title>
<style>
  html, body { margin: 0; padding: 0; overflow: hidden; background: #111; }
  canvas { display: block; }
</style>
</head>
<body>
<canvas id="c"></canvas>
<script>
  const canvas = document.getElementById('c');
  const ctx = canvas.getContext('2d');

  // 自适应窗口尺寸
  function resize() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
  }
  resize();
  window.addEventListener('resize', resize);

  // 球的属性
  const ball = {
    x: 200,
    y: 200,
    radius: 20,
    dx: 4,   // x方向速度
    dy: 3,   // y方向速度
    color: '#00ff66'
  };

  function draw() {
    // 清屏
    ctx.fillStyle = '#111';
    ctx.fillRect(0, 0, canvas.width, canvas.height);

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
