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
  Group,
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
import sensorsData from '../assets/sensorsData'

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
  cubeColor: 0,
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
camera.position.set(30, 30, 70)

/**************************************
 ******** Lights
 *************************************/
// const ambientLight = new AmbientLight(params.ambientColor, 0.75)
// scene.add(ambientLight)

// gui
//   .addColor(params, 'ambientColor')
//   .onChange(() => ambientLight.color.set(params.ambientColor))

// const sun = new DirectionalLight(params.sunColor, 2)
// sun.position.set(100, 35, 75)
// sun.castShadow = true
// sun.shadow.mapSize = new Vector2(4096, 4096)
// sun.shadow.camera.far = 250
// sun.shadow.camera.left = -250
// sun.shadow.camera.right = 250
// sun.shadow.camera.top = 250
// sun.shadow.camera.bottom = -250

// scene.add(sun)

const sun2 = new DirectionalLight(params.sunColor, 3)
sun2.position.set(50, 30, 50)
sun2.castShadow = true

sun2.shadow.mapSize = new Vector2(4096, 4096)
sun2.shadow.camera.far = 250
sun2.shadow.camera.left = -50
sun2.shadow.camera.right = 50
sun2.shadow.camera.top = 50
sun2.shadow.camera.bottom = -50

scene.add(sun2)

// const sunHelper = new DirectionalLightHelper(sun, 1)
// scene.add(sunHelper)

const sunCamHelper = new CameraHelper(sun2.shadow.camera)
sunCamHelper.visible = true
// scene.add(sunCamHelper)

// gui.add(sunCamHelper, 'visible').name('sunCamHelper')
// gui.add(sunHelper, 'visible').name('sunHelper')
// gui.addColor(params, 'sunColor').onChange(() => sun.color.set(params.sunColor))

// Bounce light
// const bounce = new DirectionalLight(params.sunColor, 1)
// bounce.position.set(0, 3.5, -7.5)
// bounce.castShadow = false

// scene.add(bounce)

// const bounceHelper = new DirectionalLightHelper(bounce, 1)
// scene.add(bounceHelper)

// const bounceCamHelper = new CameraHelper(bounce.shadow.camera)
// bounceCamHelper.visible = false
// scene.add(bounceCamHelper)

// gui.add(bounceCamHelper, 'visible').name('bounceCamHelper')
// gui.add(bounceHelper, 'visible').name('bounceHelper')
// gui
//   .addColor(params, 'bounceColor')
//   .onChange(() => bounce.color.set(params.bounceColor))

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

// gui.add(renderer, 'toneMapping', {
//   No: NoToneMapping,
//   Linear: LinearToneMapping,
//   Reinhard: ReinhardToneMapping,
//   CineonL: CineonToneMapping,
//   ACES: ACESFilmicToneMapping,
// })

/**************************************
 ******** Controls
 *************************************/
const orbitControls = new OrbitControls(camera, renderer.domElement)
// orbitControls.autoRotate = true
orbitControls.enableDamping = true
orbitControls.dampingFactor = 0.1

// const transformControl = new TransformControls(camera, renderer.domElement)

// transformControl.addEventListener('dragging-changed', function (event) {
//   orbitControls.enabled = !event.value
// })

// transformControl.attach(sun)
// scene.add(transformControl)

// gui.add(transformControl, 'visible').name('showSunTransform')

/**************************************
 ******** Post Processing
 *************************************/
// const composer = new EffectComposer(renderer)
// const ssaoPass = new SSAOPass(scene, camera, viewportSize.x, viewportSize.y)
// ssaoPass.kernelRadius = 16
// composer.addPass(ssaoPass)

// gui
//   .add(ssaoPass, 'output', {
//     Default: SSAOPass.OUTPUT.Default,
//     'SSAO Only': SSAOPass.OUTPUT.SSAO,
//     'SSAO Only + Blur': SSAOPass.OUTPUT.Blur,
//     Beauty: SSAOPass.OUTPUT.Beauty,
//     Depth: SSAOPass.OUTPUT.Depth,
//     Normal: SSAOPass.OUTPUT.Normal,
//   })
//   .onChange(function (value) {
//     ssaoPass.output = parseInt(value)
//   })
// gui.add(ssaoPass, 'kernelRadius').min(0).max(4096)
// gui.add(ssaoPass, 'minDistance').min(0.001).max(0.02)
// gui.add(ssaoPass, 'maxDistance').min(0.01).max(0.3)

