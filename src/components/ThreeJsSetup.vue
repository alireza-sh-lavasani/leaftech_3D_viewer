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
} from 'three'
import { onMounted, ref } from 'vue'

// Ref
const viewer = ref()

// Scene
const scene = new Scene()
scene.background = new Color('#444d5c')

// Camera
const camera = new PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
)
camera.position.x = 1
camera.position.y = 2
camera.position.z = 5
camera.rotation.x = -0.5

// Light
const light = new SpotLight('hsl(0, 100%, 100%)')
light.position.set(2, 3, 2)
scene.add(light)

// Renderer
const renderer = new WebGLRenderer()
renderer.setSize(1280, 720)

// Mesh
const geometry = new BoxGeometry()
const material = new MeshStandardMaterial({
  side: FrontSide,
  color: 'hsl(0, 100%, 50%)',
  wireframe: false,
})
const cube = new Mesh(geometry, material)
scene.add(cube)

// Axes
const axes = new AxesHelper(5)
scene.add(axes)

// Animate
const animate = () => {
  requestAnimationFrame(animate)

  cube.rotation.y += 0.01

  // Render
  renderer.render(scene, camera)
}

onMounted(() => {
  viewer.value.appendChild(renderer.domElement)
  animate()
})
</script>

<template>
  <h1>3D Viewer</h1>

  <main ref="viewer"></main>
</template>

<style></style>
