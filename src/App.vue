<template>
  <div class="header">
    <img alt="Spliter logo" class="logo" src="./assets/logo.svg" width="125" height="125" />
  </div>

  <div class="container">
    <div class="content-left">
      <form>
        <label>Bill</label>
        <div class="input-container">
          <span class="input-prefix">€</span>
          <input
            id="bill-input"
            type="number"
            step="0.01"
            placeholder="0"
            v-model.number="billAmount"
            @input="isInputMax5dig"
            @keypress="isInputMax5dig($event)"
          />
        </div>
      </form>

      <form>
        <label>Select Tip %</label>
        <div class="buttons-grid">
          <button
            v-for="(percentage, index) in tipPercentages"
            :key="index"
            :class="{ active: index === activeButtonIndex }"
            @click="setActiveButton(index)"
            @click.prevent="handleClick"
          >
            {{ percentage }}%
          </button>
          <button
            :key="index"
            :class="{ active: index === activeButtonIndex }"
            @click="setActiveButton(index)"
            @click.prevent="handleClick"
          >
            <div class="input-container">
              <span class="input-prefix">%</span>
              <input
                class="short-input"
                id="short-input-percentage"
                type="number"
                step="1"
                placeholder="0"
                v-model.number="customTipPercentage"
                @input="handleCustomTipPercentageInput"
                @keypress="isNumberInteger($event), isNumberMax100($event)"
                :class="{ active: customTipSelected }"
              />
            </div>
          </button>
          <button
            :key="index"
            :class="{ active: index === activeButtonIndex }"
            @click="setActiveButton(index)"
            @click.prevent="handleClick"
          >
            <div class="input-container">
              <span class="input-prefix">€</span>
              <input
                class="short-input"
                id="short-input-amount"
                type="number"
                step="0.01"
                placeholder="0"
                v-model.number="customTipAmount"
                @input="handleCustomTipAmountInput"
                @keypress="isCustomTipAmountMax5dig($event)"
                :class="{ active: customTipSelected }"
              />
            </div>
          </button>
        </div>
      </form>
      <div>
        <form>
          <label
            >Number of People
            <span class="error-message" v-if="peopleInputError">{{ peopleInputError }}</span>
          </label>
          <div class="input-container">
            <span class="input-prefix"
              ><img src="./assets/icon-person.svg" alt="Person icon"
            /></span>
            <input
              id="people-input"
              type="number"
              placeholder="0"
              step="1"
              min="1"
              v-model.number="numberOfPeople"
              @input="validateAndLimitPeopleInput"
              v-bind:class="{
                'input-valid': isValid,
                'input-invalid': !isValid
              }"
              @keypress="isNumberInteger($event), isInputMax5dig($event)"
            />
          </div>
        </form>
      </div>
    </div>

    <div class="content-right">
      <div>
        <form class="flex-form">
          <label>
            Bill Amount
            <p>/person</p>
          </label>
          <h1>€{{ bill.toFixed(2) }}</h1>
        </form>
        <form class="flex-form">
          <label>
            Tip Amount
            <p>/person</p>
          </label>
          <h1>€{{ tip.toFixed(2) }}</h1>
        </form>
        <form class="flex-form total-form">
          <label>
            Total
            <p>/person</p>
          </label>
          <h1>€{{ total.toFixed(2) }}</h1>
        </form>
      </div>
      <button class="reset-button" @click="resetAll">RESET</button>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, nextTick } from 'vue'

const billAmount = ref(null)
const selectedTipPercentage = ref(null)
const numberOfPeople = ref(1)
const bill = ref(null)
const tip = ref(null)
const total = ref(null)
const activeButtonIndex = ref(null)
const customTipSelected = ref(false)

const setActiveButton = (index) => {
  if (index === 4) {
    customTipSelected.value = true
    activeButtonIndex.value = null
    nextTick(() => {
      document.getElementById('custom-tip-percentage-input').focus()
    })
  } else if (index === 5) {
    customTipSelected.value = true
    activeButtonIndex.value = null
    nextTick(() => {
      document.getElementById('custom-tip-amount-input').focus()
    })
  } else {
    customTipSelected.value = false
    activeButtonIndex.value = index
    selectedTipPercentage.value = tipPercentages[index]
    customTipPercentage.value = null
    customTipAmount.value = null
  }
}

const tipPercentages = [5, 10, 15, 25]
const customTipPercentage = ref(null)
const customTipAmount = ref(null)

const handleCustomTipPercentageChange = () => {
  selectedTipPercentage.value = customTipPercentage.value
  activeButtonIndex.value = null
}

