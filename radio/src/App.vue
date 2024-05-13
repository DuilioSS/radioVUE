<template>
  <nav>
    
    <router-link to="/">Home</router-link> |
    <router-link to="/about">Mondo</router-link> |
    <router-link to="/contatti">Preferiti</router-link>
  </nav>
  <router-view/>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

nav {
  padding: 30px;
  background-color: #0a0a34; /* Dark blue color */
}

nav a {
  font-weight: bold;
  color: #ffffff; /* White color */
}

nav a.router-link-exact-active {
  color: #130ce5;
}
</style>
