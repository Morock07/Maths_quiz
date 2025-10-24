<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Math Quiz Game</title>
  <style>
/* === Final Merged Style === */
body {
  margin: 0;
  font-family: 'Poppins', sans-serif;
  background: linear-gradient(135deg, #f6d365 0%, #fda085 100%);
  color: #333;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.container {
  background-color: white; /* BACK TO WHITE BOX */
  padding: 30px 40px;
  border-radius: 20px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.1); /* soft shadow like 1st */
  text-align: center;
  width: 90%;
  max-width: 500px;
}

/* Heading like colorful style */
h1 {
  margin-bottom: 20px;
  color: #ff6f61;
  text-shadow: 0 0 5px rgba(255, 111, 97, 0.3);
}

/* Stats glow theme */
.stats {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
  font-size: 18px;
  color: #555;
}

/* Question */
#question {
  font-size: 24px;
  margin-bottom: 20px;
  font-weight: bold;
  color: #333;
}

/* Options vertical + button glow */
.options {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-top: 10px;
}

.options button {
  padding: 12px 20px;
  font-size: 18px;
  border: none;
  border-radius: 10px;
  background-color: #e66c6c;
  color: white;
  cursor: pointer;
  box-shadow: 0 0 10px ##d63031;
  transition: transform 0.2s, background-color 0.3s, box-shadow 0.3s;
}

.options button:hover {
  background-color: #d63031;
  transform: scale(1.05);
  box-shadow: 0 0 15px #d63031;
}

/* Right/Wrong Message */
.popup-message {
  margin-top: 20px;
  font-size: 18px;
  color: #00b894;
  font-weight: bold;
  min-height: 30px;
}

/* Start & Next Buttons */
#next-btn, #start-btn {
  margin-top: 25px;
  padding: 12px 24px;
  font-size: 18px;
  border: none;
  border-radius: 10px;
  background-color: #ff7675;
  color: white;
  cursor: pointer;
  box-shadow: 0 0 12px #ff7675;
  transition: transform 0.2s, background-color 0.3s;
}

#next-btn:hover, #start-btn:hover {
  background-color: #d63031;
  transform: scale(1.05);
}

/* Hide */
.hidden {
  display: none;
}

/* Summary Box */
#summary {
  margin-top: 25px;
  background-color: #f1f1f1;
  border-radius: 10px;
  padding: 15px;
  font-size: 16px;
  color: #333;
}

/* Navigation buttons */
.controls {
  margin-top: 15px;
}

.nav-btn {
  margin: 0 5px;
  padding: 8px 16px;
  font-size: 14px;
  border: none;
  border-radius: 8px;
  background-color: #74b9ff;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s;
}

.nav-btn:hover {
  background-color: #0984e3;
}
  </style>
</head>
<body>
  <div class="container">
    <h1 id="level-heading">Level 1</h1>
    <div id="completed-indicator" class="hidden" style="color: #00b894; font-weight: bold; margin-top: -10px; margin-bottom: 10px;">✓ Completed</div>
    <div class="stats">
      <div id="question-no">Q: 1</div>
      <div id="timer">Time Left: 12s</div>
    </div>
    <div id="question">Click Start to begin</div>
    <div class="options" id="options"></div>
    <div class="popup-message" id="popup"></div>
    
    <div id="level-complete-msg" class="popup-message hidden">
      <div style="font-size: 48px; margin-bottom: 10px;">🏆</div>
      Level Completed!
      <div id="level-stats" style="margin-top: 15px; font-size: 14px; color: #555; line-height: 1.6;">
        <div>⚡ Speed: <span id="avg-speed">0</span>s per question</div>
        <div>✅ Correct Answers: <span id="correct-count">0</span>/5</div>
        <div>⏱️ Total Time Used: <span id="time-used">0</span>s out of 60s</div>
      </div>
    </div>
    
    <div class="controls">
      <button id="prev-btn" class="nav-btn">« Prev Level</button>
      <button id="skip-btn" class="nav-btn">Skip Level</button>
      <button id="next-btn" class="nav-btn hidden">Next Level »</button>
    </div>
    
    <button id="start-btn">Start</button>
    
    <div style="margin-top: 30px; font-size: 12px; color: #888; border-top: 1px solid #eee; padding-top: 15px;">
      Created by :- Morock gamer
    </div>
  </div>

  <script>
