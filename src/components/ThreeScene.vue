<template>
  <!-- LOADING -->
  <div v-if="!ready" class="overlay">
    <Blur>
      <div class="loader">
        <div>Загрузка сцены…</div>

        <progress
          :value="progress"
          max="100"
        ></progress>

        <div class="percent">{{ progress }}%</div>
      </div>
    </Blur>
  </div>

  <!-- PAUSE -->
  <div v-else-if="paused" class="overlay">
    <Blur @play="resume" />
  </div>

  <!-- THREE -->
  <div ref="container" class="scene"></div>
</template>

<script setup>
import Blur from './Blur.vue'
import { ref, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { Sky } from 'three/examples/jsm/objects/Sky.js'

const container = ref(null)

const ready = ref(false)
const paused = ref(true)
const progress = ref(0)

let scene, camera, renderer, clock
let player
let yaw = 0
let pitch = 0
let pointerLocked = false
let lastTouch = null

const keys = {}
const SPEED = 3
const SENS = 0.001
const moveDir = new THREE.Vector3()

const hasPointerLock =
  'pointerLockElement' in document &&
  'requestPointerLock' in Element.prototype

const isTouch =
  'ontouchstart' in window ||
  navigator.maxTouchPoints > 0

function onKeyDown(e) {
  keys[e.code] = true
}
function onKeyUp(e) {
  keys[e.code] = false
}
function onCanvasClick() {
  renderer?.domElement?.requestPointerLock()
}

function onTouchMove(e) {
  e.preventDefault()
  if (!pointerLocked) return

  const t = e.touches[0]
  if (!lastTouch) {
    lastTouch = { x: t.clientX, y: t.clientY }
    return
  }

  const dx = t.clientX - lastTouch.x
  const dy = t.clientY - lastTouch.y
  lastTouch = { x: t.clientX, y: t.clientY }

  yaw -= dx * SENS * 2
  pitch -= dy * SENS * 2
  pitch = Math.max(-Math.PI / 2, Math.min(Math.PI / 2, pitch))

  player.rotation.y = yaw
  camera.rotation.x = pitch
}


/* =====================
   INIT
===================== */
function init() {
  scene = new THREE.Scene()

  /* SKY (виден сразу) */
  const sky = new Sky()
  sky.scale.setScalar(10000)
  scene.add(sky)

  const u = sky.material.uniforms
  u.turbidity.value = 10
  u.rayleigh.value = 2
  u.mieCoefficient.value = 0.005
  u.mieDirectionalG.value = 0.8

  const sun = new THREE.Vector3()
  sun.setFromSphericalCoords(1, Math.PI * 0.45, Math.PI * 0.25)
  u.sunPosition.value.copy(sun)

  /* Camera */
  camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.01,
    5000
  )

  /* Renderer */
  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  renderer.outputColorSpace = THREE.SRGBColorSpace
  renderer.toneMapping = THREE.ACESFilmicToneMapping
  container.value.appendChild(renderer.domElement)

  /* Lights */
  scene.add(new THREE.HemisphereLight(0xffffff, 0x444444, 1.2))
  const dir = new THREE.DirectionalLight(0xffffff, 1)
  dir.position.set(5, 10, 7)
  scene.add(dir)

  /* Player */
  player = new THREE.Object3D()
  player.position.set(0, 2, 5)
  player.add(camera)
  scene.add(player)

  /* Events */

  if (isTouch) {
    document.addEventListener('touchmove', onTouchMove, { passive: false })
  }

  document.addEventListener('pointerlockchange', onPointerLockChange)
  document.addEventListener('mousemove', onMouseMove)
  document.addEventListener('keydown', onKeyDown)
  document.addEventListener('keyup', onKeyUp)
  if (!isTouch) {
    renderer.domElement.addEventListener('click', onCanvasClick)
  }
  window.addEventListener('resize', onResize)

  /* MODEL with PROGRESS */
  const loader = new GLTFLoader()

  loader.load(
    '/apartment_interior.glb',
    gltf => {
      const model = gltf.scene
      scene.add(model)

      // 1️⃣ считаем размер модели
      const box = new THREE.Box3().setFromObject(model)
      const size = box.getSize(new THREE.Vector3())
      const maxSize = Math.max(size.x, size.y, size.z)

      // 2️⃣ нормализуем масштаб (~10 метров)
      const SCALE = 10 / maxSize
      model.scale.setScalar(SCALE)

      // 3️⃣ пересчитываем bounding box ПОСЛЕ scale
      box.setFromObject(model)
      const center = box.getCenter(new THREE.Vector3())
      model.position.sub(center)

      progress.value = 100
      ready.value = true
    },
    xhr => {
      if (xhr.total) {
        progress.value = Math.round((xhr.loaded / xhr.total) * 100)
      }
    },
    err => {
      console.error('Ошибка загрузки модели', err)
    }
  )

  clock = new THREE.Clock()
  animate()
}

