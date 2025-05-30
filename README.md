<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Basic Calculator</title>
<style>
  /* Reset and base styles */
  * {
    box-sizing: border-box;
  }
  body {
    background: linear-gradient(135deg, #667eea, #764ba2);
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    margin: 0;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  }
  .calculator {
    background: #1e1e2f;
    border-radius: 12px;
    box-shadow: 0 8px 24px rgba(0,0,0,0.4);
    width: 320px;
    max-width: 95vw;
    padding: 20px;
    color: #fff;
    display: flex;
    flex-direction: column;
  }
  .display {
    background: #2a2a40;
    height: 60px;
    border-radius: 8px;
    margin-bottom: 20px;
    color: #fff;
    font-size: 2.2rem;
    text-align: right;
    padding: 10px 15px;
    overflow-wrap: break-word;
    user-select: none;
  }
  .buttons {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    grid-gap: 12px;
  }
  button {
    background: #3c3c63;
    border: none;
    border-radius: 8px;
    color: #e0e0e0;
    font-size: 1.4rem;
    font-weight: 600;
    padding: 18px 0;
    cursor: pointer;
    transition: background 0.2s ease;
    user-select: none;
  }
  button:hover {
    background: #57578a;
  }
  button.operator {
    background: #ff8c00;
    color: white;
  }
  button.operator:hover {
    background: #ffa733;
  }
  button.span-two {
    grid-column: span 2;
  }
</style>
</head>
<body>
  <div class="calculator" role="application" aria-label="Basic calculator">
    <div id="display" class="display" aria-live="polite" aria-atomic="true" role="textbox" tabindex="0">0</div>
    <div class="buttons">
      <button class="span-two" id="clear" aria-label="Clear">C</button>
      <button id="backspace" aria-label="Backspace">⌫</button>
      <button class="operator" data-op="/" aria-label="Divide">÷</button>

      <button data-num="7">7</button>
      <button data-num="8">8</button>
      <button data-num="9">9</button>
      <button class="operator" data-op="*" aria-label="Multiply">×</button>

      <button data-num="4">4</button>
      <button data-num="5">5</button>
      <button data-num="6">6</button>
      <button class="operator" data-op="-" aria-label="Subtract">−</button>

      <button data-num="1">1</button>
      <button data-num="2">2</button>
      <button data-num="3">3</button>
      <button class="operator" data-op="+" aria-label="Add">+</button>

      <button data-num="0" class="span-two" aria-label="Zero">0</button>
      <button data-num=".">.</button>
      <button id="equals" class="operator" aria-label="Equals">=</button>
    </div>
  </div>

<script>
  (function() {
    const display = document.getElementById('display');
    const clearBtn = document.getElementById('clear');
    const backspaceBtn = document.getElementById('backspace');
    const equalsBtn = document.getElementById('equals');
    const buttons = document.querySelectorAll('.buttons button');
    
    let currentInput = '0';
    let lastInput = '';
    let resetDisplay = false;

    function updateDisplay() {
      display.textContent = currentInput;
    }

    function isOperator(char) {
      return ['+', '-', '*', '/'].includes(char);
    }

    function sanitizeExpression(expr) {
      // Prevent starting with operator other than minus
      if (expr.length === 0) return '0';
      // Replace multiple operators in a row with last one entered (except minus after operator)
      let result = '';
      for(let i = 0; i < expr.length; i++) {
        const c = expr[i];
        if (isOperator(c)) {
          if (i === 0 && c !== '-') {
            // ignore leading operator except minus
            continue;
          }
          // if previous char is operator, replace it unless c is minus after operator
          if (i > 0 && isOperator(expr[i-1])) {
            if (!(c === '-' && expr[i-1] !== '-')) {
              continue;
            }
          }
        }
        result += c;
      }
      return result || '0';
    }

    function appendCharacter(char) {
      if (resetDisplay) {
        // Start new input after equals
        if (isOperator(char)) {
          currentInput += char;
        } else {
          currentInput = char;
        }
        resetDisplay = false;
      } else {
        if (currentInput === '0' && char !== '.') {
          currentInput = char;
        } else {
          currentInput += char;
        }
      }
      currentInput = sanitizeExpression(currentInput);
      updateDisplay();
    }

    function clearAll() {
      currentInput = '0';
      resetDisplay = false;
      updateDisplay();
    }

    function backspace() {
      if (resetDisplay) {
        // After equals, backspace resets
        clearAll();
        return;
      }
      if (currentInput.length > 1) {
        currentInput = currentInput.slice(0, -1);
      } else {
        currentInput = '0';
      }
      updateDisplay();
    }

    function calculate() {
      try {
        // sanitize expression
        let expr = currentInput;
        expr = expr.replace(/×/g, '*').replace(/÷/g, '/');
        // Eval is unsafe generally but for this controlled input it's fine
        let result = Function('"use strict";return (' + expr + ')')();
        if (result === Infinity || result === -Infinity) {
          currentInput = 'Error';
        } else if (isNaN(result)) {
          currentInput = 'Error';
        } else {
          // Fix floating point precision issues by rounding if necessary
          if (typeof result === 'number') {
            result = Math.round((result + Number.EPSILON) * 1000000000) / 1000000000;
          }
          currentInput = result.toString();
        }
      } catch(e) {
        currentInput = 'Error';
      }

      resetDisplay = true;
      updateDisplay();
    }

    buttons.forEach(button => {
      // Digit or decimal buttons
      if (button.hasAttribute('data-num')) {
        button.addEventListener('click', e => {
          appendCharacter(button.getAttribute('data-num'));
        });
      }
      // Operator buttons
      if (button.classList.contains('operator') && button.id !== 'equals') {
        button.addEventListener('click', e => {
          appendCharacter(button.getAttribute('data-op'));
        });
      }
    });

    clearBtn.addEventListener('click', clearAll);
    backspaceBtn.addEventListener('click', backspace);
    equalsBtn.addEventListener('click', calculate);

    // Keyboard support
    document.addEventListener('keydown', (e) => {
      if ((e.key >= '0' && e.key <= '9') || e.key === '.') {
        e.preventDefault();
        appendCharacter(e.key);
      } else if (e.key === '+' || e.key === '-' || e.key === '*' || e.key === '/') {
        e.preventDefault();
        appendCharacter(e.key);
      } else if (e.key === 'Enter' || e.key === '=') {
        e.preventDefault();
        calculate();
      } else if (e.key === 'Backspace') {
        e.preventDefault();
        backspace();
      } else if (e.key.toLowerCase() === 'c') {
        e.preventDefault();
        clearAll();
      }
    });

  })();
</script>
</body>
</html>

