<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Password Generator & Checker</title>
<style>
  body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: linear-gradient(to right, #74ebd5, #ACB6E5);
    display: flex;
    justify-content: center;
    align-items: flex-start;
    padding-top: 50px;
    margin: 0;
    min-height: 100vh;
  }
  .container {
    background: #fff;
    padding: 30px 30px;
    border-radius: 20px;
    box-shadow: 0 8px 30px rgba(0,0,0,0.25);
    width: 90%;
    max-width: 450px;
    transition: all 0.3s ease;
  }
  h1, h3 {
    text-align: center;
    margin-bottom: 15px;
  }
  input[type="text"], input[type="password"], input[type="number"] {
    width: 80%;
    padding: 12px;
    font-size: 16px;
    margin-bottom: 10px;
    border-radius: 8px;
    border: 1px solid #ccc;
  }
  label {
    display: block;
    margin: 8px 0;
    font-size: 14px;
  }
  button {
    padding: 10px 20px;
    font-size: 16px;
    border: none;
    border-radius: 10px;
    background: #4CAF50;
    color: white;
    cursor: pointer;
    margin-top: 10px;
    transition: 0.3s;
    box-shadow: 0 3px 6px rgba(0,0,0,0.2);
  }
  button:hover { background: #45a049; }
  .strength-bar {
    height: 12px;
    width: 100%;
    background: #eee;
    border-radius: 6px;
    margin: 5px 0 10px 0;
  }
  .strength-bar-fill {
    height: 100%;
    width: 0%;
    border-radius: 6px;
    transition: width 0.3s;
  }
  #saved-passwords {
    margin-top: 20px;
    text-align: left;
    background: #eee;
    padding: 12px;
    border-radius: 8px;
    max-height: 180px;
    overflow-y: auto;
  }
  .password-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 5px;
  }
  .copy-btn {
    background: #2196F3;
    padding: 5px 10px;
    font-size: 12px;
    border-radius: 6px;
    cursor: pointer;
    color: white;
    border: none;
    box-shadow: 0 2px 5px rgba(0,0,0,0.2);
  }
  .copy-btn:hover { background: #0b7dda; }
  .toggle-btn {
    padding: 5px 10px;
    margin-left: 5px;
    font-size: 12px;
    border-radius: 6px;
    border: none;
    cursor: pointer;
    background: #f39c12;
    color: white;
    box-shadow: 0 2px 5px rgba(0,0,0,0.2);
  }
  .toggle-btn:hover { background: #e67e22; }
  .clear-btn {
    margin-top: 10px;
    background: #ff4d4d;
  }
  .clear-btn:hover { background: #e60000; }

  /* Responsive design */
  @media(max-width: 500px){
    input[type="text"], input[type="password"], input[type="number"]{
      width: 100%;
    }
    .container{
      padding: 20px;
      width: 95%;
    }
    button{
      width: 100%;
    }
  }
</style>
</head>
<body>
<div class="container">
  <h1>Password Generator & Checker</h1>

  <!-- Generator Section -->
  <h3>Generate Random Password</h3>
  <label>Length: <input type="number" id="gen-length" value="12" min="4" max="32"></label>
  <label><input type="checkbox" id="gen-uppercase" checked> Include Uppercase</label>
  <label><input type="checkbox" id="gen-numbers" checked> Include Numbers</label>
  <label><input type="checkbox" id="gen-symbols" checked> Include Symbols</label>
  <button id="generate-btn">Generate Password</button>
  <input type="text" id="generated-password" placeholder="Generated password" readonly>
  <div class="strength-bar"><div id="gen-strength" class="strength-bar-fill"></div></div>
  <span id="gen-strength-text"></span>

  <hr>

  <!-- User Password Checker Section -->
  <h3>Check Your Password</h3>
  <div>
    <input type="password" id="user-password" placeholder="Enter your password">
    <button class="toggle-btn" id="toggle-user">Show</button>
  </div>
  <button id="check-btn">Check & Save</button>
  <div class="strength-bar"><div id="user-strength" class="strength-bar-fill"></div></div>
  <span id="user-strength-text"></span>

  <h3>Saved Passwords:</h3>
  <div id="saved-passwords">No passwords saved yet.</div>
  <button class="clear-btn" id="clear-btn">Clear All Saved Passwords</button>
</div>

<script>
const lowerChars = 'abcdefghijklmnopqrstuvwxyz';
const upperChars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
const numberChars = '0123456789';
const symbolChars = '!@#$%^&*()_+-=[]{}|;:,.<>?';

const genLength = document.getElementById('gen-length');
const genUpper = document.getElementById('gen-uppercase');
const genNumbers = document.getElementById('gen-numbers');
const genSymbols = document.getElementById('gen-symbols');
const generateBtn = document.getElementById('generate-btn');
const genPasswordInput = document.getElementById('generated-password');
const genStrengthBar = document.getElementById('gen-strength');
const genStrengthText = document.getElementById('gen-strength-text');

const userPasswordInput = document.getElementById('user-password');
const checkBtn = document.getElementById('check-btn');
const userStrengthBar = document.getElementById('user-strength');
const userStrengthText = document.getElementById('user-strength-text');
const toggleUserBtn = document.getElementById('toggle-user');

const savedDiv = document.getElementById('saved-passwords');
const clearBtn = document.getElementById('clear-btn');

let savedPasswords = JSON.parse(localStorage.getItem('savedPasswords')) || [];
updateSavedPasswords();

function generatePassword(length, includeUpper, includeNumbers, includeSymbols) {
  let charset = lowerChars;
  if(includeUpper) charset += upperChars;
  if(includeNumbers) charset += numberChars;
  if(includeSymbols) charset += symbolChars;

  let password = '';
  for(let i=0; i<length; i++){
    password += charset[Math.floor(Math.random() * charset.length)];
  }
  return password;
}

function calculateStrength(password) {
  let score = 0;
  if(password.length >= 8) score++;
  if(/[A-Z]/.test(password)) score++;
  if(/[0-9]/.test(password)) score++;
  if(/[^A-Za-z0-9]/.test(password)) score++;
  return score;
}

function updateStrengthBar(bar, score, textElement){
  const colors = ['#ff4d4d','#ff944d','#ffd11a','#4CAF50'];
  bar.style.width = (score*25)+'%';
  bar.style.background = colors[score-1] || '#eee';
  const labels = ['Weak','Weak','Medium','Strong'];
  textElement.textContent = labels[score-1] || '';
}

function updateSavedPasswords() {
  if(savedPasswords.length === 0){
    savedDiv.textContent = 'No passwords saved yet.';
  } else {
    savedDiv.innerHTML = savedPasswords.map((p,i) => `
      <div class="password-item">
        <span>${i+1}. ${p}</span>
        <button class="copy-btn" data-pw="${p}">Copy</button>
      </div>
    `).join('');
    document.querySelectorAll('.copy-btn').forEach(btn => {
      btn.addEventListener('click', () => {
        navigator.clipboard.writeText(btn.dataset.pw);
        alert('Password copied!');
      });
    });
  }
}

generateBtn.addEventListener('click', () => {
  const pwd = generatePassword(
    parseInt(genLength.value),
    genUpper.checked,
    genNumbers.checked,
    genSymbols.checked
  );
  genPasswordInput.value = pwd;
  updateStrengthBar(genStrengthBar, calculateStrength(pwd), genStrengthText);
});

checkBtn.addEventListener('click', () => {
  const pwd = userPasswordInput.value.trim();
  if(!pwd) return alert('Please enter a password!');
  const strength = calculateStrength(pwd);
  updateStrengthBar(userStrengthBar, strength, userStrengthText);

  savedPasswords.push(pwd);
  localStorage.setItem('savedPasswords', JSON.stringify(savedPasswords));
  updateSavedPasswords();
  userPasswordInput.value = '';
});

toggleUserBtn.addEventListener('click', () => {
  if(userPasswordInput.type === 'password'){
    userPasswordInput.type = 'text';
    toggleUserBtn.textContent = 'Hide';
  } else {
    userPasswordInput.type = 'password';
    toggleUserBtn.textContent = 'Show';
  }
});

clearBtn.addEventListener('click', () => {
  if(confirm('Are you sure you want to clear all saved passwords?')){
    savedPasswords = [];
    localStorage.removeItem('savedPasswords');
    updateSavedPasswords();
  }
});
</script>
</body>
</html>
