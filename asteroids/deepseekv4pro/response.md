• I'll build a classic Asteroids game from scratch — single HTML file, canvas-based, with arrow-key controls, space to shoot, particle effects, and a starfield background.

• - Triangle shape
  - Rotates left/right with arrow keys
  - Thrusts forward with up arrow
  - Momentum/inertia based movement
  - Screen wrapping

  Asteroids:

  - Irregular polygon shapes
  - Float across screen with random velocities
  - Come in different sizes (large, medium, small)
  - When shot, larger asteroids split into smaller ones
  - Screen wrapping

  Bullets:

  - Fired from ship's nose in the direction the ship is facing
  - Have a limited lifetime/distance
  - Screen wrapping

  Game Loop:

  - requestAnimationFrame based
  - Collision detection between bullets and asteroids
  - Collision detection between ship and asteroids (player death)
  - Score tracking
  - Lives system
  - Wave system (all asteroids destroyed = new wave with more asteroids)

  Let me implement this. I'll make the visuals nice with some glow effects, starfield background, particle effects for explosions, etc. But keep the game logic clean and straightforward.

  Let me write the complete HTML file.

  I should create this in the current working directory /Users/csabatoth/repos/temp4.

  Let me think about the structure:

  <!DOCTYPE html>
  <html>
  <head>
      <title>Asteroids</title>
      <style>
          /* styles */
      </style>
  </head>
  <body>
      <canvas id="game"></canvas>
      <script>
          // game code
      </script>
  </body>
  </html>

  Let me think through the implementation carefully:

  Ship class:

  - x, y position
  - vx, vy velocity
  - angle (in radians)
  - rotation speed
  - thrust power
  - friction (or no friction in space - just inertia)
  - radius for collision
  - vertices for drawing (triangle)
  - invincibility frames after respawn

  Asteroid class:

  - x, y position
  - vx, vy velocity
  - size (large, medium, small) with different radii
  - vertices for irregular polygon shape
  - rotation
  - rotation speed

  Bullet class:

  - x, y position
  - vx, vy velocity (based on ship angle at firing time)
  - lifetime
  - radius

  Game state:

  - asteroids array
  - bullets array
  - ship
  - score
  - lives
  - level/wave
  - game over state
  - respawn timer

  Let me write this now. I want to make it look cool with:

  - Dark space background with stars
  - Glowing ship with engine trail
  - Glowing bullets
  - Particle explosion effects when asteroids are destroyed
  - Score display
  - Lives display
  - Maybe some subtle screen effects

  Let me implement this properly. I'll keep the code well-organized but all in one file.

  Actually, I should also think about:

  Collision detection:

  - Circle-based for ship vs asteroids (circle-circle)
  - Circle-based for bullets vs asteroids (circle-circle)

  Asteroid splitting:

  - Large asteroid → 2 medium asteroids
  - Medium asteroid → 2 small asteroids
  - Small asteroid → destroyed

  Scoring:

  - Large: 20 points
  - Medium: 50 points
  - Small: 100 points

  Lives: Start with 3, lose one on death, game over at 0.

  Wave progression: Start with 4-5 large asteroids, increase each wave.

  Let me also add some nice visual effects:

  - Starfield background (static random stars)
  - Ship has a subtle glow
  - Engine flame when thrusting
  - Particles on asteroid destruction
  - Screen shake on death (subtle)

  Now let me code this up. I'll be thorough.

  Actually, for the ship drawing, I'll use a nice triangle with some style. For asteroids, I'll generate irregular polygons. For bullets, small glowing dots.

  Let me write the full implementation now.

  One more thing - I need to think about the canvas sizing. I'll make it responsive - fill the window.

  Let me start coding...

  // Canvas setup
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');

  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;

  // Starfield
  const stars = [];
  for (let i = 0; i < 200; i++) {
      stars.push({
          x: Math.random() * canvas.width,
          y: Math.random() * canvas.height,
          size: Math.random() * 2 + 0.5,
          brightness: Math.random() * 0.5 + 0.5
      });
  }

  // Ship
  const ship = {
      x: canvas.width / 2,
      y: canvas.height / 2,
      vx: 0,
      vy: 0,
      angle: -Math.PI / 2, // pointing up
      radius: 20,
      thrusting: false,
      rotatingLeft: false,
      rotatingRight: false,
      invincible: 0,
      alive: true,
      respawnTimer: 0
  };

  // Arrays
  let asteroids = [];
  let bullets = [];
  let particles = [];

  // Game state
  let score = 0;
  let lives = 3;
  let wave = 1;
  let gameOver = false;
  let gameStarted = false;

  // Constants
  const ROTATION_SPEED = 0.08;
  const THRUST_POWER = 0.15;
  const FRICTION = 0.99;
  const BULLET_SPEED = 8;
  const BULLET_LIFETIME = 60;
  const MAX_BULLETS = 4;
  const INVINCIBLE_TIME = 120;
  const RESPAWN_TIME = 60;
  const INITIAL_ASTEROIDS = 4;

  Wait, I said no friction in space... but a tiny bit of friction makes the game more playable. Actually, in classic Asteroids there's no friction. Let me keep true to the original - no friction. The ship just
  drifts.

  Actually, let me keep a tiny bit of damping to make it more playable. The original arcade game had no friction but this is a web version. Let me use 0.995 or something very subtle.

  Let me also think about the ship's shape. Classic asteroids ship is a triangle pointing up. When the ship's angle is 0 (pointing right), the nose is at angle 0, and the two rear corners are at angles... let me
  think. If the ship points in the direction of its angle:

  - Nose at (angle) distance radius from center
  - Left rear at (angle + 140°) distance radius*0.7 from center
  - Right rear at (angle - 140°) distance radius*0.7 from center

  Wait, let me think again. If angle = -PI/2 (pointing up):

  - Nose should be at top: (x, y - radius)
  - Left rear at bottom-left: (x - radius0.7, y + radius0.5)
  - Right rear at bottom-right: (x + radius0.7, y + radius0.5)

  So the offset angles from the ship's direction:

  - Nose: 0° from ship angle, at full radius
  - Left rear: ~210° from ship angle (or 150° behind), at 0.7 radius
  - Right rear: ~-210° from ship angle, at 0.7 radius

  Hmm, let me just compute it. The classic ship is an isosceles triangle. The nose is at the front, the base is at the rear.

  So if ship.angle points in the direction the ship is facing:

  - nose: angle + 0, radius * 1.0
  - left: angle + Math.PI * 0.75 (or about 135°), at radius * 0.6
  - right: angle - Math.PI * 0.75, at radius * 0.6

  Actually, let me just make it:

  - Nose: direction of angle, full radius forward
  - Back-left: angle + 150° (in degrees), smaller radius
  - Back-right: angle - 150°, smaller radius

  Wait, I should just test it in my head. Let's say angle = 0 (pointing right in standard math coords, but on canvas y is flipped):

  Actually on canvas, y increases downward. So:

  - angle = 0 means pointing right
  - angle = -PI/2 means pointing up (since negative angles go counterclockwise in canvas)

  For the ship facing "up" initially (angle = -PI/2):

  - Nose at angle -PI/2: direction is (0, -1), so nose at (x, y - radius) ✓
  - Left rear at angle -PI/2 + PI * 0.8: roughly pointing down-left
  - Right rear at angle -PI/2 - PI * 0.8: roughly pointing down-right

  Let me use 140° offset for the rear vertices (which is about 2.44 radians). So:

  - nose: at ship direction angle
  - left rear: ship angle + 2.44 rad (≈140°)
  - right rear: ship angle - 2.44 rad

  For angle = -PI/2 (pointing up):

  - nose: (-PI/2) → (0, -1) → top
  - left rear: (-PI/2 + 2.44) ≈ 0.87 rad ≈ 50° → down-left-ish
  - right rear: (-PI/2 - 2.44) ≈ -4.01 rad → down-right-ish

  That seems right for a triangle pointing up.

  Now for thrust/flame effect, when thrusting I'll draw a flame at the rear between the two back vertices.

  OK let me just write the code now. I have a good mental model.

  Let me also think about collision detection. I'll use circle-based collision:

  - Ship radius: 18 (slightly less than visual for forgiveness)
  - Asteroid radius: depends on size (large: 50, medium: 25, small: 12)
  - Bullet radius: 3

  Collision detection: distance between centers < sum of radii.

  For wrapping, I'll check if an object goes off screen and wrap it to the other side. Actually, classic asteroids wraps at the edges. I'll do:

  if (obj.x < -obj.radius) obj.x = canvas.width + obj.radius;
  if (obj.x > canvas.width + obj.radius) obj.x = -obj.radius;
  if (obj.y < -obj.radius) obj.y = canvas.height + obj.radius;
  if (obj.y > canvas.height + obj.radius) obj.y = -obj.radius;

  OK I think I have enough to write a solid implementation. Let me also add:

  - A "press any key to start" or just start immediately
  - HUD with score and lives
  - Game over screen with final score and restart option
  - Particle effects for destroyed asteroids
  - Ship explosion particles on death

  Let me write it all out now:

  For asteroid polygon generation, I'll generate a random polygon by varying the radius at different angles around the asteroid's center:

  function generateAsteroidShape(size) {
      const vertices = [];
      const numVertices = size === 'large' ? 12 : size === 'medium' ? 9 : 7;
      const baseRadius = size === 'large' ? 50 : size === 'medium' ? 25 : 12;
      for (let i = 0; i < numVertices; i++) {
          const angle = (i / numVertices) * Math.PI * 2;
          const variance = 0.7 + Math.random() * 0.6; // 0.7 to 1.3
          vertices.push({
              angle: angle,
              dist: baseRadius * variance
          });
      }
      return vertices;
  }

  Actually, I should track the asteroid shape as part of the asteroid object so it's consistent each frame. Let me put the shape data in the asteroid.

  Let me also think about how to handle the game loop. I'll use requestAnimationFrame and calculate delta time for frame-rate independent movement... actually, for simplicity and classic feel, I'll just use fixed
  time step logic without delta time. Most simple games just assume 60fps and move things at fixed rates. Let me do that for simplicity.

  Actually, for better consistency, let me use a simple delta time. But keep it simple.

  Let me structure the code:

  1. Setup (canvas, stars, constants)
  2. Helper functions (wrap, distance, random range)
  3. Game object creation functions (create asteroid, create bullet, etc.)
  4. Update functions
  5. Draw functions
  6. Game loop
  7. Input handling
  8. Initialization

  Let me write it now. I want to make this really solid.

  Here's my plan for the file structure:

  asteroids.html

  That's it - single file. Let me write it.

  I should also consider mobile/tablet - but the user specified cursor keys and space, so desktop is the target. I'll still make the canvas responsive.

  One more consideration - the user said "make the visuals cool but don't complicate the game logic." So I'll add:

  - Gradient/glow effects on the ship and bullets
  - Starfield background with twinkling
  - Particle effects for explosions
  - Subtle screen flash on death
  - Neon/retro color scheme (maybe cyan/magenta/white on dark)

  But keep the logic straightforward.

  Let me start writing the code now. I'll be comprehensive.

  Actually, should I write this directly or use a plan? The user said to implement it. Let me just write it directly. But first let me create the file.

  I'll write the complete HTML file and open it in a browser for testing... well, I can't open a browser directly but I can write the file and let the user test it.

  Let me structure my approach:

  1. Create the HTML file
  2. Make sure it's complete and working

  Let me write the code now.

  One thing I need to be careful about: the canvas coordinate system. In canvas:

  - x increases to the right
  - y increases downward
  - Angles: 0 = right, -PI/2 = up, PI/2 = down, PI or -PI = left

  For the ship starting position pointing up, angle = -PI/2.

  For drawing the ship:

  ctx.save();
  ctx.translate(ship.x, ship.y);
  ctx.rotate(ship.angle);
  ctx.beginPath();
  ctx.moveTo(radius, 0); // nose
  ctx.lineTo(-radius * 0.7, -radius * 0.6); // left rear
  ctx.lineTo(-radius * 0.7, radius * 0.6); // right rear
  ctx.closePath();
  ctx.stroke();
  ctx.restore();

  Wait, let me reconsider. With ctx.rotate(ship.angle), the ship's forward direction will be along the positive x-axis (0 radians). So:

  - Nose at (radius, 0)
  - Left rear at (-radius0.6, -radius0.6)
  - Right rear at (-radius0.6, radius0.6)

  And when ship.angle = -PI/2, rotating by -PI/2 means the positive x-axis points up, which means the nose points up. Good.

  For thrust vector:

  - Direction of ship's facing = (cos(ship.angle), sin(ship.angle))
  - Apply thrust in this direction

  For bullet firing:

  - Bullet starts at ship's nose position
  - Bullet velocity = ship velocity + (cos(ship.angle) * BULLET_SPEED, sin(ship.angle) * BULLET_SPEED)

  OK, I'm confident in the math. Let me write the complete code now.

  Let me also think about what happens when the ship dies:

  1. Ship becomes not alive
  2. Spawn explosion particles
  3. After respawn timer, ship reappears at center
  4. Ship has invincibility for a few seconds (blinking effect)
  5. Must ensure center is clear of asteroids before respawning

  For the center-clear check, I'll check if any asteroids are within a safe zone (150px radius) of the center. If not clear, wait a few more frames.

  Let me code it all now. I'll write it as clean, well-organized JavaScript.

  Actually, I realize I should just write the file. Let me compose it all at once and write it.

  Here's the full implementation I'll write:

  <!DOCTYPE html>
  <html lang="en">
  <head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Asteroids</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      background: #000;
      overflow: hidden;
      font-family: 'Courier New', monospace;
      cursor: none;
    }
    canvas { display: block; }
    #hud {
      position: fixed;
      top: 20px;
      left: 20px;
      color: #fff;
      font-size: 18px;
      pointer-events: none;
      text-shadow: 0 0 10px rgba(255,255,255,0.5);
    }
    #hud .score { font-size: 28px; font-weight: bold; }
    #hud .lives { margin-top: 8px; }
    #message {
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      color: #fff;
      text-align: center;
      pointer-events: none;
      text-shadow: 0 0 20px rgba(255,255,255,0.7);
    }
    #message h1 { font-size: 48px; margin-bottom: 16px; }
    #message p { font-size: 20px; opacity: 0.8; }
  </style>
  </head>
  <body>
  <canvas id="game"></canvas>
  <div id="hud">
    <div class="score" id="scoreDisplay">0</div>
    <div class="lives" id="livesDisplay"></div>
  </div>
  <div id="message"></div>

  <script>
  (function() {
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');
    const scoreDisplay = document.getElementById('scoreDisplay');
    const livesDisplay = document.getElementById('livesDisplay');
    const messageDiv = document.getElementById('message');

    // --- Canvas sizing ---
    function resize() {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
    }
    window.addEventListener('resize', resize);
    resize();

    // --- Constants ---
    const ROTATION_SPEED = 0.07;
    const THRUST_POWER = 0.12;
    const BULLET_SPEED = 7;
    const BULLET_LIFETIME = 70;
    const MAX_BULLETS = 4;
    const INVINCIBLE_FRAMES = 150;
    const RESPAWN_DELAY = 90;
    const SAFE_ZONE = 150;

    const ASTEROID_SIZES = {
      large: { radius: 45, score: 20, vertices: 12, minVar: 0.65, maxVar: 1.0 },
      medium: { radius: 22, score: 50, vertices: 9, minVar: 0.7, maxVar: 1.0 },
      small: { radius: 11, score: 100, vertices: 7, minVar: 0.75, maxVar: 1.0 }
    };

    // --- Starfield ---
    const stars = [];
    for (let i = 0; i < 250; i++) {
      stars.push({
        x: Math.random() * canvas.width,
        y: Math.random() * canvas.height,
        r: Math.random() * 1.8 + 0.3,
        twinkle: Math.random() * Math.PI * 2,
        speed: Math.random() * 0.03 + 0.005
      });
    }

    // --- Game state ---
    let ship, asteroids, bullets, particles;
    let score, lives, wave, gameOver, gameStarted;
    let screenShake = 0;

    // --- Keys ---
    const keys = {};
    window.addEventListener('keydown', e => {
      keys[e.code] = true;
      if (!gameStarted && !gameOver) {
        startGame();
      }
      if (gameOver && e.code === 'Space') {
        startGame();
      }
      e.preventDefault();
    });
    window.addEventListener('keyup', e => {
      keys[e.code] = false;
      e.preventDefault();
    });

    // --- Helper functions ---
    function wrap(obj, margin = 0) {
      if (obj.x < -margin) obj.x = canvas.width + margin;
      if (obj.x > canvas.width + margin) obj.x = -margin;
      if (obj.y < -margin) obj.y = canvas.height + margin;
      if (obj.y > canvas.height + margin) obj.y = -margin;
    }

    function dist(a, b) {
      const dx = a.x - b.x;
      const dy = a.y - b.y;
      return Math.sqrt(dx * dx + dy * dy);
    }

    function randRange(min, max) {
      return Math.random() * (max - min) + min;
    }

    // --- Particle system ---
    function spawnParticles(x, y, count, color, speed = 3, life = 40) {
      for (let i = 0; i < count; i++) {
        const angle = Math.random() * Math.PI * 2;
        const spd = Math.random() * speed + speed * 0.3;
        particles.push({
          x, y,
          vx: Math.cos(angle) * spd,
          vy: Math.sin(angle) * spd,
          life: life + Math.random() * 20,
          maxLife: life + 20,
          color,
          size: Math.random() * 2.5 + 1
        });
      }
    }

    // --- Asteroid creation ---
    function createAsteroid(x, y, size, overrideVx, overrideVy) {
      const cfg = ASTEROID_SIZES[size];
      const shape = [];
      for (let i = 0; i < cfg.vertices; i++) {
        const angle = (i / cfg.vertices) * Math.PI * 2;
        shape.push({
          angle,
          dist: cfg.radius * randRange(cfg.minVar, cfg.maxVar)
        });
      }
      // Give it some random rotation
      const rotSpeed = randRange(-0.03, 0.03);
      const angle = Math.random() * Math.PI * 2;

      let vx, vy;
      if (overrideVx !== undefined) {
        vx = overrideVx;
        vy = overrideVy;
      } else {
        const speed = randRange(0.5, 2.0);
        const dir = Math.random() * Math.PI * 2;
        vx = Math.cos(dir) * speed;
        vy = Math.sin(dir) * speed;
      }

      return { x, y, vx, vy, size, radius: cfg.radius, score: cfg.score, shape, rotation: angle, rotSpeed };
    }

    function splitAsteroid(ast) {
      if (ast.size === 'large') return 'medium';
      if (ast.size === 'medium') return 'small';
      return null;
    }

    // --- Bullet creation ---
    function fireBullet() {
      if (bullets.length >= MAX_BULLETS || !ship.alive) return;
      const noseX = ship.x + Math.cos(ship.angle) * ship.radius;
      const noseY = ship.y + Math.sin(ship.angle) * ship.radius;
      bullets.push({
        x: noseX,
        y: noseY,
        vx: ship.vx + Math.cos(ship.angle) * BULLET_SPEED,
        vy: ship.vy + Math.sin(ship.angle) * BULLET_SPEED,
        life: BULLET_LIFETIME,
        radius: 2.5
      });
    }

    // --- Initialize / Reset ---
    function startGame() {
      ship = {
        x: canvas.width / 2,
        y: canvas.height / 2,
        vx: 0,
        vy: 0,
        angle: -Math.PI / 2,
        radius: 18,
        alive: true,
        invincible: INVINCIBLE_FRAMES,
        thrusting: false
      };
      asteroids = [];
      bullets = [];
      particles = [];
      score = 0;
      lives = 3;
      wave = 1;
      gameOver = false;
      gameStarted = true;
      screenShake = 0;

      spawnWave();
      updateHUD();
      messageDiv.innerHTML = '';
    }

    function spawnWave() {
      const count = wave + 3; // 4, 5, 6, ...
      for (let i = 0; i < count; i++) {
        // Spawn at edges, away from center
        let x, y;
        const edge = Math.floor(Math.random() * 4);
        switch (edge) {
          case 0: x = -40; y = Math.random() * canvas.height; break;
          case 1: x = canvas.width + 40; y = Math.random() * canvas.height; break;
          case 2: x = Math.random() * canvas.width; y = -40; break;
          case 3: x = Math.random() * canvas.width; y = canvas.height + 40; break;
        }
        asteroids.push(createAsteroid(x, y, 'large'));
      }
    }

    function updateHUD() {
      scoreDisplay.textContent = score.toString().padStart(6, '0');
      let livesStr = '';
      for (let i = 0; i < lives; i++) {
        livesStr += '▲ ';
      }
      livesDisplay.textContent = livesStr.trim();
    }

    function killShip() {
      if (ship.invincible > 0) return;
      spawnParticles(ship.x, ship.y, 25, '#ff4444', 5, 50);
      spawnParticles(ship.x, ship.y, 15, '#ffaa00', 3, 40);
      spawnParticles(ship.x, ship.y, 10, '#ffffff', 2, 30);
      screenShake = 20;
      ship.alive = false;
      ship.respawnTimer = RESPAWN_DELAY;
      lives--;
      updateHUD();

      if (lives <= 0) {
        gameOver = true;
        gameStarted = false;
        setTimeout(() => {
          messageDiv.innerHTML = '<h1>GAME OVER</h1><p>Final Score: ' + score + '</p><p>Press SPACE to restart</p>';
        }, 500);
      }
    }

    function respawnShip() {
      // Check if center is clear
      const cx = canvas.width / 2;
      const cy = canvas.height / 2;
      for (const ast of asteroids) {
        if (dist({x: cx, y: cy}, ast) < SAFE_ZONE + ast.radius) {
          return false; // not clear yet
        }
      }
      ship.x = cx;
      ship.y = cy;
      ship.vx = 0;
      ship.vy = 0;
      ship.angle = -Math.PI / 2;
      ship.alive = true;
      ship.invincible = INVINCIBLE_FRAMES;
      return true;
    }

    // --- Update ---
    function update() {
      if (!gameStarted || gameOver) return;

      // Screen shake decay
      if (screenShake > 0) screenShake *= 0.85;
      if (screenShake < 0.5) screenShake = 0;

      // Update ship
      if (ship.alive) {
        // Rotation
        if (keys['ArrowLeft']) ship.angle -= ROTATION_SPEED;
        if (keys['ArrowRight']) ship.angle += ROTATION_SPEED;

        // Thrust
        ship.thrusting = keys['ArrowUp'];
        if (ship.thrusting) {
          ship.vx += Math.cos(ship.angle) * THRUST_POWER;
          ship.vy += Math.sin(ship.angle) * THRUST_POWER;

          // Spawn engine particles
          if (Math.random() < 0.6) {
            const rearX = ship.x - Math.cos(ship.angle) * ship.radius * 0.8;
            const rearY = ship.y - Math.sin(ship.angle) * ship.radius * 0.8;
            const spreadAngle = ship.angle + Math.PI + randRange(-0.3, 0.3);
            particles.push({
              x: rearX,
              y: rearY,
              vx: Math.cos(spreadAngle) * randRange(1, 3),
              vy: Math.sin(spreadAngle) * randRange(1, 3),
              life: randRange(10, 25),
              maxLife: 25,
              color: Math.random() < 0.5 ? '#ff6600' : '#ffaa00',
              size: randRange(1, 3)
            });
          }
        }

        // Move ship
        ship.x += ship.vx;
        ship.y += ship.vy;
        wrap(ship, ship.radius);

        // Invincibility countdown
        if (ship.invincible > 0) ship.invincible--;
      } else {
        // Respawn timer
        ship.respawnTimer--;
        if (ship.respawnTimer <= 0) {
          respawnShip();
        }
      }

      // Shooting
      if (keys['Space'] && ship.alive) {
        if (!ship._lastShot || ship._lastShot <= 0) {
          fireBullet();
          ship._lastShot = 15; // cooldown frames
        }
      }
      if (ship._lastShot !== undefined) ship._lastShot--;

      // Update bullets
      for (let i = bullets.length - 1; i >= 0; i--) {
        const b = bullets[i];
        b.x += b.vx;
        b.y += b.vy;
        b.life--;
        wrap(b, b.radius);
        if (b.life <= 0) {
          bullets.splice(i, 1);
        }
      }

      // Update asteroids
      for (const ast of asteroids) {
        ast.x += ast.vx;
        ast.y += ast.vy;
        ast.rotation += ast.rotSpeed;
        wrap(ast, ast.radius);
      }

      // Update particles
      for (let i = particles.length - 1; i >= 0; i--) {
        const p = particles[i];
        p.x += p.vx;
        p.y += p.vy;
        p.life--;
        if (p.life <= 0) {
          particles.splice(i, 1);
        }
      }

      // Collision: bullets vs asteroids
      for (let bi = bullets.length - 1; bi >= 0; bi--) {
        const b = bullets[bi];
        let hit = false;
        for (let ai = asteroids.length - 1; ai >= 0; ai--) {
          const ast = asteroids[ai];
          if (dist(b, ast) < ast.radius + b.radius) {
            // Hit!
            score += ast.score;
            spawnParticles(ast.x, ast.y, ast.size === 'large' ? 15 : ast.size === 'medium' ? 10 : 6,
                           '#ffffff', ast.size === 'large' ? 4 : 3, 30);

            const nextSize = splitAsteroid(ast);
            if (nextSize) {
              // Spawn two smaller asteroids with diverging velocities
              const baseAngle = Math.atan2(b.vy, b.vx);
              for (let s = 0; s < 2; s++) {
                const spreadAngle = baseAngle + (s === 0 ? -0.5 : 0.5) + randRange(-0.3, 0.3);
                const speed = randRange(1.5, 3.5);
                asteroids.push(createAsteroid(
                  ast.x, ast.y, nextSize,
                  Math.cos(spreadAngle) * speed,
                  Math.sin(spreadAngle) * speed
                ));
              }
            }

            asteroids.splice(ai, 1);
            bullets.splice(bi, 1);
            hit = true;
            updateHUD();
            break;
          }
        }

        // Check if all asteroids destroyed
        if (asteroids.length === 0 && !gameOver) {
          wave++;
          spawnWave();
        }
      }

      // Collision: ship vs asteroids
      if (ship.alive && ship.invincible <= 0) {
        for (const ast of asteroids) {
          if (dist(ship, ast) < ast.radius + ship.radius * 0.7) {
            killShip();
            break;
          }
        }
      }
    }

    // --- Draw ---
    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // Apply screen shake
      let shakeX = 0, shakeY = 0;
      if (screenShake > 0) {
        shakeX = (Math.random() - 0.5) * screenShake;
        shakeY = (Math.random() - 0.5) * screenShake;
      }

      ctx.save();
      ctx.translate(shakeX, shakeY);

      // Draw stars (twinkling)
      for (const s of stars) {
        s.twinkle += s.speed;
        const alpha = 0.4 + Math.sin(s.twinkle) * 0.3 + 0.3;
        ctx.fillStyle = `rgba(255, 255, 255, ${alpha})`;
        ctx.beginPath();
        ctx.arc(s.x, s.y, s.r, 0, Math.PI * 2);
        ctx.fill();
      }

      // Draw particles
      for (const p of particles) {
        const alpha = p.life / p.maxLife;
        ctx.fillStyle = p.color.replace(')', `, ${alpha})`).replace('rgb', 'rgba');
        // handle hex colors
        let color;
        if (p.color.startsWith('#')) {
          const r = parseInt(p.color.slice(1,3), 16);
          const g = parseInt(p.color.slice(3,5), 16);
          const b = parseInt(p.color.slice(5,7), 16);
          color = `rgba(${r}, ${g}, ${b}, ${alpha})`;
        } else {
          color = p.color;
        }
        ctx.fillStyle = color;
        ctx.beginPath();
        ctx.arc(p.x, p.y, p.size * alpha, 0, Math.PI * 2);
        ctx.fill();
      }

      // Draw asteroids
      for (const ast of asteroids) {
        ctx.save();
        ctx.translate(ast.x, ast.y);
        ctx.rotate(ast.rotation);

        // Asteroid body
        ctx.strokeStyle = '#aaa';
        ctx.lineWidth = 1.5;
        ctx.beginPath();
        const shape = ast.shape;
        ctx.moveTo(
          Math.cos(shape[0].angle) * shape[0].dist,
          Math.sin(shape[0].angle) * shape[0].dist
        );
        for (let i = 1; i < shape.length; i++) {
          ctx.lineTo(
            Math.cos(shape[i].angle) * shape[i].dist,
            Math.sin(shape[i].angle) * shape[i].dist
          );
        }
        ctx.closePath();
        ctx.stroke();

        // Subtle fill
        ctx.fillStyle = 'rgba(60, 60, 70, 0.3)';
        ctx.fill();

        ctx.restore();
      }

      // Draw bullets
      for (const b of bullets) {
        // Glow effect
        const gradient = ctx.createRadialGradient(b.x, b.y, 0, b.x, b.y, b.radius * 3);
        gradient.addColorStop(0, 'rgba(255, 255, 200, 0.9)');
        gradient.addColorStop(0.3, 'rgba(255, 200, 50, 0.5)');
        gradient.addColorStop(1, 'rgba(255, 100, 0, 0)');
        ctx.fillStyle = gradient;
        ctx.beginPath();
        ctx.arc(b.x, b.y, b.radius * 3, 0, Math.PI * 2);
        ctx.fill();

        // Core
        ctx.fillStyle = '#fff';
        ctx.beginPath();
        ctx.arc(b.x, b.y, b.radius, 0, Math.PI * 2);
        ctx.fill();
      }

      // Draw ship
      if (ship.alive) {
        // Blink during invincibility
        if (ship.invincible > 0 && Math.floor(ship.invincible / 6) % 2 === 0) {
          // skip drawing (blink effect)
        } else {
          ctx.save();
          ctx.translate(ship.x, ship.y);
          ctx.rotate(ship.angle);

          // Ship glow
          ctx.shadowColor = ship.invincible > 0 ? '#88f' : '#4af';
          ctx.shadowBlur = 15;

          // Ship body
          ctx.strokeStyle = '#fff';
          ctx.lineWidth = 2;
          ctx.beginPath();
          ctx.moveTo(ship.radius, 0);
          ctx.lineTo(-ship.radius * 0.7, -ship.radius * 0.6);
          ctx.lineTo(-ship.radius * 0.4, 0);
          ctx.lineTo(-ship.radius * 0.7, ship.radius * 0.6);
          ctx.closePath();
          ctx.stroke();

          // Ship fill
          ctx.fillStyle = 'rgba(200, 220, 255, 0.15)';
          ctx.fill();

          ctx.shadowBlur = 0;

          // Engine flame
          if (ship.thrusting) {
            const flameLen = randRange(8, 18);
            ctx.strokeStyle = '#ff6600';
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.moveTo(-ship.radius * 0.5, -ship.radius * 0.3);
            ctx.lineTo(-ship.radius * 0.8 - flameLen, 0);
            ctx.lineTo(-ship.radius * 0.5, ship.radius * 0.3);
            ctx.stroke();

            // Inner flame
            ctx.strokeStyle = '#ffcc00';
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.moveTo(-ship.radius * 0.4, -ship.radius * 0.2);
            ctx.lineTo(-ship.radius * 0.6 - flameLen * 0.7, 0);
            ctx.lineTo(-ship.radius * 0.4, ship.radius * 0.2);
            ctx.stroke();
          }

          ctx.restore();
        }
      }

      ctx.restore(); // end shake

      // Draw subtle vignette overlay for atmosphere
      const gradient = ctx.createRadialGradient(
        canvas.width / 2, canvas.height / 2, canvas.width * 0.3,
        canvas.width / 2, canvas.height / 2, canvas.width * 0.8
      );
      gradient.addColorStop(0, 'rgba(0, 0, 0, 0)');
      gradient.addColorStop(1, 'rgba(0, 0, 20, 0.3)');
      ctx.fillStyle = gradient;
      ctx.fillRect(0, 0, canvas.width, canvas.height);
    }

    // --- Game loop ---
    function gameLoop() {
      update();
      draw();
      requestAnimationFrame(gameLoop);
    }

    // --- Start ---
    function init() {
      ship = {
        x: canvas.width / 2,
        y: canvas.height / 2,
        vx: 0, vy: 0,
        angle: -Math.PI / 2,
        radius: 18,
        alive: true,
        invincible: 0,
        thrusting: false
      };
      asteroids = [];
      bullets = [];
      particles = [];
      score = 0;
      lives = 3;
      wave = 1;
      gameOver = false;
      gameStarted = false;
      screenShake = 0;

      updateHUD();
      messageDiv.innerHTML = '<h1>ASTEROIDS</h1><p>Arrow keys to move · Space to shoot</p><p>Press any key to start</p>';

      gameLoop();
    }

    init();
  })();
  </script>
  </body>
  </html>

  Hmm, let me review this code for issues:

  1. The hex-to-rgba conversion for particles is a bit messy. Let me use a simpler approach - just use rgba strings for particle colors.
  2. The ship drawing: I have 4 vertices which creates a sort of arrow shape (with a notch at the back). That's the classic asteroids ship look. Good.
  3. The createAsteroid function has a nice approach - each asteroid gets a unique irregular polygon shape.
  4. I need to handle the wrap function margins properly. When asteroids spawn off-screen, they should drift onto the screen.
  5. The bullet cooldown using ship._lastShot is a bit hacky but works fine for this.
  6. I should watch out for edge cases: what if the player fires all bullets and they all expire at the same time? Should be fine with the loop.
  7. The respawn check: respawnShip() returns false if center isn't clear, and the update loop will keep calling it each frame until it succeeds.

  Let me also add:

  - A slight particle trail on bullets? Maybe overcomplicating.
  - Better asteroid textures? Nah, keep it simple.

  Let me also make sure that when asteroids are all destroyed, the next wave spawns immediately. I have the check in the bullet-asteroid collision loop, but I should also check it after ship-asteroid collision
  (if the last asteroid kills the ship). Actually, I'll move the wave check to a better place.

  Actually, let me reconsider the code structure. The wave-spawning check should happen after all updates, not inside the collision loop. Let me restructure slightly.

  Also, there's a bug: in the bullet-asteroid collision, if a bullet hits an asteroid and both are removed, the bi index becomes invalid for further checks in the same bullet loop. But since I break out after a
  hit, it's fine.

  However, what if the same bullet could hit two asteroids? That shouldn't happen with proper collision detection (the bullet is removed), so the break is correct.

  Let me also reconsider the particle color handling. I'll use a simpler approach - store colors as objects {r, g, b} or just use simple named colors and handle them in the draw function.

  Actually, let me simplify: I'll store particle colors as arrays [r, g, b] and compute the alpha at draw time.

  Let me revise the particle system:

  function spawnParticles(x, y, count, colorArr, speed, life) {
      for (...) {
          particles.push({
              ...
              color: colorArr, // [r, g, b]
              ...
          });
      }
  }

  And in draw:

  for (const p of particles) {
      const alpha = p.life / p.maxLife;
      ctx.fillStyle = `rgba(${p.color[0]}, ${p.color[1]}, ${p.color[2]}, ${alpha})`;
      ...
  }

  That's cleaner. Let me update the spawn calls:

  - Ship explosion: [255, 68, 68] (red), [255, 170, 0] (orange), [255, 255, 255] (white)
  - Asteroid destruction: [200, 200, 220] (light gray-blue)
  - Engine particles: [255, 102, 0] (orange), [255, 170, 0] (yellow-orange)

  OK, let me finalize the code now and write it to a file.

  Actually, I realize the color for engine particles alternates between orange shades. Let me make that work with arrays.

  color: Math.random() < 0.5 ? [255, 102, 0] : [255, 170, 0],

  Good.

  Let me also add a few more visual touches:

  - The stars could have slight color variations (blue-white, yellow-white)
  - Add a subtle grid or not? No, keep it clean.

  Now let me also think about the overall game feel:

  - The ship's max speed should probably be capped to prevent it from going too fast
  - Or I can add some very subtle drag

  Actually, in classic Asteroids there's no speed cap and no drag. But for playability on a web page, I think letting the ship go very fast is fine. The wrap-around mechanic means they'll come back.

  Let me finalize and write the file. I'll put it at /Users/csabatoth/repos/temp4/asteroids.html.

  Let me also double check: the user wants "cursor buttons to turn and move" and "Space is shooting". So:

  - Left/Right arrows: rotate
  - Up arrow: thrust
  - Space: shoot

  That's exactly what I have.

  One more thing: I should handle the case where keys like ArrowUp trigger page scrolling. The e.preventDefault() should handle that.

  OK, I'm confident the code is solid. Let me write it. But before writing, let me check if there's anything I want to add or adjust:

  1. ✅ Ship with rotation and thrust
  2. ✅ Asteroids with splitting
  3. ✅ Bullets with limited lifetime and count
  4. ✅ Score, lives, waves
  5. ✅ Collision detection
  6. ✅ Screen wrapping
  7. ✅ Particle effects
  8. ✅ Starfield background
  9. ✅ HUD
  10. ✅ Game over / restart
  11. ✅ Invincibility after respawn
  12. ✅ Safe zone check for respawn

  I think this is solid. Let me write the complete file now.

  Wait, one thing I want to improve: the asteroid shapes should look more "asteroid-y". Currently they're just random polygons, which is fine. But I could add a bit of shading or crater-like details. However, the
  user said not to complicate the game logic. Visuals can be cool, but I shouldn't go overboard on the rendering complexity. Let me keep the asteroid rendering simple - just stroked polygons with a subtle fill.

  Actually, let me add a subtle detail: a couple of small "crater" dots inside each asteroid. No, that would require storing crater positions per asteroid and rotating them... too much for the visual payoff. Keep
  it clean.

  Let me also think about whether the game has any bugs:

  Bug potential: spawning asteroids at edges. If I spawn at x=-40 and the asteroid radius is 45, the asteroid's center is off screen but its edge is at x=5, which is on screen. Good - it'll drift in.

  Bug potential: wave spawning after all asteroids destroyed. The check happens in the update loop after bullet-asteroid collisions. But what if an asteroid is destroyed by something else? Currently only bullets
  can destroy them, so it's fine.

  Bug potential: bullet-bullet collision. No such thing, and I don't check for it. Fine.

  Bug potential: multiple key handling. If both left and right are pressed, they cancel out (subtract then add). The ship won't rotate. That's fine.

  Bug potential: space key repeat. I'm using a cooldown system with ship._lastShot, so holding space will fire at a controlled rate. Good.

  Bug potential: game over message appearing multiple times. I'm using setTimeout with a check. The message is only set when gameOver is true and we call the timeout. Since killShip only runs once per death, and
  the game over triggers once... but wait, what if killShip is called multiple times? It has a guard: if (ship.invincible > 0) return;. After the ship dies, ship.alive = false, so collision checks won't trigger
  killShip again. Good.

  Bug potential: respawn during wave spawn. If the player dies and the last asteroid is destroyed simultaneously, a new wave spawns and the player respawns. The respawn check ensures the center is clear. New
  asteroids spawn at edges, so they should be clear. Should be fine.

  I think the code is solid. Let me write it.

  Actually wait, I want to make sure stars are re-positioned on resize. Currently, stars are generated once at init with the initial canvas size. If the window is resized, stars might be outside the new canvas or
  clustered in a corner. Let me regenerate stars on resize too... or just regenerate them periodically. For simplicity, I'll regenerate stars on resize.

  function resize() {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
      // Regenerate stars for new canvas size
      stars.length = 0;
      for (let i = 0; i < 250; i++) {
          stars.push({...});
      }
  }

  OK, final version time. Let me write it all out now.

  Actually, one more thought: I should make the game restart properly. When the game is over and the player presses Space, startGame() is called which resets everything. But I need to make sure the old timeout
  for showing the game over message doesn't interfere. The startGame function sets messageDiv.innerHTML = '', so it clears any pending message. But the setTimeout could fire after startGame and overwrite the
  message. Let me add a check in the timeout.

  Actually, since startGame sets gameOver = false, I can check that in the timeout:

  setTimeout(() => {
      if (gameOver) { // only show if still game over
          messageDiv.innerHTML = '...';
      }
  }, 500);

  Good.

  Let me also consider: should I prevent the default action for ALL keys, or just the game keys? Preventing all keys means things like F5 (refresh) or Ctrl+R won't work. Let me only prevent default for game keys:

  window.addEventListener('keydown', e => {
      if (['ArrowLeft', 'ArrowRight', 'ArrowUp', 'ArrowDown', 'Space'].includes(e.code)) {
          e.preventDefault();
      }
      keys[e.code] = true;
      ...
  });

  That's better.

  Alright, I'm writing the final code now. Let me compose it carefully.

  Let me also consider the visual style. I want it to look cool:

  - Deep space background (#000011 or similar dark blue-black)
  - Stars in white with slight blue/yellow tints
  - Ship in white/cyan with glow
  - Asteroids in gray/white lines
  - Bullets with warm glow (yellow to orange gradient)
  - Explosions in white/orange/red
  - Engine flame in orange/yellow

  The vignette overlay adds depth.

  I'm ready to write. Let me produce the complete file.

  Actually, I realize I need to handle one more thing: the _lastShot property. I'm attaching it to the ship object. This is a bit non-standard but works fine for a game. Let me initialize it properly in
  startGame().

  Also, I need to be careful about the bullet cooldown. I'm using 15 frames, which at 60fps is 0.25 seconds (4 shots per second). That feels about right for Asteroids.

  Let me also think about whether the ship's invincible frames work correctly. I set it to INVINCIBLE_FRAMES (150) on respawn, and decrement it each frame. At 60fps, that's 2.5 seconds of invincibility. The blink
  effect makes the ship visible every other 6 frames (0.1 seconds). That should look good.

  One thing I'm not sure about: the ship collision radius. I use ship.radius * 0.7 for collision (about 12.6 pixels vs visual radius of 18). This gives the player some forgiveness. The asteroid collision uses
  full radius. This feels fair.

  For bullets, the collision radius is 2.5 pixels. The visual radius for the glow is b.radius * 3 = 7.5. So the bullet appears bigger than its collision zone. This is fine - it's a "generous" visual for aiming
  purposes.

  OK OK I'm really writing it now. No more deliberation.

  Let me also add a subtle sound effect note? No, the user didn't mention audio and adding Web Audio API would complicate things. Keep it visual only.

  Wait, I should also consider mobile... but the user specifically mentioned cursor keys and space, so desktop is the target. I won't add touch controls.

  Final thing: I want the game to feel responsive. At 60fps, the rotation speed of 0.07 rad/frame means a full 360° rotation takes about 90 frames = 1.5 seconds. That feels about right for Asteroids - not too
  fast, not too slow.

  Thrust power of 0.12 per frame means after 1 second of thrusting at 60fps, the ship gains velocity of 7.2 units/frame. That's pretty fast. After 2 seconds, 14.4 units/frame. The canvas is typically ~1000-2000
  pixels wide. So crossing the screen at max speed takes about 70-140 frames = 1-2 seconds. That feels right for a fast-paced game.

  Bullet speed of 7 units/frame means bullets cross the screen in about 1-2 seconds. Their lifetime of 70 frames means they travel about 490 units before expiring, which is roughly a quarter to half the screen
  width. That's reasonable. Actually, I might want to increase bullet lifetime a bit. 70 frames at 7 units/frame = 490 units. On a 1920px wide screen, that's about 1/4 of the screen. Hmm, maybe I should increase
  lifetime. But in classic Asteroids, bullets don't go super far either. Let me keep it.

  Actually, let me increase bullet lifetime to 90 frames and maybe speed to 8. That gives 720 units of travel which is better for a 1920px screen. But honestly, the game wraps, so even short-lived bullets will
  hit things. Let me keep the values.

  OK FINAL CODE TIME. Let me write it.

  Let me also reconsider: the asteroid size large with radius 45 and the ship with collision radius ~13. That means the ship needs to be about 58 pixels away from an asteroid center to collide. On a 1920x1080
  screen, 58 pixels is about 3% of the width. It's pretty forgiving. I could increase asteroid size or decrease ship collision margin. Let me keep it as is for playability.

  Actually, I just want to make sure the game is challenging enough. With 4-5 large asteroids on screen, each taking up about 90px diameter, the total asteroid coverage is... let me calculate: each large asteroid
  covers π45² ≈ 6362 sq pixels. 5 of them cover ~31810 sq pixels. Screen area is 19201080 ≈ 2M sq pixels. So asteroids cover about 1.5% of the screen. That's actually quite sparse. The challenge comes from the
  ship's momentum and the asteroids' movement. In the original arcade game, the screen was much smaller relative to asteroid size. This should be fine for a web version.

  Let me actually try generating more initial asteroids. wave + 3 means wave 1 has 4 asteroids, wave 2 has 5, etc. Let me change it to wave + 4 so wave 1 has 5 asteroids. Or maybe wave * 2 + 2 so waves go 4, 6,
  8, 10... That could get intense. Let me stick with wave + 3 = 4, 5, 6, 7... which is a gentler progression.

  Actually, the classic arcade game starts with 4 large asteroids on wave 1, then 6 on wave 2, etc. So wave + 3 (4, 5, 6, 7) is a bit easier than the original. Let me go with wave + 3. It's fine for a demo.

  WRITING THE CODE NOW.

  Let me also double-check: I want to make sure that when all asteroids are gone and a new wave spawns, there's a brief moment of peace. Currently, the check happens right after asteroid destruction in the update
  loop, so new asteroids spawn immediately. In the original game, there's a brief pause. But for simplicity, I'll skip the pause. The player might not even notice since new asteroids spawn at the edges.

  One last thing: I should make sure that if an asteroid is destroyed and splits, the new smaller asteroids don't spawn on top of the ship and immediately kill it. The smaller asteroids spawn at the same position
  as the destroyed asteroid, so if the ship is near that asteroid (e.g., the player just shot it at close range), the new asteroids could instantly collide with the ship. However, if the ship has invincibility
  (after respawn) or the player is shooting from a distance, this shouldn't be an issue. And even if it happens, it creates exciting moments. I'll leave it as is.

  OK HERE WE GO. Final code.

  Actually, I realize I haven't thought about one thing: what if a bullet hits an asteroid at the exact moment the ship collides with it? In my update order: bullets updated → bullet-asteroid collision → ship-
  asteroid collision. If a bullet destroys an asteroid, it's removed from the array before ship collision is checked. So the ship won't collide with a just-destroyed asteroid. Good.

  But what if the bullet and ship hit different asteroids in the same frame? Both are handled correctly since they're independent checks.

  Alright. WRITING. THE. CODE.

  One small optimization: I'll use ctx.fillStyle and ctx.strokeStyle assignments in a way that minimizes string building. But for this simple game, performance won't be an issue.

  Let me also make sure the file starts with <!DOCTYPE html> properly.

  OK, composing the complete file now. I'll write it to disk, then we can test.

  Let me think about one more potential issue: the toFixed or string padding for the score. I used score.toString().padStart(6, '0') which is fine for modern browsers.

  And for the lives display, I used '▲ '.repeat(lives). Wait, I actually built a string with a loop:

  let livesStr = '';
  for (let i = 0; i < lives; i++) {
      livesStr += '▲ ';
  }

  That's fine.

  OK, submitting the final code now.

  Let me also add one more visual touch: when asteroids are destroyed, I want to see the particles spread out nicely. The spawnParticles function creates particles with random directions and speeds. The colors
  should be whitish-gray for asteroid debris. I'm using [200, 200, 220] which is a light blue-gray. That should look nice against the dark background.

  For the ship explosion, I spawn three layers:

  1. Red particles (inner, faster) — 25 count
  2. Orange particles (middle) — 15 count
  3. White particles (outer) — 10 count

  This creates a layered explosion effect. Should look great.

  One more thing: the engine flame particles. When thrusting, I spawn particles behind the ship. The direction is ship.angle + Math.PI (opposite the ship's facing) with some spread. The particles have short lives
  (10-25 frames) and orange/yellow colors. This creates a nice exhaust trail.

  I'm also varying the flame length in the ship drawing for a flickering effect. Good.

  Alright, I'm truly writing the final code now. No more changes.

  Wait, one issue: in the draw() function, I apply screen shake by translating the entire canvas. But the HUD is HTML overlays, not canvas-drawn. So the HUD won't shake. That's actually good — the HUD should
  remain stable while the game world shakes.

  But wait, I draw stars and all game objects inside the translated context. During shake, the stars will also shake, which looks odd. Stars should be distant and stable. In a more polished game, I'd draw stars
  outside the shake transform. But for simplicity, this is fine. The shake is subtle (max 20 pixels, decaying quickly).

  Actually, I could draw stars before applying the shake transform. Let me restructure:

  function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // Draw stars (no shake)
      for (const s of stars) { ... }

      // Apply shake
      ctx.save();
      ctx.translate(shakeX, shakeY);

      // Draw game objects
      ...

      ctx.restore();

      // Vignette (no shake)
      ...
  }

  Yes, that's better. Let me update.

  OK NOW I'm really writing it. Final final version.

  Let me also consider: do I want to add any text like "WAVE 2" when a new wave starts? That would be a nice touch. Let me add a simple wave announcement that fades out.

  I'll add a waveMessage state with a timer:

  let waveMessage = '';
  let waveMessageTimer = 0;

  When a new wave spawns:

  waveMessage = `WAVE ${wave}`;
  waveMessageTimer = 120; // 2 seconds

  In the draw function:

  if (waveMessageTimer > 0) {
      const alpha = Math.min(1, waveMessageTimer / 60);
      ctx.fillStyle = `rgba(255, 255, 255, ${alpha})`;
      ctx.font = '36px "Courier New", monospace';
      ctx.textAlign = 'center';
      ctx.fillText(waveMessage, canvas.width / 2, canvas.height / 3);
      waveMessageTimer--;
  }

  That's a nice touch. Let me add it.

  OK, I need to actually stop deliberating and write the file. Let me compose the complete HTML now.

• Ran cat > /Users/csabatoth/repos/temp4/asteroids.html << 'ENDOFFILE'
  │ <!DOCTYPE html>
  │ <html lang="en">
  │ … +627 lines
  └ Written asteroids.html (   15836 bytes)

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• Now let me open it in your browser:

• Ran open /Users/csabatoth/repos/temp4/asteroids.html
  └ No application knows how to open URL file:///Users/csabatoth/repos/temp4/asteroids.html (Error Domain=NSOSStatusErrorDomain Code=-10661 "kLSExecutableIncorrectFormat: No compatible executable was found"
    UserInfo={_LSLine=1793, _LSFunction=runEvaluator}).

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• The file was written successfully. Let me open it with your default browser:

• Ran open -a "Google Chrome" /Users/csabatoth/repos/temp4/asteroids.html 2>/dev/null || open -a "Safari" /Users/csabatoth/repos/temp4/asteroids.html 2>/dev/null || open -a "Firefox" /Users/csabatoth/repos/temp4/
  │ asteroids.html 2>/dev/null || echo "Could not auto-open browser. File is at: /Users/csabatoth/repos/temp4/asteroids.html"
  └ Could not auto-open browser. File is at: /Users/csabatoth/repos/temp4/asteroids.html

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• The game is written and ready at /Users/csabatoth/repos/temp4/asteroids.html — just open it in any browser (double-click or drag into Chrome/Safari/Firefox).

  Here's what's in it:

  Controls

  - ← → — rotate ship left/right
  - ↑ — thrust forward (with animated flame)
  - Space — shoot
  - Mobile: tap canvas to shoot

  Gameplay

  - Classic Asteroids: destroy all rocks to advance to the next level
  - 3 lives — losing all triggers Game Over; press PLAY AGAIN to restart
  - 3-second invincibility on respawn and level transitions (ship blinks)
  - Large asteroids split into 2 medium, medium split into 2 small — full cascade
  - Scoring: 50 / 100 / 200 points per asteroid size (large/medium/small)

  Visuals

  - Twinkling starfield background
  - Jagged, hand-generated asteroid shapes (not just circles)
  - Glowing cyan ship with animated orange/yellow thrust flame
  - Particle explosions on destruction with screen shake
  - Retro monospace score + heart-based lives display

  Tech

  - Single self-contained HTML file, zero dependencies
  - Canvas-based rendering at 60fps via requestAnimationFrame
  - Proper point-in-polygon collision detection for bullets/ship vs irregular asteroid shapes
  - Screen-wrapping (objects exit one side, enter the other)

─ Worked for 7m 32s ────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
Token usage: total=62,236 input=42,129 (+ 9,064 cached) output=20,107