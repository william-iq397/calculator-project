<template>
  <div id="body">
    <div id="container">
      <div id="result-screen">
        <span id="result-up-left">{{ result }}
          {{ result > 0 || result < 0 ? "&smt; result" : "" }}</span>
            {{ previousValue + operation + currentValue || 0 }}
      </div>
      <div id="button-container">
        <button @click="calculate(button)" id="button" v-for="(button, index) in buttons" :key="index">
          {{ button }}
        </button>
      </div>
    </div>
    <div id="history">
      <h3 v-for="(calculation, index) in historyList" :key="index" id="result-in-history">
        {{ calculation }}
      </h3>
      <span id="clear-button" @click="clearHistory">Clear</span>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import { useStorage, onKeyStroke } from "@vueuse/core";


const buttons = ref([
  "C",
  "π",
  "%",
  "Del",
  "1/x",
  "x²",
  "√x",
  "÷",
  7,
  8,
  9,
  "×",
  4,
  5,
  6,
  "-",
  1,
  2,
  3,
  "+",
  "+/-",
  0,
  ".",
  "=",
]);




// PARTS OF CALCULATION VARIABLES
let currentValue = ref("");
let previousValue = ref("");
let operation = ref("");
let resultValue = ref("");

// PROCESS OF CALCULATION AND RESULT VARIABLES
let processOfCalculation = ref("");
let result = ref("");

// RESULT IN HISTORY AND RESULT IN HISTORY STORAGE VARIABLES
let resultInHistory = ref("");
const historyStorage = useStorage('history', [])

const historyList = ref([]);

onMounted(() => (historyList.value = historyStorage.value))


// this function does everything
function calculate(button) {
  // adding numbers to the screen
  if (!isNaN(button) || button == ".") {
    if (currentValue.value.length <= 10) {
      currentValue.value += button;
    }
  }
  // clear the numbers
  if (button === "C") {
    operation.value = "";
    previousValue.value = "";
    currentValue.value = "";
    result.value = "";
  }
  // split between numbers before the operator and after the operator
  if (["×", "+", "-", "÷"].includes(button)) {
    previousValue.value = currentValue.value;
    currentValue.value += button;
    operation.value = button;
    currentValue.value = "";
  }
  // GETTING THE REST OF DIVIDING
  if (button === "%") {
    currentValue.value = currentValue.value / 100;
  }
  console.log(currentValue.value.length)

  // DELETE THE LAST NUMBER
  if (button === "Del") {
    currentValue.value = currentValue.value.substring(
      0,
      currentValue.value.length - 1
    );
  }

  // multiply the number by himself
  if (button === "x²") {
    currentValue.value = currentValue.value * currentValue.value
  }

  // the square root of the number
  if (button === '√x') {
    currentValue.value = Math.sqrt(currentValue.value)
  }

  // get the 1/X of the number
  if (button === "1/x") {
    currentValue.value = eval(1 / currentValue.value)
  }

  // if it is "π" then set the value to 3.14
  if (button === "π") {
    currentValue.value = "3.14"
  }

  // if the number is positive then set it to negative else set it to positive
  if (button === "+/-" && currentValue.value != 0) {
    currentValue.value = currentValue.value > 0 ? currentValue.value * -1 : currentValue.value * -1
  }

  // GET THE RESULT OF THE OPERATION
  if (button === "=") {

    // turning multiply into '*' to be easy calculated
    if (operation.value === "×") {
      operation.value = "*";
    }

    // turning divide into '/' to be easy calculated
    if (operation.value === "÷") {
      operation.value = '/'
    }

    // saving the process of calculation into one variable (not include the result)
    processOfCalculation.value =
      previousValue.value + " " + operation.value + " " + currentValue.value;

    // calculate( number before operation sign + the operation sign + number after operation sign)
    resultValue.value = eval(
      previousValue.value + operation.value + currentValue.value
    );

    // prepare the data to push to the history
    resultInHistory.value =
      processOfCalculation.value + " " + "=" + " " + resultValue.value;

    // push the data to the history if there is an operation
    if (operation.value) {
      historyList.value.push(resultInHistory.value);
    }

    // save the data into local storage
    // historyStorage = useStorage.setItem('result', resultInHistory.value);

    // SET THE RESULT TO THE span UP LEFT
    result.value = resultValue.value;
    // remove the numbers from the result bar WHEN CLICK THE EQUAL BUTTON
    previousValue.value = "";
    currentValue.value = "";
    operation.value = "";
  }
}

// CLEAR ALL THE CALCULATION FROM THE HISTORY
const clearHistory = () => {
  while (history.value.length > 0) history.value.pop();
};


// using keyboard keys

// adding numbers
const numbersKeys = ['1', '2', '3', '4', '5', '6', '7', '8', '9', '0', '.']

