<script setup>
import { ref, onBeforeUnmount } from 'vue'
import MovieCard from './components/MovieCard.vue'
import { fetchMovies } from './api.js'
import tmdbLogo from './assets/tmdb-logo.svg'

const draftKey = ref('')
const apiKey = ref('')
const query = ref('')
const activeQuery = ref('')
const movies = ref([])
const page = ref(1)
const totalPages = ref(0)
const totalResults = ref(0)
const loading = ref(false)
const error = ref('')

let controller
let requestId = 0

async function loadMovies(nextPage = 1, term = activeQuery.value) {
  const id = ++requestId

  controller?.abort()

  const currentController = new AbortController()
  controller = currentController

  let timedOut = false

  const timer = setTimeout(() => {
    timedOut = true
    currentController.abort()
  }, 15000)

  loading.value = true
  error.value = ''
  movies.value = []

  try {
    const data = await fetchMovies({
      apiKey: apiKey.value,
      query: term,
      page: nextPage,
      signal: currentController.signal
    })

    if (id !== requestId) return

    movies.value = data.results
    page.value = data.page
    totalPages.value = Math.min(data.total_pages, 500)
    totalResults.value = data.total_results
    activeQuery.value = term
  } catch (err) {
    if (id !== requestId) return

    error.value = timedOut
      ? 'Request timed out. Check your connection and retry.'
      : err.name === 'AbortError'
        ? ''
        : err.status !== undefined
          ? err.message
          : 'Could not reach TMDB. Check your connection or try again later.'
  } finally {
    clearTimeout(timer)

    if (id === requestId) {
      loading.value = false
    }
  }
}

function connect() {
  const key = draftKey.value.trim()

  if (!key) return

  apiKey.value = key
  draftKey.value = ''
  activeQuery.value = ''
  query.value = ''

  loadMovies(1, '')
}

function search() {
  loadMovies(1, query.value.trim())
}

function popular() {
  query.value = ''
  loadMovies(1, '')
}

function disconnect() {
  requestId++

  controller?.abort()

  apiKey.value = ''
  draftKey.value = ''
  query.value = ''
  activeQuery.value = ''

  movies.value = []
  error.value = ''
  loading.value = false

  page.value = 1
  totalPages.value = 0
  totalResults.value = 0
}

onBeforeUnmount(() => {
  requestId++
  controller?.abort()
})
</script>


