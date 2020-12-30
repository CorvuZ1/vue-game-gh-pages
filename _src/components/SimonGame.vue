<template>
<div>
    <div @click="soundPlay" class="game" ref="game">
        <input type="button" class="panel-green"
          :class="{'panel-green-click': this.panelClick.currentPanel[this.panelClick.currentPanel.length-1] == 0}">

        <input type="button" class="panel-red"
          :class="{'panel-red-click': this.panelClick.currentPanel[this.panelClick.currentPanel.length-1] == 1}">

        <input type="button" class="panel-yellow"
          :class="{'panel-yellow-click': this.panelClick.currentPanel[this.panelClick.currentPanel.length-1] == 2}">

        <input type="button" class="panel-blue"
          :class="{'panel-blue-click': this.panelClick.currentPanel[this.panelClick.currentPanel.length-1] == 3}">

        <div class="bar">
          <span class="bar_score" ref="score">Score: 0</span>
          <span class="bar_best-score" ref="bestScore"></span>
          <input @click="gameStart" type="button" class="bar_start" :class="{'bar_start-click': this.start == true}" value="Start" ref="startBtn">
          <div class="bar_diff">
            <input class="bar_diff-dot dot1" @click="diff = 1500" ref="dot1" type="radio" name="dot">
            <input class="bar_diff-dot dot2" @click="diff = 1000" ref="dot2" type="radio" name="dot">
            <input class="bar_diff-dot dot3" @click="diff = 400" ref="dot3" type="radio" name="dot">
          </div>
        </div>
    </div>
</div>
</template>

<script>
import {Howl} from "howler"

export default {
  data() {
    return {
      panelClick: {
        currentPanel: [],
        bot: [],
        player: []
      },
      clickable: false,
      panelsList: [0,1,2,3],
      start: false,
      score: 1,
      click:0,
      diff: null,
      sounds: [
        new Howl({src: ["https://s3.amazonaws.com/freecodecamp/simonSound1.mp3"]}),
        new Howl({src: ["https://s3.amazonaws.com/freecodecamp/simonSound2.mp3"]}),
        new Howl({src: ["https://s3.amazonaws.com/freecodecamp/simonSound3.mp3"]}),
        new Howl({src: ["https://s3.amazonaws.com/freecodecamp/simonSound4.mp3"]})
      ]
    }
  },
  mounted() {
    if (!localStorage.getItem("bestScore")) { // Проверка на отсутствие локальных данных
      localStorage.setItem("bestScore", 0)
    } 
    this.$refs.bestScore.innerText = `Best score: ${localStorage.getItem("bestScore")}`
  },
  methods: {
    soundPlay(event) {
      if (!this.clickable) {
        return 
      }
      switch(event.target.className) {
        case "panel-green": 
          this.playerClick(0)
          break;

        case "panel-red":
          this.playerClick(1) 
          break;

        case "panel-yellow":
          this.playerClick(2)
          break;

        case "panel-blue":
          this.playerClick(3)
          break;
      }
    },

    deepСomparison(arr1,arr2) {
        if (arr1.join("") == arr2.join("")) {
          return true
        }
    },

    playerClick(panel) {
      this.panelClick.currentPanel.push(panel)
      this.panelClick.player.push(panel)
      this.sounds[panel].play()
      setTimeout(() => {
        this.panelClick.currentPanel.pop();
      }, 200);
      return this.WinOrLose()
    },

    WinOrLose() { 
      if(this.panelClick.player.length == this.panelClick.bot.length) {
        if (this.deepСomparison(this.panelClick.player, this.panelClick.bot)) {
          this.panelClick.bot = []
          this.panelClick.player = []
          this.score++
          setTimeout(() => {
            return this.gameStart()
          },1200);
        } else {
        if (this.score >= localStorage.getItem("bestScore")) {
          localStorage.setItem("bestScore", this.score)
        }
        alert(`Lose! Score ${this.score} \nBot clicked: ${this.endExplanation(this.panelClick.bot)} \nYou clicked: ${this.endExplanation(this.panelClick.player)}`)
        window.location.reload()
      }
    }
  },

  endExplanation(results) {
      let arr = []
      for(var i = 0; i < results.length; i++) {
        arr.push(results[i]+1)
      }
      return arr.join(", ")
    },

    botClick(panel) {
      return new Promise((resolve, reject) => {
        this.panelClick.currentPanel.push(panel) // проверка на изменение класса
        this.panelClick.bot.push(panel) // все кликнутые ботом панели
        this.sounds[panel].play()
        setTimeout(() => {
          this.panelClick.currentPanel.pop();
          resolve()
        },this.diff);
      })
    },

    async gameStart() {
      if (this.diff === null) {
        alert("Set difficulty")
        return
      }
      // Блокировка оставшихся кнопок после начала игры 
      switch(this.diff) {
        case 1500: 
          this.$refs.dot2.disabled = true; 
          this.$refs.dot3.disabled = true; 
          break;
        case 1000: 
          this.$refs.dot1.disabled = true; 
          this.$refs.dot3.disabled = true; 
          break;
        case 400: 
          this.$refs.dot1.disabled = true; 
          this.$refs.dot2.disabled = true; 
          break;
      }
      this.start = true
      this.clickable = false
      this.$refs.score.innerText = `Score: ${this.score}`
      this.$refs.startBtn.disabled = true
      for(let i = 0; i < this.score; i++) {
        let random = this.randomNum()
        await this.botClick(random)
      }
      this.clickable = true
    },

    randomNum() {
      return this.panelsList[Math.floor(Math.random() * 4)]
    }
  }
}
</script>

