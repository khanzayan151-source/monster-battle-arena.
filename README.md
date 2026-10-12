# monster-battle-arena.

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Monster Battle Arena</title>

<style>
* { box-sizing: border-box; }

body {
  margin: 0;
  font-family: Arial, sans-serif;
  color: white;
  text-align: center;
  background: linear-gradient(160deg, #111827, #301047, #071b32);
  min-height: 100vh;
}

header { padding: 22px 12px 8px; }

h1 {
  color: #ffd166;
  margin-bottom: 5px;
  font-size: clamp(26px, 7vw, 42px);
}

.subtitle { color: #cbd5e1; }

.arena {
  max-width: 720px;
  margin: 12px auto;
  padding: 16px;
}

.fighters {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.fighter {
  padding: 15px 8px;
  background: #ffffff12;
  border: 1px solid #ffffff30;
  border-radius: 18px;
}

.monster {
  font-size: clamp(58px, 17vw, 100px);
  margin: 8px 0;
  transition: transform 0.2s;
}

.player .monster { animation: float 2s infinite ease-in-out; }
.enemy .monster { animation: float 2.4s infinite ease-in-out; }

@keyframes float {
  50% { transform: translateY(-7px); }
}

.health-bg {
  height: 14px;
  background: #3f1727;
  border-radius: 20px;
  overflow: hidden;
  margin: 12px 0 5px;
}

.health {
  height: 100%;
  width: 100%;
  background: #45e080;
  transition: width 0.35s;
}

.enemy .health { background: #ff6464; }

.stats { color: #dbeafe; font-size: 14px; }

.vs {
  color: #ffd166;
  font-size: 25px;
  font-weight: bold;
  margin: 15px;
}

h2 { color: #ffd166; font-size: 21px; }

.controls {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  margin: 18px 0;
}

button {
  border: none;
  border-radius: 12px;
  padding: 14px 8px;
  font-size: 15px;
  font-weight: bold;
  cursor: pointer;
  color: #111827;
  background: #ffd166;
  touch-action: manipulation;
}

button:active { transform: scale(0.97); }

button:disabled {
  opacity: 0.45;
  cursor: not-allowed;
}

.special { background: #ff8fab; }
.defend { background: #7dd3fc; }
.restart { background: #86efac; width: 100%; }

#message {
  min-height: 58px;
  padding: 15px;
  background: #00000040;
  border-radius: 12px;
  line-height: 1.5;
}

#log {
  text-align: left;
  background: #00000035;
  border-radius: 12px;
  padding: 14px;
  min-height: 80px;
  line-height: 1.8;
  color: #e2e8f0;
  font-size: 14px;
}

footer {
  padding: 20px;
  color: #a5b4fc;
  font-size: 13px;
}

@media (max-width: 350px) {
  .fighter { padding: 10px 5px; }
  button { font-size: 13px; }
}
</style>
</head>

<body>
<header>
  <h1>⚔️ MONSTER BATTLE</h1>
  <div class="subtitle">Enter the arena. Fight for glory!</div>
</header>

<main class="arena">
  <div class="fighters">
    <section class="fighter player">
      <h2>YOUR DRAGON</h2>
      <div class="monster">🐉</div>
      <div id="playerHP" class="stats">HP: 100 / 100</div>
      <div class="health-bg">
        <div id="playerBar" class="health"></div>
      </div>
      <div class="stats">Fire Dragon</div>
    </section>

    <section class="fighter enemy">
      <h2>ENEMY TITAN</h2>
      <div class="monster">🦖</div>
      <div id="enemyHP" class="stats">HP: 100 / 100</div>
      <div class="health-bg">
        <div id="enemyBar" class="health"></div>
      </div>
      <div class="stats">Thunder Titan</div>
    </section>
  </div>

  <div class="vs">⚡ VS ⚡</div>

  <h2>Choose Your Move</h2>

  <div class="controls">
    <button onclick="attack('fire')">🔥 Fire Attack</button>
    <button onclick="attack('claw')">🐾 Claw Strike</button>
    <button class="special" onclick="attack('special')">
      💥 Dragon Blast
    </button>
    <button class="defend" onclick="attack('defend')">
      🛡️ Defend
    </button>
  </div>

  <div id="message" role="status" aria-live="polite">
    Your dragon is ready! Choose your first move.
  </div>

  <h2>📜 Battle Log</h2>
  <div id="log">The battle is about to begin...</div>

  <br>
  <button class="restart" onclick="restartGame()">
    🔄 Restart Battle
  </button>

  <p class="stats">Battles won: <strong id="wins">0</strong></p>
</main>

<footer>
  Monster Battle Arena | A student game development project
</footer>

<script>
let player = 100;
let enemy = 100;
let defending = false;
let gameOver = false;
let wins = 0;
let specialReady = true;

const $ = id => document.getElementById(id);

function updateUI() {
  $("playerHP").textContent = `HP: ${player} / 100`;
  $("enemyHP").textContent = `HP: ${enemy} / 100`;
  $("playerBar").style.width = player + "%";
  $("enemyBar").style.width = enemy + "%";

  $("playerBar").style.background =
    player < 30 ? "#ff6464" : "#45e080";

  $("enemyBar").style.background =
    enemy < 30 ? "#ff6464" : "#ff6464";

  document.querySelectorAll(".controls button")
    .forEach(button => button.disabled = gameOver);

  $("wins").textContent = wins;
}

function addLog(text) {
  $("log").innerHTML =
    text + "<br>" + $("log").innerHTML;

  // Keep the battle log short and readable.
  $("log").innerHTML =
    $("log").innerHTML.split("<br>").slice(0, 7).join("<br>");
}

function attack(move) {
  if (gameOver) return;

  let damage = 0;
  let message = "";

  if (move === "fire") {
    damage = Math.floor(Math.random() * 11) + 15;
    message = "🔥 Your Fire Attack deals " + damage + " damage!";
  }

  if (move === "claw") {
    damage = Math.floor(Math.random() * 9) + 10;
    message = "🐾 Your Claw Strike deals " + damage + " damage!";
  }

  if (move === "special") {
    if (!specialReady) {
      $("message").textContent =
        "Dragon Blast has already been used this battle!";
      return;
    }

    specialReady = false;
    damage = Math.floor(Math.random() * 11) + 25;
    message = "💥 DRAGON BLAST deals " + damage + " damage!";
    document.querySelector(".special").textContent =
      "💥 Blast Used";
  }

  if (move === "defend") {
    defending = true;
    message = "🛡️ You prepare to block the next attack!";
    addLog(message);
    $("message").textContent = message;
    enemyTurn();
    return;
  }

  enemy = Math.max(0, enemy - damage);
  addLog(message);
  updateUI();

  if (enemy <= 0) {
    finishGame(true);
    return;
  }

  $("message").textContent = message;
  enemyTurn();
}

function enemyTurn() {
  if (gameOver) return;

  let damage = Math.floor(Math.random() * 12) + 8;

  if (defending) {
    damage = Math.ceil(damage / 2);
    defending = false;
    addLog("🛡️ Your defence reduces the damage!");
  }

  player = Math.max(0, player - damage);

  const message = "🦖 The Titan attacks for " + damage + " damage!";
  addLog(message);
  updateUI();

  if (player <= 0) {
    finishGame(false);
  } else {
    $("message").textContent =
      message + " Your turn!";
  }
}

function finishGame(won) {
  gameOver = true;

  if (won) {
    wins++;
    $("message").textContent =
      "🏆 VICTORY! Your dragon defeated the Titan!";
    addLog("🏆 You won the battle!");
  } else {
    $("message").textContent =
      "💔 DEFEAT! The Titan won. Try another strategy!";
    addLog("💔 The Titan won this round.");
  }

  updateUI();
}

function restartGame() {
  player = 100;
  enemy = 100;
  defending = false;
  gameOver = false;
  specialReady = true;

  document.querySelector(".special").textContent =
    "💥 Dragon Blast";

  $("message").textContent =
    "A new battle begins! Choose your move.";

  $("log").textContent = "The monsters face each other again.";
  updateUI();
}

updateUI();
</script>
</body>
</html>
