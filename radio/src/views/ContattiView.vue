<template>
    <v-container>
     
      <v-row>
        <v-col v-for="radio in preferiti" :key="radio.id" cols="12" sm="6" md="3" lg="3">
          <v-card class="d-flex flex-row card" max-height="200px" min-height="200px" flat tile
            @mouseenter="mostraComandi(radio)" @mouseleave="mostraComandi(radio)">
            <div class="d-flex flex-column">
              <v-card-title>{{ radio.name }}</v-card-title>
              <v-img :src="radio.favicon || defaultImage" class="card-image" :alt="radio.name" />
            </div>
            <v-spacer></v-spacer>
            <div v-if="radio.showControls" class="controls">
              <v-btn @click="cambiaRiproduzione(radio)" class="ms-2" variant="text" size="small">
                <v-icon :size="40" class="Icon">{{ radio.playing ? "mdi-stop" : "mdi-play" }}</v-icon>
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
  import playImage from '../assets/play.png';
  import stopImage from '../assets/stop.png';
  
  export default {
    name: 'PaginaPrincipale',
    data() {
      return {
        icon: "mdi-play",
        preferiti: [],
        defaultImage,
        playImage,
        stopImage,
      }
    },
    methods: {
      mostraComandi(radio) {
        radio.showControls = true;
      },
      nascondiComandi(radio) {
        radio.showControls = false;
      },
      cambiaRiproduzione(radio) {
        if (radio.playing) {
          this.icon = "mdi-play";
          this.fermaSingola(radio);
        } else {
          this.icon = "mdi-stop";
          this.interrompiTutto(); // Interrompi tutte le altre radio
          const audioUrl = radio.url_resolved || radio.url;
          if (audioUrl.includes('m3u8')) {
            //file m3u8
            if (Hls.isSupported()) {
              const hls = new Hls();
              hls.loadSource(audioUrl);
              hls.attachMedia(radio.audioPlayer);
            } else {
              //mp3
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
        this.preferiti.forEach(radio => {
          if (radio.playing) {
            this.fermaSingola(radio);
          }
        });
      },
      cambiaPreferito(radio) {
        radio.favorite = !radio.favorite;
        if (!radio.favorite) {
          let preferiti = JSON.parse(localStorage.getItem('preferiti')) || [];
          preferiti = this.preferiti.filter(radio => radio.favorite)
          console.log(JSON.stringify(preferiti));
          localStorage.setItem('preferiti', JSON.stringify(preferiti));
        } else {
          this.salvaPreferito();
        }
      },
      salvaPreferito() {
        let preferiti = JSON.parse(localStorage.getItem('preferiti')) || [];
        preferiti = this.preferiti.filter(radio => radio.favorite)
        localStorage.setItem('preferiti', JSON.stringify(preferiti));
        console.log(localStorage.getItem('preferiti'));
      },
  
      refresh() {
        this.preferiti = JSON.parse(localStorage.getItem('preferiti')) || [];
        console.log(this.preferiti);
        this.preferiti.forEach(radio => {
          radio.favorite = true;
          radio.showControls = true;
          radio.playing = false;
          radio.audioPlayer = new Audio();
        });
        console.log(this.preferiti);
      }
    },
    created() {
      this.refresh();
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
  
  .image {
    width: 50px;
    height: 50px;
  }
  
  .ms-2 {
    margin-bottom: 12px;
    height: 40px;
  }
  
  .heart-container {
    display: inline-block;
    cursor: pointer;
    margin-left: 5px;
    margin-top: 100px;
  }
  
  .Icon {
    width: 40px !important;
    height: 40px !important;
  }
  
  .titleText {
    color: aliceblue;
  }
  .v-card{
  background-color: #c8c8cd;
  color: rgb(176, 179, 181);

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
  
  .bnt-play {
    background-color: Green;
    font-size: 40px;
    padding-bottom: 40px;
    margin-left: -40px;
    width: 100px;
  }
  
  h1 {
    color: #333;
    text-align: center;
    font-size: 2em;
    margin-bottom: 20px;
  }
  
  .v-row {
    justify-content: space-around;
  }
  
  .card {
    max-width: 400px;
    box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);
    transition: transform 0.3s;
    border-radius: 10px !important;
    position: relative;
  }
  
  .card:hover {
    box-shadow: 0 8px 16px 0 rgba(0, 0, 0, 1);
  }
  
  .card-image {
    width: 100px;
    height: 100px;
    margin-left: 10px;
    object-fit: cover;
    border-radius: 5px;
    margin-right: 0px;
  }
  
  
  .v-card-title {
    color: #333;
    font-size: 1.2em;
    font-weight: bold;
  }
  
  .controls {
    position: absolute;
    bottom: 10px;
    left: 160px;
    /* Changed from 'right' to 'left' */
    display: flex;
    align-items: center;
  }
  
  .controls .v-btn {
    margin-right: 50px;
    /* Changed from 'margin-left' to 'margin-right' */
    margin-top: 100px;
  }
  
  .sound-wave {
    display: flex;
    align-items: center;
    height: 20px;
    margin-left: 10px;
    /* Adjust this if necessary */
    margin-top: 100px;
    /* Added to push the sound wave down */
  }
  
  .bar {
    width: 4px;
    height: 100%;
    margin: 0 2px;
    background-color: #333;
    animation: pulse 0.8s infinite ease-in-out alternate;
  }
  
  .bar:nth-child(1) {
    animation-delay: 0s;
  }
  
  .bar:nth-child(2) {
    animation-delay: 0.1s;
  }
  
  .bar:nth-child(3) {
    animation-delay: 0.2s;
  }
  
  .bar:nth-child(4) {
    animation-delay: 0.3s;
  }
  
  @keyframes pulse {
    0% {
      transform: scaleY(0.1);
    }
  
    100% {
      transform: scaleY(1);
    }
  }
  </style>