numbersKeys.forEach(numberKey => {
  onKeyStroke(numberKey, (e) => {
    if (currentValue.value.length <= 10) {
      if (currentValue.value.length === 0 && numberKey == '0') addNumber = false;
      else {
        currentValue.value += numberKey
      }
    }
  }, { eventName: 'keypress' })
})

// adding operations
const operationKeys = ['+', '*', '-', '/']

operationKeys.forEach(operatorKey => {
  onKeyStroke(operatorKey, (e) => {
    currentValue.value += operatorKey
  }, { eventName: 'keypress' })
})

// delete number
onKeyStroke('Backspace', (e) => {
  currentValue.value = currentValue.value.slice(
    0,
    currentValue.value.length - 1
  )
}, { eventName: 'keydown' })

// get the result using (= , Enter) keys
onKeyStroke('Enter', (e) => {
  // turning multiply into '*' to be easy calculated

  if (operation.value === "×") {
    operation.value = "*";
  }

  // turning divide into '/' to be easy calculated
  if (operation.value === "÷") {
    operation.value = '/'
  }

  // saving the process of calculation into one variable (not include the result)
  processOfCalculation.value =
    previousValue.value + " " + operation.value + " " + currentValue.value;


  // calculate( number before operation sign + the operation sign + number after operation sign)
  resultValue.value = eval(
    previousValue.value + operation.value + currentValue.value
  );
  // prepare the data to push to the history
  resultInHistory.value =
    processOfCalculation.value + " " + "=" + " " + resultValue.value;


  // push the data to the history if there is an operation

  historyList.value.push(resultInHistory.value);


  /* SAVING THE DATA OF THE HISTORY INTO THE localStorage */
  historyStorage.value = historyList.value

  // remove the numbers from the result bar WHEN CLICK THE EQUAL BUTTON
  previousValue.value = "";
  currentValue.value = "";
  operation.value = "";
}, { eventName: 'keyup' })

</script>

<style>
#body {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

#container {
  width: 40%;
  height: 70%;
  background: linear-gradient(0deg, rgb(5, 4, 4) 23%, #343434 80%);
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  align-items: center;
  border-radius: 10px;
}

#result-screen {
  width: 90%;
  height: 4rem;
  background: linear-gradient(36deg,
      rgba(27, 27, 27, 0.89) 50%,
      rgba(15, 15, 15, 0.726) 90%);
  border-radius: 10px;
  display: flex;
  justify-content: flex-end;
  align-items: center;
  font-size: 1.5rem;
  color: white;
  padding: 1rem;
  padding-top: 0.5rem;
  position: relative;
  transition: 0.3s all ease;
}

#button-container {
  width: 90%;
  height: 70%;
  background-color: linear-gradient(180deg,
      #343434 10%,
      rgba(255, 255, 255, 0) 10%);
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: repeat(6, 1fr);
  gap: 0.2rem;
  border-radius: 10px;
}

#button {
  font-family: "Franklin Gothic Medium", "Arial Narrow", Arial, sans-serif;
  background-color: linear-gradient(25deg,
      rgba(255, 255, 255, 0) 10%,
      10% #434343);
  color: white;
  font-size: 2rem;
  border-radius: 10px;
  transition: 0.3s all ease;
}

#button:hover {
  background-color: white;
  color: black;
  transition: 0.2s all ease;
}

#result-up-left {
  font-size: larger;
  width: 100%;
  height: 2em;
  position: absolute;
  top: 5px;
  left: 10px;
}

#history {
  width: 40%;
  height: 70%;
  background-color: #434343;
  display: flex;
  flex-direction: column;
  align-items: center;
  border-radius: 10px;
  margin-inline: 1rem;
  overflow-y: scroll;
  position: relative;
}

#result-in-history {
  padding: 0.2em;
  text-align: left;
  font-family: monospace;
  width: 100%;
  font-size: 1.5rem;
}

#clear-button {
  position: absolute;
  top: 10px;
  right: 10px;
  background-color: rgba(128, 128, 128, 0.534);
  width: 5rem;
  height: 2rem;
  display: grid;
  place-items: center;
  cursor: pointer;
  border-radius: 10px;
}

@media (max-width: 500px) {
  #body {
    font-size: 1rem !important;
  }
}

@media (max-width: 733px) {
  #body {
    flex-direction: column;
    gap: 1rem;
  }

  #container {
    width: 90%;
    margin-block: 1rem;
  }

  #history {
    width: 90%;
    margin-block: 1rem;
  }

  #result-up-left {
    width: 100%;
  }
}

@media (min-width: 1500px) {
  #button {
    font-size: 2.5rem;
  }

  #history {
    width: 25%;
  }

  #clear-button {
    padding: 2rem 4rem;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 1.5rem;
  }
}

::-webkit-scrollbar {
  width: 10px;
}

::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 10px;
}

::-webkit-scrollbar-thumb:hover {
  background: #555;
}
</style>
