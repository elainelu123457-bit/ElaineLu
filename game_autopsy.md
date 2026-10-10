# Piano Tiles - Code Autopsy

## Game Overview

For this activity, I made a simple **Piano Tiles** game using HTML, CSS, and JavaScript. The game has four lanes controlled by **F, G, H, and J**. Tiles fall down the screen, and the player has to press the matching key while the tile is inside the hit zone. The game gradually gets faster, and sometimes two tiles appear at the same time.

## HTML, CSS, and JavaScript

My game is all contained in **one HTML file**, but it uses three different parts:

- **HTML** creates the webpage and canvas where the game appears.
- **CSS** controls how the game looks.
- **JavaScript** controls the actual game, including movement, keyboard input, scoring, lives, and game-over logic.

Most of the actual game mechanics are written in JavaScript.

---

## 1. The Canvas

```html
<canvas id="game" width="600" height="700"></canvas>
```

The canvas is the area where the game is drawn. Mine is **600 pixels wide and 700 pixels tall**.

JavaScript connects to it using:

```javascript
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
```

`ctx` lets JavaScript draw shapes and text on the canvas.

If I changed the width or height, the visible game area would become bigger or smaller. I might also have to change some positions so everything still lines up correctly.

---

## 2. State Variables

```javascript
let tiles = [];
let score = 0;
let lives = 5;
let speed = 5.5;
let timer = 0;
let playing = false;
let gameOver = false;
```

These variables keep track of the current state of the game.

- `tiles` stores the tiles currently on the screen.
- `score` stores how many tiles I have hit.
- `lives` stores how many mistakes I have left.
- `speed` controls how fast the tiles fall.
- `timer` controls when new tiles appear.
- `playing` keeps track of whether the game is running.
- `gameOver` keeps track of whether the player has lost.

---

## 3. Tiles and Movement

Each tile stores its lane and y-position:

```javascript
tiles.push({
    lane: lane1,
    y: 250
});
```

`lane` determines which column the tile is in, while `y` determines how far down the screen it is.

The tiles move with:

```javascript
tiles[i].y += speed;
```

Every time the game updates, the y-position increases, moving the tile downward.

The game also gradually gets faster:

```javascript
speed += 0.002;
```

---

## 4. The Game Loop

```javascript
function gameLoop() {
    update();
    draw();
    requestAnimationFrame(gameLoop);
}

gameLoop();
```

`gameLoop()` is the main function that runs repeatedly.

- `update()` moves the tiles and checks the game rules.
- `draw()` redraws the game.
- `requestAnimationFrame(gameLoop)` tells the browser to run the loop again on the next frame.

The basic process is:

**Update → Draw → Next Frame → Repeat**

---

## 5. Input Handling

The controls are:

```javascript
const keys = ["f", "g", "h", "j"];
```

JavaScript detects keyboard input with:

```javascript
document.addEventListener("keydown", function(event) {
    let key = event.key.toLowerCase();
    let lane = keys.indexOf(key);

    if (lane !== -1) {
        pressLane(lane);
    }
});
```

Each key represents a lane:

- F = lane 0
- G = lane 1
- H = lane 2
- J = lane 3

When I press one of these keys, the program runs `pressLane()` for that lane.

---

## 6. Hit Zone / Collision Detection

The hit zone is defined with:

```javascript
const hitY = 540;
const hitHeight = 110;
```

The program checks whether a tile overlaps the hit zone:

```javascript
let inHitZone =
    tile.y + tileHeight >= hitY &&
    tile.y <= hitY + hitHeight;
```

It then checks whether the player pressed the correct lane:

```javascript
if (tile.lane === lane && inHitZone) {
    tiles.splice(i, 1);
    score++;
    return;
}
```

This means a successful hit requires both the **correct lane** and the **correct timing**.

---

## 7. Scoring

The score starts at:

```javascript
let score = 0;
```

When I correctly hit a tile:

```javascript
tiles.splice(i, 1);
score++;
```

`tiles.splice(i, 1)` removes the tile and `score++` adds one point.

---

## 8. Lives and Game Over

The game starts with five lives:

```javascript
let lives = 5;
```

When I make a mistake:

```javascript
function loseLife() {
    lives--;

    if (lives <= 0) {
        playing = false;
        gameOver = true;
    }
}
```

`lives--` subtracts one life.

When the number of lives reaches zero, the game ends.

---

## 9. Double Tiles

Sometimes the game creates two tiles at once:

```javascript
if (Math.random() < 0.60) {
```

`Math.random()` generates a random number between 0 and 1. This gives about a **60% chance** of creating a second tile.

The program makes sure the second tile is in a different lane:

```javascript
while (lane2 === lane1) {
    lane2 = Math.floor(Math.random() * 4);
}
```

---

## 10. My Deliberate Change - Five Lives

Originally, one missed tile or wrong key immediately ended the game.

I changed the code by adding:

```javascript
let lives = 5;
```

I also added:

```javascript
function loseLife() {
    lives--;

    if (lives <= 0) {
        playing = false;
        gameOver = true;
    }
}
```

### What I Observed

Before my change:

**1 mistake → Game Over**

After my change:

**Mistake → Lose 1 life → Keep playing → Game Over after 5 mistakes**

This showed me that changing a state variable and the game-over logic can make a big difference in how the game feels to play.

---

## Three Important Parts of My Code

### 1. The Canvas
The HTML canvas creates the **600 x 700 pixel area** where the game is displayed.

### 2. The Hit Zone
The hit-zone comparison checks whether a tile is in the correct position when I press its corresponding key.

### 3. The Game Loop
`gameLoop()` continuously updates and redraws the game, creating the movement and gameplay.

Overall, **HTML provides the game area, CSS controls its appearance, and JavaScript controls the actual game mechanics.**
