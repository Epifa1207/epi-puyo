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
</html>
