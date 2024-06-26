<template class="tm">
    <div ref="container"></div>
    <div id="selectedRadioInfo" class="selected-radio-info" v-if="selectedRadio">
        <img :src="getRadioImage(selectedRadio)" class="radio-logo" alt="Radio logo">
        <h4>{{ selectedRadio.name }}</h4>
        <h3>{{ selectedRadio.country }}</h3>
        <v-btn :icon="selectedRadio.playing ? 'mdi-stop' : 'mdi-play'" @click="togglePlayPause(selectedRadio)"></v-btn>
    </div>
</template>

<script>
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import earthTexture from '../assets/textureTerra.jpg';
import Hls from 'hls.js';
import defaultImage from '../assets/logo.png';

export default {
    name: 'WorldView',
    data() {
        return {
            camera: null,
            renderer: null,
            controls: null,
            earthRadius: 1,
            radio: [],
            selectedRadio: null,
            defaultImage,
            isFavorite: false,
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
            this.camera.position.z = 2;

            this.renderer = new THREE.WebGLRenderer();
            const width = window.innerWidth ; // Imposta la larghezza al 80% della finestra del browser
            const height = window.innerHeight; // Imposta l'altezza al 60% della finestra del browser
            this.renderer.setSize(width, height);
            this.$refs.container.appendChild(this.renderer.domElement);

            this.controls = new OrbitControls(this.camera, this.renderer.domElement);
            this.controls.enableDamping = true;

            const scene = new THREE.Scene();

            const geometry = new THREE.SphereGeometry(this.earthRadius, 64, 64); // Increase segments for smoother surface
            const texture = new THREE.TextureLoader().load(earthTexture);
            const material = new THREE.MeshPhongMaterial({ map: texture });
            const earth = new THREE.Mesh(geometry, material);
            scene.add(earth);

            const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
            scene.add(ambientLight);

            const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
            directionalLight.position.set(1, 1, 1).normalize();
            scene.add(directionalLight);

            this.scene = scene;

            this.raycaster = new THREE.Raycaster();
            this.raycaster.params.Points.threshold = 0.05;
        },
        onDocumentMouseClick(event) {
            event.preventDefault();
            this.mouse.x = (event.offsetX / this.renderer.domElement.clientWidth) * 2 - 1;
            this.mouse.y = -(event.offsetY / this.renderer.domElement.clientHeight) * 2 + 1;
            this.raycaster.setFromCamera(this.mouse, this.camera);
            const intersects = this.raycaster.intersectObjects(this.scene.children);

            if (intersects.length > 0) {
                console.log('Clicked on:', intersects[0].object.userData);
                this.handleMarkerClick(intersects[0]);
            }
        },
        handleMarkerClick(intersectedObject) {
            if (intersectedObject.object.onClick) {
                intersectedObject.object.onClick();
                const previousRadio = this.selectedRadio;
                this.selectedRadio = intersectedObject.object.userData;
                this.$nextTick(() => {
                    if (this.selectedRadio.url_resolved || this.selectedRadio.url) {
                        this.lazyLoadAudio(this.selectedRadio);
                        if (previousRadio && previousRadio !== this.selectedRadio) {
                            this.pauseRadio(previousRadio);
                        }
                    }
                });
            }
        },

        lazyLoadAudio(radio) {
            console.log('Lazy loading audio for:', radio);
            if (!radio.audioPlayer) {
                console.log('Creating new audio player');
                radio.audioPlayer = new Audio(radio.url_resolved || radio.url);
                console.log('Audio player:', radio.audioPlayer);
                radio.audioPlayer.onloadeddata = () => {
                    console.log('Audio loaded');
                };
                radio.audioPlayer.onerror = () => {
                    console.error('Error loading audio');
                };
            }
            else {
                console.log('Audio player already exists');
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
                const material = new THREE.MeshBasicMaterial({ color: 0x77a345 });
                const marker = new THREE.Mesh(geometry, material);
                marker.position.set(x, y, z);
                marker.userData = radio;
                marker.onClick = () => {
                    console.log('Clicked on marker:', marker.userData);
                };
                this.scene.add(marker);

                const invisibleSphereGeometry = new THREE.SphereGeometry(0.015, 32, 32);
                const invisibleSphereMaterial = new THREE.MeshBasicMaterial({ visible: false });
                const invisibleSphere = new THREE.Mesh(invisibleSphereGeometry, invisibleSphereMaterial);
                invisibleSphere.position.set(x, y, z);
                invisibleSphere.userData = radio;
                invisibleSphere.onClick = marker.onClick;
                this.scene.add(invisibleSphere);
            } else {
                console.error('Longitude or latitude is null');
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
            const response = await fetch('https://de1.api.radio-browser.info/json/stations/search?limit=400&hidebroken=true&has_geo_info=true&order=clickcount&reverse=true');// https://nl1.api.radio-browser.info/json/stations/search?limit=100&has_geo_info=true&countrycode=IT&hidebroken=true&order=clickcount&reverse=true
            const data = await response.json();
            return data;
        },
        async getRadios() {
            try {
                this.radio = await this.fetchRadios();
                this.radio.forEach(radio => {
                    this.addMarker(radio.geo_long, radio.geo_lat, radio);
                    radio.playing = false;
                    radio.audioPlayer = new Audio();
                });
               
            } catch (error) {
                console.error('Error fetching radios:', error);
            }
        },
        togglePlayPause(radio) {
            if (radio.playing) {
                this.pauseRadio(radio);
            } else {
                this.pauseAllRadios(); // Pause all other radios
                this.playRadio(radio);
            }
        },
        playRadio(radio) {
            console.log('playRadio called', radio);
            const audioUrl = radio.url_resolved || radio.url;
            console.log('Audio URL:', audioUrl);
            if (audioUrl.includes('m3u8')) {
                if (Hls.isSupported()) {
                    console.log('HLS is supported');
                    const hls = new Hls();
                    hls.loadSource(audioUrl);
                    hls.attachMedia(radio.audioPlayer);
                } else {
                    console.error('HLS is not supported in this browser.');
                }
            } else {
                console.log('Setting audio source:', audioUrl);
                radio.audioPlayer.src = audioUrl;
            }
            radio.audioPlayer.play()
                .then(() => {
                    console.log('Audio started playing');
                    radio.playing = true;
                })
                .catch(error => {
                    console.error('Error playing audio:', error);
                    if (error.name === 'NotAllowedError') {
                        console.error('Please ensure that the audio playback is allowed in your browser settings.');
                    }
                });
        },
        getRadioImage(radio) {
            if (radio.playing) {
                return 'https://i.pinimg.com/originals/a5/5a/68/a55a685b8375807667122027d72de120.gif';
            } else {
                return radio.favicon ? radio.favicon : "https://img.freepik.com/vetores-premium/icone-ou-simbolo-vintage-da-estacao-de-radio-online_8071-25787.jpg";
            }
        },
        pauseRadio(radio) {
            radio.audioPlayer.pause();
            radio.playing = false;
        },
        pauseAllRadios() {
            this.radio.forEach(radio => {
                if (radio.playing) {
                    this.pauseRadio(radio);
                }
            });
        },
    },
    created() {
        this.getRadios();
    }
}
</script>

<style scoped>
.selected-radio-info {
    position: fixed;
    top: 20px;
    left: 20px;
    background-color: rgba(255, 255, 255, 0.8);
    padding: 10px;
    border-radius: 5px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.3);
}
.radio-logo {
    width: 60px;
    height: 60px;
    border-radius: 30px;
}
</style>
