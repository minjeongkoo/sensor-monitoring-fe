<template>
  <div ref="container" class="lidar-container"></div>
</template>

<script>
import * as THREE from 'three'
import { io } from "socket.io-client"

export default {
  name: 'HomeView',
  data() {
    return {
      socket: null,
      pointsMesh: null
    }
  },
  mounted() {
    this.scene = new THREE.Scene()
    this.camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      1000
    )
    this.renderer = new THREE.WebGLRenderer({ antialias: true })
    this.renderer.setSize(window.innerWidth, window.innerHeight)
    this.$refs.container.appendChild(this.renderer.domElement)

    this.initScene()
    this.animate()
    this.initSocket()
  },
  methods: {
    initScene() {
      this.camera.position.set(0, 0, 100)
      this.scene.add(new THREE.GridHelper(200, 20))
      this.scene.add(new THREE.AxesHelper(100))
    },
    animate: function () {
      requestAnimationFrame(this.animate)
      this.renderer.render(this.scene, this.camera)
    },
    initSocket() {
      this.socket = io("http://localhost:3000");

      this.socket.on("connect", () => {
        console.log("WebSocket connected");
      });

      this.socket.on("lidar-points", (points) => {
        if (!Array.isArray(points)) return;

        const vertices = new Float32Array(points.length * 3);
        for (let i = 0; i < points.length; i++) {
          vertices[i * 3 + 0] = points[i].x;
          vertices[i * 3 + 1] = points[i].y;
          vertices[i * 3 + 2] = points[i].z;
        }

        const geometry = new THREE.BufferGeometry();
        geometry.setAttribute("position", new THREE.BufferAttribute(vertices, 3));

        const material = new THREE.PointsMaterial({ color: 0xff44aa, size: 2 });

        if (this.pointsMesh) {
          this.scene.remove(this.pointsMesh);
        }

        this.pointsMesh = new THREE.Points(geometry, material);
        this.scene.add(this.pointsMesh);
      });
    }
  }
}

</script>

<style scoped>
.lidar-container {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  margin: 0;
  padding: 0;
}
</style>
