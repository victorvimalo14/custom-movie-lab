<script setup>
import { ref } from 'vue'

defineProps({
  movie: {
    type: Object,
    required: true
  }
})

const showDetails = ref(false)

function posterUrl(path) {
  return path
    ? `https://image.tmdb.org/t/p/w500${path}`
    : ''
}

function backdropUrl(path) {
  return path
    ? `https://image.tmdb.org/t/p/w1280${path}`
    : ''
}

function movieYear(date) {
  return date ? date.slice(0, 4) : '—'
}

function rating(value) {
  return value ? Number(value).toFixed(1) : '—'
}

function openDetails() {
  showDetails.value = true
}

function closeDetails() {
  showDetails.value = false
}
</script>


<template>

  <article class="movie-card">

    <!-- POSTER -->

    <div
      class="poster-container"
      @click="openDetails"
    >

      <img
        v-if="movie.poster_path"
        class="movie-poster"
        :src="posterUrl(movie.poster_path)"
        :alt="`${movie.title} poster`"
        loading="lazy"
      />

      <div
        v-else
        class="poster-placeholder"
      >
        NO POSTER
      </div>


      <!-- HOVER OVERLAY -->

      <div class="movie-overlay">

        <div class="overlay-top">

          <span class="movie-type">
            FILM
          </span>

          <span class="movie-rating">
            ★ {{ rating(movie.vote_average) }}
          </span>

        </div>


        <div class="overlay-bottom">

          <p class="movie-overview">
            {{ movie.overview || 'No description available.' }}
          </p>


          <!-- REAL BUTTON -->

          <button
            type="button"
            class="discover-button"
            @click.stop="openDetails"
          >

            VIEW DETAILS

            <span>→</span>

          </button>

        </div>

      </div>


      <!-- RATING -->

      <div class="rating-badge">

        <span>★</span>

        {{ rating(movie.vote_average) }}

      </div>

    </div>


    <!-- INFORMATION -->

    <div class="movie-info">

      <h3>
        {{ movie.title }}
      </h3>


      <div class="movie-meta">

        <span>
          {{ movieYear(movie.release_date) }}
        </span>

        <span class="meta-dot"></span>

        <span>
          TMDB
        </span>

      </div>

    </div>


    <!-- =========================================
         MOVIE DETAILS MODAL
    ========================================== -->

    <Teleport to="body">

      <div
        v-if="showDetails"
        class="details-backdrop"
        @click.self="closeDetails"
      >

        <div class="details-modal">

          <!-- CLOSE -->

          <button
            type="button"
            class="close-button"
            aria-label="Close movie details"
            @click="closeDetails"
          >
            ×
          </button>


          <!-- BACKDROP IMAGE -->

          <div
            v-if="movie.backdrop_path"
            class="details-backdrop-image"
            :style="{
              backgroundImage: `url(${backdropUrl(movie.backdrop_path)})`
            }"
          ></div>

          <div
            v-else
            class="details-backdrop-image fallback"
          ></div>


          <!-- MODAL CONTENT -->

          <div class="details-content">

            <div class="details-poster">

              <img
                v-if="movie.poster_path"
                :src="posterUrl(movie.poster_path)"
                :alt="`${movie.title} poster`"
              />

            </div>


            <div class="details-text">

              <span class="details-label">
                MOVIE DETAILS
              </span>


              <h2>
                {{ movie.title }}
              </h2>


              <div class="details-meta">

                <span>
                  {{ movieYear(movie.release_date) }}
                </span>

                <span class="meta-separator">
                  /
                </span>

                <span class="details-rating">
                  ★ {{ rating(movie.vote_average) }}
                </span>

              </div>


              <p class="details-overview">

                {{
                  movie.overview ||
                  'No description is available for this movie.'
                }}

              </p>


              <div class="details-source">

                <span class="source-dot"></span>

                DATA FROM TMDB

              </div>

            </div>

          </div>

        </div>

      </div>

    </Teleport>

  </article>

</template>


<style scoped>

/* =========================================
   CARD
========================================= */

.movie-card {

  width: 100%;

  min-width: 0;

  color: #f5f4ef;

}


/* =========================================
   POSTER
========================================= */

.poster-container {

  position: relative;

  width: 100%;

  aspect-ratio: 2 / 3;

  overflow: hidden;

  background: #111;

  border: 1px solid #252525;

  cursor: pointer;

  transition:
    border-color .3s ease,
    box-shadow .3s ease;

}


