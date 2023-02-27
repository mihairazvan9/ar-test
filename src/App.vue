<script setup>
  import { RouterLink, RouterView } from 'vue-router'
  import {onMounted} from 'vue'
  import * as THREE from 'three'
  onMounted(() => {
    // Create a Three.js scene
    let scene = new THREE.Scene();
    let camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    let renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    // Add a cube to the scene
    let cubeGeometry = new THREE.BoxGeometry(1, 1, 1);
    let cubeMaterial = new THREE.MeshNormalMaterial();
    let cube = new THREE.Mesh(cubeGeometry, cubeMaterial);
    scene.add(cube);

    // Set up the AR environment
    navigator.xr.addEventListener('devicechange', checkXRSupport);
    checkXRSupport();

    async function checkXRSupport() {
      const xrSupported = await navigator.xr.isSessionSupported('immersive-ar');
      if (xrSupported) {
        // Create a WebXR AR session
        const xrSession = await navigator.xr.requestSession('immersive-ar');
        const xrReferenceSpace = await xrSession.requestReferenceSpace('local');
        const xrViewerPose = await xrSession.requestAnimationFrame(onXRFrame);

        // Set the camera position based on the viewer pose
        const position = xrViewerPose.transform.position;
        const quaternion = xrViewerPose.transform.orientation;
        camera.position.set(position.x, position.y, position.z);
        camera.quaternion.set(quaternion.x, quaternion.y, quaternion.z, quaternion.w);

        // Render the scene
        function onXRFrame(time, xrFrame) {
          const pose = xrFrame.getViewerPose(xrReferenceSpace);
          if (pose) {
            const view = pose.views[0];
            const viewport = xrSession.renderState.baseLayer.getViewport(view);
            renderer.setSize(viewport.width, viewport.height);
            renderer.setPixelRatio(window.devicePixelRatio);
            xrSession.requestAnimationFrame(onXRFrame);
            cube.position.set(0, 0, -1).applyMatrix4(view.transform.inverse.matrix);
            renderer.render(scene, camera);
          }
        }
      }
    }
  })
</script>

<template>
  <header>

  </header>

</template>

<style scoped>
header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style>
