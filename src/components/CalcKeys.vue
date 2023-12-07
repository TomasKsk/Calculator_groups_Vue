<template>
    <div class="calc-keys">
        <button class="op1" @click="handleOperandClick('+')">+</button>
        <button class="op2" @click="handleOperandClick('-')">-</button>
        <button class="op3" @click="handleOperandClick('x')">x</button>
        <button class="op4" @click="handleOperandClick('÷')">÷</button>
        <button @click="handleNumberClick('7')">7</button>
        <button @click="handleNumberClick('8')">8</button>
        <button @click="handleNumberClick('9')">9</button>
        <button @click="handleNumberClick('4')">4</button>
        <button @click="handleNumberClick('5')">5</button>
        <button @click="handleNumberClick('6')">6</button>
        <button @click="handleNumberClick('1')">1</button>
        <button @click="handleNumberClick('2')">2</button>
        <button @click="handleNumberClick('3')">3</button>
        <button @click="handleNumberClick('0')">0</button>
        <button class="op5" @click="handleDotClick">.</button>
        <button id="AC-key" class="op6" @click="handleDel(delKey)">{{ delKey }}</button>
        <button class="op7" @click="handleEqualClick">=</button>
    </div>
</template>

<script>
export default {
    props: {
        delKey: String,
    },
    methods: {
        handleNumberClick(num) {

        },
        handleDotClick() {

        },

    }
}
</script>