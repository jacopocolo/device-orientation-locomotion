<template>
  <div>
    <button id="startButton">Start Demo</button>
    <div
      id="three"
      @touchstart="handleInput"
      @touchmove="handleInput"
      @touchend="handleInput"
      @mousedown="handleInput"
      @mousemove="handleInput"
      @mouseup="handleInput"
    ></div>
    <div id="info">
      <span id="x"></span><br />
      <span id="y"></span><br />
      <span id="z"></span>
    </div>
  </div>
</template>

<script>
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader.js";
import InfiniteGridHelper from "./components/InfiniteGridHelper.js";
import { DeviceOrientationControls } from "./components/DeviceOrientationControls.js";

const renderer = new THREE.WebGLRenderer({
  antialias: true,
});
const camera = new THREE.PerspectiveCamera(
  100,
  window.innerWidth / window.innerHeight,
  1,
  1000
);
export let scene, controls;
let grid, plane;
let previewPoint;

export default {
  name: "App",
  data() {
    return {
      position: new THREE.Vector3(),
      mouseDown: false,
      offset: 0,
    };
  },
  components: {},
  methods: {
    init: function () {
      let main = document.getElementById("three");
      renderer.setPixelRatio(window.devicePixelRatio);
      renderer.setClearColor(0x000000, 0); // the default
      renderer.setSize(window.innerWidth, window.innerHeight);
      main.appendChild(renderer.domElement);
      renderer.domElement.id = "threeJsCanvas";
      scene = new THREE.Scene();
      scene.background = new THREE.Color(0x222222);

      var hemiLight = new THREE.HemisphereLight(0xffffff, 0xffffff, 0.6);
      hemiLight.color.setHSL(0.6, 0.75, 0.5);
      hemiLight.groundColor.setHSL(0.095, 0.5, 0.5);
      hemiLight.position.set(0, 500, 0);
      scene.add(hemiLight);

      var dirLight = new THREE.DirectionalLight(0xffffff, 1);
      dirLight.position.set(-1, 0.75, 1);
      dirLight.position.multiplyScalar(50);
      dirLight.name = "dirlight";
      // dirLight.shadowCameraVisible = true;

      scene.add(dirLight);

      const geometry = new THREE.BoxGeometry(2, 5, 2);
      const materialGreen = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
      const cubeGreen = new THREE.Mesh(geometry, materialGreen);
      scene.add(cubeGreen);
      cubeGreen.position.set(10, 0, -10);

      const materialRed = new THREE.MeshBasicMaterial({ color: 0xff0000 });
      const cubeRed = new THREE.Mesh(geometry, materialRed);
      scene.add(cubeRed);
      cubeRed.position.set(0, 0, 20);

      const materialBlue = new THREE.MeshBasicMaterial({ color: 0x0000ff });
      const cubeBlue = new THREE.Mesh(geometry, materialBlue);
      scene.add(cubeBlue);
      cubeBlue.position.set(-10, 0, -10);

      const previewPointGeometry = new THREE.CircleGeometry(1, 16);
      const previewPointMaterial = new THREE.MeshBasicMaterial({
        color: 0xffff00,
      });
      previewPoint = new THREE.Mesh(previewPointGeometry, previewPointMaterial);
      previewPoint.rotation.x = -Math.PI / 2;
      scene.add(previewPoint);
      previewPoint.visible = false;

      const pg = new THREE.PlaneGeometry(10, 10);
      const pm = new THREE.MeshBasicMaterial({
        color: 0xffff00,
        side: THREE.DoubleSide,
        transparent: true,
        opacity: 0.1,
      });
      plane = new THREE.Mesh(pg, pm);
      plane.rotation.x = Math.PI / 2;
      scene.add(plane);

      const loader = new GLTFLoader();
      const dracoLoader = new DRACOLoader();
      dracoLoader.setDecoderConfig({ type: "js" });
      dracoLoader.setDecoderPath("https://www.gstatic.com/draco/v1/decoders/");
      loader.setDRACOLoader(dracoLoader);

      loader.load(
        // resource URL
        "/turbine.gltf",
        // called when the resource is loaded
        function (gltf) {
          console.log(gltf);
          /*for (let i = 0; i < 10; i++) {
            let model = gltf.scene.clone();
            scene.add(model);
            model.scale.set(10, 10, 10);
            model.position.y = 0;
            model.position.x = -i * THREE.MathUtils.randInt(30, 40);
            model.position.z = -i * THREE.MathUtils.randInt(30, 40);
            model.rotation.y = THREE.MathUtils.randFloat(0, Math.PI);
          }*/
        },
        // called while loading is progressing
        function (xhr) {
          console.log((xhr.loaded / xhr.total) * 100 + "% loaded");
        },
        // called when loading has errors
        function (error) {
          console.log(error);
        }
      );

      controls = new DeviceOrientationControls(camera);
      //controls.disconnect();
      controls.alphaOffset = Math.PI * 2;

      grid = new InfiniteGridHelper(1, 10, new THREE.Color(0x00ff00), 100);
      scene.add(grid);
      camera.position.set(0, 1.8, 5);
      plane.position.set(camera.position.x, 0, camera.position.z);
      window.addEventListener("resize", this.onWindowResize);
      window.addEventListener("orientationchange", this.onWindowResize);
      this.onWindowResize();
      renderer.render(scene, camera);
    },
    animate: function () {
      window.requestAnimationFrame(this.animate);
      controls.update();
      renderer.render(scene, camera);
    },
    onWindowResize: function () {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.left = (900 * camera.aspect) / -2;
      camera.right = (900 * camera.aspect) / 2;
      camera.top = 900 / 2;
      camera.bottom = -900 / 2;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.setViewport(0, 0, window.innerWidth, window.innerHeight);
      renderer.render(scene, camera);
    },
    cameraForward: function () {
      const direction = new THREE.Vector3();
      camera.getWorldDirection(direction);
      direction.y = 0;
      camera.position.addScaledVector(direction, 1.0);
    },
    previewMovementPosition: function (x, y) {
      let raycaster = new THREE.Raycaster();
      previewPoint.visible = true;
      raycaster.setFromCamera(
        new THREE.Vector2(
          (x / window.innerWidth) * 2 - 1,
          -(y / window.innerHeight) * 2 + 1
        ),
        camera
      );
      const intersects = raycaster.intersectObjects([plane]);
      this.position = intersects[0].point;
      previewPoint.position.set(this.position.x, 0, this.position.z);
    },
    setMovementRotation: function (x, y) {
      let raycaster = new THREE.Raycaster();
      previewPoint.visible = true;
      raycaster.setFromCamera(
        new THREE.Vector2(
          (x / window.innerWidth) * 2 - 1,
          -(y / window.innerHeight) * 2 + 1
        ),
        camera
      );
      const intersects = raycaster.intersectObjects([plane]);
      let rotation = intersects[0].point;
      var angleRadians = Math.atan2(
        rotation.z - this.position.z,
        rotation.x - this.position.x
      );
      this.offset = angleRadians;
      console.log("this.offset");
      console.log(this.offset);
    },
    moveToPosition: function () {
      previewPoint.visible = false;
      camera.position.set(this.position.x, 1.8, this.position.z);
      plane.position.set(this.position.x, 0, this.position.z);
      controls.alphaOffset = controls.alphaOffset + this.offset;
      console.log("controls.alphaOffset");
      console.log(controls.alphaOffset);
    },
    handleInput: function (event) {
      event.preventDefault();
      switch (event.type) {
        case "touchstart":
          this.mouseDown = true;
          if (this.mouseDown === true) {
            this.previewMovementPosition(
              event.changedTouches[0].screenX,
              event.changedTouches[0].screenY
            );
          }
          break;
        case "mousedown":
          this.mouseDown = true;
          if (this.mouseDown === true) {
            this.previewMovementPosition(event.clientX, event.clientY);
          }
          break;
        case "touchmove":
          if (this.mouseDown === true) {
            this.setMovementRotation(
              event.changedTouches[0].screenX,
              event.changedTouches[0].screenY
            );
          }
          break;
        case "mousemove":
          if (this.mouseDown === true) {
            this.setMovementRotation(event.clientX, event.clientY);
          }
          break;
        case "touchend":
          if (this.mouseDown === true) {
            this.moveToPosition();
            this.mouseDown = false;
          }
          break;
        case "mouseup":
          if (this.mouseDown === true) {
            this.moveToPosition();
            this.mouseDown = false;
          }
          break;
        default:
        //nothing;
      }
    },
  },
  mounted() {
    const startButton = document.getElementById("startButton");
    startButton.addEventListener(
      "click",
      function () {
        this.init();
        this.animate();
        startButton.remove();
      }.bind(this),
      false
    );
  },
};
</script>

<style>
body,
html,
#app {
  width: 100%;
  height: 100%;
  overflow: hidden;
  margin: 0;
  padding: 0;
  border: 0;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  touch-action: manipulation;
  font-weight: 900;
  color: #1c1c1e;
}
#startButton {
  width: 100%;
  height: 500px;
}
#three {
  width: 100%;
  height: 100%;
}
#info {
  color: red;
  position: absolute;
  z-index: 2;
  top: 0;
  right: 0;
}
</style>
