<template>
  <v-container>
    <v-row>
      <v-col v-for="radio in radios" :key="radio.id" cols="12" sm="6" md="3" lg="3">
        <v-card class="card" max-height="200px" min-height="200px" flat tile
          @mouseenter="mostraComandi(radio)" @mouseleave="mostraComandi(radio)">
          <div class="card-content">
            <v-card-title>{{ radio.name }}</v-card-title>
            <v-img :src="radio.favicon || defaultImage" class="card-image" :alt="radio.name" />
          </div>
          <div v-if="radio.showControls" class="controls">
            <v-btn @click="cambiaRiproduzione(radio)" class="play-button" variant="text" size="small">
              <v-icon :size="40" class="Icon">{{ radio.playing ? 'mdi-stop' : 'mdi-play' }}</v-icon>
            </v-btn>
            <div @click="cambiaPreferito(radio)" class="heart-container">
              <div :class="['heart', { 'liked': radio.favorite }]"></div>
            </div>
          </div>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import Hls from 'hls.js';
import defaultImage from '../assets/imagenotfound.webp';

export default {
  name: 'HomeView',
  data() {
    return {
      radios: [],
      defaultImage,
    }
  },
  methods: {
    async callAPI() {
      const response = await fetch('https://nl1.api.radio-browser.info/json/stations/search?limit=100&countrycode=IT&hidebroken=true&order=clickcount&reverse=true');
      const data = await response.json();
      return data.filter(radio => radio.countrycode === 'IT').map(radio => ({
        ...radio,
        favorite: false,
        showControls: true,
        playing: false,
        audioPlayer: new Audio(),
      }));
    },
    async getRadios() {
      try {
        this.radios = await this.callAPI();
        const preferiti = JSON.parse(localStorage.getItem('preferiti')) || [];
        this.radios.forEach(radio => {
          const fav = preferiti.find(f => f.changeuuid === radio.changeuuid);
          radio.favorite = fav ? fav.favorite : false;
        });
      } catch (error) {
        console.error('Errore durante il recupero delle radio:', error);
      }
    },
    mostraComandi(radio) {
      radio.showControls = true;
    },
    cambiaRiproduzione(radio) {
      if (radio.playing) {
        this.fermaSingola(radio);
      } else {
        this.interrompiTutto();
        const audioUrl = radio.url_resolved || radio.url;
        if (audioUrl.includes('m3u8')) {
          if (Hls.isSupported()) {
            const hls = new Hls();
            hls.loadSource(audioUrl);
            hls.attachMedia(radio.audioPlayer);
          } else {
            console.error('HLS non è supportato in questo browser.');
          }
        } else {
          radio.audioPlayer.src = audioUrl;
        }
        radio.audioPlayer.play()
          .catch(error => {
            console.error('Errore durante la riproduzione audio:', error);
            if (error.name === 'NotAllowedError') {
              console.error('Assicurati che la riproduzione audio sia consentita nelle impostazioni del tuo browser.');
            }
          });
        radio.playing = true;
      }
    },
    fermaSingola(radio) {
      radio.audioPlayer.pause();
      radio.playing = false;
    },
    interrompiTutto() {
      this.radios.forEach(radio => {
        if (radio.playing) {
          this.fermaSingola(radio);
        }
      });
    },
    cambiaPreferito(radio) {
      radio.favorite = !radio.favorite;
      if (!radio.favorite) {
        let preferiti = JSON.parse(localStorage.getItem('preferiti')) || [];
        preferiti = this.radios.filter(radio => radio.favorite)
        console.log(JSON.stringify(preferiti));
        localStorage.setItem('preferiti', JSON.stringify(preferiti));
      } else {
        this.salvaPreferito();
      }
    },
    salvaPreferito() {
      let preferiti = JSON.parse(localStorage.getItem('preferiti')) || [];
      preferiti = this.radios.filter(radio => radio.favorite)
      localStorage.setItem('preferiti', JSON.stringify(preferiti));
      console.log(localStorage.getItem('preferiti'));
    },
  },
  created() {
    this.getRadios();
    const preferiti = JSON.parse(localStorage.getItem('preferiti')) || [];
    this.radios.forEach(radio => {
      const fav = preferiti.find(f => f.changeuuid === radio.changeuuid);
      radio.favorite = fav ? fav.favorite : false;
    });
  },
}
</script>

<style scoped>
.v-container {
  background-color: #0a0a34;
  max-width: 12000px;
  border-radius: 15px;
  padding: 20px;
}

.card {
  max-width: 400px;
  box-shadow: 0 4px 8px 0 rgba(3, 3, 3, 0.2);
  transition: transform 0.3s;
  border-radius: 25px;
  position: relative;
}

.card:hover {
  box-shadow: 0 8px 16px 0 rgb(255, 255, 255);
}

.card-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  
}

.card-image {
  width: 100px;
  height: 100px;
  object-fit: cover;
  border-radius: 10px;
}

.v-card-title {
  color: #333;
  font-size: 1.2em;
  font-weight: bold;
}

.controls {
  position: absolute;
  bottom: 10px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  align-items: center;
  justify-content: space-between; /* Added */
  width: 100%; /* Added */
}

.play-button {
  margin-right: 20px; /* Increased */
}

.heart-container {
  display: inline-block;
  cursor: pointer;
  margin-left: 5px;
}

.Icon {
  width: 40px !important;
  height: 40px !important;
}

.heart {
  width: 40px;
  height: 40px;
  margin-left: -34px;
  background: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z" fill="%23C1C1C1"/></svg>') center no-repeat;
  background-size: 100%;
}

.heart.liked {
  background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path d="M16.5 3c-1.74 0-3.41.81-4.5 2.09C10.91 3.81 9.24 3 7.5 3 4.42 3 2 5.42 2 8.5c0 3.78 3.4 6.86 8.55 11.54L12 21.35l1.45-1.32C18.6 15.36 22 12.28 22 8.5 22 5.42 19.58 3 16.5 3z" fill="%23FF0000"/></svg>');
}

@media (max-width: 600px) {
  .card {
    max-width: 100%;
  }
}
</style>
