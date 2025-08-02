
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>横スクロールアクションゲーム</title>
  <style>
    body {
      background-color: white;
      margin: 0;
      padding: 0;
      text-align: center;
      font-family: sans-serif;
    }
    canvas {
      background: #87CEEB;
      display: block;
      margin: 0 auto;
      border: 1px solid #444;
      /* スマホ対応: タッチ操作を有効にする */
      touch-action: none;
    }
    #startButton, #retryButton {
      margin-top: 10px;
    }
    #retryButton {
      display: none;
    }
  </style>
</head>
<body>
<canvas id="gameCanvas" width="800" height="400"></canvas>
<button id="startButton" onclick="startGame()">GAME START</button>
<button id="retryButton" onclick="restartGame()">もう一度プレイ</button>
<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");
const canvasWidth = canvas.width;
const canvasHeight = canvas.height;
const groundHeight = 40;
const groundY = canvasHeight - groundHeight;

let score = 0;
let speed = 3;
let hp = 3;
let gameOver = false;
let gameStarted = false;
let frame = 0;
let skyPhase = 0;
let skyColor = "#87CEEB";
let targetSkyColor = skyColor;
let lastEnemyType = null;
let gameOverImageCache = null;
let explosions = [];
let gameOverOpacity = 0;
const MAX_HP = 4;
let animationId; // ゲームループのIDを保持する変数

function blendColors(c1, c2, amount) {
  const [r1, g1, b1] = [parseInt(c1.slice(1, 3), 16), parseInt(c1.slice(3, 5), 16), parseInt(c1.slice(5, 7), 16)];
  const [r2, g2, b2] = [parseInt(c2.slice(1, 3), 16), parseInt(c2.slice(3, 5), 16), parseInt(c2.slice(5, 7), 16)];
  const r = Math.round(r1 + (r2 - r1) * amount);
  const g = Math.round(g1 + (g2 - g1) * amount);
  const b = Math.round(b1 + (b2 - b1) * amount);
  return `#${r.toString(16).padStart(2, '0')}${g.toString(16).padStart(2, '0')}${b.toString(16).padStart(2, '0')}`;
}

let imagesLoadedCount = 0;
const totalImages = 38;

function imageLoaded() {
    imagesLoadedCount++;
    if (imagesLoadedCount === totalImages) {
        drawInitialScreen();
    }
}

let playerNormalWalkImages = [new Image(), new Image()];
playerNormalWalkImages[0].onload = playerNormalWalkImages[1].onload = imageLoaded;
playerNormalWalkImages[0].src = "player1.png";
playerNormalWalkImages[1].src = "player1.2.png";
let playerNormalJumpImage = new Image();
playerNormalJumpImage.onload = imageLoaded;
playerNormalJumpImage.src = "player1.3.png";

let playerDamaged1WalkImages = [new Image(), new Image()];
playerDamaged1WalkImages[0].onload = playerDamaged1WalkImages[1].onload = imageLoaded;
playerDamaged1WalkImages[0].src = "player_damaged_1.png";
playerDamaged1WalkImages[1].src = "player_damaged_1.2.png";
let playerDamaged1JumpImage = new Image();
playerDamaged1JumpImage.onload = imageLoaded;
playerDamaged1JumpImage.src = "player_damaged_1.3.png";

let playerDamaged2WalkImages = [new Image(), new Image()];
playerDamaged2WalkImages[0].onload = playerDamaged2WalkImages[1].onload = imageLoaded;
playerDamaged2WalkImages[0].src = "player_damaged_2.png";
playerDamaged2WalkImages[1].src = "player_damaged_2.2.png";
let playerDamaged2JumpImage = new Image();
playerDamaged2JumpImage.onload = imageLoaded;
playerDamaged2JumpImage.src = "player_damaged_2.3.png";

let player2WalkImages = [new Image(), new Image()];
player2WalkImages[0].onload = player2WalkImages[1].onload = imageLoaded;
player2WalkImages[0].src = "player2.png";
player2WalkImages[1].src = "player2.2.png";
let player2JumpImage = new Image();
player2JumpImage.onload = imageLoaded;
player2JumpImage.src = "player2.3.png";

