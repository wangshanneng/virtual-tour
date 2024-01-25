<template>
  <div class="container" ref="container"></div>
</template>

<script setup>
import gsap from "gsap";
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

  // 创建客厅
  let liveroom = new Room("客厅", 0, "./img/livingroom/");

  // 创建厨房
  let kitchenPosition = new THREE.Vector3(-5, 0, -10);
  let kitEuler = new THREE.Euler(0, -Math.PI / 2, 0);
  let kitchen = new Room(
    "厨房",
    3,
    "./img/kitchen/",
    kitchenPosition,
    kitEuler
  );

  // 创建厨房的文字
  let kitchenTextPosition = new THREE.Vector3(-1, 0, -3);
  let kitchenText = new SpriteText("厨房", kitchenTextPosition);

  kitchenText.onClick(() => {
    gsap.to(camera.position, {
      x: kitchenPosition.x,
      y: kitchenPosition.y,
      z: kitchenPosition.z,
      duration: 1,
    });
  });
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
      let texture = new THREE.TextureLoader().load(textureUrl + item + ".jpg");
      // 处理图片角度问题
      if (item === `${roomIndex}_d` || item === `${roomIndex}_u`) {
        texture.rotation = Math.PI;
        texture.center = new THREE.Vector2(0.5, 0.5);
      }

      boxMaterials.push(new THREE.MeshBasicMaterial({ map: texture }));
    });

    const cube = new THREE.Mesh(geometry, boxMaterials);
    cube.position.copy(position);
    cube.rotation.copy(eular);
    scene.add(cube);
  }
}

class SpriteText {
  constructor(text, position) {
    this.callbacks = [];

    const canvas = document.createElement("canvas");
    canvas.width = 1024;
    canvas.height = 1024;

    const context = canvas.getContext("2d");
    context.fillStyle = "rgba(100, 100, 100, 0.7)";
    context.fillRect(0, 265, 1024, 512);
    context.textAlign = "center";
    context.textBaseline = "middle";
    context.font = "bold 200px arial";
    context.fillStyle = "white";
    context.fillText(text, 512, 512);
    let texture = new THREE.CanvasTexture(canvas);

    const material = new THREE.SpriteMaterial({
      map: texture,
      transparent: true,
    });

    const sprite = new THREE.Sprite(material);
    sprite.scale.set(0.5, 0.5, 0.5);
    this.sprite = sprite;
    sprite.position.copy(position);
    scene.add(sprite);

    let mouse = new THREE.Vector2();
    let raycaster = new THREE.Raycaster();

    window.addEventListener("click", (event) => {
      mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
      mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
      raycaster.setFromCamera(mouse, camera);
      let intersection = raycaster.intersectObject(sprite);
      if (intersection.length > 0) {
        this.callbacks.forEach((callback) => {
          callback();
        });
      }
    });
  }
  onClick(callback) {
    this.callbacks.push(callback);
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
