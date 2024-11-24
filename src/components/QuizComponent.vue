<template>
  <div class="quiz-container">
    <!-- Výběr kategorie otázek -->
    <div v-if="!quizStarted" class="category-selection">
      <h2>Vyberte kategorii:</h2>
      <button 
        v-for="(questions, category) in questionSets" 
        :key="category"
        @click="startQuiz(category)"
        class="category-button"
      >
        {{ category }}
      </button>
    </div>

    <div v-else-if="!isFinished" class="question-section">
      <h2>Otázka {{ currentIndex + 1 }} z {{ shuffledQuestions.length }}</h2>
      <p class="question">{{ currentQuestion?.question }}</p>
      
      <div class="options">
        <button 
          v-for="option in shuffledOptions" 
          :key="option"
          :disabled="answered"
          @click="checkAnswer(option)"
          :class="{ 
            'selected': selectedAnswer === option,
            'correct': answered && option === currentQuestion?.correctAnswer,
            'incorrect': answered && selectedAnswer === option && option !== currentQuestion?.correctAnswer
          }"
        >
          {{ option }}
        </button>
      </div>

      <button 
        v-if="answered" 
        @click="nextQuestion" 
        class="next-button"
      >
        Další otázka
      </button>
    </div>

    <div v-else class="results">
      <h2>Výsledky</h2>
      <p>Úspěšnost: {{ Math.round((correctAnswers / shuffledQuestions.length) * 100) }}%</p>
      <p>Správně: {{ correctAnswers }} z {{ shuffledQuestions.length }}</p>
      
      <div v-if="wrongAnswers.length > 0" class="wrong-answers">
        <h3>Špatné odpovědi:</h3>
        <div v-for="(wrong, index) in wrongAnswers" :key="index">
          <p>Otázka: {{ wrong.question }}</p>
          <p>Vaše odpověď: {{ wrong.selected }}</p>
          <p>Správná odpověď: {{ wrong.correct }}</p>
        </div>
      </div>

      <div class="final-buttons">
        <button @click="resetQuiz" class="reset-button">
          Hrát znovu stejnou kategorii
        </button>
        <button @click="backToCategories" class="category-button">
          Zpět na výběr kategorie
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { vlastiveda_lib_kh_par_questions } from './vlastiveda_lib_kh_par.js'

// Stav
const questionSets = {
  'Vlastivěda - Liberecký, Královehradecký a Pardubický kraj': vlastiveda_lib_kh_par_questions,
  //'Matematika': mathQuestions,
  //'Historie': historyQuestions
}

const currentQuestions = ref([])
const shuffledQuestions = ref([])
const currentIndex = ref(0)
const selectedAnswer = ref(null)
const answered = ref(false)
const correctAnswers = ref(0)
const wrongAnswers = ref([])
const isFinished = ref(false)
const shuffledOptions = ref([])
const quizStarted = ref(false)

// Computed properties
const currentQuestion = computed(() => shuffledQuestions.value[currentIndex.value])

// Metody
const shuffleArray = (array) => {
  const newArray = [...array]
  for (let i = newArray.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [newArray[i], newArray[j]] = [newArray[j], newArray[i]]
  }
  return newArray
}

const shuffleQuestions = () => {
  shuffledQuestions.value = shuffleArray([...currentQuestions.value])
}

const shuffleCurrentOptions = () => {
  if (currentQuestion.value) {
    shuffledOptions.value = shuffleArray([...currentQuestion.value.options])
  }
}

const resetQuizState = () => {
  currentIndex.value = 0
  selectedAnswer.value = null
  answered.value = false
  correctAnswers.value = 0
  wrongAnswers.value = []
  isFinished.value = false
  shuffledQuestions.value = []
  shuffledOptions.value = []
}

const initializeQuiz = () => {
  shuffleQuestions()
  shuffleCurrentOptions()
}

const startQuiz = (category) => {
  currentQuestions.value = questionSets[category]
  quizStarted.value = true
  initializeQuiz()
}

const backToCategories = () => {
  quizStarted.value = false
  resetQuizState()
}

const checkAnswer = (selected) => {
  selectedAnswer.value = selected
  answered.value = true
  
  if (selected === currentQuestion.value.correctAnswer) {
    correctAnswers.value++
  } else {
    wrongAnswers.value.push({
      question: currentQuestion.value.question,
      selected: selected,
      correct: currentQuestion.value.correctAnswer
    })
  }
}

const nextQuestion = () => {
  selectedAnswer.value = null
  answered.value = false
  
  if (currentIndex.value < shuffledQuestions.value.length - 1) {
    currentIndex.value++
    shuffleCurrentOptions()
  } else {
    isFinished.value = true
  }
}

const resetQuiz = () => {
  resetQuizState()
  initializeQuiz()
}

// Lifecycle hooks
onMounted(() => {
  // Případná inicializace při mounted
})
</script>

<style scoped>
.quiz-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

.category-selection {
  text-align: center;
  margin-bottom: 20px;
}

.category-button {
  margin: 10px;
  padding: 15px 30px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.category-button:hover {
  background-color: #45a049;
}

.options {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin: 20px 0;
}

button {
  padding: 10px 20px;
  border: 1px solid #ccc;
  border-radius: 4px;
  cursor: pointer;
}

button:disabled {
  cursor: not-allowed;
}

.selected {
  background-color: #e0e0e0;
}

.correct {
  background-color: #4caf50;
  color: white;
}

.incorrect {
  background-color: #f44336;
  color: white;
}

.next-button, .reset-button {
  background-color: #2196f3;
  color: white;
  border: none;
  margin-top: 20px;
}

.wrong-answers {
  margin-top: 20px;
  padding: 20px;
  background-color: #fff3f3;
  border-radius: 4px;
}

.final-buttons {
  display: flex;
  gap: 10px;
  justify-content: center;
  margin-top: 20px;
}

.question {
  font-size: 1.2em;
  margin: 20px 0;
  font-weight: 500;
}
</style>