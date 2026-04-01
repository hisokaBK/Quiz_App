<script setup lang="ts">
import axios from "axios";
import { computed, onBeforeUnmount, onMounted, ref, watch } from "vue";
import { useRoute, useRouter } from "vue-router";
import type { Question } from "../types/quiz";

const router = useRouter();
const route = useRoute();

interface TriviaApiQuestion {
  question: string;
  correct_answer: string;
  incorrect_answers: string[];
}

interface TriviaApiResponse {
  response_code: number;
  results: TriviaApiQuestion[];
}

const selected = ref<string>("");
const questions = ref<Question[]>([]);
const currentIndex = ref(0);
const isLoading = ref(true);
const loadError = ref("");
const answers = ref<Record<number, string>>({});
const timePerQuestion = 30;
const timeLeft = ref(timePerQuestion);
const timerId = ref<number | null>(null);

const subject = computed(() => String(route.query.subject ?? "Geometry"));
const userName = computed(() => {
  const routeUser = String(route.query.user ?? "").trim();
  if (routeUser) {
    return routeUser;
  }
  return localStorage.getItem("quiz_user_name") ?? "You";
});
const currentQuestion = computed(() => questions.value[currentIndex.value]);
const progressLabel = computed(() => {
  if (!questions.value.length) {
    return "Question 0/0";
  }
  return `Question ${currentIndex.value + 1}/${questions.value.length}`;
});
const formattedTime = computed(() => {
  const minutes = Math.floor(timeLeft.value / 60)
    .toString()
    .padStart(2, "0");
  const seconds = (timeLeft.value % 60).toString().padStart(2, "0");
  return `${minutes}:${seconds}`;
});
const timerPercent = computed(() => (timeLeft.value / timePerQuestion) * 100);

function stopTimer(): void {
  if (timerId.value !== null) {
    window.clearInterval(timerId.value);
    timerId.value = null;
  }
}

function startTimer(): void {
  stopTimer();
  timeLeft.value = timePerQuestion;

  timerId.value = window.setInterval(() => {
    if (timeLeft.value > 0) {
      timeLeft.value -= 1;
      return;
    }

    stopTimer();
    goNext();
  }, 1000);
}

function decodeHtml(text: string): string {
  const textarea = document.createElement("textarea");
  textarea.innerHTML = text;
  return textarea.value;
}

