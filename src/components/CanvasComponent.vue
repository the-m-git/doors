<template>
  <div @mousemove.stop="onMouseMove" :style="{ 'aspect-ratio': aspect }"
    style="overflow: hidden; position: fixed; inset: 0;">
    <div style="position: fixed; background: #000; padding: 20px; color:#fff">
      <div>Press up/down/left/right</div>
      <div>width: {{ doorWidth.toFixed(1) }}</div>
      <div>height: {{ doorHeight.toFixed(1) }}</div>
    </div>
    <canvas ref="canvasRef" />
  </div>
</template>

<script setup>
import * as THREE from 'three';
import gsap from "gsap";
import { ref, onMounted, computed, watch } from 'vue'
import wood from '@/assets/img/wood.jpg'
import bg from '@/assets/img/field.hdr'
import gold from '@/assets/img/gold.jpg'


import { RGBELoader } from '../modules/RGBELoader.js'


const doorWidth = ref(1)
const doorHeight = ref(1)
const step = 0.1
const min = 0.5
const max = 1.5
const backgroundTexture = new RGBELoader().load(bg, (txtr) => {
  txtr.mapping = THREE.EquirectangularReflectionMapping;
  scene.background = txtr;
  scene.environment = txtr
})
const woodTexture = new THREE.TextureLoader().load(wood)
const goldTexture = new THREE.TextureLoader().load(gold)
backgroundTexture.minFilter = THREE.LinearFilter



const canvasRef = ref(null)
let renderer
const scene = new THREE.Scene();
scene.background = backgroundTexture

const light = new THREE.DirectionalLight(0x404040, 6)
light.position.set(0, 3, 14);
scene.add(light)

const width = ref(window.innerWidth)
const height = ref(window.innerHeight)
const aspect = computed(() => (width.value / height.value))
const camera = new THREE.PerspectiveCamera(45, width.value / height.value, 0.1, 400);

const geometryDoor = new THREE.BoxGeometry(3, 5, 0.3);
const materialDoor = new THREE.MeshLambertMaterial({
  map: woodTexture,
});

const geometryKnob = new THREE.TorusGeometry(0.4, 0.04)
const materialKnob = new THREE.MeshLambertMaterial({
  map: goldTexture
})

const door = new THREE.Mesh(geometryDoor, materialDoor);
const knob = new THREE.Mesh(geometryKnob, materialKnob);

knob.rotation.y = Math.PI / 2
knob.position.x = 1.1
const group = new THREE.Group();
group.add(door)
group.add(knob)
group.position.z = -3
scene.add(group);
camera.rotation.z = 0
camera.position.y = 1;
camera.position.z = 5;
scene.add(camera);


onMounted(() => {
  addEventListener('resize', resizeCanvas)
  renderer = new THREE.WebGLRenderer({ canvas: canvasRef.value, antialias: true });
  updateRenderer()
  updateCamera()
  loop()
  addEventListener('keydown', onkeydown)
})
const onkeydown = (e) => {
  if ((e.key === 'ArrowLeft') && (doorWidth.value > min)) {
    doorWidth.value -= step
    gsap.to(group.scale, { duration: 0.4, x: doorWidth.value, ease: "expoScale(0.5,7,none)" });
  }
  if ((e.key === 'ArrowRight') && (doorWidth.value < max)) {
    doorWidth.value += step
    gsap.to(group.scale, { duration: 0.4, x: doorWidth.value, ease: "expoScale(0.5,7,none)" });
  }
  if ((e.key === 'ArrowUp') && (doorHeight.value < max)) {
    doorHeight.value += step
    gsap.to(group.scale, { duration: 0.4, y: doorHeight.value, ease: "expoScale(0.5,7,none)" });
  }
  if ((e.key === 'ArrowDown') && (doorHeight.value > min)) {
    doorHeight.value -= step
    gsap.to(group.scale, { duration: 0.4, y: doorHeight.value, ease: "expoScale(0.5,7,none)" });
  }
}
const opening = () => {
  gsap.to(camera.position, { delay: 1, duration: 1, z: 6, ease: "expoScale(0.5,7,none)" });
}

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
const onMouseMove = (e) => {
  requestAnimationFrame(() => {
    const xPercent = (e.clientX / width.value) - 0.5
    const yPercent = (e.clientY / height.value) - 0.5
    camera.rotation.y = - xPercent * 2
    camera.rotation.x = - yPercent * 2
  })

}
watch(aspect, loop)
</script>