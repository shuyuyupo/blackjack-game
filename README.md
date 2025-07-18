<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ブラックジャック</title>
  <style>
    body {
      font-family: "Segoe UI", sans-serif;
      text-align: center;
      padding: 20px;
      margin: 0;
      background: linear-gradient(to right, #d0f0ff, #f0faff);
    }

    h1 {
      color: #0088cc;
      text-shadow: 1px 1px 2px #aaa;
    }

    .status-bar {
      margin-bottom: 10px;
      font-size: 18px;
    }

    .game-area {
      display: flex;
      justify-content: space-around;
      align-items: flex-start;
      flex-wrap: wrap;
      margin: 10px 0;
    }

    .hand {
      border: 3px solid #00bfff;
      border-radius: 15px;
      padding: 10px;
      width: 45%;
      background-color: #ffffffcc;
      position: relative;
      min-height: 140px;
      box-shadow: 2px 2px 8px #bbb;
      margin: 10px;
    }

    .label {
      font-weight: bold;
      font-size: 18px;
      margin-bottom: 8px;
      color: #0088cc;
    }

    .cards {
      margin: 8px 0;
    }

    .card {
      display: inline-block;
      margin: 0 3px;
      padding: 8px 12px;
      border: 2px solid #00bfff;
      border-radius: 8px;
      background-color: white;
      box-shadow: 1px 1px 4px #ccc;
      font-size: 20px;
    }

    .buttons {
      margin-top: 20px;
    }

    button {
      padding: 12px 20px;
      margin: 8px;
      font-size: 16px;
      border-radius: 8px;
      border: none;
      background-color: #00bfff;
      color: white;
      cursor: pointer;
      transition: background-color 0.2s;
    }

    button:hover {
      background-color: #0099cc;
    }

    .card-animate {
      animation: fadeIn 0.4s ease-in;
    }

    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(-10px); }
      to { opacity: 1; transform: translateY(0); }
    }

    .result-text {
      font-size: 20px;
      font-weight: bold;
      margin-top: 8px;
    }

    .vs-text {
      font-size: 24px;
      font-weight: bold;
      color: #00aaff;
      text-shadow: 1px 1px 2px #ccc;
      margin: 10px 0;
    }

    /* --- モバイル縦画面用 --- */
    @media screen and (max-width: 600px) and (orientation: portrait) {
      .game-area {
        flex-direction: column;
        align-items: center;
      }

      .hand {
        width: 90%;
        padding: 6px;
        margin: 6px 0;
      }

      .card {
        margin: 0 2px;
        padding: 6px 10px;
        font-size: 16px;
      }

      .vs-text {
        margin: 4px 0;
        font-size: 20px;
      }

      .buttons {
        display: flex;
        flex-direction: column;
        gap: 8px;
        align-items: center;
      }

      button {
        width: 90%;
        max-width: 300px;
      }

      .status-bar {
        font-size: 16px;
      }

      input[type="number"] {
        width: 80px;
        font-size: 16px;
      }

      .label {
        font-size: 16px;
      }

      .result-text {
        font-size: 16px;
      }
    }

    /* --- スマホ横画面用 --- */
    @media screen and (max-width: 800px) and (orientation: landscape) {
      .game-area {
        flex-direction: row;
        flex-wrap: wrap;
        justify-content: center;
        gap: 10px;
      }

      .hand {
        width: 42%;
        padding: 6px;
        margin: 6px;
      }

      .card {
        font-size: 18px;
        padding: 6px 10px;
      }

      .vs-text {
        font-size: 22px;
        margin: 6px;
      }

      .buttons {
        flex-direction: row;
        flex-wrap: wrap;
        justify-content: center;
      }

      button {
        font-size: 14px;
        padding: 10px 14px;
        margin: 6px;
      }

      .status-bar {
        font-size: 14px;
      }

      input[type="number"] {
        width: 70px;
        font-size: 14px;
      }

      .label,
      .result-text {
        font-size: 16px;
      }
    }
  </style>
