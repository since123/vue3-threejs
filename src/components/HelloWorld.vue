<script setup>
import * as THREE from "three";
import { onMounted, ref, reactive } from "vue";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader.js";
// defineProps({
//   msg: String,
// })
import { STLLoader } from "three/examples/jsm/loaders/STLLoader.js";

let controls;
let canvasDom = ref(null);
// 创建场景
const scene = new THREE.Scene();
// 创建相机
const camera = new THREE.PerspectiveCamera(
  2,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(0, 1, 1);
// 创建渲染器
const renderer = new THREE.WebGLRenderer({
  // 抗锯齿
  antialias: true,
});
// 渲染器大小
renderer.setSize(window.innerWidth, window.innerHeight);
const render = () => {
  renderer.render(scene, camera);
  requestAnimationFrame(render);
};

// 修改某些部位
let wheels = [],
  carBody = [];

// 创建材质
const bodyMaterial = new THREE.MeshPhysicalMaterial({
  color: 0xff0000,
  metalness: 1,
  roughness: 0.5,
  clearcoat: 1,
  clearcoatRoughness: 0,
});
onMounted(() => {
  // 把渲染器插入dom
  canvasDom.value.appendChild(renderer.domElement);
  //初始化渲染器， 渲染背景
  renderer.setClearColor("#000");
  scene.background = new THREE.Color("#ccc");
  scene.environment = new THREE.Color("#ccc");
  render();
  // 添加网格地面
  const gridHelper = new THREE.GridHelper(10, 10);
  gridHelper.material.opacity = 0.2;
  gridHelper.material.transparent = true;
  scene.add(gridHelper);

  // 添加控制区
  controls = new OrbitControls(camera, renderer.domElement);
  controls.update();

  // 添加模型

  const loader = new GLTFLoader();
  const dracoLoader = new DRACOLoader();
  dracoLoader.setDecoderPath("./draco/gltf/");
  loader.setDRACOLoader(dracoLoader);

  loader.load("./model/电池2.glb", (gltf) => {
    const bmw = gltf.scene;

    bmw.traverse((child) => {
      if (child.isMesh) {
        console.log("child", child.name);
      }
      if (
        (child.isMesh &&
          child.name.includes(
            "图层_01(172B2667-07C9-4190-9A50-A73F38D256C8)"
          )) ||
        child.name.includes("图层_01(E03918B5-CFF5-44D8-848D-9B973E71E291)")
      ) {
        carBody = child;
        carBody.material = bodyMaterial;
      }
    });
    scene.add(bmw);
  });

  // 添加灯光
  // 平行光
  const dirLight = new THREE.DirectionalLight(0xffffff, 2);
  //光源等位置
  dirLight.position.set(1, 3, 10);
  //可以产生阴影
  dirLight.castShadow = true;
  dirLight.shadow.mapSize = new THREE.Vector2(1024, 1024);
  scene.add(dirLight);
});
</script>

<template>
  <div class="home">
    <div class="canvas-container" ref="canvasDom"></div>
  </div>
</template>

<style scoped></style>
