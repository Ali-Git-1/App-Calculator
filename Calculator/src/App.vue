<template>
  <div class="app-container" :class="{ 'dark-theme': isDark }">
    <div class="calculator">
      <!-- نمایشگر -->
      <div class="screen-container">
        <input type="text" class="screen" :value="display" readonly />
      </div>

      <!-- دکمه‌ها -->
      <div class="keypad">
        <div class="btn-row">
          <button class="btn btn-clear" @click="clearAll">c</button>
          <button class="btn btn-action" @click="handlePercent">%</button>
          <button class="btn btn-action" @click="handleBackspace">
            <img src="./assets/delete.png" alt="back" />
          </button>
          <button class="btn btn-op" @click="setOperation('/')">÷</button>
        </div>

        <div class="btn-row">
          <button class="btn" @click="appendNumber(7)">7</button>
          <button class="btn" @click="appendNumber(8)">8</button>
          <button class="btn" @click="appendNumber(9)">9</button>
          <button class="btn btn-op" @click="setOperation('*')">×</button>
        </div>

        <div class="btn-row">
          <button class="btn" @click="appendNumber(4)">4</button>
          <button class="btn" @click="appendNumber(5)">5</button>
          <button class="btn" @click="appendNumber(6)">6</button>
          <button class="btn btn-op" @click="setOperation('-')">-</button>
        </div>

        <div class="btn-row">
          <button class="btn" @click="appendNumber(1)">1</button>
          <button class="btn" @click="appendNumber(2)">2</button>
          <button class="btn" @click="appendNumber(3)">3</button>
          <button class="btn btn-op" @click="setOperation('+')">+</button>
        </div>

        <div class="btn-row">
          <button class="btn btn-theme" @click="toggleTheme">T</button>
          <button class="btn" @click="appendNumber(0)">0</button>
          <button class="btn" @click="appendDot">.</button>
          <button class="btn btn-equal" @click="calculate">=</button>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup>
import { ref } from 'vue'

const display = ref('0')
const num1 = ref('')
const op = ref('')
const num2 = ref('')
const isEvaluated = ref(false) // مشخص می‌کند آیا مساوی زده شده یا نه
const isDark = ref(false)

// تبدیل کاراکتر کد به کاراکتر نمایشی
const getOpSymbol = (operator) => {
  if (operator === '/') return '÷'
  if (operator === '*') return '×'
  return operator
}

const appendNumber = (n) => {
  // اگر محاسبه تمام شده بود و عدد جدید زده شد، از اول شروع کن
  if (isEvaluated.value) {
    clearAll()
  }

  const str = n.toString()

  if (!op.value) {
    // در حال تایپ عدد اول
    num1.value = num1.value === '0' || !num1.value ? str : num1.value + str
    display.value = num1.value
  } else {
    // در حال تایپ عدد دوم
    num2.value = num2.value === '0' || !num2.value ? str : num2.value + str
    display.value = `${num1.value} ${getOpSymbol(op.value)} ${num2.value}`
  }
}

const appendDot = () => {
  if (isEvaluated.value) {
    clearAll()
  }

  if (!op.value) {
    if (!num1.value) num1.value = '0.'
    else if (!num1.value.includes('.')) num1.value += '.'
    display.value = num1.value
  } else {
    if (!num2.value) num2.value = '0.'
    else if (!num2.value.includes('.')) num2.value += '.'
    display.value = `${num1.value} ${getOpSymbol(op.value)} ${num2.value}`
  }
}

const setOperation = (operator) => {
  if (display.value === 'Error') return

  // اگر نتیجه قبلی وجود دارد، روی همان نتیجه عملیات جدید انجام بده
  if (isEvaluated.value) {
    isEvaluated.value = false
    num2.value = ''
  }

  // اگر عدد اول هنوز خالی بود
  if (!num1.value) {
    num1.value = '0'
  }

  // اگر کاربر عدد دوم را هم زده بود و دوباره عملگر زد، اول قبلی را حساب کن
  if (num2.value) {
    calculate()
    isEvaluated.value = false
  }

  op.value = operator
  display.value = `${num1.value} ${getOpSymbol(operator)}`
}