<template>

  <!-- =====================================================
       WELCOME / API KEY SCREEN
  ====================================================== -->

  <div
    v-if="!apiKey"
    class="welcome-screen"
  >

    <div class="welcome-noise"></div>

    <div class="welcome-content">

      <div class="welcome-logo">

        <span class="logo-mark">M</span>

        <span>MOVIELAB</span>

      </div>


      <div class="welcome-line"></div>


      <p class="welcome-kicker">
        YOUR PERSONAL CINEMA DATABASE
      </p>


      <h1>
        FIND SOMETHING
        <span>WORTH WATCHING.</span>
      </h1>


      <p class="welcome-description">
        Explore thousands of films using live data from TMDB.
      </p>


      <form
        class="welcome-form"
        @submit.prevent="connect"
      >

        <label for="tmdb-key">
          TMDB API KEY
        </label>


        <div class="welcome-input">

          <input
            id="tmdb-key"
            v-model="draftKey"
            type="password"
            placeholder="Paste your API Key (v3)"
            autocomplete="off"
            spellcheck="false"
            required
          />


          <button
            type="submit"
            :disabled="!draftKey.trim()"
          >
            ENTER
            <span>→</span>
          </button>

        </div>


        <p>
          Your key stays only in this browser tab.
          <a
            href="https://www.themoviedb.org/settings/api"
            target="_blank"
            rel="noopener"
          >
            Get your TMDB key
          </a>
        </p>

      </form>


      <div class="welcome-footer">

        <span>VUE + TMDB API</span>

        <span>WEEK 04</span>

      </div>

    </div>

  </div>


  <!-- =====================================================
       MAIN APPLICATION
  ====================================================== -->

  <div
    v-else
    class="app-shell"
  >

    <!-- SIDEBAR -->

    <aside class="sidebar">

      <div class="sidebar-logo">

        <span class="logo-mark">M</span>

        <div>
          <strong>MOVIE</strong>
          <span>LAB</span>
        </div>

      </div>


      <nav class="side-navigation">

        <p>MENU</p>


        <a
          href="#home"
          class="active"
        >
          <span>⌂</span>
          Home
        </a>


        <a href="#movies">
          <span>◈</span>
          Discover
        </a>


        <a href="#movies">
          <span>★</span>
          Popular
        </a>


        <a href="#about">
          <span>?</span>
          About
        </a>

      </nav>


      <div class="sidebar-bottom">

        <div class="sidebar-status">

          <span class="status-dot"></span>

          TMDB CONNECTED

        </div>


        <button
          class="disconnect"
          @click="disconnect"
        >
          Change API key
        </button>

      </div>

    </aside>


    <!-- MAIN CONTENT -->

    <main
      id="home"
      class="main-content"
    >

      <!-- =================================================
           CINEMATIC HERO
      ================================================== -->

      <section class="cinema-hero">

        <!-- ACTUAL IMAGE -->

        <img
          class="hero-image"
          src="/images/cinema-bg.jpg"
          alt=""
          aria-hidden="true"
        />


        <!-- DARK GRADIENT -->

        <div class="hero-dark"></div>


        <!-- CONTENT -->

        <div class="hero-content">

          <div class="hero-tag">

            <span></span>

            MOVIELAB DISCOVER

          </div>


          <h1>

            STORIES

            <br>

            <em>WAITING</em>

            <br>

            TO BE FOUND.

          </h1>


          <p class="hero-description">
            Search through thousands of movies,
            discover something unexpected,
            and find your next obsession.
          </p>


          <!-- SEARCH -->

          <form
            class="big-search"
            @submit.prevent="search"
          >

            <span class="search-icon">
              ⌕
            </span>


            <input
              v-model="query"
              type="search"
              placeholder="Search for a movie..."
              autocomplete="off"
            />


            <button
              type="submit"
              :disabled="loading"
            >
              SEARCH
            </button>

          </form>


          <button
            class="popular-button"
            @click="popular"
          >

            <span>★</span>

            Browse popular movies

          </button>

        </div>


        <div class="hero-number">
          01
        </div>

      </section>


      <!-- =================================================
           MOVIE CATALOG
      ================================================== -->

      <section
        id="movies"
        class="movies-section"
        :aria-busy="loading"
      >

        <div class="section-top">

          <div>

            <span class="section-label">

              {{ activeQuery
                ? 'SEARCH RESULTS'
                : 'NOW SHOWING'
              }}

            </span>


            <h2>

              {{ activeQuery
                ? `Results for “${activeQuery}”`
                : 'Popular right now'
              }}

            </h2>

          </div>


          <div
            v-if="!loading && movies.length"
            class="result-info"
          >

            <strong>
              {{ totalResults }}
            </strong>

            RESULTS

          </div>

        </div>


        <!-- LOADING -->

        <div
          v-if="loading"
          class="loading-state"
        >

          <div class="loader"></div>

          <span>
            SEARCHING THE DATABASE
          </span>

        </div>


        <!-- ERROR -->

        <div
          v-else-if="error"
          class="error-state"
        >

          <div class="error-icon">
            !
          </div>


          <h3>
            Something went wrong.
          </h3>


          <p>
            {{ error }}
          </p>


          <button @click="search">
            TRY AGAIN →
          </button>

        </div>


        <!-- MOVIES -->

        <template v-else>

          <div
            v-if="movies.length"
            class="movie-grid"
          >

            <div
              v-for="(movie, index) in movies"
              :key="movie.id"
              class="movie-wrapper"
              :style="{ '--delay': `${index * 40}ms` }"
            >

              <div class="movie-number">
                {{ String(index + 1).padStart(2, '0') }}
              </div>


              <MovieCard :movie="movie" />

            </div>

          </div>


          <!-- EMPTY -->

          <div
            v-else
            class="empty-state"
          >

            <div class="empty-number">
              404
            </div>


            <h3>
              Nothing found.
            </h3>


            <p>
              Try another movie title or explore what's popular.
            </p>


            <button @click="popular">
              SHOW POPULAR MOVIES →
            </button>

          </div>


          <!-- PAGINATION -->

          <div
            v-if="totalPages > 1"
            class="pagination"
          >

            <button
              :disabled="page <= 1"
              @click="loadMovies(page - 1)"
            >
              ←
            </button>


            <div>

              <span>PAGE</span>

              <strong>
                {{ String(page).padStart(2, '0') }}
              </strong>

              <span>
                /
                {{ String(totalPages).padStart(2, '0') }}
              </span>

            </div>


            <button
              :disabled="page >= totalPages"
              @click="loadMovies(page + 1)"
            >
              →
            </button>

          </div>

        </template>

      </section>


      <!-- =================================================
           FOOTER
      ================================================== -->

      <footer
        id="about"
        class="cinema-footer"
      >

        <div class="footer-title">
          MOVIELAB
        </div>


        <div class="footer-info">

          <div>

            <span>PROJECT</span>

            <strong>
              VUE + TMDB API
            </strong>

          </div>


          <div>

            <span>COURSE</span>

            <strong>
              WEB DESIGN / WEEK 04
            </strong>

          </div>


          <div>

            <span>DATA PROVIDED BY</span>

            <a
              href="https://www.themoviedb.org/"
              target="_blank"
              rel="noopener"
            >

              <img
                :src="tmdbLogo"
                alt="TMDB"
              />

            </a>

          </div>

        </div>


        <p>
          This product uses the TMDB API but is not endorsed or certified by TMDB.
        </p>

      </footer>

    </main>

  </div>

