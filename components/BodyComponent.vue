<template>
  <div class="body-component-wrapper">
    <div id="about">
      Evan Drivakos is a software engineer with a passion for creating and learning. He is currently a student at Uop and
      is working towards a degree in computer science. He has experience with Java, C++, C#, Python, and JavaScript.
      He is also working on a project that uses the Three.js library.
    </div>
  </div>
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
    camera.position.x = 0;

    // Create a renderer
    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.getElementById('three-container').appendChild(renderer.domElement);
    //load the model
    const loader = new GLTFLoader();

    loader.load('/3dModels/nike_shoe/scene.gltf', function (gltf) {
      const model = gltf.scene;
      model.scale.set(0.9, 0.9, 0.9); // Adjust the scale as needed
      model.rotateX(0.6);
      model.rotateY(0.9);
      scene.add(model);
    }, undefined, function (error) {
      console.error(error);
    });

    THREE.Cache.enabled = true;

    //animate scene
    const animate = () => {
      requestAnimationFrame(animate);

      // Apply the rotation to the model if it exists
      if (this.model && !this.mouseDown) {
        this.model.rotation.copy(this.modelRotation);
      }

      renderer.render(scene, camera);
    };

    animate(); // Call the animate function
  },
};
</script>
