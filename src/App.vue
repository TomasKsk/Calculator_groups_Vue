<template>
  <div id="calc-main" class="calc-grid">
    <div class="calc-display">
      <div id="menu-icon-place" class="storage-window">{{ state.menuIcon }}</div>
      <div class="calc-current">{{ state.calcDisp }}</div>
      <div class="calc-mem">{{ state.calcMem.join('') }}</div>
      <div><span class="save-button saveIcon" id="save-icon-place">{{ state.saveIco }}</span></div>
    </div>

    <div class="calc-keys">
      <button v-for="num in ['+', '-', 'x', '÷']" :key="num">{{ num }}</button>

      <button v-for="num in [7, 8, 9, 4, 5, 6, 1, 2, 3, 0]" :key="num">{{ num }}</button> 

      <button class="op5">.</button>
      <button id="AC-key" class="op6">{{ state.delKey }}</button>
      <button class="op7">=</button>
    </div>

    <div id="calc-mem-storage" class="calc-mem-storage">
      <div v-for="(item, key) in state.calcStorage" :key="key" class="storage-item" :id="key">
        <div>
          <h3>
            <span class="editable" data-type="header" :data-idparent="key">{{ item.name }}</span>
            <button class="delete-mem" data-type="deleteButton" :data-idparent="key">x</button>
          </h3>
        </div>
        <div v-for="(value, index) in item.calculation" :key="index">
          <template v-if="typeof value === 'number'">
            <strong>
              <span v-if="index === item.calculation.length - 1" :data-idparent="key" :data-index="index">{{ value }}</span>
              <span v-else class="editable" data-type="number" :data-idparent="key" :data-index="index">{{ value }}</span>
            </strong>
            <span v-if="index !== item.calculation.length - 1" class="editable" data-type="comment" :data-idparent="key" :data-index="index">{{ item.comments[index] }}</span>
            <button v-if="index === item.calculation.length - 1" data-type="addNum" :data-index="index" :data-idparent="key">Add</button>
          </template>
          <template v-else>
            <span v-if="index === item.calculation.length - 2">{{ value }}</span>
            <span v-else class="editable" data-type="operator" :data-idparent="key" :data-index="index">{{ value }}</span>
          </template>
        </div>
      </div>
  </div>
</div>

  
</template>