/**************************************
 ******** Materials
 *************************************/
const material = new MeshStandardMaterial({
  side: FrontSide,
  color: params.color,
  wireframe: false,
})

material.envMapIntensity = 1

// gui.add(material, 'metalness').min(0).max(1).step(0.001)
// gui.add(material, 'roughness').min(0).max(1).step(0.001)

// gui
//   .addColor(params, 'color')
//   .onChange(() => material.color.set(params.color))
//   .name('Building Color')

/**************************************
 ******** Color Ramp
 *************************************/
//Color Ramp Class
class ColorRamp {
  //Takes a list of colors
  constructor(list) {
    this.list = list
    this.color = new Color()
  }

  //Returns color at position t
  at(t) {
    //Find indexes
    let start = Math.floor(t * this.list.length)
    let end = (start + 1) % this.list.length

    //Amount between two colors
    let tt = t * this.list.length - start

    //Copy and interpolate, return color
    this.color.copy(this.list[start])
    this.color.lerp(this.list[end], tt)
    return this.color
  }
}

//Create Ramp Instance (https://coolors.co/c8ffbe-edffab-ba9593-89608e-623b5a)
var ramp = new ColorRamp([
  new Color('#1922B3'),
  new Color('#5134E6'),
  new Color('#AA39F0'),
  new Color('#CE42F1'),
  new Color('#F673F4'),
  new Color('#FBC2F9'),
])

// gui
//   .add(params, 'cubeColor')
//   .min(0)
//   .max(0.5)
//   .step(0.001)
//   .onChange(() => cubeMat.color.set(ramp.at(params.cubeColor)))

/**************************************
 ******** Range conversion
 *************************************/

const rangeConverter = (
  oldValue,
  { oldMin = 0, oldMax = 4000, newMin = 0, newMax = 0.5 }
) => {
  const oldRange = oldMax - oldMin
  const newRange = newMax - newMin
  const newValue = ((oldValue - oldMin) * newRange) / oldRange + newMin

  return newValue
}

/**************************************
 ******** Mesh
 *************************************/
const plane = new PlaneGeometry(15, 15)
const planeMat = new MeshStandardMaterial({ color: '#fff' })

const planeMesh = new Mesh(plane, planeMat)
planeMesh.rotateX(Math.PI * -0.5)
planeMesh.receiveShadow = true

// scene.add(planeMesh)

const geometry = new BoxGeometry()

const cubeMat = new MeshStandardMaterial({
  color: ramp.at(params.cubeColor),
})

const cube = new Mesh(geometry, cubeMat)
cube.position.set(2, 0.5, 0)
cube.castShadow = true

// scene.add(cube)

/**************************************
 ******** Building
 *************************************/
stlLoader.load('assets/STL_files/test_building.stl', geo => {
  const building = new Mesh(geo, material)
  // gui
  //   .add(building.position, 'x')
  //   .name('Test Position')
  //   .min(-3)
  //   .max(3)
  //   .step(0.01)

  building.receiveShadow = true
  building.castShadow = true

  // scene.add(building)
})

/**************************************
 ******** Helmet
 *************************************/
gltfLoader.load('assets/glTF/FlightHelmet.gltf', gltf => {
  gltf.scene.traverse(object => {
    if (object.isMesh) object.castShadow = true
  })

  // scene.add(gltf.scene)
})

/**************************************
 ******** Building GLTF
 *************************************/
const group = new Group()

gltfLoader.load('assets/glTF/building.glb', building_gltf => {
  building_gltf.scene.traverse(object => {
    if (object.isMesh) {
      object.castShadow = true
      object.receiveShadow = true
    }
  })

  // group.add(building_gltf.scene)
  // scene.add(group)
})

/**************************************
 ******** Terrain
 *************************************/
const terrainParams = {
  visible: true,
}

