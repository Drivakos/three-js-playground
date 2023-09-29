<template>
  <div class="body-component-wrapper">
    <div id="about">
      Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec euismod, nisl eget ultricies ultrices, nunc
      Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec euismod, nisl eget ultricies ultrices, nunc
      Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec euismod, nisl eget ultricies ultrices, nunc
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
    const camera = new THREE.PerspectiveCamera(5, window.innerWidth / window.innerHeight, 0.1, 10);
    camera.position.z = 5;
    camera.position.x = 0;

    // Create a renderer
    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.getElementById('three-container').appendChild(renderer.domElement);
    //load the model
    const loader = new GLTFLoader();

    loader.load('/3dModels/nike_air_jordan_1/scene.gltf', function (gltf) {
      const model = gltf.scene;
      model.scale.set(0.9, 0.9, 0.9); // Adjust the scale as needed
      model.rotateX(0.6);
      model.rotateY(0.9);
      scene.add(model);
    }, undefined, function (error) {
      console.error(error);
    });

    THREE.Cache.enabled = true;
    //adding random cube in order to understand physics :P
    const geometry = new THREE.BoxGeometry(100, 3, 16, 100);
    const material = new THREE.MeshStandardMaterial({color: 0x00ff00, wireframe: true});
    const cube = new THREE.Mesh(geometry, material);
    scene.add(cube);

    //animate scene
    const animate = () => {
      requestAnimationFrame(animate);
      //to do add rotation and animation
      scene.rotation.y += 0.01;
      cube.rotation.x += 0.01;
      renderer.render(scene, camera);
    };

    animate(); // Call the animate function
  },
};
</script>
