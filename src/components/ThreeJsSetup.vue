<script setup>
import {
  Scene,
  PerspectiveCamera,
  WebGLRenderer,
  Color,
  AxesHelper,
  Clock,
  DirectionalLight,
  Vector2,
  CameraHelper,
  CubeTextureLoader,
  sRGBEncoding,
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
import { GUI } from 'dat.gui'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import sensorsData from '../assets/sensorsData'

/**************************************
 ******** Model Loaders
 *************************************/
const gltfLoader = new GLTFLoader()

/**************************************
 ******** GUI
 *************************************/
const gui = new GUI()
const sceneSetup = gui.addFolder('Scene Setup')
const sunSetup = gui.addFolder('Sun Setup')
const sensorsLayers = gui.addFolder('sensors')

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

const backgroundParams = {
  visible_env: true,
  background_color: '#9ba3b0',
}

scene.background = backgroundParams.visible_env
  ? environment
  : backgroundParams.background_color
scene.environment = environment

// GUI

sceneSetup
  .add(backgroundParams, 'visible_env')
  .name('Visible ENV')
  .onChange(() => {
    scene.background = backgroundParams.visible_env
      ? environment
      : new Color(backgroundParams.background_color)
  })

sceneSetup.addColor(backgroundParams, 'background_color').onChange(() => {
  scene.background = new Color(backgroundParams.background_color)
})

/**************************************
 ******** Camera
 *************************************/
const camera = new PerspectiveCamera(
  45,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
)
camera.position.set(30, 20, 70)

const sun = new DirectionalLight(params.sunColor, 3)
sun.position.set(-50, 30, 50)
sun.castShadow = true

sun.shadow.mapSize = new Vector2(4096, 4096)
sun.shadow.camera.far = 300
sun.shadow.camera.left = -50
sun.shadow.camera.right = 50
sun.shadow.camera.top = 50
sun.shadow.camera.bottom = -50

scene.add(sun)

const sunCamHelper = new CameraHelper(sun.shadow.camera)
sunCamHelper.visible = false
scene.add(sunCamHelper)

// GUI
sunSetup.add(sunCamHelper, 'visible').name('Sun Shaper')
sunSetup.add(sun.position, 'x').name('Sun X').min(-200).max(200).step(0.001)
sunSetup.add(sun.position, 'y').name('Sun Y').min(-200).max(200).step(0.001)
sunSetup.add(sun.position, 'z').name('Sun Z').min(-200).max(200).step(0.001)

// const sunHelper = new DirectionalLightHelper(sun, 1)
// scene.add(sunHelper)

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

/**************************************
 ******** Controls
 *************************************/
const orbitControls = new OrbitControls(camera, renderer.domElement)
// orbitControls.autoRotate = true
orbitControls.enableDamping = true
orbitControls.dampingFactor = 0.1

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
 ******** Entire Scene Group
 *************************************/
const entireScene = new Group()
scene.add(entireScene)

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
  sceneSetup
    .add(terrainParams, 'visible')
    .name('Terrain')
    .onChange(() =>
      terrain.scene.traverse(object => {
        if (object.isMesh) {
          object.visible = terrainParams.visible
        }
      })
    )

  entireScene.add(terrain.scene)
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
  sceneSetup
    .add(baseBuildingParams, 'visible')
    .name('Base Building')
    .onChange(() =>
      baseBuilding.scene.traverse(object => {
        if (object.isMesh) {
          object.visible = baseBuildingParams.visible
        }
      })
    )

  entireScene.add(baseBuilding.scene)
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
    sceneSetup
      .add(surroundingParams, 'visible')
      .name('Surroundings')
      .onChange(() =>
        surroundingBuildings.scene.traverse(object => {
          if (object.isMesh) {
            object.visible = surroundingParams.visible
          }
        })
      )

    entireScene.add(surroundingBuildings.scene)
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
  sceneSetup
    .add(treesParams, 'visible')
    .name('Trees')
    .onChange(() =>
      trees.scene.traverse(object => {
        if (object.isMesh) {
          object.visible = treesParams.visible
        }
      })
    )

  entireScene.add(trees.scene)
})

/**************************************
 ******** Window Glasses
 *************************************/
const glassesParams = {
  visible: true,
}

gltfLoader.load('assets/glTF/glasses.glb', glasses => {
  // GUI
  sceneSetup
    .add(glassesParams, 'visible')
    .name('Glasses')
    .onChange(() =>
      glasses.scene.traverse(object => {
        if (object.isMesh) {
          object.visible = glassesParams.visible
        }
      })
    )

  entireScene.add(glasses.scene)
})

/**************************************
 ******** Particles
 *************************************/
const sensorsGroup = new Group()

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

sensorsGroup.add(sensors)
sensorsGroup.rotateX(Math.PI * -0.5)

entireScene.add(sensorsGroup)

// GUI
sensorsLayers.add(sensorsGroup, 'visible').name('Sun Hours')

/**************************************
 ******** Y Rotation Fix
 *************************************/
entireScene.rotateY(Math.PI * -0.36)

/**************************************
 ******** Axes
 *************************************/
const axes = new AxesHelper(50)
scene.add(axes)

const clock = new Clock()

const tick = () => {
  // Time
  const elapsedTime = clock.getElapsedTime()

  // required if controls.enableDamping or controls.autoRotate are set to true
  orbitControls.update()

  // sunHelper.update()

  /**************************************
   ******** Render
   *************************************/
  renderer.render(scene, camera)
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

<style>
h1 {
  color: black;
  font-weight: bold;
}
</style>
