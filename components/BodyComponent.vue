<template>
  <div id="three-container"></div>
</template>

<script>
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { UnrealBloomPass } from 'three/examples/jsm/postprocessing/UnrealBloomPass.js';
import {EffectComposer} from "three/addons/postprocessing/EffectComposer";
import {RenderPass} from "three/addons/postprocessing/RenderPass";
import {OutputPass} from "three/addons/postprocessing/OutputPass";
import {GUI} from "three/addons/libs/lil-gui.module.min";
import {GlitchPass} from "three/addons/postprocessing/GlitchPass";
import {SSAOPass} from "three/addons/postprocessing/SSAOPass";
import {LUTPass} from "three/addons/postprocessing/LUTPass";
import {BlendShader} from "three/addons/shaders/BlendShader";
import {ShaderPass} from "three/addons/postprocessing/ShaderPass";
import {SavePass} from "three/addons/postprocessing/SavePass";
import {CopyShader} from "three/addons/shaders/CopyShader";

export default {
  data() {
    return {
      fov: 5,
      camera: null,
      model: null,
    };
  },
  methods: {
    getDebuggerLine(position, color) {
      const points = [];
      points.push( new THREE.Vector3( 0, 0, 0 ) );
      const debuggerMaterial = new THREE.LineBasicMaterial( { color } );
      const debuggerGeometry = new THREE.BufferGeometry().setFromPoints( points );
      points.push(position);
      debuggerGeometry.setFromPoints(points);
      return new THREE.Line( debuggerGeometry, debuggerMaterial );
    }
  },
  mounted() {
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(this.fov, (window.innerWidth) / (window.innerHeight), 0.1, 10);
    camera.position.z = 5;
    camera.position.x = 0;

    const ambientLight = new THREE.AmbientLight(0xffffff);
    scene.add(ambientLight);

    const pointLight = new THREE.PointLight(0xffffff, 12);
    pointLight.position.set(1, 1, 1);

    const pointLight2 = new THREE.PointLight(0xffffff, 6);
    pointLight2.position.set(-1, 1, -1);

    scene.add(camera);
    scene.add(pointLight);
    scene.add(pointLight2);

    const controlParams = {
      orbit: true,
      orbitSpeed: 0.005,
      shakeIntensity: 0.02
    }

    const bloomParams = {
      enabled: true,
      threshold: 0.62,
      strength: 0.23,
      radius: 0.27,
    }

    const toneMappingParams = {
      toneMapping: THREE.ACESFilmicToneMapping,
      exposure: 1.00
    };

    const glitchParams = {
      enabled: false,
      goWild: false,
      threshold: 0.5,
      curF: 0.5,
    }

    const ssaoParams = {
      enabled: true,
      kernelRadius: 16,
      minDistance: 0.005,
      maxDistance: 0.1,
    }

    const lutParams = {
      enabled: false,
      intensity: 1.0,
    }

    const blurParams = {
      mixRatio: 0.1
    }

    const bloomPass = new UnrealBloomPass(
        new THREE.Vector2( window.innerWidth, window.innerHeight ),
        bloomParams.strength,
        bloomParams.radius,
        bloomParams.threshold
    );
    bloomPass.enabled = bloomParams.enabled

    let renderer = new THREE.WebGLRenderer({
      alpha: true,
      antialias: true,
    });

    const savePass = new SavePass(new THREE.WebGLRenderTarget(window.innerWidth, window.innerHeight, {
      minFilter: THREE.LinearFilter,
      magFilter: THREE.LinearFilter,
      format: THREE.RGBAFormat
    }));

    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.toneMapping = toneMappingParams.toneMapping;
    renderer.toneMappingExposure = toneMappingParams.exposure;


    const renderScene  = new RenderPass( scene, camera );
    const outputPass = new OutputPass(CopyShader);
    outputPass.renderToScreen = true;

    const composer = new EffectComposer( renderer );
    const glitchPass = new GlitchPass();

    glitchPass.enabled = glitchParams.enabled;
    glitchPass.goWild = glitchParams.goWild;
    glitchPass.threshold = glitchParams.threshold;
    glitchPass.curF = glitchParams.curF;

    const ssaoPass = new SSAOPass( scene, camera );
    ssaoPass.enabled = ssaoParams.enabled;
    ssaoPass.kernelRadius = ssaoParams.kernelRadius;
    ssaoPass.minDistance = ssaoParams.minDistance;
    ssaoPass.maxDistance = ssaoParams.maxDistance;

    const lutPass = new LUTPass();
    lutPass.enabled = lutParams.enabled;
    lutPass.intensity = lutParams.intensity;

    const blendPass = new ShaderPass(BlendShader, "tDiffuse1");
    blendPass.uniforms["tDiffuse2"].value = savePass.renderTarget.texture;
    blendPass.uniforms["mixRatio"].value = blurParams.mixRatio;

    composer.addPass( renderScene );
    composer.addPass( ssaoPass );
    composer.addPass( bloomPass );
    composer.addPass( lutPass );
    composer.addPass( blendPass );
    composer.addPass( glitchPass );
    composer.addPass( savePass );
    composer.addPass( outputPass );

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
    controls.enableZoom = true;
    controls.enablePan = false;
    controls.minDistance = 2;
    controls.maxDistance = 5;
    controls.maxPolarAngle = Math.PI * 0.5;

    const gui = new GUI();

    const controlFolder = gui.addFolder( 'Controls' );

    controlFolder.add( controlParams, 'orbit' ).onChange( function ( value ) {
      controlParams.orbit = value;
    } );

    controlFolder.add( controlParams, 'orbitSpeed', 0.0, 1.0 ).step( 0.001 ).onChange( function ( value ) {
      controlParams.orbitSpeed = Number( value );
    } );

    controlFolder.add( controlParams, 'shakeIntensity', 0.0, 0.05 ).step( 0.001 ).onChange( function ( value ) {
      controlParams.shakeIntensity = Number( value );
    } );

    const bloomFolder = gui.addFolder( 'Bloom' );

    bloomFolder.add( bloomParams, 'enabled' ).onChange( function ( value ) {
      bloomPass.enabled = value;
    } );

    bloomFolder.add( bloomParams, 'threshold', 0.0, 1.0 ).step( 0.01 ).onChange( function ( value ) {
      bloomPass.threshold = Number( value );
    } );

    bloomFolder.add( bloomParams, 'strength', 0.0, 2.0 ).onChange( function ( value ) {
      bloomPass.strength = Number( value );
    } );

    bloomFolder.add( bloomParams, 'radius', 0.0, 1.0 ).step( 0.01 ).onChange( function ( value ) {
      bloomPass.radius = Number( value );
    } );

    const glitchFolder = gui.addFolder( 'Glitch' );

    glitchFolder.add( glitchParams, 'enabled' ).onChange( function ( value ) {
      glitchPass.enabled = value;
    } );

    glitchFolder.add( glitchParams, 'goWild' ).onChange( function ( value ) {
      glitchPass.goWild = value;
    } );

    glitchFolder.add( glitchParams, 'threshold', 0.0, 1.0 ).step( 0.01 ).onChange( function ( value ) {
      glitchPass.threshold = Number( value );
    } );

    glitchFolder.add( glitchParams, 'curF', 0.0, 1.0 ).step( 0.01 ).onChange( function ( value ) {
      glitchPass.curF = Number( value );
    } );

    const ssaoFolder = gui.addFolder( 'SSAO' );

    ssaoFolder.add( ssaoParams, 'enabled' ).onChange( function ( value ) {
      ssaoPass.enabled = value;
    } );

    ssaoFolder.add( ssaoParams, 'kernelRadius', 0.0, 1.0 ).step( 0.01 ).onChange( function ( value ) {
      ssaoPass.kernelRadius = Number( value );
    } );

    ssaoFolder.add( ssaoParams, 'minDistance', 0.0, 1.0 ).step( 0.01 ).onChange( function ( value ) {
      ssaoPass.minDistance = Number( value );
    } );

    ssaoFolder.add( ssaoParams, 'maxDistance', 0.0, 1.0 ).step( 0.01 ).onChange( function ( value ) {
      ssaoPass.maxDistance = Number( value );
    } );

    const lutFolder = gui.addFolder( 'LUT' );

    lutFolder.add( lutParams, 'enabled' ).onChange( function ( value ) {
      lutPass.enabled = value;
    } );

    lutFolder.add( lutParams, 'intensity', 0.0, 10.0 ).step( 0.01 ).onChange( function ( value ) {
      lutPass.intensity = Number( value );
    } );

    const blurFolder = gui.addFolder( 'Blur' );

    blurFolder.add( blurParams, 'mixRatio', 0.0, 1.0 ).step( 0.01 ).onChange( function ( value ) {
      blendPass.uniforms["mixRatio"].value = Number( value );
    } );

    const toneMappingFolder = gui.addFolder( 'ToneMapping' );

    toneMappingFolder.add( toneMappingParams, 'toneMapping', {
      No: THREE.NoToneMapping,
      Linear: THREE.LinearToneMapping,
      Reinhard: THREE.ReinhardToneMapping,
      Cineon: THREE.CineonToneMapping,
      ACESFilmic: THREE.ACESFilmicToneMapping
    } ).onChange( function ( value ) {
      renderer.toneMapping = Number( value );
    } );

    toneMappingFolder.add( toneMappingParams, 'exposure', 0.0, 2.0 ).step( 0.01 ).onChange( function ( value ) {
      renderer.toneMappingExposure = Math.pow( value, 4.0 );
    } );

    window.addEventListener('resize', onWindowResize, false);

    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth - (window.innerWidth * 0.2), window.innerHeight - (window.innerHeight * 0.2));
    }

    const animate = () => {
      requestAnimationFrame(animate);

      if (this.model) {
        // Rotate the model
        if (controlParams.orbit) {
          this.model.rotation.y += controlParams.orbitSpeed;
        }

        this.model.position.x = Math.sin(Date.now() * 0.001) * Math.random() * controlParams.shakeIntensity;
        this.model.position.y = Math.sin(Date.now() * 0.001) * Math.random() * controlParams.shakeIntensity;
        this.model.position.z = Math.sin(Date.now() * 0.001) * Math.random() * controlParams.shakeIntensity;
      }

      composer.render()
    };

    animate();
  },
};
</script>
