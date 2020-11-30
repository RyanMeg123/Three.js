<!--  -->
<template>
  <div id="coverBlack"></div>
</template>

<script>
// 导入的其他文件 例如：import moduleName from 'modulePath';
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
class BasicView {
  constructor(camera, renderer, scene) {
    this.camera = camera;
    this.renderer = renderer;
    this.camera = camera;
    this.scene = scene;
  }
  // 窗口大小时的事件处理程序。
  handleResize(event) {
    this.camera.aspect = window.innerWidth / window.innerHeight;
    this.camera.updateProjectionMatrix();
    this.renderer.setSize(window.innerWidth, window.innerHeight);
  }
  // 开始渲染。
  startRendering() {
    this.update();
  }
  // 中调用的方法。
  update() {
    requestAnimationFrame(this.update.bind(this));
    this.onTick();
    this.render();
  }
  // 立即执行渲染。
  render() {
    this.renderer.render(this.scene, this.camera);
  }
  // 是每帧执行的函数
  onTick() {}
}
export default {
  //import所引入的组件注册
  components: {},

  data() {
    return {
      containerElement: null,
      camera: null,
      scene: null,
      renderer: null
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      this.containerElement = document.createElement("div");
      let box = document.getElementById("coverBlack");
      console.log(box, "box");
      box.appendChild(this.containerElement);
      this.scene = new THREE.Scene();
      this.camera = new THREE.PerspectiveCamera(
        45,
        window.innerWidth / window.innerHeight,
        1,
        200000
      );
      this.camera.position.z = -1000;
      // アンチエイリアス設定有無
      var needAntialias = window.devicePixelRatio == 1.0;
      this.renderer = new THREE.WebGLRenderer({
        antialias: needAntialias
      });
      this.renderer.setClearColor(0x0);
      this.renderer.setPixelRatio(window.devicePixelRatio);
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.containerElement.appendChild(this.renderer.domElement);
      window.addEventListener(
        "resize",
        e => {
          this.handleResize(e);
        },
        false
      );
    },
    handleResize(event) {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
    }
  }
};
</script>
<style lang='stylus' scoped>
// @import url(); 引入公共css类
#coverBlack {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 1;
  width: 100%;
  height: 100%;
  opacity: 1;
  background: #000000;
}
</style>