<style lang="sass">
    
  .game
    display: grid
    grid-template-columns: repeat(2,1fr)
    min-width: 500px
    min-height: 500px
    border-radius: 100%
    box-shadow: 0px 0px 35px black
    position: relative
    overflow: hidden

    .panel-green, .panel-red, .panel-yellow, .panel-blue
      height: 250px
      display: block
      border: none
      cursor: pointer
      position: relative
        
    .panel-green
      background-color: rgba(0, 128, 0, 1)
      right: 7px
      bottom: 7px
    
    .panel-green-click
      background-color: rgba(0, 128, 0, 0.5)
      border-right: 1px solid black
      border-bottom: 1px solid black

    .panel-red
      background-color: rgba(250, 10, 0, 1)
      left: 7px
      bottom: 7px
    
    .panel-red-click
      background-color: rgba(250, 0, 0, 0.5)
      border-left: 1px solid black
      border-bottom: 1px solid black
        
    .panel-yellow
      background-color: rgba(255, 170, 0, 1)
      top: 7px
      right: 7px
      
    .panel-yellow-click
      background-color: rgba(255, 215, 0, 0.5)
      border-top: 1px solid black
      border-right: 1px solid black

    .panel-blue
      background-color: rgba(0, 0, 255, 1)
      top: 7px
      left: 7px
      
    .panel-blue-click
      background-color: rgba(0, 0, 255, 0.5)
      border-top: 1px solid black
      border-left: 1px solid black

    .bar
      width: 220px
      height: 220px
      position: absolute
      background-color: white
      border-radius: 100%
      background-image: radial-gradient(white, gray 90%)
      display: flex
      justify-content: center
      align-items: center
      flex-wrap: wrap
      flex-direction: column
      box-shadow: 0px 0px 15px 0px black
      position: absolute
      justify-self: center
      align-self: center

    .bar_score, .bar_best-score
      font-size: 18px
      position: relative
      bottom: 40px
      font-family: Arial 

    .bar_best-score
      margin-top: 6px

    .bar_start
      font-weight: bold
      font-size: 23px
      font-family: Arial
      cursor: pointer
      border: 1px solid #3c3c3c
      border-radius: 10px
      padding: 5px 15px
      background-color: transparent
      transition-duration: 0.5s
      position: relative
      bottom: 13px
      &:disabled
        color: white
    
    .bar_diff
      display: flex
      width: 150px
      justify-content: space-around
      position: relative
      top: 7px

    .bar_diff-dot
      position: relative
      cursor: pointer
      width: 13px
      height: 13px
      border-radius: 50px
      &:disabled
        cursor: default
    
    .dot1, .dot2, .dot3        
      -webkit-appearance: none
      -moz-appearance: none
      -o-appearance: none
      -ms-appearance: none
      -khtml-appearance: none
      appearance: none
      box-sizing: border-box
      &:checked
        border: 2px solid white
      &:disabled
        display: none

    .dot1
      background-color: green

    .dot2
      background-color: orange

    .dot3
      background-color: red

    .bar_diff-dot::after
      content: ""
      width: 10px
      font-size: 15px
      position: absolute
      top: 18px

    .dot1::after
      content: "Easy"
      right: 15px

    .dot2::after
      content: "Medium"
      right: 24px

    .dot3::after
      content: "Hard"
      right: 13px

    .bar_start-click
      color: white
      background-color: rgba(0,0,0, 0.7)
</style>

