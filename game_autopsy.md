# Piano Tiles – Game Code Autopsy

## Game Overview

For this activity, I created a simple **Piano Tiles** game with the help of ChatGPT. The game uses HTML, CSS, and JavaScript and runs directly in a web browser.

The game has four lanes controlled by the **F, G, H, and J** keys. Tiles move down the screen, and the player has to press the corresponding key while a tile is inside the hit zone. Sometimes two tiles appear simultaneously, and the game gradually gets faster. The player has five lives before the game ends.

## AI-Assisted Development

I used **ChatGPT to generate the code** for my game. I described the game I wanted, including its mechanics, controls, appearance, and features, and AI wrote the HTML, CSS, and JavaScript.

After running the game, I tested it and asked AI to make several changes, including:
- Changing the background to blue.
- Adjusting the starting speed and acceleration.
- Changing the controls to F, G, H, and J.
- Adding the possibility of two simultaneous tiles.
- Changing the game-over system to allow five mistakes.

For this code autopsy, I examined the AI-generated code to understand what each major part does and how the different functions work together.

Although AI generated the code, I made the design decisions, tested the gameplay, requested modifications, and worked to understand how the program functions.

---

## 1. HTML, CSS, and JavaScript

My game is contained in **one HTML file**, but it uses three different parts:

- **HTML** creates the webpage and the canvas where the game appears.
- **CSS** controls the appearance of the webpage, including its colors, positioning, and borders.
- **JavaScript** controls the actual game mechanics, including tile movement, keyboard input, scoring, lives, and game-over logic.

Most of the game is written in JavaScript.

### Example CSS

```css
body {
    margin: 0;
    background: #263746;
    font-family: Arial;
}
```

This sets the background color and font of the webpage.

---

## 2. The Canvas

```html
<canvas id="game" width="600" height="700"></canvas>
```

The canvas is the area where the game is drawn. Mine is **600 pixels wide and 700 pixels tall**.

JavaScript connects to the canvas using:

```javascript
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
```

`ctx` allows JavaScript to draw shapes and text on the canvas.

If I changed the canvas width or height, the visible game area would become larger or smaller. However, I might also need to adjust the positions of other objects so everything still lines up correctly.

---

## 3. State Variables

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

- `tiles` stores all the tiles currently on the screen.
- `score` stores the number of successful hits.
- `lives` stores how many mistakes the player can still make.
- `speed` controls how fast the tiles fall.
- `timer` controls when new tiles appear.
- `playing` determines whether the game is currently running.
- `gameOver` determines whether the player has lost.

These variables are declared near the beginning of the JavaScript code.

---

## 4. Creating and Moving Tiles

Each tile stores its lane and vertical position:

```javascript
tiles.push({
    lane: lane1,
    y: 250
});
```

`lane` determines which of the four columns the tile appears in, while `y` determines its vertical position.

The tiles move using:

```javascript
tiles[i].y += speed;
```

Every time the game updates, the tile's y-position increases. Since larger y-values are farther down the canvas, this makes the tile move downward.

The game also gradually increases its speed:

```javascript
speed += 0.002;
```

This makes the game progressively harder.

---

## 5. The Game Loop

```javascript
function gameLoop() {
    update();
    draw();
    requestAnimationFrame(gameLoop);
}

gameLoop();
```

`gameLoop()` is the main function that runs repeatedly.

It calls three important functions:

- `update()` changes the game state by moving tiles, increasing speed, creating new tiles, and checking for misses.
- `draw()` redraws the game so the player can see the changes.
- `requestAnimationFrame(gameLoop)` tells the browser to run the loop again on the next animation frame.

This usually happens around 60 times per second, depending on the browser and display.

The basic process is:

**Update → Draw → Next Frame → Repeat**

---

## 6. Input Handling

The controls are stored in an array:

```javascript
const keys = ["f", "g", "h", "j"];
```

JavaScript detects keyboard input using an event listener:

```javascript
document.addEventListener("keydown", function(event) {
    let key = event.key.toLowerCase();
    let lane = keys.indexOf(key);

    if (lane !== -1) {
        pressLane(lane);
    }
});
```

When I press a key, the program finds its position in the `keys` array.

- F = lane 0
- G = lane 1
- H = lane 2
- J = lane 3

