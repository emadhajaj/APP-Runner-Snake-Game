# APP-Runner-Snake-Game



# 1- The game files

## `index.html`

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Snake — App Runner Demo</title>
  <link rel="stylesheet" href="styles.css" />
</head>
<body>
  <div class="app">
    <h1>Snake</h1>
    <div id="score">Score: 0</div>
    <canvas id="game" width="400" height="400"></canvas>

    <div class="controls">
      <button id="btn-up">↑</button>
      <div>
        <button id="btn-left">←</button>
        <button id="btn-right">→</button>
      </div>
      <button id="btn-down">↓</button>
    </div>

    <div class="actions">
      <button id="start">Start</button>
      <button id="pause">Pause</button>
    </div>
    <footer>Built for AWS App Runner demo</footer>
  </div>

  <script src="game.js"></script>
</body>
</html>
```

## `styles.css`

```css
*{box-sizing:border-box;font-family:Inter,system-ui,-apple-system,Segoe UI,Roboto,Arial;}
body{background:#0b1220;color:#e6eef7;display:flex;min-height:100vh;align-items:center;justify-content:center;margin:0;padding:20px}
.app{width:420px;text-align:center}
h1{margin:0 0 6px;font-size:28px}
#score{margin-bottom:8px}
canvas{display:block;border-radius:8px;margin:0 auto 12px;background:#071029;box-shadow:0 6px 18px rgba(0,0,0,0.6)}
.controls{display:flex;flex-direction:column;gap:6px;align-items:center;margin-bottom:10px}
.controls > div{display:flex;gap:6px}
button{padding:8px 12px;border-radius:8px;border:none;background:#1f6feb;color:white;font-weight:600;cursor:pointer}
.actions{display:flex;gap:8px;justify-content:center;margin-bottom:8px}
footer{font-size:12px;color:#9fb4d9;margin-top:8px}
```

## `game.js`

```javascript
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');

const SCALE = 20;
const COLS = canvas.width / SCALE; // 20
const ROWS = canvas.height / SCALE; // 20

let snake = [];
let food = null;
let dx = 1, dy = 0;
let playing = false;
let speedMs = 100;
let loop = null;
let score = 0;

function init(){
  snake = [{x: Math.floor(COLS/2), y: Math.floor(ROWS/2)}];
  dx = 1; dy = 0;
  score = 0;
  placeFood();
  updateScore();
  startLoop();
  playing = true;
}

function placeFood(){
  food = { x: Math.floor(Math.random()*COLS), y: Math.floor(Math.random()*ROWS) };
  if (snake.some(s => s.x===food.x && s.y===food.y)) placeFood();
}

function startLoop(){
  if(loop) clearInterval(loop);
  loop = setInterval(gameTick, speedMs);
}

function gameTick(){
  const head = {x: snake[0].x + dx, y: snake[0].y + dy};

  // wrap-around
  if(head.x < 0) head.x = COLS - 1;
  if(head.x >= COLS) head.x = 0;
  if(head.y < 0) head.y = ROWS - 1;
  if(head.y >= ROWS) head.y = 0;

  // collision with body
  if(snake.some(s => s.x===head.x && s.y===head.y)){ gameOver(); return; }

  snake.unshift(head);

  if(head.x === food.x && head.y === food.y){
    score++;
    updateScore();
    placeFood();
  } else {
    snake.pop();
  }
  draw();
}

function draw(){
  // background
  ctx.fillStyle = '#071029';
  ctx.fillRect(0,0,canvas.width,canvas.height);

  // food
  ctx.fillStyle = '#ff6b6b';
  ctx.fillRect(food.x*SCALE, food.y*SCALE, SCALE, SCALE);

  // snake
  ctx.fillStyle = '#2ecc71';
  snake.forEach((p, i) => {
    ctx.fillRect(p.x*SCALE, p.y*SCALE, SCALE, SCALE);
    ctx.strokeStyle = '#071029';
    ctx.strokeRect(p.x*SCALE, p.y*SCALE, SCALE, SCALE);
  });
}

function updateScore(){ document.getElementById('score').textContent = 'Score: ' + score; }

function gameOver(){
  clearInterval(loop);
  playing = false;
  alert('Game over — score: ' + score);
}

document.addEventListener('keydown', e=>{
  const key = e.key;
  if((key === 'ArrowUp' || key === 'w') && !(dx===0 && dy===1)){ dx=0; dy=-1; }
  if((key === 'ArrowDown' || key === 's') && !(dx===0 && dy===-1)){ dx=0; dy=1; }
  if((key === 'ArrowLeft' || key === 'a') && !(dx===1 && dy===0)){ dx=-1; dy=0; }
  if((key === 'ArrowRight' || key === 'd') && !(dx===-1 && dy===0)){ dx=1; dy=0; }
});

document.getElementById('start').addEventListener('click', ()=>{ if(!playing) init(); });
document.getElementById('pause').addEventListener('click', ()=>{
  if(playing){ clearInterval(loop); playing=false; } else { startLoop(); playing=true; }
});

['up','down','left','right'].forEach(k => {
  const btn = document.getElementById('btn-' + k);
  if(btn){
    btn.addEventListener('click', ()=>{
      if(k==='up' && !(dx===0 && dy===1)){ dx=0; dy=-1; }
      if(k==='down' && !(dx===0 && dy===-1)){ dx=0; dy=1; }
      if(k==='left' && !(dx===1 && dy===0)){ dx=-1; dy=0; }
      if(k==='right' && !(dx===-1 && dy===0)){ dx=1; dy=0; }
    });
  }
});
```



---



## `Dockerfile` 

```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```



---

# 2- Push the image to AWS ECR

> You must have aws and docker installed on your local machine

1. **Create repository** → choose **Private** → Repository name: `snake-game`
2. Create an IAM user with full control
3. go to **IAM** → Users → Select your user → **Security credentials** → **Create access key**.
4. Configure the user on the local machine
	- `aws configure`
	- put the access key and the secret access key from the credentials 
	-  **Region**: usually `us-east-1`
	- **Output format**: `json` 
5. Go back to AWS ECR, choose your repo then push the `view push commands` button
6. Run the steps in a bash terminal in the folder that contains the game's files
	- `aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 198945929565.dkr.ecr.us-east-1.amazonaws.com`
	- `docker build -t snake-game .`
	- `docker tag snake-game:latest 198945929565.dkr.ecr.us-east-1.amazonaws.com/snake-game:latest`
	- `docker push 198945929565.dkr.ecr.us-east-1.amazonaws.com/snake-game:latest`


# 3- Create App Runner service from the Console

1. **Create an App Runner service** 
    
2. **Source and deployment**:
    - For **Source** choose **Container registry**.
    - For **Provider** choose **Amazon ECR**.
    - Click **Select** and pick the `snake-game` repository and the tag (e.g. `latest`).
    - Enable **Automatic deploy** 
    
3. **IAM / permissions**: Create new service role
	App Runner needs permission to pull from ECR, the console will create a role for you automatically
	
4. **Service settings**:
    - **Service name** `snake-svc`.
    - **Port**: **set to 80** 
    - **Instance configuration**: default CPU/memory is fine for this demo
    - Health check:  (HTTP /) 
    
5. **Environment variables**: none needed for the static game.


# 4- Play

1. wait until the service is fully running
2. press Deploy
3. wait again
4. open the public URL in the App runner service
5. Enjoy 😉
