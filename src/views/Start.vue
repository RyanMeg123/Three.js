<!--  -->
<template>
  <div id="canvas-container">
    <canvas id="animation-canvas"></canvas>
  </div>
</template>

<script>
// 导入的其他文件 例如：import moduleName from 'modulePath';
// import '@/utils/canvas.js'
const settings = {
  pointCount: 150,
  startSpeed: 1.7,
  availableColors: ["#aaa01f", "#bba301", "#eaf2a1", "#aacccc", "#faf3aa"]
};
let context;
let objects;
let mousePos = {};
class Point {
  constructor(canvas, color, x, y, z, v) {
    this.canvas = canvas;
    this.color = color;
    this.x = x;
    this.y = y;
    this.z = z;
    this.v = v;
    this.radius = this.calulateRadius();
  }
  // 计算的圆角大小
  calulateRadius() {
    return Math.abs(this.z / 220);
  }
  // 计算位置
  calculatePosition() {
    const diameter = 2 * this.radius;
    if (
      this.x > this.canvas.width ||
      this.y > this.canvas.height ||
      this.x + diameter < 0 ||
      this.y + diameter < 0
    ) {
      this.x = this.canvas.width / 2;
      this.y = this.canvas.height / 2;
      this.z = -40;
    }
    this.radius = this.calulateRadius();
    this.x += this.v.x;
    this.y += this.v.y;
    this.z += this.v.z;
  }
  adjustTowardsMouse(distToMouse, mouseX, mouseY) {
    if (mouseX > this.x) {
      this.v.x += 10 / distToMouse;
    }
    if (mouseY > this.Y) {
      this.v.y += 10 / distToMouse;
    }
    if (mouseX <= this.x) {
      this.v.x -= 10 / distToMouse;
    }
    if (mouseY <= this.Y) {
      this.v.y -= 10 / distToMouse;
    }
  }
  draw(context) {
    context.beginPath();
    context.strokeStyle = this.color;
    context.fillStyle = this.color;
    context.arc(this.x, this.y, this.radius, 0, 2 * Math.PI);
    context.fill();
  }
}
export default {
  //import所引入的组件注册
  components: {},

  data() {
    return {
      canvas: null,
      isBlowing: false,
      mx: null,
      my: null,
      ox: null,
      oy: null,
      windowHalfX: null,
      windowHalfY: null
    };
  },
  mounted() {
    this.init();
    window.addEventListener("resize", () => {
      this.init();
    });
    // 初始化拖放对象
    var box = document.getElementById("canvas-container");
    // 获取页面中被拖放元素的引用指针
    box.style.position = "absolute"; // 绝对定位
    box.style.width = "100%"; // 定义宽度
    box.style.height = "100%"; // 定义高度
    document.addEventListener("mousemove", event => {
      // console.log(event, '??')
      this.handleMouseMove(this.e(event));
    });
    // window.addEventListener("mousedown", this.handleMouseDown(), false);
    // window.addEventListener("mouseup", this.handleMouseUp(), false);
  },
  methods: {
    e(event) {
      console.log(event, "e");
      // 定义事件对象标准化函数
      if (!event) {
        // 兼容IE浏览器
        event = window.event;
        event.target = event.srcElement;
        event.layerX = event.offsetX;
        event.layerY = event.offsetY;
      }
      event.mx = event.pageX || event.clientX + document.body.scrollLeft;
      // 计算鼠标指针的x轴距离
      event.my = event.pageY || event.clientY + document.body.scrollTop;
      // 计算鼠标指针的y轴距离
      return event; // 返回标准化的事件对象
    },
    draw() {
      context.clearRect(0, 0, this.canvas.width, this.canvas.height);
      objects.forEach(object => {
        object.calculatePosition();
        if (mousePos) {
          const xDiff = object.x - mousePos.x;
          const yDiff = object.y - mousePos.y;
          const pointDist = Math.sqrt(Math.pow(xDiff, 2) + Math.pow(yDiff, 2));
          if (pointDist < 250 + object.radius) {
            object.adjustTowardsMouse(pointDist, mousePos.x, mousePos.y);
          }
        }
        object.draw(context);
      });
      requestAnimationFrame(this.draw);
    },
    bindMouseMove() {
      let listener = document.addEventListener("mousemove", e => {
        //document.removeEventListener('mousemove', listener);
        mousePos.x = e.clientX;
        mousePos.y = e.clientY;
        //bindMouseMove();
      });
    },
    init() {
      let HEIGHT = window.innerHeight;
      let WIDTH = window.innerWidth;
      this.windowHalfX = WIDTH / 2;
      this.windowHalfY = HEIGHT / 2;
      this.canvas = document.getElementById("animation-canvas");
      this.canvas.width = window.innerWidth;
      this.canvas.height = window.innerHeight;
      objects = [];
      context = this.canvas.getContext("2d");
      for (let i = 0; i <= settings.pointCount; i++) {
        let plusOrMinusX = Math.random() < 0.5 ? -1 : 1;
        let plusOrMinusY = Math.random() < 0.5 ? -1 : 1;
        const x = Math.random() * this.canvas.width;
        const y = Math.random() * this.canvas.height;
        const z = Math.random() * -50;
        const v = {
          x: plusOrMinusX * Math.random() * settings.startSpeed,
          y: plusOrMinusY * Math.random() * settings.startSpeed,
          z: Math.random() * settings.startSpeed * 4
        };
        const colorIndex = Math.floor(
          Math.random() * settings.availableColors.length
        );
        objects.push(
          new Point(
            this.canvas,
            settings.availableColors[colorIndex],
            x,
            y,
            z,
            v
          )
        );
      }
      requestAnimationFrame(() => {
        this.draw();
      });
      //   this.bindMouseMove();

      //   document.addEventListener("touchstart", handleTouchStart, false);
      //   document.addEventListener("touchend", handleTouchEnd, false);
      //   document.addEventListener("touchmove", handleTouchMove, false);
    },
    handleMouseMove(event) {
      console.log(event, "handleMouseMove");
      mousePos = { x: event.clientX, y: event.clientY };
    },
    handleMouseDown(event) {
      this.isBlowing = true;
    },
    handleMouseUp(event) {
      this.isBlowing = false;
    },
    handleTouchStart(event) {
      if (event.touches.length > 1) {
        event.preventDefault();
        mousePos = { x: event.touches[0].pageX, y: event.touches[0].pageY };
        this.isBlowing = true;
      }
    }
  }
};
</script>
<style lang='stylus' scoped>
// @import url(); 引入公共css类
#canvas-container {
  background-color: #000;
}
</style>