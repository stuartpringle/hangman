<script>
  import { defineComponent } from 'vue'
  import { nextTick } from 'vue'
  import RandomWordList from '@/components/RandomWordList.vue'

  export default defineComponent({
    data() {
      return {
        game_word: '',
        wrongLetters: [],
        correctLetters: [],
        displayedCorrectString: '',
        displayedWrongString: '',
        alphabet: ["A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z"],
        hangingManStates: [1, 2, 3, 4, 5, 6, 7, 'dead', 'alive'],
        playAgainButtonText: 'play',
        difficultyLevel: 2,
        difficultyLevelText: 'easy',
        maxHangingManStateBeforeDeath: 7,
        gameState: 'playing',
        infoMessage: 'type or press a letter below to guess the word above',
        gameWordText: '',
        gameWordTextClass: '',
      }
    },
    methods: {
      getKeyboardInput(e) {
        let cmd = String.fromCharCode(e.keyCode).toLowerCase();
        //console.log(cmd)
        this.testLetter(cmd)// do stuff
      },
      async init() {
        if(this.game_word == '') {
          this.emitter.emit("getDifficulty")
          this.emitter.emit("getNewWord", this.difficultyLevel)
          this.emitter.emit("getWrongLetters")
          this.emitter.emit("getCorrectLetters")
          this.gameState = 'playing',
          this.infoMessage = 'type or press a letter below to guess the word above',
          this.gameWordText = '',
          this.gameWordTextClass = '',
          this.playAgainButtonText = 'reset'
          await nextTick()
        }
      },
      resetGame() {
        this.game_word = ''
        this.init()
      },
      getLettersToShow() {
        this.displayedCorrectString = ''
        for(let i = 0; i < this.game_word.length; i++) {
          this.displayedCorrectString += ' '
        }
        this.displayedCorrectString = this.updateDisplayedLetters(this.displayedCorrectString, this.correctLetters, this.game_word)

        this.displayedWrongString = ''
        for(let i = 0; i < this.alphabet.length; i++) {
          this.displayedWrongString += ' '
        }
        //send correct letters as the '|' string into the displayedWrongString variable
        this.displayedWrongString = this.updateDisplayedLetters(this.displayedWrongString, this.correctLetters, this.alphabet.join(''), true)

        //now fill the rest of it with the normal stuff
        this.displayedWrongString = this.updateDisplayedLetters(this.displayedWrongString, this.wrongLetters, this.alphabet.join(''))
        //console.log('ll' + this.displayedWrongString + 'll')
      },
      updateDisplayedLetters(displayedString, actualLetters, fullString, fillWithPlaceholder = false) {
        const placeholder = '|'
        let temp = displayedString.split('')
        for(let i = 0; i < actualLetters.length; i++) {
          //find all instances of current letter in displayedString
          let pos = fullString.toLowerCase().indexOf(actualLetters[i].toLowerCase());
          let increment = -1
          let counter = 0
          while(pos != -1 && counter < 100) {
            counter++
            let pos = fullString.toLowerCase().indexOf(actualLetters[i].toLowerCase(), increment + 1);
            increment = pos
            if(fillWithPlaceholder) {
              temp[pos] = placeholder
            } else {
              temp[pos] = actualLetters[i].toLowerCase()
            }
          }
        }
        //console.log(fullString)
        return temp.join('')
      },
      testLetter(event) {
        if(this.gameState !== 'playing') {
          return
        }
        //console.log(event.target.tagName)
        let new_letter = event
        //String.fromCharCode($event.charCode)
        //we are only interested in alphabetical characters
        if(this.alphabet.join('').toLowerCase().indexOf(new_letter.toLowerCase()) < 0) {
          return
        }

        if(this.game_word.indexOf(new_letter) > -1) {
          if(this.correctLetters.indexOf(new_letter.toLowerCase()) < 0) {
            //console.log('correct letter: ' + new_letter)
            this.emitter.emit("addCorrectLetter", new_letter.toLowerCase())
          }
        } else {
          if(this.wrongLetters.indexOf(new_letter.toLowerCase()) < 0) {
            //console.log('incorrect letter: ' + new_letter)
            this.emitter.emit("addWrongLetter", new_letter.toLowerCase())
          }
        }
      },
      youWin() {
        this.playAgainButtonText = 'play again'
        this.gameState = 'victory'
        this.infoMessage = 'you saved the stick-figure man'
        this.gameWordText = this.game_word
        this.gameWordTextClass = 'correct'
        setTimeout(() => {  this.emitter.emit("setHangingManState", 'alive') }, 1500);
      },
      youLose() {
        this.playAgainButtonText = 'play again'
        this.gameState = 'defeat'
        this.infoMessage = 'you lose. it\'s very sad.'
        this.gameWordText = this.game_word
        this.gameWordTextClass = 'wrong'
        setTimeout(() => {  this.emitter.emit("setHangingManState", 'dead') }, 1500);
      },
      updateDifficulty() {
        this.emitter.emit("setDifficulty", 1);
        this.resetGame();
      },
    },
    watch: {
      /*
      // whenever main_input changes, this function will run
      main_input(newInput, oldInput) {
        if(newInput !== null) {
          this.main_input = null
          this.testLetter(newInput)
        }
      }
      */
    },
    mounted(){
      this.init()
      this.emitter.emit("updateInfoPaneBlurbEvent", 'We\'ve got a word just for you.  Can you guess it, or will you hang?')
    },
    created() {
      window.addEventListener('keypress', this.getKeyboardInput);

      this.emitter.on("receiveDifficulty", (evt) => {
        this.difficultyLevel = evt

        this.difficultyLevelText = 'easy'
        if(this.difficultyLevel > 2 && this.difficultyLevel < 5) {
          this.difficultyLevelText = 'medium'
        } else if(this.difficultyLevel > 4 && this.difficultyLevel < 7) {
          this.difficultyLevelText = 'hard'
        }
      });

      this.emitter.on("receiveCorrectLetters", (evt) => {
        this.correctLetters = evt
        this.getLettersToShow()

        let tmp = this.game_word.split('')
        let allLettersInWordFound = true
        for(let i = 0; i < this.game_word.length; i++) {
          //console.log('letter', tmp[i])
          if(this.correctLetters.indexOf(tmp[i]) < 0) {
            allLettersInWordFound = false
          }

        }
        if(allLettersInWordFound) {
          this.youWin()
        }
      });

      this.emitter.on("receiveWrongLetters", (evt) => {
        this.wrongLetters = evt
        this.getLettersToShow()
        if(this.wrongLetters.length > this.hangingManStates[this.hangingManStates.indexOf(this.maxHangingManStateBeforeDeath) - 2]) {
          this.youLose();
        } else {
          this.emitter.emit("setHangingManState", this.hangingManStates[this.wrongLetters.length])
        }
      });

      this.emitter.on("receiveNewWord", (new_word) => {
        this.game_word = new_word
        console.log(this.game_word)
        this.getLettersToShow()
      });
    },
    destroyed() {
      window.removeEventListener('keypress', this.getKeyboardInput);
    },
  })