let player3WalkImages = [new Image(), new Image()];
player3WalkImages[0].onload = player3WalkImages[1].onload = imageLoaded;
player3WalkImages[0].src = "player3.png";
player3WalkImages[1].src = "player3.2.png";
let player3JumpImage = new Image();
player3JumpImage.onload = imageLoaded;
player3JumpImage.src = "player3.3.png";

let startScreenImage = new Image();
startScreenImage.onload = imageLoaded;
startScreenImage.src = "start_screen.png";

const originalJumpPower = -20;
const slowJumpPower = -10;
const statusEffectDuration = 180;

let player = {
  x: 50,
  y: groundY - 60,
  width: 60,
  height: 60,
  vy: 0,
  gravity: 1.2,
  jumpPower: originalJumpPower,
  isJumping: false,
  imageIndex: 0,
  status: null,
  statusTimer: 0
};

let heartImg = new Image();
heartImg.onload = imageLoaded;
heartImg.src = "heart.png";

let enemyAnimations = {
  bom: {
    images: [new Image(), new Image()],
    width: 30,
    height: 30,
    animationSpeed: 10,
    isBouncing: false,
    isExplosive: true,
    speedMultiplier: 1,
    damage: 1
  },
  enemy2: {
    images: [new Image(), new Image(), new Image()],
    width: 40,
    height: 40,
    animationSpeed: 10,
    isBouncing: false,
    isExplosive: false,
    speedMultiplier: 1,
    damage: 1
  },
  enemy3: {
    images: [new Image(), new Image(), new Image()],
    width: 50,
    height: 50,
    animationSpeed: 15,
    isBouncing: false,
    isExplosive: false,
    speedMultiplier: 1,
    damage: 1
  },
  enemy4: {
    images: [new Image(), new Image(), new Image()],
    width: 45,
    height: 45,
    animationSpeed: 8,
    isBouncing: true,
    jumpPower: -18,
    gravity: 0.8,
    groundOffset: 0,
    isExplosive: false,
    speedMultiplier: 1,
    damage: 1
  },
  enemy5: {
    images: [new Image(), new Image(), new Image(), new Image()],
    width: 60,
    height: 60,
    animationSpeed: 5,
    isBouncing: false,
    isExplosive: false,
    speedMultiplier: 2,
    damage: 2
  },
  enemy6: {
    images: [new Image(), new Image(), new Image(), new Image()],
    width: 50,
    height: 50,
    animationSpeed: 6,
    isBouncing: false,
    isExplosive: false,
    speedMultiplier: 1.5,
    damage: -1
  },
  enemy7: {
    images: [new Image(), new Image(), new Image()],
    width: 70,
    height: 70,
    animationSpeed: 5,
    isBouncing: false,
    isExplosive: false,
    speedMultiplier: 1.1,
    damage: 4
  },
  enemy8: {
    images: [new Image(), new Image(), new Image()],
    width: 30,
    height: 30,
    animationSpeed: 10,
    isBouncing: false,
    isExplosive: false,
    speedMultiplier: 1,
    damage: 1,
    statusEffect: "slowJump"
  }
};

enemyAnimations.bom.images[0].onload = enemyAnimations.bom.images[1].onload = imageLoaded;
enemyAnimations.bom.images[0].src = "bom.png";
enemyAnimations.bom.images[1].src = "bom2.png";

enemyAnimations.enemy2.images[0].onload = enemyAnimations.enemy2.images[1].onload = enemyAnimations.enemy2.images[2].onload = imageLoaded;
enemyAnimations.enemy2.images[0].src = "enemy2.png";
enemyAnimations.enemy2.images[1].src = "enemy2.2.png";
enemyAnimations.enemy2.images[2].src = "enemy2.3.png";

enemyAnimations.enemy3.images[0].onload = enemyAnimations.enemy3.images[1].onload = enemyAnimations.enemy3.images[2].onload = imageLoaded;
enemyAnimations.enemy3.images[0].src = "enemy3.png";
enemyAnimations.enemy3.images[1].src = "enemy3.2.png";
enemyAnimations.enemy3.images[2].src = "enemy3.3.png";

