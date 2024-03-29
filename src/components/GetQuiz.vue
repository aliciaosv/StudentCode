<script setup>
import ModalComp from '../components/ModalComp.vue'
import ProgressBar from '../components/ProgressBar.vue'
import DifficultyComp from '../components/DifficultyComp.vue'
import ResultComp from '../components/ResultComp.vue'
import CountDown from '../components/CountDown.vue'
import axios from 'axios'
import { ref, computed } from 'vue'

import { useRoute } from 'vue-router'
const route = useRoute()

const category = ref(route.params.category)
const level = ref(route.params.level)
const amount = ref(route.params.amount)

const quizData = ref(null)
const currentQuestionIndex = ref(0)
const progress = ref(0)

const selectedAnswer = ref('')
const optionsButtonDisabled = ref(false)
const isAnswerIncorrect = ref(null)
const isAnswerIncorrect2 = ref(false)
const revealAnswer = ref(false)

const buttonDisabled = ref(true)
const buttonText = ref('Continue')

const countDown = ref('')

const getQuiz = async (category, level, amount) => {
  try {
    const result = await axios.get(
      `https://quizapi.io/api/v1/questions?apiKey=Fn3mWDcTNToCVxnnLtiH2OXe9XSGTcpUFpl3SUUq&limit=${amount.value}&tags=${category.value}&difficulty=${level.value}`
    )
    quizData.value = result.data
    console.log(quizData.value.length) //Antal frågor
  } catch (error) {
    console.log(error)
  }
}
const currentQuestion = computed(() => {
  return quizData.value[currentQuestionIndex.value]
})
getQuiz(category, level, amount)

// console.log('route.params.category ' + category.value)
// console.log('route.params.level ' + level.value)
// console.log('route.params.amount ' + amount.value)

//A-B-C-D
function getLetter(word) {
  const letter = word.split('_')[1].toUpperCase()
  return letter
}
//Räkna poäng
const totalCorrectAnswers = ref(0)
//Välja svar
function selectAnswer(key) {
  const selected = getLetter(key)
  // console.log('Selected answer ' + selected)
  selectedAnswer.value = selected

  const correctAnswer = getCorrectAnswer()
  isAnswerIncorrect2.value = false
  if (selected === correctAnswer) {
    // console.log(totalCorrectAnswers.value);
    totalCorrectAnswers.value++
  } else {
    // console.log('incorrect answer')
    isAnswerIncorrect.value = key
    isAnswerIncorrect2.value = true
  }
}
//Kontrollera om ett svar är valt
function isSelected(key) {
  // console.log(key); //answer_a
  const isSelectedAnswer = getLetter(key)
  if (selectedAnswer.value === isSelectedAnswer) {
    buttonDisabled.value = false
  }
  return selectedAnswer.value === isSelectedAnswer //True eller false
}

//Hämta rätt svar med en bokstav
const getCorrectAnswer = () => {
  for (const key in currentQuestion.value.correct_answers) {
    if (currentQuestion.value.correct_answers[key] === 'true') {
      // console.log(key); //answer_a_correct
      const answer = getLetter(key)
      // console.log('Correct answer ' + answer) //A
      return answer
    }
  }
}

//Funktion som visar hur många sekunder man har till nästa fråga /Alicia
function startCountdown(seconds) {
  if (seconds > 0 && currentQuestionIndex.value === quizData.value.length - 1) {
    countDown.value = `Your results in... ${seconds}`
    setTimeout(() => {
      startCountdown(seconds - 1)
    }, 1000)
  }
  else if (seconds > 0) {
    countDown.value = `Next question in... ${seconds}`
    setTimeout(() => {
      startCountdown(seconds - 1)
    }, 1000)
  }
  else {
    countDown.value = ''
  }
}


//Nästa fråga + progress bar
function nextQuestion() {
  revealAnswer.value = true
  optionsButtonDisabled.value = true

  if (selectedAnswer.value !== getCorrectAnswer()) {//Visar rätt svar
    selectedAnswer.value = getCorrectAnswer()
  }
  // Kallar på funktionen ovan /Alicia
  startCountdown(3)

  setTimeout(() => {
    currentQuestionIndex.value++
    selectedAnswer.value = ''
    revealAnswer.value = false
    optionsButtonDisabled.value = false
    buttonDisabled.value = true
  }, 3000)

  if (currentQuestionIndex.value === quizData.value.length - 1) {//sista fråga
    buttonDisabled.value = true
    buttonText.value = 'DONE'
    saveDataToLocalStorage();

  }
  progress.value += 100 / quizData.value.length
}
// Spara data i local storage
const saveDataToLocalStorage = () => {
  console.log("sparar");
  const categoryKey = `quizDataObject_${currentQuestion.value.tags[0].name }_${ currentQuestion.value.difficulty }`;

  const dataResult = {
    category: currentQuestion.value.tags[0].name,
    questionAmount: quizData.value.length,
    correctAnswers: totalCorrectAnswers.value,
    difficulty: currentQuestion.value.difficulty
  };
  const results = JSON.stringify(dataResult);
  localStorage.setItem(categoryKey, results);
}
</script>

