<script>
  import RandomWordList from '@/components/RandomWordList.vue'
  import { defineComponent } from 'vue'

  export default defineComponent({
    data() {
      return {
        infoPaneBlurb: '',
        wrongLetters: [],
        correctLetters: [],
        mainWord: '',
        hangingManState: '1',
      }
    },
    components: {
      RandomWordList
    },
    methods: {
      setInfoPaneBlurb(new_menu_blurb_text) {
        this.infoPaneBlurb = new_menu_blurb_text
      },
      addWrongLetter(new_letter) {
        this.wrongLetters.push(new_letter)
        this.emitter.emit("receiveWrongLetters", this.wrongLetters);
      },
      addCorrectLetter(new_letter) {
        this.correctLetters.push(new_letter)
        this.emitter.emit("receiveCorrectLetters", this.correctLetters);
      },
      getNewWord() {
        const words = RandomWordList.data().wordList;
        const randomId = Math.floor(Math.random() * words.length);
        this.wrongLetters = []
        this.correctLetters = ['-']
        this.mainWord = words[randomId]
        this.emitter.emit("receiveNewWord", this.mainWord);
      },
    },
    created() {
      this.emitter.on("updateInfoPaneBlurbEvent", (evt) => {
        this.setInfoPaneBlurb(evt);
      });
      this.emitter.on("addWrongLetter", (evt) => {
        this.addWrongLetter(evt);
      });
      this.emitter.on("addCorrectLetter", (evt) => {
        this.addCorrectLetter(evt);
      });
      this.emitter.on("setHangingManState", (new_state) => {
        this.hangingManState = new_state
      });
      this.emitter.on("getWrongLetters", (evt) => {
        this.emitter.emit("receiveWrongLetters", this.wrongLetters);
      });
      this.emitter.on("getCorrectLetters", (evt) => {
        this.emitter.emit("receiveCorrectLetters", this.correctLetters);
      });
      this.emitter.on("getNewWord", (evt) => {
        return this.getNewWord();
      });
    }
  })
</script>

<template>
  <div class="hanging-man">
    <img :src="[ '/hanging-man-' + hangingManState + '.png' ]">
  </div>
  <div class="greetings">
    <br />
    <h1 class="green">Hangman</h1>
    <h3>{{ infoPaneBlurb }}</h3>
  </div>
</template>

<style scoped>
.hanging-man {
  position: relative;
  background-image: url('/scaffold.png');
  width: 379px;
  height: 607px;
}

.hanging-man img {
  position: absolute;
  left: 160px;
  top: 85px;
}

h1 {
  font-weight: 500;
  font-size: 2.6rem;
  top: -10px;
}

h3 {
  font-size: 1.2rem;
  height: 50px;
}

.greetings h1,
.greetings h3 {
  text-align: center;
}

@media (max-width: 1023px) {
  .greetings {
    margin: auto;
    text-align: center;
  }
  .hanging-man {
    margin: auto;
  }
}

@media (min-width: 1024px) {
  .greetings h1,
  .greetings h3 {
    text-align: left;
  }
}
</style>
