<template>
  <div class="container" ref="container"></div>
</template>

<script setup>
import * as THREE from "three";
import { onMounted, ref } from "vue";

// 初始化场景
const scene = new THREE.Scene();

// 初始化相机
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
// 设置相机位置
camera.position.set(0, 0, 0);
// 初始化渲染器
const renderer = new THREE.WebGLRenderer({
  antialias: true,
  alpha: true,
  logarithmicDepthBuffer: true,
});
renderer.setSize(window.innerWidth, window.innerHeight);

// 辅助坐标系
const axes = new THREE.AxesHelper(5);
scene.add(axes);

const container = ref(null);

const render = () => {
  renderer.render(scene, camera);
  requestAnimationFrame(render);
};

onMounted(() => {
  container.value.appendChild(renderer.domElement);
  render();

  let isMouseDown = false;
  container.value.addEventListener("mousedown", () => {
    isMouseDown = true;
  });
  container.value.addEventListener("mouseup", () => {
    isMouseDown = false;
  });
  container.value.addEventListener("mouseout", () => {
    isMouseDown = false;
  });
  container.value.addEventListener("mousemove", (event) => {
    if (isMouseDown) {
      camera.rotation.y += event.movementX * 0.002;
      camera.rotation.x += event.movementY * 0.002;
      camera.rotation.order = "YXZ";
    }
  });

  let liveroom = new Room("客厅", 0, "./img/livingroom/");
});
class Room {
  constructor(
    name,
    roomIndex,
    textureUrl,
    position = new THREE.Vector3(0, 0, 0),
    eular = new THREE.Euler(0, 0, 0)
  ) {
    this.name = name;
    // 创建立方体
    const geometry = new THREE.BoxGeometry(10, 10, 10);
    geometry.scale(1, 1, -1);

    let arr = [
      `${roomIndex}_l`,
      `${roomIndex}_r`,
      `${roomIndex}_u`,
      `${roomIndex}_d`,
      `${roomIndex}_b`,
      `${roomIndex}_f`,
    ];

    let boxMaterials = [];
    arr.forEach((item) => {
      boxMaterials.push(
        new THREE.MeshBasicMaterial({
          map: new THREE.TextureLoader().load(textureUrl + item + ".jpg"),
        })
      );
    });

    const cube = new THREE.Mesh(geometry, boxMaterials);
    cube.position.copy(position);
    scene.add(cube);
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
}

.container {
  height: 100vh;
  width: 100vw;
  background-color: #f0f0f0;
}
</style>
