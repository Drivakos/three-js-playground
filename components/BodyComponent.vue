<template>
    <div id="three-container"></div>
</template>

<script>
import * as THREE from 'three';
import {GLTFLoader} from 'three/addons/loaders/GLTFLoader.js';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

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

    // Create a camera
    const camera = new THREE.PerspectiveCamera(this.fov, window.innerWidth / window.innerHeight, 0.1, 10);
    camera.position.z = 5;
    camera.position.x = 0;

    const ambientLight = new THREE.AmbientLight( 0xffffff );
    scene.add( ambientLight );

    const pointLight = new THREE.PointLight( 0xffffff, 15 );
    camera.add( pointLight );
    scene.add( camera );

    // Create a renderer
    let renderer = new THREE.WebGLRenderer({
      alpha: true // NOTE: only this is important for a clear background!!!
    });
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.getElementById('three-container').appendChild(renderer.domElement);

    //load the model
    const loader = new GLTFLoader();

    loader.load('/3dModels/nike_air_jordan_1/scene.gltf', function (gltf) {
      const model = gltf.scene;
      model.scale.set(0.9, 0.9, 0.9);

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
      renderer.render(scene, camera);
    };

    animate();
  }
};
</script>
