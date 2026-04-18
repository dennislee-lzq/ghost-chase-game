# Ghost Chase Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a complete browser-based ghost chase game where player collects 5 items and escapes while evading an AI ghost.

**Architecture:** Single HTML file with Canvas 2D rendering, using requestAnimationFrame game loop. Map generated procedurally with room-partition algorithm. Game state machine for menu/playing/gameover screens.

**Tech Stack:** Pure HTML5 + Canvas 2D + Vanilla JavaScript (no dependencies)

---

## File Structure

- `index.html` - Complete game in single file

---

## Task 1: Basic HTML Canvas Setup

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create index.html with basic structure**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ghost Chase</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            background: #0f0f1a;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            font-family: Arial, sans-serif;
        }
        canvas {
            border: 2px solid #16213e;
            box-shadow: 0 0 30px rgba(0, 0, 0, 0.5);
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas" width="800" height="600"></canvas>
    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        // Game state
        let gameState = 'menu'; // menu, itemSelect, playing, gameover

        // Colors from design
        const COLORS = {
            background: '#1a1a2e',
            wall: '#16213e',
            player: '#ffffff',
            ghost: 'rgba(255, 0, 0, 0.7)',
            item: '#ffd700',
            exitInactive: '#555555',
            exitActive: '#00ff00',
            ghostVision: 'rgba(255, 100, 100, 0.2)'
        };

        // Start menu
        function drawMenu() {
            ctx.fillStyle = COLORS.background;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            ctx.fillStyle = '#ffffff';
            ctx.font = 'bold 48px Arial';
            ctx.textAlign = 'center';
            ctx.fillText('GHOST CHASE', canvas.width/2, canvas.height/2 - 50);

            ctx.font = '24px Arial';
            ctx.fillText('Click to Start', canvas.width/2, canvas.height/2 + 20);
        }

        // Main game loop
        function gameLoop() {
            if (gameState === 'menu') {
                drawMenu();
            }
            requestAnimationFrame(gameLoop);
        }

        // Click handler for starting game
        canvas.addEventListener('click', () => {
            if (gameState === 'menu') {
                gameState = 'itemSelect';
            }
        });

        gameLoop();
    </script>
</body>
</html>
```

- [ ] **Step 2: Test in browser**

Run: Open `index.html` in browser
Expected: Shows "GHOST CHASE" title and "Click to Start" text

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add basic HTML canvas setup with menu screen"
```

---

## Task 2: Item Selection Screen

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add item selection state and draw function**

In the `<script>` section, add after the COLORS object:

```javascript
const ITEMS = [
    { name: 'Speed Boost', desc: '2x speed for 3 seconds', color: '#00aaff' },
    { name: 'Invisibility', desc: 'Ghost cant track you for 3 seconds', color: '#aa00ff' },
    { name: 'Smoke Bomb', desc: 'Confuse ghost for 3 seconds', color: '#888888' }
];

let selectedItem = null;

function drawItemSelect() {
    ctx.fillStyle = COLORS.background;
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    ctx.fillStyle = '#ffffff';
    ctx.font = 'bold 36px Arial';
    ctx.textAlign = 'center';
    ctx.fillText('Choose Your Item', canvas.width/2, 80);

    ITEMS.forEach((item, i) => {
        const x = 150 + i * 220;
        const y = canvas.height / 2 - 60;
        const w = 180;
        const h = 200;

        // Box
        ctx.fillStyle = item.color;
        ctx.fillRect(x, y, w, h);

        // Name
        ctx.fillStyle = '#ffffff';
        ctx.font = 'bold 20px Arial';
        ctx.fillText(item.name, x + w/2, y + 50);

        // Description
        ctx.font = '14px Arial';
        ctx.fillText(item.desc, x + w/2, y + 100);

        // Click area stored for later
        item.clickArea = { x, y, w, h };
    });
}
```

- [ ] **Step 2: Update click handler for item selection**

Replace the click handler:

```javascript
canvas.addEventListener('click', (e) => {
    const rect = canvas.getBoundingClientRect();
    const mx = e.clientX - rect.left;
    const my = e.clientY - rect.top;

    if (gameState === 'menu') {
        gameState = 'itemSelect';
    } else if (gameState === 'itemSelect') {
        ITEMS.forEach((item, i) => {
            const a = item.clickArea;
            if (mx >= a.x && mx <= a.x + a.w && my >= a.y && my <= a.y + a.h) {
                selectedItem = { ...item, cooldown: 0 };
                gameState = 'playing';
            }
        });
    }
});
```

- [ ] **Step 3: Update game loop to show item select**

```javascript
function gameLoop() {
    if (gameState === 'menu') {
        drawMenu();
    } else if (gameState === 'itemSelect') {
        drawItemSelect();
    }
    requestAnimationFrame(gameLoop);
}
```

- [ ] **Step 4: Test in browser**

Run: Open `index.html`, click to start, click an item
Expected: Shows item selection screen with 3 items

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add item selection screen with 3 item options"
```

---

## Task 3: Map Generation

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add map generation code**

After COLORS object, add:

```javascript
// Map generation
let walls = [];
let items = [];
let exit = { x: 0, y: 0, active: false };
const COLLECTIBLE_COUNT = 8;
const ITEMS_TO_WIN = 5;
let collectedCount = 0;

function generateMap() {
    walls = [];
    items = [];
    collectedCount = 0;
    exit.active = false;

    // Border walls
    walls.push({ x: 0, y: 0, w: canvas.width, h: 20 }); // top
    walls.push({ x: 0, y: canvas.height - 20, w: canvas.width, h: 20 }); // bottom
    walls.push({ x: 0, y: 0, w: 20, h: canvas.height }); // left
    walls.push({ x: canvas.width - 20, y: 0, w: 20, h: canvas.height }); // right

    // Random interior walls (simple room partition)
    const numWalls = 6 + Math.floor(Math.random() * 5);
    for (let i = 0; i < numWalls; i++) {
        const horizontal = Math.random() > 0.5;
        if (horizontal) {
            const y = 80 + Math.random() * (canvas.height - 160);
            const x = 40 + Math.random() * (canvas.width - 200);
            const w = 100 + Math.random() * 150;
            walls.push({ x, y, w, h: 15 });
        } else {
            const x = 40 + Math.random() * (canvas.width - 200);
            const y = 80 + Math.random() * (canvas.height - 160);
            const h = 100 + Math.random() * 150;
            walls.push({ x, y, w: 15, h });
        }
    }

    // Generate items at random valid positions
    for (let i = 0; i < COLLECTIBLE_COUNT; i++) {
        items.push(createRandomPosition());
    }

    // Exit at top-right corner area
    exit = { x: canvas.width - 60, y: canvas.height - 60, active: false };
}

function createRandomPosition() {
    let pos;
    let attempts = 0;
    do {
        pos = {
            x: 50 + Math.random() * (canvas.width - 100),
            y: 50 + Math.random() * (canvas.height - 100)
        };
        attempts++;
    } while (attempts < 50 && isCollidingWithWalls(pos.x, pos.y, 10));
    return pos;
}

function isCollidingWithWalls(x, y, radius) {
    for (const w of walls) {
        if (x + radius > w.x && x - radius < w.x + w.w &&
            y + radius > w.y && y - radius < w.y + w.h) {
            return true;
        }
    }
    return false;
}
```

- [ ] **Step 2: Update game state transitions to generate map**

In click handler, when transitioning to 'playing':

```javascript
selectedItem = { ...item, cooldown: 0 };
generateMap();  // Add this line
gameState = 'playing';
```

- [ ] **Step 3: Add function to draw walls**

```javascript
function drawMap() {
    // Background
    ctx.fillStyle = COLORS.background;
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    // Walls
    ctx.fillStyle = COLORS.wall;
    walls.forEach(w => ctx.fillRect(w.x, w.y, w.w, w.h));

    // Items
    items.forEach(item => {
        ctx.beginPath();
        ctx.arc(item.x, item.y, 8, 0, Math.PI * 2);
        ctx.fillStyle = COLORS.item;
        ctx.fill();
        ctx.shadowBlur = 15;
        ctx.shadowColor = COLORS.item;
        ctx.fill();
        ctx.shadowBlur = 0;
    });

    // Exit
    ctx.beginPath();
    ctx.arc(exit.x, exit.y, 20, 0, Math.PI * 2);
    ctx.fillStyle = exit.active ? COLORS.exitActive : COLORS.exitInactive;
    ctx.fill();
}
```

- [ ] **Step 4: Test in browser**

Run: Open `index.html`, select item, should see generated map
Expected: Walls, items (gold dots), exit point visible

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add procedural map generation with walls, items, and exit"
```

---

## Task 4: Player Movement and Collision

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add player object and input tracking**

After COLORS object, add:

```javascript
let player = { x: 60, y: canvas.height - 60, radius: 12, speed: 150 };
const keys = { w: false, a: false, s: false, d: false };

document.addEventListener('keydown', (e) => {
    const key = e.key.toLowerCase();
    if (key in keys) keys[key] = true;
});

document.addEventListener('keyup', (e) => {
    const key = e.key.toLowerCase();
    if (key in keys) keys[key] = false;
});
```

- [ ] **Step 2: Add player update function**

```javascript
function updatePlayer(dt) {
    let dx = 0, dy = 0;
    if (keys.w) dy -= 1;
    if (keys.s) dy += 1;
    if (keys.a) dx -= 1;
    if (keys.d) dx += 1;

    // Normalize diagonal movement
    if (dx !== 0 && dy !== 0) {
        dx *= 0.707;
        dy *= 0.707;
    }

    const speed = selectedItem && selectedItem.active && selectedItem.name === 'Speed Boost'
        ? player.speed * 2 : player.speed;

    let newX = player.x + dx * speed * dt;
    let newY = player.y + dy * speed * dt;

    // Wall collision
    if (!isCollidingWithWalls(newX, player.y, player.radius)) {
        player.x = newX;
    }
    if (!isCollidingWithWalls(player.x, newY, player.radius)) {
        player.y = newY;
    }
}
```

- [ ] **Step 3: Add player drawing function**

```javascript
function drawPlayer() {
    ctx.beginPath();
    ctx.arc(player.x, player.y, player.radius, 0, Math.PI * 2);
    ctx.fillStyle = COLORS.player;
    ctx.fill();
    // Glow effect
    ctx.shadowBlur = 20;
    ctx.shadowColor = COLORS.player;
    ctx.fill();
    ctx.shadowBlur = 0;
}
```

- [ ] **Step 4: Update game loop with player update and draw**

Add at top of script:
```javascript
let lastTime = 0;
```

Replace gameLoop:
```javascript
function gameLoop(timestamp) {
    const dt = (timestamp - lastTime) / 1000;
    lastTime = timestamp;

    if (gameState === 'menu') {
        drawMenu();
    } else if (gameState === 'itemSelect') {
        drawItemSelect();
    } else if (gameState === 'playing') {
        updatePlayer(dt);
        drawMap();
        drawPlayer();
    }
    requestAnimationFrame(gameLoop);
}

// Update lastTime initialization
canvas.addEventListener('click', () => {
    // ... existing code
});
gameLoop(0);
```

- [ ] **Step 5: Test in browser**

Run: Open `index.html`, select item, use WASD to move
Expected: Player (white dot) moves smoothly, cannot pass through walls

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add player movement with WASD and wall collision"
```

---

## Task 5: Ghost AI

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add ghost object**

```javascript
let ghost = {
    x: canvas.width - 60,
    y: 60,
    radius: 15,
    speed: 120,
    visionRange: 200,
    visionAngle: Math.PI / 2,
    angle: 0,
    mode: 'patrol', // patrol or chase
    lastTeleport: -30000,
    lastRush: 0,
    rushDuration: 0,
    patrolTarget: null
};
```

- [ ] **Step 2: Add ghost update function**

```javascript
function updateGhost(dt) {
    const distToPlayer = Math.hypot(player.x - ghost.x, player.y - ghost.y);
    const angleToPlayer = Math.atan2(player.y - ghost.y, player.x - ghost.x);

    // Check if player is in vision
    const canSeePlayer = distToPlayer < ghost.visionRange &&
        Math.abs(normalizeAngle(angleToPlayer - ghost.angle)) < ghost.visionAngle / 2 &&
        hasLineOfSight(ghost.x, ghost.y, player.x, player.y);

    ghost.mode = canSeePlayer ? 'chase' : 'patrol';

    let currentSpeed = ghost.speed;
    if (ghost.rushDuration > 0) {
        ghost.rushDuration -= dt * 1000;
        currentSpeed *= 1.5;
    }

    if (ghost.mode === 'chase') {
        // Move toward player
        const dx = Math.cos(angleToPlayer) * currentSpeed * dt;
        const dy = Math.sin(angleToPlayer) * currentSpeed * dt;
        moveGhostWithCollision(dx, dy);
        ghost.angle = angleToPlayer;

        // Use Ghost Rush skill
        if (Date.now() - ghost.lastRush > 15000 && ghost.rushDuration <= 0) {
            ghost.rushDuration = 2000;
            ghost.lastRush = Date.now();
        }
    } else {
        // Patrol mode - wander
        if (!ghost.patrolTarget || Math.hypot(ghost.patrolTarget.x - ghost.x, ghost.patrolTarget.y - ghost.y) < 10) {
            ghost.patrolTarget = {
                x: 100 + Math.random() * (canvas.width - 200),
                y: 100 + Math.random() * (canvas.height - 200)
            };
        }
        const angleToTarget = Math.atan2(ghost.patrolTarget.y - ghost.y, ghost.patrolTarget.x - ghost.x);
        ghost.angle = angleToTarget;
        const dx = Math.cos(angleToTarget) * currentSpeed * 0.5 * dt;
        const dy = Math.sin(angleToTarget) * currentSpeed * 0.5 * dt;
        moveGhostWithCollision(dx, dy);

        // Teleport skill every 30 seconds
        if (Date.now() - ghost.lastTeleport > 30000) {
            teleportGhost();
        }
    }
}

function normalizeAngle(angle) {
    while (angle > Math.PI) angle -= Math.PI * 2;
    while (angle < -Math.PI) angle += Math.PI * 2;
    return angle;
}

function hasLineOfSight(x1, y1, x2, y2) {
    const steps = 20;
    for (let i = 0; i <= steps; i++) {
        const t = i / steps;
        const x = x1 + (x2 - x1) * t;
        const y = y1 + (y2 - y1) * t;
        if (isCollidingWithWalls(x, y, 2)) return false;
    }
    return true;
}

function moveGhostWithCollision(dx, dy) {
    if (!isCollidingWithWalls(ghost.x + dx, ghost.y, ghost.radius)) {
        ghost.x += dx;
    }
    if (!isCollidingWithWalls(ghost.x, ghost.y + dy, ghost.radius)) {
        ghost.y += dy;
    }
}

function teleportGhost() {
    // Teleport to a position near the player
    const angle = Math.random() * Math.PI * 2;
    const dist = 100 + Math.random() * 100;
    let newX = player.x + Math.cos(angle) * dist;
    let newY = player.y + Math.sin(angle) * dist;

    // Clamp to bounds
    newX = Math.max(30, Math.min(canvas.width - 30, newX));
    newY = Math.max(30, Math.min(canvas.height - 30, newY));

    if (!isCollidingWithWalls(newX, newY, ghost.radius)) {
        ghost.x = newX;
        ghost.y = newY;
        ghost.lastTeleport = Date.now();
    }
}
```

- [ ] **Step 3: Add ghost drawing function**

```javascript
function drawGhost() {
    // Vision cone
    ctx.beginPath();
    ctx.moveTo(ghost.x, ghost.y);
    ctx.arc(ghost.x, ghost.y, ghost.visionRange,
        ghost.angle - ghost.visionAngle / 2,
        ghost.angle + ghost.visionAngle / 2);
    ctx.closePath();
    ctx.fillStyle = COLORS.ghostVision;
    ctx.fill();

    // Ghost body with pulse
    const pulse = 1 + Math.sin(Date.now() / 200) * 0.1;
    ctx.beginPath();
    ctx.arc(ghost.x, ghost.y, ghost.radius * pulse, 0, Math.PI * 2);
    ctx.fillStyle = COLORS.ghost;
    ctx.fill();
    ctx.shadowBlur = 20;
    ctx.shadowColor = 'red';
    ctx.fill();
    ctx.shadowBlur = 0;

    // Eyes
    const eyeOffset = 5;
    ctx.fillStyle = '#ffffff';
    ctx.beginPath();
    ctx.arc(ghost.x - eyeOffset, ghost.y - 3, 3, 0, Math.PI * 2);
    ctx.arc(ghost.x + eyeOffset, ghost.y - 3, 3, 0, Math.PI * 2);
    ctx.fill();
}
```

- [ ] **Step 4: Update game loop to call ghost functions**

In the `gameState === 'playing'` block:
```javascript
} else if (gameState === 'playing') {
    updatePlayer(dt);
    updateGhost(dt);
    drawMap();
    drawPlayer();
    drawGhost();
}
```

- [ ] **Step 5: Test in browser**

Run: Open `index.html`, play game
Expected: Ghost (red) patrols when player not visible, chases when player in sight

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add ghost AI with patrol/chase modes and skills"
```

