<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>ブラックジャック</title>
  <style>
    body {
      font-family: "Segoe UI", sans-serif;
      text-align: center;
      padding: 30px;
      background: #f3f4f6;
    }

    h1 {
      font-size: 36px;
      margin-bottom: 10px;
    }

    .status-bar {
      font-size: 20px;
      margin-bottom: 15px;
    }

    label, input[type="number"] {
      font-size: 18px;
    }

    input[type="number"] {
      padding: 5px;
      width: 80px;
      text-align: center;
      margin-left: 10px;
    }

    .game-area {
      display: flex;
      justify-content: center;
      gap: 40px;
      margin-top: 30px;
      flex-wrap: wrap;
    }

    .hand {
      background: white;
      padding: 20px;
      border-radius: 12px;
      box-shadow: 0 4px 8px rgba(0,0,0,0.1);
      width: 280px;
    }

    .hand h2 {
      font-size: 20px;
      margin-bottom: 10px;
    }

    .cards {
      font-size: 28px;
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
      border-radius: 8px;
      cursor: pointer;
      transition: transform 0.2s ease;
    }

    button:hover:not(:disabled) {
      transform: scale(1.05);
    }

    button:disabled {
      background-color: #ccc;
      cursor: not-allowed;
    }

    #start-btn { background-color: #3b82f6; color: white; }
    #hit-btn { background-color: #10b981; color: white; }
    #stand-btn { background-color: #f59e0b; color: white; }
    #restart-btn { background-color: #6b7280; color: white; }

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

    .win { color: green; }
    .lose { color: red; }
    .draw { color: orange; }

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
      <h2>あなたの手札</h2>
      <div id="player-cards" class="cards"></div>
      <p>合計: <span id="player-score">0</span></p>
    </div>

    <div class="hand">
      <h2>ディーラーの手札</h2>
      <div id="dealer-cards" class="cards"></div>
      <p>合計: <span id="dealer-score">?</span></p>
    </div>
  </div>

  <div class="buttons">
    <button id="start-btn" onclick="startGame()">ゲーム開始</button>
    <button id="hit-btn" onclick="hit()" disabled>もう一枚引く</button>
    <button id="stand-btn" onclick="stand()" disabled>勝負</button>
    <button id="restart-btn" onclick="resetGame()" style="display:none;">もう一度プレイ</button>
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
        span.style.animationDelay = `${i * 0.1}s`;
        playerCardsEl.appendChild(span);
      });

      dealerHand.forEach((card, i) => {
        const span = document.createElement("span");
        span.textContent = (isStand || i === 0) ? card.card : '🂠';
        span.classList.add("card-animate");
        span.style.animationDelay = `${i * 0.1}s`;
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
      const result = document.getElementById("result");
      result.innerText = '';
      result.className = '';
    }
  </script>
</body>
</html>
