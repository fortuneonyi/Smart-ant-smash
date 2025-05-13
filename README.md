<!DOCTYPE html>
<html lang="en">
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Ant Smash</title>
  <style>
    html, body {
      margin: 0;
      padding: 0;
      overflow: hidden;
      touch-action: manipulation;
    }
    canvas {
      display: block;
      background: linear-gradient(to bottom right, #e0f7fa, #ffe0b2);
    }
  </style>
</head>
<body>
<canvas id="gameCanvas"></canvas>
<script>
  const canvas = document.getElementById("gameCanvas");
  const ctx = canvas.getContext("2d");

  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;

  let ants = [];
  let particles = [];
  let colors = ["#f44336", "#4caf50", "#2196f3", "#ffeb3b", "#e91e63"];
  let bgColorIndex = 0;

  class Ant {
    constructor(x, y, isBoss = false) {
      this.x = x;
      this.y = y;
      this.size = isBoss ? 60 : 30;
      this.speed = isBoss ? 0.5 : 1 + Math.random();
      this.color = isBoss ? "black" : "#795548";
      this.boss = isBoss;
      this.direction = Math.random() < 0.5 ? -1 : 1;
    }

    draw() {
      ctx.beginPath();
      ctx.arc(this.x, this.y, this.size / 2, 0, Math.PI * 2);
      ctx.fillStyle = this.color;
      ctx.fill();
    }

    update() {
      this.y += this.speed;
      this.x += this.direction * 0.5;
      if (this.y > canvas.height + this.size) {
        this.respawn();
      }
      this.draw();
    }

    respawn() {
      this.x = Math.random() * canvas.width;
      this.y = -this.size;
      this.speed = this.boss ? 0.5 : 1 + Math.random();
    }
  }

  class Particle {
    constructor(x, y, color) {
      this.x = x;
      this.y = y;
      this.radius = Math.random() * 5 + 2;
      this.color = color;
      this.vx = Math.random() * 4 - 2;
      this.vy = Math.random() * -4;
      this.alpha = 1;
    }

    update() {
      this.x += this.vx;
      this.y += this.vy;
      this.vy += 0.1;
      this.alpha -= 0.02;
    }

    draw() {
      ctx.globalAlpha = this.alpha;
      ctx.beginPath();
      ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
      ctx.fillStyle = this.color;
      ctx.fill();
      ctx.globalAlpha = 1;
    }
  }

  function createAnts(count) {
    for (let i = 0; i < count; i++) {
      const isBoss = i % 10 === 0;
      ants.push(new Ant(Math.random() * canvas.width, Math.random() * -canvas.height, isBoss));
    }
  }

  function createParticles(x, y, color) {
    for (let i = 0; i < 10; i++) {
      particles.push(new Particle(x, y, color));
    }
  }

  canvas.addEventListener("touchstart", function(e) {
    const touch = e.touches[0];
    const x = touch.clientX;
    const y = touch.clientY;

    for (let i = ants.length - 1; i >= 0; i--) {
      let ant = ants[i];
      let dx = x - ant.x;
      let dy = y - ant.y;
      let distance = Math.sqrt(dx * dx + dy * dy);
      if (distance < ant.size / 2) {
        createParticles(ant.x, ant.y, ant.color);
        ants.splice(i, 1);
        spawnAnt();
        break;
      }
    }
  });

  function spawnAnt() {
    const isBoss = Math.random() < 0.1;
    ants.push(new Ant(Math.random() * canvas.width, -30, isBoss));
  }

  function animate() {
    ctx.fillStyle = colors[bgColorIndex % colors.length];
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    ants.forEach(ant => ant.update());

    for (let i = particles.length - 1; i >= 0; i--) {
      particles[i].update();
      particles[i].draw();
      if (particles[i].alpha <= 0) {
        particles.splice(i, 1);
      }
    }

    requestAnimationFrame(animate);
  }

  function cycleBgColor() {
    bgColorIndex++;
    setTimeout(cycleBgColor, 3000);
  }

  createAnts(10);
  animate();
  cycleBgColor();

  window.addEventListener('resize', () => {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
  });
</script>
</body>
</html>
