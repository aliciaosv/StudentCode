<script setup>
// import { useRoute } from 'vue-router'

// const route = useRoute()
</script>

<template>
  <nav class="navbar navbar-expand-lg navbar-light">
    <div class="container-xxl">
      <!-- toggle-knapp för mobile-view -->
      <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#main-nav"
        aria-controls="main-nav" aria-expanded="false" aria-label="Toggle Navigation">
        <span class="navbar-toggler-icon" />
      </button>

      <!-- navbarlänkar -->
      <div class="collapse navbar-collapse justify-content-lg-center align-center" id="main-nav">
        <ul class="navbar-nav">
          <li class="nav-item">
            <router-link to="/" class="nav-link" :class="{ active: $route.path === '/' }">Home</router-link>
          </li>
          <li class="nav-item">
            <router-link to="/QuizPage" class="nav-link"
              :class="{ active: $route.path.startsWith('/Quiz') }">Quizzes</router-link>
          </li>
          <li class="nav-item">
            <router-link to="/MyResults" class="nav-link" :class="{ active: $route.path === '/MyResults' }">My
              Results</router-link>
          </li>
          <li class="nav-item">
            <router-link to="/About" class="nav-link" :class="{ active: $route.path === '/About' }">About
              Us</router-link>
          </li>
        </ul>
      </div>
    </div>
  </nav>
</template>

<style scoped>
.nav-link {
  padding: 10px;
  margin: 5px;
  height: 40px;
}

.active {
  background-color: #f5eddf;
  color: black;
  border-radius: 2px;
  transition: background-color 0.3s ease;
}
</style>