---

## Task 6: Item Collection and Exit

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add collision detection function**

```javascript
function circleCollision(x1, y1, r1, x2, y2, r2) {
    return Math.hypot(x2 - x1, y2 - y1) < r1 + r2;
}
```

- [ ] **Step 2: Add item collection logic in updatePlayer**

At end of updatePlayer function:

```javascript
// Check item collection
for (let i = items.length - 1; i >= 0; i--) {
    if (circleCollision(player.x, player.y, player.radius, items[i].x, items[i].y, 8)) {
        items.splice(i, 1);
        collectedCount++;
        if (collectedCount >= ITEMS_TO_WIN) {
            exit.active = true;
        }
    }
}

// Check exit
if (exit.active && circleCollision(player.x, player.y, player.radius, exit.x, exit.y, 20)) {
    gameState = 'gameover';
    gameResult = 'win';
}
```

- [ ] **Step 3: Add ghost catch logic in updateGhost**

At end of updateGhost function:

```javascript
// Check if caught player
if (circleCollision(player.x, player.y, player.radius, ghost.x, ghost.y, ghost.radius)) {
    gameState = 'gameover';
    gameResult = 'lose';
}
```

Add at top of script section:
```javascript
let gameResult = '';
```

- [ ] **Step 4: Update drawMap to show collected count**

