<script lang="ts">
  export default {
    data() {
      return {
        main_input: null,
        game_word: '',
        wrongLetters: [],
        correctLetters: [],
        displayedCorrectString: '',
        displayedWrongString: '',
        alphabet: ["A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z"],
        hangingManStates: [1, 2, 3, 4, 5, 6, 7, 'dead'],
        displayInput: false
      }
    },
    methods: {
      init() {
        if(this.game_word == '') {
          this.emitter.emit("getNewWord")
          this.emitter.emit("getWrongLetters")
          this.emitter.emit("getCorrectLetters")
          document.getElementById('play-again').innerHTML = 'reset'
          this.displayInput = true
          this.$refs.input_container.focus()
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
        this.displayedCorrectString = this.updateDispalyedLetters(this.displayedCorrectString, this.correctLetters, this.game_word)

        this.displayedWrongString = ''
        for(let i = 0; i < this.alphabet.length; i++) {
          this.displayedWrongString += ' '
        }
        this.displayedWrongString = this.updateDispalyedLetters(this.displayedWrongString, this.wrongLetters, this.alphabet.join(''))
        //console.log('ll' + this.displayedWrongString + 'll')
      },
      updateDispalyedLetters(displayedString, actualLetters, fullString) {
        let temp = displayedString.split('')
        for(let i = 0; i < actualLetters.length; i++) {
          let pos = fullString.toLowerCase().indexOf(actualLetters[i].toLowerCase());
          let increment = -1
          let counter = 0
          while(pos != -1 && counter < 100) {
            counter++
            let pos = fullString.toLowerCase().indexOf(actualLetters[i].toLowerCase(), increment + 1);
            increment = pos
            temp[pos] = actualLetters[i].toLowerCase()
          }
        }
        //console.log(fullString)
        return temp.join('')
      },
      testLetter(new_letter) {
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
      youLose() {
        document.getElementById('input-container').style.display = 'none'
        document.getElementById('play-again').innerHTML = 'play again'
        setTimeout(() => {  this.emitter.emit("setHangingManState", this.hangingManStates[this.hangingManStates.length - 1]) }, 2000);
      },
    },
    watch: {
      // whenever main_input changes, this function will run
      main_input(newInput, oldInput) {
        if(newInput !== null) {
          this.main_input = null
          this.testLetter(newInput)
        }
      }
    },
    mounted(){
      this.emitter.emit("updateInfoPaneBlurbEvent", 'We\'ve got a word just for you.  Can you guess it, or will you hang?')
    },
    created() {
      //this.init()
      this.emitter.on("receiveCorrectLetters", (evt) => {
        this.correctLetters = evt
        this.getLettersToShow()
      });
      this.emitter.on("receiveWrongLetters", (evt) => {
        this.wrongLetters = evt
        this.getLettersToShow()
        if(this.wrongLetters.length > this.hangingManStates.length - 3) {
          this.youLose();
        } else {
          this.emitter.emit("setHangingManState", this.hangingManStates[this.wrongLetters.length])
        }
      });
      this.emitter.on("receiveNewWord", (new_word) => {
        this.game_word = new_word
      });
    }
  }
</script>

<template>
  <div class="play">
    <div class="left-col column">
      <div class="main-word-wrapper">
        <ul>
          <li v-for="(letter, index) in displayedCorrectString.split('')">
            <div class="letter">{{ letter }}</div>
            <div class="underline"></div>
          </li>
        </ul>
      </div>
    </div>
    <div class="right-col column">
      <div class="wrong-letters-wrapper">
        <div v-for="(letter, index) in displayedWrongString.split('')">
          <span :class="[ letter == ' ' ? 'inactive' : 'active' ]">
            {{ letter == ' ' ? alphabet[index].toLowerCase() : letter }}
          </span>
        </div>
        <input ref="input_container" class="input-container" id="input-container" :class="[displayInput ? 'active' : 'inactive']" v-model="main_input" placeholder="guess here" />
        <button @click="resetGame" id="play-again">play</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
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

.column {
  width: 50%;
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

.inactive {
  color: #ccc;
}

.active {
  color: #000;
}

.main-word-wrapper ul {
  display: flex;
  flex-wrap: wrap;
  flex-direction: row;
  font-size: 30pt;
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

@media (min-width: 1024px) {
  .play {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>