gltfLoader.load('assets/glTF/terrain.glb', terrain => {
  terrain.scene.traverse(object => {
    if (object.isMesh) {
      object.receiveShadow = true
    }
  })

  // GUI
  gui
    .add(terrainParams, 'visible')
    .name('Terrain')
    .onChange(() =>
      terrain.scene.traverse(object => {
        if (object.isMesh) {
          object.visible = terrainParams.visible
        }
      })
    )

  scene.add(terrain.scene)
})

/**************************************
 ******** Base Building
 *************************************/
const baseBuildingParams = {
  visible: true,
}

gltfLoader.load('assets/glTF/baseBuilding.glb', baseBuilding => {
  baseBuilding.scene.traverse(object => {
    if (object.isMesh) {
      object.receiveShadow = true
      object.castShadow = true
    }
  })

  // GUI
  gui
    .add(baseBuildingParams, 'visible')
    .name('Base Building')
    .onChange(() =>
      baseBuilding.scene.traverse(object => {
        if (object.isMesh) {
          object.visible = baseBuildingParams.visible
        }
      })
    )

  scene.add(baseBuilding.scene)
})

/**************************************
 ******** Surrounding Buildings
 *************************************/
const surroundingParams = {
  visible: true,
}

gltfLoader.load(
  'assets/glTF/surroundingBuildings.glb',
  surroundingBuildings => {
    // GUI
    gui
      .add(surroundingParams, 'visible')
      .name('Surroundings')
      .onChange(() =>
        surroundingBuildings.scene.traverse(object => {
          if (object.isMesh) {
            object.visible = surroundingParams.visible
          }
        })
      )

    scene.add(surroundingBuildings.scene)
  }
)

/**************************************
 ******** Trees
 *************************************/
const treesParams = {
  visible: true,
}

gltfLoader.load('assets/glTF/trees.glb', trees => {
  // GUI
  gui
    .add(treesParams, 'visible')
    .name('Trees')
    .onChange(() =>
      trees.scene.traverse(object => {
        if (object.isMesh) {
          object.visible = treesParams.visible
        }
      })
    )

  scene.add(trees.scene)
})

/**************************************
 ******** Window Glasses
 *************************************/
const glassesParams = {
  visible: true,
}

gltfLoader.load('assets/glTF/glasses.glb', glasses => {
  // GUI
  gui
    .add(glassesParams, 'visible')
    .name('Glasses')
    .onChange(() =>
      glasses.scene.traverse(object => {
        if (object.isMesh) {
          object.visible = glassesParams.visible
        }
      })
    )

  scene.add(glasses.scene)
})

/**************************************
 ******** Particles
 *************************************/

// Placement
const sensorsGeo = new BufferGeometry()
const count = sensorsData.length

const sensorPositions = new Float32Array(count * 3)
const sensorColors = new Float32Array(count * 3)

let flatPositions = []
let rgbFlatColors = []

sensorsData.forEach(({ utmX, utmY, utmZ, sun_hours }) => {
  flatPositions.push(utmX)
  flatPositions.push(utmY)
  flatPositions.push(utmZ)

  rgbFlatColors.push(ramp.at(rangeConverter(sun_hours, {})).r)
  rgbFlatColors.push(ramp.at(rangeConverter(sun_hours, {})).g)
  rgbFlatColors.push(ramp.at(rangeConverter(sun_hours, {})).b)
})

for (let i = 0; i < count * 3; i++) {
  // sensorPositions[i] = (Math.random() - 0.5) * 10
  sensorPositions[i] = flatPositions[i]
  sensorColors[i] = rgbFlatColors[i]
}

sensorsGeo.setAttribute('position', new BufferAttribute(sensorPositions, 3))
sensorsGeo.setAttribute('color', new BufferAttribute(sensorColors, 3))

// Material
const sensorsMat = new PointsMaterial({
  size: 0.5,
  sizeAttenuation: true,
  vertexColors: true,
})

const sensors = new Points(sensorsGeo, sensorsMat)

group.add(sensors)
group.rotateX(Math.PI * -0.5)

scene.add(group)

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

  // sunHelper.update()
  // bounceHelper.update()

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
