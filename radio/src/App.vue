<template>
  <nav>
    <div class="left-section">
      <div class="logo">
        <img src="@/assets/logo.jpeg" alt="Logoo" />
      </div>
      <div class="title">
        SotoRadio
      </div>
    </div>
    <div class="right-section">
      <router-link to="/">Home</router-link> |
      <router-link to="/about">Mondo</router-link> |
      <router-link to="/contatti">Preferiti</router-link>
    </div>
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
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 30px;
  background-color: #0a0a34; /* Dark blue color */
}

nav .left-section {
  display: flex;
  align-items: center;
}

nav .title {
  font-weight: bold;
  color: #ffffff; /* White color */
  margin-left: 10px; /* Spazio tra il titolo e il logo */
}

nav .logo img {
  width: 40px; /* Dimensione desiderata del logo */
  height: 40px;
  border-radius: 25px; /* Dimensione desiderata del logo */
  /* Aggiungi altri stili come border-radius, box-shadow, ecc. se necessario */
}

nav a {
  font-weight: bold;
  color: #ffffff; /* White color */
}

nav a.router-link-exact-active {
  color: #130ce5;
}
</style>
