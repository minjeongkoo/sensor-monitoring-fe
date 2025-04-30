<template>
  <div ref="container" class="lidar-container"></div>
</template>

<script>
import * as THREE from 'three'

export default {
  name: 'HomeView',
  data() {
    return {
      socket: null,
      pointsMesh: null
    }
  },
  mounted() {
    this.initThree()
    this.initSocket()
  },
  methods: {
    initThree() {
      this.scene = new THREE.Scene()

      this.camera = new THREE.PerspectiveCamera(
        75,
        window.innerWidth / window.innerHeight,
        0.1,
        1000
      )
      this.camera.position.set(0, 0, 100)

      this.renderer = new THREE.WebGLRenderer({ antialias: true })
      this.renderer.setSize(window.innerWidth, window.innerHeight)
      this.renderer.setClearColor(0x000000)
      this.$refs.container.appendChild(this.renderer.domElement)

      const grid = new THREE.GridHelper(200, 20)
      this.scene.add(grid)

      const axes = new THREE.AxesHelper(100)
      this.scene.add(axes)

      // 기본 점 (검정색, 중심에 표시)
      const geometry = new THREE.BufferGeometry()
      const vertices = new Float32Array([0, 0, 0])
      geometry.setAttribute('position', new THREE.BufferAttribute(vertices, 3))

      const material = new THREE.PointsMaterial({ color: 0x000000, size: 3 })
      this.pointsMesh = new THREE.Points(geometry, material)
      this.scene.add(this.pointsMesh)

      this.animate()
    },

    animate() {
      requestAnimationFrame(this.animate)
      this.renderer.render(this.scene, this.camera)
    },

    initSocket() {
      this.socket = new WebSocket('ws://localhost:3000')

      this.socket.addEventListener('open', () => {
        console.log('[WebSocket] Connected')
      })

      this.socket.addEventListener('message', (event) => {
        const points = JSON.parse(event.data)

        if (points.length > 0) {
          const vertices = new Float32Array(points.length * 3)
          for (let i = 0; i < points.length; i++) {
            const p = points[i]
            vertices[i * 3] = p.x
            vertices[i * 3 + 1] = p.y
            vertices[i * 3 + 2] = p.z
          }

          const geometry = new THREE.BufferGeometry()
          geometry.setAttribute('position', new THREE.BufferAttribute(vertices, 3))

          const material = new THREE.PointsMaterial({ color: 0xff44aa, size: 2 })

          if (this.pointsMesh) {
            this.scene.remove(this.pointsMesh)
          }

          this.pointsMesh = new THREE.Points(geometry, material)
          this.scene.add(this.pointsMesh)
        }
      })
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
