<script>
  // List of calculator buttons
  const calculatorButtons = ["C", "√", "⌫", "÷", "x", "-", "=", "+"];
  // List of digit buttons
  const digits = ["7", "8", "9", "4", "5", "6", "1", "2", "3", "0", "."];

  // Variables to store calculator state
  let displayToLCD = "";
  let firstNumber = "";
  let secondNumber = "";
  let operator = "";
  let lastSign = "";

  // Function to clear calculator memory
  const clearMemory = () => {
    firstNumber = "";
    secondNumber = "";
    operator = "";
  };

  // Function to handle button clicks
  const handleClick = (btnValue) => {
    // Clear memory if "C" button is clicked
    if (btnValue === "C") {
      clearMemory();
      displayToLCD = firstNumber;
      return;
    }

    // Limit the length of the first number to 10 digits
    if (firstNumber.length > 10) {
      firstNumber = firstNumber.substring(0, 10);
    }

    // Handle backspace button ("⌫")
    if (btnValue === "⌫") {
      if (secondNumber) {
        clearMemory();
        displayToLCD = "";
        return;
      }
      // Remove the last digit from the first number
      firstNumber = firstNumber.substring(0, firstNumber.length - 1);
      displayToLCD = firstNumber;
    }

    // Do nothing if "=" is clicked without any numbers or operators
    if (btnValue === "=" && !firstNumber && !secondNumber) return;

    // Handle consecutive "=" clicks
    if (lastSign === "=" && !digits.includes(btnValue)) {
      operator = "";
      firstNumber = "";
    }

    // Handle digit buttons
    if (digits.includes(btnValue)) {
      if (lastSign === "=") clearMemory();

      if (btnValue === "0" && firstNumber === "") return;

      // Handle decimal point
      if (btnValue === ".") {
        if (firstNumber.includes(".")) return;
        if (firstNumber === "") {
          firstNumber = "0.";
          displayToLCD = firstNumber;
          return;
        }
      }
      // Concatenate the digit to the first number
      firstNumber = `${firstNumber}${btnValue}`;
      displayToLCD = firstNumber;
    } else {
      // Handle non-digit buttons
      if (btnValue === "√") {
        // Calculate the square root of the current number
        if (!firstNumber && !secondNumber) return;
        const number = firstNumber || secondNumber;
        let result = Math.sqrt(parseFloat(number)).toString();
        result = result.substring(0, 11);
        secondNumber = result;
        displayToLCD = result;
        return;
      }

      // Handle evaluation buttons (+, -, *, /, =)
      handleEvaluation(btnValue);
    }

    // DEBUG
    // Log the current state
    // console.log(
    //   "First: ",
    //   firstNumber,
    //   "Second: ",
    //   secondNumber,
    //   "Sign: ",
    //   operator
    // );

    // Update the last sign
    lastSign = btnValue;
  };

  // Function to handle the evaluation of expressions
  const handleEvaluation = (sign) => {
    let result;

    if (["÷", "x", "-", "+"].includes(sign)) {
      // Perform the calculation for multiplication, division, addition, and subtraction
      result = makeEval(firstNumber, secondNumber);

      // Update the state for the next operation
      secondNumber = secondNumber ? secondNumber : firstNumber;
      firstNumber = "";
      operator = sign;

      // Display the result if available
      if (result) {
        secondNumber = result;
        firstNumber = "";
        displayToLCD = result;
      }
    } else if (sign === "=") {
      // Perform the calculation for equality
      result = makeEval(firstNumber, secondNumber);
      secondNumber = result ? result : firstNumber || secondNumber;

      // Display the result if available
      if (result) displayToLCD = result;
    }
  };

  // Function to perform arithmetic calculations
  function makeEval(firstNumber, secondNumber) {
    let result;
    if (secondNumber && firstNumber) {
      // Perform the calculation based on the operator
      if (operator === "+") {
        result = (
          parseFloat(secondNumber) + parseFloat(firstNumber)
        ).toString();
      } else if (operator === "x") {
        result = (
          parseFloat(secondNumber) * parseFloat(firstNumber)
        ).toString();
      } else if (operator === "÷") {
        result = (
          parseFloat(secondNumber) / parseFloat(firstNumber)
        ).toString();
      } else if (operator === "-") {
        result = (
          parseFloat(secondNumber) - parseFloat(firstNumber)
        ).toString();
      }
    }
    // Limit the length of the result
    return limitLengthNumber(result);
  }

  // Function to limit the length of a number
  function limitLengthNumber(number) {
    if (number && number.length > 10) return parseFloat(number).toPrecision(5);
    return number;
  }
</script>

<main id="app">
  <div class="container">
    <!-- LCD -->
    <input
      class="item"
      id="LCD"
      type="text"
      name="LCD"
      maxlength="11"
      bind:value={displayToLCD}
      readonly
    />

    <!-- Buttons -->
    {#each calculatorButtons as button (button)}
      <button
        class={`item button ${button === "C" ? "red-btn" : null} ${
          button === "=" ? "blue-btn" : null
        }`}
        on:click={() => handleClick(button)}>{button}</button
      >
    {/each}
    {#each digits as digit (digit)}
      <button class="item button" on:click={() => handleClick(digit)}
        >{digit}</button
      >
    {/each}
  </div>
</main>

<style>
  #app {
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .container {
    width: 320px;
    background-color: rgb(209, 209, 209);
    border-radius: 2px;
    padding: 2px;
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    grid-template-rows: repeat(6, 1fr);
    gap: 5px;
    color: rgb(228, 228, 228);
  }

  #LCD {
    width: 100%;
    height: 100%;
    outline: none;
    background: none;
    border: none;
    color: black;
    font-size: 3em;
    border-bottom: 1px solid rgb(182, 182, 182);
    padding: 0 10px;
    text-align: right;
  }

  .item:nth-child(1) {
    grid-column: 1 / -1;
  }

  .item:nth-child(9) {
    grid-row: 5 / span 2;
    grid-column: 4 / -1;
  }

  .item:nth-child(7) {
    grid-row: 4 / 5;
    grid-column: 4 / 5;
  }

  .item:nth-child(6) {
    grid-row: 3 / 4;
    grid-column: 4 / 5;
  }

  .item:nth-child(8) {
    grid-row: 6 / 7;
    grid-column: 3 / 4;
  }

  .button {
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 24px;
    font-weight: 500;
    outline: none;
    border: none;
    background-color: rgb(55, 78, 99);
    border-radius: 5%;
    cursor: pointer;
    transition:
      background-color 0.1s,
      box-shadow 0.3s;
  }

  .button:hover {
    background-color: rgb(41, 58, 73);
  }

  .button:active {
    background-color: rgb(61, 84, 102);
    font-size: 20px;
    box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
  }

  .red-btn {
    background-color: rgb(252, 53, 53);
  }

  .red-btn:hover {
    background-color: rgb(182, 36, 36);
  }

  .red-btn:active {
    background-color: rgb(226, 80, 80);
    font-size: 22px;
    box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
  }

  .blue-btn {
    background-color: rgb(33, 33, 202);
  }

  .blue-btn:hover {
    background-color: rgb(18, 18, 175);
  }

  .blue-btn:active {
    background-color: rgb(37, 37, 204);
    font-size: 22px;
    box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
  }
</style>