<template>
  <!-- Container börjar -->
  <div v-if="quizData && currentQuestion" class="container d-flex flex-column">
    <div class="flex">
      <DifficultyComp :difficulty="currentQuestion.difficulty" />
      <h4 class="mx-auto my-2" style="color: #204764; font-weight: bolder">
        {{ currentQuestion.tags[0].name }} <!-- Category -->
      </h4>
      <ModalComp /> <!-- Modal om att stänga quiz-->
    </div>

    <ProgressBar :progress-percent="progress" :total-question="quizData.length" />
    <div class="justify-content-between align-items-center">
        <CountDown />
    </div>

    <!-- FRÅGA -->
    <h4 class="mx-auto my-3 question" style="max-width: 80%">
      {{ currentQuestionIndex + 1 }}. {{ currentQuestion.question }}
    </h4>

    <!-- ALTERNATIV -->
    <div v-for="(answer, key) in currentQuestion.answers" :key="key" class="d-flex mt-3">
      <div v-if="answer" class="mx-auto alternatives" :class="{
    disableButton: optionsButtonDisabled,
    default: !isSelected(key),
    reveal: !revealAnswer && isSelected(key),
    correct: revealAnswer && isSelected(key) && getCorrectAnswer(),
    wrong:
      revealAnswer &&
      !isSelected(key) &&
      key === isAnswerIncorrect &&
      isAnswerIncorrect2
  }" @click="selectAnswer(key)" :active="isSelected(key)">
        <p v-if="revealAnswer && isSelected(key) && getCorrectAnswer()" class="icon">
          <img src="../assets/check.svg" alt="check-symbol" /> <!-- Rätt svar icon -->
        </p>
        <span v-else class="circle d-flex justify-content-center">{{ getLetter(key) }} <!-- A-B-C-D -->
        </span>
        <p>{{ answer }}</p>
      </div>
    </div>
    <span class="d-flex justify-content-center my-2" style="font-weight: bold">{{ countDown }}</span>
    <BButton class="mx-auto px-4 my-2 blueBtn" style="max-width: 75%" variant="success" @click="nextQuestion()"
      :disabled="optionsButtonDisabled || buttonDisabled">
      {{ buttonText }}
    </BButton>
  </div>
  <!-- EFTER QUIZZET -->
  <div v-else class="container d-flex flex-column flex-wrap" style="max-width: 45%; background-color: #f4f3f6">
    <ResultComp v-if="quizData" :correct="totalCorrectAnswers" :questions-amount="quizData.length" />
  </div>
</template>

<style scoped>
.container {
  margin: 2.5rem auto;
  border-radius: 10px;
  background-color: #f5eddf;
  border-radius: 20px;
  max-width: 65%;
  box-shadow: rgba(50, 50, 93, 0.25) 0px 50px 100px -20px,
    rgba(0, 0, 0, 0.3) 0px 30px 60px -30px,
    rgba(10, 37, 64, 0.35) 0px -2px 6px 0px inset !important;
}

/*Options and colors*/
.disableButton {
  pointer-events: none;
  cursor: default;
}

.alternatives {/*button*/
  width: 15rem;
  box-sizing: border-box;/*Jämlika knappar*/
  display: flex;
  align-items: center;
  cursor: pointer;
  overflow-wrap: anything;
  border-radius: 20px;
  box-shadow: rgba(50, 50, 93, 0.25) 0px 50px 100px -20px,
    rgba(0, 0, 0, 0.3) 0px 30px 60px -30px,
    rgba(10, 37, 64, 0.35) 0px -2px 6px 0px inset !important;
}

.alternatives>p {
  /*text*/
  padding: 5px;
  margin: 0.5rem 0;
  overflow-wrap: anywhere;
}

.default {
  background-color: #f4f3f6;
}

.default:hover {
  background-color: #e1dfe3;
}

.correct {
  background-color: #198754;
}

.wrong {
  border: 4px solid #ff544d;
}

.reveal {
  background-color: #f5e76c;
}

.circle {
  border: 1px solid black;
  min-width: 30px;
  cursor: pointer;
  border-radius: 100%;
  background-color: #f5eddf;
  margin: 0.4rem !important;
  vertical-align: middle;
  box-shadow: rgba(50, 50, 93, 0.25) 0px 50px 100px -20px,
    rgba(0, 0, 0, 0.3) 0px 30px 60px -30px,
    rgba(10, 37, 64, 0.35) 0px -2px 6px 0px inset !important;
}

.circle:hover {
  background-color: #f5e76c !important;
}

.flex {
  display: flex;
  justify-content: space-evenly;
  align-items: center;
  padding-top: 10px;
  padding-bottom: 10px;
}

.blueBtn {
  margin-bottom: 2rem !important;
}

@media only screen and (min-width: 530px) {
  .alternatives {
    width: 20rem;
  }

  .question {
    max-width: 20rem;
  }
}

@media only screen and (min-width: 850px) {
  .alternatives {
    max-width: 28rem;
  }

  .question {
    max-width: 25rem;
  }
}
</style>
