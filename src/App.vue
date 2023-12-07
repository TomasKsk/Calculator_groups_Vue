<template>
  <div id="calc-main" class="calc-grid">
    <div class="calc-display">
      <div id="menu-icon-place" class="storage-window" @click="handleMenu">{{ state.menuIcon }}</div>
      <div class="calc-current">{{ state.calcDisp }}</div>
      <div class="calc-mem">{{ state.calcMem }}</div>
      <div><span class="save-button saveIcon" id="save-icon-place" @click="saveCalc">{{ state.saveIco }}</span></div>
    </div>

    <div class="calc-keys">
      <button v-for="num in ['+', '-', 'x', '÷']" :key="num">{{ num }}</button>

      <button v-for="num in [7, 8, 9, 4, 5, 6, 1, 2, 3, 0]" :key="num">{{ num }}</button> 

      <button class="op5" @click="handleDotClick">.</button>
      <button id="AC-key" class="op6" @click="handleDel(delKey)">{{ state.delKey }}</button>
      <button class="op7" @click="handleEqualClick">=</button>
    </div>
  </div>
</template>

<script>
import { reactive, onMounted, onUnmounted } from 'vue';
//import CalcMemory from '@/components/CalcMemory.vue';
//import CalcDisplay from '@/components/CalcDisplay.vue';
//import CalcKeys from '@/components/CalcKeys.vue';

const operandArr = ["÷", "x", "-", "+"];
const dataTypes = ['header', 'number', 'comment', 'operator'];

export default {
  watch: {
    calcStorage: {
      handler(newValue) {
        // local storage
      },
      deep: true
    },
  },
  setup() {
    const state = reactive({
      calcStorage: {},
      calcMem: [],
      menuIcon: '≡',
      saveIco: '',
      calcDisp: '',
      calcOp: '',
      delKey: 'C',
    });

    const handleNumberClick = (num, e) => {
      if (e.target.matches('button')) {
        if (!state.calcMem.includes('=')) {
          state.calcDisp += num;
          if (state.calcOp !== '') {
            state.calcOp = '';
          }
          if (state.delKey === 'CE') {
            state.delKey = 'C';
          }
        } else {
          if (state.delKey === 'CE') {
            state.delKey = 'C';
          }
          state.calcDisp = '';
          state.calcMem = [];
          state.calcDisp += num;
        }
      }
    };

    const handleOperandClick = (num, e) => {
      // Your logic for operand click
    };

    const handleClick = (e) => {
      let num = e.target.innerHTML;

      if (!isNaN(+num)) {
        handleNumberClick(num, e);
      }
    };

    // Set up event listeners when the component is mounted
    onMounted(() => {
      document.querySelector('.calc-grid').addEventListener('click', handleClick);
    });

    // Clean up event listeners when the component is unmounted
    onUnmounted(() => {
      document.querySelector('.calc-grid').removeEventListener('click', handleClick);
    });

    // Return the methods you want to expose to the template
    return { state, handleNumberClick, handleOperandClick };
  },
}

</script>