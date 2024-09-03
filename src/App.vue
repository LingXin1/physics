<script setup>
import { ref, onMounted } from "vue";
import * as THREE from "three";
import GUI from "lil-gui";
import gsap from "gsap";

const canvas = ref(null);
const gii = new GUI();

let scene, camera, renderer, textloader;
let modelsarr = [];

// ! 场景
scene = new THREE.Scene();

// ! 灯光
const directional = new THREE.DirectionalLight("#fff", 2);
directional.position.set(1, 1, 0);
scene.add(directional);

// ! 相机
const cameragroup = new THREE.Group();
camera = new THREE.PerspectiveCamera(
  45,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(0, 0, 5);
cameragroup.add(camera);
scene.add(cameragroup);

// todo 加载卡通材质
textloader = new THREE.TextureLoader();
const gradientMap = textloader.load("./img/5.jpg");
gradientMap.magFilter = THREE.NearestFilter;

// todo 添加物体
const height_ratio = 4;
const toonmaterial = new THREE.MeshToonMaterial({
  color: "#fff",
  gradientMap,
});

const cone = new THREE.Mesh(
  new THREE.ConeGeometry(0.8, 1.5, 32, 6),
  toonmaterial
);
cone.position.set(2, -height_ratio * 0, 0);

const tour = new THREE.Mesh(
  new THREE.TorusGeometry(0.8, 0.2, 32),
  toonmaterial
);
tour.position.set(-2, -height_ratio * 1, 0);

const torus = new THREE.Mesh(
  new THREE.TorusKnotGeometry(0.6, 0.2, 64, 16),
  toonmaterial
);
torus.position.set(2, -height_ratio * 2, 0);

scene.add(cone, tour, torus);
modelsarr.push(cone, tour, torus);

// todo 添加粒子
const vertices = 1000;
const verticesarr = [];
for (let i = 0; i < vertices * 3; i++) {
  let index = i * 3;
  verticesarr[index] = (Math.random() - 0.5) * 2 * 3;
  verticesarr[index + 1] = (Math.random() - 0.5) * 2 * 10;
  verticesarr[index + 2] = (Math.random() - 0.5) * 2 * 2;
}
const pointgeometry = new THREE.BufferGeometry();
pointgeometry.setAttribute(
  "position",
  new THREE.Float32BufferAttribute(verticesarr, 3)
);

const point = new THREE.Points(
  pointgeometry,
  new THREE.PointsMaterial({
    size: 0.01,
    color: "#fff",
  })
);

scene.add(point);

// ! 动画函数
const clock = new THREE.Clock();
const tick = () => {
  requestAnimationFrame(tick);

  const delta = clock.getDelta();
  modelsarr.forEach((item) => {
    item.rotation.x += delta * 0.1;
    item.rotation.y += delta * 0.5;
    item.rotation.z += delta * 0.1;
  });

  renderer.render(scene, camera);
};

// ! 初始化
onMounted(() => {
  // todo 渲染器
  renderer = new THREE.WebGLRenderer({
    canvas: canvas.value,
    alpha: true,
  });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.render(scene, camera);
  tick();
});

// ! 滚动事件
let first = 0;
window.addEventListener("scroll", () => {
  const scrolltop = scrollY / window.innerHeight;
  const scrollround = Math.round(scrolltop);
  if (scrollround !== first) {
    first = scrollround;
    gsap.to(modelsarr[first].rotation, {
      duration: 1.5,
      x: "+=1.5",
      y: "+=2",
      z: "+=0.5",
    });
  }
  cameragroup.position.y = -scrolltop * height_ratio;
});

// ! 移动事件
window.addEventListener("mousemove", (e) => {
  const mouseposition = {
    x: (e.pageX / window.innerWidth - 0.5) * 2,
    y: -(e.clientY / window.innerHeight - 0.5) * 2,
  };
  camera.position.set(mouseposition.x * 0.2, mouseposition.y * 0.2, 5);
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
  <section id="main">
    <li>
      <h3>MY PORTFOLIO</h3>
    </li>
    <li>
      <h3>CALL MY PHONE</h3>
    </li>
    <li>
      <h3>CONTACT ME</h3>
    </li>
  </section>
</template>

<style scoped lang="less">
#canvas {
  position: fixed;
  top: 0;
}

#main {
  li {
    display: flex;
    align-items: center;
    width: 100%;
    height: 100vh;
    h3 {
      color: #fff;
      font-size: 4.7917vw;
    }
    &:nth-of-type(2n-1) {
      justify-content: left;
      h3 {
        margin-left: 5.2083vw;
      }
    }
    &:nth-of-type(2n) {
      justify-content: right;
      h3 {
        margin-right: 5.2083vw;
      }
    }
  }
}
</style>
