<script setup lang="ts">
import { computed, ref } from "vue";
import { useRouter } from "vue-router";

const router = useRouter();
const userName = ref("");

const canStart = computed(() => userName.value.trim().length > 0);

function goToQuiz(): void {
  const name = userName.value.trim();
  if (!name) {
    return;
  }

  localStorage.setItem("quiz_user_name", name);
  router.push({ name: "Quiz", query: { user: name } });
}
</script>

<template>
  <main class="home-page">
    <section class="intro-card">
      <p class="eyebrow">Welcome to Quizzax</p>
      <h1>Sharpen your skills with short and fun quiz sessions.</h1>
      <p class="description">
        Quizzax helps you practice topic-based questions, track progress, and improve your
        score over time.
      </p>

      <label class="field" for="username">Your name</label>
      <input
        id="username"
        v-model="userName"
        type="text"
        class="input"
        placeholder="Enter your name"
        @keyup.enter="goToQuiz"
      />

      <button class="start-btn" type="button" :disabled="!canStart" @click="goToQuiz">
        Start Quiz
      </button>
    </section>
  </main>
</template>

<style scoped>
.home-page {
  min-height: 100svh;
  display: grid;
  place-items: center;
  padding: 2rem 1rem;
}

.intro-card {
  width: min(720px, 100%);
  border: 1px solid rgba(8, 88, 95, 0.18);
  border-radius: 1.25rem;
  background: linear-gradient(150deg, #d8ec77 0%, #d0e773 100%);
  box-shadow: 0 18px 44px rgba(12, 53, 59, 0.18);
  padding: 2rem;
}

.eyebrow {
  margin: 0;
  font-size: 0.82rem;
  text-transform: uppercase;
  letter-spacing: 0.09em;
  color: rgba(9, 63, 71, 0.75);
}

h1 {
  margin: 0.55rem 0;
  font-family: "Sora", "Segoe UI", sans-serif;
  color: #0b4f57;
  font-size: clamp(1.5rem, 3vw, 2.25rem);
  line-height: 1.2;
  max-width: 24ch;
}

.description {
  margin: 0 0 1.25rem;
  color: rgba(9, 63, 71, 0.85);
  max-width: 55ch;
}

.field {
  display: block;
  margin-bottom: 0.35rem;
  font-weight: 600;
  color: #0b4f57;
}

.input {
  width: 100%;
  border: 1px solid rgba(8, 88, 95, 0.24);
  border-radius: 0.75rem;
  padding: 0.75rem 0.9rem;
  font: inherit;
  color: #083f47;
  background: #f7fbde;
  outline: none;
}

.input:focus {
  border-color: #0a6972;
  box-shadow: 0 0 0 3px rgba(10, 105, 114, 0.14);
}

.start-btn {
  margin-top: 0.9rem;
  border: 0;
  border-radius: 999px;
  padding: 0.72rem 1.1rem;
  font: inherit;
  font-weight: 600;
  color: #d8ec77;
  background: #06585e;
  cursor: pointer;
}

.start-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}


.icon-graph::after {
  width: 1.2rem;
  height: 0.9rem;
  border-left: 0;
  border-bottom: 0;
  border-radius: 999px 999px 0 0;
  top: 0.65rem;
  left: 0.45rem;
}

.side-card h3 {
  margin: 0;
  font-family: "Sora", "Segoe UI", sans-serif;
}

.stat-row {
  margin-top: 0.85rem;
  display: flex;
  justify-content: space-between;
}

.stat-row p {
  margin: 0;
  color: rgba(9, 63, 71, 0.7);
}

.progress {
  margin-top: 0.5rem;
  border-radius: 999px;
  height: 0.45rem;
  background: #d2d8bf;
  overflow: hidden;
}

.progress span {
  display: block;
  width: 74%;
  height: 100%;
  background: linear-gradient(120deg, #0f6e76, #2b9096);
}

.ghost {
  margin-top: 1rem;
  width: 100%;
  border-radius: 999px;
  border: 1px solid var(--teal-900);
  background: transparent;
  color: var(--teal-900);
  padding: 0.62rem;
  font: inherit;
}

@media (max-width: 920px) {
  .content-grid {
    grid-template-columns: 1fr;
  }
}
</style>
