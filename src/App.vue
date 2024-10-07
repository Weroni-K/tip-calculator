<template>
  <header>
    <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />
  </header>

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
                :class="{ active: customTipSelected }"
                :maxlength="maxLength"
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
                :class="{ active: customTipSelected }"
              />
            </div>
          </button>
        </div>
      </form>
      <div>
        <form>
          <label>Number of People</label>
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

const billAmount = ref()
const selectedTipPercentage = ref()
const numberOfPeople = ref(1)
const total = ref()
const tip = ref()
const bill = ref()

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
const customTipPercentage = ref('')
const customTipAmount = ref('')

const calculateAll = () => {
  let tipAmount = 0
  if (selectedTipPercentage.value === customTipAmount.value) {
    tipAmount = customTipAmount.value
  } else {
    tipAmount = (billAmount.value * selectedTipPercentage.value) / 100
  }

  tip.value = tipAmount / numberOfPeople.value || 0
  total.value = (billAmount.value + tipAmount) / numberOfPeople.value || 0
  bill.value = billAmount.value / numberOfPeople.value || 0
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
}
</script>
