<template>
  <p>Lines of code : {{ animatedCodes }}</p>
  <p>Lines per Day : {{ perLoop }}</p>
  <p v-if="checkEvent('bugEngine')">Technical Debt : {{ (bugEngine * 100).toFixed(1) }}%</p>
  <p v-if="checkEvent('bugEngine')">Code Efficiency : {{ (codeChecks * 100).toFixed(1) }}%</p>
  <p v-if="checkEvent('bugEngine')">Code removed for bugs : {{ totalBugs.toFixed(0) }}</p>
  <button v-on:click="updateBoth(1)">werk</button>
  <h3 v-if="checkEvent('forHire')">Hire</h3>
  <div v-for="hiree in forHire" :key="hiree.name">
    <div v-if="hiree.unlocked">
      <p>{{ hiree.name }} : {{ hiree.cost }} lines : writes {{ hiree.perLoop }}/sec</p>
      <button
        :disabled="!afford(hiree.cost)"
        v-on:click="hire(hiree.cost, hiree.perLoop, hiree.type)"
      >
        hire
      </button>
    </div>
  </div>
</template>
<script lang="js">
import gsap from 'gsap'
document.addEventListener('contextmenu', (event) => event.preventDefault())

function newHire(name, cost, perLoop, type) {
  return {
    name: name,
    cost: cost,
    perLoop: perLoop,
    type: type,
    unlocked: false
  }
}

function newEvent(name, eventAt) {
  return {
    name: name,
    eventAt: eventAt,
    unlocked: false
  }
}

export default {
  data() {
    return {
      runningCodes: 0,
      allTimeCodes: 0,
      tweenedCodes: 0,
      perLoop: 0,
      bugEngine: 0,
      codeChecks: 0,
      forHire: [],
      eventsList: [],
      saveTimer: '',
      codeTimer: ''
    }
  },
  computed: {
    animatedCodes: function () {
      return this.tweenedCodes.toFixed(0)
    },
    totalBugs: function () {
      return this.perLoop * (this.bugEngine - this.codeChecks).toFixed(2)
    }
  },
  created() {
    this.saveTimer = setInterval(this.saveLocal, 5000)
    this.codeTimer = setInterval(this.codeLoop, 1000)

    this.forHire.push(newHire('Intern', 100, 1, 1))
    this.forHire.push(newHire('Junior Dev', 1000, 20, 1))
    this.forHire.push(newHire('Senior Dev', 5000, 120, 1))
    this.forHire.push(newHire('QA Engineer', 50000, 0.01, 2))

    this.eventsList.push(newEvent('forHire', 100))
    this.eventsList.push(newEvent('bugEngine', 100000))

    if (localStorage.getItem('eventsList')) this.pullLocal()
    else this.saveLocal()
    this.codeLoop()
  },
  watch: {
    allTimeCodes: function () {
      if (this.allTimeCodes >= this.eventAt('forHire')) this.unlockEvent('forHire')
      if (this.allTimeCodes >= this.eventAt('bugEngine')) this.unlockEvent('bugEngine')
      if (this.checkEvent('bugEngine'))
        this.bugEngine = 0.1 + Math.floor((this.allTimeCodes - 100000) / 1000000) * 0.15

      for (let i = 0; i < this.forHire.length; i++) {
        if (this.allTimeCodes >= this.forHire[i].cost) this.forHire[i].unlocked = true
      }
    },
    runningCodes: function (newValue) {
      gsap.to(this.$data, { duration: 0.5, tweenedCodes: newValue })
    }
  },
  methods: {
    saveLocal() {
      this.pushLocal()
    },
    cancelSaveLocal() {
      clearInterval(this.saveTimer)
    },
    codeLoop() {
      this.updateBoth(this.perLoop)
    },
    cancelCodeLoop() {
      clearInterval(this.codeTimer)
    },
    updateBoth(amount) {
      this.allTimeCodes += amount
      this.runningCodes += amount
      this.allTimeCodes -= Math.ceil(amount * (this.bugEngine - this.codeChecks))
      this.runningCodes -= Math.ceil(amount * (this.bugEngine - this.codeChecks))
    },
    hire(cost, xfactor, type) {
      // 1 = coders  : add coder
      // 2 = testers : reduce bugs
      this.runningCodes -= cost
      if (type === 1) this.perLoop += xfactor
      if (type === 2) this.codeChecks += xfactor
    },
    afford(amount) {
      return this.runningCodes >= amount
    },
    checkEvent(name) {
      return this.eventsList.filter(function (item) {
        return item.name === name
      })[0].unlocked
    },
    unlockEvent(name) {
      this.eventsList.filter(function (item) {
        return item.name === name
      })[0].unlocked = true
    },
    eventAt(name) {
      return this.eventsList.filter(function (item) {
        return item.name === name
      })[0].eventAt
    },
    pullLocal() {
      this.eventsList = JSON.parse(localStorage.getItem('eventsList'))
      this.runningCodes = parseInt(localStorage.getItem('runningCodes'))
      this.allTimeCodes = parseInt(localStorage.getItem('allTimeCodes'))
      this.perLoop = parseInt(localStorage.getItem('perLoop'))
      this.codeChecks = parseFloat(localStorage.getItem('codeChecks'))
    },
    pushLocal() {
      localStorage.setItem('runningCodes', this.runningCodes)
      localStorage.setItem('allTimeCodes', this.allTimeCodes)
      localStorage.setItem('perLoop', this.perLoop)
      localStorage.setItem('codeChecks', this.codeChecks)
      localStorage.setItem('eventsList', JSON.stringify(this.eventsList))
    }
  },
  beforeUnmount() {
    clearInterval(this.timer)
  }
}
</script>
