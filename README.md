<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Algebra Block Blast - Enhanced</title>
<style>
  body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    margin: 0; padding: 0;
    background: #222;
    color: #eee;
    display: flex; flex-direction: column;
    height: 100vh;
    align-items: center; justify-content: center;
  }
  #loginScreen, #gameScreen {
    display: none;
    width: 100%; max-width: 480px;
    text-align: center;
  }
  #loginScreen.active, #gameScreen.active {
    display: block;
  }
  input[type="text"] {
    font-size: 1.2rem;
    padding: 0.5rem;
    width: 70%;
    border-radius: 6px;
    border: none;
  }
  button {
    font-size: 1.2rem;
    margin-left: 1rem;
    padding: 0.6rem 1rem;
    border-radius: 6px;
    border: none;
    background-color: #3cb371;
    color: white;
    cursor: pointer;
  }
  button:hover {
    background-color: #359e5a;
  }

  #question {
    font-size: 2rem;
    margin: 1.5rem 0 1rem 0;
  }

  #blocks {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 12px;
  }
  .block {
    background: #4a90e2;
    color: white;
    width: 80px;
    height: 80px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 1.8rem;
    border-radius: 10px;
    user-select: none;
    cursor: pointer;
    box-shadow: 0 0 10px #4a90e2;
    transition: transform 0.2s ease;
  }
  .block:hover {
    transform: scale(1.1);
    box-shadow: 0 0 15px #66b3ff;
  }
  .block.correct {
    background: #3cb371;
    box-shadow: 0 0 20px #3cb371;
  }
  .block.wrong {
    background: #d9534f;
    box-shadow: 0 0 20px #d9534f;
  }

  #scoreboard {
    margin-top: 1rem;
    font-size: 1.3rem;
  }
  #timer {
    margin-top: 0.5rem;
    font-size: 1.3rem;
    font-weight: bold;
  }

  #logoutBtn {
    margin-top: 20px;
    background-color: #d9534f;
  }
  #logoutBtn:hover {
    background-color: #b23b3b;
  }

  #leaderboard {
    margin-top: 1.5rem;
    background: #333;
    padding: 1rem;
    border-radius: 10px;
    max-height: 180px;
    overflow-y: auto;
    font-size: 1rem;
  }
  #leaderboard h3 {
    margin-top: 0;
  }
</style>
</head>
<body>

<div id="loginScreen" class="active">
  <h1>Welcome to Algebra Block Blast!</h1>
  <p>Enter your username to start:</p>
  <input type="text" id="usernameInput" placeholder="Your username" />
  <button id="loginBtn">Start Game</button>
</div>

<div id="gameScreen">
  <h2>Hello, <span id="displayUsername"></span>!</h2>
  <div id="question">Question will appear here</div>
  <div id="blocks"></div>
  <div id="scoreboard">Score: 0</div>
  <div id="timer">Time Left: 20s</div>
  <button id="logoutBtn">Logout</button>

  <div id="leaderboard">
    <h3>Leaderboard (Top Scores)</h3>
    <ol id="leaderboardList"></ol>
  </div>
</div>

<audio id="soundCorrect" src="https://actions.google.com/sounds/v1/cartoon/clang_and_wobble.ogg"></audio>
<audio id="soundWrong" src="https://actions.google.com/sounds/v1/cartoon/boing.ogg"></audio>