const questionElement = document.getElementById('question');
const optionsElement = document.getElementById('options');
const popupElement = document.getElementById('popup');
const timerElement = document.getElementById('timer');
const questionNoElement = document.getElementById('question-no');
const levelHeading = document.getElementById('level-heading');
const startBtn = document.getElementById('start-btn');
const nextBtn = document.getElementById('next-btn');
const skipBtn = document.getElementById('skip-btn');
const prevBtn = document.getElementById('prev-btn');
const levelCompleteMsg = document.getElementById('level-complete-msg');
const completedIndicator = document.getElementById('completed-indicator');

let currentQuestion = 0;
let currentLevel = 1;
let timer;
let timeLeft = 12;
let score = 0;
let completedLevels = new Set(); // Track completed levels
let levelStartTime = 0;
let correctAnswersInLevel = 0;
let totalTimeUsedInLevel = 0;

// Generate 100 levels with progressively harder questions
function generateLevels() {
  const levels = {};
  
  for (let level = 1; level <= 100; level++) {
    const questions = [];
    
    for (let q = 0; q < 5; q++) {
      let question;
      
      if (level <= 10) {
        // Basic addition/subtraction (1-20)
        const operations = ['+', '-'];
        const op = operations[Math.floor(Math.random() * operations.length)];
        let a = Math.floor(Math.random() * 15) + 1;
        let b = Math.floor(Math.random() * 10) + 1;
        
        if (op === '-' && a < b) [a, b] = [b, a];
        
        const answer = op === '+' ? a + b : a - b;
        question = {
          q: `${a} ${op} ${b} =`,
          a: answer.toString(),
          o: generateOptions(answer, 4)
        };
      } else if (level <= 30) {
        // Addition/subtraction/multiplication (1-50)
        const operations = ['+', '-', '×'];
        const op = operations[Math.floor(Math.random() * operations.length)];
        let a, b, answer;
        
        if (op === '×') {
          a = Math.floor(Math.random() * 12) + 1;
          b = Math.floor(Math.random() * 12) + 1;
          answer = a * b;
        } else {
          a = Math.floor(Math.random() * 40) + 1;
          b = Math.floor(Math.random() * 30) + 1;
          if (op === '-' && a < b) [a, b] = [b, a];
          answer = op === '+' ? a + b : a - b;
        }
        
        question = {
          q: `${a} ${op} ${b} =`,
          a: answer.toString(),
          o: generateOptions(answer, 4)
        };
      } else if (level <= 60) {
        // All operations including division
        const operations = ['+', '-', '×', '÷'];
        const op = operations[Math.floor(Math.random() * operations.length)];
        let a, b, answer;
        
        if (op === '÷') {
          b = Math.floor(Math.random() * 12) + 1;
          answer = Math.floor(Math.random() * 20) + 1;
          a = b * answer;
        } else if (op === '×') {
          a = Math.floor(Math.random() * 15) + 1;
          b = Math.floor(Math.random() * 15) + 1;
          answer = a * b;
        } else {
          a = Math.floor(Math.random() * 80) + 1;
          b = Math.floor(Math.random() * 50) + 1;
          if (op === '-' && a < b) [a, b] = [b, a];
          answer = op === '+' ? a + b : a - b;
        }
        
        question = {
          q: `${a} ${op} ${b} =`,
          a: answer.toString(),
          o: generateOptions(answer, 4)
        };
      } else {
        // Advanced operations with larger numbers
        const operations = ['+', '-', '×', '÷'];
        const op = operations[Math.floor(Math.random() * operations.length)];
        let a, b, answer;
        
        if (op === '÷') {
          b = Math.floor(Math.random() * 20) + 2;
          answer = Math.floor(Math.random() * 50) + 1;
          a = b * answer;
        } else if (op === '×') {
          a = Math.floor(Math.random() * 25) + 1;
          b = Math.floor(Math.random() * 20) + 1;
          answer = a * b;
        } else {
          a = Math.floor(Math.random() * 200) + 1;
          b = Math.floor(Math.random() * 100) + 1;
          if (op === '-' && a < b) [a, b] = [b, a];
          answer = op === '+' ? a + b : a - b;
        }
        
        question = {
          q: `${a} ${op} ${b} =`,
          a: answer.toString(),
          o: generateOptions(answer, 4)
        };
      }
      
      questions.push(question);
    }
    
    levels[level] = questions;
  }
  
  return levels;
}

