# onbekend43-hub.github.io
<!DOCTYPE html>
<html>
<head>
    <title>Mijn Website</title>
    <style>
        body { font-family: Arial; background:#f0f0f0; }
        h1 { color: #333; }
    </style>
</head>
<body>
    <h1>Welkom op mijn website!</h1>
    <p>Deze website heb ik helemaal zelf gemaakt.</p>
</body>
</html>
<!--
Flappy Bird - single-file HTML game
Save as: index.html
Open in your browser (double-click) or host on GitHub Pages / Netlify.
Controls: SPACE or click/tap to flap. Press R or click after Game Over to restart.
-->

<!doctype html>
<html lang="nl">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Flappy — eenvoudige variant</title>
  <style>
    /* Simpele, schone styling */
    :root{--bg:#70c5ce;--ground:#ded895;--pipe:#2d8f57;--bird:#ffcc00}
    html,body{height:100%;margin:0;font-family:Inter, system-ui, Arial}
    body{display:flex;align-items:center;justify-content:center;background:linear-gradient(#9fe3ec, var(--bg));}
    .wrap{width:100%;max-width:520px;padding:16px;box-sizing:border-box}
    canvas{width:100%;height:auto;border-radius:12px;box-shadow:0 8px 30px rgba(10,20,30,0.15);background:transparent;display:block}
    .info{display:flex;justify-content:space-between;align-items:center;margin-top:8px;color:#123;}
    .btn{background:#fff;padding:6px 10px;border-radius:8px;cursor:pointer;border:1px solid rgba(0,0,0,0.08)}
    .small{font-size:13px;color:#034}
  </style>
</head>
<body>
  <div class="wrap">
    <canvas id="game"></canvas>
    <div class="info">
      <div class="small">Score: <span id="score">0</span> — Highscore: <span id="high">0</span></div>
      <div class="small">Druk <strong>SPACE</strong> of tik om te flapperen</div>
    </div>
  </div>

<script>
// Eenvoudige FlappyBird-variant
// Alles in één bestand — geen externe assets nodig.
(() => {
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');

  // Responsieve canvas (keeps ratio)
  const GAME_W = 400; // logical size
  const GAME_H = 600;
  function resizeCanvas(){
    const maxW = Math.min(window.innerWidth - 40, 520);
    const scale = maxW / GAME_W;
    canvas.width = GAME_W;
    canvas.height = GAME_H;
    canvas.style.width = (GAME_W * scale) + 'px';
    canvas.style.height = (GAME_H * scale) + 'px';
  }
  window.addEventListener('resize', resizeCanvas);
  resizeCanvas();

  // Game state
  let frames = 0;
  let pipes = [];
  let score = 0;
  let high = Number(localStorage.getItem('flappy_high') || 0);
  const scoreEl = document.getElementById('score');
  const highEl = document.getElementById('high');
  highEl.textContent = high;

  // Bird
  const bird = {
    x: 80,
    y: 250,
    w: 34,
    h: 24,
    dy: 0,
    gravity: 0.55,
    jump: -9.5,
    rotation: 0,
    alive: true
  };

  // Pipes
  const PIPE_WIDTH = 60;
  const GAP = 150;
  const PIPE_SPEED = 2.2;
  const SPAWN_EVERY = 100; // frames

  function spawnPipe(){
    // random top pipe bottom Y
    const padding = 40;
    const topHeight = padding + Math.random() * (GAME_H - GAP - padding*2 - 120);
    pipes.push({x: GAME_W + 20, top: topHeight, passed: false});
  }

  // Input
  function flap(){
    if (!bird.alive) return; // no flapping after death
    bird.dy = bird.jump;
  }
  window.addEventListener('keydown', e => {
    if (e.code === 'Space'){
      e.preventDefault();
      if (!bird.alive){ reset(); } else flap();
    }
    if (e.key.toLowerCase() === 'r') reset();
  });
  window.addEventListener('mousedown', e => {
    if (!bird.alive){ reset(); } else flap();
  });
  window.addEventListener('touchstart', e => {
    e.preventDefault();
    if (!bird.alive){ reset(); } else flap();
  }, {passive:false});

  // Physics & collision
  function update(){
    frames++;

    // Bird physics
    bird.dy += bird.gravity;
    bird.y += bird.dy;
    bird.rotation = Math.max(-0.6, Math.min(1.0, bird.dy / 10));

    // Spawn pipes
    if (frames % SPAWN_EVERY === 0) spawnPipe();

    // Move pipes and check collisions
    for (let i = pipes.length - 1; i >= 0; i--){
      const p = pipes[i];
      p.x -= PIPE_SPEED;

      // scoring — when bird passes pipe
      if (!p.passed && p.x + PIPE_WIDTH < bird.x){
        p.passed = true;
        score++;
        scoreEl.textContent = score;
      }

      // collision check with rectangles
      const birdBox = {left: bird.x - bird.w/2, right: bird.x + bird.w/2, top: bird.y - bird.h/2, bottom: bird.y + bird.h/2};
      // top pipe rect
      const topRect = {left: p.x, right: p.x + PIPE_WIDTH, top: 0, bottom: p.top};
      const bottomRect = {left: p.x, right: p.x + PIPE_WIDTH, top: p.top + GAP, bottom: GAME_H - 80};

      if (rectIntersect(birdBox, topRect) || rectIntersect(birdBox, bottomRect)){
        die();
      }

      // remove offscreen pipes
      if (p.x + PIPE_WIDTH < -50) pipes.splice(i,1);
    }

    // ground collision
    if (bird.y + bird.h/2 >= GAME_H - 80){
      bird.y = GAME_H - 80 - bird.h/2;
      die();
    }
    // ceiling
    if (bird.y - bird.h/2 <= 0){
      bird.y = bird.h/2;
      bird.dy = 0;
    }
  }

  function rectIntersect(a,b){
    return !(a.right < b.left || a.left > b.right || a.bottom < b.top || a.top > b.bottom);
  }

  function die(){
    if (!bird.alive) return;
    bird.alive = false;
    // update highscore
    if (score > high){
      high = score;
      localStorage.setItem('flappy_high', high);
      highEl.textContent = high;
    }
  }

  function reset(){
    frames = 0; pipes = [];
    score = 0; scoreEl.textContent = 0;
    bird.y = 250; bird.dy = 0; bird.alive = true;
  }

  // Draw
  function draw(){
    // clear
    ctx.clearRect(0,0,canvas.width,canvas.height);

    // background sky (gradient)
    const g = ctx.createLinearGradient(0,0,0,canvas.height);
    g.addColorStop(0,'#9fe3ec');
    g.addColorStop(1,'#70c5ce');
    ctx.fillStyle = g;
    ctx.fillRect(0,0,canvas.width,canvas.height);

    // draw pipes
    for (const p of pipes){
      // top
      drawRoundRect(ctx, p.x, 0, PIPE_WIDTH, p.top, 6, '#2d8f57');
      // bottom
      drawRoundRect(ctx, p.x, p.top + GAP, PIPE_WIDTH, canvas.height - (p.top + GAP) - 80, 6, '#2d8f57');
      // pipe caps
      ctx.fillStyle = '#1f6b3f';
      ctx.fillRect(p.x - 6, p.top - 12, PIPE_WIDTH + 12, 12);
      ctx.fillRect(p.x - 6, p.top + GAP, PIPE_WIDTH + 12, 12);
    }

    // ground
    ctx.fillStyle = '#ded895';
    ctx.fillRect(0, canvas.height - 80, canvas.width, 80);
    // ground pattern (simple)
    ctx.fillStyle = 'rgba(0,0,0,0.03)';
    for (let x = 0; x < canvas.width; x += 16){ ctx.fillRect(x, canvas.height - 80, 8, 80); }

    // draw bird (circle + eye) with rotation
    ctx.save();
    ctx.translate(bird.x, bird.y);
    ctx.rotate(bird.rotation);
    // body
    ctx.fillStyle = '#ffcc00';
    roundRect(ctx, -bird.w/2, -bird.h/2, bird.w, bird.h, 6, true, false);
    // beak
    ctx.fillStyle = '#ff8a00';
    ctx.beginPath(); ctx.moveTo(bird.w/2 - 2, 0); ctx.lineTo(bird.w/2 + 12, -6); ctx.lineTo(bird.w/2 + 12, 6); ctx.closePath(); ctx.fill();
    // eye
    ctx.fillStyle = '#222'; ctx.beginPath(); ctx.arc(-2, -4, 3, 0, Math.PI*2); ctx.fill();
    ctx.restore();

    // overlay text when dead
    if (!bird.alive){
      ctx.fillStyle = 'rgba(0,0,0,0.5)';
      ctx.fillRect(0,0,canvas.width,canvas.height);
      ctx.fillStyle = '#fff'; ctx.textAlign = 'center';
      ctx.font = 'bold 32px Arial'; ctx.fillText('Game Over', canvas.width/2, canvas.height/2 - 20);
      ctx.font = '18px Arial'; ctx.fillText('Klik of druk SPACE om opnieuw te beginnen', canvas.width/2, canvas.height/2 + 18);
    }
  }

  // Utilities for drawing rounded rects (pixel-perfect)
  function roundRect(ctx, x, y, w, h, r, fill, stroke){
    if (typeof r === 'undefined') r = 5;
    ctx.beginPath();
    ctx.moveTo(x + r, y);
    ctx.arcTo(x + w, y, x + w, y + h, r);
    ctx.arcTo(x + w, y + h, x, y + h, r);
    ctx.arcTo(x, y + h, x, y, r);
    ctx.arcTo(x, y, x + w, y, r);
    ctx.closePath();
    if (fill) ctx.fill();
    if (stroke) ctx.stroke();
  }
  function drawRoundRect(ctx,x,y,w,h,r,fillStyle){ ctx.fillStyle = fillStyle; roundRect(ctx,x,y,w,h,r,true,false); }

  // game loop
  function loop(){
    if (bird.alive) update();
    draw();
    requestAnimationFrame(loop);
  }

  // start
  loop();

})();
</script>
</body>
</html>
