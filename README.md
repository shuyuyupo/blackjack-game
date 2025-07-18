<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>ブラックジャック</title>
  <style>
    body {
      font-family: "Arial Rounded MT Bold", "Comic Sans MS", cursive;
      text-align: center;
      padding: 30px;
      background: linear-gradient(to bottom right, #e0f7ff, #ccf2ff);
      color: #333;
    }

    h1 {
      font-size: 40px;
      margin-bottom: 20px;
      color: #00bfff;
      text-shadow: 1px 1px 3px #fff;
    }

    .status-bar {
      font-size: 20px;
      margin-bottom: 15px;
      background: #e0faff;
      padding: 10px 20px;
      border-radius: 20px;
      display: inline-block;
      box-shadow: 0 4px 8px rgba(0,0,0,0.1);
    }

    label, input[type="number"] {
      font-size: 18px;
    }

    input[type="number"] {
      padding: 8px;
      width: 100px;
      border-radius: 12px;
      border: 2px solid #00bfff;
      text-align: center;
      background: #fff;
    }

    .game-area {
      display: flex;
      justify-content: center;
      gap: 40px;
      margin-top: 30px;
      flex-wrap: wrap;
      align-items: flex-start;
    }

    .hand {
      background: #f0fcff;
      padding: 20px;
      border-radius: 20px;
      box-shadow: 0 4px 8px rgba(0,0,0,0.1);
      width: 300px;
      border: 2px solid #00ced1;
      position: relative;
    }

    .hand h2 {
      font-size: 22px;
      margin-bottom: 10px;
      color: #1e90ff;
    }

    .cards {
      font-size: 32px;
      margin: 10px 0;
      min-height: 40px;
    }

    .buttons {
      margin: 20px;
    }

    button {
      padding: 10px 20px;
      font-size: 16px;
      margin: 5px;
      border: none;
      border-radius: 12px;
      cursor: pointer;
      transition: transform 0.2s ease;
      box-shadow: 0 4px 6px rgba(0,0,0,0.1);
    }

    button:hover:not(:disabled) {
      transform: scale(1.05);
    }

    button:disabled {
      background-color: #ddd;
      cursor: not-allowed;
    }

    #start-btn { background-color: #00bfff; color: white; }
    #hit-btn { background-color: #87ceeb; color: white; }
    #stand-btn { background-color: #4682b4; color: white; }
    #restart-btn { background-color: #5f9ea0; color: white; }
    #reset-all-btn { background-color: #1e90ff; color: white; }

    #result {
      font-size: 22px;
      margin-top: 20px;
      font-weight: bold;
      white-space: pre-wrap;
      opacity: 0;
      transition: opacity 0.5s ease;
    }

    #result.show {
      opacity: 1;
    }

    .win { color: #32cd32; }
    .lose { color: #dc143c; }
    .draw { color: #1e90ff; }

    .card-animate {
      display: inline-block;
      opacity: 0;
      transform: translateY(-20px);
      animation: fadeSlideIn 0.4s ease-out forwards;
    }

    @keyframes fadeSlideIn {
      to {
        opacity: 1;
        transform: translateY(0);
      }
    }

    .result-label {
      font-size: 28px;
      font-weight: bold;
      margin-bottom: 5px;
      height: 32px;
    }

    .win-label { color: #32cd32; text-shadow: 1px 1px 2px #fff; }
    .lose-label { color: #dc143c; text-shadow: 1px 1px 2px #fff; }
    .draw-label { color: #1e90ff; text-shadow: 1px 1px 2px #fff; }

    .vs-text {
      font-size: 36px;
      font-weight: bold;
      color: #1e90ff;
      margin-top: 90px;
    }
/* スマホの横画面に対応 */
@media screen and (max-width: 768px) and (orientation: landscape) {
  .game-area {
    flex-direction: column;
    align-items: center;
    gap: 20px;
  }

  .vs-text {
    margin-top: 10px;
    margin-bottom: 10px;
  }

  .hand {
    width: 90vw;
  }

  .cards {
    font-size: 28px;
  }

  .buttons {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 10px;
  }
}
  </style>
</head>
<body>
  <h1>ブラックジャック</h1>
  <p class="status-bar">💰 所持金：$<span id="money">1000</span></p>

  <label>🎯 ベット額：
    <input type="number" id="bet-input" min="1" value="100">
  </label>

  <div class="game-area">
    <div class="hand">
      <div id="player-label" class="result-label"></div>
      <h2>あなたの手札</h2>
      <div id="player-cards" class="cards"></div>
      <p>合計: <span id="player-score">0</span></p>
    </div>

    <div class="vs-text">VS</div>

    <div class="hand">
      <div id="dealer-label" class="result-label"></div>
      <h2>ディーラーの手札</h2>
      <div id="dealer-cards" class="cards"></div>
      <p>合計: <span id="dealer-score">?</span></p>
    </div>
  </div>

  <div class="buttons">
    <button id="start-btn" onclick="startGame()">ゲーム開始</button>
    <button id="hit-btn" onclick="hit()" disabled>もう一枚引く</button>
    <button id="stand-btn" onclick="stand()" disabled>勝負</button>
    <button id="restart-btn" onclick="resetGame()" style="display:none;">勝負を続ける</button>
    <button id="reset-all-btn" onclick="resetAll()">最初からやり直す</button>
  </div>

  <p id="result"></p>

  <script>
    const suits = ['♠', '♥', '♦', '♣'];
    const ranks = ['2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K', 'A'];
    const values = {
      '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9,
      '10': 10, 'J': 10, 'Q': 10, 'K': 10, 'A': 11
    };

    let deck = [], playerHand = [], dealerHand = [];
    let isStand = false;
    let money = 1000;
    let bet = 100;

    function createDeck() {
      deck = [];
      for (let suit of suits) {
        for (let rank of ranks) {
          deck.push({ card: rank + suit, value: values[rank] });
        }
      }
      deck.sort(() => Math.random() - 0.5);
    }

    function calculateScore(hand) {
      let score = hand.reduce((sum, card) => sum + card.value, 0);
      let aceCount = hand.filter(card => card.card.startsWith('A')).length;
      while (score > 21 && aceCount > 0) {
        score -= 10;
        aceCount--;
      }
      return score;
    }

    function displayCards() {
      const playerCardsEl = document.getElementById("player-cards");
      const dealerCardsEl = document.getElementById("dealer-cards");

      playerCardsEl.innerHTML = '';
      dealerCardsEl.innerHTML = '';

      playerHand.forEach((card, i) => {
        const span = document.createElement("span");
        span.textContent = card.card;
        span.classList.add("card-animate");
        span.style.animationDelay = ${i * 0.1}s;
        playerCardsEl.appendChild(span);
      });

      dealerHand.forEach((card, i) => {
        const span = document.createElement("span");
        span.textContent = (isStand || i === 0) ? card.card : '🂠';
        span.classList.add("card-animate");
        span.style.animationDelay = ${i * 0.1}s;
        dealerCardsEl.appendChild(span);
      });

      document.getElementById("player-score").innerText = calculateScore(playerHand);
      document.getElementById("dealer-score").innerText = isStand ? calculateScore(dealerHand) : '?';
    }

    function updateMoneyDisplay() {
      document.getElementById("money").innerText = money;
    }

    function hit() {
      if (isStand) return;
      playerHand.push(deck.pop());
      displayCards();
      if (calculateScore(playerHand) > 21) {
        lose("バースト！あなたの負けです。");
      }
    }

    function stand() {
      if (isStand) return;
      isStand = true;
      while (calculateScore(dealerHand) < 17) {
        dealerHand.push(deck.pop());
      }
      displayCards();
      const playerScore = calculateScore(playerHand);
      const dealerScore = calculateScore(dealerHand);
      if (dealerScore > 21 || playerScore > dealerScore) {
        win("あなたの勝ちです！");
      } else if (playerScore < dealerScore) {
        lose("あなたの負けです。");
      } else {
        draw("引き分けです。");
      }
    }

    function win(msg) {
      money += bet;
      endGame(msg, "win");
    }

    function lose(msg) {
      money -= bet;
      endGame(msg, "lose");
    }

    function draw(msg) {
      endGame(msg, "draw");
    }

    function endGame(message, resultClass) {
      const resultElem = document.getElementById("result");
      resultElem.innerText = message;
      resultElem.className = resultClass + ' show';

      const playerLabel = document.getElementById("player-label");
      const dealerLabel = document.getElementById("dealer-label");

      playerLabel.innerText = '';
      dealerLabel.innerText = '';

      if (resultClass === 'win') {
        playerLabel.innerText = 'WIN';
        playerLabel.className = 'result-label win-label';
        dealerLabel.innerText = 'LOSE';
        dealerLabel.className = 'result-label lose-label';
      } else if (resultClass === 'lose') {
        playerLabel.innerText = 'LOSE';
        playerLabel.className = 'result-label lose-label';
        dealerLabel.innerText = 'WIN';
        dealerLabel.className = 'result-label win-label';
      } else {
        playerLabel.innerText = 'DRAW';
        dealerLabel.innerText = 'DRAW';
        playerLabel.className = 'result-label draw-label';
        dealerLabel.className = 'result-label draw-label';
      }

      document.getElementById("hit-btn").disabled = true;
      document.getElementById("stand-btn").disabled = true;
      document.getElementById("restart-btn").style.display = "inline-block";
      updateMoneyDisplay();

      if (money <= 0) {
        resultElem.innerText += "\n所持金がなくなりました。ゲーム終了。";
        document.getElementById("restart-btn").disabled = true;
        document.getElementById("start-btn").style.display = "none";
        document.getElementById("bet-input").disabled = true;
      }
    }

    function startGame() {
      bet = parseInt(document.getElementById("bet-input").value);
      if (isNaN(bet) || bet <= 0 || bet > money) {
        alert("正しいベット額を入力してください（1〜" + money + "）");
        return;
      }

      createDeck();
      playerHand = [deck.pop(), deck.pop()];
      dealerHand = [deck.pop(), deck.pop()];
      isStand = false;

      displayCards();
      updateMoneyDisplay();

      const result = document.getElementById("result");
      result.innerText = '';
      result.className = '';

      document.getElementById("player-label").innerText = '';
      document.getElementById("dealer-label").innerText = '';
      document.getElementById("player-label").className = 'result-label';
      document.getElementById("dealer-label").className = 'result-label';

      document.getElementById("hit-btn").disabled = false;
      document.getElementById("stand-btn").disabled = false;
      document.getElementById("start-btn").disabled = true;
      document.getElementById("bet-input").disabled = true;
    }

    function resetGame() {
      document.getElementById("hit-btn").disabled = true;
      document.getElementById("stand-btn").disabled = true;
      document.getElementById("start-btn").disabled = false;
      document.getElementById("restart-btn").style.display = "none";
      document.getElementById("bet-input").disabled = false;
      document.getElementById("player-cards").innerText = '';
      document.getElementById("dealer-cards").innerText = '';
      document.getElementById("player-score").innerText = '0';
      document.getElementById("dealer-score").innerText = '?';
      document.getElementById("player-label").innerText = '';
      document.getElementById("dealer-label").innerText = '';
      document.getElementById("result").innerText = '';
      document.getElementById("result").className = '';
    }

    function resetAll() {
      money = 1000;
      updateMoneyDisplay();
      resetGame();
      document.getElementById("start-btn").style.display = "inline-block";
      document.getElementById("restart-btn").disabled = false;
      document.getElementById("bet-input").disabled = false;
    }
  </script>
</body>
</html>
