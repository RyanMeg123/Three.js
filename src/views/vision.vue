<!--  -->
<template>
  <div id="vision-container"></div>
</template>

<script>
// 导入的其他文件 例如：import moduleName from 'modulePath';
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import Stats from "@/utils/stats.js";
import '@/utils/vision2.js'
export default {
  //import所引入的组件注册
  components: {},

  data() {
    return {
      scene: null,
      camera: null,
      renderer: null,
      container: null,
      HEIGHT: null,
      WIDTH: null,
      fieldOfView: null,
      aspectRatio: null,
      nearPlane: null,
      farPlane: null,
      stats: null,
      geometry: null,
      particleCount: null,
      i: null,
      h: null,
      color: null,
      size: null,
      materials: [],
      mouseX: 0,
      mouseY: 0,
      windowHalfX: 0,
      windowHalfY: 0,
      cameraZ: 0,
      fogHex: 0,
      fogDensity: 0,
      parameters: {},
      parameterCount: null,
      particles: null
    };
  },
  mounted() {
    // this.init();
    // this.animate();
  },
  methods: {
    init() {
      this.HEIGHT = window.innerHeight;
      this.WIDTH = window.innerWidth;
      this.windowHalfX = this.WIDTH / 2;
      this.windowHalfY = this.HEIGHT / 2;

      this.fieldOfView = 45;
      this.aspectRatio = this.WIDTH / this.HEIGHT;
      this.nearPlane = 1;
      this.farPlane = 3000;

      this.cameraZ = this.farPlane / 3; /*	So, 1000? Yes! move on!	*/
      this.fogHex = 0x000000; /* As black as your heart.	*/
      this.fogDensity = 0.0007; /* So not terribly dense?	*/

      this.camera = new THREE.PerspectiveCamera(
        this.fieldOfView,
        this.aspectRatio,
        this.nearPlane,
        this.farPlane
      );
      this.camera.position.z = this.cameraZ;

      this.scene = new THREE.Scene();
      this.scene.fog = new THREE.FogExp2(this.fogHex, this.fogDensity);

      this.container = document.createElement("div");
      let box = document.getElementById("vision-container");
      box.appendChild(this.container);
      box.style.margin = 0;
      box.style.overflow = "hidden";

      this.geometry = new THREE.Geometry(); /*	NO ONE SAID ANYTHING ABOUT MATH! UGH!	*/

      this.particleCount = 20000; /* Leagues under the sea */
      console.log(this.particleCount, "particleCount");
      for (let a = 0; a < 20000; a++) {
        var vertex = new THREE.Vector3();
        vertex.x = Math.random() * 2000 - 1000;
        vertex.y = Math.random() * 2000 - 1000;
        vertex.z = Math.random() * 2000 - 1000;

        this.geometry.vertices.push(vertex);
      }

      /*	We can't stop here, this is bat country!	*/

      this.parameters = [
        [[1, 1, 0.5], 5],
        [[0.95, 1, 0.5], 4],
        [[0.9, 1, 0.5], 3],
        [[0.85, 1, 0.5], 2],
        [[0.8, 1, 0.5], 1]
      ];
      this.parameterCount = this.parameters.length;

      for (let i = 0; i < this.parameterCount; i++) {
        this.color = this.parameters[i][0];
        this.size = this.parameters[i][1];

        this.materials[i] = new THREE.PointCloudMaterial({
          size: this.size
        });

        this.particles = new THREE.PointCloud(this.geometry, this.materials[i]);

        this.particles.rotation.x = Math.random() * 6;
        this.particles.rotation.y = Math.random() * 6;
        this.particles.rotation.z = Math.random() * 6;

        this.scene.add(this.particles);
      }

      this.renderer = new THREE.WebGLRenderer(); /*	Rendererererers particles.	*/
      this.renderer.setPixelRatio(
        window.devicePixelRatio
      ); /*	Probably 1; unless you're fancy.	*/
      this.renderer.setSize(
        this.WIDTH,
        this.HEIGHT
      ); /*	Full screen baby Wooooo!	*/

      this.container.appendChild(
        this.renderer.domElement
      ); /* Let's add all this crazy junk to the page.	*/

      this.stats = new Stats();
      this.stats.domElement.style.position = "absolute";
      this.stats.domElement.style.top = "0px";
      this.stats.domElement.style.right = "0px";
      this.container.appendChild(this.stats.domElement);

      /* Event Listeners */

      window.addEventListener("resize", this.onWindowResize, false);
      document.addEventListener("mousemove", this.onDocumentMouseMove, false);
      document.addEventListener("touchstart", this.onDocumentTouchStart, false);
      document.addEventListener("touchmove", this.onDocumentTouchMove, false);
    },
    animate() {
      requestAnimationFrame(this.animate);
      this.render();
      this.stats.update();
    },
    render() {
      var time = Date.now() * 0.00005;

      this.camera.position.x += (this.mouseX - this.camera.position.x) * 0.05;
      this.camera.position.y += (-this.mouseY - this.camera.position.y) * 0.05;

      this.camera.lookAt(this.scene.position);

      for (let i = 0; i < this.scene.children.length; i++) {
        var object = this.scene.children[i];

        if (object instanceof THREE.PointCloud) {
          object.rotation.y = time * (i < 4 ? i + 1 : -(i + 1));
        }
      }

      for (let i = 0; i < this.materials.length; i++) {
        this.color = this.parameters[i][0];

        this.h = ((360 * (this.color[0] + time)) % 360) / 360;
        this.materials[i].color.setHSL(this.h, this.color[1], this.color[2]);
      }

      this.renderer.render(this.scene, this.camera);
    },
    onDocumentMouseMove(e) {
      this.mouseX = e.clientX - this.windowHalfX;
      this.mouseY = e.clientY - this.windowHalfY;
    },
    onDocumentTouchStart(e) {
      if (e.touches.length === 1) {
        e.preventDefault();
        this.mouseX = e.touches[0].pageX - this.windowHalfX;
        this.mouseY = e.touches[0].pageY - this.windowHalfY;
      }
    },
    onDocumentTouchMove(e) {
      if (e.touches.length === 1) {
        e.preventDefault();
        this.mouseX = e.touches[0].pageX - this.windowHalfX;
        this.mouseY = e.touches[0].pageY - this.windowHalfY;
      }
    },
    onWindowResize() {
      this.windowHalfX = window.innerWidth / 2;
      this.windowHalfY = window.innerHeight / 2;

      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
    }
  }
};
</script>
<style>
</style>