</template>


<style>

@import url('https://fonts.googleapis.com/css2?family=DM+Mono:wght@400;500&family=DM+Sans:wght@400;500;600;700&family=Space+Grotesk:wght@400;500;600;700&display=swap');


/* =====================================================
   VARIABLES
===================================================== */

:root {

  --black: #070707;

  --dark: #0d0d0d;

  --panel: #131313;

  --line: #292929;

  --white: #f5f4ef;

  --muted: #898987;

  --red: #ff3838;

}


/* =====================================================
   GLOBAL
===================================================== */

* {
  box-sizing: border-box;
}


html {
  scroll-behavior: smooth;
}


body {

  margin: 0;

  background: var(--black);

  color: var(--white);

  font-family: 'DM Sans', sans-serif;

}


button,
input {
  font: inherit;
}


button {
  cursor: pointer;
}


button:disabled {

  cursor: not-allowed;

  opacity: .35;

}


/* =====================================================
   WELCOME SCREEN
===================================================== */

.welcome-screen {

  min-height: 100vh;

  background:
    radial-gradient(
      circle at 75% 35%,
      #281313 0,
      transparent 35%
    ),
    #070707;

  position: relative;

  overflow: hidden;

}


.welcome-noise {

  position: absolute;

  inset: 0;

  opacity: .035;

  background-image:
    repeating-linear-gradient(
      0deg,
      #fff 0,
      #fff 1px,
      transparent 1px,
      transparent 4px
    );

}


.welcome-content {

  position: relative;

  width: min(1100px, 90%);

  margin: auto;

  min-height: 100vh;

  padding: 70px 0;

  display: flex;

  flex-direction: column;

  justify-content: center;

}


.welcome-logo {

  display: flex;

  align-items: center;

  gap: 14px;

  font-family: 'Space Grotesk';

  font-weight: 700;

  letter-spacing: 2px;

}


.logo-mark {

  width: 42px;

  height: 42px;

  display: grid;

  place-items: center;

  background: var(--red);

  color: #000;

  font-weight: 900;

  transform: skew(-10deg);

}


.welcome-line {

  width: 100%;

  height: 1px;

  background: var(--line);

  margin: 60px 0 35px;

}


.welcome-kicker,
.section-label,
.hero-tag {

  font-family: 'DM Mono';

  font-size: 11px;

  letter-spacing: 2px;

  color: var(--red);

}


.welcome-content h1 {

  max-width: 900px;

  margin: 20px 0;

  font-family: 'Space Grotesk';

  font-size: clamp(55px, 9vw, 120px);

  line-height: .88;

  letter-spacing: -6px;

}


.welcome-content h1 span {

  color: transparent;

  -webkit-text-stroke: 1px #777;

}


.welcome-description {

  max-width: 500px;

  color: var(--muted);

  font-size: 17px;

  line-height: 1.6;

}


.welcome-form {

  width: min(700px, 100%);

  margin-top: 40px;

}


.welcome-form label {

  display: block;

  font-family: 'DM Mono';

  font-size: 10px;

  letter-spacing: 2px;

  color: #777;

  margin-bottom: 10px;

}


.welcome-input {

  display: flex;

  border-bottom: 2px solid var(--white);

}


.welcome-input input {

  flex: 1;

  min-width: 0;

  padding: 18px 0;

  border: 0;

  outline: 0;

  background: transparent;

  color: white;

  font-size: 18px;

}


