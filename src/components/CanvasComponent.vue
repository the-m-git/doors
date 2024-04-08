<template>
  <div :style="{ 'aspect-ratio': aspect }" style="overflow: hidden;">
    <canvas ref="canvasRef" />
  </div>
</template>

<script setup>
import * as THREE from 'three';
import { ref, onMounted, computed, watch } from 'vue'
const canvasRef = ref(null)

const scene = new THREE.Scene();
const width = ref(window.innerWidth)
const height = ref(window.innerHeight)
const aspect = computed(() => (width.value / height.value))
const camera = new THREE.PerspectiveCamera(75, width.value / height.value, 0.1, 1000);
let renderer
const geometry = new THREE.BoxGeometry(3, 5, 0.4);
const material = new THREE.MeshBasicMaterial({ color: 'purple' });
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);

camera.position.z = 5;
scene.add(camera);


onMounted(() => {
  addEventListener('resize', resizeCanvas)
  renderer = new THREE.WebGLRenderer({ canvas: canvasRef.value, antialias: true });
  updateRenderer()
  updateCamera()
  loop()
})

const updateRenderer = () => {
  renderer.setSize(width.value, height.value);
  renderer.render(scene, camera)
}
const updateCamera = () => {
  camera.aspect = aspect.value
  camera.updateProjectionMatrix()
}
const loop = () => {
  requestAnimationFrame(loop)
  updateRenderer()
  updateCamera()
  renderer.render(scene, camera)
}
const resizeCanvas = () => {
  width.value = window.innerWidth
  height.value = window.innerHeight
}
watch(aspect, loop)
</script>