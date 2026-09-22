<template>
  <div class="app-container" :class="{ 'dark-theme': isDark }">
    <div class="calculator">
      <div class="display-container">
        <!-- بخش لیست تاریخچه -->
        <div class="history-list" ref="historyContainer" v-if="history.length > 0">
          <div v-for="(item, index) in history" :key="index" class="history-item">
            {{ item }}
          </div>
        </div>

        <!-- نمایشگر اصلی فعلی -->
        <div class="current-display">
          {{ display }}
        </div>
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
import { nextTick, onMounted, ref, watch } from 'vue'

const display = ref('0')
const num1 = ref('')
const op = ref('')
const num2 = ref('')
const isEvaluated = ref(false) // مشخص می‌کند آیا مساوی زده شده یا نه
const isDark = ref(false)
const history = ref([])

const historyContainer = ref(null)

const scrollToBottom = () => {
  nextTick(() => {
    if (historyContainer.value) {
      historyContainer.value.scrollTop = historyContainer.value.scrollHeight
    }
  })
}

// بارگذاری تاریخچه از حافظه محلی هنگام بالا آمدن برنامه
onMounted(async () => {
  const saved = localStorage.getItem('calc_history')
  if (saved) {
    history.value = JSON.parse(saved)
    await nextTick() // صبر می‌کنه تا لیست تاریخچه اول کامل توی صفحه رسم بشه
    scrollToBottom()
  }
})

// هر بار تاریخچه تغییر کرد، خودکار در حافظه ذخیره شود
watch(
  history,
  (newVal) => {
    localStorage.setItem('calc_history', JSON.stringify(newVal))
  },
  { deep: true },
)

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

  const expression = `${num1.value} ${getOpSymbol(op.value)} ${num2.value} = ${result}`
  display.value = expression

  if (result === 'Error') {
    num1.value = ''
    op.value = ''
    num2.value = ''
  } else {
    // افزودن محاسبه به تاریخچه (جدیدترین محاسبات در بالا یا پایین)
    history.value.push(expression)
    // اگر محاسبات زیاد شد، مثلا فقط ۱۰ تای آخر رو نگه دار
    if (history.value.length > 15) {
      history.value.shift()
    }

    num1.value = result.toString()
    op.value = ''
    num2.value = ''
  }

  isEvaluated.value = true
  scrollToBottom()
}
const clearHistory = () => {
  history.value = []
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
  // اگر محاسبه تمام شده باشد، به کاربر اجازه بده آخرین رقم نتیجه را پاک کند
  if (isEvaluated.value) {
    isEvaluated.value = false
    // num1 هم‌اکنون حاصل قبلی را دارد
    num1.value = num1.value.slice(0, -1)
    display.value = num1.value || '0'
    return
  }

  // اگر در حال نوشتن عدد دوم هستیم
  if (num2.value !== '') {
    num2.value = num2.value.slice(0, -1)
    display.value = num2.value
      ? `${num1.value} ${getOpSymbol(op.value)} ${num2.value}`
      : `${num1.value} ${getOpSymbol(op.value)}`
  }
  // اگر عدد دوم خالی است و عملگر وجود دارد
  else if (op.value !== '') {
    op.value = ''
    display.value = num1.value || '0'
  }
  // اگر فقط در حال نوشتن عدد اول هستیم
  else if (num1.value !== '') {
    num1.value = num1.value.slice(0, -1)
    display.value = num1.value || '0'
  }
}

const toggleTheme = () => {
  isDark.value = !isDark.value
}
</script>
<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html,
body,
#app {
  width: 100%;
  height: 100%;
  overflow: hidden; /* جلوگیری کامل از هرگونه اسکرول افقی/عمودی کل صفحه */
  touch-action: none; /* جلوگیری از رفتارهای ناخواسته اسکرول در وب‌ویوی موبایل */
}

/* شبکه دکمه‌ها به صورت گرید منظم */
.buttons-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 12px;
  width: 100%;
}
.app-container {
  width: 100%;
  height: 100%;
  height: 100dvh; /* هماهنگ با نوار استاتوس‌بار و ناوبری اندروید */
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f3f4f6;
  overflow: hidden;
}

.calculator {
  width: 100%;
  height: 100%; /* در موبایل کل صفحه رو پر می‌کنه */
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 16px;
  background-color: #ffffff;
}

/* برای کامپیوتر یا تبلت: حالت کارت با ابعاد فیکس به خودش بگیره */
@media (min-width: 480px) {
  .calculator {
    width: 360px;
    height: auto;
    max-height: 90vh;
    border-radius: 28px;
    box-shadow: 0 16px 36px rgba(0, 0, 0, 0.1);
    padding: 24px 20px;
  }
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

/* ردیف دکمه‌ها */
.btn-row {
  display: flex;
  gap: 14px;
  margin-bottom: 12px;
}
/* تنظیم کل صفحه ماشین حساب برای موبایل */
.calculator-app {
  width: 100%;
  max-width: 420px;
  min-height: 100vh;
  min-height: 100dvh; /* پشتیبانی دقیق از ارتفاع داینامیک موبایل */
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: env(safe-area-inset-top, 20px) 16px env(safe-area-inset-bottom, 20px) 16px;
  box-sizing: border-box;
}

.btn {
  flex: 1;
  aspect-ratio: 1;
  border-radius: 50%;
  border: none;
  font-size: 1.55rem;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  background-color: #f3f4f6; /* رنگ پس‌زمینه ملایم برای کلیدهای عدد */
  color: #1f2937;
  transition:
    transform 0.1s ease,
    background-color 0.2s ease;
}

.btn:active {
  transform: scale(0.92);
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
/* محفظه نمایشگر و تاریخچه */
.display-container {
  flex: 1; /* فضای خالی بالا را هوشمند پر می‌کند */
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  padding: 16px;
  border-radius: 20px;
  margin-bottom: 16px;
  background: var(--display-bg, rgba(0, 0, 0, 0.03));
  overflow: hidden;
}

.history-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  overflow-y: auto;
  max-height: 900px;
  margin-bottom: 8px;
  scroll-behavior: smooth;
  scrollbar-width: none; /* مخفی‌سازی در فایرفاکس */
  -ms-overflow-style: none; /* اینترنت اکسپلورر و اج */
}

/* مخفی‌سازی در کروم و سافاری */
.history-list::-webkit-scrollbar {
  display: none;
}

.history-item {
  font-size: 1rem;
  opacity: 0.6;
  text-align: right;
}

.history-item:hover {
  opacity: 0.9;
}
/* عدد جاری اصلی */
.current-display {
  font-size: clamp(2rem, 5vw, 0.5rem); /* تنظیم خودکار سایز فونت برای موبایل */
  font-weight: 700;
  text-align: right;
  word-break: break-all;
  line-height: 1.2;
}
</style>