enemyAnimations.enemy4.images[0].onload = enemyAnimations.enemy4.images[1].onload = enemyAnimations.enemy4.images[2].onload = imageLoaded;
enemyAnimations.enemy4.images[0].src = "enemy4.png";
enemyAnimations.enemy4.images[1].src = "enemy4.2.png";
enemyAnimations.enemy4.images[2].src = "enemy4.3.png";

enemyAnimations.enemy5.images[0].onload = enemyAnimations.enemy5.images[1].onload = enemyAnimations.enemy5.images[2].onload = enemyAnimations.enemy5.images[3].onload = imageLoaded;
enemyAnimations.enemy5.images[0].src = "enemy5.png";
enemyAnimations.enemy5.images[1].src = "enemy5.2.png";
enemyAnimations.enemy5.images[2].src = "enemy5.3.png";
enemyAnimations.enemy5.images[3].src = "enemy5.4.png";

enemyAnimations.enemy6.images[0].onload = enemyAnimations.enemy6.images[1].onload = enemyAnimations.enemy6.images[2].onload = enemyAnimations.enemy6.images[3].onload = imageLoaded;
enemyAnimations.enemy6.images[0].src = "enemy6.png";
enemyAnimations.enemy6.images[1].src = "enemy6.2.png";
enemyAnimations.enemy6.images[2].src = "enemy6.3.png";
enemyAnimations.enemy6.images[3].src = "enemy6.4.png";

enemyAnimations.enemy7.images[0].onload = enemyAnimations.enemy7.images[1].onload = enemyAnimations.enemy7.images[2].onload = imageLoaded;
enemyAnimations.enemy7.images[0].src = "enemy7.png";
enemyAnimations.enemy7.images[1].src = "enemy7.2.png";
enemyAnimations.enemy7.images[2].src = "enemy7.3.png";

enemyAnimations.enemy8.images[0].onload = enemyAnimations.enemy8.images[1].onload = enemyAnimations.enemy8.images[2].onload = imageLoaded;
enemyAnimations.enemy8.images[0].src = "enemy8.png";
enemyAnimations.enemy8.images[1].src = "enemy8.2.png";
enemyAnimations.enemy8.images[2].src = "enemy8.3.png";

const enemyTypesNormal = ["bom", "enemy2", "enemy3", "enemy4", "enemy6", "enemy7"];
const enemyTypesAfter2000 = ["bom", "enemy2", "enemy3", "enemy4", "enemy6", "enemy7", "enemy8"];
const enemyTypesSlowJump = ["bom", "enemy2", "enemy3", "enemy4", "enemy6", "enemy8"];
let buildings = [];
let trees = [];
let clouds = [];
let sun = { x: 100, y: 50 };
let moon = { x: 100, y: 50 };

for (let i = 0; i < 5; i++) {
  buildings.push({ x: 200 * i + 300, y: groundY - 120, width: 50, height: 120 });
  trees.push({ x: 200 * i + 250, y: groundY - 40, width: 20, height: 40 });
  clouds.push({ x: 150 * i + 100, y: 50, radius: 20 });
}

let enemies = [];
let holes = [];
let healingItems = [];

let healingItemImages = [new Image(), new Image()];
healingItemImages[0].onload = healingItemImages[1].onload = imageLoaded;
healingItemImages[0].src = "healing_item.png";
healingItemImages[1].src = "healing_item2.png";

const healingItemAnimationSpeed = 15;

function createHealingItem() {
  healingItems.push({
    x: canvasWidth,
    y: groundY - 30,
    width: 30,
    height: 30,
    animationFrame: 0,
    animationCounter: 0
  });
}

