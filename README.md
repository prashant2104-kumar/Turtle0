<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Subway Surfer - 2D Runner</title>
<style>
  /* Reset & base styles */
  * {
    box-sizing: border-box;
    margin: 0; padding: 0;
  }
  body, html {
    height: 100%;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: linear-gradient(to top, #0f2027, #203a43, #2c5364);
    overflow: hidden;
    user-select: none;
  }

  #game {
    position: relative;
    width: 400px;
    height: 600px;
    margin: 40px auto;
    background: linear-gradient(to bottom, #74ebd5, #ACB6E5);
    border-radius: 15px;
    box-shadow: 0 10px 30px rgb(0 0 0 / 0.4);
    overflow: hidden;
  }

  /* Track lanes */
  .lane {
    position: absolute;
    bottom: 0;
    width: 33.3333%;
    height: 100%;
    border-left: 2px solid rgba(255,255,255,0.1);
    border-right: 2px solid rgba(255,255,255,0.1);
    pointer-events: none;
  }
  .lane:nth-child(1) {
    left: 0;
  }
  .lane:nth-child(2) {
    left: 33.3333%;
  }
  .lane:nth-child(3) {
    left: 66.6666%;
  }
  /* Track road base */
  #road {
    position: absolute;
    bottom: 0;
    width: 100%;
    height: 180px;
    background: repeating-linear-gradient(
      45deg,
      #444, #444 20px,
      #555 20px, #555 40px
    );
    box-shadow:
      inset 0 0 15px #0008,
      0 10px 10px #000a;
    z-index: 4;
  }

  /* Player */
  #player {
    position: absolute;
    bottom: 180px;
    width: 70px;
    height: 90px;
    left: calc(33.3333% + (33.3333% / 2) - 35px);
    background: linear-gradient(135deg, #ff416c, #ff4b2b);
    border-radius: 20px;
    box-shadow:
      0 5px 15px #e63946aa,
      inset 0 0 15px #fb3640;
    transition: left 0.2s ease, bottom 0.2s ease;
    z-index: 10;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  /* Player surfboard styling */
  #player::before {
    content: "";
    position: absolute;
    bottom: -15px;
    width: 90px;
    height: 20px;
    background: linear-gradient(90deg, #ffd452, #ffb347);
    border-radius: 45% 45% 0 0;
    box-shadow: 0 2px 7px #ffaf17cc;
  }

  /* Player face */
  #player-head {
    width: 40px;
    height: 40px;
    background: #fff;
    border-radius: 50%;
    box-shadow: 0 0 15px #ff4b2bbb inset;
    position: relative;
    z-index: 12;
  }
  #player-head::before {
    content: "";
    position: absolute;
    top: 12px;
    left: 8px;
    width: 6px;
    height: 6px;
    border-radius: 50%;
    background: #000;
    box-shadow: 15px 0 #000;
  }

  /* Obstacle */
  .obstacle {
    position: absolute;
    bottom: 180px;
    width: 50px;
    height: 70px;
    border-radius: 15px;
    background: linear-gradient(135deg, #333, #111);
    box-shadow: 0 0 15px #222 inset;
    z-index: 7;
    transition: bottom 0.1s ease;
  }
  .obstacle.spike {
    background: linear-gradient(135deg, #f44336, #d32f2f);
    clip-path: polygon(50% 0%, 100% 100%, 0 100%);
    width: 40px;
    height: 40px;
    bottom: 190px;
  }

  /* Score */
  #scoreBoard {
    position: absolute;
    top: 15px;
    left: 15px;
    color: #fff;
    font-size: 20px;
    font-weight: 700;
    letter-spacing: 1px;
    text-shadow: 0 0 5px #0007;
    z-index: 15;
  }

  /* Game over screen */
  #gameOverScreen {
    position: absolute;
    top: 0; left: 0; right: 0; bottom: 0;
    background: rgba(0,0,0,0.85);
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    color: #ff4b2b;
    font-size: 32px;
    font-weight: 900;
    z-index: 20;
    opacity: 0;
    pointer-events: none;
    transition: opacity 0.3s ease;
    padding: 0 20px;
    text-align: center;
  }
  #gameOverScreen.visible {
    opacity: 1;
    pointer-events: all;
  }
  #gameOverScreen button {
    margin-top: 25px;
    padding: 12px 30px;
    font-size: 18px;
    font-weight: 700;
    color: #fff;
    background: #ff4b2b;
    border: none;
    border-radius: 40px;
    cursor: pointer;
    box-shadow: 0 5px 15px #ff4b2bbb;
    transition: background-color 0.3s ease;
    user-select: none;
  }
  #gameOverScreen button:hover {
    background: #d5381c;
  }

