AURUM Dynamics NitiPulse-1 Interactive Concept

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AURUM Dynamics NitiPulse-1</title>
    <script src="https://unpkg.com/three@0.150.1/build/three.min.js"></script>
    <style>
        body { margin: 0; overflow: hidden; background: #050505; font-family: 'Segoe UI', sans-serif; color: white; }
        #ui-container {
            position: absolute;
            top: 20px;
            left: 20px;
            z-index: 10;
            background: rgba(0, 0, 0, 0.7);
            padding: 20px;
            border-radius: 12px;
            border: 1px solid #c5a059;
            backdrop-filter: blur(10px);

            max-width: 300px;
        }
        h1 { margin: 0 0 5px 0; font-size: 1.2rem; color: #c5a059; letter-spacing: 2px; }
        p { margin: 0 0 15px 0; font-size: 0.8rem; opacity: 0.8; }
        .controls { display: flex; flex-direction: column; gap: 10px; }
        button {
            background: #c5a059;
            border: none;
            color: black;
            padding: 10px;
            border-radius: 6px;
            cursor: pointer;
            font-weight: bold;
            transition: all 0.3s;
            text-transform: uppercase;
            font-size: 0.7rem;
        }
        button:hover { background: #e0bc7a; transform: translateY(-2px); }
        button:active { transform: translateY(0); }

        .status { margin-top: 15px; font-size: 0.7rem; border-top: 1px solid #333; padding-top: 10px; }
        .label { color: #c5a059; font-weight: bold; }
        #canvas-overlay {
            position: absolute;
            bottom: 20px;
            right: 20px;
            pointer-events: none;
            text-align: right;
        }
        .hud-text { font-size: 2rem; font-weight: 900; opacity: 0.1; margin: 0; }
    </style>
</head>
<body>

    <div id="ui-container">
        <h1>NITIPULSE-1</h1>
        <p>AURUM DYNAMICS | Concept Visualizer</p>
        
        <div class="controls">

            <button onclick="triggerPulse()">Trigger Muscle Pulse</button>
            <button onclick="toggleGrill()">Toggle Active Grill</button>
            <button onclick="resetCamera()">Reset View</button>
        </div>

        <div class="status">
            <div><span class="label">ENGINE:</span> <span id="engine-status">IDLE</span></div>
            <div><span class="label">TEMP:</span> <span id="temp-status">24°C</span></div>
            <div><span class="label">AERO:</span> <span id="aero-status">CLOSED</span></div>
        </div>
    </div>

    <div id="canvas-overlay">
        <p class="hud-text">AURUM</p>
        <p class="hud-text">DYNAMICS</p>
    </div>

    <script>
        let scene, camera, renderer, car, nitinolBundles = [], grillSlats = [];

        let isGrillOpen = false;
        let isPulsing = false;

        function init() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x050505);
            scene.fog = new THREE.Fog(0x050505, 10, 50);

            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(5, 2, 5);

            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(window.devicePixelRatio);
            document.body.appendChild(renderer.domElement);

            // Lights
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.4);
            scene.add(ambientLight);


            const spotLight = new THREE.SpotLight(0xc5a059, 1);
            spotLight.position.set(5, 10, 5);
            scene.add(spotLight);

            const pointLight = new THREE.PointLight(0xffffff, 0.5);
            pointLight.position.set(-5, 5, -5);
            scene.add(pointLight);

            // Ground
            const grid = new THREE.GridHelper(100, 50, 0x333333, 0x111111);
            grid.position.y = -0.5;
            scene.add(grid);

            createCar();
            animate();
        }

        function createCar() {
            car = new THREE.Group();


            // Main Body (Futuristic Wedge)
            const bodyGeo = new THREE.BoxGeometry(4, 0.6, 2);
            const bodyMat = new THREE.MeshStandardMaterial({ color: 0x222222, metalness: 0.9, roughness: 0.2 });
            const body = new THREE.Mesh(bodyGeo, bodyMat);
            car.add(body);

            // Cockpit / Glass Hood Area
            const hoodGeo = new THREE.BoxGeometry(1.5, 0.2, 1.4);
            const hoodMat = new THREE.MeshPhysicalMaterial({ 
                color: 0xffffff, 
                transparent: true, 
                opacity: 0.3, 
                transmission: 0.9, 
                thickness: 0.5 
            });
            const hood = new THREE.Mesh(hoodGeo, hoodMat);
            hood.position.set(0.5, 0.35, 0);
            car.add(hood);


            // Nitinol Engine Components (Visible under hood)
            for(let i = -2; i <= 2; i++) {
                const bundleGeo = new THREE.CylinderGeometry(0.05, 0.05, 1, 8);
                const bundleMat = new THREE.MeshStandardMaterial({ color: 0xcd7f32, emissive: 0x000000 });
                const bundle = new THREE.Mesh(bundleGeo, bundleMat);
                bundle.rotation.z = Math.PI / 2;
                bundle.position.set(0.5, 0.2, i * 0.25);
                car.add(bundle);
                nitinolBundles.push(bundle);
            }

            // Active Grill (Front Slats)
            const grillContainer = new THREE.Group();
            grillContainer.position.set(2, 0, 0);
            for(let i = 0; i < 5; i++) {
                const slatGeo = new THREE.BoxGeometry(0.02, 0.4, 1.8);
                const slatMat = new THREE.MeshStandardMaterial({ color: 0x111111 });
                const slat = new THREE.Mesh(slatGeo, slatMat);
                slat.position.y = (i - 2) * 0.1;

                grillContainer.add(slat);
                grillSlats.push(slat);
            }
            car.add(grillContainer);

            // Vortex Intakes (Side Channels)
            const intakeGeo = new THREE.TorusGeometry(0.3, 0.05, 16, 32, Math.PI);
            const intakeMat = new THREE.MeshStandardMaterial({ color: 0xc5a059 });
            const intakeL = new THREE.Mesh(intakeGeo, intakeMat);
            intakeL.position.set(-0.5, 0, 1.05);
            intakeL.rotation.y = Math.PI/2;
            car.add(intakeL);

            const intakeR = new THREE.Mesh(intakeGeo, intakeMat);
            intakeR.position.set(-0.5, 0, -1.05);
            intakeR.rotation.y = -Math.PI/2;
            car.add(intakeR);

            // Wheels

            const wheelGeo = new THREE.CylinderGeometry(0.4, 0.4, 0.3, 32);
            const wheelMat = new THREE.MeshStandardMaterial({ color: 0x000000 });
            const positions = [[1.5, -0.2, 0.9], [1.5, -0.2, -0.9], [-1.5, -0.2, 0.9], [-1.5, -0.2, -0.9]];
            positions.forEach(pos => {
                const wheel = new THREE.Mesh(wheelGeo, wheelMat);
                wheel.rotation.x = Math.PI / 2;
                wheel.position.set(...pos);
                car.add(wheel);
            });

            scene.add(car);
        }

        function triggerPulse() {
            if (isPulsing) return;
            isPulsing = true;
            document.getElementById('engine-status').innerText = "PULSING";
            document.getElementById('engine-status').style.color = "#ff4400";
            document.getElementById('temp-status').innerText = "85°C";


            let startTime = Date.now();
            const duration = 1000;

            function animatePulse() {
                let elapsed = Date.now() - startTime;
                let progress = elapsed / duration;

                if (progress < 1) {
                    const intensity = Math.sin(progress * Math.PI);
                    nitinolBundles.forEach(b => {
                        b.material.emissive.setHex(0xff4400);
                        b.material.emissiveIntensity = intensity * 2;
                        b.scale.x = 1 - (intensity * 0.2); // Simulating contraction
                    });
                    requestAnimationFrame(animatePulse);
                } else {
                    nitinolBundles.forEach(b => {
                        b.material.emissive.setHex(0x000000);
                        b.scale.x = 1;

                    });
                    isPulsing = false;
                    document.getElementById('engine-status').innerText = "IDLE";
                    document.getElementById('engine-status').style.color = "white";
                    document.getElementById('temp-status').innerText = "32°C";
                }
            }
            animatePulse();
        }

        function toggleGrill() {
            isGrillOpen = !isGrillOpen;
            document.getElementById('aero-status').innerText = isGrillOpen ? "OPEN" : "CLOSED";
            document.getElementById('aero-status').style.color = isGrillOpen ? "#c5a059" : "white";
            
            grillSlats.forEach((slat, i) => {
                const targetRotation = isGrillOpen ? Math.PI / 2.5 : 0;
                slat.rotation.z = targetRotation;
            });

        }

        function resetCamera() {
            camera.position.set(5, 2, 5);
            camera.lookAt(0, 0, 0);
        }

        function animate() {
            requestAnimationFrame(animate);
            
            // Subtle rotation for display
            if (car) car.rotation.y += 0.002;

            renderer.render(scene, camera);
        }

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();

            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        window.onload = init;
    </script>
</body>
</html>