function createEnemy(typeName = null) {
  let enemyData;

  if (typeName) {
    enemyData = enemyAnimations[typeName];
  } else {
    let availableEnemyTypes;
    if (player.status === "slowJump") {
        availableEnemyTypes = enemyTypesSlowJump;
    } else {
        availableEnemyTypes = score < 2000 ? enemyTypesNormal : enemyTypesAfter2000;
    }
    
    typeName = availableEnemyTypes[Math.floor(Math.random() * availableEnemyTypes.length)];
    enemyData = enemyAnimations[typeName];
  }

  let y;
  let initialVy = 0;
  let currentGravity = 0;
  let currentJumpPower = 0;
  let isBouncingEnemy = enemyData.isBouncing || false;

  if (typeName === "bom") {
    let yOptions = [groundY - enemyData.height, groundY - enemyData.height - 50, groundY - enemyData.height - 80];
    y = yOptions[Math.floor(Math.random() * yOptions.length)];
  } else if (isBouncingEnemy) {
    y = groundY - enemyData.height - (enemyData.initialYOffset || 0);
    initialVy = enemyData.jumpPower || 0;
    currentGravity = enemyData.gravity || 0;
    currentJumpPower = enemyData.jumpPower || 0;
  }
  else {
    y = groundY - enemyData.height;
  }

  enemies.push({ 
    x: canvasWidth, 
    y: y, 
    width: enemyData.width, 
    height: enemyData.height, 
    type: typeName,
    animationFrame: 0,
    animationCounter: 0,
    vy: initialVy,
    gravity: currentGravity,
    jumpPower: currentJumpPower,
    isBouncing: isBouncingEnemy,
    groundYAdjust: groundY - (enemyData.groundOffset || 0),
    isExplosive: enemyData.isExplosive,
    speedMultiplier: enemyData.speedMultiplier,
    damage: enemyData.damage
  });
}

let spawnTimer = 0;
const spawnInterval = 150;

function spawnObstacle() {
  spawnTimer = 0;
  let rand = Math.random();
  
  if (skyPhase === 2 && Math.random() < 0.1) {
    createEnemy("enemy5");
    return;
  }

  if (score > 5000) {
    if (rand < 0.1) {
      createHealingItem();
    } else if (rand < 0.3) {
      holes.push({ x: canvasWidth, width: Math.random() * 80 + 50 });
    } else {
      createEnemy();
    }
  } else if (score > 4000) {
    if (rand < 0.25) {
      holes.push({ x: canvasWidth, width: Math.random() * 80 + 50 });
    } else {
      createEnemy();
    }
  } else {
    createEnemy();
  }
}

function updateSkyColor() {
  if (score % 1500 === 0 && score !== 0) {
    skyPhase = (skyPhase + 1) % 3;
    if (skyPhase === 0) targetSkyColor = "#87CEEB";
    else if (skyPhase === 1) targetSkyColor = "#FFA07A";
    else targetSkyColor = "#191970";
  }
  skyColor = blendColors(skyColor, targetSkyColor, 0.02);
}

function drawBackground() {
  ctx.fillStyle = skyColor;
  ctx.fillRect(0, 0, canvasWidth, canvasHeight);

  if (skyPhase === 0) {
    ctx.beginPath();
    ctx.arc(sun.x, sun.y, 25, 0, Math.PI * 2);
    ctx.fillStyle = "yellow";
    ctx.fill();
  } else if (skyPhase === 2) {
    ctx.beginPath();
    ctx.arc(moon.x, moon.y, 20, 0, Math.PI * 2);
    ctx.fillStyle = "yellow";
    ctx.fill();
  }

  ctx.fillStyle = "white";
  for (let cloud of clouds) {
    ctx.beginPath();
    ctx.arc(cloud.x, cloud.y, cloud.radius, 0, Math.PI * 2);
    ctx.arc(cloud.x + 20, cloud.y, cloud.radius, 0, Math.PI * 2);
    ctx.arc(cloud.x + 10, cloud.y - 10, cloud.radius, 0, Math.PI * 2);
    ctx.fill();
    cloud.x -= 0.5;
    if (cloud.x < -30) cloud.x = canvasWidth + Math.random() * 100;
  }

  ctx.fillStyle = "#555";
  for (let b of buildings) {
    ctx.fillRect(b.x, b.y, b.width, b.height);
    ctx.fillStyle = "#ccc";
    for (let i = 5; i < b.height; i += 20) {
      ctx.fillRect(b.x + 10, b.y + i, 10, 10);
    }
    b.x -= speed;
    if (b.x + b.width < 0) b.x = canvasWidth + Math.random() * 100;
    ctx.fillStyle = "#555";
  }

  for (let t of trees) {
    ctx.fillStyle = "brown";
    ctx.fillRect(t.x + 7, t.y + 20, 6, 20);
    ctx.beginPath();
    ctx.arc(t.x + 10, t.y + 15, 12, 0, Math.PI * 2);
    ctx.fillStyle = "green";
    ctx.fill();
    t.x -= speed;
    if (t.x + t.width < 0) t.x = canvasWidth + Math.random() * 100;
  }
}

