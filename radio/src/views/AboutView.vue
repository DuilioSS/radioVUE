<template>
  <v-container>
    <div ref="container">

    </div>
  </v-container>
</template>

<script>

import * as THREE from 'three';
import mondoJpg from '../assets/mondo.jpg';

export default {
  mounted() {
    this.init();
  },
  data() {
    return {
      radios: [],
    }
  },
  methods: {
    async fetchRadios() {
      const response = await fetch('https://nl1.api.radio-browser.info/json/stations/search?limit=100&countrycode=IT&hidebroken=true&order=clickcount&reverse=true&has_geo_info=true');
      const data = await response.json();
      return data.filter(radio => radio.countrycode === 'IT').map(radio => ({
        ...radio,
        favorite: false,
        showControls: false,
        playing: false,
        audioPlayer: new Audio(),
      }));
    },
    retrieveFavorites() {
      const favorites = JSON.parse(localStorage.getItem('favorites')) || [];
      this.radios.forEach(radio => {
        const fav = favorites.find(f => f.changeuuid === radio.changeuuid);
        radio.favorite = fav ? fav.favorite : false;
      });
    },
    async getRadios() {
      try {
        this.radios = await this.fetchRadios();
        //this.retrieveFavorites();
      } catch (error) {
        console.error('Error fetching radios:', error);
      }
    },
    toRadians(degrees) {
      return degrees * Math.PI / 180;
    },
    // Conversione delle coordinate latitudine e longitudine in coordinate spaziali
    latLongToVector3(lat, lon, radius) {
      const phi = (90 - lat) * Math.PI / 180;
      const theta = (lon + 180) * Math.PI / 180;
      const x = -radius * Math.sin(phi) * Math.cos(theta);
      const y = radius * Math.cos(phi);
      const z = radius * Math.sin(phi) * Math.sin(theta);
      return new THREE.Vector3(x, y, z);
    },
    onMouseMove() {
      const deltaMove = {
        x: (event.offsetX - this.previousMousePosition.x) * this.sensitivity,
        y: (event.offsetY - this.previousMousePosition.y) * this.sensitivity
      };

      if (this.isDragging) {
        const deltaRotationQuaternion = new THREE.Quaternion()
          .setFromEuler(new THREE.Euler(
            this.toRadians(deltaMove.y * 0.5),
            this.toRadians(deltaMove.x * 0.5),
            0,
            'XYZ'
          ));

        this.sphere.quaternion.multiplyQuaternions(deltaRotationQuaternion, this.sphere.quaternion);
      }

      this.previousMousePosition = {
        x: event.offsetX,
        y: event.offsetY
      };
    },
    onMouseDown() {
      this.isDragging = true;
    },
    onMouseUp() {
      this.isDragging = false;
    },
    onMouseWheel() {
      const newCameraDistance = this.camera.position.z + event.deltaY * 0.1;
      if (newCameraDistance >= this.minCameraDistance && newCameraDistance <= this.maxCameraDistance) {
        this.camera.position.z = newCameraDistance;
      } else if (newCameraDistance < this.minCameraDistance) {
        this.camera.position.z = this.minCameraDistance;
      } else {
        this.camera.position.z = this.maxCameraDistance;
      }
    },

    init() {
      // Setup
      this.scene = new THREE.Scene();
      this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
      this.renderer = new THREE.WebGLRenderer();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.$refs.container.appendChild(this.renderer.domElement);

      // Texture
      const textureLoader = new THREE.TextureLoader();
      const texture = textureLoader.load(mondoJpg);

      // Sphere
      const radius = 5;
      const geometry = new THREE.SphereGeometry(radius, 32, 32);
      const material = new THREE.MeshBasicMaterial({ map: texture });
      this.sphere = new THREE.Mesh(geometry, material);
      this.scene.add(this.sphere);

      // Camera
      this.camera.position.z = 10;
      this.minCameraDistance = 6;
      this.maxCameraDistance = 13;

      // Lighting
      const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
      this.scene.add(ambientLight);

      const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
      directionalLight.position.set(0, 1, 1).normalize();
      this.scene.add(directionalLight);

      // Mouse control
      this.mouse = new THREE.Vector2();
      this.raycaster = new THREE.Raycaster();
      this.plane = new THREE.Plane(new THREE.Vector3(0, 0, 1), 0);
      this.intersection = new THREE.Vector3();

      this.isDragging = false;
      this.previousMousePosition = {
        x: 0,
        y: 0
      };

      this.sensitivity = 0.2; // Sensibilità del mouse

      // Creazione del punto rosso per New York
      const newYorkLat = 45.716667;
      const newYorkLon = 9.816667;
      const newYorkPosition = this.latLongToVector3(newYorkLat, newYorkLon, radius); // Il raggio della sfera è 5
      const geometryPoint = new THREE.SphereGeometry(0.04, 32, 32);
      const materialPoint = new THREE.MeshBasicMaterial({ color: 0xff0000 });
      const point = new THREE.Mesh(geometryPoint, materialPoint);
      this.sphere.add(point); // Aggiungi il punto come figlio della sfera
      point.position.copy(newYorkPosition);

      window.addEventListener('mousemove', this.onMouseMove, false);
      window.addEventListener('mousedown', this.onMouseDown, false);
      window.addEventListener('mouseup', this.onMouseUp, false);
      window.addEventListener('wheel', this.onMouseWheel, false);

      // Render loop
      this.animate();
    },
    animate() {
      requestAnimationFrame(this.animate);
      this.renderer.render(this.scene, this.camera);
    },
    
  created() {
    this.getRadios();
    this.retrieveFavorites();
  },
  }
}
</script>



<style scoped>
body {
  margin: 0;
  overflow: hidden;
}

canvas {
  display: block;
}
</style>