.welcome-input input::placeholder {
  color: #555;
}


.welcome-input button {

  border: 0;

  padding: 0 24px;

  background: var(--red);

  color: white;

  font-weight: 800;

}


.welcome-input button span {

  margin-left: 15px;

  font-size: 20px;

}


.welcome-form > p {

  font-size: 12px;

  color: #666;

}


.welcome-form a {

  color: var(--red);

}


.welcome-footer {

  position: absolute;

  bottom: 30px;

  left: 0;

  right: 0;

  display: flex;

  justify-content: space-between;

  color: #555;

  font-family: 'DM Mono';

  font-size: 10px;

  letter-spacing: 1px;

}


/* =====================================================
   APP
===================================================== */

.app-shell {

  min-height: 100vh;

  display: flex;

}


/* =====================================================
   SIDEBAR
===================================================== */

.sidebar {

  width: 230px;

  min-height: 100vh;

  position: fixed;

  left: 0;

  top: 0;

  bottom: 0;

  border-right: 1px solid var(--line);

  background: #0b0b0b;

  padding: 30px 25px;

  display: flex;

  flex-direction: column;

  z-index: 20;

}


.sidebar-logo {

  display: flex;

  align-items: center;

  gap: 12px;

  font-family: 'Space Grotesk';

}


.sidebar-logo .logo-mark {

  width: 34px;

  height: 34px;

}


.sidebar-logo strong,
.sidebar-logo span:last-child {

  display: block;

}


.sidebar-logo strong {

  font-size: 13px;

}


.sidebar-logo span:last-child {

  color: var(--red);

  font-size: 13px;

}


.side-navigation {

  margin-top: 80px;

}


.side-navigation p {

  color: #444;

  font-family: 'DM Mono';

  font-size: 9px;

  letter-spacing: 2px;

}


.side-navigation a {

  display: flex;

  align-items: center;

  gap: 15px;

  padding: 14px 0;

  color: #777;

  text-decoration: none;

  font-size: 13px;

  transition: .2s;

}


.side-navigation a span {

  width: 20px;

  font-size: 15px;

}


.side-navigation a:hover,
.side-navigation a.active {

  color: white;

}


.side-navigation a.active {

  border-right: 2px solid var(--red);

}


.sidebar-bottom {

  margin-top: auto;

}


.sidebar-status {

  font-family: 'DM Mono';

  font-size: 8px;

  letter-spacing: 1px;

  color: #666;

  display: flex;

  align-items: center;

  gap: 8px;

}


.status-dot {

  width: 6px;

  height: 6px;

  background: var(--red);

  border-radius: 50%;

}


.disconnect {

  margin-top: 20px;

  border: 0;

  background: transparent;

  color: #555;

  font-size: 11px;

  padding: 0;

}


.disconnect:hover {
  color: white;
}


/* =====================================================
   MAIN
===================================================== */

.main-content {

  margin-left: 230px;

  width: calc(100% - 230px);

}


/* =====================================================
   CINEMATIC HERO
===================================================== */

.cinema-hero {

  min-height: 720px;

  position: relative;

  overflow: hidden;

  border-bottom: 1px solid var(--line);

  display: flex;

  align-items: center;

  background: #050505;

}


/* REAL IMAGE */

.hero-image {

  position: absolute;

  inset: 0;

  width: 100%;

  height: 100%;

  object-fit: cover;

  object-position: center center;

  z-index: 0;

  display: block;

}


/* Dark overlay */

.hero-dark {

  position: absolute;

  inset: 0;

  z-index: 1;

  background:

    linear-gradient(
      90deg,
      rgba(3, 3, 3, 0.94) 0%,
      rgba(3, 3, 3, 0.78) 25%,
      rgba(3, 3, 3, 0.38) 52%,
      rgba(3, 3, 3, 0.08) 80%,
      rgba(3, 3, 3, 0.20) 100%
    ),

    linear-gradient(
      0deg,
      rgba(0, 0, 0, 0.65) 0%,
      rgba(0, 0, 0, 0.05) 55%,
      rgba(0, 0, 0, 0.15) 100%
    );

}


/* Hero text */

.hero-content {

  position: relative;

  z-index: 3;

  padding: 80px;

  max-width: 850px;

}


.hero-tag {

  display: flex;

  align-items: center;

  gap: 10px;

  text-shadow:
    0 2px 10px rgba(0, 0, 0, .9);

}


