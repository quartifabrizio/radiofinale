<template>
    <div ref="container"></div>
    <div class="navbar" v-if="selectedRadio">
        <img :src="selectedRadio.favicon || defaultImage" class="radio-logo" alt="Radio logo">
        <h4>{{ selectedRadioName }}</h4>
        <button @click="togglePlayPause(selectedRadio)">{{ isRadioPlaying ? 'Pause' : 'Play' }}</button>
        <div v-if="selectedRadio.playing" class="sound-wave">
            <div class="bar"></div>
            <div class="bar"></div>
            <div class="bar"></div>
            <div class="bar"></div>
        </div>
        <div @click="toggleFavorite(selectedRadio)" class="heart-container">
            <div :class="['heart', { 'liked': selectedRadio.favorite }]"></div>
        </div>
    </div>
</template>

<script>
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import Hls from 'hls.js';
import defaultImage from '../assets/logo.png';
import earthTexture from '../assets/2k_earth_daymap.jpg';


export default {
    name: 'ThreeJsScene',
    data() {
        return {
            camera: null,
            renderer: null,
            controls: null,
            earthRadius: 1,
            radios: [],
            selectedRadio: null,
            selectedRadioName: '',
            isRadioPlaying: false,
            defaultImage,
        };
    },
    mounted() {
        this.init();
        this.animate();
        this.getRadios();
        window.addEventListener('resize', this.handleWindowResize);
        this.raycaster = new THREE.Raycaster();
        this.mouse = new THREE.Vector2();
        window.addEventListener('click', this.onDocumentMouseClick);
    },
    beforeUnmount() {
        window.removeEventListener('resize', this.handleWindowResize);
        window.removeEventListener('click', this.onDocumentMouseClick);
    },
    methods: {
        init() {
            this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            this.camera.position.z = 5;
            this.renderer = new THREE.WebGLRenderer();
            this.renderer.setSize(window.innerWidth, window.innerHeight);
            this.$refs.container.appendChild(this.renderer.domElement);
            this.controls = new OrbitControls(this.camera, this.renderer.domElement);
            this.controls.enableDamping = true;

            const scene = new THREE.Scene();
            const geometry = new THREE.SphereGeometry(this.earthRadius, 64, 64);
            const material = new THREE.MeshPhongMaterial({ map: new THREE.TextureLoader().load(earthTexture) });
            const earth = new THREE.Mesh(geometry, material);
            scene.add(earth);

            const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
            scene.add(ambientLight);
            const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
            directionalLight.position.set(1, 1, 1).normalize();
            scene.add(directionalLight);

            this.scene = scene;
        },

        onDocumentMouseClick(event) {
            event.preventDefault();
            this.mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
            this.mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
            this.raycaster.setFromCamera(this.mouse, this.camera);
            const intersects = this.raycaster.intersectObjects(this.scene.children);

            if (intersects.length > 0) {
                this.handleMarkerClick(intersects[0]);
            }
        },

        handleMarkerClick(intersectedObject) {
            if (intersectedObject.object.onClick) {
                intersectedObject.object.onClick();
                const { latitude, longitude } = intersectedObject.object.userData;
                const clickedRadio = this.radios.find(radio => radio.geo_lat === latitude && radio.geo_long === longitude);
                if (clickedRadio) {
                    this.selectedRadio = clickedRadio;
                    this.selectedRadioName = clickedRadio.name;
                    this.isRadioPlaying = clickedRadio.playing;
                } else {
                    console.error('Radio non trovata per le coordinate cliccate');
                }
            }
        },

        addMarker(longitude, latitude, radio) {
            if (longitude !== null && latitude !== null) {
                const phi = (90 - latitude) * (Math.PI / 180);
                const theta = (longitude + 180) * (Math.PI / 180);
                const x = -this.earthRadius * Math.sin(phi) * Math.cos(theta);
                const y = this.earthRadius * Math.cos(phi);
                const z = this.earthRadius * Math.sin(phi) * Math.sin(theta);

                const geometry = new THREE.SphereGeometry(0.01, 32, 32);
                const material = new THREE.MeshBasicMaterial({ color: 0xFF0000 });
                const marker = new THREE.Mesh(geometry, material);
                marker.position.set(x, y, z);
                marker.userData = { latitude, longitude };
                marker.onClick = () => {
                    console.log('Clicked on marker:', marker.userData);
                };
                this.scene.add(marker);

                radio.marker = marker;

                const invisibleSphereGeometry = new THREE.SphereGeometry(0.4, 32, 32);
                const invisibleSphereMaterial = new THREE.MeshBasicMaterial({ visible: false });
                const invisibleSphere = new THREE.Mesh(invisibleSphereGeometry, invisibleSphereMaterial);
                invisibleSphere.position.set(x, y, z);
                invisibleSphere.userData = radio;
                invisibleSphere.onClick = marker.onClick;
                this.scene.add(invisibleSphere);
            } else {
                console.error('Longitude or latitude is null. Skipping marker addition.');
            }
        },

        animate() {
            requestAnimationFrame(this.animate);

            if (this.controls) {
                this.controls.update();
            }

            if (this.renderer && this.scene && this.camera) {
                this.renderer.render(this.scene, this.camera);
            }
        },

        handleWindowResize() {
            this.camera.aspect = window.innerWidth / window.innerHeight;
            this.camera.updateProjectionMatrix();
            this.renderer.setSize(window.innerWidth, window.innerHeight);
        },

        async fetchRadios() {
            const response = await fetch('https://de1.api.radio-browser.info/json/stations/search?limit=100&hidebroken=true&has_geo_info=true&order=clickcount&reverse=true');
            const data = await response.json();
            return data;
        },

        async getRadios() {
    try {
        this.radios = await this.fetchRadios();
        this.radios.forEach(radio => {
            if (radio.geo_lat !== null && radio.geo_long !== null) {
                console.log(`Radio con coordinate - Latitudine: ${radio.geo_lat}, Longitudine: ${radio.geo_long}`);
                this.addMarker(radio.geo_long, radio.geo_lat, radio);
                radio.playing = false;
                radio.audioPlayer = new Audio();
            }
        });
        this.retrieveFavorites();
    } catch (error) {
        console.error('Error fetching radios:', error);
    }
},


        togglePlayPause(radio) {
            if (radio.playing) {
                this.pauseRadio(radio);
            } else {
                this.pauseAllRadios();
                this.playRadio(radio);
            }
        },

        playRadio(radio) {
            const audioUrl = radio.url_resolved || radio.url;
            if (audioUrl.includes('m3u8')) {
                if (Hls.isSupported()) {
                    const hls = new Hls();
                    hls.loadSource(audioUrl);
                    hls.attachMedia(radio.audioPlayer);
                } else {
                    console.error('HLS is not supported in this browser.');
                }
            } else {
                radio.audioPlayer.src = audioUrl;
            }
            radio.audioPlayer.play()
                .then(() => {
                    radio.playing = true;
                    this.isRadioPlaying = true;
                })
                .catch(error => {
                    console.error('Error playing audio:', error);
                    if (error.name === 'NotAllowedError') {
                        console.error('Please ensure that the audio playback is allowed in your browser settings.');
                    }
                });
        },

        pauseRadio(radio) {
            radio.audioPlayer.pause();
            radio.playing = false;
            this.isRadioPlaying = false;
        },

        pauseAllRadios() {
            this.radios.forEach(radio => {
                if (radio.playing) {
                    this.pauseRadio(radio);
                }
            });
        },

        toggleFavorite(radio) {
            radio.favorite = !radio.favorite;
            // Aggiungi qui la logica per salvare il preferito sul server o in locale
        }
    }
};
</script>

<style scoped>
.navbar {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100%;
    background-color: #fff;
    box-shadow: 0 -2px 5px rgba(0, 0, 0, 0.3);
    display: flex;
    align-items: center;
    justify-content: space-around;
    padding: 10px;
}

.radio-logo {
    width: 50px;
    height: 50px;
    border-radius: 25px;
}

.sound-wave {
    display: flex;
    align-items: center;
    height: 20px;
    margin-left: 10px;
    margin-top: 2px;
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
        transform: scaleY(1);
    }

    100% {
        transform: scaleY(1.5);
    }
}
</style>