function drawPlayer() {
  let currentWalkImages;
  let currentJumpImage;

  if (player.status === "slowJump") {
    currentWalkImages = player3WalkImages;
    currentJumpImage = player3JumpImage;
  } else if (hp === 4) {
    currentWalkImages = player2WalkImages;
    currentJumpImage = player2JumpImage;
  } else if (hp === 3) {
    currentWalkImages = playerNormalWalkImages;
    currentJumpImage = playerNormalJumpImage;
  } else if (hp === 2) {
    currentWalkImages = playerDamaged1WalkImages;
    currentJumpImage = playerDamaged1JumpImage;
  } else {
    currentWalkImages = playerDamaged2WalkImages;
    currentJumpImage = playerDamaged2JumpImage;
  }

  let img = player.isJumping ? currentJumpImage : currentWalkImages[Math.floor(frame / 10) % 2];
  ctx.drawImage(img, player.x, player.y, player.width, player.height);
}

function drawEnemies() {
  for (let e of enemies) {
    const enemyData = enemyAnimations[e.type];
    if (!enemyData || enemyData.images.length === 0) continue;

    if (enemyData.animationSpeed > 0) {
      e.animationCounter++;
      if (e.animationCounter >= enemyData.animationSpeed) {
        e.animationFrame = (e.animationFrame + 1) % enemyData.images.length;
        e.animationCounter = 0;
      }
    }

    const currentImage = enemyData.images[e.animationFrame];
    ctx.drawImage(currentImage, e.x, e.y, e.width, e.height);
  }
}

function drawHealingItems() {
  for (let item of healingItems) {
    item.animationCounter++;
    if (item.animationCounter >= healingItemAnimationSpeed) {
      item.animationFrame = (item.animationFrame + 1) % healingItemImages.length;
      item.animationCounter = 0;
    }
    const currentImage = healingItemImages[item.animationFrame];
    ctx.drawImage(currentImage, item.x, item.y, item.width, item.height);
  }
}

function drawExplosions() {
  for (let i = 0; i < explosions.length; i++) {
    let exp = explosions[i];
    ctx.save();
    ctx.globalAlpha = exp.opacity;

    ctx.beginPath();
    let gradient = ctx.createRadialGradient(exp.x, exp.y, 0, exp.x, exp.y, exp.radius);
    gradient.addColorStop(0, "rgba(255, 255, 0, " + exp.opacity + ")");
    gradient.addColorStop(0.5, "rgba(255, 165, 0, " + exp.opacity + ")");
    gradient.addColorStop(1, "rgba(255, 0, 0, 0)");
    ctx.fillStyle = gradient;
    ctx.arc(exp.x, exp.y, exp.radius, 0, Math.PI * 2);
    ctx.fill();

    ctx.restore();
  }
}

function drawGround() {
  ctx.fillStyle = "#654321";
  ctx.fillRect(0, groundY, canvasWidth, groundHeight);

  ctx.fillStyle = "black";
  for (let h of holes) {
    ctx.fillRect(h.x, groundY, h.width, groundHeight);
  }
}

function drawHearts() {
  for (let i = 0; i < hp; i++) {
    ctx.drawImage(heartImg, 10 + i * 30, 10, 25, 25);
  }
}

function drawScore() {
  ctx.fillStyle = "black";
  ctx.font = "16px sans-serif";
  ctx.fillText("スコア: " + score, canvasWidth / 2 - 40, 30);
}

