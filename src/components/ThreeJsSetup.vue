<script setup>
import {
  Scene,
  PerspectiveCamera,
  WebGLRenderer,
  BoxGeometry,
  MeshStandardMaterial,
  Mesh,
  FrontSide,
  Color,
  SpotLight,
  AxesHelper,
  Clock,
} from 'three'
import { onMounted, ref } from 'vue'
import gsap from 'gsap'
import STLLoader from './loaders/STLLoader'

// Ref
const viewer = ref()

// Scene
const scene = new Scene()
scene.background = new Color('#444d5c')

// Camera
const camera = new PerspectiveCamera(
  45,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
)
camera.position.set(1, 1, 9)

// camera.rotation.x = -0.5

// Light
const light = new SpotLight('hsl(0, 100%, 100%)')
light.position.set(10, 5, 5)
scene.add(light)

// Renderer
const renderer = new WebGLRenderer()
renderer.setSize(1280, 720)

// Mesh
const geometry = new BoxGeometry()
const material = new MeshStandardMaterial({
  side: FrontSide,
  color: 'hsl(0, 0, 50%)',
  wireframe: false,
})
const cube = new Mesh(geometry, material)
// scene.add(cube)

const loader = new STLLoader()

loader.load('assets/STL_files/test_building.stl', geo => {
  const building = new Mesh(geo, material)
  // building.rotation.y = Math.PI * -0.5

  scene.add(building)

  camera.lookAt(building.position)
})

// Axes
const axes = new AxesHelper(50)
scene.add(axes)

const clock = new Clock()

// Animatations
const tick = () => {
  // Time
  const elapsedTime = clock.getElapsedTime()

  // Object transformations
  // camera.position.y = Math.sin(elapsedTime * 1)
  // camera.position.x = Math.cos(elapsedTime * 1)
  // camera.lookAt(cube.position)

  // Render
  renderer.render(scene, camera)

  requestAnimationFrame(tick)
}

onMounted(() => {
  viewer.value.appendChild(renderer.domElement)
  tick()
})

// gsap.to(cube.position, { x: 2, duration: 2 })
// gsap.to(cube.position, { x: 0, duration: 2, delay: 2 })
</script>

<template>
  <h1>3D Viewer</h1>

  <main ref="viewer"></main>
</template>

<style></style>
