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
import { onMounted, ref, watchEffect } from 'vue'
// import gsap from 'gsap'
import STLLoader from './loaders/STLLoader'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { GUI } from 'dat.gui'

/**************************************
 ******** STL Load
 *************************************/
const loader = new STLLoader()
loader.load('assets/STL_files/test_building.stl', geo => {
  /**************************************
   ******** GUI
   *************************************/
  const gui = new GUI()

  /**************************************
   ******** Scene
   *************************************/
  const scene = new Scene()
  scene.background = new Color('#444d5c')

  /**************************************
   ******** Camera
   *************************************/
  const camera = new PerspectiveCamera(
    45,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  )
  camera.position.set(3, 3, 9)

  /**************************************
   ******** Light
   *************************************/
  const light = new SpotLight('hsl(0, 100%, 100%)')
  light.position.set(10, 5, 5)
  scene.add(light)

  /**************************************
   ******** Renderer
   *************************************/
  const viewportSize = {
    x: 1280,
    y: 720,
  }
  const renderer = new WebGLRenderer()
  renderer.setSize(viewportSize.x, viewportSize.y)
  document.querySelector('#main').appendChild(renderer.domElement)

  /**************************************
   ******** Controls
   *************************************/
  const controls = new OrbitControls(camera, renderer.domElement)
  // controls.autoRotate = true
  controls.enableDamping = true
  controls.dampingFactor = 0.1

  /**************************************
   ******** Mesh
   *************************************/
  const geometry = new BoxGeometry()
  const material = new MeshStandardMaterial({
    side: FrontSide,
    color: 'hsl(0, 0, 50%)',
    wireframe: false,
  })
  const cube = new Mesh(geometry, material)
  // scene.add(cube)

  /**************************************
   ******** Building
   *************************************/
  const building = new Mesh(geo, material)
  gui
    .add(building.position, 'x')
    .name('Test Position')
    .min(-3)
    .max(3)
    .step(0.01)

  scene.add(building)

  /**************************************
   ******** Axes
   *************************************/
  const axes = new AxesHelper(50)
  scene.add(axes)

  const clock = new Clock()

  /**************************************
   ******** Animatations
   *************************************/
  // gsap.to(cube.position, { x: 2, duration: 2 })
  // gsap.to(cube.position, { x: 0, duration: 2, delay: 2 })

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

    // required if controls.enableDamping or controls.autoRotate are set to true
    controls.update()

    /**************************************
     ******** Render
     *************************************/
    renderer.render(scene, camera)

    requestAnimationFrame(tick)
  }

  tick()
})
</script>

<template>
  <h1>3D Viewer</h1>

  <main id="main"></main>
</template>

<style></style>
