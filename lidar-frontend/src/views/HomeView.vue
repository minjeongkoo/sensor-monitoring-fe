<template>
  <div id="app">
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script>
import * as THREE from 'three';

export default {
  name: 'HomeView',
  data() {
    return {
      socket: null,
      scene: null,
      camera: null,
      renderer: null,
      pointsMesh: null
    };
  },
  mounted() {
    this.initThree();
    this.initSocket();
  },
  methods: {
    initThree() {
      const width = window.innerWidth;
      const height = window.innerHeight;

      this.scene = new THREE.Scene();
      this.camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 2000);
      this.camera.position.z = 800;

      this.renderer = new THREE.WebGLRenderer({ canvas: this.$refs.canvas, antialias: true });
      this.renderer.setSize(width, height);
      this.renderer.setPixelRatio(window.devicePixelRatio);

      // 고정된 축 추가
      const axesHelper = new THREE.AxesHelper(300);
      this.scene.add(axesHelper);

      this.animate();
    },
    animate() {
      requestAnimationFrame(this.animate);
      this.renderer.render(this.scene, this.camera);
    },
    initSocket() {
      this.socket = new WebSocket('ws://localhost:3000');

      this.socket.addEventListener('message', (event) => {
        const points = JSON.parse(event.data);

        if (this.pointsMesh) {
          this.scene.remove(this.pointsMesh);
        }

        const geometry = new THREE.BufferGeometry();
        const positions = new Float32Array(points.length * 3);

        for (let i = 0; i < points.length; i++) {
          positions[i * 3] = points[i].x;
          positions[i * 3 + 1] = points[i].y;
          positions[i * 3 + 2] = points[i].z;
        }

        geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
        const material = new THREE.PointsMaterial({ size: 3, color: 0xffffff });
        this.pointsMesh = new THREE.Points(geometry, material);
        this.scene.add(this.pointsMesh);
      });
    }
  }
};
</script>

<style>
#app {
  margin: 0;
  padding: 0;
  overflow: hidden;
}
canvas {
  display: block;
}
</style>
