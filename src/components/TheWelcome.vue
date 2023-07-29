<template>
  <!-- quiz game -->
  <div class="quiz-game">
    <div class="quiz-game__header">
      <h1 class="quiz-game__title">Quiz Game</h1>
      <div class="quiz-game__score">
        <span class="quiz-game__score-text">Score: {{ score }}</span>
      </div>
    </div>
    <div class="quiz-game__body">
      <div class="quiz-game__question" dir="rtl">
        <span class="quiz-game__question-text">{{ question }}</span>
      </div>
      <!-- time circle -->
      <div class="quiz-game__time">
        <div class="quiz-game__timer">
          <div class="quiz-game__timer-bar" :class="timerBarClass" :style="{ width: timeProgress }"></div>
        </div>
        <!-- <div class="quiz-game__timer-text">{{ timeLeft }}</div> -->
      </div>
      <div class="quiz-game__answers">
        <div class="quiz-game__answer" v-for="answer in answers" :key="answer">
          <!-- if correct -->
          <button class="quiz-game__answer-button" :class="{
            'quiz-game__answer-button--correct':
              selectedAnswer && answer === correctAnswer,
            'quiz-game__answer-button--wrong':
              selectedAnswer && answer !== correctAnswer,
          }" @click="checkAnswer(answer)">
            {{ answer }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'Home',
  data() {
    return {
      questions: [
        {
          id: 1,
          question: 'ما هي جنسية اللاعب السابق باتريك فييرا ',
          answers: ['فرنسي', 'افريقي'],
          correctAnswer: 'فرنسي',
        },
        {
          id: 2,
          question: 'Which country won the FIFA World Cup in 2018?',
          answers: ['Brazil', 'Germany', 'France', 'Argentina'],
          correctAnswer: 'France',
        },
        {
          id: 3,
          question: 'من هو أفضل لاعب في العالم؟',
          answers: ['ميسي', 'رونالدو', 'نيمار', 'محمد صلاح'],
          correctAnswer: 'رونالدو',
        },
      ],
      questionIndex: 0,
      question: '',
      answers: [],
      correctAnswer: '',
      score: 0,
      selectedAnswer: null,
      timeLeft: 10,
      timer: null,
      showCollision: false,
    };
  },

  methods: {
    getRandomQuestion() {
      const questionData = this.questions[this.questionIndex];
      if (questionData) {
        this.question = questionData.question;
        this.answers = this.shuffleArray(questionData.answers);
        this.correctAnswer = questionData.correctAnswer;
        console.log('Question:', questionData);
        this.questionIndex++;
        console.log('questionIndex:', this.questionIndex);
        this.startTimer();
      } else {
        // End of questions, display collision or any other ending logic
      }
    },

    checkAnswer(answer) {
      clearInterval(this.timer);
      this.selectedAnswer = answer;

      if (answer === null) {
        // Handle time's up here
        this.updateScore(false);
      } else {
        const isCorrect = answer === this.correctAnswer;

        if (isCorrect) {
          this.updateScore(true);
          this.playSound('correct.webm'); // Play the correct sound
        } else {
          this.updateScore(false);
          this.playSound('wrong.webm'); // Play the wrong sound
        }
      }

      setTimeout(() => {
        this.selectedAnswer = null;
        this.getRandomQuestion();
      }, 3000);
    },

    shuffleArray(array) {
      return array.sort(() => Math.random() - 0.5);
    },

    playSound(sound) {
      try {
        const baseUrl = new URL(import.meta.url);
        const fileUrl = new URL(`../assets/${sound}`, baseUrl);
        const audio = new Audio(fileUrl);
        audio.play();
      } catch (error) {
        console.error('Error playing sound:', error);
      }
    },

    updateScore(isCorrect) {
      if (isCorrect) {
        this.score++;
      } else {
        // Optionally, you can add logic for incorrect answers here
      }
    },

    startTimer() {
      this.timeLeft = 10; // Reset the time left to 10 seconds
      this.timer = setInterval(() => {
        if (this.timeLeft > 0) {
          this.timeLeft--;
        } else {
          clearInterval(this.timer); // Clear the timer interval when time runs out
          this.checkAnswer(null); // Call checkAnswer with a null value to indicate time's up
        }
      }, 1000);
    },
  },

  computed: {
    shouldRunCollision() {
      return this.questionIndex % 6 === 0 && this.questionIndex !== 0;
    },

    timeProgress() {
      return (this.timeLeft / 10) * 100 + '%';
    },

    timerBarClass() {
      return this.timeLeft <= 3 ? 'quiz-game__timer-bar--warning' : '';
    },
  },

  watch: {
    shouldRunCollision(newValue) {
      this.showCollision = newValue;
    },

    showCollision(newValue) {
      if (newValue) {
        setTimeout(() => {
          this.showCollision = false;
        }, 6000);
        // push to game two
      }
    },
  },

  mounted() {
    this.getRandomQuestion();
  },
};
</script>

<style scoped>
.collison {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.quiz-game {
  max-width: 600px;
  margin: 0 auto;
  font-family: sans-serif;
}

.quiz-game__header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  background-color: #6200ee;
  color: white;
}

.quiz-game__title {
  margin: 0;
  font-size: 24px;
}

.quiz-game__score {
  font-size: 18px;
}

.quiz-game__body {
  padding: 20px;
}

.quiz-game__question {
  margin-bottom: 20px;
  font-size: 18px;
  color: #e0e0e0;
}

.quiz-game__answers {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  grid-gap: 10px;
}

.quiz-game__answer-button {
  width: 100%;
  padding: 10px;
  border: none;
  border-radius: 5px;
  background-color: #6200ee;
  color: white;
  cursor: pointer;
}

.quiz-game__answer-button--correct {
  background-color: #03dac6;
  animation: correct 1s ease-in-out;
}

.quiz-game__answer-button--wrong {
  background-color: #cf6679;
  animation: wrong 1s ease-in-out;
}

.quiz-game__timer {
  position: relative;
  width: 100%;
  height: 10px;
  background-color: #e0e0e0;
  border-radius: 5px;
  margin-bottom: 20px;
}

.quiz-game__timer-bar {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: linear-gradient(to right, #03dac6, #6200ee);
  border-radius: 5px;
  animation: timer 10s linear;
}

.quiz-game__timer-bar--warning {
  background-image: linear-gradient(to right, #cf6679, #6200ee);
}

.quiz-game__timer-text {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 12px;
  color: #6200ee;
}




@keyframes correct {
  0% {
    transform: scale(1) rotate(0deg);
  }

  25% {
    transform: scale(1.2) rotate(15deg);
  }

  50% {
    transform: scale(1.2) rotate(-15deg);
  }

  75% {
    transform: scale(1.2) rotate(15deg);
  }

  100% {
    transform: scale(1) rotate(0deg);
  }
}

@keyframes wrong {
  0% {
    transform: scale(1);
  }

  25% {
    transform: scale(0.8) translateX(-10px);
  }

  50% {
    transform: scale(0.8) translateX(10px);
  }

  75% {
    transform: scale(0.8) translateX(-10px);
  }

  100% {
    transform: scale(1) translateX(0);
  }
}
</style>