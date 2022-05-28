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
  Vector3,
} from 'three'
import { onMounted, ref } from 'vue'
import gsap from 'gsap'
import STLLoader from './loaders/STLLoader'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

// Ref
const viewer = ref()
const cursor = ref({ x: 0, y: 0 })

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
camera.position.set(3, 3, 9)

// Light
const light = new SpotLight('hsl(0, 100%, 100%)')
light.position.set(10, 5, 5)
scene.add(light)

// Renderer
const viewportSize = {
  x: 1280,
  y: 720,
}
const renderer = new WebGLRenderer()
renderer.setSize(viewportSize.x, viewportSize.y)

// Mesh
const geometry = new BoxGeometry()
const material = new MeshStandardMaterial({
  side: FrontSide,
  color: 'hsl(0, 0, 50%)',
  wireframe: false,
})
const cube = new Mesh(geometry, material)
// scene.add(cube)

// Building
const loader = new STLLoader()
loader.load('assets/STL_files/test_building.stl', geo => {
  const building = new Mesh(geo, material)
  scene.add(building)
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

  // camera.position.x = Math.sin(cursor.value.x * -2 * Math.PI) * 10
  // camera.position.z = Math.cos(cursor.value.x * -2 * Math.PI) * 10
  // camera.position.y = cursor.value.y * 10
  // camera.lookAt(new Vector3(0))

  // camera.position.x = cursor.x
  // camera.position.y = cursor.y

  // Render
  renderer.render(scene, camera)

  requestAnimationFrame(tick)
}

onMounted(() => {
  viewer.value.appendChild(renderer.domElement)

  const controls = new OrbitControls(camera, renderer.domElement)

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