.movie-card:hover .poster-container {

  border-color: #ff3838;

  box-shadow:
    0 15px 45px rgba(0, 0, 0, .55),
    0 0 0 1px rgba(255, 56, 56, .15);

}


/* =========================================
   POSTER IMAGE
========================================= */

.movie-poster {

  width: 100%;

  height: 100%;

  display: block;

  object-fit: cover;

  transition:
    transform .5s cubic-bezier(.2, .7, .2, 1),
    filter .5s ease;

}


.movie-card:hover .movie-poster {

  transform: scale(1.06);

  filter:
    brightness(.45)
    saturate(.85);

}


/* =========================================
   PLACEHOLDER
========================================= */

.poster-placeholder {

  width: 100%;

  height: 100%;

  display: grid;

  place-items: center;

  color: #555;

  font-family: 'DM Mono', monospace;

  font-size: 9px;

  letter-spacing: 2px;

}


/* =========================================
   HOVER OVERLAY
========================================= */

.movie-overlay {

  position: absolute;

  inset: 0;

  padding: 14px;

  display: flex;

  flex-direction: column;

  justify-content: space-between;

  opacity: 0;

  background:
    linear-gradient(
      180deg,
      rgba(0, 0, 0, .65),
      transparent 35%,
      rgba(0, 0, 0, .95)
    );

  transition:
    opacity .3s ease;

}


.movie-card:hover .movie-overlay {

  opacity: 1;

}


/* =========================================
   TOP
========================================= */

.overlay-top {

  display: flex;

  justify-content: space-between;

  align-items: center;

}


.movie-type {

  padding: 5px 7px;

  background: #ff3838;

  color: #fff;

  font-family: 'DM Mono', monospace;

  font-size: 8px;

  letter-spacing: 1px;

}


.movie-rating {

  font-family: 'DM Mono', monospace;

  font-size: 10px;

  color: #fff;

  text-shadow:
    0 2px 8px #000;

}


/* =========================================
   BOTTOM
========================================= */

.overlay-bottom {

  transform:
    translateY(10px);

  transition:
    transform .35s ease;

}


.movie-card:hover .overlay-bottom {

  transform:
    translateY(0);

}


.movie-overview {

  margin: 0 0 15px;

  color: #ddd;

  font-size: 11px;

  line-height: 1.5;

  display: -webkit-box;

  -webkit-line-clamp: 5;

  -webkit-box-orient: vertical;

  overflow: hidden;

}


/* =========================================
   REAL DETAILS BUTTON
========================================= */

.discover-button {

  width: 100%;

  display: flex;

  align-items: center;

  justify-content: space-between;

  padding: 10px 0;

  border: 0;

  border-top: 1px solid rgba(255, 255, 255, .2);

  background: transparent;

  color: #ff3838;

  font-family: 'DM Mono', monospace;

  font-size: 8px;

  letter-spacing: 1px;

  cursor: pointer;

}


.discover-button span {

  font-size: 16px;

  transition:
    transform .2s ease;

}


.discover-button:hover span {

  transform:
    translateX(5px);

}


.discover-button:hover {

  color: white;

}


/* =========================================
   RATING BADGE
========================================= */

.rating-badge {

  position: absolute;

  right: 9px;

  bottom: 9px;

  z-index: 3;

  padding: 6px 8px;

  background: rgba(0, 0, 0, .88);

  border: 1px solid rgba(255, 255, 255, .12);

  font-family: 'DM Mono', monospace;

  font-size: 9px;

  color: white;

  transition:
    opacity .2s ease,
    transform .2s ease;

}


.rating-badge span {

  color: #ff3838;

}


.movie-card:hover .rating-badge {

  opacity: 0;

  transform:
    translateY(5px);

}


/* =========================================
   TITLE
========================================= */

.movie-info {

  padding-top: 13px;

}


.movie-info h3 {

  margin: 0;

  color: #f5f4ef;

  font-family: 'Space Grotesk', sans-serif;

  font-size: 15px;

  font-weight: 600;

  line-height: 1.15;

  white-space: nowrap;

  overflow: hidden;

  text-overflow: ellipsis;

}


.movie-meta {

  margin-top: 7px;

  display: flex;

  align-items: center;

  gap: 8px;

  color: #666;

  font-family: 'DM Mono', monospace;

  font-size: 8px;

  letter-spacing: .5px;

}


