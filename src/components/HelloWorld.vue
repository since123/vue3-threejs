<script setup>
import * as THREE from 'three'
import { onMounted, ref, reactive } from 'vue';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import {
  DRACOLoader
} from 'three/examples/jsm/loaders/DRACOLoader.js'
// defineProps({
//   msg: String,
// })
import { STLLoader } from 'three/examples/jsm/loaders/STLLoader.js'

let controls
let canvasDom = ref(null)
// 创建场景
const scene = new THREE.Scene();
// 创建相机
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
camera.position.set(0, 1, 3)
// 创建渲染器
const renderer = new THREE.WebGLRenderer({
  // 抗锯齿
  antialias: true
})
// 渲染器大小
renderer.setSize(window.innerWidth, window.innerHeight)
const render = () => {
  renderer.render(scene, camera)
  requestAnimationFrame(render)
}

// 修改某些部位
let wheels = [], carBody = []

// 创建材质
const bodyMaterial = new THREE.MeshPhysicalMaterial({
  color: 0xff0000,
  metalness: 1,
  roughness: 0.5,
  clearcoat: 1,
  clearcoatRoughness: 0,

})
onMounted(() => {
  // 把渲染器插入dom
  canvasDom.value.appendChild(renderer.domElement)
  //初始化渲染器， 渲染背景
  renderer.setClearColor('#000')
  scene.background = new THREE.Color('#ccc')
  scene.environment = new THREE.Color('#ccc')
  render()
  // 添加网格地面
  const gridHelper = new THREE.GridHelper(10, 10)
  gridHelper.material.opacity = 0.2
  gridHelper.material.transparent = true
  scene.add(gridHelper)

  // 添加控制区
  controls = new OrbitControls(camera, renderer.domElement)
  controls.update()

  // 添加模型

  const loader = new GLTFLoader()
  const dracoLoader = new DRACOLoader()
  dracoLoader.setDecoderPath('./draco/gltf/')
  loader.setDRACOLoader(dracoLoader)


  loader.load('./model/miyu(1).glb', (gltf) => {
    console.log("gltf", gltf)
    const bmw = gltf.scene


    bmw.traverse(child => {
      if (child.isMesh) {
        console.log(child.name)
      }
      if (child.isMesh && child.name.includes('Object_7')) {
        wheels.push(child)
      }
      if (child.isMesh && child.name.includes('Object_21')) {
        carBody = child
        carBody.material = bodyMaterial
      }

    })
    scene.add(bmw)
  })
  // 电池模型
  // const loader = new STLLoader()
  // loader.load("./model/0.stl", function (geometry) {
  //   //创建纹理
  //   var mat = new THREE.MeshLambertMaterial({ color: 0x00ffff });
  //   var mesh = new THREE.Mesh(geometry, mat);
  //   mesh.rotation.x = -0.5 * Math.PI; //将模型摆正
  //   mesh.scale.set(0.1, 0.1, 0.1); //缩放
  //   geometry.center(); //居中显示

  //   console.log(geometry)
  //   scene.add(mesh);
  // });

  // 添加灯光
  const light1 = new THREE.DirectionalLight(0xffffff, 1)
  light1.position.set(0, 0, 10)
  scene.add(light1)

  const light2 = new THREE.DirectionalLight(0xffffff, 1)
  light2.position.set(0, 0, -10)
  scene.add(light2)

  const light3 = new THREE.DirectionalLight(0xffffff, 1)
  light3.position.set(10, 0, 0)
  scene.add(light3)

  const light4 = new THREE.DirectionalLight(0xffffff, 1)
  light4.position.set(-10, 0, 0)
  scene.add(light4)

  const light5 = new THREE.DirectionalLight(0xffffff, 1)
  light5.position.set(0, 10, 10)
  scene.add(light5)

})
</script>

<template>
  <div class="home">
    <div class="canvas-container" ref="canvasDom"></div>
  </div>

</template>

<style scoped>
* {
  margin: 0;
  padding: 0;
}
</style>
