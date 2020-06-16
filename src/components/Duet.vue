<template>
  <div>
    <h1 class="text-yellow-600 text-2xl font-medium">Learn the words</h1>
    <div v-if="gameover" class="max-w-md h-auto m-auto rounded-md bg-yellow-500 p-3">
      <span>Thank you playing the game</span>
      <button class="block p-2 mx-auto my-4 rounded-md bg-gray-800 text-gray-100 tracking-wide text-lg focus:outline-none md:hover:font-semibold" @click="restartQuiz()">
        Restart
      </button>
    </div>
    <div v-else>
      <div v-if="!loading" class="select-none"> 
        <div v-if="playing" :class="processing ? 'pointer-events-none opacity-0' : 'pointer-events-auto opacity-100'" class="flex flex-col max-w-md h-auto m-auto rounded-md bg-yellow-500 p-3 items-stretch mt-2 transition delay-1000 duration-1000 ease-linear">
            <p class="font-lg tracking-wider mt-4 ">Which word is <i> {{ currentWord.type }} </i> to <strong> {{ currentWord.word }} </strong> </p>
            <button v-for="(option, index) in currentWord.options" :key="index" @click="evaluateResponse(index, $event)" type="button" 
              class="p-2 m-2 rounded-md bg-gray-800 text-gray-100 tracking-wide text-lg focus:outline-none md:hover:font-semibold transition duration-500 ease-linear">
              {{ option }}
            </button>
            <div>
              <span v-for="(word, index) in words" :key="index" 
                class="inline-block mx-2 rounded-full" 
                :class="[results[index] == -1 ? 'bg-gray-500' : results[index] == 0 ? 'bg-red-700' : 'bg-green-700', index == currentWordIndex ? 'w-4 h-4' : 'w-3 h-3']">
              </span>
            </div>
        </div>
        <div v-else class="flex items-center max-w-md h-auto m-auto rounded-md bg-yellow-500 p-6 mt-2">
          <div class="flex flex-col items-center justify-center w-20 h-20 rounded-full mx-4 border-teal-700 border-2 text-2xl tracking-wide text-gray-700">
              <span class="block border-b-2 border-teal-700">{{ score }}</span>
              <span class="block">{{ setSize }}</span>
          </div>
          <div class="text-lg">{{ citation[score] }}</div>
          <button class="inline-block p-2 mx-auto rounded-md bg-gray-800 text-gray-400 tracking-wide text-lg focus:outline-none hover:text-white" @click="initQuiz()">Next round</button>
        </div>
        <div v-if="!playing" class="max-w-md h-auto m-auto rounded-md bg-teal-500 p-6 mt-3 shadow text-left">
          <h2 class="text-teal-100 text-xl">Review the words</h2>
          <ul class="">
            <li v-for="(word, index5) in words" :key="index5" class="bg-teal-100 p-2 mt-2">
              <span class="inline-block w-3 h-3 rounded-full mr-2" :class="results[index5] == 0 ? 'bg-red-700' : 'bg-green-700'"></span>
              <button type="button" @click="showWordLookup(word.word, index5)" class="focus:outline-none">{{ word.word }}</button>
              <span class="text-xs inline-block ml-2">({{ word.type }})</span>
              <span class="block ml-2">
                <span v-for="(option, index2) in word.options" :key="index2" class="mx-2" :class="word.answer == index2 ? 'text-green-700' : 'text-red-700'">
                  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512" class="inline-block w-3 h-3 ml-2 fill-current" :class="word.answer == index2 ? 'text-green-700' : 'text-red-700'">
                    <path v-if="word.answer == index2"  d="M173.898 439.404l-166.4-166.4c-9.997-9.997-9.997-26.206 0-36.204l36.203-36.204c9.997-9.998 26.207-9.998 36.204 0L192 312.69 432.095 72.596c9.997-9.997 26.207-9.997 36.204 0l36.203 36.204c9.997 9.997 9.997 26.206 0 36.204l-294.4 294.401c-9.998 9.997-26.207 9.997-36.204-.001z"/>
                    <path v-else d="M242.72 256l100.07-100.07c12.28-12.28 12.28-32.19 0-44.48l-22.24-22.24c-12.28-12.28-32.19-12.28-44.48 0L176 189.28 75.93 89.21c-12.28-12.28-32.19-12.28-44.48 0L9.21 111.45c-12.28 12.28-12.28 32.19 0 44.48L109.28 256 9.21 356.07c-12.28 12.28-12.28 32.19 0 44.48l22.24 22.24c12.28 12.28 32.2 12.28 44.48 0L176 322.72l100.07 100.07c12.28 12.28 32.2 12.28 44.48 0l22.24-22.24c12.28-12.28 12.28-32.19 0-44.48L242.72 256z"/>
                  </svg>
                  {{option}}
                </span> 
              </span>
              <div v-if="!(dic[index5] === null)" class="bg-gray-200 ml-4 p-2">
                <i class="text-xs" v-if="dic[index5].length == 0">Sorry. Synonyms not found</i>
                <div class="" v-for="(synset, index4) in dic[index5]" :key="index4">
                  {{ synset.text + " " }}<i class="text-xs">({{ synset.pos}})</i>
                  <div>
                    <i class="text-xs">synonyms</i>
                    <span v-for="(syn, index3) in synset.tr" :key="index3" class="inline-block mx-2">{{ syn.text }}<i class="text-xs">({{ syn.pos }})</i></span>
                  </div>
                </div>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  const axios = require('axios').default;
  export default {
    data() {
      return {
        stock: [],
        history: [],
        processing: false,
        setSize: 5,
        words: [],
        currentWord: {},
        currentWordIndex: null,
        score: 0,
        loading: true,
        results:[],
        citation: ["Keep playing", "Good effort", "Great work", "Well done", "Almost perfect", "Perfect" ],
        dic: [],
        //gameover: false,
      }
    },

    created() {
      this.fetchWords();
    },

    computed: {
      playing: function() {
        return this.currentWordIndex < this.setSize;
      },

      gameover: function() {
        return this.stock.length - this.history.length <= 3;
      },
    },

    methods: {

      showWordLookup: function (word, index) {
        if (this.dic[index] === null) {  
          let key = "dict.1.1.20200608T052136Z.2faddc153c045974.7755c6efaa43abe14808473ed0060f98a405a5fe";
          let url = `https://dictionary.yandex.net/api/v1/dicservice.json/lookup?key=${key}&lang=en-en&text="${word}"`;
          axios
            .get(url)
            .then(
              response => {
                console.log(response.data.def);
                this.$set(this.dic, index, response.data.def);
              }
            )
        }
      },

      restartQuiz() {
        this.history = [];
        this.setSize = 5;
        this.initQuiz();
      },

      showNextQuestion() {
        this.currentWordIndex = this.currentWordIndex + 1;
        if (this.currentWordIndex < this.setSize) {
          this.currentWord = this.words[this.currentWordIndex];

        } else {
          console.log("End of quiz");
        }
      },

      initQuiz() {
        let vm = this;
        vm.results = [];
        vm.dic = [];
        vm.words = [];
        vm.score = 0;
        let i = 0;
        let availableQuestionsSize = vm.stock.length - vm.history.length;
        while ((i < vm.setSize) && (i < availableQuestionsSize) ) {
          let no = Math.floor(Math.random() * vm.stock.length);
          if (vm.history.indexOf(no) == -1) {
            vm.history.push(no);
            vm.words.push(vm.stock[no]);
            vm.results.push(-1);
            vm.dic.push(null);
            i++;
          }
        }
        vm.setSize = vm.words.length;
        vm.currentWordIndex = 0;
        vm.currentWord = vm.words[vm.currentWordIndex];
        // vm.gameover = (vm.setSize == 0);
      },

      fetchWords() {
        let vm = this;
        vm.loading = true;
        axios
          .get('words.json')
          .then(
            response => {
              vm.stock = response.data.words;
              vm.initQuiz();
              vm.loading = false;
            } 
          )
      },

      evaluateResponse(res, event) {
        let vm = this;
        // let index = this.questionNos.indexOf(this.questionNo);
        let el = event.target;
        let bg = el.style.backgroundColor;
        let responseColor;
        vm.processing = true;
        if (res == parseInt(vm.currentWord.answer)) {
          vm.results[vm.currentWordIndex] = 1;
          vm.score++;
          responseColor = "#2F855A";
        } else {
          this.results[vm.currentWordIndex] = 0;
          responseColor = "#C53030";
        }
        el.style.backgroundColor = responseColor;
        setTimeout(function() {
          el.style.backgroundColor = bg;
          vm.showNextQuestion();
          vm.processing = false;
        }, 1500);
      },

    },

  }
</script>


<style>

</style>

