<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ブラックジャック</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      text-align: center;
      padding: 20px;
      margin: 0;
      background: linear-gradient(to right, #e0f7fa, #ffffff);
    }
    h1 {
      color: #007acc;
      text-shadow: 1px 1px 2px #a0c4ff;
    }
    .game-area {
      display: flex;
      justify-content: center;
      align-items: flex-start;
      flex-wrap: wrap;
      gap: 20px;
      margin-bottom: 20px;
    }
    .hand {
      border: 3px solid #81d4fa;
      border-radius: 15px;
      padding: 10px;
      width: 280px;
      background-color: #ffffffee;
      box-shadow: 0 4px 8px rgba(0,0,0,0.1);
      position: relative;
    }
    .hand h2 {
      margin-top: 0;
    }
    .cards {
      min-height: 40px;
      margin-bottom: 10px;
    }
    .card-animate {
      animation: pop 0.3s ease;
    }
    @keyframes pop {
      0% { transform: scale(0.5); opacity: 0; }
      100% { transform: scale(1); opacity: 1; }
    }
    .buttons {
      margin: 20px 0;
    }
    button {
      background-color: #4fc3f7;
      border: none;
      color: white;
      padding: 12px 20px;
      margin: 5px;
      border-radius: 10px;
      cursor: pointer;
      font-size: 16px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
      transition: background-color 0.2s;
    }
    button:hover {
      background-color: #039be5;
    }
    .status-bar {
      margin-top: 10px;
      font-size: 18px;
    }
    .result {
      position: absolute;
      top: -25px;
      left: 50%;
      transform: translateX(-50%);
      font-size: 20px;
      font-weight: bold;
      color: #007acc;
      text-shadow: 1px 1px 2px white;
    }
    .vs-text {
      font-size: 28px;
      font-weight: bold;
      color: #007acc;
      margin: 10px 0;
    }

    /* ---------- スマホ対応 ---------- */
    @media screen and (max-width: 600px) and (orientation: portrait) {
      .game-area {
        flex-direction: column;
        align-items: center;
        gap: 10px;
      }
      .hand {
        width: 90%;
        margin: 5px 0;
      }
      .vs-text {
        margin: 0;
      }
    }

    @media screen and (max-width: 900px) and (orientation: landscape) {
      .game-area {
        flex-direction: row;
        flex-wrap: wrap;
        justify-content: center;
        align-items: flex-start;
        gap: 15px;
      }
      .hand {
        width: 42%;
      }
    }

  </style>
</head>
<body>
  <h1>ブラックジャック</h1>
  <div class="status-bar">
    所持金: <span id="money">1000</span>円 |
    ベット: <input type="number" id="bet-input" value="100" min="1" max="1000" style="width:80px;">
  </div>
  <div class="game-area">
    <div class="hand" id="player-hand">
      <div class="result" id="player-result"></div>
      <h2>あなた</h2>
      <div class="cards" id="player-cards"></div>
      <div>合計: <span id="player-total">0</span></div>
    </div>

    <div class="vs-text">VS</div>

    <div class="hand" id="dealer-hand">
      <div class="result" id="dealer-result"></div>
      <h2>ディーラー</h2>
      <div class="cards" id="dealer-cards"></div>
      <div>合計: <span id="dealer-total">0</span></div>
    </div>
  </div>
  <div class="buttons">
    <button onclick="startGame()">ゲーム開始</button>
    <button onclick="hit()">ヒット</button>
    <button onclick="stand()">スタンド</button>
  </div>

  <script>
    let playerCards = [];
    let dealerCards = [];
    let deck = [];
    let money = 1000;
    let bet = 100;
    let gameOver = false;

    function createDeck() {
      const suits = ['♠', '♥', '♦', '♣'];
      const values = ['A', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K'];
      const newDeck = [];
      for (const suit of suits) {
        for (const value of values) {
          newDeck.push({ suit, value, card: value + suit });
        }
      }
      return newDeck;
    }

    function shuffleDeck(deck) {
      for (let i = deck.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [deck[i], deck[j]] = [deck[j], deck[i]];
      }
    }

    function getCardValue(card) {
      if (['J', 'Q', 'K'].includes(card.value)) return 10;
      if (card.value === 'A') return 11;
      return parseInt(card.value);
    }

    function calculateTotal(cards) {
      let total = 0;
      let aces = 0;
      for (const card of cards) {
        const value = getCardValue(card);
        total += value;
        if (card.value === 'A') aces++;
      }
      while (total > 21 && aces > 0) {
        total -= 10;
        aces--;
      }
      return total;
    }

    function renderCards(cards, elementId) {
      const element = document.getElementById(elementId);
      element.innerHTML = '';
      for (const card of cards) {
        const span = document.createElement('span');
        span.textContent = card.card;
        span.className = 'card-animate';
        span.style.margin = '0 4px';
        span.style.fontSize = '24px';
        span.style.color = (card.card.includes('♥') || card.card.includes('♦')) ? 'red' : 'black';
        element.appendChild(span);
      }
    }

    function updateDisplay() {
      document.getElementById('player-total').textContent = calculateTotal(playerCards);
      document.getElementById('dealer-total').textContent = calculateTotal(dealerCards);
      document.getElementById('money').textContent = money;
      renderCards(playerCards, 'player-cards');
      renderCards(dealerCards, 'dealer-cards');
    }

    function startGame() {
      if (gameOver) {
        document.getElementById('player-result').textContent = '';
        document.getElementById('dealer-result').textContent = '';
        gameOver = false;
      }

      bet = parseInt(document.getElementById('bet-input').value);
      if (bet > money || bet <= 0) {
        alert("正しいベット額を入力してください。");
        return;
      }

      deck = createDeck();
      shuffleDeck(deck);
      playerCards = [deck.pop(), deck.pop()];
      dealerCards = [deck.pop(), deck.pop()];
      updateDisplay();
    }

    function hit() {
      if (gameOver) return;
      playerCards.push(deck.pop());
      updateDisplay();
      const total = calculateTotal(playerCards);
      if (total > 21) {
        endGame('lose');
      }
    }

    function stand() {
      if (gameOver) return;
      while (calculateTotal(dealerCards) < 17) {
        dealerCards.push(deck.pop());
      }
      updateDisplay();
      const playerTotal = calculateTotal(playerCards);
      const dealerTotal = calculateTotal(dealerCards);
      if (dealerTotal > 21 || playerTotal > dealerTotal) {
        endGame('win');
      } else if (playerTotal < dealerTotal) {
        endGame('lose');
      } else {
        endGame('draw');
      }
    }

    function endGame(result) {
      gameOver = true;
      if (result === 'win') {
        money += bet;
        document.getElementById('player-result').textContent = 'WIN';
        document.getElementById('dealer-result').textContent = 'LOSE';
      } else if (result === 'lose') {
        money -= bet;
        document.getElementById('player-result').textContent = 'LOSE';
        document.getElementById('dealer-result').textContent = 'WIN';
      } else {
        document.getElementById('player-result').textContent = 'DRAW';
        document.getElementById('dealer-result').textContent = 'DRAW';
      }
      updateDisplay();
    }

    updateDisplay();
  </script>
</body>
</html>