The program then calls `pressLane()` to check whether the player successfully hit a tile in that lane.

---

## 7. Hit Zone and Collision Detection

The hit zone is defined by:

```javascript
const hitY = 540;
const hitHeight = 110;
```

The hit zone begins at y = 540 and is 110 pixels tall.

The program checks whether a tile overlaps this area:

```javascript
let inHitZone =
    tile.y + tileHeight >= hitY &&
    tile.y <= hitY + hitHeight;
```

The first condition checks whether the bottom of the tile has reached the hit zone. The second checks whether the top of the tile has not passed the bottom of the hit zone.

Both conditions must be true for the tile to overlap the hit zone.

The program also checks whether the player pressed the correct lane:

```javascript
if (tile.lane === lane && inHitZone) {
    tiles.splice(i, 1);
    score++;
    return;
}
```

A successful hit requires both the **correct lane and correct timing**.

---

## 8. Scoring

The score is stored in this variable:

```javascript
let score = 0;
```

When the player correctly hits a tile:

```javascript
tiles.splice(i, 1);
score++;
```

`tiles.splice(i, 1)` removes the tile from the array.

`score++` increases the score by one.

The updated score is then displayed on the canvas.

---

## 9. Lives and Game-Over Logic

The game starts with five lives:

```javascript
let lives = 5;
```

When the player makes a mistake, this function runs:

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

The program checks whether the number of lives is less than or equal to zero.

If it is, `playing` becomes `false` and `gameOver` becomes `true`, ending the game.

A mistake occurs when the player presses an incorrect key or lets a tile pass the hit zone.

---

## 10. Double Tiles

I also added the possibility of two tiles appearing simultaneously.

```javascript
if (Math.random() < 0.60) {
```

`Math.random()` generates a random number between 0 and 1.

Since the program checks whether this number is below `0.60`, there is approximately a **60% chance** of creating a second tile.

The program also makes sure the second tile appears in a different lane:

```javascript
while (lane2 === lane1) {
    lane2 = Math.floor(Math.random() * 4);
}
```

The `while` loop keeps choosing another lane until it is different from the first one.

---

## 11. Deliberate Change – Adding Five Lives

One of the changes I made during development was modifying the game-over system.

Originally, the game ended immediately when the player missed a tile or pressed the wrong key.

I wanted to make the game more forgiving, so I asked ChatGPT to modify the code to allow five mistakes before game over.

### Original Behavior

The original game ended immediately after a mistake:

```javascript
playing = false;
gameOver = true;
```

### Updated Code

The modified version introduces a new state variable:

```javascript
let lives = 5;
```

It also uses a new function:

```javascript
function loseLife() {
    lives--;

    if (lives <= 0) {
        playing = false;
        gameOver = true;
    }
}
```

Instead of immediately ending the game, the program calls:

```javascript
loseLife();
```

### Effect of the Change

Before the change:

**1 mistake → Game Over**

After the change:

**Mistake → Lose 1 life → Continue playing → Game Over after 5 mistakes**

This modification changes the game's difficulty and allows players to recover from occasional mistakes.

It also helped me understand how state variables and conditional statements control the game's behavior.

---

## 12. Three Important Parts of My Code

### 1. The Game Loop

```javascript
function gameLoop() {
    update();
    draw();
    requestAnimationFrame(gameLoop);
}
```

This repeatedly updates and redraws the game, creating the movement and animation.

### 2. The Hit-Zone Comparison

```javascript
let inHitZone =
    tile.y + tileHeight >= hitY &&
    tile.y <= hitY + hitHeight;
```

This checks whether a tile is in the correct position to be hit.

### 3. The Keyboard Input

```javascript
let lane = keys.indexOf(key);

if (lane !== -1) {
    pressLane(lane);
}
```

This converts the key pressed by the player into a lane number and checks whether the player successfully hit a tile.

---

## Reflection

This project helped me better understand how HTML, CSS, and JavaScript work together to create a browser game.

Although ChatGPT generated the code, I was able to examine its structure and identify the functions responsible for movement, input handling, collision detection, scoring, and game-over logic.

I also learned that changing a relatively small part of the code, such as the number of lives or the speed, can significantly affect the gameplay.

Overall, this activity helped me understand how different programming concepts, including **variables, arrays, loops, functions, conditional statements, and event listeners**, work together in a functioning game.