<script>
import { reactive, onMounted, onUnmounted } from 'vue';


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
      calcMemCount: 0,
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

    const handleDotClick = () => {
      if (!state.calcDisp.includes('.')) {
        state.calcDisp += '.';
      }
    };

    const handleOperandClick = (num, e) => {
      if (e.target.matches('button')) {
        if (state.calcDisp !== '') {
          if (state.calcMem.includes('=')) {
            state.calcOp = num;
            state.calcMem = [state.calcDisp, num];
            state.calcDisp = '';
          } else {
            state.calcOp = num;
            state.calcMem = [...state.calcMem, state.calcDisp, num];
            state.calcDisp = '';
          }
        } else {
          if (state.calcMem.length > 0) {
            state.calcOp = num;
            state.calcMem = [...state.calcMem.slice(0, -1), num];
          }
        };
      }
    };

    const parseOp = (num1, op, num2) => {
      num1 = +(num1);
      num2 = +(num2);
      if (op === '+') return num1 + num2;
      if (op === '-') return num1 - num2;
      if (op === 'x' || op === '*') return num1 * num2;
      if (op === '÷' || op === '/') return num1 / num2;
    }

    const recalc = (arr) => {
      let temp2 = [...arr];
      let temp = parseOp(temp2[0], temp2[1], temp2[2]);
      temp2.splice(0, 3);

      while (temp2.length > 0 && temp2[0] !== '=') {
        let cur = temp2.splice(0, 2);
        temp = parseOp(temp, cur[0], cur[1]);
      }
      return temp;
    }

    const handleEqualClick = (e) => {
      if (state.calcDisp !== '' && state.calcMem.length > 1) {
        let tempMem = [...state.calcMem, state.calcDisp];
        let tempDisp = recalc(tempMem);
        state.calcMem = [...state.calcMem, state.calcDisp, '=', tempDisp];
        state.calcDisp = tempDisp;
        state.saveIco = '<';
      }
    };

    const handleDel = (operand) => {
      if (operand === 'C' && state.calcDisp !== '') {
        state.calcDisp = '';
        state.delKey = state.calcMem.length > 0 ? 'CE' : 'C';
      } else if (operand === 'CE') {
        if (state.calcDisp !== '') {
          state.calcDisp = '';
          state.delKey = 'CE';
        } else {
          state.calcMem = [];
          state.delKey = 'C';
        };
      }
    };

    const saveCalc = () => {
      let name = `calc_${state.calcMemCount}`;
      let newCalc = state.calcMem.map(a => (typeof +(a) == 'number' && !isNaN(a)) ? +(a) : a);

      state.calcStorage = {
        ...state.calcStorage,
        [name]: {
          'calculation': newCalc,
          'comments': newCalc.slice(0,-2).map(a => (typeof a == 'number' && a !== '=') ? '...' : null).concat(null, null),
          'name': name
        }
      };

      state.calcMemCount += 1;
      state.saveIco = '';
      state.calcMem = [];
      state.calcDisp = '';
      console.log(state.calcStorage)
    };

    const handleMenu = (e) => {
      let sel = e.target.innerHTML;
      if (sel === '≡' || sel === 'x') {
        let sel = document.querySelector('.calc-mem-storage')
        let condi = sel !== null;
        state.menuIcon = state.menuIcon === '≡' ? 'x' : '≡';
        if (condi) return sel.classList.toggle('visible');
      }
    };

    const reindexKeys = (obj) => {
      const currKeys = Object.keys(obj);
      console.log(currKeys)
      const mapping = currKeys.reduce((acc, key, index) => {
        const newKey = `calc_${index}`;
        acc[key] = newKey;
        return acc;
      }, {});
      const updatedObj = Object.fromEntries(
        Object.entries(obj).map(([oldKey, value]) => {
          const newKey = mapping[oldKey];
          const currentName = value.name;

          const updatedName = currentName.includes('calc_') ? newKey : currentName;

          const updatedValue = {
            ...value,
            name: updatedName,
          };
          return [newKey, updatedValue];
        })
      );
      return updatedObj;
    }

    const handleDelete = (e) => {
      console.log('handling delete')
      const parent = e.target.dataset.idparent;
      const entries = Object.entries(state.calcStorage);
      console.log(entries);
      const filtered = entries.filter(([key]) => key !== parent);
      const newObj = Object.fromEntries(filtered);
      console.log(newObj)
      state.calcStorage = reindexKeys(newObj);
      const keys = Object.keys(state.calcStorage);
      state.calcMemCount = keys.length > 0 ? +(keys.pop().replace(/\D+/, '')) + 1 : 0;
    };

    const addNum = (e) => {
      const key = e.target.dataset.idparent;
      const updatedCalc = [...state.calcStorage[key]['calculation'].slice(0, -2), '+', 0, ...state.calcStorage[key]['calculation'].slice(-2)];
      const updatedComm = [...state.calcStorage[key]['comments'].slice(0, -2), null, '...', null, null];

      state.calcStorage = {
        ...state.calcStorage,
        [key]: {
          ...state.calcStorage[key],
          calculation: updatedCalc,
          comments: updatedComm,
        }
      };
    };

    const selectMe = (e) => {
      e.target.setAttribute('contenteditable', true);
      document.execCommand('selectAll', false, null);
    }

    const handleStorage = (e) => {
      const dataT = e.target.dataset.type;
      if (dataTypes.includes(dataT)) {
        selectMe(e);
      }
    };

    // Listeners for click events and conditionals
    const handleClick = (e) => {
      let num = e.target.innerHTML;
      let numId = e.target.id;
      let dataT = e.target.dataset;
      let type = e.target.dataset.type;

      if (dataT.type) {
        handleStorage(e);
      }; 

      if (!isNaN(+num)) {
        handleNumberClick(num, e);
      } else if (num === '.') {
        handleDotClick();
      } else if (operandArr.includes(num) && numId !== 'menu-icon-place' && type !== 'deleteButton') {
        handleOperandClick(num, e);
      } else if (num === '=' && numId !== 'menu-icon-place' && !state.calcMem.includes('=')) {
        handleEqualClick();
      } else if (['C', 'CE'].includes(num)) {
        handleDel(num);
      } else if (numId === 'save-icon-place') {
        saveCalc();
      } else if (numId === 'menu-icon-place') {
        handleMenu(e);
      } else if (type === 'deleteButton') {
        handleDelete(e);
      } else if (type === 'addNum') {
        addNum(e);
      }
    };

    const handleKey = (e) => {
      const data = e.target.dataset;
      const type = data.type;
      const parent = data.idparent;
      const index = data.index;
      const inner = e.target.innerHTML;

      if (e.key === 'Enter') {
        e.target.setAttribute('contenteditable', false);

        if (type === 'header') {
          state.calcStorage[parent]['name'] = inner;
        }

        if (type === 'comment') {
          state.calcStorage[parent]['comments'][index] = inner;
        }

        if (type === 'number') {
          const thisCalc = state.calcStorage[parent]['calculation'].map((a,b) => (b == index) ? +inner : a);
          const newObj = {
            ...state.calcStorage[parent],
            calculation: [...thisCalc.slice(0, -1), recalc(thisCalc)]
          };

          state.calcStorage = {
            ...state.calcStorage,
            [parent]: newObj
          };
          
        } 
      }
    };

    // Set up event listeners when the component is mounted
    onMounted(() => {
      document.querySelector('.calc-grid').addEventListener('click', handleClick);
      window.addEventListener('keydown', handleKey);
    });

    // Clean up event listeners when the component is unmounted
    onUnmounted(() => {
      document.querySelector('.calc-grid').removeEventListener('click', handleClick);
      window.removeEventListener('keydown', handleKey);
      console.log(state.calcStorage)
    });

    // Return the methods you want to expose to the template
    return { state, handleNumberClick, handleOperandClick, handleDel };
  },
}

</script>