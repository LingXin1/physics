<script setup>
import { ref, onMounted } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";

// ? 物理库
import * as CANNON from "cannon-es";

import GUI from "lil-gui";
import gsap from "gsap";

const canvas = ref(null);

let scene, world, camera, renderer, control;

const audio = new Audio("./audio/hit.mp3");

// ! 场景与物理
scene = new THREE.Scene();
world = new CANNON.World({
  gravity: new CANNON.Vec3(0, -9.82, 0),
  allowSleep: true,
});
world.broadphase = new CANNON.SAPBroadphase(world);
const base = new CANNON.Material();
world.defaultContactMaterial = new CANNON.ContactMaterial(base, base, {
  friction: 0.1,
  restitution: 0.8,
});
scene.add(new THREE.AxesHelper(1));

// ! 添加球体函数
const addsphere = () => {
  // ? 球体半径
  const radius = Math.random() * (0.5 - 0.1);
  // ? 球体位置
  const sphereX = Math.random() * (1 + 1) - 1;
  const sphereZ = Math.random() * (1 + 1) - 1;

  spheres(radius, { x: sphereX, y: 4, z: sphereZ });
};
// ! 添加盒子
const addbox = () => {
  // ? 盒子大小
  const size = Math.random() * (0.8 - 0.1);
  // ? 盒子位置
  const sphereX = Math.random() * (1 + 1) - 1;
  const sphereZ = Math.random() * (1 + 1) - 1;
  boxs(size, { x: sphereX, y: 4, z: sphereZ });
};

const fun = {
  addsphere,
  addbox,
};
const gui = new GUI();
gui.add(fun, "addsphere");
gui.add(fun, "addbox");

// ! 灯光
const directional = new THREE.DirectionalLight("#fff", 2);
directional.castShadow = true;
directional.shadow.mapSize.set(2048, 2048);
directional.shadow.camera.far = 20;
directional.position.set(5, 5, 5);
scene.add(directional);

// ! 相机
camera = new THREE.PerspectiveCamera(
  45,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(5, 5, 5);

// ! 地面
const paln = new THREE.Mesh(
  new THREE.PlaneGeometry(10, 10),
  new THREE.MeshStandardMaterial({
    side: THREE.DoubleSide,
  })
);
paln.receiveShadow = true;
paln.rotation.x = -Math.PI / 2;

const physicsplan = new CANNON.Body({
  mass: 0,
  shape: new CANNON.Plane(),
});
physicsplan.quaternion.setFromEuler(-Math.PI / 2, 0, 0);

world.addBody(physicsplan);
scene.add(paln);

const shapes = [];
// ! 球体函数
const spheres = (radius, position) => {
  // todo 创造球
  const sphere = new THREE.Mesh(
    new THREE.SphereGeometry(radius, 64, 32),
    new THREE.MeshStandardMaterial({
      roughness: 0.4,
      metalness: 0.6,
    })
  );
  sphere.castShadow = true;
  sphere.position.set(position.x, position.y, position.z);
  // todo 创造物理

  const physicssphere = new CANNON.Body({
    mass: 0.5,
    shape: new CANNON.Sphere(radius),
    position: new CANNON.Vec3(position.x, position.y, position.z),
  });

  physicsplan.addEventListener("collide", (e) => {
    const impact = e.contact.getImpactVelocityAlongNormal();
    if (impact > 1.5) {
      audio.volume = Math.random();
      audio.currentTime = 0;
      audio.play();
    }
  });

  world.addBody(physicssphere);
  scene.add(sphere);

  shapes.push({
    shape: sphere,
    physicsshape: physicssphere,
  });
};

const boxss = [];
const boxs = (size, position) => {
  // todo 创造盒子
  const box = new THREE.Mesh(
    new THREE.BoxGeometry(size, size, size),
    new THREE.MeshStandardMaterial({
      roughness: 0.4,
      metalness: 0.6,
    })
  );
  box.castShadow = true;
  box.position.set(position.x, position.y, position.z);
  // todo 创造物理

  const physicsbox = new CANNON.Body({
    mass: 0.5,
    shape: new CANNON.Box(new CANNON.Vec3(size / 2, size / 2, size / 2)),
    position: new CANNON.Vec3(position.x, position.y, position.z),
  });

  physicsbox.addEventListener("collide", (e) => {
    const impact = e.contact.getImpactVelocityAlongNormal();
    if (impact > 1.5) {
      audio.volume = Math.random();
      audio.currentTime = 0;
      audio.play();
    }
  });

  world.addBody(physicsbox);
  scene.add(box);

  boxss.push({
    shape: box,
    physicsshape: physicsbox,
  });
};

// ! 动画函数
const clock = new THREE.Clock();
const tick = () => {
  const delta = clock.getDelta();
  requestAnimationFrame(tick);
  control.update();

  world.step(1 / 60, delta, 3);
  shapes.forEach((item) => {
    item.shape.position.copy(item.physicsshape.position);
  });
  boxss.forEach((item) => {
    item.shape.position.copy(item.physicsshape.position);
    item.shape.quaternion.copy(item.physicsshape.quaternion);
  });
  renderer.render(scene, camera);
};

// ! 初始化
onMounted(() => {
  // ! 轨道控制器
  control = new OrbitControls(camera, canvas.value);
  // todo 渲染器
  renderer = new THREE.WebGLRenderer({
    canvas: canvas.value,
    alpha: true,
    antialias: true,
  });
  renderer.shadowMap.enabled = true;
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.render(scene, camera);
  tick();
});

// ! 自适应
window.addEventListener("resize", () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
</script>

<template>
  <canvas id="canvas" ref="canvas"></canvas>
</template>

<style scoped lang="less">
#canvas {
  position: fixed;
  top: 0;
}
</style>
