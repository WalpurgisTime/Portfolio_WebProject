<script>
	import { onMount } from 'svelte';
	import * as THREE from 'three';
	import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
	import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

	let container;
	let model;
	let waterMaterial;
	let angle = 0;
	let targetAngle = 0;

	onMount(() => {
		const scene = new THREE.Scene();
		scene.background = new THREE.Color(0x87CEEB); // Set background color to sky blue

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
      gl_FragColor = vec4(0.0, 0.0, color, 1.0);
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
		const ambientLight = new THREE.AmbientLight(0xffffff, 1);
		scene.add(ambientLight);

		camera.position.set(0, 1.6, 3);

		const animate = () => {
			requestAnimationFrame(animate);
			if (waterMaterial) {
				waterMaterial.uniforms.time.value += 0.05; // Update time uniform
			}
			angle += (targetAngle - angle) * 0.1; // Smooth the angle change
			const a = 5; // Semi-major axis
			const b = 3; // Semi-minor axis
			const x = a * Math.cos(angle);
			const z = b * Math.sin(angle);
			if (model) {
				model.rotation.y = angle; // Rotate the model along the Y-axis
				model.position.set(x, 0, z);
			}
			controls.update(); // Update controls
			renderer.render(scene, camera);
		};
		animate();

		// Add event listener for mouse wheel
		window.addEventListener('wheel', (event) => {
			targetAngle += event.deltaY * 0.01; // Adjust the speed as needed
		});

		// Add event listener for window resize
		window.addEventListener('resize', () => {
			camera.aspect = window.innerWidth / window.innerHeight;
			camera.updateProjectionMatrix();
			renderer.setSize(window.innerWidth, window.innerHeight);
		});

		console.log('Three.js scene initialized');
	});
</script>

<div class="page-wrapper">
	<div bind:this={container} class="three-container"></div>
</div>

<style>
    .page-wrapper {
        width: 100vw;
        height: 100vh;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .three-container {
        width: 700px;
        height: 400px;
        display: flex;
        justify-content: center;
        align-items: center;
        background-color: #ffffff; /* Fond clair */
        border-radius: 12px; /* Coins arrondis */
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15); /* Ombre plus marquée */
        overflow: hidden; /* Évite les débordements visuels */
    }
</style>