<script>
  const loginScreen = document.getElementById('loginScreen');
  const gameScreen = document.getElementById('gameScreen');
  const usernameInput = document.getElementById('usernameInput');
  const loginBtn = document.getElementById('loginBtn');
  const displayUsername = document.getElementById('displayUsername');
  const questionDiv = document.getElementById('question');
  const blocksDiv = document.getElementById('blocks');
  const scoreboard = document.getElementById('scoreboard');
  const logoutBtn = document.getElementById('logoutBtn');
  const timerDiv = document.getElementById('timer');
  const leaderboardList = document.getElementById('leaderboardList');
  const soundCorrect = document.getElementById('soundCorrect');
  const soundWrong = document.getElementById('soundWrong');

  let username = '';
  let score = 0;
  let currentAnswer = null;
  let timer = null;
  let timeLeft = 20; // seconds per question

  // Utility functions for random int in range inclusive
  function randInt(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
  }

  // Generate random algebra 1 problem
  // Types: multiplication or adding/subtracting integers
  function generateProblem() {
    const problemType = Math.random() < 0.5 ? 'multiplication' : 'integerAddSub';

    if (problemType === 'multiplication') {
      // Multiplication up to 12
      const a = randInt(1, 12);
      const b = randInt(1, 12);
      currentAnswer = a * b;
      return `${a} × ${b} = ?`;
    } else {
      // Adding or subtracting integers between -20 and 20
      const a = randInt(-20, 20);
      const b = randInt(-20, 20);
      const op = Math.random() < 0.5 ? '+' : '-';
      currentAnswer = op === '+' ? a + b : a - b;
      // Format nicely with parentheses for negatives
      const formatNum = n => (n < 0 ? `(${n})` : n);
      return `${formatNum(a)} ${op} ${formatNum(b)} = ?`;
    }
  }

  // Generate blocks with one correct answer and multiple wrong answers
  function generateBlocks(correctAnswer) {
    blocksDiv.innerHTML = '';

    // Put correct answer in a random block
    const answers = new Set();
    answers.add(correctAnswer);

    // Generate 5 wrong answers (total 6 blocks)
    while (answers.size < 6) {
      let wrong;
      // For wrong answers, pick near the correct answer +- 10
      wrong = correctAnswer + randInt(-10, 10);
      // Avoid duplicates and correct answer itself
      if (wrong !== correctAnswer) {
        answers.add(wrong);
      }
    }

    // Shuffle answers
    const answerArray = Array.from(answers);
    for (let i = answerArray.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [answerArray[i], answerArray[j]] = [answerArray[j], answerArray[i]];
    }

    // Create block elements
    answerArray.forEach(ans => {
      const block = document.createElement('div');
      block.className = 'block';
      block.textContent = ans;
      block.addEventListener('click', () => handleBlockClick(block, ans));
      blocksDiv.appendChild(block);
    });
  }

  function handleBlockClick(block, ans) {
    if (ans === currentAnswer) {
      block.classList.add('correct');
      playSound(true);
      score++;
      scoreboard.textContent = `Score: ${score}`;
      clearInterval(timer);
      setTimeout(nextQuestion, 800);
    } else {
      block.classList.add('wrong');
      playSound(false);
      score = Math.max(0, score - 1);
      scoreboard.textContent = `Score: ${score}`;
      // Shake block as feedback (optional)
      block.style.animation = 'shake 0.3s';
      block.addEventListener('animationend', () => {
        block.style.animation = '';
      }, {once: true});
    }
  }

  function nextQuestion() {
    questionDiv.textContent = generateProblem();
    generateBlocks(currentAnswer);
    resetTimer();
  }

  function resetTimer() {
    clearInterval(timer);
    timeLeft = 20;
    timerDiv.textContent = `Time Left: ${timeLeft}s`;
    timer = setInterval(() => {
      timeLeft--;
      timerDiv.textContent = `Time Left: ${timeLeft}s`;
      if (timeLeft <= 0) {
        clearInterval(timer);
        // Penalize score and auto-next question
        score = Math.max(0, score - 1);
        scoreboard.textContent = `Score: ${score}`;
        playSound(false);
        nextQuestion();
      }
    }, 1000);
  }

  function playSound(correct) {
    if (correct) {
      soundCorrect.currentTime = 0;
      soundCorrect.play();
    } else {
      soundWrong.currentTime = 0;
      soundWrong.play();
    }
  }

  // Save and load high scores in localStorage
  function saveHighScore(name, score) {
    let leaderboard = JSON.parse(localStorage.getItem('algebraBlockBlastLeaderboard')) || [];
    // Check if user already has a high score saved, update if current score higher
    const existing = leaderboard.find(item => item.name === name);
    if (existing) {
      if (score > existing.score) {
        existing.score = score;
      }
    } else {
      leaderboard.push({name, score});
    }
    // Sort leaderboard descending by score
    leaderboard.sort((a,b) => b.score - a.score);
    // Keep top 10 only
    leaderboard = leaderboard.slice(0,10);
    localStorage.setItem('algebraBlockBlastLeaderboard', JSON.stringify(leaderboard));
    return leaderboard;
  }

  function loadLeaderboard() {
    const leaderboard = JSON.parse(localStorage.getItem('algebraBlockBlastLeaderboard')) || [];
    leaderboardList.innerHTML = '';
    if (leaderboard.length === 0) {
      leaderboardList.innerHTML = '<li>No scores yet.</li>';
    } else {
      leaderboard.forEach(item => {
        const li = document.createElement('li');
        li.textContent = `${item.name}: ${item.score}`;
        leaderboardList.appendChild(li);
      });
    }
  }

  loginBtn.addEventListener('click', () => {
    const name = usernameInput.value.trim();
    if (name.length < 1) {
      alert('Please enter a username!');
      return;
    }
    username = name;
    displayUsername.textContent = username;
    score = 0;
    scoreboard.textContent = 'Score: 0';
    loginScreen.classList.remove('active');
    gameScreen.classList.add('active');
    loadLeaderboard();
    nextQuestion();
  });

  logoutBtn.addEventListener('click', () => {
    clearInterval(timer);
    // Save score to leaderboard before logout
    saveHighScore(username, score);
    loadLeaderboard();

    username = '';
    usernameInput.value = '';
    loginScreen.classList.add('active');
    gameScreen.classList.remove('active');
  });

  // Optional shake animation for wrong answers
  const styleSheet = document.createElement('style');
  styleSheet.textContent = `
    @keyframes shake {
      0% { transform: translateX(0); }
      20% { transform: translateX(-5px); }
      40% { transform: translateX(5px); }
      60% { transform: translateX(-5px); }
      80% { transform: translateX(5px); }
      100% { transform: translateX(0); }
    }
  `;
  document.head.appendChild(styleSheet);
</script>

</body>
</html>
