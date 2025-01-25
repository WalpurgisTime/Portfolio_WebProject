<script>
	import { onMount } from 'svelte';
	import * as THREE from 'three';
	import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
	import I_Albedo from '$lib/images/Albedo.png';
	import I_Height from '$lib/images/Height.jpg';
	import I_Normal from '$lib/images/Normal.png';

	let container;
	let displacementStrength = 0.1;

	let scene, camera, renderer, controls, sphere, material;

	function initScene() {
		scene = new THREE.Scene();
		scene.background = new THREE.Color(0x87ceeb);

		// Camera setup
		camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
		camera.position.set(0, 2, 5); // Adjusted to match the cylinder script
		camera.lookAt(0, 0, 0);

		renderer = new THREE.WebGLRenderer({ antialias: true });
		renderer.setSize(window.innerWidth, window.innerHeight);
		container.appendChild(renderer.domElement);

		// OrbitControls
		controls = new OrbitControls(camera, renderer.domElement);
		controls.enableDamping = true;


		// Sphere geometry and material
		const geometry = new THREE.SphereGeometry(1, 64, 64);

		material = new THREE.ShaderMaterial({
			uniforms: {
				uRockAlbedo: { value: new THREE.TextureLoader().load(I_Albedo) },
				uRockHeight: { value: new THREE.TextureLoader().load(I_Height) },
				uRockNormal: { value: new THREE.TextureLoader().load(I_Normal) },
				uDispStrength: { value: displacementStrength },
			},
			vertexShader: `
                varying vec2 vUV;
                varying vec3 vNormal;

                uniform sampler2D uRockHeight;
                uniform float uDispStrength;

                void main() {
                    vUV = uv;
                    vNormal = normal;

                    // Calculer le déplacement depuis la height map
                    float height = texture2D(uRockHeight, uv).r * 2.0 - 1.0;
                    vec3 displacedPosition = position + normal * (height * uDispStrength);

                    gl_Position = projectionMatrix * modelViewMatrix * vec4(displacedPosition, 1.0);
                }
            `,
			fragmentShader: `
                varying vec2 vUV;
                varying vec3 vNormal;

                uniform sampler2D uRockAlbedo;
                uniform sampler2D uRockNormal;

                void main() {
                    vec3 albedo = texture2D(uRockAlbedo, vUV).rgb;
                    vec3 normal = texture2D(uRockNormal, vUV).rgb * 2.0 - 1.0;
                    normal = normalize(normal);

                    vec3 lightDirection = normalize(vec3(1.0, 1.0, 1.0));
                    float lightIntensity = max(dot(normal, lightDirection), 0.0);

                    vec3 color = albedo * lightIntensity;
                    gl_FragColor = vec4(color, 1.0);
                }
            `,
		});

		sphere = new THREE.Mesh(geometry, material);
		scene.add(sphere);
		sphere.position.set(-4, 0, 0);

		// Lighting
		const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
		scene.add(ambientLight);

		const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
		directionalLight.position.set(5, 5, 5);
		scene.add(directionalLight);

		// Animation loop
		const animate = () => {
			requestAnimationFrame(animate);
			controls.update(); // Ensure damping works
			renderer.render(scene, camera);
		};

		animate();

		// Handle window resizing
		window.addEventListener('resize', () => {
			camera.aspect = window.innerWidth / window.innerHeight;
			camera.updateProjectionMatrix();
			renderer.setSize(window.innerWidth, window.innerHeight);
		});
	}

	onMount(() => {
		initScene();
	});

	// Update displacement strength dynamically
	$: if (material) {
		material.uniforms.uDispStrength.value = displacementStrength;
	}
</script>

<div class="page-wrapper">
	<div bind:this={container} class="three-container"></div>
	<div class="controls">
		<label for="displacement-strength">Height Map Strength</label>
		<input
			id="displacement-strength"
			type="range"
			min="0"
			max="1"
			step="0.01"
			bind:value={displacementStrength}
		/>
		<span>{displacementStrength.toFixed(2)}</span>
	</div>
</div>

<style>
    .page-wrapper {
        width: 100vw;
        height: 100vh;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }

    .three-container {
        width: 700px;
        height: 400px;
        background-color: #ffffff;
        border-radius: 12px;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
        overflow: hidden;
    }

    .controls {
        margin-top: 20px;
        display: flex;
        flex-direction: column;
        gap: 10px;
        width: 700px;
    }

    label {
        margin-bottom: 5px;
        font-weight: bold;
    }

    input[type='range'] {
        width: 100%;
    }

    span {
        margin-top: 5px;
        text-align: right;
        font-size: 14px;
    }
</style>