function shuffleOptions(options: string[]): string[] {
  const shuffled = [...options];
  for (let i = shuffled.length - 1; i > 0; i -= 1) {
    const j = Math.floor(Math.random() * (i + 1));
    [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
  }
  return shuffled;
}

async function fetchQuestions(): Promise<void> {
  isLoading.value = true;
  loadError.value = "";

  try {
    const response = await axios.get<TriviaApiResponse>(
      "https://opentdb.com/api.php?amount=5&category=21&difficulty=medium"
    );

    if (response.data.response_code !== 0) {
      throw new Error("API returned a non-zero response code.");
    }

    questions.value = response.data.results.map((item, index) => {
      const options = shuffleOptions([
        ...item.incorrect_answers.map(decodeHtml),
        decodeHtml(item.correct_answer),
      ]);

      return {
        id: index + 1,
        question: decodeHtml(item.question),
        options,
        correctAnswer: decodeHtml(item.correct_answer),
      };
    });

    currentIndex.value = 0;
    selected.value = "";
    answers.value = {};
  } catch {
    loadError.value = "Failed to load questions. Please try again.";
  } finally {
    isLoading.value = false;
  }
}

function skipQuestion(): void {
  const questionId = currentQuestion.value?.id;
  if (questionId !== undefined) {
    delete answers.value[questionId];
  }

  if (currentIndex.value < questions.value.length - 1) {
    currentIndex.value += 1;
    selected.value = "";
    return;
  }

  goNext();
}

function goNext(): void {
  if (!questions.value.length) {
    return;
  }

  const questionId = currentQuestion.value?.id;
  if (questionId !== undefined) {
    if (selected.value) {
      answers.value[questionId] = selected.value;
    } else {
      delete answers.value[questionId];
    }
  }

  if (currentIndex.value < questions.value.length - 1) {
    currentIndex.value += 1;
    selected.value = "";
    return;
  }

  stopTimer();
  const finalScore = questions.value.reduce((score, question) => {
    const answer = answers.value[question.id];
    if (answer === question.correctAnswer) {
      return score + 1;
    }
    return score;
  }, 0);
  const totalQuestions = questions.value.length;
  const finalPercent = totalQuestions ? Math.round((finalScore / totalQuestions) * 100) : 0;

  localStorage.setItem(
    "quiz_last_result",
    JSON.stringify({
      user: userName.value,
      subject: subject.value,
      score: finalScore,
      total: totalQuestions,
      percent: finalPercent,
    })
  );

  router.push({
    name: "Result",
    query: {
      subject: subject.value,
      score: String(finalScore),
      total: String(totalQuestions),
      user: userName.value,
    },
  });
}

onMounted(() => {
  fetchQuestions();
});

watch([questions, currentIndex], () => {
  if (isLoading.value || loadError.value || !currentQuestion.value) {
    stopTimer();
    return;
  }

  startTimer();
});

onBeforeUnmount(() => {
  stopTimer();
});
</script>

<template>
  <main class="page">
    <header class="head">
      <button type="button" @click="router.push({ name: 'Home' })">Back</button>
      <div>
        <p>{{ subject }}</p>
        <h1>Quiz Session</h1>
      </div>
      <div class="head-meta">
        <span>{{ progressLabel }}</span>
        <strong class="timer">{{ formattedTime }}</strong>
      </div>
    </header>

    <section class="layout">
      <article class="panel question-panel">
        <p v-if="isLoading">Loading questions...</p>
        <p v-else-if="loadError" class="error-message">{{ loadError }}</p>
        <template v-else-if="currentQuestion">
          <div class="timer-track" aria-hidden="true">
            <span :style="{ width: `${timerPercent}%` }"></span>
          </div>

          <h2>{{ currentQuestion.question }}</h2>

          <div class="options">
            <button
              v-for="option in currentQuestion.options"
              :key="option"
              class="option"
              :class="{ chosen: selected === option }"
              type="button"
              @click="selected = option"
            >
              {{ option }}
            </button>
          </div>

          <footer class="actions">
            <button class="btn ghost" type="button" @click="skipQuestion">Skip</button>
            <button class="btn solid" type="button" @click="goNext">Save &amp; Next</button>
          </footer>
        </template>
      </article>

      <aside class="panel side-panel">
        <h3>Progress</h3>
        <div class="progress-grid">
          <span
            v-for="n in questions.length || 10"
            :key="n"
            :class="{ done: n <= currentIndex + 1, active: n === currentIndex + 1 }"
          >
            {{ n }}
          </span>
        </div>

        <h3>Status</h3>
        <p v-if="isLoading">Fetching questions from OpenTDB...</p>
        <p v-else-if="loadError" class="error-message">{{ loadError }}</p>
        <p v-else-if="selected" class="ok">Answer saved. Continue to the next question.</p>
        <p v-else>Select one option to answer this question.</p>
      </aside>
    </section>
  </main>
</template>

<style scoped>
.page {
  width: min(1120px, 100% - 2rem);
  margin: 2rem auto;
  display: grid;
  gap: 1rem;
}

.head,
.panel {
  border-radius: 1.25rem;
  border: 1px solid rgba(8, 88, 95, 0.16);
  box-shadow: 0 14px 34px rgba(12, 53, 59, 0.12);
}

.head {
  background: #f3f4ef;
  padding: 1rem 1.1rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.8rem;
}

.head-meta {
  display: grid;
  justify-items: end;
  gap: 0.25rem;
}

.timer {
  background: #d8ec77;
  border: 1px solid #c8de61;
  border-radius: 999px;
  padding: 0.18rem 0.58rem;
  font-size: 0.8rem;
  color: #0b4f57;
}

.head button {
  border-radius: 999px;
  border: 1px solid var(--teal-900);
  background: transparent;
  color: var(--teal-900);
  padding: 0.45rem 0.8rem;
  font: inherit;
}

.head p {
  margin: 0;
  text-align: center;
  color: rgba(9, 63, 71, 0.7);
  font-size: 0.78rem;
}

h1 {
  margin: 0.15rem 0 0;
  font-size: 1.4rem;
  text-align: center;
  font-family: "Sora", "Segoe UI", sans-serif;
}

.layout {
  display: grid;
  grid-template-columns: 1.45fr 1fr;
  gap: 1rem;
}

.panel {
  background: #f4f4ef;
  padding: 1rem;
}

h2 {
  margin: 0;
  font-family: "Sora", "Segoe UI", sans-serif;
  font-size: 1.4rem;
}

.timer-track {
  height: 0.5rem;
  width: 100%;
  border-radius: 999px;
  background: #dde0d5;
  overflow: hidden;
  margin-bottom: 0.85rem;
}

.timer-track span {
  display: block;
  height: 100%;
  background: linear-gradient(90deg, #0e7773, #2a9b8e);
  transition: width 1s linear;
}

.options {
  margin-top: 1rem;
  display: grid;
  gap: 0.7rem;
}

.option {
  width: 100%;
  border: 1px solid #cfd2c5;
  background: #fff;
  border-radius: 999px;
  padding: 0.72rem;
  font: inherit;
  color: var(--ink);
}

.option.chosen {
  background: #d8ec77;
  border-color: #d8ec77;
}

.actions {
  margin-top: 1rem;
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 0.6rem;
}

.btn {
  border-radius: 999px;
  padding: 0.68rem;
  font: inherit;
}

.ghost {
  border: 1px solid var(--teal-800);
  color: var(--teal-800);
  background: #fff;
}

.solid {
  border: 0;
  background: var(--teal-900);
  color: #fff;
}

.side-panel h3 {
  margin: 0;
  font-family: "Sora", "Segoe UI", sans-serif;
}

.progress-grid {
  margin: 0.8rem 0 1rem;
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 0.42rem;
}

.progress-grid span {
  text-align: center;
  padding: 0.45rem 0;
  border-radius: 0.6rem;
  border: 1px solid #cfd2c5;
  background: #fff;
  font-size: 0.85rem;
}

.progress-grid span.done {
  background: #d8ec77;
  border-color: #d8ec77;
}

.progress-grid span.active {
  background: var(--teal-900);
  border-color: var(--teal-900);
  color: #fff;
}

ul {
  margin: 0.7rem 0 0;
  padding-left: 1rem;
  color: rgba(9, 63, 71, 0.8);
}

li + li {
  margin-top: 0.35rem;
}

.ok {
  margin-top: 0.65rem;
  color: #0e7773;
  font-weight: 600;
}

.error-message {
  color: #b42318;
  font-weight: 600;
}

@media (max-width: 920px) {
  .layout {
    grid-template-columns: 1fr;
  }
}
</style>