function checkCollision() {
  const currentHp = hp;

  for (let i = 0; i < enemies.length; i++) {
    let e = enemies[i];
    if (
      player.x < e.x + e.width - 10 &&
      player.x + player.width > e.x + 10 &&
      player.y < e.y + e.height - 10 &&
      player.y + player.height > e.y + 10
    ) {
      if (e.isExplosive) {
        explosions.push({
          x: e.x + e.width / 2,
          y: e.y + e.height / 2,
          radius: 0,
          maxRadius: 50,
          opacity: 1,
          duration: 30,
          currentFrame: 0
        });
      }

      const damageTaken = e.damage;
      const initialHp = hp;
      const enemyType = e.type;

      if (enemyType === "enemy6") {
          hp = Math.min(MAX_HP, hp + 1);
      } else {
          hp -= damageTaken;
      }
      
      if (enemyAnimations[enemyType].statusEffect === "slowJump") {
          player.status = "slowJump";
          player.statusTimer = statusEffectDuration;
          player.jumpPower = slowJumpPower;
      }

      if (hp <= 0) {
        if (enemyType === "enemy8" && initialHp === 1) {
            lastEnemyType = "hp1_enemy8";
        } else if (enemyType === "enemy7" && initialHp === 3) {
            lastEnemyType = "strong_hp3";
        } else if (enemyType === "enemy7" && initialHp === 2) {
            lastEnemyType = "strong_hp2";
        } else if (enemyType === "enemy7" && initialHp === 1) {
            lastEnemyType = "strong_hp1";
        } else if (damageTaken >= 4 && hp <= 0) { 
            lastEnemyType = "strong_hit";
        } else {
            lastEnemyType = enemyType;
        }

        gameOver = true;
        document.getElementById("retryButton").style.display = "inline-block";
        loadGameOverImage();
      }

      enemies.splice(i, 1);
      return;
    }
  }

  for (let i = 0; i < healingItems.length; i++) {
    let item = healingItems[i];
    if (
      player.x < item.x + item.width &&
      player.x + player.width > item.x &&
      player.y < item.y + item.height &&
      player.y + player.height > item.y
    ) {
      healingItems.splice(i, 1);
      
      if (hp === 3) {
          hp = 1;
      } else {
          hp = Math.min(MAX_HP, hp + 1);
      }
      
      return;
    }
  }

  if (player.y > canvasHeight) {
    gameOver = true;
    document.getElementById("retryButton").style.display = "inline-block";

    let isFallingIntoHole = false;
    for (let h of holes) {
      if (
        player.x + player.width > h.x &&
        player.x < h.x + h.width
      ) {
        isFallingIntoHole = true;
        break;
      }
    }

    if (isFallingIntoHole) {
        lastEnemyType = "hole_fall";
    } else {
        lastEnemyType = "other_fall";
    }

    loadGameOverImage();
    return;
  }
}

function loadGameOverImage() {
    if (lastEnemyType === "hp1_enemy8") {
        gameOverImageCache = new Image();
        gameOverImageCache.onload = imageLoaded;
        gameOverImageCache.src = "gameover_hp1_enemy8.png";
        return;
    }
    if (lastEnemyType === "strong_hp3") {
        gameOverImageCache = new Image();
        gameOverImageCache.onload = imageLoaded;
        gameOverImageCache.src = "gameover_strong_hp3.png";
        return;
    }
    if (lastEnemyType === "strong_hp2") {
        gameOverImageCache = new Image();
        gameOverImageCache.onload = imageLoaded;
        gameOverImageCache.src = "gameover_strong_hp2.png";
        return;
    }
    if (lastEnemyType === "strong_hp1") {
        gameOverImageCache = new Image();
        gameOverImageCache.onload = imageLoaded;
        gameOverImageCache.src = "gameover_strong_hp1.png";
        return;
    }
    if (lastEnemyType === "strong_hit") {
        gameOverImageCache = new Image();
        gameOverImageCache.onload = imageLoaded;
        gameOverImageCache.src = "gameover_strong.png";
        return;
    }
    if (lastEnemyType === "hole_fall") {
        let imagePrefix;
        if (score <= 6000) {
            imagePrefix = "gameover_4000-6000";
        } else if (score <= 8000) {
            imagePrefix = "gameover_6001-8000";
        } else if (score <= 10000) {
            imagePrefix = "gameover_8001-10000";
        } else {
            imagePrefix = "gameover_10001-12000";
        }
        gameOverImageCache = new Image();
        gameOverImageCache.onload = imageLoaded;
        gameOverImageCache.src = `${imagePrefix}_fall.png`;
        return;
    }

    if (lastEnemyType === "other_fall") {
        gameOverImageCache = new Image();
        gameOverImageCache.onload = imageLoaded;
        gameOverImageCache.src = "gameover_fall.png";
        return;
    }

    let imagePrefix;
    if (score <= 3000) {
        imagePrefix = "gameover_0-3000";
    } else if (score <= 6000) {
        imagePrefix = "gameover_3001-6000";
    } else if (score <= 9000) {
        imagePrefix = "gameover_6001-9000";
    } else {
        imagePrefix = "gameover_9001-12000";
    }

    if (lastEnemyType) {
        gameOverImageCache = new Image();
        gameOverImageCache.onload = imageLoaded;
        gameOverImageCache.src = `${imagePrefix}_${lastEnemyType}.png`;
    }
}

