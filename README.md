<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Snakes & Worms .io</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
    body {
        margin: 0;
        background: #ccc; /* gray background */
        font-family: Arial, sans-serif;
        text-align: center;
        overflow-x: hidden;
    }
    #gameContainer {
        display: inline-block;
        position: relative;
        margin-top: 20px;
    }
    canvas {
        background: #888; /* gray background */
        background-image: radial-gradient(circle, #000 1px, transparent 1px);
        background-size: 40px 40px; /* snake tattoo pattern */
        border: 5px solid #000; /* black frame */
        display: block;
        border-radius: 10px;
    }
    #score, #level {
        font-size: 20px;
        margin-top: 10px;
        color: #000;
    }
    #progressContainer {
        width: 600px;
        height: 20px;
        background-color: #ddd;
        border-radius: 10px;
        margin: 10px auto;
        overflow: hidden;
    }
    #progressBar {
        height: 100%;
        width: 0;
        background-color: #0f0;
        transition: width 0.2s;
    }
    #startButton, #loadingText {
        margin-top: 20px;
        padding: 12px 25px;
        font-size: 20px;
        cursor: pointer;
        border: none;
        border-radius: 8px;
        display: none;
    }
    #startButton {
        background-color: #0f0;
        color: #000;
    }
    #loadingText {
        background-color: #444;
        color: #fff;
        border-radius: 5px;
        padding: 10px 20px;
    }
</style>
</head>
<body>
<h1>Snakes & Worms</h1>
<div id="gameContainer">
    <canvas id="gameCanvas" width="600" height="600"></canvas>
    <div id="score">Length: 5</div>
    <div id="level">Level: 1</div>
    <div id="progressContainer">
        <div id="progressBar"></div>
    </div>
    <div id="loadingText">Loading... 0%</div>
    <button id="startButton">START</button>
</div>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");
const box = 20;

let snake, direction, food, length, level, blocksToEat, gameInterval;
let snakeColors = [];

const scoreEl = document.getElementById("score");
const levelEl = document.getElementById("level");
const progressBar = document.getElementById("progressBar");
const startButton = document.getElementById("startButton");
const loadingText = document.getElementById("loadingText");

// --- LOADING SCREEN ---
let loadPercent = 0;
loadingText.style.display = "block";
let loadingInterval = setInterval(() => {
    loadPercent++;
    loadingText.innerText = `Loading... ${loadPercent}%`;
    if(loadPercent >= 100){
        clearInterval(loadingInterval);
        loadingText.style.display = "none";
        startButton.style.display = "inline-block";
    }
}, 100); // 10 seconds

// --- INIT GAME ---
function initGame(lvl=1){
    snake = [{x: 9*box, y: 10*box}];
    snakeColors = ["#0f0"]; // initial snake color
    direction = "RIGHT";
    length = 5;
    level = lvl;
    blocksToEat = 30 + (level-1)*5;
    spawnFood();
    scoreEl.innerText = "Length: " + length;
    levelEl.innerText = "Level: " + level;
    updateProgress();
    startButton.style.display = "none";

    if(gameInterval) clearInterval(gameInterval);
    gameInterval = setInterval(draw, 100);
}

// --- SPAWN FOOD ---
function spawnFood(){
    // random color for food
    const colors = ["#FF0000","#00FF00","#0000FF","#FFFF00","#FF00FF","#00FFFF","#FFA500"];
    const color = colors[Math.floor(Math.random() * colors.length)];
    food = {
        x: Math.floor(Math.random() * 30) * box,
        y: Math.floor(Math.random() * 30) * box,
        color: color
    };
}

// --- DRAW SNAKE ---
function drawSnake(){
    snake.forEach((part, index) => {
        ctx.fillStyle = snakeColors[index];
        ctx.fillRect(part.x, part.y, box, box);
        ctx.strokeStyle = "#111";
        ctx.strokeRect(part.x, part.y, box, box);
    });
}

// --- DRAW FOOD ---
function drawFood(){
    ctx.fillStyle = food.color;
    ctx.fillRect(food.x, food.y, box, box);
}

// --- CHANGE DIRECTION ---
document.addEventListener("keydown", (e) => {
    if(e.keyCode == 37 && direction != "RIGHT") direction = "LEFT";
    if(e.keyCode == 38 && direction != "DOWN") direction = "UP";
    if(e.keyCode == 39 && direction != "LEFT") direction = "RIGHT";
    if(e.keyCode == 40 && direction != "UP") direction = "DOWN";
});

// --- COLLISION CHECK ---
function collision(head, array){
    return array.some(part => head.x === part.x && head.y === part.y);
}

// --- UPDATE PROGRESS BAR ---
function updateProgress(){
    const percent = ((30 + (level-1)*5 - blocksToEat) / (30 + (level-1)*5)) * 100;
    progressBar.style.width = percent + "%";
}

// --- DRAW GAME ---
function draw(){
    ctx.clearRect(0,0,canvas.width,canvas.height);
    drawSnake();
    drawFood();

    let headX = snake[0].x;
    let headY = snake[0].y;

    if(direction === "LEFT") headX -= box;
    if(direction === "UP") headY -= box;
    if(direction === "RIGHT") headX += box;
    if(direction === "DOWN") headY += box;

    // Snake eats food
    if(headX === food.x && headY === food.y){
        length++;
        blocksToEat--;
        snakeColors.unshift(food.color); // add food color to snake
        spawnFood();
        scoreEl.innerText = "Length: " + length;
        updateProgress();
    } else {
        snake.pop();
        snakeColors.pop();
    }

    let newHead = {x: headX, y: headY};

    // Game over
    if(headX < 0 || headX >= canvas.width || headY < 0 || headY >= canvas.height || collision(newHead, snake)){
        clearInterval(gameInterval);
        alert("Game Over! Final Length: " + length);
        initGame(1);
        return;
    }

    snake.unshift(newHead);

    // Level complete
    if(blocksToEat <= 0){
        clearInterval(gameInterval);
        startButton.style.display = "inline-block";
    }
}

// --- START BUTTON ---
startButton.addEventListener("click", () => {
    initGame(level + 1);
});
</script>
</body>
</html>
