<template>
  <div ref="container"></div>
  <div class="color_list">
    <div
      v-for="(item, index) in colorList"
      :key="index"
      :style="{ background: item }"
      class="color_item"
      @click="handleClickByColor(item)">
      <div class="color_item_inner" :style="{ background: item }"></div>
    </div>
  </div>
</template>

<script setup lang="ts">
import * as THREE from "three";
// 引入gltf模型加载库GLTFLoader.js
import { GLTF, GLTFLoader } from "three/addons/loaders/GLTFLoader.js";
import { OrbitControls } from "three/examples/jsm/Addons.js";
import { onMounted, ref } from "vue";
let scene: THREE.Scene, renderer: THREE.WebGLRenderer, camera: THREE.PerspectiveCamera, controls: OrbitControls, gltfModel: GLTF;
// 创建GLTF加载器对象
const loader: GLTFLoader = new GLTFLoader();
let container = ref();

// 创建场景
const setScene = () => {
  scene = new THREE.Scene();
  renderer = new THREE.WebGLRenderer();
  renderer.setSize(innerWidth, innerHeight);
  (container.value as HTMLElement).appendChild(renderer.domElement);
};
//创建相机
const setCamera = () => {
  camera = new THREE.PerspectiveCamera(75, innerWidth / innerHeight, 1, 1000);
  camera.position.set(0, 2, 5);
};
// 加载模型
const loaderGLTF = () => {
  loader.load("/glb/Lamborghini.glb", function (gltf) {
    gltfModel = gltf;
    console.log("控制台查看加载gltf文件返回的对象结构", gltf);
    console.log("gltf对象场景属性", gltf.scene);
    // 返回的场景对象gltf.scene插入到threejs场景中
    scene.add(gltf.scene);
    renderer.render(scene, camera); // 加载后立即渲染一次
  });
};
// 光线
const setAmbientLight = () => {
  const light = new THREE.AmbientLight(0xffffff); // 柔和的白光
  scene.add(light);
};

// 控制器
const setControls = () => {
  // 轨道控制器
  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true; // 阻尼惯性
  controls.enableZoom = true; // 允许缩放
  controls.enablePan = true; // 允许平移
  controls.enableRotate = true; // 允许旋转
  controls.minDistance = 1; // 最小距离
  controls.maxDistance = 100; // 最大距离
  controls.maxPolarAngle = Math.PI / 2; // 最大角度
  controls.minPolarAngle = Math.PI / 4; // 最小角度
};

// 渲染
let animate = function () {
  requestAnimationFrame(animate);
  // 更新控制器
  controls.update();
  renderer.render(scene, camera);
};

// 颜色
const colorList = [
  "rgb(216, 27, 67)",
  "rgb(142, 36, 170)",
  "rgb(81, 45, 168)",
  "rgb(48, 63, 159)",
  "rgb(30, 136, 229)",
  "rgb(0, 137, 123)",
  "rgb(67, 160, 71)",
  "rgb(251, 192, 45)",
  "rgb(245, 124, 0)",
  "rgb(230, 74, 25)",
  "rgb(233, 30, 78)",
  "rgb(156, 39, 176)",
  "rgb(0, 0, 0)",
]; // 车身颜色数组

const handleClickByColor = (color: string) => {
  const newColor = new THREE.Color(color);
  gltfModel.scene.traverse((child) => {
    // 判断是网格材质，并且有颜色属性 后续还需要判断模型模块的名称
    if (child instanceof THREE.Mesh && "color" in child.material) {
      child.material.color.set(newColor);
      child.material.needsUpdate = true; // 确保材质更新
    }
  });
};

onMounted(() => {
  setScene();
  setCamera();
  loaderGLTF();
  setAmbientLight();
  setControls();
  animate();
});
</script>

<style scoped>
.color_list {
  position: fixed;
  top: 50px;
  right: 0;
  width: 80px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.color_item {
  cursor: pointer;
  width: 30px;
  height: 30px;
  border-radius: 50%;
}
</style>