.hero-tag span {

  width: 30px;

  height: 1px;

  background: var(--red);

}


.cinema-hero h1 {

  font-family: 'Space Grotesk';

  font-size: clamp(65px, 8vw, 115px);

  line-height: .85;

  letter-spacing: -7px;

  margin: 35px 0;

  max-width: 800px;

  color: #f5f4ef;

  text-shadow:
    0 4px 30px rgba(0, 0, 0, .95);

}


.cinema-hero h1 em {

  color: var(--red);

  font-style: normal;

}


.hero-description {

  color: #d0d0d0;

  max-width: 500px;

  line-height: 1.6;

  text-shadow:
    0 2px 15px rgba(0, 0, 0, .95);

}


/* =====================================================
   SEARCH
===================================================== */

.big-search {

  margin-top: 35px;

  width: min(650px, 100%);

  height: 60px;

  display: flex;

  border: 1px solid rgba(255, 255, 255, .28);

  background: rgba(8, 8, 8, .88);

  backdrop-filter: blur(4px);

  box-shadow:
    0 15px 40px rgba(0, 0, 0, .45);

}


.search-icon {

  width: 55px;

  display: grid;

  place-items: center;

  color: #999;

  font-size: 25px;

}


.big-search input {

  flex: 1;

  min-width: 0;

  border: 0;

  outline: 0;

  background: transparent;

  color: white;

  font-size: 15px;

}


.big-search input::placeholder {

  color: #777;

}


.big-search button {

  border: 0;

  background: var(--red);

  color: white;

  padding: 0 25px;

  font-family: 'DM Mono';

  font-size: 10px;

  font-weight: 600;

}


.popular-button {

  margin-top: 18px;

  background: transparent;

  border: 0;

  padding: 0;

  color: #ccc;

  font-size: 12px;

  text-shadow:
    0 2px 10px rgba(0, 0, 0, .9);

}


.popular-button:hover {

  color: white;

}


.popular-button span {

  color: var(--red);

  margin-right: 8px;

}


.hero-number {

  position: absolute;

  z-index: 3;

  right: 60px;

  bottom: 35px;

  font-family: 'DM Mono';

  font-size: 11px;

  color: #aaa;

}


/* =====================================================
   MOVIES
===================================================== */

.movies-section {

  padding: 80px;

  background: var(--black);

}


.section-top {

  display: flex;

  align-items: end;

  justify-content: space-between;

  border-bottom: 1px solid var(--line);

  padding-bottom: 25px;

  margin-bottom: 45px;

}


.section-top h2 {

  font-family: 'Space Grotesk';

  font-size: 40px;

  margin: 10px 0 0;

  letter-spacing: -2px;

}


.result-info {

  font-family: 'DM Mono';

  font-size: 9px;

  color: #555;

}


.result-info strong {

  color: var(--red);

  font-size: 18px;

  margin-right: 5px;

}


/* =====================================================
   MOVIE GRID
===================================================== */

.movie-grid {

  display: grid;

  grid-template-columns:
    repeat(5, minmax(0, 1fr));

  gap: 35px 20px;

}


.movie-wrapper {

  position: relative;

  animation: reveal .5s both;

  animation-delay: var(--delay);

}


.movie-number {

  position: absolute;

  z-index: 3;

  top: 10px;

  left: 10px;

  font-family: 'DM Mono';

  font-size: 9px;

  color: white;

  background: #000;

  padding: 5px 7px;

}


.movie-wrapper :deep(.movie-card) {

  transition:
    transform .3s ease,
    filter .3s ease;

}


.movie-wrapper:hover :deep(.movie-card) {

  transform: translateY(-8px);

  filter: brightness(1.08);

}


@keyframes reveal {

  from {

    opacity: 0;

    transform: translateY(20px);

  }

  to {

    opacity: 1;

    transform: translateY(0);

  }

}


/* =====================================================
   LOADING
===================================================== */

.loading-state {

  min-height: 300px;

  display: grid;

  place-items: center;

  align-content: center;

  gap: 20px;

  color: #555;

  font-family: 'DM Mono';

  font-size: 10px;

  letter-spacing: 2px;

}


.loader {

  width: 35px;

  height: 35px;

  border: 2px solid #333;

  border-top-color: var(--red);

  border-radius: 50%;

  animation: spin 1s linear infinite;

}


