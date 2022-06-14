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
  Vector2,
  HemisphereLight,
  CameraHelper,
  CubeTextureLoader,
  sRGBEncoding,
  ReinhardToneMapping,
  CineonToneMapping,
  ACESFilmicToneMapping,
  NoToneMapping,
  LinearToneMapping,
  BufferGeometry,
  PointsMaterial,
  Points,
  BufferAttribute,
} from 'three'
import { onMounted, ref, render, watchEffect } from 'vue'
// import gsap from 'gsap'
import STLLoader from './loaders/STLLoader'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { TransformControls } from 'three/examples/jsm/controls/TransformControls'
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer'
import { SSAOPass } from 'three/examples/jsm/postprocessing/SSAOPass'
import { GUI } from 'dat.gui'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'

/**************************************
 ******** Model Loaders
 *************************************/
const stlLoader = new STLLoader()
const gltfLoader = new GLTFLoader()

/**************************************
 ******** GUI
 *************************************/
const gui = new GUI()

/**************************************
 ******** Parameters
 *************************************/
const params = {
  color: '#fff',
  sunColor: '#fff',
  bounceColor: '#fff',
  ambientColor: '#fff',
  shine: 10,
}

/**************************************
 ******** Scene
 *************************************/
const scene = new Scene()
scene.background = new Color('#444d5c')

/**************************************
 ******** Environment
 *************************************/
const cubeTexureLoader = new CubeTextureLoader()
const environment = cubeTexureLoader.load([
  'assets/HDRI/_1/px.png',
  'assets/HDRI/_1/nx.png',
  'assets/HDRI/_1/py.png',
  'assets/HDRI/_1/ny.png',
  'assets/HDRI/_1/pz.png',
  'assets/HDRI/_1/nz.png',
])

scene.background = environment
scene.environment = environment

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
 ******** Lights
 *************************************/
const ambientLight = new AmbientLight(params.ambientColor, 0.75)
// scene.add(ambientLight)

gui
  .addColor(params, 'ambientColor')
  .onChange(() => ambientLight.color.set(params.ambientColor))

const sun = new DirectionalLight(params.sunColor, 2)
sun.position.set(10, 3.5, 7.5)
sun.castShadow = true
sun.shadow.mapSize = new Vector2(4096, 4096)
sun.shadow.camera.far = 25

scene.add(sun)

const sunHelper = new DirectionalLightHelper(sun, 1)
scene.add(sunHelper)

const sunCamHelper = new CameraHelper(sun.shadow.camera)
sunCamHelper.visible = false
scene.add(sunCamHelper)

gui.add(sunCamHelper, 'visible').name('sunCamHelper')
gui.add(sunHelper, 'visible').name('sunHelper')
gui.addColor(params, 'sunColor').onChange(() => sun.color.set(params.sunColor))

// Bounce light
const bounce = new DirectionalLight(params.sunColor, 1)
bounce.position.set(0, 3.5, -7.5)
bounce.castShadow = false

scene.add(bounce)

const bounceHelper = new DirectionalLightHelper(bounce, 1)
scene.add(bounceHelper)

const bounceCamHelper = new CameraHelper(bounce.shadow.camera)
bounceCamHelper.visible = false
scene.add(bounceCamHelper)

gui.add(bounceCamHelper, 'visible').name('bounceCamHelper')
gui.add(bounceHelper, 'visible').name('bounceHelper')
gui
  .addColor(params, 'bounceColor')
  .onChange(() => bounce.color.set(params.bounceColor))

/**************************************
 ******** Renderer
 *************************************/
const viewportSize = {
  x: 1280,
  y: 720,
}
const renderer = new WebGLRenderer()
renderer.shadowMap.enabled = true
renderer.setSize(viewportSize.x, viewportSize.y)
renderer.physicallyCorrectLights = true

renderer.outputEncoding = sRGBEncoding

gui.add(renderer, 'toneMapping', {
  No: NoToneMapping,
  Linear: LinearToneMapping,
  Reinhard: ReinhardToneMapping,
  CineonL: CineonToneMapping,
  ACES: ACESFilmicToneMapping,
})