const handleCustomTipAmountChange = () => {
  selectedTipPercentage.value = customTipAmount.value
  activeButtonIndex.value = null
}

const calculateAll = () => {
  const billValue = billAmount.value || 0
  const peopleValue = numberOfPeople.value || 1
  const selectedTip = selectedTipPercentage.value || 0

  let tipAmount = 0
  if (customTipAmount.value) {
    tipAmount = customTipAmount.value
  } else {
    tipAmount = (billValue * selectedTip) / 100
  }

  tip.value = tipAmount / peopleValue || 0
  total.value = (billValue + tipAmount) / peopleValue || 0
  bill.value = billValue / peopleValue || 0
}

watch([billAmount, selectedTipPercentage, numberOfPeople], calculateAll, { immediate: true })

const resetAll = () => {
  billAmount.value = null
  selectedTipPercentage.value = null
  customTipAmount.value = null
  customTipPercentage.value = null
  numberOfPeople.value = 1
  total.value = null
  tip.value = null
  bill.value = null
  activeButtonIndex.value = null
  peopleInputError.value = ''
  isValid.value = true
}

const isInputMax5dig = (evt) => {
  let inputValue = evt.target.value

  // allow only one zero
  if (/^0[0-9]/.test(inputValue)) {
    evt.target.value = inputValue.replace(/^0+/, '0')
  }

  // only up to two decimal places for bill input
  if (inputValue.includes('.')) {
    const [integerPart, decimalPart] = inputValue.split('.')
    if (decimalPart.length > 2) {
      evt.target.value = `${integerPart}.${decimalPart.slice(0, 2)}`
    }
  }

  // max input 99999
  let parsedValue = parseFloat(inputValue) || 0
  if (parsedValue > 99999) {
    evt.preventDefault()
    evt.target.value = 99999
  }

  // update the ref based on the input field ID
  switch (evt.target.id) {
    case 'bill-input':
      billAmount.value = parseFloat(evt.target.value)
      break
    case 'people-input':
      numberOfPeople.value = parseFloat(evt.target.value)
      break
  }
}

const isNumberInteger = (evt) => {
  const keysAllowed = ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9']
  const keyPressed = evt.key
  if (!keysAllowed.includes(keyPressed)) {
    evt.preventDefault()
  }
}

const isNumberMax100 = (evt) => {
  let inputValue = parseFloat(evt.target.value) || 0 // ensure numeric value

  if (inputValue > 100) {
    evt.target.value = 100
    customTipPercentage.value = 100
    selectedTipPercentage.value = 100
  } else if (evt.target.value.length > 2) {
    evt.target.value = evt.target.value.slice(0, 2) // max 2 digits
  } else {
    customTipPercentage.value = inputValue
    selectedTipPercentage.value = inputValue
  }
}
const handleCustomTipPercentageInput = (evt) => {
  handleCustomTipPercentageChange()
  isNumberMax100(evt)
}

const isCustomTipAmountMax5dig = (evt) => {
  let inputValue = evt.target.value

  // max 2 decimal places
  if (inputValue.includes('.')) {
    const [integerPart, decimalPart] = inputValue.split('.')
    if (decimalPart.length > 2) {
      evt.target.value = `${integerPart}.${decimalPart.slice(0, 2)}`
    }
  }

  // input max 99999
  let parsedValue = parseFloat(inputValue) || 0
  if (parsedValue > 99999) {
    evt.preventDefault()
    evt.target.value = 99999
  }
  customTipAmount.value = parseFloat(evt.target.value)
}

const handleCustomTipAmountInput = (evt) => {
  handleCustomTipAmountChange()
  isCustomTipAmountMax5dig(evt)
}

const peopleInputError = ref('')
const isValid = ref(true)

const validateAndLimitPeopleInput = (evt) => {
  let inputValue = evt.target.value

  // validate number of people (can't be 0 or empty)
  if (inputValue === '0' || isNaN(inputValue) || inputValue === '') {
    evt.preventDefault()
    evt.target.value = '' // clear the input if it's 0
    numberOfPeople.value = '' // update with empty value
    peopleInputError.value = "Can't be 0"
    isValid.value = false
  } else {
    peopleInputError.value = ''
    numberOfPeople.value = parseFloat(inputValue) // update with valid value
    isValid.value = true
  }

  // input max 99999
  let parsedValue = parseFloat(inputValue) || 0
  if (parsedValue > 99999) {
    evt.preventDefault()
    evt.target.value = 99999
    numberOfPeople.value = 99999
  }
}
</script>
