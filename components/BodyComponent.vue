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
  data() {
    return {
      fov: 5,
      camera: null,
      scene: null,
      renderer: null,
      lastScrollTop: 0,
    };
  },
  mounted() {

    // Create a scene
    const scene = new THREE.Scene();
    const ambientLight = new THREE.AmbientLight(0xffffff, 1.5);
    scene.add(ambientLight);

    // Create a camera
    const camera = new THREE.PerspectiveCamera(this.fov, window.innerWidth / window.innerHeight, 0.1, 10);
    camera.position.z = 5;
    camera.position.x = 0;

    // Create a renderer
    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.getElementById('three-container').appendChild(renderer.domElement);

    //add grid helper
    const gridHelper = new THREE.GridHelper(100, 100, 0xaec6cf, 0xaec6cf)
    scene.add(gridHelper)

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

    window.addEventListener('resize', onWindowResize, false)
    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight
      camera.updateProjectionMatrix()
      renderer.setSize(window.innerWidth, window.innerHeight)
      this.renderer.render()
    }

    //animate scene
    const animate = () => {
      requestAnimationFrame(animate);
      //to do add rotation and animation
      //if user scrolls down, rotate the model
      console.log(scrollY, scrollX)
      console.log(this.lastScrollTop, 'last scroll top')
      if (scrollY > this.lastScrollTop) {
        scene.rotateX(0.01);
        this.lastScrollTop = scrollY <= 0 ? 0 : scrollY;
      }
      else {
        scene.rotateY(0.01);
        this.lastScrollTop = scrollY <= 0 ? 0 : scrollY;
      }
      renderer.render(scene, camera);
    };

    // Add a scroll event listener
    animate(); // Call the animate function

  },
  methods: {
    animate() {
      requestAnimationFrame(this.animate);
      // Add your rotation and animation logic here
      this.renderer.setClearColor(0x000000, 0); // Use a clear color with alpha set to 0
      document.getElementById('three-container').appendChild(renderer.domElement);
      this.renderer.render(this.scene, this.camera);
    },
  }
};
</script>
