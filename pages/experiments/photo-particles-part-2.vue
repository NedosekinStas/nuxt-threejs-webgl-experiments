<template>
  <section class="section">
    <div class="hero is-fullheight">
      <div 
        id="three-element" 
        ref="threeElement"/>
      <canvas 
        id="photo-canvas" 
        ref="photoCanvas"/>
      <img 
        ref="image" 
        src="/images/jon-snow-original.jpg">
      <button @click="startScene">Start</button>
    </div>
  </section>
</template>

<script>
import * as THREE from "three-full";

export default {
  components: {},
  data() {
    return {
      scene: undefined,
      camera: undefined,
      renderer: undefined,
      particleCount: 10000,
      particles: undefined,
      particleSystem: undefined,
      started: false,
      trackballControls: undefined,
      clock: undefined,
      imageData: undefined,
      ORBIT_RATE: 0.01,
      DENSITY: 7,
      CENTRE_MASS: 5,
      AGGRESSION: 20,
      WIDTH: undefined,
      HEIGHT: undefined,
      DEPTH: undefined
    };
  },
  mounted() {
    // set up remaining constants
    this.WIDTH = window.innerWidth;
    this.HEIGHT = window.innerHeight;
    this.DEPTH = Math.max(this.WIDTH, this.HEIGHT);

    console.log(THREE);

    // threejs page load setup
    this.setupCameraSceneRenderer();
    this.addLight();
    this.positionCamera();
    this.setupClock();
    this.setupTrackballControls();
    this.bindWindowEvents();
    this.$refs.threeElement.appendChild(this.renderer.domElement);
  },
  methods: {
    /* THREE scene setup functions */
    setupCameraSceneRenderer() {
      // set up scene, camera, renderer, and axes
      this.scene = new THREE.Scene();
      this.camera = new THREE.OrthographicCamera(
        window.innerWidth / -40,
        window.innerWidth / 40,
        window.innerHeight / 40,
        window.innerHeight / -40,
        1,
        100
      );
      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.setClearColor(new THREE.Color(0x000000));
      this.renderer.setSize(this.WIDTH, this.HEIGHT);
      this.renderer.shadowMap.enabled = true;
      this.renderer.shadowMapSoft = true;
      this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    },
    addLight() {
      // add spotlight and ambient light
      const spotLight = new THREE.DirectionalLight(0xffffff);
      spotLight.position.set(-40, 60, -10);
      spotLight.shadow.mapSize = new THREE.Vector2(1024, 1024);
      spotLight.shadow.camera.far = 100;
      spotLight.shadow.camera.near = 10;
      spotLight.castShadow = true;
      spotLight.decay = 2;
      spotLight.penumbra = 0.05;
      this.scene.add(spotLight);

      const ambientLight = new THREE.AmbientLight(0x3c3c3c);
      this.scene.add(ambientLight);
    },
    positionCamera() {
      // position the camera for the scene
      this.camera.position.set(10, 0, 0);
      this.camera.lookAt(this.scene.position);
    },
    setupClock() {
      this.clock = new THREE.Clock();
    },
    setupTrackballControls() {
      this.trackballControls = new THREE.TrackballControls(this.camera);
      this.trackballControls.rotateSpeed = 1.7;
    },
    /* textures, geometries, meshes */
    createParticles() {
      const material = new THREE.PointsMaterial({
        blending: THREE.AdditiveBlending,
        // map: ImageUtils.loadTexture("images/particle.png"),
        size: this.DENSITY * 1.5,
        opacity: 1,
        vertexColors: true,
        sizeAttenuation: true
      });
      const geometry = new THREE.Geometry();
      const step = this.DENSITY * 4;

      // go thru the pixels
      for (let x = 0; x < this.WIDTH; x += step) {
        for (let y = this.HEIGHT; y >= 0; y -= this.DENSITY) {
          const pixel = y * this.WIDTH * 4 + x;

          // grab actual pixel data, ignoring transparencies
          if (this.imageData[pixel + 3] > 0) {
            const pixelColor =
              (this.imageData.data[pixel] << 16) +
              (this.imageData.data[pixel + 1] << 8) +
              this.imageData.data[pixel + 2];
            const color = new THREE.Color(pixelColor);
            const vector = new THREE.Vector3(-300 + x / 4, 240 - y, 0);

            geometry.vertices.push(vector);
            geometry.colors.push(color);
          }
        }
      }

      const particleSystem = new THREE.Points(geometry, material);
      particleSystem.sortParticles = true;

      this.scene.add(particleSystem);
    },
    test(particles) {
      const canvasContext = this.$refs.photoCanvas.getContext("2d");
      canvasContext.clearRect(
        0,
        0,
        this.$refs.photoCanvas.width,
        this.$refs.photoCanvas.height
      );

      for (let i = 0; i < particles.length; i++) {
        const particle = particles[i];
        canvasContext.fillRect(particle.x, particle.y, 1, 1);
      }
    },
    /* start and render functions */
    getCanvasImageData() {
      const canvasContext = this.$refs.photoCanvas.getContext("2d");

      canvasContext.drawImage(
        this.$refs.image,
        0,
        0,
        this.$refs.photoCanvas.width,
        this.$refs.photoCanvas.height
      );

      this.imageData = canvasContext.getImageData(
        0,
        0,
        this.$refs.photoCanvas.width,
        this.$refs.photoCanvas.height
      );
    },
    startScene() {
      if (this.started) return;

      this.getCanvasImageData();
      this.createParticles();
      this.renderScene();

      this.started = !this.started;
    },
    renderScene() {
      this.trackballControls.update(this.clock.getDelta());
      this.renderer.render(this.scene, this.camera);
      window.RAF = requestAnimationFrame(this.renderScene);
    },
    /* events */
    bindWindowEvents() {
      // window resize
      window.addEventListener("resize", this.handleWindowResize);
      // rotation keys
      window.addEventListener("keypress", this.changeCameraPosition);
    },
    handleWindowResize() {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
    },
    changeCameraPosition(e) {
      const theta = 0.2; //the speed of rotation
      const x = this.camera.position.x;
      const y = this.camera.position.y;
      const z = this.camera.position.z;

      e.preventDefault();

      switch (e.key) {
        case "a": // left
          this.camera.position.x = x * Math.cos(theta) + z * Math.sin(theta);
          this.camera.position.z = z * Math.cos(theta) - x * Math.sin(theta);
          break;
        case "w": // up
          this.camera.position.y = y * Math.cos(theta) - z * Math.sin(theta);
          this.camera.position.z = z * Math.cos(theta) + y * Math.sin(theta);
          break;
        case "s": // down
          this.camera.position.y = y * Math.cos(theta) + z * Math.sin(theta);
          this.camera.position.z = z * Math.cos(theta) - y * Math.sin(theta);
          break;
        case "d": // right
          this.camera.position.x = x * Math.cos(theta) - z * Math.sin(theta);
          this.camera.position.z = z * Math.cos(theta) + x * Math.sin(theta);
          break;
        default:
          break;
      }

      this.camera.lookAt(this.scene.position);
    }
  }
};
</script>

<style scoped>
#photo-canvas,
img {
  position: absolute;
  top: 10px;
  left: 10px;
  height: 125px;
  width: 140px;
  background: white;
}

img {
  opacity: 0;
}

button {
  position: absolute;
  top: 10px;
  left: 0;
  right: 0;
  margin: auto;
  display: block;
  width: 80px;
  background: red;
  color: white;
  border-color: transparent;
  border-radius: 10px;
  outline: unset;
}
</style>