</head>
<body>
  <h1>ブラックジャック</h1>
  <div class="status-bar">
    所持金: <span id="money">1000</span> 円　
    ベット: <input type="number" id="bet-input" value="100" min="1" step="10"> 円
  </div>
  <div class="game-area">
    <div class="hand" id="player-hand">
      <div class="label">あなた</div>
      <div class="cards" id="player-cards"></div>
      <div class="result-text" id="player-result"></div>
    </div>
    <div class="vs-text">VS</div>
    <div class="hand" id="dealer-hand">
      <div class="label">ディーラー</div>
      <div class="cards" id="dealer-cards"></div>
      <div class="result-text" id="dealer-result"></div>
    </div>
  </div>
  <div class="buttons">
    <button onclick="startGame()">ゲーム開始</button>
    <button onclick="hit()">ヒット</button>
    <button onclick="stand()">スタンド</button>
  </div>

  <script>
    const suits = ['♥', '♦', '♣', '♠'];
    const ranks = ['2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K', 'A'];
    let deck, playerCards, dealerCards;
    let money = 1000;

    function createDeck() {
      const deck = [];
      for (let suit of suits) {
        for (let rank of ranks) {
          deck.push({ suit, rank, card: rank + suit });
        }
      }
      return deck.sort(() => Math.random() - 0.5);
    }

    function getCardValue(card) {
      if (['J', 'Q', 'K'].includes(card.rank)) return 10;
      if (card.rank === 'A') return 11;
      return parseInt(card.rank);
    }

    function calculateScore(cards) {
      let score = 0;
      let aces = 0;
      for (let card of cards) {
        score += getCardValue(card);
        if (card.rank === 'A') aces++;
      }
      while (score > 21 && aces > 0) {
        score -= 10;
        aces--;
      }
      return score;
    }

    function displayCards(cards, elementId) {
      const area = document.getElementById(elementId);
      area.innerHTML = '';
      for (let card of cards) {
        const span = document.createElement('span');
        span.className = 'card card-animate';
        span.innerText = card.card;
        span.style.color = (card.suit === '♥' || card.suit === '♦') ? 'red' : 'black';
        area.appendChild(span);
      }
    }

    function updateMoney() {
      document.getElementById('money').innerText = money;
    }

    function startGame() {
      const bet = parseInt(document.getElementById('bet-input').value);
      if (bet > money || bet <= 0) {
        alert("適切なベット額を入力してください");
        return;
      }

      deck = createDeck();
      playerCards = [deck.pop(), deck.pop()];
      dealerCards = [deck.pop(), deck.pop()];

      displayCards(playerCards, 'player-cards');
      displayCards(dealerCards, 'dealer-cards');

      document.getElementById('player-result').innerText = '';
      document.getElementById('dealer-result').innerText = '';
    }

    function hit() {
      playerCards.push(deck.pop());
      displayCards(playerCards, 'player-cards');
      const playerScore = calculateScore(playerCards);
      if (playerScore > 21) {
        endGame();
      }
    }

    function stand() {
      while (calculateScore(dealerCards) < 17) {
        dealerCards.push(deck.pop());
      }
      endGame();
    }

    function endGame() {
      const bet = parseInt(document.getElementById('bet-input').value);
      const playerScore = calculateScore(playerCards);
      const dealerScore = calculateScore(dealerCards);

      displayCards(dealerCards, 'dealer-cards');

      let resultPlayer = '';
      let resultDealer = '';

      if (playerScore > 21) {
        resultPlayer = 'LOSE';
        resultDealer = 'WIN';
        money -= bet;
      } else if (dealerScore > 21 || playerScore > dealerScore) {
        resultPlayer = 'WIN';
        resultDealer = 'LOSE';
        money += bet;
      } else if (playerScore < dealerScore) {
        resultPlayer = 'LOSE';
        resultDealer = 'WIN';
        money -= bet;
      } else {
        resultPlayer = 'DRAW';
        resultDealer = 'DRAW';
      }

      document.getElementById('player-result').innerText = resultPlayer;
      document.getElementById('dealer-result').innerText = resultDealer;
      updateMoney();
    }
  </script>
</body>
</html>
