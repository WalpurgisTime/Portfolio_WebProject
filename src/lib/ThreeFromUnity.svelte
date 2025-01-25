<script>
	import { onMount } from 'svelte';
	import * as THREE from 'three';
	import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

	let container;
	let colorA = { r: 0, g: 0, b: 1, a: 1 }; // Bleu
	let colorB = { r: 1, g: 1, b: 1, a: 1 }; // Blanc
	let colorStart = 0.2;
	let colorEnd = 0.8;
	let material, scene, camera, renderer, controls, cylinder;

	function hexToRgb(hex) {
		const bigint = parseInt(hex.replace("#", ""), 16);
		const r = ((bigint >> 16) & 255) / 255;
		const g = ((bigint >> 8) & 255) / 255;
		const b = (bigint & 255) / 255;
		return { r, g, b };
	}

	function initScene() {
		scene = new THREE.Scene();
		scene.background = new THREE.Color(0xffffff);

		camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
		camera.position.set(0, 2, 5);

		renderer = new THREE.WebGLRenderer({ antialias: true });
		renderer.setSize(window.innerWidth, window.innerHeight);
		container.appendChild(renderer.domElement);

		controls = new OrbitControls(camera, renderer.domElement);
		controls.enableDamping = true;

		const geometry = new THREE.CylinderGeometry(0.5, 0.5, 2, 64);

		material = new THREE.ShaderMaterial({
			uniforms: {
				_ColorA: { value: new THREE.Vector4(colorA.r, colorA.g, colorA.b, colorA.a) },
				_ColorB: { value: new THREE.Vector4(colorB.r, colorB.g, colorB.b, colorB.a) },
				_ColorStart: { value: colorStart },
				_ColorEnd: { value: colorEnd },
				time: { value: 0.0 }
			},
			vertexShader: `
                varying vec2 vUv;
                varying vec3 vNormal;

                void main() {
                    vUv = uv;
                    vNormal = normalize(normal);
                    gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
                }
            `,
			fragmentShader: `
                uniform vec4 _ColorA;
                uniform vec4 _ColorB;
                uniform float _ColorStart;
                uniform float _ColorEnd;
                uniform float time;

                varying vec2 vUv;
                varying vec3 vNormal;

                #define TAU 6.28318530718

                float InverseLerp(float a, float b, float value) {
                    return (value - a) / (b - a);
                }

                void main() {
                    float xOffset = cos(vUv.x * TAU * 8.0) * 0.05;
                    float wave = cos((vUv.y + xOffset - time * 0.2) * TAU * 5.0) * 0.5 + 0.5;
                    wave *= 1.0 - vUv.y;

                    float mask = step(0.999, abs(vNormal.y)) == 0.0 ? 1.0 : 0.0;
                    vec4 gradient = mix(_ColorA, _ColorB, vUv.y);

                    gl_FragColor = gradient * wave * mask;
                }
            `,
			transparent: true
		});

		cylinder = new THREE.Mesh(geometry, material);
		scene.add(cylinder);

		const light = new THREE.AmbientLight(0xffffff, 1);
		scene.add(light);

		const animate = () => {
			material.uniforms.time.value += 0.05;
			material.uniforms._ColorA.value.set(colorA.r, colorA.g, colorA.b, colorA.a);
			material.uniforms._ColorB.value.set(colorB.r, colorB.g, colorB.b, colorB.a);
			material.uniforms._ColorStart.value = colorStart;
			material.uniforms._ColorEnd.value = colorEnd;

			requestAnimationFrame(animate);
			controls.update();
			renderer.render(scene, camera);
		};

		animate();

		window.addEventListener('resize', () => {
			camera.aspect = window.innerWidth / window.innerHeight;
			camera.updateProjectionMatrix();
			renderer.setSize(window.innerWidth, window.innerHeight);
		});
	}

	onMount(() => {
		initScene();
	});
</script>

<div class="page-wrapper">
	<div bind:this={container} class="three-container"></div>
	<div class="controls">
		<label>
			Color A:
			<input
				type="color"
				value="#0000ff"
				on:input={(e) => {
                    const { r, g, b } = hexToRgb(e.target.value);
                    colorA.r = r;
                    colorA.g = g;
                    colorA.b = b;
                }}
			/>
		</label>
		<label>
			Color B:
			<input
				type="color"
				value="#ffffff"
				on:input={(e) => {
                    const { r, g, b } = hexToRgb(e.target.value);
                    colorB.r = r;
                    colorB.g = g;
                    colorB.b = b;
                }}
			/>
		</label>
		<label>
			Start:
			<input type="range" min="0" max="1" step="0.01" bind:value={colorStart} />
		</label>
		<label>
			End:
			<input type="range" min="0" max="1" step="0.01" bind:value={colorEnd} />
		</label>
	</div>
</div>

<style>
    .page-wrapper {
        width: 100vw;
        height: 100vh;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
    }

    .three-container {
        width: 700px;
        height: 400px;
        display: flex;
        justify-content: center;
        align-items: center;
        background-color: #ffffff;
        border-radius: 12px;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
        overflow: hidden;
    }

    .controls {
        margin-top: 20px;
        display: flex;
        gap: 10px;
    }

    label {
        display: flex;
        flex-direction: column;
        font-size: 14px;
        text-align: center;
    }

    input[type="color"],
    input[type="range"] {
        margin-top: 5px;
    }
</style>