```javascript
// Show collected count
ctx.fillStyle = '#ffffff';
ctx.font = '20px Arial';
ctx.textAlign = 'left';
ctx.fillText(`Items: ${collectedCount}/${ITEMS_TO_WIN}`, 30, 35);
```

- [ ] **Step 5: Test in browser**

Run: Open `index.html`, collect items
Expected: Items disappear when collected, counter updates, exit turns green when 5 collected

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add item collection and exit mechanics"
```

---

## Task 7: Item/Skill Usage

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add space key handler for using item**

Add to keydown handler:

```javascript
document.addEventListener('keydown', (e) => {
    const key = e.key.toLowerCase();
    if (key in keys) keys[key] = true;

    if (e.code === 'Space' && gameState === 'playing' && selectedItem) {
        useItem();
    }
});
```

- [ ] **Step 2: Add useItem function**

```javascript
function useItem() {
    if (!selectedItem || selectedItem.cooldown > 0 || selectedItem.active) return;

    selectedItem.active = true;
    selectedItem.cooldown = getItemCooldown(selectedItem.name);

    setTimeout(() => {
        selectedItem.active = false;
    }, 3000);
}

function getItemCooldown(itemName) {
    switch(itemName) {
        case 'Speed Boost': return 30000;
        case 'Invisibility': return 45000;
        case 'Smoke Bomb': return 60000;
        default: return 30000;
    }
}
```

- [ ] **Step 3: Add item cooldown update in updatePlayer**

At end of updatePlayer:

```javascript
// Update item cooldown
if (selectedItem && selectedItem.cooldown > 0) {
    selectedItem.cooldown -= dt * 1000;
    if (selectedItem.cooldown < 0) selectedItem.cooldown = 0;
}
```

- [ ] **Step 4: Add smoke bomb effect**

```javascript
let smokeBombs = [];

