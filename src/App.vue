<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
import AppMenu from '@/components/AppMenu.vue'

const copyrightYear = new Date().getFullYear()
</script>

<template>
  <header>
    <!--<img alt="Vue logo" class="logo" src="@/assets/logo.svg" width="125" height="125" />-->

    <div class="wrapper">
      <AppMenu />

      <nav>
        <RouterLink to="/">Play</RouterLink>
        <RouterLink to="/about">About Me</RouterLink>
      </nav>
    </div>
  </header>

  <!-- Display the actual page contents -->
  <div class="main-panel-wrapper">
    <RouterView />
  </div>
  <footer>
    <div class="copyright-text">&copy; {{ copyrightYear }} <a href="mailto:contact@stuartpringle.ca">stuart pringle</a></div>
  </footer>
</template>

<style>
@import '@/assets/base.css';

header {

  line-height: 1.5;
  grid-area: sidebar;
}

footer {
  padding-top: 30px;
  display: block;
  text-align: center;
  grid-area: footer;
}

.main-panel-wrapper {
  grid-area: main-app;
}

#app {
  max-width: 1280px;
  margin: 0 auto;
  padding: 2rem;

  font-weight: normal;

  grid-template-columns: 33% 33% 33%;
  grid-template-rows: auto;
  grid-template-areas:
  "header header header"
  "sidebar main-app main-app"
  "footer footer footer";

}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

a,
.green {
  text-decoration: none;
  color: hsla(160, 100%, 37%, 1);
  transition: 0.4s;
}

@media (hover: hover) {
  a:hover {
    background-color: hsla(160, 100%, 37%, 0.2);
  }
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (max-width: 1023px) {
  .main-panel-wrapper,
  footer,
  header {
    width: 100%;
  }
  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
    width: 100%;
  }
  body #app {
    grid-template-columns: 100%;
    grid-template-rows: auto;
    grid-template-areas:
    "header"
    "sidebar"
    "main-app"
    "footer";
  }
}

@media (min-width: 1024px) {
  body {
    display: flex;
    place-items: center;
  }

  body #app {
    display: grid;
    grid-template-columns: 1fr 1fr;
    padding: 0 2rem;
  }

  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
    max-height: 100vh;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }

  footer {
    min-height: 60px;
  }
}
</style>