/* =====================
   POINTER LOCK
===================== */
function onPointerLockChange() {
  // 🔒 защита: событие могло прийти после dispose
  if (!renderer?.domElement) return

  const locked = document.pointerLockElement === renderer.domElement
  pointerLocked = locked
  paused.value = !locked

  if (!locked) {
    lastTouch = null
    keys.KeyW = false
    keys.KeyA = false
    keys.KeyS = false
    keys.KeyD = false
  }
}


/* =====================
   CONTROLS
===================== */
function onMouseMove(e) {
  if (!pointerLocked) return

  yaw -= e.movementX * SENS
  pitch -= e.movementY * SENS
  pitch = Math.max(-Math.PI / 2, Math.min(Math.PI / 2, pitch))

  player.rotation.y = yaw
  camera.rotation.x = pitch
}

function move(delta) {
  moveDir.set(0, 0, 0)

  if (keys['KeyW']) moveDir.z -= 1
  if (keys['KeyS']) moveDir.z += 1
  if (keys['KeyA']) moveDir.x -= 1
  if (keys['KeyD']) moveDir.x += 1
  if (keys['Space']) moveDir.y += 1
  if (keys['ShiftLeft'] || keys['ControlLeft']) moveDir.y -= 1

  if (moveDir.lengthSq()) {
    moveDir.normalize()
    moveDir.applyQuaternion(player.quaternion)
    player.position.addScaledVector(moveDir, SPEED * delta)
  }
}

/* =====================
   LOOP
===================== */
let rafId

function animate() {
  rafId = requestAnimationFrame(animate)

  const delta = Math.min(clock.getDelta(), 0.05)
  if (!paused.value) move(delta)

  renderer.render(scene, camera)
}

/* =====================
   UTILS
===================== */
function resume() {
  lastTouch = null
  // 📱 iPad / touch — без pointer lock
  if (!hasPointerLock || isTouch) {
    paused.value = false
    pointerLocked = true
    return
  }

  // 🖥 Desktop
  renderer?.domElement?.requestPointerLock()
}

function onResize() {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

onMounted(init)

onUnmounted(() => {
  cancelAnimationFrame(rafId)
  if (!isTouch && renderer?.domElement) {
    renderer.domElement.removeEventListener('click', onCanvasClick)
  }
  document.removeEventListener('touchmove', onTouchMove)
  window.removeEventListener('resize', onResize)
  document.removeEventListener('mousemove', onMouseMove)
  document.removeEventListener('pointerlockchange', onPointerLockChange)
  document.removeEventListener('keydown', onKeyDown)
  document.removeEventListener('keyup', onKeyUp)

  if (renderer?.domElement) {
    if (!isTouch && renderer?.domElement) {
      renderer.domElement.removeEventListener('click', onCanvasClick)
    }
    renderer.dispose()
    renderer.forceContextLoss()
    renderer.domElement.remove()
  }
})
</script>

<style scoped>
.scene {
  width: 100vw;
  height: 100vh;
}

.overlay {
  position: fixed;
  inset: 0;
  z-index: 10;
}

.loader {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  text-align: center;
}

progress {
  width: 220px;
  height: 12px;
}

.percent {
  font-size: 14px;
  opacity: 0.7;
}
</style>
