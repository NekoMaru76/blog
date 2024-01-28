+++ title = 'Mobile Snake' date = 2024-01-28T20:56:53+07:00 draft = true +++

# How to Create a Snake Web Game

## Requirements

1. Code Editor.
2. NodeJS & NPM.
3. Browser.

### Requirements Installation for Android

1. Install Termux from F-Droid.
2. Open Termux.
3. Run this command to prepare the environment needed for NodeJS.

```sh
pkg update -y && pkg upgrade -y
```

4. Install NodeJS with this command.

```sh
pkg install -y node
```

### Requirements Installation for Linux

1. Open your shell.
2. Install NodeJS.

## Preparation

1. Open the project folder with this command.

```sh
cd <path/to/folder>
```

If you haven't created any folder, you can use the `mkdir` command. Android
users can use the prefix `/sdcard/`. 2. Make a NPM package.

```sh
npm init -y
```

3. Let's install some packages for development process.

```sh
npm i -D prettier@14.2.1 serve@3.2.4
```

4. Let's add some scripts into the `package.json` to fasten our development
   progress.

```json
{
	...
	"scripts": {
		"lint": "npx prettier -w --use-tabs --tab-width 4 .",
		"start": "npx serve src"
	},
	...
}
```

5. Let's create some files and a folder. Here's the folder structure of our
   project.

```
snake/
├─ node_modules/
├─ src/
│  ├─ index.html
│  ├─ script.js
│  ├─ style.css
├─ package-lock.json
├─ package.json
```

Let's run this command to make the files and the folder you don't have!

```sh
mkdir src/
touch src/index.html src/style.css src/script.js
```

6. You're ready to code now!

## Let's Code!

1. Let's make the HTML base code. Here are list you need to know:

- We will be having 1 center canvas and some buttons spread across the web.
- We will include `style.css` and `script.js` into HTML.
- We will use `eruda` as our DevTools in mobile.
- There will be movement buttons (Left, Right, Up, Down) and control buttons
  (Start, Stop, Toggle Wall, Reset).

```html
<!doctype html>
<html>
	<head>
		<link rel="stylesheet" href="style.css" />
		<script src="https://cdnjs.cloudflare.com/ajax/libs/eruda/3.0.1/eruda.min.js"></script>
		<script>
			eruda.init();
		</script>
	</head>
	<body>
		<div id="canvas-container">
			<canvas id="canvas"></canvas>
		</div>
		<div id="movement-container">
			<button id="left" class="movement-button button button-sizing">
				Left
			</button>
			<div id="movement-vertical-container">
				<button id="up" class="movement-button button button-sizing">
					Up
				</button>
				<button id="down" class="movement-button button button-sizing">
					Down
				</button>
			</div>
			<button id="right" class="movement-button button button-sizing">
				Right
			</button>
			<button id="start" class="control-button button button-sizing">
				Start
			</button>
			<button id="pause" class="control-button button button-sizing">
				Pause
			</button>
			<button id="reset" class="control-button button button-sizing">
				Reset
			</button>
			<button id="wall" class="control-button button button-sizing">
				Wall
			</button>
		</div>
		<script src="script.js"></script>
	</body>
</html>
```

2. Let's style it now!

```css
html,
body {
	margin: 0;
	width: 100%;
	height: 100%;
}

#canvas-container {
	display: flex;
	justify-content: center;
	align-items: center;
	width: 100%;
	height: 100%;
}

#movement-container {
	display: flex;
	top: 0;
	position: absolute;
	width: 100%;
	height: 100%;
	z-index: 1;
}

#movement-vertical-container {
	margin-left: auto;
	margin-right: auto;
	height: 100%;
	position: relative;
}

#left {
	margin-left: 0;
	margin-right: auto;
	margin-top: auto;
	margin-bottom: auto;
}

#right {
	margin-top: auto;
	margin-bottom: auto;
	margin-left: auto;
	margin-right: 0;
}

#down {
	position: absolute;
	bottom: 0;
	top: auto;
}

.movement-button {
	position: relative;
	display: block;
}

.button {
	opacity: 0.8;
	background-color: grey;
	color: white;
}

@media only screen and (max-width: 1080px) {
	.button-sizing {
		width: 15vw;
		height: 15vw;
		font-size: 5vw;
	}
}

@media only screen and (min-width: 1081px) {
	.button-sizing {
		width: 5vw;
		height: 5vw;
		font-size: 1vw;
	}
}

#canvas {
	position: relative;
}

.control-button {
	position: absolute;
}

#start {
	top: auto;
	bottom: 0;
}

#reset {
	top: auto;
	bottom: 0;
	left: auto;
	right: 0;
}

#wall {
	left: auto;
	right: 0;
}
```

3. Now let's create the JS code! First we will need to get the `canvas` and the
   context.

```js
const canv = document.querySelector("canvas");
const ctx = canv.getContext("2d");
```

4. Then let's get the buttons.

