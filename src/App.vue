<template>
    <div id="app">
      <canvas ref="canvas"></canvas>
    </div>
  </template>
  
  <script setup>
  import { onMounted, ref } from 'vue';
  import * as THREE from 'three';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  
  const canvas = ref(null);
  
  onMounted(() => {
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    camera.position.z = 10;
  
    const renderer = new THREE.WebGLRenderer({
      canvas: canvas.value,
      antialias: true
    });
    renderer.setSize(window.innerWidth, window.innerHeight);
  
    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
  
    const lidarOrigin = new THREE.Mesh(
      new THREE.SphereGeometry(0.2, 16, 16),
      new THREE.MeshBasicMaterial({ color: 0x000000 })
    );
    scene.add(lidarOrigin);
  
    function animate() {
      requestAnimationFrame(animate);
      controls.update();
      renderer.render(scene, camera);
    }
    animate();
  
    // WebSocket 연결
    const socket = new WebSocket('ws://localhost:3000');
  
    socket.onmessage = (event) => {
      const points = JSON.parse(event.data);
      points.forEach((p) => {
        const point = new THREE.Mesh(
          new THREE.SphereGeometry(0.02, 8, 8),
          new THREE.MeshBasicMaterial({ color: 0xff0000 })
        );
        point.position.set(p.x / 1000, p.y / 1000, p.z / 1000); // mm 단위니까 m로 나눔
        scene.add(point);
      });
    };
  });
  </script>
  
  <style scoped>
  canvas {
    width: 100vw;
    height: 100vh;
    display: block;
  }
  </style>
  