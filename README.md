<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ブラックジャック</title>
  <style>
    body {
      font-family: sans-serif;
      text-align: center;
      padding: 20px;
      margin: 0;
      background: #f9f9f9;
    }

    h1 {
      font-size: 2rem;
      margin-bottom: 0.5em;
    }

    .money-area, .bet-area, .button-area, #result {
      margin: 10px 0;
    }

    input[type="number"] {
      padding: 8px;
      width: 120px;
      font-size: 1rem;
      margin-top: 5px;
    }

    button {
      padding: 10px 20px;
      font-size: 1rem;
      margin: 5px;
      border: none;
      border-radius: 5px;
      background-color: #007BFF;
      color: white;
      cursor: pointer;
      transition: transform 0.2s ease;
    }

    button:hover:not(:disabled) {
      transform: scale(1.05);
      background-color: #0056b3;
    }

    button:disabled {
      background-color: #ccc;
      cursor: not-allowed;
    }

    .cards {
      font-size: 2rem;
      min-height: 2.5em;
      display: flex;
      justify-content: center;
      gap: 0.5em;
      flex-wrap: wrap;
      margin-bottom: 5px;
    }

    #result {
      font-weight: bold;
      font-size: 1.2rem;
      opacity: 0;
      transition: opacity 0.5s ease;
      white-space: pre-wrap;
    }

    #result.show {
      opacity: 1;
    }

    .card-animate {
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

    @media (max-width: 600px) {
      body {
        padding: 10px;
      }

      h1 {
        font-size: 1.5rem;
      }

      .cards {
        font-size: 1.5rem;
      }

      button, input[type="number"] {
        font-size: 0.9rem;
        width: 100%;
        max-width: 200px;
      }
    }
  </style>
</head>
<body>
  <h1>ブラックジャック</h1>

  <div class="money-area">
    所持金: <span id="money">1000</span> 円
  </div>

  <div class="bet-area">
    <label for="bet-input">ベット額:</label><br>
    <input type="number" id="bet-input" min="1" max="1000" value="100">
  </div>

  <div class="button-area">
    <button id="start-btn">ゲーム開始</button>
    <button id="hit-btn" disabled>ヒット</button>
    <button id="stand-btn" disabled>スタンド</button>
    <button id="restart-btn" style="display:none;">リスタート</button>
  </div>

  <h2>プレイヤー</h2>
  <div class="cards" id="player-cards"></div>
  <div>スコア: <span id="player-score">0</span></div>

  <h2>ディーラー</h2>
  <div class="cards" id="dealer-cards"></div>
  <div>スコア: <span id="dealer-score">0</span></div>

  <div id="result"></div>

  <script>
    let deck, playerHand, dealerHand, isStand = false, money = 1000, bet = 0;

    const suits = ["♠", "♥", "♦", "♣"];
    const ranks = ["A", "2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K"];
    const values = {"A": 11, "2": 2, "3": 3, "4": 4, "5": 5, "6": 6, "7": 7, "8": 8, "9": 9, "10": 10, "J": 10, "Q": 10, "K": 10};

    function createDeck() {
      deck = [];
      for (let suit of suits) {
        for (let rank of ranks) {
          deck.push({ card: ${suit}${rank}, value: values[rank] });
        }
      }
      for (let i = deck.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [deck[i], deck[j]] = [deck[j], deck[i]];
      }
    }

    function drawCard() {
      return deck.pop();
    }

    function calculateScore(hand) {
      let score = hand.reduce((sum, card) => sum + card.value, 0);
      let aces = hand.filter(c => c.value === 11).length;
      while (score > 21 && aces--) score -= 10;
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
        span.textContent = (isStand || i === 0) ? card.card : "🂠";
        span.classList.add("card-animate");
        span.style.animationDelay = ${i * 0.1}s;
        dealerCardsEl.appendChild(span);
      });

      document.getElementById("player-score").textContent = calculateScore(playerHand);
      document.getElementById("dealer-score").textContent = isStand ? calculateScore(dealerHand) : "?";
    }

    function updateMoneyDisplay() {
      document.getElementById("money").textContent = money;
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

    document.getElementById("start-btn").onclick = () => {
      bet = parseInt(document.getElementById("bet-input").value);
      if (isNaN(bet) || bet <= 0 || bet > money) {
        alert("正しいベット額を入力してください。");
        return;
      }

      isStand = false;
      createDeck();
      playerHand = [drawCard(), drawCard()];
      dealerHand = [drawCard(), drawCard()];
      displayCards();

      document.getElementById("hit-btn").disabled = false;
      document.getElementById("stand-btn").disabled = false;
      document.getElementById("restart-btn").style.display = "none";
      document.getElementById("result").className = "";
      document.getElementById("result").textContent = "";

      updateMoneyDisplay();
    };

    document.getElementById("hit-btn").onclick = () => {
      playerHand.push(drawCard());
      displayCards();
      if (calculateScore(playerHand) > 21) {
        money -= bet;
        endGame("バースト！あなたの負け！", "lose");
      }
    };

    document.getElementById("stand-btn").onclick = () => {
      isStand = true;
      while (calculateScore(dealerHand) < 17) {
        dealerHand.push(drawCard());
      }
      displayCards();

      const playerScore = calculateScore(playerHand);
      const dealerScore = calculateScore(dealerHand);
      if (dealerScore > 21 || playerScore > dealerScore) {
        money += bet;
        endGame("あなたの勝ち！", "win");
      } else if (dealerScore === playerScore) {
        endGame("引き分け！", "draw");
      } else {
        money -= bet;
        endGame("あなたの負け！", "lose");
      }
    };

    document.getElementById("restart-btn").onclick = () => {
      document.getElementById("start-btn").click();
    };
  </script>
</body>
</html>