.meta-dot {

  width: 3px;

  height: 3px;

  border-radius: 50%;

  background: #ff3838;

}


/* =========================================
   DETAILS MODAL
========================================= */

.details-backdrop {

  position: fixed;

  inset: 0;

  z-index: 9999;

  display: flex;

  align-items: center;

  justify-content: center;

  padding: 30px;

  background:
    rgba(0, 0, 0, .82);

  backdrop-filter:
    blur(8px);

}


/* =========================================
   MODAL
========================================= */

.details-modal {

  position: relative;

  width: min(900px, 100%);

  max-height: 90vh;

  overflow: hidden;

  background: #111;

  border: 1px solid #333;

  box-shadow:
    0 30px 100px rgba(0, 0, 0, .8);

  animation:
    modal-in .25s ease-out;

}


@keyframes modal-in {

  from {

    opacity: 0;

    transform:
      translateY(20px)
      scale(.98);

  }

  to {

    opacity: 1;

    transform:
      translateY(0)
      scale(1);

  }

}


/* =========================================
   MODAL BACKDROP
========================================= */

.details-backdrop-image {

  position: absolute;

  top: 0;

  left: 0;

  right: 0;

  height: 300px;

  background-size: cover;

  background-position: center;

  opacity: .35;

}


.details-backdrop-image::after {

  content: "";

  position: absolute;

  inset: 0;

  background:
    linear-gradient(
      180deg,
      rgba(17, 17, 17, .15),
      #111 100%
    );

}


.details-backdrop-image.fallback {

  background:
    linear-gradient(
      135deg,
      #222,
      #111
    );

}


/* =========================================
   CLOSE BUTTON
========================================= */

.close-button {

  position: absolute;

  top: 15px;

  right: 15px;

  z-index: 10;

  width: 42px;

  height: 42px;

  display: grid;

  place-items: center;

  border: 1px solid rgba(255, 255, 255, .25);

  background: rgba(0, 0, 0, .65);

  color: white;

  font-size: 25px;

  line-height: 1;

  cursor: pointer;

}


.close-button:hover {

  background: #ff3838;

  border-color: #ff3838;

}


/* =========================================
   MODAL CONTENT
========================================= */

.details-content {

  position: relative;

  z-index: 2;

  min-height: 500px;

  padding: 180px 55px 55px;

  display: grid;

  grid-template-columns: 210px 1fr;

  gap: 45px;

  align-items: end;

}


.details-poster img {

  width: 100%;

  display: block;

  border:
    1px solid #333;

  box-shadow:
    0 20px 50px rgba(0, 0, 0, .65);

}


.details-text {

  padding-bottom: 5px;

}


.details-label {

  color: #ff3838;

  font-family: 'DM Mono', monospace;

  font-size: 9px;

  letter-spacing: 2px;

}


.details-text h2 {

  margin: 12px 0;

  color: #f5f4ef;

  font-family: 'Space Grotesk', sans-serif;

  font-size: clamp(35px, 5vw, 60px);

  line-height: .95;

  letter-spacing: -2px;

}


.details-meta {

  display: flex;

  align-items: center;

  gap: 12px;

  margin-bottom: 25px;

  color: #888;

  font-family: 'DM Mono', monospace;

  font-size: 10px;

}


.meta-separator {

  color: #444;

}


.details-rating {

  color: #ff3838;

}


.details-overview {

  max-width: 580px;

  color: #c2c2c2;

  font-size: 14px;

  line-height: 1.7;

}


.details-source {

  margin-top: 25px;

  display: flex;

  align-items: center;

  gap: 8px;

  color: #555;

  font-family: 'DM Mono', monospace;

  font-size: 8px;

  letter-spacing: 1px;

}


.source-dot {

  width: 5px;

  height: 5px;

  background: #ff3838;

  border-radius: 50%;

}


/* =========================================
   MOBILE
========================================= */

@media (max-width: 650px) {

  .movie-info h3 {

    font-size: 13px;

  }


  .movie-overlay {

    display: none;

  }


  /* Modal */

  .details-backdrop {

    padding: 15px;

  }


  .details-modal {

    max-height: 92vh;

    overflow-y: auto;

  }


  .details-backdrop-image {

    height: 220px;

  }


  .details-content {

    padding:
      130px
      25px
      30px;

    display: block;

  }


  .details-poster {

    width: 130px;

    margin-bottom: 25px;

  }


  .details-text h2 {

    font-size: 36px;

  }

}
</style>