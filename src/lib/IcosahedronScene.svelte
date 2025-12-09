<script>
  import { onMount } from 'svelte';
  // @ts-ignore
  import * as THREE from 'three';
  // 1. Import des contrôles
  // @ts-ignore
  import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

  let canvasContainer;

  onMount(() => {
    // --- Initialisation Scène & Caméra ---
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x242424);

    const camera = new THREE.PerspectiveCamera(
      75, 
      canvasContainer.clientWidth / canvasContainer.clientHeight, 
      0.1, 
      1000
    );
    // @ts-ignore
    camera.position.z = 3;

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(canvasContainer.clientWidth, canvasContainer.clientHeight);
    canvasContainer.appendChild(renderer.domElement);

    // --- Contrôles (Ajout) ---
    // 2. Création des contrôles liés à la caméra et au canvas
    // @ts-ignore
    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true; // Donne une inertie fluide à la rotation
    controls.dampingFactor = 0.05;
    controls.enableZoom = true;    // Permet de zoomer (molette)
    controls.enablePan = false;    // Désactive le déplacement latéral (optionnel, souvent mieux pour un globe)
    
    // --- Icosaèdre ---
    const geometry = new THREE.IcosahedronGeometry(1, 0);
    const material = new THREE.MeshBasicMaterial({ 
      color: 0x646cff, 
      wireframe: true 
    });
    const icosahedron = new THREE.Mesh(geometry, material);
    // @ts-ignore
    scene.add(icosahedron);

    // --- Lumières ---
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
    scene.add(ambientLight);
    const pointLight = new THREE.PointLight(0xffffff, 1);
    // @ts-ignore
    pointLight.position.set(5, 5, 5);
    scene.add(pointLight);

    // --- Boucle d'animation ---
    let frameId;
    const animate = () => {
      frameId = requestAnimationFrame(animate);

      // 3. Mise à jour des contrôles à chaque frame (nécessaire pour le damping)
      controls.update();

      // J'ai retiré la rotation automatique pour vous laisser le contrôle manuel
      // icosahedron.rotation.x += 0.005; 

      renderer.render(scene, camera);
    };
    animate();

    // --- Redimensionnement ---
    const handleResize = () => {
      if (!canvasContainer) return;
      const width = canvasContainer.clientWidth;
      const height = canvasContainer.clientHeight;
      
      renderer.setSize(width, height);
      camera.aspect = width / height;
      camera.updateProjectionMatrix();
    };
    window.addEventListener('resize', handleResize);

    // --- Nettoyage ---
    return () => {
      window.removeEventListener('resize', handleResize);
      cancelAnimationFrame(frameId);
      
      controls.dispose(); // Bien penser à nettoyer les contrôles
      geometry.dispose();
      material.dispose();
      renderer.dispose();
      if (canvasContainer && renderer.domElement) {
        canvasContainer.removeChild(renderer.domElement);
      }
    };
  });
</script>

<div class="scene-container" bind:this={canvasContainer}></div>

<style>
  .scene-container {
    width: 100%;
    height: 600px; /* J'ai agrandi un peu la zone pour mieux naviguer */
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 4px 6px rgba(0,0,0,0.3);
    cursor: grab; /* Change le curseur pour indiquer qu'on peut attraper */
  }
  
  .scene-container:active {
    cursor: grabbing;
  }
</style>