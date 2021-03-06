<template>
  <section class="section">
    <div class="hero is-fullheight">
      <div 
        id="three-element" 
        ref="threeElement" 
        @click.prevent="addCube"/>
      <button @click.prevent="startScene">Start</button>
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
      stats: undefined,
      plane: undefined,
      planeDimensions: [60, 40],
      objectCount: 0,
      started: false,
      step: 0,
      trackballControls: undefined,
      clock: undefined
    };
  },
  mounted() {
    this.setup();
    this.addPlane();
    this.addLight();
    this.positionCamera();
    this.setupClock();
    this.setupTrackballControls();
    this.$refs.threeElement.appendChild(this.renderer.domElement);
  },
  methods: {
    setup() {
      // set up scene, camera, renderer, and axes
      this.scene = new THREE.Scene();
      this.camera = new THREE.PerspectiveCamera(
        45,
        window.innerWidth / window.innerHeight,
        0.1,
        1000
      );
      this.renderer = new THREE.WebGLRenderer();
      this.renderer.setClearColor(new THREE.Color(0x000000));
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.renderer.shadowMap.enabled = true;
      this.renderer.shadowMapSoft = true;
      this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;

      window.addEventListener("resize", this.handleWindowResize);
    },
    handleWindowResize() {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
    },
    addPlane() {
      // create plane geometry and material
      const planeGeometry = new THREE.PlaneGeometry(
        this.planeDimensions[0],
        this.planeDimensions[1],
        1,
        1
      );
      const planeMaterial = new THREE.MeshBasicMaterial({ color: 0xaaaaaa });
      this.plane = new THREE.Mesh(planeGeometry, planeMaterial);
      this.plane.rotation.x = -0.5 * Math.PI;
      this.plane.position.set(15, 0, 0);
      this.plane.receiveShadow = true;

      // add plane to the scene
      this.scene.add(this.plane);
    },
    addCube() {
      const cubeSize = Math.ceil(Math.random() * 3);
      const cubeGeometry = new THREE.BoxGeometry(cubeSize, cubeSize, cubeSize);
      const cubeMaterial = new THREE.MeshLambertMaterial({
        color: Math.random() * 0xffffff
      });
      const cube = new THREE.Mesh(cubeGeometry, cubeMaterial);

      cube.castShadow = true;
      cube.name = `cube-${this.scene.children.length}`;
      cube.position.x =
        -30 + Math.round(Math.random() * this.planeDimensions[0]);
      cube.position.y = Math.round(Math.random() * 5);
      cube.position.z =
        -20 + Math.round(Math.random() * this.planeDimensions[1]);

      this.scene.add(cube);
      this.objectCount++;
    },
    updateGeometryRotation(shape, coords) {
      shape.rotation.x += coords[0];
      shape.rotation.y += coords[1];
      shape.rotation.z += coords[2];
    },
    addLight() {
      // add spotlight and ambient light
      const spotLight = new THREE.SpotLight(0xffffff);
      spotLight.position.set(-40, 60, -10);
      spotLight.shadow.mapSize = new THREE.Vector2(1024, 1024);
      spotLight.shadow.camera.far = 130;
      spotLight.shadow.camera.near = 40;
      spotLight.castShadow = true;
      spotLight.decay = 2;
      spotLight.penumbra = 0.05;
      this.scene.add(spotLight);

      const ambientLight = new THREE.AmbientLight(0x3c3c3c);
      this.scene.add(ambientLight);
    },
    positionCamera() {
      // position the camera for the scene
      this.camera.position.set(-30, 40, 30);
      this.camera.lookAt(this.scene.position);
    },
    setupClock() {
      this.clock = new THREE.Clock();
    },
    setupTrackballControls() {
      this.trackballControls = new THREE.TrackballControls(
        this.camera,
        this.renderer.domElement
      );
      this.trackballControls.rotateSpeed = 1.0;
      this.trackballControls.zoomSpeed = 1.2;
      this.trackballControls.panSpeed = 0.8;
      this.trackballControls.noZoom = false;
      this.trackballControls.noPan = false;
      this.trackballControls.staticMoving = false;
      this.trackballControls.dynamicDampingFactor = 0.3;
      this.trackballControls.keys = [65, 83, 68];
    },
    renderScene() {
      const rotationArray = [0.02, 0.02, 0.02];

      this.trackballControls.update(this.clock.getDelta());

      this.scene.traverse(obj => {
        if (obj instanceof THREE.Mesh && obj !== this.plane) {
          obj.rotation.x += 0.002;
          obj.rotation.z += 0.002;
        }
      });

      window.RAF = requestAnimationFrame(this.renderScene);
      this.renderer.render(this.scene, this.camera);
    },
    startScene() {
      if (this.started) return;

      this.renderScene();

      this.started = !this.started;
    }
  }
};
</script>

<style scoped>
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