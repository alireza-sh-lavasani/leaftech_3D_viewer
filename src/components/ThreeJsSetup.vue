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
  AmbientLight,
  MeshLambertMaterial,
  MeshPhongMaterial,
  DirectionalLight,
  PlaneGeometry,
  DoubleSide,
  DirectionalLightHelper,
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
   ******** Parameters
   *************************************/
  const params = {
    color: '#fff',
    directionalLightColor: '#ffffe8',
    ambientLightColor: '#c5dde0',
    shine: 10,
  }

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
  const ambientLight = new AmbientLight(params.ambientLightColor, 0.4)
  scene.add(ambientLight)

  gui
    .addColor(params, 'ambientLightColor')
    .onChange(() => ambientLight.color.set(params.ambientLightColor))

  const directionalLight = new DirectionalLight(
    params.directionalLightColor,
    0.75
  )

  directionalLight.position.set(10, 3.5, 7.5)
  scene.add(directionalLight)

  const sun = new DirectionalLightHelper(directionalLight, 1)
  scene.add(sun)

  gui
    .addColor(params, 'directionalLightColor')
    .onChange(() => directionalLight.color.set(params.directionalLightColor))

  gui
    .add(directionalLight.position, 'x')
    .min(-20)
    .max(20)
    .step(0.001)
    .name('Sun X')
  gui
    .add(directionalLight.position, 'y')
    .min(-20)
    .max(20)
    .step(0.001)
    .name('Sun Y')
  gui
    .add(directionalLight.position, 'z')
    .min(-20)
    .max(20)
    .step(0.001)
    .name('Sun Z')

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
   ******** Materials
   *************************************/
  const material = new MeshStandardMaterial({
    side: FrontSide,
    color: params.color,
    wireframe: false,
  })

  gui.add(material, 'metalness').min(0).max(1).step(0.001)
  gui.add(material, 'roughness').min(0).max(1).step(0.001)

  gui
    .addColor(params, 'color')
    .onChange(() => material.color.set(params.color))
    .name('Building Color')

  /**************************************
   ******** Mesh
   *************************************/
  const plane = new PlaneGeometry(15, 15)
  const planeMat = new MeshPhongMaterial({ color: '#fff' })

  const planeMesh = new Mesh(plane, planeMat)
  planeMesh.rotateX(Math.PI * -0.5)
  scene.add(planeMesh)

  const geometry = new BoxGeometry()
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