```js
const leftButton = document.getElementById("left");
const rightButton = document.getElementById("right");
const upButton = document.getElementById("up");
const downButton = document.getElementById("down");
const startButton = document.getElementById("start");
const pauseButton = document.getElementById("pause");
const resetButton = document.getElementById("reset");
const wallButton = document.getElementById("wall");
```

5. Let's also make variables for game states.

```js
const sizeWidth = 25;
const sizeHeight = 25;

let apple, snake, boxSize;
let isPlaying = false,
	isEnd = false,
	isWallEnabled = false,
	direction = 0; // 0: left, 1: up, 2: right, 3: down
```

6. Make some functions to change game states of some cases. Here are the list of
   the functions for some cases we wanna create:

- Toggle Wall.
- Pause.
- Stop.
- Start.
- Reset.
- End.
- Win.
- Lose.
- Start the update clock.
- Set apple's position randomly.
- Set the snake's position into initial position.
- Init.
- Setup.
- Update.

```js
function setSnake() {
	snake = [[Math.round(sizeWidth / 2), Math.round(sizeHeight / 2)]];
}

function setApple() {
	if (sizeWidth * sizeHeight <= snake.length) return win();

	apple = {
		x: Math.floor(Math.random() * sizeWidth) + 1,
		y: Math.floor(Math.random() * sizeHeight) + 1,
	};

	for (const [bodyX, bodyY] of snake) {
		if (apple.x === bodyX && apple.y === bodyY) return setApple();
	}
}

function init() {
	setSnake();
	setApple();
}

function wall() {
	isWallEnabled = !isWallEnabled;
}

function pause() {
	isPlaying = false;
	isEnd = false;
}

function start() {
	if (isEnd) init();

	isPlaying = true;
	isEnd = false;
}

function reset() {
	init();

	isPlaying = false;
	isEnd = false;
}

function end() {
	isPlaying = false;
	isEnd = true;
}

function win() {
	alert("You win!");
	end();
}

function lose() {
	alert("You lose!");
	end();
}

function clock() {
	setInterval(update, 500);
}

function setup() {
	clock();
	resize();
	init();
}

function update() {
	if (isPlaying) {
		let [x, y] = snake[0];

		switch (direction) {
			case 0:
				if (--x < 1) {
					if (isWallEnabled) return lose();
					else x = sizeWidth;
				}
				break;
			case 1:
				if (--y < 1) {
					if (isWallEnabled) return lose();
					else y = sizeHeight;
				}
				break;
			case 2:
				if (++x > sizeWidth) {
					if (isWallEnabled) return lose();
					else x = 1;
				}
				break;
			case 3:
				if (++y > sizeHeight) {
					if (isWallEnabled) return lose();
					else y = 1;
				}
				break;
		}

		snake.splice(0, 0, [x, y]);

		if (snake[0][0] !== apple.x || snake[0][1] !== apple.y) snake.pop();
		else setApple();

		for (const [bodyX, bodyY] of snake.slice(1, snake.length)) {
			if (bodyX === x && bodyY === y) return lose();
		}
	}

	render();
}
```

7. Listen for events from buttons.

```js
leftButton.addEventListener("click", () => {
	if (direction !== 2) direction = 0;
});
rightButton.addEventListener("click", () => {
	if (direction !== 0) direction = 2;
});
upButton.addEventListener("click", () => {
	if (direction !== 3) direction = 1;
});
downButton.addEventListener("click", () => {
	if (direction !== 1) direction = 3;
});
pauseButton.addEventListener("click", pause);
startButton.addEventListener("click", start);
resetButton.addEventListener("click", reset);
wallButton.addEventListener("click", wall);
```

8. Make a function to resize the size of canvas and listen for `resize` event.

```js
function resize() {
	const winWidth = window.innerWidth;
	const winHeight = window.innerHeight;

	if (winWidth < winHeight) {
		boxSize = Math.floor(winWidth / sizeWidth);

		canv.width = winWidth;
		canv.height = boxSize * sizeHeight;
	} else {
		boxSize = Math.floor(winHeight / sizeHeight);

		canv.height = winHeight;
		canv.width = boxSize * sizeWidth;
	}
}

window.addEventListener("resize", resize);
```

9. Let's make a function to render our states.

```js
function render() {
	ctx.fillStyle = isWallEnabled ? "yellow" : "black";

	ctx.fillRect(0, 0, canv.width, canv.height);

	if (snake) {
		for (let i = 0; i < snake.length; i++) {
			const [x, y] = snake[i];

			ctx.fillStyle = i ? "green" : "white";

			ctx.fillRect(
				boxSize * (x - 1),
				boxSize * (y - 1),
				boxSize,
				boxSize,
			);
		}
	}

	if (!apple) return;

	ctx.fillStyle = "red";

	ctx.fillRect(
		boxSize * (apple.x - 1),
		boxSize * (apple.y - 1),
		boxSize,
		boxSize,
	);
}
```

