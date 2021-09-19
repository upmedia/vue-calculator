<template>
  <div class="bg-purple-700 w-80 rounded-3xl m-auto relative overflow-hidden">
    <div class="top flex justify-between mb-8 p-4">
      <button
        @click="showHistory"
        class="h-8 w-8 rounded-full flex items-center justify-center bg-purple-800 text-white"
        :class="{ 'opacity-50 cursor-default': historyData.length == 0 }"
        :diabled="historyData.length == 0"
      >
        <clock-icon class="h-5 w-5" />
      </button>

      <button
        @click="deleteValue"
        class="h-8 w-8 rounded-full flex items-center justify-center bg-purple-800 text-white"
      >
        <back-space-icon class="h-5 w-5" />
      </button>
    </div>

    <ul
      v-if="isHistory"
      class="absolute bottom-0 left-0 bg-white py-6 px-4 rounded-3xl h-96 w-80 z-50 overflow-y-auto"
    >
      <li v-for="operation in historyData" :key="operation.operations" class="border-b pb-2">
        <small>{{ operation.operations }}</small>
        <h2 v-text="operation.sum" />
      </li>
    </ul>

    <div class="middle text-center mb-12 h-24">
      <div class="overflow-x-hidden mt-4">
        <small
          v-if="selectedOperation"
          class="font-bold text-purple-400 text-xs"
        >
          {{ prevNum }} {{ selectedOperation }} {{ currentNum }}
        </small>
        <h1
          class="text-white text-[2.5rem] tracking-[-1px] font-semibold"
          v-text="currentNum"
        />
      </div>
    </div>

    <div class="bottom bg-white py-6 px-4 grid grid-cols-4 grid-flow-row gap-2 rounded-3xl">
      <button class="main-button" @click="pressed('c')">c</button>
      <button class="main-button text-gray-900" @click="pressed('()')">()</button>
      <button class="main-button text-gray-900" @click="pressed('%')">%</button>
      <button class="main-button text-gray-900" @click="pressed('/')">/</button>

      <button class="main-button" @click="pressed('7')">7</button>
      <button class="main-button" @click="pressed('8')">8</button>
      <button class="main-button" @click="pressed('9')">9</button>
      <button class="main-button text-gray-900" @click="pressed('*')">*</button>

      <button class="main-button" @click="pressed('4')">4</button>
      <button class="main-button" @click="pressed('5')">5</button>
      <button class="main-button" @click="pressed('6')">6</button>
      <button class="main-button text-gray-900" @click="pressed('-')">-</button>

      <button class="main-button" @click="pressed('1')">1</button>
      <button class="main-button" @click="pressed('2')">2</button>
      <button class="main-button" @click="pressed('3')">3</button>
      <button class="main-button text-gray-900" @click="pressed('+')">+</button>

      <button class="main-button" @click="negateValue">+/-</button>
      <button class="main-button" @click="pressed('0')">0</button>
      <button class="main-button" @click="pressed('.')">.</button>
      <button class="main-button" @click="pressed('=')">=</button>
    </div>


  </div>

</template>

<script>
import { ref, onMounted } from 'vue'
import ClockIcon from '@/components/icons/ClockIcon.vue'
import BackSpaceIcon from '@/components/icons/BackSpaceIcon.vue'

export default {
  components: { ClockIcon, BackSpaceIcon },

  setup() {
    let historyData = []
    const operations = ["+", "-", "*", "/", "%", "()"]
    const numbers = ["1", "2", "3", "4", "5", "6", "7", "8", "9", "0", "."]
    const currentNum = ref('')
    const prevNum = ref('')
    const selectedOperation = ref('')
    const total = ref('')
    const isHistory = ref(false)



    const pressed = (value) => {
      if (value === "=" || value === "Enter") {
        historyData.push({
          operations: `${prevNum.value} ${selectedOperation.value} ${currentNum.value}`,
          sum: eval(`${prevNum.value} ${selectedOperation.value} ${currentNum.value}`)
        })
        calculate()
      }
      else if (value.toLowerCase() === 'backspace') deleteValue()
      else if (value === "%") percentage()
      else if (value === "c") clear()
      else if (operations.includes(value)) applyOperation(value)
      else if (numbers.includes(value)) appendNumber(value)
    }

    const appendNumber = (value) => {
      currentNum.value = currentNum.value + value;
    }

    const applyOperation = (value) => {
      calculate();
      prevNum.value = currentNum.value;
      currentNum.value = "";
      selectedOperation.value = value;
    }

    const calculate = () => {
      if (selectedOperation.value === "*") multiply();
      if (selectedOperation.value === "()") multiply();
      else if (selectedOperation.value === "/") divide();
      else if (selectedOperation.value === "%") percentage();
      else if (selectedOperation.value === "-") subtract();
      else if (selectedOperation.value === "+") sum();
      prevNum.value = "";
      selectedOperation.value = "";
    }

    const multiply = () => {
      currentNum.value = prevNum.value * currentNum.value
      total.value = currentNum.value
    }

    const divide = () => {
      currentNum.value = prevNum.value / currentNum.value
      total.value = currentNum.value
    }

    const percentage = () => {
      currentNum.value = currentNum.value / 100
      total.value = currentNum.value
    }

    const subtract = () => {
      currentNum.value = prevNum.value - currentNum.value
      total.value = currentNum.value
    }

    const sum = () => {
      currentNum.value = +prevNum.value + +currentNum.value
      total.value = currentNum.value
    }

    const clear = () => {
      currentNum.value = ""
      prevNum.value = ""
      selectedOperation.value = ""
      currentNum.value = ""
    }

    const deleteValue = () => {
      if(currentNum.value.length !== 0 && currentNum.value !== '0') {
        currentNum.value = currentNum.value.slice(0, -1)
      }
    }

    const negateValue = () => {
      if(currentNum.value  === '0') {
        return currentNum.value  = '-0'
      }

      if(currentNum.value === '-') {
        return currentNum.value = '0'
      }

      if(currentNum.value === '0.') {
        return currentNum.value = `-${currentNum.value}`
      }

      if(currentNum.value === '-0.') {
        return currentNum.value = '0.'
      }

      currentNum.value = `${currentNum.value * -1}`
    }

    const showHistory = () => {
        if (historyData.length == 0) {
          return
        }

        isHistory.value = !isHistory.value;
    }

    const handleKeydown = (e) => {
        pressed(e.key)
    }

    onMounted(() => window.addEventListener('keydown', handleKeydown));

    return {
      currentNum,
      pressed,
      selectedOperation,
      prevNum,
      total,
      isHistory,
      showHistory,
      negateValue,
      deleteValue,
      historyData,
      ClockIcon,
    }
  }

}
</script>


