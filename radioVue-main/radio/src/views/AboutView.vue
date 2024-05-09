<template>
  <div ref="canvasContainer" class="about"></div>
</template>

<script>
import * as THREE from 'three';
export default {
  mounted() {
    // Inizializzazione della scena, della camera e del renderer
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);
    this.$refs.canvasContainer.appendChild(renderer.domElement);

    // Creazione della Terra
    const earthGeometry = new THREE.SphereGeometry(10, 64, 64);
    const earthTexture = new THREE.TextureLoader().load('../assets/2k_earth_daymap.jpg');
    const earthMaterial = new THREE.MeshBasicMaterial({ map: earthTexture });
    const earth = new THREE.Mesh(earthGeometry, earthMaterial);
    scene.add(earth);

    // Array di posizioni radio (esempio)
    const radioPositions = [
      { latitude: 40.7128, longitude: -74.0060 }, // New York
      { latitude: 51.5074, longitude: -0.1278 },  // London
      // Aggiungi altre posizioni radio ottenute dalla tua richiesta API
    ];

    // Creazione dei pin per le radio
    const pinGeometry = new THREE.SphereGeometry(0.2, 32, 32);
    const pinMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000 });
    radioPositions.forEach(pos => {
      const lat = pos.latitude * (Math.PI / 180);
      const lon = -pos.longitude * (Math.PI / 180);
      const x = 10 * Math.cos(lat) * Math.cos(lon);
      const y = 10 * Math.sin(lat);
      const z = 10 * Math.cos(lat) * Math.sin(lon);

      const pin = new THREE.Mesh(pinGeometry, pinMaterial);
      pin.position.set(x, y, z);
      scene.add(pin);
    });

    // Posizionamento della camera
    camera.position.set(0, 0, 30);

    // Aggiunta di una luce
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
    scene.add(ambientLight);

    // Funzione di rendering per animare la scena
    const animate = function () {
      requestAnimationFrame(animate);

      // Rotazione della Terra
      earth.rotation.y += 0.001;

      renderer.render(scene, camera);
    };

    animate();
  }
};
</script>

<style>
  /* Stile facoltativo per il canvas */
  canvas {
    display: block;
    width: 100%;
    height: 100%;
  }
</style>
