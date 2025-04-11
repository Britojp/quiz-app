<template>
  <v-app theme="dark">
    <v-container fluid class="py-10" style="background-color: #000; min-height: 100vh;">
      <v-row>
        <v-col cols="12" md="8">
          <v-card
            class="px-6 py-6"
            style="background-color: #4A148C; color: white; border-radius: 16px;"
            elevation="12"
          >
            <v-card-title class="text-h5 font-weight-bold d-flex justify-space-between align-center">
              <div>
                {{ decodeHTML(currentQuestion.question) }}
              </div>
              <div class="text-caption font-weight-light">⏱️ {{ timeLeft }}s</div>
            </v-card-title>

            <v-progress-linear
              :value="(timeLeft / totalTime) * 100"
              height="8"
              color="deep-purple-lighten-1"
              class="mb-4"
              striped
              :indeterminate="!answered && timeLeft === 0"
            />

            <v-card-subtitle class="text-subtitle-1">
              Dificuldade: {{ currentQuestion.difficulty || 'N/A' }}<br />
              Categoria: {{ currentQuestion.category || 'N/A' }}
            </v-card-subtitle>

            <v-card-text>
              <v-btn
                v-for="(option, index) in currentQuestion.options"
                :key="index"
                class="my-2"
                block
                :color="getOptionColor(option)"
                :variant="answered ? 'flat' : 'elevated'"
                :disabled="answered"
                @click="selectAnswer(option)"
              >
                {{ decodeHTML(option) }}
              </v-btn>

              <div v-if="answered" class="mt-4 text-subtitle-2 text-center">
                <span :class="isCorrect ? 'text-green' : 'text-red'">
                  {{ isCorrect ? '✅ Resposta correta!' : '❌ Resposta incorreta.' }}
                </span>
              </div>
            </v-card-text>

            <v-card-actions>
              <v-spacer />
              <v-btn
                v-if="answered || timeLeft === 0"
                @click="nextQuestion"
                color="deep-purple-darken-3"
                variant="flat"
                class="text-white"
              >
                Próxima
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-col>

        <v-col cols="12" md="4" class="pt-4 pt-md-0 d-flex flex-column align-center">
          <v-card class="mb-6 mx-auto" elevation="10" max-width="300" style="background-color: #311B92; color: white; border-radius: 16px;">
            <v-card-title class="text-h6 font-weight-bold text-center">Pontuação 1</v-card-title>
            <v-card-text class="d-flex justify-center align-center">
              <v-btn icon @click="pontos1--" color="red-darken-2" class="mx-2">
                <v-icon>mdi-minus</v-icon>
              </v-btn>
              <span class="text-h4 font-weight-bold mx-3">{{ pontos1 }}</span>
              <v-btn icon @click="pontos1++" color="green-darken-2" class="mx-2">
                <v-icon>mdi-plus</v-icon>
              </v-btn>
            </v-card-text>
          </v-card>

          <v-card class="mx-auto" elevation="10" max-width="300" style="background-color: #311B92; color: white; border-radius: 16px;">
            <v-card-title class="text-h6 font-weight-bold text-center">Pontuação 2</v-card-title>
            <v-card-text class="d-flex justify-center align-center">
              <v-btn icon @click="pontos2--" color="red-darken-2" class="mx-2">
                <v-icon>mdi-minus</v-icon>
              </v-btn>
              <span class="text-h4 font-weight-bold mx-3">{{ pontos2 }}</span>
              <v-btn icon @click="pontos2++" color="green-darken-2" class="mx-2">
                <v-icon>mdi-plus</v-icon>
              </v-btn>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>


<script>
import axios from 'axios'

export default {
  data() {
    return {
      questions: [],
      currentQuestionIndex: 0,
      selectedAnswer: null,
      answered: false,
      timeLeft: 10,
      totalTime: 10,
      timer: null,
    }
  },
  computed: {
    currentQuestion() {
      return this.questions[this.currentQuestionIndex] || {}
    },
    isCorrect() {
      return this.selectedAnswer === this.currentQuestion.answer
    },
  },
  methods: {
    decodeHTML(html) {
      const txt = document.createElement('textarea')
      txt.innerHTML = html
      return txt.value
    },
    selectAnswer(option) {
      if (!this.answered) {
        this.selectedAnswer = option
        this.answered = true
        clearInterval(this.timer)
      }
    },
    nextQuestion() {
      clearInterval(this.timer)
      if (this.currentQuestionIndex < this.questions.length - 1) {
        this.currentQuestionIndex++
        this.resetState()
      } else {
        alert('🎉 Você completou o trivia!')
        this.currentQuestionIndex = 0
        this.fetchQuestions()
      }
    },
    resetState() {
      this.selectedAnswer = null
      this.answered = false
      this.timeLeft = this.totalTime
      this.startTimer()
    },
    getOptionColor(option) {
      if (!this.answered) return 'deep-purple-darken-2'
      if (option === this.currentQuestion.answer) return 'green'
      if (option === this.selectedAnswer) return 'red'
      return 'deep-purple-darken-2'
    },
    async fetchQuestions() {
      try {
        const quantidadeDePerguntas = 100
        const token = 'eyJhbGciOiJIUzI1NiJ9.eyJpZCI6InVycGZxS1dDc2d0cTJrQWktUHdNOSIsImV4cCI6MTc0NDA4OTQ3MH0.B6zVwtpW3aAbwUxcP0_FG-gBuKLMkRpxFGA5RX6wLHk'

        const url = `https://tryvia.ptr.red/api.php?amount=${quantidadeDePerguntas}&token=${token}`

        const res = await axios.get(url)

        this.questions = res.data.results.map((q) => {
          const options = [...q.incorrect_answers, q.correct_answer]
          const shuffled = options.sort(() => 0.5 - Math.random())
          return {
            question: q.question,
            options: shuffled,
            answer: q.correct_answer,
            category: q.category,
            difficulty: q.difficulty,
          }
        })

        this.resetState()
      } catch (error) {
        console.error('❌ Erro ao buscar perguntas:', error)
      }
    },
    startTimer() {
      this.timer = setInterval(() => {
        if (this.timeLeft > 0) {
          this.timeLeft--
        } else {
          clearInterval(this.timer)
          this.answered = true
        }
      }, 1000)
    },
  },
  mounted() {
    this.fetchQuestions()
  },
  beforeDestroy() {
    clearInterval(this.timer)
  },
}
</script>
