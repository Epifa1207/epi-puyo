# epi-puyo
party game
   * index.html
   * script.js
   * style.css (必要に応じて)
   * bgm.mp3
   * drop.mp3
   * clear.mp3
   * chain.mp3
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ぷよぷよ風ゲーム</title>
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #f0f0f0;
            font-family: sans-serif;
        }
        #game-container {
            display: none;
            flex-direction: column;
            align-items: center;
        }
        #game-board {
            display: grid;
            grid-template-columns: repeat(6, 30px);
            grid-gap: 1px;
            background-color: #000;
        }
        .cell {
            width: 30px;
            height: 30px;
            background-color: #fff;
            box-sizing: border-box;
        }
        #title-screen, #game-over-screen {
            text-align: center;
        }
        #title-screen button, #game-over-screen button {
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
        }
        #next-piece {
            display: grid;
            grid-template-columns: repeat(2, 30px);
            grid-gap: 1px;
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <div id="title-screen">
        <h1>ぷよぷよ風ゲーム</h1>
        <button id="start-button">ゲーム開始</button>
    </div>
    <div id="game-container">
        <div id="score">スコア: 0</div>
        <div id="chain-message"></div>
        <div id="timer">残り時間: 180秒</div>
        <div id="game-board"></div>
        <div id="next-piece"></div>
    </div>
    <div id="game-over-screen">
        <h1>ゲームオーバー</h1>
        <p>スコア: <span id="final-score">0</span></p>
        <p>プレイ時間: <span id="play-time">0</span>秒</p>
        <button id="retry-button">リトライ</button>
    </div>
    <audio id="bgm" src="bgm.mp3" loop></audio>
    <audio id="drop-sound" src="drop.mp3"></audio>
    <audio id="clear-sound" src="clear.mp3"></audio>
    <audio id="chain-sound" src="chain.mp3"></audio>
    <script src="script.js"></script>
</body>
  // 省略
// ゲーム開始・終了画面
const titleScreen = document.getElementById('title-screen');
const gameContainer = document.getElementById('game-container');
const gameOverScreen = document.getElementById('game-over-screen');
const startButton = document.getElementById('start-button');
const retryButton = document.getElementById('retry-button');
const finalScore = document.getElementById('final-score');
const playTime = document.getElementById('play-time');
//ハイスコア
let highScore = localStorage.getItem('highScore') || 0;

startButton.addEventListener('click', () => {
    titleScreen.style.display = 'none';
    gameContainer.style.display = 'flex';
    gameOverScreen.style.display = 'none';
    bgm.play();
    initBoard();
});

retryButton.addEventListener('click', () => {
    gameOverScreen.style.display = 'none';
    gameContainer.style.display = 'flex';
    resetGame();
});

function gameOver() {
    clearInterval(gameInterval);
    clearInterval(timerInterval);
    bgm.pause();
    finalScore.textContent = score;
    playTime.textContent = 180 - timeLeft;
    gameContainer.style.display = 'none';
    gameOverScreen.style.display = 'block';
    if (score > highScore){
        highScore = score;
        localStorage.setItem('highScore', highScore);
    }
}

function resetGame() {
    score = 0;
    timeLeft = 180;
    board.forEach(row => row.fill(null));
    updateBoard();
    spawnPiece();
    spawnNextPiece();
    gameInterval = setInterval(moveDown, dropInterval);
    startTimer();
    bgm.play();
}
// 効果音
const dropSound = document.getElementById('drop-sound');
const clearSound = document.getElementById('clear-sound');
const chainSound = document.getElementById('chain-sound');

function playSound(sound) {
    sound.currentTime = 0;
    sound.play();
}

function moveDown() {
    if (canMove(0, 1)) {
        currentPiece.y++;
    } else {
        fixPiece();
        clearPuyo();
        if (gameOverCheck()) {
            gameOver();
            return;
        }
        spawnPiece();
        spawnNextPiece();
    }
    updateBoard();
    playSound(dropSound); // ドロップ音を再生
}

function gameOverCheck() {
    for (let x = 0; x < boardWidth; x++) {
        if (board[0][x]) {
            return true; // 一番上のラインにぷよが残っていたらゲームオーバー
        }
    }
    return false;
}

function fixPiece() {
    currentPiece.blocks.forEach(block => {
        if (block) {
            board[currentPiece.y + block.y][currentPiece.x + block.x] = block.color;
        }
    });
}
function clearPuyo() {
    let cleared = false;
    let chainCount = 0;
    do {
        cleared = false;
        chainCount++;
        let groups = findPuyoGroups();
        let clearedPuyoCount = 0;

        groups.forEach(group => {
            if (group.length >= 4) {
                cleared = true;
                clearedPuyoCount += group.length;
                group.forEach(puyo => {
                    board[puyo.y][puyo.x] = null;
                });
            }
        });

        if (cleared) {
            playSound(chainSound); // 連鎖音を再生
            score += clearedPuyoCount * 10 * chainCount;
            scoreDisplay.textContent = `スコア: ${score}`;
            chainMessage.textContent = chainCount > 1 ? `${chainCount}連鎖！` : '';
            updateBoard();
            dropPuyo();
        } else {
            chainMessage.textContent = ''; // 連鎖が終わったらメッセージをクリア
        }
    } while (cleared);
}
function dropPuyo() {
    let dropped;
    do {
        dropped = false;
        for (let x = 0; x < boardWidth; x++) {
            for (let y = boardHeight - 2; y >= 0; y--) {
                if (board[y][x] && !board[y + 1][x]) {
                    board[y + 1][x] = board[y][x];
                    board[y][x] = null;
                    dropped = true;
                }
            }
        }
    } while (dropped);
    updateBoard();
}
function findPuyoGroups() {
    let groups = [];
    let visited = board.map(row => row.map(() => false));

    for (let y = 0; y < boardHeight; y++) {
        for (let x = 0; x < boardWidth; x++) {
            if (board[y][x] && !visited[y][x]) {
                let group = [];
                let color = board[y][x];
                findGroup(x, y, color, group, visited);
                groups.push(group);
            }
        }
    }
    return groups;
}
function findGroup(x, y, color, group, visited) {
    if (x < 0 || x >= boardWidth || y < 0 || y >= boardHeight || visited[y][x] || board[y][x] !== color) {
        return;
    }

</html>
