<template>
  <div id="three-container"></div>
</template>

<script>
import * as THREE from 'three';
import {GLTFLoader} from 'three/addons/loaders/GLTFLoader.js';

export default {
  mounted() {
    // Create a scene
    const scene = new THREE.Scene();
    const ambientLight = new THREE.AmbientLight(0xffffff, 1.5);
    scene.add(ambientLight);
    // Create a camera
    const camera = new THREE.PerspectiveCamera(100, window.innerWidth / window.innerHeight, 0.1, 100);
    camera.position.z = 5;

    // Create a renderer
    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.getElementById('three-container').appendChild(renderer.domElement);
    //load the model
    const loader = new GLTFLoader();

    loader.load('/3dModels/funko/scene.gltf', function (gltf) {
      const model = gltf.scene;
      model.scale.set(0.01, 0.01, 0.01); // Adjust the scale as needed
      scene.add(model);
    }, undefined, function (error) {
      console.error(error);
    });

    THREE.Cache.enabled = true;

    //animate scene
    function animate() {
      requestAnimationFrame(animate);
      renderer.render(scene, camera);
    }

    animate(); // Call the animate function to start the loop
  }
};
</script>