@keyframes spin {

  to {
    transform: rotate(360deg);
  }

}


/* =====================================================
   ERROR / EMPTY
===================================================== */

.error-state,
.empty-state {

  padding: 100px 20px;

  text-align: center;

  border: 1px solid var(--line);

}


.error-icon,
.empty-number {

  font-family: 'Space Grotesk';

  font-size: 70px;

  font-weight: 700;

  color: var(--red);

}


.error-state h3,
.empty-state h3 {

  font-size: 25px;

}


.error-state p,
.empty-state p {

  color: #666;

}


.error-state button,
.empty-state button {

  margin-top: 15px;

  border: 0;

  background: var(--red);

  color: white;

  padding: 13px 20px;

  font-size: 10px;

  font-weight: 800;

}


/* =====================================================
   PAGINATION
===================================================== */

.pagination {

  margin-top: 60px;

  padding-top: 25px;

  border-top: 1px solid var(--line);

  display: flex;

  justify-content: space-between;

  align-items: center;

}


.pagination button {

  width: 45px;

  height: 45px;

  border: 1px solid #333;

  background: transparent;

  color: white;

  font-size: 20px;

}


.pagination button:hover:not(:disabled) {

  background: var(--red);

  color: white;

}


.pagination div {

  font-family: 'DM Mono';

  color: #555;

  font-size: 10px;

}


.pagination strong {

  color: white;

  font-size: 18px;

  margin: 0 7px;

}


/* =====================================================
   FOOTER
===================================================== */

.cinema-footer {

  border-top: 1px solid var(--line);

  padding: 60px 80px 40px;

  background: #080808;

}


.footer-title {

  font-family: 'Space Grotesk';

  font-size: 55px;

  font-weight: 700;

  letter-spacing: -3px;

}


.footer-info {

  display: grid;

  grid-template-columns:
    repeat(3, 1fr);

  gap: 30px;

  margin: 50px 0;

}


.footer-info div {

  display: flex;

  flex-direction: column;

  gap: 8px;

}


.footer-info span {

  color: #444;

  font-family: 'DM Mono';

  font-size: 9px;

}


.footer-info strong {

  font-size: 11px;

}


.footer-info img {

  width: 80px;

}


.cinema-footer > p {

  border-top: 1px solid var(--line);

  padding-top: 20px;

  color: #444;

  font-size: 10px;

}


/* =====================================================
   RESPONSIVE
===================================================== */

@media (max-width: 1200px) {

  .movie-grid {

    grid-template-columns:
      repeat(4, 1fr);

  }

}


@media (max-width: 900px) {

  .sidebar {

    width: 70px;

    padding: 25px 15px;

  }


  .sidebar-logo div,
  .side-navigation p,
  .sidebar-status,
  .disconnect {

    display: none;

  }


  .side-navigation a {

    justify-content: center;

  }


  .main-content {

    margin-left: 70px;

    width: calc(100% - 70px);

  }


  .hero-content,
  .movies-section,
  .cinema-footer {

    padding-left: 45px;

    padding-right: 45px;

  }


  .movie-grid {

    grid-template-columns:
      repeat(3, 1fr);

  }

}


@media (max-width: 650px) {

  .welcome-content {

    padding: 40px 0;

  }


  .welcome-content h1 {

    letter-spacing: -3px;

  }


  .welcome-footer {

    position: static;

    margin-top: 60px;

  }


  .sidebar {

    display: none;

  }


  .main-content {

    margin-left: 0;

    width: 100%;

  }


  .cinema-hero {

    min-height: 650px;

  }


  .hero-image {

    object-position: 62% center;

  }


  .hero-content,
  .movies-section,
  .cinema-footer {

    padding-left: 22px;

    padding-right: 22px;

  }


  .cinema-hero h1 {

    font-size: 58px;

    letter-spacing: -4px;

  }


  .big-search {

    height: auto;

    flex-wrap: wrap;

  }


  .big-search input {

    height: 58px;

  }


  .big-search button {

    width: 100%;

    height: 50px;

  }


  .movie-grid {

    grid-template-columns:
      repeat(2, 1fr);

    gap: 25px 12px;

  }


  .section-top {

    align-items: start;

    flex-direction: column;

    gap: 20px;

  }


  .section-top h2 {

    font-size: 30px;

  }


  .footer-title {

    font-size: 40px;

  }


  .footer-info {

    grid-template-columns: 1fr;

  }

}

</style>