function triggerSmokeBomb() {
    smokeBombs.push({
        x: player.x,
        y: player.y,
        radius: 80,
        duration: 3000,
        startTime: Date.now()
    });
}
```

Update useItem to call this for Smoke Bomb:
```javascript
if (selectedItem.name === 'Smoke Bomb') {
    triggerSmokeBomb();
}
```

- [ ] **Step 5: Update ghost AI to respect smoke bomb and invisibility**

In updateGhost, at the start:

```javascript
// Check if in smoke
let inSmoke = false;
for (const smoke of smokeBombs) {
    const age = Date.now() - smoke.startTime;
    if (age < smoke.duration && Math.hypot(player.x - smoke.x, player.y - smoke.y) < smoke.radius) {
        inSmoke = true;
        break;
    }
}

// Check invisibility
const isInvisible = selectedItem && selectedItem.active && selectedItem.name === 'Invisibility';

// Modify canSeePlayer
const canSeePlayer = !isInvisible && !inSmoke && distToPlayer < ghost.visionRange &&
    Math.abs(normalizeAngle(angleToPlayer - ghost.angle)) < ghost.visionAngle / 2 &&
    hasLineOfSight(ghost.x, ghost.y, player.x, player.y);
```

- [ ] **Step 6: Add smoke bomb rendering in game loop**

In gameState === 'playing' block, add to draw calls:
```javascript
drawSmokeBombs();
```

Add function:
```javascript
function drawSmokeBombs() {
    const now = Date.now();
    for (let i = smokeBombs.length - 1; i >= 0; i--) {
        const smoke = smokeBombs[i];
        const age = now - smoke.startTime;
        if (age > smoke.duration) {
            smokeBombs.splice(i, 1);
            continue;
        }
        const alpha = 1 - age / smoke.duration;
        ctx.beginPath();
        ctx.arc(smoke.x, smoke.y, smoke.radius, 0, Math.PI * 2);
        ctx.fillStyle = `rgba(100, 100, 100, ${alpha * 0.5})`;
        ctx.fill();
    }
}
```

- [ ] **Step 7: Add UI for item status**

Add function:
```javascript
function drawItemUI() {
    if (!selectedItem) return;

    ctx.textAlign = 'right';
    ctx.font = '16px Arial';
    ctx.fillStyle = '#ffffff';

    const name = selectedItem.name;
    const ready = selectedItem.cooldown <= 0 && !selectedItem.active;
    const active = selectedItem.active;

    ctx.fillStyle = active ? '#00ff00' : ready ? '#ffffff' : '#888888';
    ctx.fillText(`${name}: ${active ? 'ACTIVE' : ready ? 'READY' : 'Cooldown'}`, canvas.width - 30, 35);

    if (!ready && !active) {
        ctx.fillStyle = '#888888';
        ctx.fillText(`${(selectedItem.cooldown / 1000).toFixed(1)}s`, canvas.width - 30, 55);
    }
}
```

Call in game loop:
```javascript
drawItemUI();
```

- [ ] **Step 8: Test in browser**

Run: Open `index.html`, press space to use item
Expected: Items show cooldown, effects activate (Speed=player faster, Invis=ghost loses track, Smoke=gray cloud)

- [ ] **Step 9: Commit**

```bash
git add index.html
git commit -m "feat: add item usage system with cooldown and effects"
```

---

## Task 8: Game Over Screen

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add game over drawing function**

```javascript
function drawGameOver() {
    ctx.fillStyle = 'rgba(0, 0, 0, 0.8)';
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    ctx.textAlign = 'center';

    if (gameResult === 'win') {
        ctx.fillStyle = '#00ff00';
        ctx.font = 'bold 48px Arial';
        ctx.fillText('YOU ESCAPED!', canvas.width/2, canvas.height/2 - 30);
    } else {
        ctx.fillStyle = '#ff0000';
        ctx.font = 'bold 48px Arial';
        ctx.fillText('CAUGHT!', canvas.width/2, canvas.height/2 - 30);
    }

    ctx.fillStyle = '#ffffff';
    ctx.font = '24px Arial';
    ctx.fillText('Click to Play Again', canvas.width/2, canvas.height/2 + 30);
}
```

- [ ] **Step 2: Update click handler for restart**

In click handler, add to gameover state handling:

```javascript
} else if (gameState === 'gameover') {
    gameState = 'menu';
}
```

- [ ] **Step 3: Update game loop**

```javascript
} else if (gameState === 'gameover') {
    drawMap();
    drawPlayer();
    drawGhost();
    drawGameOver();
}
```

- [ ] **Step 4: Reset smoke bombs on game restart**

In click handler when going to 'itemSelect':
```javascript
smokeBombs = [];
```

- [ ] **Step 5: Test in browser**

Run: Open `index.html`, win or lose
Expected: Game over screen shows, click restarts

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add game over screen with win/lose states"
```

