<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>リアルタイム対戦カードゲーム</title>
  <style>
    body { font-family: sans-serif; text-align: center; margin: 0; padding: 20px; }
    input, button { padding: 10px; margin: 10px; font-size: 16px; }
    .hidden { display: none; }
    #result { font-size: 24px; margin-top: 20px; }
  </style>
</head>
<body>
  <h2>合言葉で対戦しよう！</h2>
  <input type="text" id="roomInput" placeholder="合言葉（roomId）" />
  <input type="text" id="playerName" placeholder="プレイヤー名" />
  <button onclick="joinGame()">参加</button>

  <div id="gameArea" class="hidden">
    <h3>あなたの手札: <span id="myHand"></span>（合計 <span id="myTotal"></span>）</h3>
    <h3>相手の手札: <span id="opponentHand"></span>（合計 <span id="opponentTotal"></span>）</h3>
    <div id="result"></div>
  </div>

  <!-- Firebase SDK -->
  <script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-database.js"></script>

  <script>
    const firebaseConfig = {
      apiKey: "YOUR_API_KEY",
      authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
      databaseURL: "https://YOUR_PROJECT_ID.firebaseio.com",
      projectId: "YOUR_PROJECT_ID",
    };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();

    let roomId, playerKey, myHand, myTotal;

    function joinGame() {
      roomId = document.getElementById('roomInput').value.trim();
      const name = document.getElementById('playerName').value.trim();
      if (!roomId || !name) return alert("合言葉と名前を入力してください");

      const playerRef1 = db.ref(`rooms/${roomId}/player1`);
      const playerRef2 = db.ref(`rooms/${roomId}/player2`);

      playerRef1.once("value").then(snap => {
        if (!snap.exists()) {
          playerKey = 'player1';
          startGame(playerRef1, name);
        } else {
          playerRef2.once("value").then(snap2 => {
            if (!snap2.exists()) {
              playerKey = 'player2';
              startGame(playerRef2, name);
            } else {
              alert("この部屋は満員です。別の合言葉を使ってください。");
            }
          });
        }
      });
    }

    function startGame(ref, name) {
      myHand = [rand(), rand()];
      myTotal = myHand[0] + myHand[1];
      ref.set({ name, hand: myHand, total: myTotal });

      document.getElementById("myHand").innerText = myHand.join(', ');
      document.getElementById("myTotal").innerText = myTotal;
      document.getElementById("gameArea").classList.remove("hidden");

      // 相手の手札監視
      const opponentKey = playerKey === 'player1' ? 'player2' : 'player1';
      db.ref(`rooms/${roomId}/${opponentKey}`).on("value", snapshot => {
        const val = snapshot.val();
        if (val) {
          document.getElementById("opponentHand").innerText = val.hand.join(', ');
          document.getElementById("opponentTotal").innerText = val.total;
          checkWinner(myTotal, val.total);
        }
      });
    }

    function checkWinner(my, opponent) {
      const result = document.getElementById("result");
      if (my > opponent) result.innerText = "あなたの勝ち！";
      else if (my < opponent) result.innerText = "あなたの負け…";
      else result.innerText = "引き分け！";
    }

    function rand() {
      return Math.floor(Math.random() * 10) + 1;
    }
  </script>
</body>
</html>