/**************************************
 ******** Controls
 *************************************/
const orbitControls = new OrbitControls(camera, renderer.domElement)
// orbitControls.autoRotate = true
orbitControls.enableDamping = true
orbitControls.dampingFactor = 0.1

const transformControl = new TransformControls(camera, renderer.domElement)

transformControl.addEventListener('dragging-changed', function (event) {
  orbitControls.enabled = !event.value
})

transformControl.attach(sun)
scene.add(transformControl)

gui.add(transformControl, 'visible').name('showSunTransform')

/**************************************
 ******** Post Processing
 *************************************/
const composer = new EffectComposer(renderer)
const ssaoPass = new SSAOPass(scene, camera, viewportSize.x, viewportSize.y)
ssaoPass.kernelRadius = 16
composer.addPass(ssaoPass)

gui
  .add(ssaoPass, 'output', {
    Default: SSAOPass.OUTPUT.Default,
    'SSAO Only': SSAOPass.OUTPUT.SSAO,
    'SSAO Only + Blur': SSAOPass.OUTPUT.Blur,
    Beauty: SSAOPass.OUTPUT.Beauty,
    Depth: SSAOPass.OUTPUT.Depth,
    Normal: SSAOPass.OUTPUT.Normal,
  })
  .onChange(function (value) {
    ssaoPass.output = parseInt(value)
  })
gui.add(ssaoPass, 'kernelRadius').min(0).max(4096)
gui.add(ssaoPass, 'minDistance').min(0.001).max(0.02)
gui.add(ssaoPass, 'maxDistance').min(0.01).max(0.3)

/**************************************
 ******** Materials
 *************************************/
const material = new MeshStandardMaterial({
  side: FrontSide,
  color: params.color,
  wireframe: false,
})

material.envMapIntensity = 1

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
const planeMat = new MeshStandardMaterial({ color: '#fff' })

const planeMesh = new Mesh(plane, planeMat)
planeMesh.rotateX(Math.PI * -0.5)
planeMesh.receiveShadow = true

scene.add(planeMesh)

const geometry = new BoxGeometry()
const cube = new Mesh(geometry, material)
// scene.add(cube)

/**************************************
 ******** Building
 *************************************/
stlLoader.load('assets/STL_files/test_building.stl', geo => {
  const building = new Mesh(geo, material)
  gui
    .add(building.position, 'x')
    .name('Test Position')
    .min(-3)
    .max(3)
    .step(0.01)

  building.receiveShadow = true
  building.castShadow = true

  // scene.add(building)
})

/**************************************
 ******** Helmet
 *************************************/
gltfLoader.load('assets/glTF/FlightHelmet.gltf', gltf => {
  console.log(gltf)

  // scene.add(gltf.scene)
})

/**************************************
 ******** Particles
 *************************************/
// Placement
const sensorsGeo = new BufferGeometry()
const count = 50000

const sensorPositions = new Float32Array(count * 3)
const sensorColors = new Float32Array(count * 3)

for (let i = 0; i < count * 3; i++) {
  sensorPositions[i] = (Math.random() - 0.5) * 10
  sensorColors[i] = Math.random()
}

sensorsGeo.setAttribute('position', new BufferAttribute(sensorPositions, 3))
sensorsGeo.setAttribute('color', new BufferAttribute(sensorColors, 3))

// Material
const sensorsMat = new PointsMaterial({
  size: 0.05,
  sizeAttenuation: true,
  vertexColors: true
})

const sensors = new Points(sensorsGeo, sensorsMat)
scene.add(sensors)

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
  orbitControls.update()

  sunHelper.update()
  bounceHelper.update()

  /**************************************
   ******** Render
   *************************************/
  renderer.render(scene, camera)
  // composer.render()

  requestAnimationFrame(tick)
}

/**************************************
 ******** On Mount
 *************************************/
onMounted(() => {
  document.querySelector('#main').appendChild(renderer.domElement)
  tick()
})
</script>

<template>
  <h1>3D Viewer</h1>

  <main id="main"></main>
</template>

<style></style>