const calculate = () => {
  // بدون داشتن عملگر و عدد دوم، محاسبه انجام نمی‌شود
  if (!op.value || !num2.value || isEvaluated.value) return

  const n1 = parseFloat(num1.value)
  const n2 = parseFloat(num2.value)
  let result = 0

  switch (op.value) {
    case '+':
      result = n1 + n2
      break
    case '-':
      result = n1 - n2
      break
    case '*':
      result = n1 * n2
      break
    case '/':
      result = n2 !== 0 ? n1 / n2 : 'Error'
      break
  }

  // نمایش فرمت کامل: 82 + 10 = 92
  display.value = `${num1.value} ${getOpSymbol(op.value)} ${num2.value} = ${result}`

  if (result === 'Error') {
    num1.value = ''
    op.value = ''
    num2.value = ''
  } else {
    // نتیجه، عدد اول مرحله بعدی می‌شود
    num1.value = result.toString()
    op.value = ''
    num2.value = ''
  }

  isEvaluated.value = true
}

const clearAll = () => {
  display.value = '0'
  num1.value = ''
  op.value = ''
  num2.value = ''
  isEvaluated.value = false
}

const handlePercent = () => {
  if (isEvaluated.value) {
    isEvaluated.value = false
  }

  if (num2.value) {
    num2.value = (parseFloat(num2.value) / 100).toString()
    display.value = `${num1.value} ${getOpSymbol(op.value)} ${num2.value}`
  } else if (num1.value) {
    num1.value = (parseFloat(num1.value) / 100).toString()
    display.value = num1.value
  }
}

const handleBackspace = () => {
  if (isEvaluated.value) {
    clearAll()
    return
  }

  if (num2.value) {
    num2.value = num2.value.slice(0, -1)
    display.value = num2.value
      ? `${num1.value} ${getOpSymbol(op.value)} ${num2.value}`
      : `${num1.value} ${getOpSymbol(op.value)}`
  } else if (op.value) {
    op.value = ''
    display.value = num1.value
  } else if (num1.value) {
    num1.value = num1.value.slice(0, -1)
    display.value = num1.value || '0'
  }
}

const toggleTheme = () => {
  isDark.value = !isDark.value
}
</script>
<style scoped>
.app-container {
  width: 100vw;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f3f4f6;
  transition: background-color 0.3s ease;
}

.calculator {
  width: 320px;
  background-color: #ffffff;
  border-radius: 24px;
  padding: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
  display: flex;
  flex-direction: column;
  gap: 14px;
  transition: background-color 0.3s ease;
}

.screen {
  width: 100%;
  height: 65px;
  background-color: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 14px;
  font-size: 28px;
  text-align: right;
  padding: 0 14px;
  box-sizing: border-box;
  color: #1e293b;
  outline: none;
}

.keypad {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.btn-row {
  display: flex;
  gap: 10px;
}

.btn {
  flex: 1;
  height: 56px;
  border: none;
  border-radius: 14px;
  background-color: #f1f5f9;
  font-size: 20px;
  font-weight: 600;
  color: #1e293b;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: transform 0.1s;
}

.btn:active {
  transform: scale(0.95);
}

.btn img {
  width: 22px;
  height: 22px;
  object-fit: contain;
}

.btn-clear {
  background-color: #fecaca;
  color: #991b1b;
}
.btn-action {
  background-color: #e2e8f0;
  color: #334155;
}
.btn-op {
  background-color: #fed7aa;
  color: #9a3412;
}
.btn-equal {
  background-color: #bbf7d0;
  color: #166534;
}
.btn-theme {
  background-color: #e2e8f0;
}

.app-container.dark-theme {
  background-color: #0b0f19;
}

.app-container.dark-theme .calculator {
  background-color: #1e293b;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
}

.app-container.dark-theme .screen {
  background-color: #334155;
  border-color: #475569;
  color: #f8fafc;
}

.app-container.dark-theme .btn {
  background-color: #334155;
  color: #f8fafc;
}

.app-container.dark-theme .btn img {
  filter: invert(1);
}

.app-container.dark-theme .btn-clear {
  background-color: #dc2626;
  color: #fff;
}
.app-container.dark-theme .btn-op {
  background-color: #ea580c;
  color: #fff;
}
.app-container.dark-theme .btn-equal {
  background-color: #16a34a;
  color: #fff;
}
</style>