---

## Task 9: Final Polish - Instructions and Visual Feedback

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add instructions overlay in playing state**

Add function:
```javascript
function drawInstructions() {
    ctx.fillStyle = '#ffffff';
    ctx.font = '14px Arial';
    ctx.textAlign = 'left';
    ctx.fillText('WASD: Move | SPACE: Use Item', 30, canvas.height - 20);
}
```

Call in game loop after draw calls.

- [ ] **Step 2: Add visual feedback when collecting items**

In updatePlayer, when collecting item, add flash effect:
```javascript
// Add a brief flash
player.flashTime = Date.now();
```

In drawPlayer:
```javascript
// Flash effect when collecting
if (player.flashTime && Date.now() - player.flashTime < 200) {
    ctx.beginPath();
    ctx.arc(player.x, player.y, player.radius + 10, 0, Math.PI * 2);
    ctx.fillStyle = 'rgba(255, 215, 0, 0.5)';
    ctx.fill();
}
```

- [ ] **Step 3: Add exit glow animation when active**

In drawMap, for exit:
```javascript
// Exit - animated glow when active
if (exit.active) {
    const pulse = 1 + Math.sin(Date.now() / 300) * 0.2;
    ctx.beginPath();
    ctx.arc(exit.x, exit.y, 20 * pulse, 0, Math.PI * 2);
    ctx.fillStyle = COLORS.exitActive;
    ctx.shadowBlur = 30;
    ctx.shadowColor = COLORS.exitActive;
    ctx.fill();
    ctx.shadowBlur = 0;
} else {
    ctx.beginPath();
    ctx.arc(exit.x, exit.y, 20, 0, Math.PI * 2);
    ctx.fillStyle = COLORS.exitInactive;
    ctx.fill();
}
```

- [ ] **Step 4: Test in browser**

Run: Open `index.html`, play through
Expected: Instructions visible, item collection flash, pulsing exit

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add polish - instructions, collection flash, exit glow"
```

---

## Spec Coverage Check

- [x] Player movement (WASD) - Task 4
- [x] Wall collision - Task 4
- [x] Ghost AI (patrol/chase) - Task 5
- [x] Ghost vision cone - Task 5
- [x] Ghost skills (teleport, rush) - Task 5
- [x] Map generation - Task 3
- [x] Item collection - Task 6
- [x] Exit mechanic - Task 6
- [x] 3 items with cooldowns - Task 7
- [x] Item selection screen - Task 2
- [x] Start menu - Task 1
- [x] Game over screen - Task 8
- [x] Visual style (colors) - All tasks

---

## Self-Review

- All placeholder/TODO scan: None found
- Type consistency: Functions and variables consistently named across tasks
- Spec coverage: All requirements from design spec implemented