</script>

<template>
  <div class="play" ref="playContainer">
    <div class="left-col column">
      <div class="main-word-wrapper">
        <ul>
          <li v-for="(letter, index) in displayedCorrectString.split('')">
            <div class="letter">{{ letter }}</div>
            <div v-if="letter !== '-'" class="underline"></div>
          </li>
        </ul>
      </div>
    </div>
    <div class="right-col column">
      <div class="message-wrapper">
        <h1 class="actual-word" :class="gameWordTextClass">{{ gameWordText }}</h1>
        <h2 class="message">
          {{ infoMessage }}
        </h2>
      </div>
      <div class="wrong-letters-wrapper">
        <div v-for="(letter, index) in displayedWrongString.split('')">
          <span v-if="letter == ' '" class="inactive" @click="testLetter(alphabet[index].toLowerCase())">{{ alphabet[index].toLowerCase() }}</span>
          <span v-else-if="letter == '|'" class="correct">{{ alphabet[index].toLowerCase() }}</span>
          <span v-else class="active">{{ letter }}</span>
        </div>
      </div>
      <div class="input-wrapper">
        <button @click="resetGame" ref="play_again" id="play-again">{{ playAgainButtonText }}</button>
        <button @click="updateDifficulty" :class="`difficulty-level-button level-${difficultyLevelText} level-${difficultyLevelText}-${difficultyLevel}`">{{ difficultyLevelText }}</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.message {
  width: 100%;
  margin: auto;
  margin-top: 30px;
  text-align: center;
}
button {
  padding: 20px;
  border-radius: 10px;
  border: 2px solid hsla(160, 100%, 37%, 1);
  display: block;
  font-size: 30px;
  background: none;
  margin: auto;
  margin-top: 30px;
  width: 190px;
  text-align: center;
}

