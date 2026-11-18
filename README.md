# Quizzax — Vue Quiz Application

A browser-based quiz application built with Vue 3 and TypeScript. The frontend code is stored under the repository's `backend/` directory.

## Features

- Enter a name before starting a quiz.
- Fetch trivia questions from the Open Trivia Database (OpenTDB) API.
- Show multiple-choice answers in randomized order.
- Use a countdown timer for each question.
- Track quiz progress and display a result screen.

## Technologies

Vue 3, TypeScript, Vite, Vue Router and Axios.

## Layout

- `backend/src/pages/Home.vue` — welcome screen and player name.
- `backend/src/pages/Quiz.vue` — question fetching, answer selection and timer.
- `backend/src/pages/Result.vue` — quiz results.
- `backend/src/router/` — routes.
- `backend/src/components/` — reusable UI components.

## Run locally

~~~bash
cd backend
npm install
npm run dev
~~~

Open the local URL shown by Vite. A network connection is needed for questions fetched from OpenTDB.

## Scope

The folder is named `backend`, but the code in it is a Vue frontend application. The nested README is the starter Vite/Vue template; this root README documents the actual quiz project.