function generateOptions(correct, count) {
  const options = [correct];
  const range = Math.max(10, Math.floor(correct * 0.5));
  
  while (options.length < count) {
    let option = correct + (Math.floor(Math.random() * range * 2) - range);
    if (option < 0) option = Math.abs(option);
    if (!options.includes(option)) {
      options.push(option);
    }
  }
  
  // Shuffle options
  for (let i = options.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [options[i], options[j]] = [options[j], options[i]];
  }
  
  return options.map(String);
}

const levels = generateLevels();

function startLevel() {
  startBtn.classList.add('hidden');
  nextBtn.classList.add('hidden');
  levelCompleteMsg.classList.add('hidden');
  currentQuestion = 0;
  correctAnswersInLevel = 0;
  totalTimeUsedInLevel = 0;
  levelStartTime = Date.now();
  updateCompletedIndicator();
  showQuestion();
}

function showQuestion() {
  clearInterval(timer);
  timeLeft = 12;
  const questions = levels[currentLevel];
  if (currentQuestion >= questions.length) {
    completedLevels.add(currentLevel); // Mark level as completed
    showLevelStats();
    levelCompleteMsg.classList.remove('hidden');
    nextBtn.classList.remove('hidden');
    return;
  }
  
  const q = questions[currentQuestion];
  questionElement.textContent = q.q;
  questionNoElement.textContent = `Q: ${currentQuestion + 1}`;
  optionsElement.innerHTML = '';
  
  q.o.forEach(option => {
    const btn = document.createElement('button');
    btn.textContent = option;
    btn.onclick = () => checkAnswer(option, q.a);
    optionsElement.appendChild(btn);
  });
  
  startTimer();
}

function checkAnswer(selected, correct) {
  clearInterval(timer);
  const timeUsedForThisQuestion = 12 - timeLeft;
  totalTimeUsedInLevel += timeUsedForThisQuestion;
  
  if (selected === correct) {
    popupElement.textContent = "Correct!";
    correctAnswersInLevel++;
    score++;
  } else {
    popupElement.textContent = "Wrong!";
  }
  currentQuestion++;
  setTimeout(showQuestion, 1000);
}

function startTimer() {
  timerElement.textContent = `Time Left: ${timeLeft}s`;
  timer = setInterval(() => {
    timeLeft--;
    timerElement.textContent = `Time Left: ${timeLeft}s`;
    if (timeLeft <= 0) {
      clearInterval(timer);
      totalTimeUsedInLevel += 12; // Full time used when time runs out
      popupElement.textContent = "Time's up!";
      currentQuestion++;
      setTimeout(showQuestion, 1000);
    }
  }, 1000);
}

function updateCompletedIndicator() {
  if (completedLevels.has(currentLevel)) {
    completedIndicator.classList.remove('hidden');
  } else {
    completedIndicator.classList.add('hidden');
  }
}

function showLevelStats() {
  const avgSpeed = (totalTimeUsedInLevel / 5).toFixed(1);
  document.getElementById('avg-speed').textContent = avgSpeed;
  document.getElementById('correct-count').textContent = correctAnswersInLevel;
  document.getElementById('time-used').textContent = totalTimeUsedInLevel;
}

startBtn.onclick = startLevel;

nextBtn.onclick = () => {
  currentLevel++;
  if (currentLevel > 100) {
    alert("🎉 Congratulations! You've completed all 100 levels! You're a Math Master! 🏆");
    return;
  }
  levelHeading.textContent = `Level ${currentLevel}`;
  popupElement.textContent = '';
  updateCompletedIndicator();
  startLevel();
};

skipBtn.onclick = () => {
  nextBtn.click();
};

prevBtn.onclick = () => {
  if (currentLevel > 1) {
    currentLevel--;
    levelHeading.textContent = `Level ${currentLevel}`;
    popupElement.textContent = '';
    updateCompletedIndicator();
    startLevel();
  }
};
  </script>
</body>
</html>
