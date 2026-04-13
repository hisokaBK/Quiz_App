<script setup lang="ts">
import { computed } from "vue";
import { useRoute, useRouter } from "vue-router";

const router = useRouter();
const route = useRoute();
const subject = computed(() => String(route.query.subject ?? "Geometry"));
const userName = computed(() => {
  const routeUser = String(route.query.user ?? "").trim();
  if (routeUser) {
    return routeUser;
  }
  const localUser = localStorage.getItem("quiz_user_name")?.trim();
  return localUser || "You";
});
const cachedResult = computed(() => {
  try {
    const raw = localStorage.getItem("quiz_last_result");
    if (!raw) {
      return null;
    }
    return JSON.parse(raw) as { score?: number; total?: number; percent?: number };
  } catch {
    return null;
  }
});
const score = computed(() => {
  const routeScore = Number(route.query.score);
  const raw = Number.isFinite(routeScore) ? routeScore : Number(cachedResult.value?.score ?? 0);
  return Number.isFinite(raw) ? raw : 0;
});
const total = computed(() => {
  const routeTotal = Number(route.query.total);
  const raw = Number.isFinite(routeTotal) && routeTotal > 0 ? routeTotal : Number(cachedResult.value?.total ?? 0);
  return Number.isFinite(raw) && raw > 0 ? raw : 0;
});
const percent = computed(() => {
  if (!total.value) {
    return 0;
  }
  return Math.round((score.value / total.value) * 100);
});

const players = computed(() => {
  const base = [
    { name: userName.value, score: percent.value },
    { name: "Leslie", score: Math.max(55, Math.min(99, percent.value + 4)) },
    { name: "Alex", score: Math.max(52, Math.min(98, percent.value + 2)) },
    { name: "Nora", score: Math.max(48, Math.min(97, percent.value - 3)) },
    { name: "Mason", score: Math.max(45, Math.min(96, percent.value - 6)) },
  ];

  return base
    .sort((a, b) => b.score - a.score)
    .map((player, index) => ({ ...player, rank: index + 1 }));
});
</script>

<template>
  <main class="page">
    <header class="head">
      <button type="button" @click="router.push({ name: 'Quiz', query: { subject } })">Back To Quiz</button>
      <h1>Leaderboard</h1>
      <button type="button" @click="router.push({ name: 'Home' })">Home</button>
    </header>

    <section class="layout">
      <article class="panel board">
        <div class="score-box">
          <p>Your Final Score</p>
          <strong>{{ score }}/{{ total || "-" }}</strong>
          <span>{{ percent }}%</span>
        </div>

        <div class="tabs">
          <button class="tab active" type="button">Weekly</button>
          <button class="tab" type="button">All Time</button>
        </div>

        <div class="podium">
          <div class="pillar second">2</div>
          <div class="pillar first">1</div>
          <div class="pillar third">3</div>
        </div>

        <ul class="ranking">
          <li v-for="player in players" :key="player.name" :class="{ me: player.name === userName }">
            <span>#{{ player.rank }}</span>
            <strong>{{ player.name }}</strong>
            <em>{{ player.score }}%</em>
          </li>
        </ul>
      </article>

      <aside class="panel promo">
        <h2>Think <span>Outside the Box</span> with Quizzax</h2>
        <p>Take your learning to the next level with interactive and personalised quizzes.</p>
        <button type="button" @click="router.push({ name: 'Home' })">Continue</button>
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
  background: #d8ec77;
  padding: 1rem 1.1rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 0.7rem;
}

.head button {
  border-radius: 999px;
  border: 1px solid rgba(9, 63, 71, 0.22);
  background: rgba(255, 255, 255, 0.6);
  color: var(--ink);
  padding: 0.46rem 0.82rem;
  font: inherit;
}

h1 {
  margin: 0;
  font-size: 1.45rem;
  font-family: "Sora", "Segoe UI", sans-serif;
}

.layout {
  display: grid;
  grid-template-columns: 1.5fr 1fr;
  gap: 1rem;
}

.panel {
  background: #f4f4ef;
  padding: 1rem;
}

.tabs {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  border-radius: 999px;
  background: #fff;
  border: 1px solid #d1d3c7;
  padding: 0.2rem;
}

.score-box {
  margin-bottom: 0.9rem;
  border-radius: 1rem;
  border: 1px solid rgba(8, 88, 95, 0.16);
  background: #d8ec77;
  padding: 0.75rem;
  display: grid;
  grid-template-columns: 1fr auto;
  align-items: center;
  gap: 0.25rem 0.6rem;
}

.score-box p {
  margin: 0;
  font-size: 0.82rem;
  color: rgba(9, 63, 71, 0.72);
}

.score-box strong {
  font-size: 1.3rem;
  color: var(--teal-900);
  font-family: "Sora", "Segoe UI", sans-serif;
}

.score-box span {
  grid-column: 2;
  grid-row: 1 / span 2;
  justify-self: end;
  border-radius: 999px;
  background: #f4f4ef;
  border: 1px solid rgba(8, 88, 95, 0.2);
  padding: 0.3rem 0.6rem;
  font-weight: 700;
  color: var(--teal-900);
}

.tab {
  border: 0;
  border-radius: 999px;
  background: transparent;
  padding: 0.52rem;
  font: inherit;
  color: rgba(9, 63, 71, 0.72);
}

.tab.active {
  background: var(--teal-900);
  color: #d8ec77;
}

.podium {
  margin-top: 1rem;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  align-items: end;
  gap: 0.65rem;
}

.pillar {
  border-radius: 0.85rem 0.85rem 0.5rem 0.5rem;
  display: grid;
  place-items: center;
  color: var(--teal-900);
  font-weight: 700;
  background: #dce8aa;
  border: 1px solid rgba(8, 88, 95, 0.16);
}

.first {
  height: 8rem;
  background: var(--teal-900);
  color: #d8ec77;
}

.second {
  height: 6.2rem;
}

.third {
  height: 5.2rem;
}

.ranking {
  margin: 1rem 0 0;
  padding: 0;
  list-style: none;
}

.ranking li {
  display: grid;
  grid-template-columns: 4rem 1fr auto;
  align-items: center;
  gap: 0.6rem;
  background: #fff;
  border: 1px solid #d1d3c7;
  border-radius: 0.8rem;
  padding: 0.6rem 0.75rem;
}

.ranking li + li {
  margin-top: 0.5rem;
}

.ranking span {
  color: rgba(9, 63, 71, 0.66);
}

.ranking em {
  font-style: normal;
  font-weight: 600;
}

.ranking li.me {
  border-color: rgba(8, 88, 95, 0.36);
  background: #ecf6bb;
}

.promo {
  background: linear-gradient(155deg, #074c56, #0a6471);
  color: #d2ef9b;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

h2 {
  margin: 0;
  font-family: "Sora", "Segoe UI", sans-serif;
  font-size: clamp(1.8rem, 3vw, 2.4rem);
  line-height: 1.05;
}

h2 span {
  color: #f0f6c2;
}

.promo p {
  margin: 0.8rem 0 0;
  color: rgba(217, 239, 161, 0.9);
}

.promo button {
  width: fit-content;
  margin-top: 1rem;
  border: 1px solid #d2ef9b;
  border-radius: 999px;
  background: transparent;
  color: #d2ef9b;
  padding: 0.56rem 1rem;
  font: inherit;
}

@media (max-width: 920px) {
  .layout {
    grid-template-columns: 1fr;
  }
}
</style>