</style>
</head>
<body>
  <div id="game" tabindex="0" aria-label="Subway Surfer 2D Game">
    <div id="road"></div>
    <div class="lane"></div>
    <div class="lane"></div>
    <div class="lane"></div>

    <div id="player" aria-live="polite" aria-atomic="true">
      <div id="player-head"></div>
    </div>

    <div id="scoreBoard" aria-label="Score">Score: 0</div>

    <div id="gameOverScreen" role="alertdialog" aria-modal="true" aria-labelledby="gameOverTitle" aria-describedby="gameOverScore">
      <div id="gameOverTitle">Game Over</div>
      <div id="gameOverScore">You scored: 0</div>
      <button id="restartBtn" aria-label="Restart game">Restart</button>
    </div>
  </div>

<script>
(() => {
  const game = document.getElementById('game');
  const player = document.getElementById('player');
  const lanes = [0, 1, 2];
  const laneWidth = game.clientWidth / 3;
  const scoreBoard = document.getElementById('scoreBoard');
  const gameOverScreen = document.getElementById('gameOverScreen');
  const gameOverScore = document.getElementById('gameOverScore');
  const restartBtn = document.getElementById('restartBtn');

  // Game state variables
  let playerLane = 1; // middle lane = 1
  let playerY = 0; // 0 = on ground, >0 = jumping
  let isJumping = false;
  let jumpSpeed = 0;
  let gravity = 0.0055;
  let obstacles = [];
  let gameSpeed = 0.0035;
  let spawnTimer = 0;
  let spawnInterval = 1600; // ms
  let lastTimestamp = null;
  let score = 0;
  let running = false;

  // Player settings
  const jumpHeight = 1.3; // how high the jump is in y units

  // Setup initial player position
  function updatePlayerPosition() {
    const leftPos = playerLane * laneWidth + laneWidth / 2 - player.clientWidth / 2;
    player.style.left = leftPos + 'px';
    // bottom: 180px base + jump offset in px (scale jump)
    player.style.bottom = (180 + playerY * 150) + 'px';
  }

  // Obstacle class
  class Obstacle {
    constructor(lane) {
      this.lane = lane;
      this.yPos = 600; // starts off screen at bottom
      this.width = 50;
      this.height = 70;
      this.element = document.createElement('div');
      this.element.classList.add('obstacle');

      // Randomize obstacle type (simple block or spike)
      if (Math.random() < 0.4) {
        this.element.classList.add('spike');
        this.height = 40;
        this.width = 40;
      }

      game.appendChild(this.element);
      this.updatePosition();
    }

    updatePosition() {
      const leftPos = this.lane * laneWidth + laneWidth / 2 - this.width / 2;
      this.element.style.left = leftPos + 'px';
      this.element.style.bottom = this.yPos + 'px';
    }

    move(deltaTime) {
      this.yPos -= deltaTime * 300 * gameSpeed * 1000;
      this.updatePosition();
    }

    isOffScreen() {
      return this.yPos + this.height < 0;
    }

    collidesWithPlayer() {
      // Player rectangle
      const playerX = playerLane * laneWidth + laneWidth / 2 - player.clientWidth / 2;
      const playerYBottom = 180 + playerY * 150;
      const playerRect = {
        x: playerX,
        y: playerYBottom,
        width: player.clientWidth,
        height: player.clientHeight
      };

      // Obstacle rectangle
      const obsX = this.lane * laneWidth + laneWidth / 2 - this.width / 2;
      const obsY = this.yPos;
      const obsRect = {
        x: obsX,
        y: obsY,
        width: this.width,
        height: this.height
      };

      // Collision detection - AABB
      const overlapX = playerRect.x < obsRect.x + obsRect.width &&
                       playerRect.x + playerRect.width > obsRect.x;
      const overlapY = playerRect.y < obsRect.y + obsRect.height &&
                       playerRect.y + playerRect.height > obsRect.y;
      return overlapX && overlapY;
    }

    destroy() {
      game.removeChild(this.element);
    }
  }

  // Handle input
  function handleKeyDown(e) {
    if (!running) return;
    switch(e.key) {
      case 'ArrowLeft':
      case 'a':
      case 'A':
        if (playerLane > 0) {
          playerLane--;
          updatePlayerPosition();
        }
        break;

      case 'ArrowRight':
      case 'd':
      case 'D':
        if (playerLane < 2) {
          playerLane++;
          updatePlayerPosition();
        }
        break;

      case 'ArrowUp':
      case 'w':
      case 'W':
      case ' ':
        if (!isJumping) {
          isJumping = true;
          jumpSpeed = 0.02;
        }
        break;
    }
  }
  
  // Reset game state
  function resetGame() {
    obstacles.forEach(obs => obs.destroy());
    obstacles = [];
    playerLane = 1;
    playerY = 0;
    isJumping = false;
    jumpSpeed = 0;
    score = 0;
    spawnTimer = 0;
    gameSpeed = 0.0035;
    lastTimestamp = null;
    running = true;
    updatePlayerPosition();
    scoreBoard.textContent = "Score: 0";
    gameOverScreen.classList.remove('visible');
  }

  // Game over
  function gameOver() {
    running = false;
    gameOverScore.textContent = `You scored: ${score}`;
    gameOverScreen.classList.add('visible');
  }

  // Game loop
  function loop(timestamp) {
    if (!running) return;
    if (!lastTimestamp) lastTimestamp = timestamp;
    const deltaTime = timestamp - lastTimestamp;
    lastTimestamp = timestamp;

    // Increase score
    score += deltaTime * 0.02;
    scoreBoard.textContent = `Score: ${Math.floor(score)}`;

    // Increase speed every 10 seconds
    if (Math.floor(score) % 100 === 0 && gameSpeed < 0.01) {
      gameSpeed += 0.000001 * deltaTime;
    }

    // Spawn obstacles
    spawnTimer += deltaTime;
    if (spawnTimer > spawnInterval) {
      spawnTimer = 0;
      const lane = lanes[Math.floor(Math.random() * lanes.length)];
      obstacles.push(new Obstacle(lane));
      // Increase difficulty by reducing interval gradually to a minimum
      if (spawnInterval > 600) {
        spawnInterval -= 10;
      }
    }

    // Update obstacles
    for (let i = obstacles.length -1; i >= 0; i--) {
      obstacles[i].move(deltaTime);
      if (obstacles[i].isOffScreen()) {
        obstacles[i].destroy();
        obstacles.splice(i, 1);
        continue;
      }
      if (obstacles[i].collidesWithPlayer()) {
        gameOver();
        return;
      }
    }

    // Update jump physics
    if (isJumping) {
      playerY += jumpSpeed * deltaTime;
      jumpSpeed -= gravity * deltaTime;

      if (playerY <= 0) {
        playerY = 0;
        isJumping = false;
        jumpSpeed = 0;
      }
      updatePlayerPosition();
    }

    requestAnimationFrame(loop);
  }

  // Initialize
  function init() {
    updatePlayerPosition();
    resetGame();
  }

  // Event listeners
  window.addEventListener('keydown', handleKeyDown);
  restartBtn.addEventListener('click', () => {
    resetGame();
    requestAnimationFrame(loop);
    game.focus();
  });

  // Start game
  init();
  requestAnimationFrame(loop);
  game.focus();

})();
</script>
</body>
</html>

