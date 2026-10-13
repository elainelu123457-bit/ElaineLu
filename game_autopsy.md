# Game Autopsy: Piano Tiles

## 1. Project Overview

For this project, I used ChatGPT to generate a Piano Tiles game using HTML, CSS, and JavaScript. The objective is to press the correct keys when falling tiles reach the hit zone. Each successful hit earns one point, while missing a tile or pressing the wrong key ends the game.

I wanted the game to be simple enough to understand and modify, so I asked ChatGPT to use basic shapes instead of images and to keep everything in a single HTML file.

The game includes:
- Four lanes controlled by the F, G, H, and J keys.
- Falling rectangular tiles.
- A highlighted hit zone near the bottom of the screen.
- A score counter.
- A chance for two tiles to appear simultaneously.
- Gradually increasing difficulty as the tiles move faster.
- A Play/Replay button.

ChatGPT generated the code based on my instructions. I tested the game, requested changes, adjusted the difficulty, and analyzed how different parts of the code worked.

---

## 2. Programming Languages and Structure

The game uses HTML, CSS, and JavaScript.

### HTML

HTML creates the basic structure of the webpage, including the canvas where the game appears.

```html
<canvas id="game" width="600" height="700"></canvas>
```

This creates a canvas that is 600 pixels wide and 700 pixels tall. JavaScript draws the game inside this canvas.

### CSS

CSS controls the appearance of the webpage.

```css
body {
    margin: 0;
    background: #263746;
    font-family: Arial;
}

canvas {
    display: block;
    margin: 25px auto;
    border: 4px solid white;
}
```

The `background` property sets the dark blue background, while `margin: 25px auto` centers the canvas horizontally.

### JavaScript

JavaScript controls the game's behavior, including moving tiles, detecting keyboard input, registering hits, and calculating the score.

```javascript
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
```

The first line finds the canvas element using its ID. The second line creates a 2D drawing context, allowing JavaScript to draw shapes and text onto the canvas.

---

## 3. Understanding the Code

### A. Variables and Game State

The game uses variables to store information about its current state.

```javascript
let tiles = [];
let score = 0;
let speed = 5.5;
let timer = 0;
let playing = false;
let gameOver = false;
```

- `tiles = []` creates an empty array that stores the falling tiles.
- `score = 0` tracks the player's points.
- `speed = 5.5` determines how quickly tiles move downward.
- `timer = 0` controls when new tiles appear.
- `playing = false` indicates whether the game is currently running.
- `gameOver = false` records whether the player has lost.

When the player starts or restarts the game, the `startGame()` function resets these variables.

```javascript
function startGame() {
    tiles = [];
    score = 0;
    speed = 5.5;
    timer = 0;
    playing = true;
    gameOver = false;
}
```

This ensures that each new game begins with an empty board, a score of zero, and the original starting speed.

### B. Creating Falling Tiles

The `makeTiles()` function generates new tiles.

```javascript
function makeTiles() {
    let lane1 = Math.floor(Math.random() * 4);

    tiles.push({
        lane: lane1,
        y: 250
    });

    if (Math.random() < 0.60) {
        let lane2 = Math.floor(Math.random() * 4);

        while (lane2 === lane1) {
            lane2 = Math.floor(Math.random() * 4);
        }

        tiles.push({
            lane: lane2,
            y: 250
        });
    }
}
```

First, `Math.random()` generates a random decimal between 0 and 1. Multiplying it by 4 and applying `Math.floor()` produces an integer from 0 to 3, corresponding to one of the four lanes.

The `tiles.push()` statement adds a new tile to the `tiles` array. Each tile is represented by an object containing its lane and vertical position.

The second part of the function gives the game a 60% chance of generating another tile at the same time.

The `while` loop ensures that the second tile appears in a different lane from the first one. Without this loop, two tiles could be generated in the same lane and overlap.

### C. Game Loop and Tile Movement

The game uses a function called `gameLoop()` to continuously update and redraw the screen.

```javascript
function gameLoop() {
    update();
    draw();
    requestAnimationFrame(gameLoop);
}

gameLoop();
```