10. Let's listen to the `load` event.

```js
window.addEventListener("load", setup);
```

11. Your JS code should look like this.

```js
const canv = document.querySelector("canvas");
const ctx = canv.getContext("2d");

const leftButton = document.getElementById("left");
const rightButton = document.getElementById("right");
const upButton = document.getElementById("up");
const downButton = document.getElementById("down");
const startButton = document.getElementById("start");
const pauseButton = document.getElementById("pause");
const resetButton = document.getElementById("reset");
const wallButton = document.getElementById("wall");

const sizeWidth = 25;
const sizeHeight = 25;

let apple, snake, boxSize;
let isPlaying = false,
	isEnd = false,
	isWallEnabled = false,
	direction = 0; // 0: left, 1: up, 2: right, 3: down

function setSnake() {
	snake = [[Math.round(sizeWidth / 2), Math.round(sizeHeight / 2)]];
}

function setApple() {
	if (sizeWidth * sizeHeight <= snake.length) return win();

	apple = {
		x: Math.floor(Math.random() * sizeWidth) + 1,
		y: Math.floor(Math.random() * sizeHeight) + 1,
	};

	for (const [bodyX, bodyY] of snake) {
		if (apple.x === bodyX && apple.y === bodyY) return setApple();
	}
}

function init() {
	setSnake();
	setApple();
}

function wall() {
	isWallEnabled = !isWallEnabled;
}

function pause() {
	isPlaying = false;
	isEnd = false;
}

function start() {
	if (isEnd) init();

	isPlaying = true;
	isEnd = false;
}

function reset() {
	init();

	isPlaying = false;
	isEnd = false;
}

function end() {
	isPlaying = false;
	isEnd = true;
}

function win() {
	alert("You win!");
	end();
}

function lose() {
	alert("You lose!");
	end();
}

function clock() {
	setInterval(update, 500);
}

function setup() {
	clock();
	resize();
	init();
}

function update() {
	if (isPlaying) {
		let [x, y] = snake[0];

		switch (direction) {
			case 0:
				if (--x < 1) {
					if (isWallEnabled) return lose();
					else x = sizeWidth;
				}
				break;
			case 1:
				if (--y < 1) {
					if (isWallEnabled) return lose();
					else y = sizeHeight;
				}
				break;
			case 2:
				if (++x > sizeWidth) {
					if (isWallEnabled) return lose();
					else x = 1;
				}
				break;
			case 3:
				if (++y > sizeHeight) {
					if (isWallEnabled) return lose();
					else y = 1;
				}
				break;
		}

		snake.splice(0, 0, [x, y]);

		if (snake[0][0] !== apple.x || snake[0][1] !== apple.y) snake.pop();
		else setApple();

		for (const [bodyX, bodyY] of snake.slice(1, snake.length)) {
			if (bodyX === x && bodyY === y) return lose();
		}
	}

	render();
}

leftButton.addEventListener("click", () => {
	if (direction !== 2) direction = 0;
});
rightButton.addEventListener("click", () => {
	if (direction !== 0) direction = 2;
});
upButton.addEventListener("click", () => {
	if (direction !== 3) direction = 1;
});
downButton.addEventListener("click", () => {
	if (direction !== 1) direction = 3;
});
pauseButton.addEventListener("click", pause);
startButton.addEventListener("click", start);
resetButton.addEventListener("click", reset);
wallButton.addEventListener("click", wall);

function resize() {
	const winWidth = window.innerWidth;
	const winHeight = window.innerHeight;

	if (winWidth < winHeight) {
		boxSize = Math.floor(winWidth / sizeWidth);

		canv.width = winWidth;
		canv.height = boxSize * sizeHeight;
	} else {
		boxSize = Math.floor(winHeight / sizeHeight);

		canv.height = winHeight;
		canv.width = boxSize * sizeWidth;
	}
}

window.addEventListener("resize", resize);

function render() {
	ctx.fillStyle = isWallEnabled ? "yellow" : "black";

	ctx.fillRect(0, 0, canv.width, canv.height);

	if (snake) {
		for (let i = 0; i < snake.length; i++) {
			const [x, y] = snake[i];

			ctx.fillStyle = i ? "green" : "white";

			ctx.fillRect(
				boxSize * (x - 1),
				boxSize * (y - 1),
				boxSize,
				boxSize,
			);
		}
	}

	if (!apple) return;

	ctx.fillStyle = "red";

	ctx.fillRect(
		boxSize * (apple.x - 1),
		boxSize * (apple.y - 1),
		boxSize,
		boxSize,
	);
}

window.addEventListener("load", setup);
```

12. You guys now can run the website with this command.

```js
npm run start
```

Then open your browser with the URL showed in the console.

## Source Code

https://github.com/NekoMaru76/mobile-snake

## Support Me

Paypal: NekoMaru76{{< lineBreak >}} Discord: @ardofulian