function update() {
  if (!gameStarted || gameOver) {
    if (gameOver) {
        if (gameOverOpacity < 1) {
            gameOverOpacity += 0.01;
        } else {
            cancelAnimationFrame(animationId);
        }
    }
    return;
  }

  frame++;

  if (player.statusTimer > 0) {
    player.statusTimer--;
    if (player.statusTimer <= 0) {
      player.status = null;
      player.jumpPower = originalJumpPower;
    }
  }

  spawnTimer++;
  if (spawnTimer >= spawnInterval) {
    spawnObstacle();
  }

  player.vy += player.gravity;
  player.y += player.vy;

  let onGround = false;
  if (player.y + player.height >= groundY) {
    onGround = true;
    for (let h of holes) {
      if (player.x + player.width > h.x && player.x < h.x + h.width) {
        onGround = false;
        break;
      }
    }
  }

  if (onGround) {
    player.y = groundY - player.height;
    player.vy = 0;
    player.isJumping = false;
  }

  for (let i = 0; i < enemies.length; i++) {
    let e = enemies[i];
    e.x -= speed * e.speedMultiplier;
    if (e.isBouncing) {
      e.vy += e.gravity;
      e.y += e.vy;
      if (e.y >= e.groundYAdjust - e.height) {
        e.y = e.groundYAdjust - e.height;
        e.vy = e.jumpPower;
      }
    }
  }

  for (let i = 0; i < healingItems.length; i++) {
    healingItems[i].x -= speed;
  }

  for (let i = 0; i < explosions.length; i++) {
    let exp = explosions[i];
    exp.currentFrame++;
    exp.radius = exp.maxRadius * (exp.currentFrame / exp.duration);
    exp.opacity = 1 - (exp.currentFrame / exp.duration);
    if (exp.currentFrame >= exp.duration) {
      explosions.splice(i, 1);
      i--;
    }
  }

  for (let i = 0; i < holes.length; i++) {
    holes[i].x -= speed;
  }

  enemies = enemies.filter(e => e.x + e.width > 0);
  holes = holes.filter(h => h.x + h.width > 0);
  healingItems = healingItems.filter(item => item.x + item.width > 0);

  score++;
  updateSkyColor();
  if (score % 200 === 0 && speed < 7) speed++;
  checkCollision();
}

function drawInitialScreen() {
    if (startScreenImage.complete) {
        ctx.drawImage(startScreenImage, 0, 0, canvasWidth, canvasHeight);
    } else {
        ctx.fillStyle = "#87CEEB";
        ctx.fillRect(0, 0, canvasWidth, canvasHeight);
        ctx.fillStyle = "#654321";
        ctx.fillRect(0, groundY, canvasWidth, groundHeight);
    }
}

