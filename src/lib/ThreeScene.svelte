<script>
	import { onMount } from 'svelte';
	import * as THREE from 'three';
	import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
	import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

	let container;
	let model;
	let waterMaterial;

	onMount(() => {
		const scene = new THREE.Scene();
		const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
		const renderer = new THREE.WebGLRenderer({ antialias: true });
		renderer.setSize(window.innerWidth, window.innerHeight);
		container.appendChild(renderer.domElement);

		const controls = new OrbitControls(camera, renderer.domElement);
		controls.enableDamping = true; // Enable damping (inertia)
		controls.dampingFactor = 0.25; // Damping factor
		controls.screenSpacePanning = false; // Disable panning

		const loader = new GLTFLoader();
		loader.load('/models/untitled.gltf', (gltf) => {
			model = gltf.scene;
			scene.add(model);

			// Create water material using GLSL
			waterMaterial = new THREE.ShaderMaterial({
				uniforms: {
					time: { value: 1.0 },
					resolution: { value: new THREE.Vector2() }
				},
				vertexShader: `
     uniform float time;
     varying vec2 vUv;
     varying float vWave;
     void main() {
      vUv = uv;
      vec3 pos = position;
      pos.z += sin(pos.x * 10.0 + time) * 0.1;
      pos.z += sin(pos.y * 10.0 + time) * 0.1;
      vWave = pos.z;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
     }
    `,
				fragmentShader: `
     uniform float time;
     varying vec2 vUv;
     varying float vWave;
     void main() {
      vec2 p = vUv;
      float color = 0.0;
      color += sin(p.x * 10.0 + time) * 0.5 + 0.5;
      color += sin(p.y * 10.0 + time) * 0.5 + 0.5;
      color *= vWave;
      gl_FragColor = vec4(0.0, 0.0, color, 1.0); // Blue color
     }
    `
			});

			// Apply water material to the model
			model.traverse((child) => {
				if (child.isMesh) {
					child.material = waterMaterial;
				}
			});
		}, undefined, (error) => {
			console.error('An error happened', error);
		});

		// Add more lights to the scene
		const hemisphereLight = new THREE.HemisphereLight(0xffffff, 0x444444, 1.5);
		hemisphereLight.position.set(0, 200, 0);
		scene.add(hemisphereLight);

		const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
		directionalLight.position.set(0, 200, 100);
		scene.add(directionalLight);

		// Add ambient light to brighten the background
		const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
		scene.add(ambientLight);

		camera.position.set(0, 1.6, 3);

		const animate = () => {
			requestAnimationFrame(animate);
			if (waterMaterial) {
				waterMaterial.uniforms.time.value += 0.05; // Update time uniform
			}
			controls.update(); // Update controls
			renderer.render(scene, camera);
		};
		animate();
	});
</script>

<div bind:this={container} class="three-container"></div>

<style>
    .three-container {
        width: 100%;
        height: 100vh;
    }
</style>