The `update()` function changes the game state, while `draw()` displays the updated game.

`requestAnimationFrame(gameLoop)` tells the browser to run the function again before the next screen repaint. This creates continuous animation.

Inside `update()`, the following code controls tile movement:

```javascript
timer++;

if (timer >= 50) {
    makeTiles();
    timer = 0;
}

speed += 0.002;

for (let tile of tiles) {
    tile.y += speed;

    if (tile.y > hitY + hitHeight) {
        playing = false;
        gameOver = true;
    }
}
```

The timer increases every frame. Once it reaches 50, the game generates new tiles and resets the timer.

`speed += 0.002` gradually increases the falling speed, making the game more difficult over time.

The `for...of` loop goes through every tile in the array. Each frame, `tile.y += speed` increases the tile's vertical position, making it move downward.

If a tile passes below the hit zone without being hit, the game ends.

### D. Keyboard Input

The game detects keyboard input using an event listener.

```javascript
const keys = ["f", "g", "h", "j"];

document.addEventListener("keydown", function(event) {
    let key = event.key.toLowerCase();
    let lane = keys.indexOf(key);

    if (lane !== -1) {
        pressLane(lane);
    }
});
```

The `keys` array stores the four controls in order.

When the player presses a key, the `keydown` event listener runs.

`event.key.toLowerCase()` converts the pressed key to lowercase so that the program recognizes it consistently.

`keys.indexOf(key)` searches the array for the pressed key and returns its position:

- F corresponds to lane 0.
- G corresponds to lane 1.
- H corresponds to lane 2.
- J corresponds to lane 3.

If the key is not in the array, `indexOf()` returns `-1`, and the game ignores it.

Otherwise, the program calls `pressLane(lane)` to check whether the player successfully hit a tile.

### E. Hit Detection and Scoring

The `pressLane()` function determines whether a key press counts as a successful hit.

```javascript
function pressLane(lane) {
    if (!playing) return;

    for (let i = 0; i < tiles.length; i++) {
        let tile = tiles[i];

        let inHitZone =
            tile.y + tileHeight >= hitY &&
            tile.y <= hitY + hitHeight;

        if (tile.lane === lane && inHitZone) {
            tiles.splice(i, 1);
            score++;
            return;
        }
    }

    playing = false;
    gameOver = true;
}
```

First, `if (!playing) return;` prevents the function from registering hits when the game is not running.

The `for` loop checks the tiles stored in the array.

The `inHitZone` variable checks whether a tile overlaps the hit zone. It compares the bottom of the tile with the top of the hit zone and the top of the tile with the bottom of the hit zone.

The next `if` statement checks two conditions:

```javascript
if (tile.lane === lane && inHitZone)
```

1. The tile must be in the lane corresponding to the key pressed.
2. The tile must overlap the hit zone.

The `&&` operator means both conditions must be true.

If the hit is successful:

```javascript
tiles.splice(i, 1);
score++;
return;
```

- `tiles.splice(i, 1)` removes the successfully hit tile from the array.
- `score++` increases the score by one.
- `return` exits the function so that one key press only registers one hit.

If no matching tile is found, the code sets `playing` to `false` and `gameOver` to `true`, ending the game.

This same function also allows players to hit double tiles. Each key press is processed separately, so two tiles can be hit within the same hit window.

### F. Drawing the Game

The game uses the canvas drawing context to display tiles and other elements.

For example:

```javascript
function drawTiles() {
    ctx.fillStyle = DARK;

    for (let tile of tiles) {
        ctx.fillRect(
            tile.lane * laneWidth + 5,
            tile.y,
            laneWidth - 10,
            tileHeight
        );
    }
}
```

`ctx.fillRect()` draws a rectangle using four values: its horizontal position, vertical position, width, and height.

The horizontal position depends on the tile's lane:

```javascript
tile.lane * laneWidth + 5
```

This places each tile in the correct column.

The vertical position is determined by `tile.y`, which increases as the tile falls.

The `draw()` function calls the different drawing functions in order:

```javascript
function draw() {
    drawBackground();
    drawBoard();
    drawTiles();
    drawKeys();
    drawScore();
    drawMenu();
}
```

By redrawing these elements every frame, the game creates the appearance of moving tiles.

---

## 4. Changes I Made

Although ChatGPT generated the code, I tested the game and requested specific changes to improve the gameplay.

### Change 1: Keyboard Controls

**Original controls:** D, F, J, K

**Final controls:** F, G, H, J

I changed the controls because I found it more comfortable to use four adjacent keys.

The final code uses:

```javascript
const keys = ["f", "g", "h", "j"];
const keyLabels = ["F", "G", "H", "J"];
```

The first array determines which keyboard inputs the game recognizes, while the second array controls the labels displayed on the screen.

This change helped me understand how arrays can connect user input to different parts of a game.

### Change 2: Adding Double Tiles

Originally, the game only generated one tile at a time. I wanted to make it more challenging by occasionally requiring players to press two keys at nearly the same time.

I asked ChatGPT to modify the tile-generation function so that it could create two tiles in different lanes.

The code uses:

```javascript
if (Math.random() < 0.60)
```

I initially tested a probability of 35%:

```javascript
if (Math.random() < 0.35)
```

However, I noticed that double tiles did not appear frequently enough, so I increased the probability to 60%.

I also examined the `while` loop that prevents both tiles from appearing in the same lane.

This change helped me understand how random numbers and loops can be used to create variation in gameplay.

### Change 3: Adjusting Speed

I experimented with different starting speeds because some versions were too slow while others were too difficult.

I eventually settled on:

```javascript
let speed = 5.5;
```

The code also gradually increases the speed:

```javascript
speed += 0.002;
```

Since this statement runs every frame while the game is active, the tiles move faster as the game continues.

I learned that even small changes to numerical values can significantly affect how a game feels.

---

## 5. Testing and Debugging

I tested the game multiple times after generating and modifying the code.

| Test | Observation | Adjustment |
|---|---|---|
| Original controls | The keys felt less comfortable to use. | Changed D, F, J, K to F, G, H, J. |
| Single tiles | The game felt too simple with only one tile at a time. | Added the possibility of double tiles. |
| 35% double-tile probability | Double tiles did not appear frequently enough. | Increased the probability to 60%. |
| Higher starting speeds | Tiles moved too quickly to react comfortably. | Reduced the starting speed to 5.5. |
| Lower starting speeds | Gameplay felt too slow. | Kept gradual acceleration to increase difficulty. |

I also had to figure out how to run the game correctly on my Mac.

Initially, I had problems with file formats when trying to save the code. I learned that the file needed to be saved as `.html` rather than `.rtf` or `.txt` so that the browser would interpret it as a webpage.

I eventually used Terminal and a text editor to save and open the HTML file.

---

## 6. Reflection

Through this project, I learned more about how JavaScript uses arrays, loops, conditional statements, and event listeners to create an interactive game. I found the hit-detection system particularly interesting because it checks both the tile's position and the player's keyboard input before registering a successful hit.

Testing different versions of the game also helped me understand how small changes in the code can significantly affect gameplay. For example, increasing the probability of double tiles from 35% to 60% made the game more challenging, while adjusting the speed made it more playable.

After seeing my classmates present their games, I also noticed how much sound effects, animations, and visual details could improve the overall experience. Although my game functions as intended, it currently has a relatively simple design compared to some of the games I saw. This made me realize that creating a working game is only one part of the process, and that presentation and user experience are also important.

---

## 7. Goals for Next Class

After seeing my classmates' games, I want to focus on making my Piano Tiles game more interactive and visually appealing.

My first goal is to **add sound effects**, especially different piano notes for each lane when a tile is successfully hit. I also want to add a sound effect for missed tiles to provide clearer feedback to the player.

My second goal is to **improve the graphics and animations**. I want to experiment with more detailed tiles, a better background, and visual effects when players successfully hit a tile. I think these changes would make the game feel more polished and engaging.

Next class, I plan to use ChatGPT to help me implement these features, test them individually, and examine the new code to understand how sound and animation work in JavaScript.