function draw() {
  if (!gameStarted) {
    drawInitialScreen();
    return;
  }

  drawBackground();
  drawGround();
  drawPlayer();
  drawEnemies();
  drawHealingItems();
  drawExplosions();
  drawHearts();
  drawScore();

  if (gameOver) {
    ctx.fillStyle = `rgba(0, 0, 0, ${gameOverOpacity * 0.4})`;
    ctx.fillRect(0, 0, canvasWidth, canvasHeight);

    ctx.save();
    ctx.globalAlpha = gameOverOpacity;
    if (gameOverImageCache && gameOverImageCache.complete) {
        const imageWidth = gameOverImageCache.width;
        const imageHeight = gameOverImageCache.height;
        const canvasAspectRatio = canvasWidth / canvasHeight;
        const imageAspectRatio = imageWidth / imageHeight;
        let drawWidth, drawHeight, offsetX, offsetY;
        
        if (imageAspectRatio > canvasAspectRatio) {
            drawWidth = imageWidth * (canvasHeight / imageHeight);
            drawHeight = canvasHeight;
            offsetX = (canvasWidth - drawWidth) / 2;
            offsetY = 0;
        } else {
            drawWidth = canvasWidth;
            drawHeight = imageHeight * (canvasWidth / imageWidth);
            offsetX = 0;
            offsetY = (canvasHeight - drawHeight) / 2;
        }
        ctx.drawImage(gameOverImageCache, offsetX, offsetY, drawWidth, drawHeight);
    }
    ctx.restore();

    ctx.save();
    ctx.globalAlpha = gameOverOpacity * 0.7;
    ctx.fillStyle = "red";
    ctx.font = "bold 40px sans-serif";
    ctx.textAlign = "center";
    ctx.fillText("GAME OVER", canvasWidth / 2, canvasHeight - 50);

    ctx.fillStyle = "white";
    ctx.font = "bold 24px sans-serif";
    ctx.fillText(`最終スコア: ${score}`, canvasWidth / 2, canvasHeight - 90);
    ctx.restore();
  }
}

let inputBuffer = "";

function loop() {
  update();
  draw();
  animationId = requestAnimationFrame(loop);
}

function startGame(startScore = 0) {
  if (gameStarted) return;
  
  score = startScore;
  gameStarted = true;
  document.getElementById("startButton").style.display = "none";
  speed = 3;
  
  if (animationId) {
      cancelAnimationFrame(animationId);
  }
  loop();
}

document.addEventListener("keydown", e => {
  if (gameStarted) {
    if (e.code === "Space" && !player.isJumping && !gameOver) {
      player.vy = player.jumpPower;
      player.isJumping = true;
    }
  } else {
    if (e.key.length === 1 && /[a-zA-Z0-9]/.test(e.key)) {
        inputBuffer += e.key.toLowerCase();
    } else if (e.key === "Enter") {
        const match = inputBuffer.match(/^test(\d+)$/);
        if (match) {
            const scoreValue = parseInt(match[1], 10);
            startGame(scoreValue);
        }
        inputBuffer = "";
    } else if (e.key === "Backspace" || e.key === "Delete") {
      inputBuffer = inputBuffer.slice(0, -1);
    }
  }
});

canvas.addEventListener("touchstart", () => {
  if (gameStarted && !gameOver && !player.isJumping) {
    player.vy = player.jumpPower;
    player.isJumping = true;
  }
});

function restartGame() {
  if (animationId) {
      cancelAnimationFrame(animationId);
  }
  
  enemies = [];
  explosions = [];
  holes = [];
  healingItems = [];
  score = 0;
  hp = 3;
  gameOver = false;
  speed = 3;
  skyPhase = 0;
  targetSkyColor = skyColor = "#87CEEB";
  lastEnemyType = null;
  gameOverImageCache = null;
  gameOverOpacity = 0;
  player.y = groundY - player.height;
  player.vy = 0;
  player.isJumping = false;
  player.status = null;
  player.statusTimer = 0;
  player.jumpPower = originalJumpPower;
  document.getElementById("retryButton").style.display = "none";
  document.getElementById("startButton").style.display = "inline-block";
  gameStarted = false;
  drawInitialScreen();
}

window.onload = () => {
    drawInitialScreen();
};

</script>
</body>
</html>