button:hover, button:active {
  background-color: #daefdd;
}

.actual-word {
  text-align: center;
  margin-top: 30px;
}

.actual-word.correct {
  color: hsla(160, 100%, 37%, 1);
}

.actual-word.wrong {
  color: red;
}

.difficulty-level-button {
  background-image: url('/difficulty-graph.png');
  background-position: center;
  background-size: 90% 80%;
  background-repeat: no-repeat;
  position: relative;
  text-shadow: 1px 1px 1px white;
}

.difficulty-level-button.level-easy-1 {
  background-image: url('/difficulty-graph-1.png');
}
.difficulty-level-button.level-easy-2 {
  background-image: url('/difficulty-graph-2.png');
}
.difficulty-level-button.level-medium-3 {
  background-image: url('/difficulty-graph-3.png');
}
.difficulty-level-button.level-medium-4 {
  background-image: url('/difficulty-graph-4.png');
}
.difficulty-level-button.level-hard-5 {
  background-image: url('/difficulty-graph-5.png');
}
.difficulty-level-button.level-hard-6 {
  background-image: url('/difficulty-graph-6.png');
}

.column {
  width: 100%;
}

.column .wrong-letters-wrapper,
.column .input-wrapper {
  margin-top: 30px;
  text-align: center;
  width: 100%;
}

.input-container {
  padding: 20px;
  border-radius: 10px;
  border: 2px solid hsla(160, 100%, 37%, 1);
  display: block;
  font-size: 30px;
  width: 190px;
  margin: auto;
  margin-top: 30px;
}

.input-container.inactive {
  display: none;
}

.input-container:focus {
  outline: none;
}

.main-word-wrapper {
  width: 100%;
}

.wrong-letters-wrapper {
  display: flex;
  flex-wrap: wrap;
  flex-direction: row;
  font-size: 30pt;
}

.wrong-letters-wrapper > div > span {
  padding: 10px;
}

.wrong-letters-wrapper .inactive {
  color: #ccc;
  cursor: pointer;
}

.wrong-letters-wrapper .correct {
  color: #000;
}

.wrong-letters-wrapper .active {
  color: red;
}

.main-word-wrapper ul {
  display: flex;
  flex-wrap: wrap;
  flex-direction: row;
  font-size: 30pt;
  justify-content: center;
}

.main-word-wrapper li {
  width: 50px;
  list-style-type: none;
}

.main-word-wrapper li .letter {
  height: 60px;
}

.underline {
  border-bottom: 2px solid black;
  height: 2px;
  width: 50%;
}
.play {
  display: flex;
  align-items: flex-start;
  flex-wrap: wrap;
  flex-direction: column;
}
@media (max-width: 1023px) {
  .wrong-letters-wrapper {
    justify-content: center;
  }
}
@media (min-width: 1024px) {
  .play {
    min-height: calc(100vh - 60px);
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .column {
    width: 50%;
    min-width: 490px;
  }

  .input-wrapper button {
    display: inline-block;
    margin: 10px;
  }
}
</style>