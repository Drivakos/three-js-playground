<template>
  <div id="three-container"></div>
</template>

<script>
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

export default {
  data() {
    return {
      fov: 5,
      camera: null,
      model: null,
    };
  },
  mounted() {
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(this.fov, window.innerWidth / window.innerHeight, 0.1, 10);
    camera.position.z = 5;
    camera.position.x = 0;

    const ambientLight = new THREE.AmbientLight(0xffffff);
    scene.add(ambientLight);

    const pointLight = new THREE.PointLight(0xffffff, 15);
    camera.add(pointLight);
    scene.add(camera);

    let renderer = new THREE.WebGLRenderer({
      alpha: true,
      antialias: true,
    });

    renderer.setSize(window.innerWidth, window.innerHeight);
    document.getElementById('three-container').appendChild(renderer.domElement);

    const loader = new GLTFLoader();

    loader.load('/3dModels/nike_air_jordan_1/scene.gltf', (gltf) => {
      const model = gltf.scene;
      model.scale.set(0.8, 0.8, 0.8);
      scene.add(model);
      this.model = model;
    }, undefined, (error) => {
      console.error(error);
    });

    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enableZoom = false;
    controls.minDistance = 2;
    controls.maxDistance = 5;

    window.addEventListener('resize', onWindowResize, false);
    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    }

    const animate = () => {
      requestAnimationFrame(animate);

      // Rotate the model
      if (this.model) {
        this.model.rotation.y += 0.005;
      }

      renderer.render(scene, camera);
    };

    animate();
  },
};
</script>
