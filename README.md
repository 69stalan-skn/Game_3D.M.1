<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>3D Tool Matcher - เกมจับคู่เครื่องมือช่าง</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;600&display=swap" rel="stylesheet">
    <style>
        body { margin: 0; overflow: hidden; font-family: 'Kanit', sans-serif; background-color: #1a202c; user-select: none; }
        canvas { display: block; width: 100vw; height: 100vh; }
        #ui-layer { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; display: flex; flex-direction: column; justify-content: space-between; }
        .pointer-events-auto { pointer-events: auto; }
        
        /* สไตล์สำหรับปุ่มให้ดูมีมิติ */
        .btn-3d {
            transition: all 0.1s ease;
            box-shadow: 0 4px 6px rgba(0,0,0,0.3), 0 8px 0 #0f766e;
        }
        .btn-3d:active {
            transform: translateY(4px);
            box-shadow: 0 0px 0 rgba(0,0,0,0.3), 0 4px 0 #0f766e;
        }

        /* Message Box Animation */
        @keyframes popIn {
            0% { transform: scale(0.5); opacity: 0; }
            100% { transform: scale(1); opacity: 1; }
        }
        .animate-pop { animation: popIn 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards; }
    </style>
</head>
<body>

    <!-- 3D Canvas -->
    <canvas id="gameCanvas"></canvas>

    <!-- UI Layer -->
    <div id="ui-layer" class="p-4 md:p-8">
        
        <!-- Header / HUD -->
        <div id="hud" class="hidden flex justify-between items-start">
            <div class="bg-gray-900/80 backdrop-blur text-white px-6 py-3 rounded-2xl border border-gray-700 shadow-xl">
                <h1 class="text-xl md:text-2xl font-bold text-teal-400">Tool Matcher 3D</h1>
                <p id="player-display" class="text-sm text-teal-200 font-semibold mb-1"></p>
                <p id="level-display" class="text-sm md:text-base text-gray-300">ระดับ: ยากมาก (ข้อ 1/10)</p>
            </div>
            <div class="bg-gray-900/80 backdrop-blur text-white px-6 py-3 rounded-2xl border border-gray-700 shadow-xl text-center">
                <p class="text-sm text-gray-400">คะแนน</p>
                <p id="score-display" class="text-2xl font-bold text-yellow-400">0</p>
            </div>
        </div>

        <!-- Task Display (New) -->
        <div id="task-display" class="hidden absolute top-4 md:top-8 left-0 w-full text-center pointer-events-none z-10 px-4">
            <div class="inline-block bg-blue-900/85 backdrop-blur text-white px-8 py-3 rounded-full border-2 border-blue-400 shadow-[0_0_15px_rgba(59,130,246,0.5)]">
                <p class="text-sm md:text-base text-blue-300 font-semibold mb-1">ปัญหาที่พบ</p>
                <p id="task-text" class="text-lg md:text-2xl font-bold text-white tracking-wide"></p>
            </div>
        </div>

        <!-- Start Screen -->
        <div id="start-screen" class="absolute inset-0 bg-gray-900/90 backdrop-blur-sm flex flex-col items-center justify-center pointer-events-auto z-50">
            <div class="bg-gray-800 p-8 rounded-3xl shadow-2xl border border-gray-700 text-center max-w-md w-full mx-4">
                <div class="w-24 h-24 bg-teal-500 rounded-full flex items-center justify-center mx-auto mb-6 shadow-lg shadow-teal-500/50">
                    <svg class="w-12 h-12 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"></path><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"></path></svg>
                </div>
                <h1 class="text-3xl font-bold text-white mb-2">เกมช่างสมจริง 3D</h1>
                <p class="text-gray-400 mb-6">บททดสอบ 10 ข้อ (ระดับยากมาก)<br>วิเคราะห์ปัญหาและเลือกเครื่องมือให้แม่นยำ</p>
                
                <div class="mb-6 flex flex-col gap-3 text-left">
                    <input type="text" id="player-fname" placeholder="ชื่อ" class="w-full px-4 py-3 rounded-xl bg-gray-700 text-white border border-gray-600 focus:outline-none focus:border-teal-500 focus:ring-1 focus:ring-teal-500 transition-colors">
                    <input type="text" id="player-lname" placeholder="นามสกุล" class="w-full px-4 py-3 rounded-xl bg-gray-700 text-white border border-gray-600 focus:outline-none focus:border-teal-500 focus:ring-1 focus:ring-teal-500 transition-colors">
                    <p id="login-error" class="text-red-400 text-sm hidden">กรุณากรอกชื่อและนามสกุลให้ครบถ้วนก่อนเริ่มเกม</p>
                </div>

                <button id="start-btn" class="btn-3d pointer-events-auto bg-teal-500 hover:bg-teal-400 text-white font-bold py-4 px-10 rounded-2xl text-xl w-full">
                    เริ่มเกม
                </button>
            </div>
        </div>

        <!-- Message Overlay (Correct/Wrong/End) -->
        <div id="message-overlay" class="absolute inset-0 flex items-center justify-center pointer-events-none hidden z-40">
            <div id="message-box" class="bg-gray-800/95 p-8 rounded-3xl shadow-2xl border-2 text-center max-w-sm w-full mx-4 pointer-events-auto">
                <h2 id="message-title" class="text-4xl font-bold mb-4"></h2>
                <p id="message-desc" class="text-gray-300 mb-6 text-lg"></p>
                <button id="next-btn" class="btn-3d bg-blue-500 hover:bg-blue-400 text-white font-bold py-3 px-8 rounded-xl w-full hidden">ต่อไป</button>
                <button id="restart-btn" class="btn-3d bg-teal-500 hover:bg-teal-400 text-white font-bold py-3 px-8 rounded-xl w-full hidden">เล่นอีกครั้ง</button>
            </div>
        </div>

        <!-- Instructions (Bottom Left) -->
        <div id="instruction" class="hidden absolute bottom-6 left-4 md:bottom-8 md:left-8 z-10 pointer-events-none">
            <div class="inline-block bg-gray-900/80 backdrop-blur text-white px-5 py-3 md:px-8 md:py-3 rounded-full border border-gray-700 shadow-xl">
                <p class="text-sm md:text-lg font-semibold flex items-center gap-2">
                    <svg class="w-5 h-5 md:w-6 md:h-6 text-teal-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 15l-2 5L9 9l11 4-5 2zm0 0l5 5M7.188 2.239l.777 2.897M5.136 7.965l-2.898-.777M13.95 4.05l-2.122 2.122m-5.657 5.656l-2.12 2.122"></path></svg>
                    แตะที่เครื่องมือช่างที่ตรงกับงาน
                </p>
            </div>
        </div>

        <!-- Tool Labels Container -->
        <div id="tool-labels" class="absolute inset-0 pointer-events-none z-20"></div>
    </div>

    <script>
        // ==========================================
        // 1. SETUP THREE.JS & VARIABLES
        // ==========================================
        const canvas = document.getElementById('gameCanvas');
        const renderer = new THREE.WebGLRenderer({ canvas, antialias: true, alpha: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        renderer.shadowMap.enabled = true;
        renderer.shadowMap.type = THREE.PCFSoftShadowMap;

        const scene = new THREE.Scene();
        // Background gradient simulation using fog
        scene.fog = new THREE.FogExp2(0x1a202c, 0.05);

        const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 100);
        camera.position.set(0, 3, 12);
        camera.lookAt(0, 0, 0);

        // Lights
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
        scene.add(ambientLight);

        const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
        dirLight.position.set(5, 10, 7);
        dirLight.castShadow = true;
        dirLight.shadow.mapSize.width = 1024;
        dirLight.shadow.mapSize.height = 1024;
        scene.add(dirLight);
        
        const spotLight = new THREE.SpotLight(0x4fd1c5, 1);
        spotLight.position.set(0, -5, 5);
        spotLight.lookAt(0,0,0);
        scene.add(spotLight);

        // --- เพิ่มแสงไฮไลท์สีสันเฉพาะจุดสำหรับเครื่องมือทั้ง 3 ชิ้น ---
        const highlight1 = new THREE.PointLight(0x00e5ff, 1.5, 8); // สีฟ้า Cyan
        highlight1.position.set(-4, -0.5, 2);
        scene.add(highlight1);

        const highlight2 = new THREE.PointLight(0xffd700, 1.5, 8); // สีทอง Gold
        highlight2.position.set(0, -0.5, 2);
        scene.add(highlight2);

        const highlight3 = new THREE.PointLight(0xff007f, 1.5, 8); // สีชมพู Pink
        highlight3.position.set(4, -0.5, 2);
        scene.add(highlight3);

        // Raycaster for interaction
        const raycaster = new THREE.Raycaster();
        const mouse = new THREE.Vector2();

        // Game State
        let currentQuestion = 0; 
        let score = 0;
        let isInputEnabled = false;
        let playerName = "";
        
        // Active 3D Objects
        let targetObject = null;
        let toolObjects = [];
        let correctToolIndex = -1;
        
        // Tool Labels
        let toolLabels = [];

        // ==========================================
        // 2. PROCEDURAL 3D MODELS (MATERIALS & BUILDERS)
        // ==========================================
        const matMetal = new THREE.MeshStandardMaterial({ color: 0x8892b0, metalness: 0.9, roughness: 0.2 });
        const matWood = new THREE.MeshStandardMaterial({ color: 0x8b5a2b, metalness: 0.1, roughness: 0.8 });
        const matPlasticRed = new THREE.MeshStandardMaterial({ color: 0xe53e3e, metalness: 0.2, roughness: 0.5 });
        const matPlasticBlue = new THREE.MeshStandardMaterial({ color: 0x3182ce, metalness: 0.2, roughness: 0.5 });
        const matPlasticYellow = new THREE.MeshStandardMaterial({ color: 0xd69e2e, metalness: 0.2, roughness: 0.5 });
        const matConcrete = new THREE.MeshStandardMaterial({ color: 0x718096, metalness: 0.1, roughness: 0.9 });
        const matGreenBoard = new THREE.MeshStandardMaterial({ color: 0x276749, metalness: 0.3, roughness: 0.6 });
        const matCopper = new THREE.MeshStandardMaterial({ color: 0xc53030, metalness: 0.8, roughness: 0.3 });
        const matBrick = new THREE.MeshStandardMaterial({ color: 0x9b2c2c, metalness: 0.1, roughness: 0.9 });
        const matRubber = new THREE.MeshStandardMaterial({ color: 0x1a202c, metalness: 0.1, roughness: 0.9 });
        const matCardboard = new THREE.MeshStandardMaterial({ color: 0xd2b48c, metalness: 0.0, roughness: 0.9 });
        const matCeramic = new THREE.MeshStandardMaterial({ color: 0xf7fafc, metalness: 0.1, roughness: 0.2 });
        const matGlass = new THREE.MeshStandardMaterial({ color: 0xbee3f8, transparent: true, opacity: 0.6, roughness: 0.1 });

        // Helper to enable shadows
        function setupShadows(group) {
            group.traverse(child => {
                if (child.isMesh) {
                    child.castShadow = true;
                    child.receiveShadow = true;
                }
            });
            return group;
        }

        // --- TOOLS ---
        function createHammer() {
            const group = new THREE.Group();
            const handle = new THREE.Mesh(new THREE.CylinderGeometry(0.15, 0.12, 2, 16), matWood);
            handle.position.y = -0.5;
            const head = new THREE.Mesh(new THREE.BoxGeometry(1.2, 0.4, 0.3), matMetal);
            head.position.y = 0.5;
            group.add(handle, head);
            group.userData.toolName = "ค้อน";
            return setupShadows(group);
        }

        function createScrewdriver() {
            const group = new THREE.Group();
            const handle = new THREE.Mesh(new THREE.CylinderGeometry(0.2, 0.15, 1, 16), matPlasticYellow);
            handle.position.y = -0.5;
            const shaft = new THREE.Mesh(new THREE.CylinderGeometry(0.05, 0.05, 1.5, 16), matMetal);
            shaft.position.y = 0.75;
            group.add(handle, shaft);
            group.userData.toolName = "ไขควง";
            return setupShadows(group);
        }

        function createWrench() {
            const group = new THREE.Group();
            const handle = new THREE.Mesh(new THREE.BoxGeometry(0.3, 2, 0.1), matMetal);
            const head1 = new THREE.Mesh(new THREE.TorusGeometry(0.25, 0.1, 16, 32, Math.PI * 1.5), matMetal);
            head1.position.y = 1;
            const head2 = new THREE.Mesh(new THREE.TorusGeometry(0.2, 0.1, 16, 32, Math.PI * 1.5), matMetal);
            head2.position.y = -1;
            group.add(handle, head1, head2);
            group.userData.toolName = "ประแจเลื่อน";
            return setupShadows(group);
        }

        function createSaw() {
            const group = new THREE.Group();
            const blade = new THREE.Mesh(new THREE.BoxGeometry(2.5, 0.8, 0.05), matMetal);
            blade.position.x = 0.5;
            const handle = new THREE.Mesh(new THREE.BoxGeometry(0.4, 1.2, 0.2), matWood);
            handle.position.set(-1, -0.4, 0);
            handle.rotation.z = Math.PI / 8;
            group.add(blade, handle);
            group.userData.toolName = "เลื่อย";
            return setupShadows(group);
        }

        function createDrill() {
            const group = new THREE.Group();
            const body = new THREE.Mesh(new THREE.BoxGeometry(1.5, 0.8, 0.5), matPlasticBlue);
            const handle = new THREE.Mesh(new THREE.BoxGeometry(0.4, 1.2, 0.4), matPlasticBlue);
            handle.position.set(-0.4, -0.8, 0);
            handle.rotation.z = Math.PI / 16;
            const bit = new THREE.Mesh(new THREE.CylinderGeometry(0.05, 0.05, 1, 16), matMetal);
            bit.rotation.z = Math.PI / 2;
            bit.position.set(1.2, 0, 0);
            group.add(body, handle, bit);
            group.userData.toolName = "สว่านไฟฟ้า";
            return setupShadows(group);
        }

        function createPliers() {
            const group = new THREE.Group();
            const arm1 = new THREE.Group();
            const h1 = new THREE.Mesh(new THREE.CylinderGeometry(0.1, 0.1, 1.5), matPlasticRed);
            h1.position.set(0.3, -0.7, 0);
            const j1 = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.02, 1), matMetal);
            j1.position.set(-0.1, 0.5, 0);
            arm1.add(h1, j1);
            arm1.rotation.z = Math.PI / 8;

            const arm2 = new THREE.Group();
            const h2 = new THREE.Mesh(new THREE.CylinderGeometry(0.1, 0.1, 1.5), matPlasticRed);
            h2.position.set(-0.3, -0.7, 0);
            const j2 = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.02, 1), matMetal);
            j2.position.set(0.1, 0.5, 0);
            arm2.add(h2, j2);
            arm2.rotation.z = -Math.PI / 8;
            
            const pivot = new THREE.Mesh(new THREE.CylinderGeometry(0.12, 0.12, 0.2), matMetal);
            pivot.rotation.x = Math.PI/2;

            group.add(arm1, arm2, pivot);
            group.userData.toolName = "คีม";
            return setupShadows(group);
        }

        function createTapeMeasure() {
            const group = new THREE.Group();
            const body = new THREE.Mesh(new THREE.BoxGeometry(1.2, 1.2, 0.5), matPlasticYellow);
            const tape = new THREE.Mesh(new THREE.BoxGeometry(2, 0.05, 0.3), matMetal);
            tape.position.set(1.5, -0.4, 0);
            group.add(body, tape);
            group.userData.toolName = "ตลับเมตร";
            return setupShadows(group);
        }

        function createSolderingIron() {
            const group = new THREE.Group();
            const handle = new THREE.Mesh(new THREE.CylinderGeometry(0.15, 0.15, 1.2), new THREE.MeshStandardMaterial({color: 0x111111}));
            handle.position.y = -0.5;
            const shaft = new THREE.Mesh(new THREE.CylinderGeometry(0.05, 0.05, 1), matMetal);
            shaft.position.y = 0.6;
            const tip = new THREE.Mesh(new THREE.ConeGeometry(0.05, 0.3, 16), matMetal);
            tip.position.y = 1.25;
            group.add(handle, shaft, tip);
            group.userData.toolName = "หัวแร้งบัดกรี";
            return setupShadows(group);
        }

        function createTrowel() {
            const group = new THREE.Group();
            const blade = new THREE.Mesh(new THREE.BoxGeometry(1.5, 0.05, 1), matMetal);
            blade.scale.set(1, 1, 1);
            
            const stem = new THREE.Mesh(new THREE.CylinderGeometry(0.05, 0.05, 0.5), matMetal);
            stem.position.set(-0.5, 0.25, 0);
            
            const handle = new THREE.Mesh(new THREE.CylinderGeometry(0.15, 0.15, 1), matWood);
            handle.rotation.z = Math.PI / 2;
            handle.position.set(-1, 0.5, 0);

            group.add(blade, stem, handle);
            group.userData.toolName = "เกรียง";
            return setupShadows(group);
        }

        // --- NEW TOOLS ---
        function createPaintBrush() {
            const group = new THREE.Group();
            const handle = new THREE.Mesh(new THREE.CylinderGeometry(0.12, 0.1, 1.2), matWood);
            handle.position.y = -0.5;
            const metalBand = new THREE.Mesh(new THREE.BoxGeometry(0.35, 0.25, 0.15), matMetal);
            metalBand.position.y = 0.2;
            const bristles = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.5, 0.1), new THREE.MeshStandardMaterial({color: 0x222222}));
            bristles.position.y = 0.55;
            group.add(handle, metalBand, bristles);
            group.userData.toolName = "แปรงทาสี";
            return setupShadows(group);
        }

        function createCutter() {
            const group = new THREE.Group();
            const body = new THREE.Mesh(new THREE.BoxGeometry(0.3, 1.5, 0.1), matPlasticRed);
            const blade = new THREE.Mesh(new THREE.BoxGeometry(0.18, 0.6, 0.02), matMetal);
            blade.position.set(0.05, 0.9, 0);
            group.add(body, blade);
            group.userData.toolName = "คัตเตอร์";
            return setupShadows(group);
        }

        function createPlunger() {
            const group = new THREE.Group();
            const handle = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.08, 1.5), matWood);
            handle.position.y = 0.5;
            const cup = new THREE.Mesh(new THREE.CylinderGeometry(0.5, 0.6, 0.4, 16), matRubber);
            cup.position.y = -0.4;
            group.add(handle, cup);
            group.userData.toolName = "ที่ปั๊มส้วม";
            return setupShadows(group);
        }

        function createAxe() {
            const group = new THREE.Group();
            const handle = new THREE.Mesh(new THREE.CylinderGeometry(0.1, 0.15, 1.8), matWood);
            const head = new THREE.Mesh(new THREE.BoxGeometry(0.8, 0.4, 0.1), matMetal);
            head.position.set(0.3, 0.7, 0);
            group.add(handle, head);
            group.userData.toolName = "ขวาน";
            return setupShadows(group);
        }

        function createLevel() {
            const group = new THREE.Group();
            const bar = new THREE.Mesh(new THREE.BoxGeometry(2, 0.3, 0.15), matPlasticYellow);
            const vial = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.08, 0.2), matGlass);
            vial.rotation.z = Math.PI / 2;
            const fluid = new THREE.Mesh(new THREE.CylinderGeometry(0.06, 0.06, 0.1), new THREE.MeshStandardMaterial({color: 0x9ae6b4}));
            fluid.rotation.z = Math.PI / 2;
            group.add(bar, vial, fluid);
            group.userData.toolName = "ระดับน้ำ";
            return setupShadows(group);
        }

        function createAllenWrench() {
            const group = new THREE.Group();
            const arm1 = new THREE.Mesh(new THREE.CylinderGeometry(0.06, 0.06, 1.2, 6), matMetal);
            arm1.position.y = -0.3;
            const arm2 = new THREE.Mesh(new THREE.CylinderGeometry(0.06, 0.06, 0.6, 6), matMetal);
            arm2.rotation.z = Math.PI / 2;
            arm2.position.set(0.3, 0.27, 0);
            group.add(arm1, arm2);
            group.userData.toolName = "ประแจแอล";
            return setupShadows(group);
        }

        function createChisel() {
            const group = new THREE.Group();
            const handle = new THREE.Mesh(new THREE.CylinderGeometry(0.15, 0.15, 0.8), matPlasticBlue);
            handle.position.y = -0.4;
            const shaft = new THREE.Mesh(new THREE.BoxGeometry(0.2, 1, 0.05), matMetal);
            shaft.position.y = 0.5;
            group.add(handle, shaft);
            group.userData.toolName = "สิ่ว";
            return setupShadows(group);
        }

        function createCrowbar() {
            const group = new THREE.Group();
            const bar = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.08, 2, 6), matRubber);
            const tip = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.04, 0.4, 6), matMetal);
            tip.rotation.z = Math.PI / 4;
            tip.position.set(-0.15, 1.1, 0);
            group.add(bar, tip);
            group.userData.toolName = "ชะแลง";
            return setupShadows(group);
        }

        function createCaulkingGun() {
            const group = new THREE.Group();
            const tube = new THREE.Mesh(new THREE.CylinderGeometry(0.2, 0.2, 1.5), matPlasticYellow);
            const handle = new THREE.Mesh(new THREE.BoxGeometry(0.15, 0.6, 0.15), matMetal);
            handle.position.set(-0.2, -0.7, 0);
            handle.rotation.z = -Math.PI / 8;
            const nozzle = new THREE.Mesh(new THREE.ConeGeometry(0.05, 0.4), matPlasticYellow);
            nozzle.position.y = 0.9;
            group.add(tube, handle, nozzle);
            group.userData.toolName = "ปืนยิงซิลิโคน";
            return setupShadows(group);
        }

        function createWireStripper() {
            const group = new THREE.Group();
            const h1 = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.08, 1), matPlasticYellow);
            h1.position.set(0.2, -0.5, 0);
            h1.rotation.z = Math.PI / 16;
            const h2 = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.08, 1), matPlasticYellow);
            h2.position.set(-0.2, -0.5, 0);
            h2.rotation.z = -Math.PI / 16;
            const jaw = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.6, 0.05), matMetal);
            jaw.position.y = 0.3;
            group.add(h1, h2, jaw);
            group.userData.toolName = "คีมปอกสายไฟ";
            return setupShadows(group);
        }

        // --- TARGETS (MATERIALS TO FIX) ---
        function createTargetNail() {
            const group = new THREE.Group();
            const wood = new THREE.Mesh(new THREE.BoxGeometry(3, 0.5, 2), matWood);
            const nail = new THREE.Mesh(new THREE.CylinderGeometry(0.05, 0.05, 0.8), matMetal);
            nail.position.y = 0.5;
            const nailHead = new THREE.Mesh(new THREE.CylinderGeometry(0.15, 0.15, 0.05), matMetal);
            nailHead.position.y = 0.9;
            group.add(wood, nail, nailHead);
            group.userData.task = "ตะปูที่ยังตอกไม่มิด";
            return setupShadows(group);
        }

        function createTargetScrew() {
            const group = new THREE.Group();
            const plate = new THREE.Mesh(new THREE.BoxGeometry(2, 0.2, 2), matMetal);
            const screw = new THREE.Mesh(new THREE.CylinderGeometry(0.2, 0.2, 0.1), matMetal);
            screw.position.y = 0.15;
            // Add a groove
            const groove = new THREE.Mesh(new THREE.BoxGeometry(0.4, 0.11, 0.05), new THREE.MeshStandardMaterial({color: 0x222222}));
            groove.position.y = 0.15;
            group.add(plate, screw, groove);
            group.userData.task = "สกรูยึดแผ่นโลหะหลวม";
            return setupShadows(group);
        }

        function createTargetBolt() {
            const group = new THREE.Group();
            const block = new THREE.Mesh(new THREE.BoxGeometry(2, 1, 2), matMetal);
            const bolt = new THREE.Mesh(new THREE.CylinderGeometry(0.3, 0.3, 0.4, 6), matMetal); // Hexagon
            bolt.position.y = 0.7;
            group.add(block, bolt);
            group.userData.task = "น็อตหกเหลี่ยมต้องการการขันให้แน่น";
            return setupShadows(group);
        }

        function createTargetWood() {
            const group = new THREE.Group();
            const log = new THREE.Mesh(new THREE.BoxGeometry(4, 0.8, 1), matWood);
            // Draw a cut line
            const line = new THREE.Mesh(new THREE.BoxGeometry(0.05, 0.85, 1.05), new THREE.MeshStandardMaterial({color: 0x111111}));
            group.add(log, line);
            group.userData.task = "ท่อนไม้ที่ต้องตัดตามรอย";
            return setupShadows(group);
        }

        function createTargetWall() {
            const group = new THREE.Group();
            const wall = new THREE.Mesh(new THREE.BoxGeometry(3, 3, 0.5), matConcrete);
            const mark = new THREE.Mesh(new THREE.CylinderGeometry(0.2, 0.2, 0.55), new THREE.MeshStandardMaterial({color: 0x000000}));
            mark.rotation.x = Math.PI / 2;
            group.add(wall, mark);
            group.userData.task = "กำแพงคอนกรีตที่ต้องเจาะรูแขวนกรอบรูป";
            return setupShadows(group);
        }

        function createTargetWire() {
            const group = new THREE.Group();
            const wireGroup = new THREE.Group();
            
            // create a curved tube
            class CustomSinCurve extends THREE.Curve {
                constructor(scale = 1) { super(); this.scale = scale; }
                getPoint(t, optionalTarget = new THREE.Vector3()) {
                    const tx = t * 3 - 1.5;
                    const ty = Math.sin(t * Math.PI * 2) * 0.5;
                    const tz = 0;
                    return optionalTarget.set(tx, ty, tz).multiplyScalar(this.scale);
                }
            }
            const path = new CustomSinCurve(1.5);
            const tube = new THREE.Mesh(new THREE.TubeGeometry(path, 20, 0.08, 8, false), matCopper);
            
            wireGroup.add(tube);
            wireGroup.rotation.z = Math.PI/4;
            group.add(wireGroup);
            group.userData.task = "สายไฟทองแดงที่ยาวเกินไปต้องตัดและดัด";
            return setupShadows(group);
        }

        function createTargetDimension() {
            const group = new THREE.Group();
            const floor = new THREE.Mesh(new THREE.BoxGeometry(4, 0.1, 4), new THREE.MeshStandardMaterial({color: 0x4a5568}));
            // Grid lines
            const gridHelper = new THREE.GridHelper(4, 4, 0xffffff, 0xffffff);
            gridHelper.position.y = 0.1;
            group.add(floor, gridHelper);
            group.userData.task = "พื้นที่ว่างเปล่าที่ต้องวัดขนาดเพื่อปูกระเบื้อง";
            return setupShadows(group);
        }

        function createTargetCircuit() {
            const group = new THREE.Group();
            const board = new THREE.Mesh(new THREE.BoxGeometry(3, 0.1, 2), matGreenBoard);
            const chip = new THREE.Mesh(new THREE.BoxGeometry(0.6, 0.2, 0.6), new THREE.MeshStandardMaterial({color: 0x111111}));
            chip.position.set(-0.5, 0.1, 0.5);
            // solder points
            const solder1 = new THREE.Mesh(new THREE.SphereGeometry(0.08), matMetal);
            solder1.position.set(0.5, 0.05, -0.5);
            const solder2 = new THREE.Mesh(new THREE.SphereGeometry(0.08), matMetal);
            solder2.position.set(0.8, 0.05, -0.5);
            group.add(board, chip, solder1, solder2);
            group.userData.task = "แผงวงจรอิเล็กทรอนิกส์ที่มีจุดบัดกรีหลุด";
            return setupShadows(group);
        }

        function createTargetBricks() {
            const group = new THREE.Group();
            for(let i=0; i<3; i++) {
                for(let j=0; j<2; j++) {
                    const brick = new THREE.Mesh(new THREE.BoxGeometry(0.8, 0.4, 0.4), matBrick);
                    brick.position.set(i*1 - 1, j*0.5, 0);
                    if (j%2===1) brick.position.x += 0.5; // offset
                    group.add(brick);
                }
            }
            group.userData.task = "ผนังอิฐที่กำลังก่อและต้องการปาดปูนซีเมนต์";
            return setupShadows(group);
        }

        function createTargetPipe() {
            const group = new THREE.Group();
            const pipe1 = new THREE.Mesh(new THREE.CylinderGeometry(0.3, 0.3, 2), matMetal);
            pipe1.position.y = -1;
            const pipe2 = new THREE.Mesh(new THREE.CylinderGeometry(0.3, 0.3, 2), matMetal);
            pipe2.position.set(1, 0, 0);
            pipe2.rotation.z = Math.PI / 2;
            const joint = new THREE.Mesh(new THREE.SphereGeometry(0.42), matMetal);
            
            // Drop of water
            const drop = new THREE.Mesh(new THREE.SphereGeometry(0.1), new THREE.MeshStandardMaterial({color: 0x63b3ed, transparent: true, opacity: 0.8, roughness: 0}));
            drop.position.set(0, -0.6, 0.35);
            drop.scale.set(1, 1.5, 1);

            group.add(pipe1, pipe2, joint, drop);
            group.userData.task = "ข้อต่อท่อประปาเหล็กหลวมจนมีน้ำรั่วซึม";
            return setupShadows(group);
        }

        // --- NEW TARGETS ---
        function createTargetFadedWall() {
            const group = new THREE.Group();
            const wall = new THREE.Mesh(new THREE.BoxGeometry(3, 3, 0.5), new THREE.MeshStandardMaterial({color: 0xcbd5e0}));
            const patch = new THREE.Mesh(new THREE.BoxGeometry(1.5, 1.5, 0.55), new THREE.MeshStandardMaterial({color: 0x718096}));
            group.add(wall, patch);
            group.userData.task = "ผนังสีลอกที่ต้องการการทาสีซ่อมแซม";
            return setupShadows(group);
        }

        function createTargetBox() {
            const group = new THREE.Group();
            const box = new THREE.Mesh(new THREE.BoxGeometry(2, 2, 2), matCardboard);
            const tape = new THREE.Mesh(new THREE.BoxGeometry(2.05, 0.2, 2.05), new THREE.MeshStandardMaterial({color: 0xe2e8f0}));
            group.add(box, tape);
            group.userData.task = "กล่องกระดาษที่ปิดเทปกาวแน่นหนาต้องกรีดเปิด";
            return setupShadows(group);
        }

        function createTargetToilet() {
            const group = new THREE.Group();
            const base = new THREE.Mesh(new THREE.CylinderGeometry(1, 1, 0.5), matCeramic);
            const hole = new THREE.Mesh(new THREE.CylinderGeometry(0.4, 0.4, 0.51), new THREE.MeshStandardMaterial({color: 0x2d3748}));
            group.add(base, hole);
            group.userData.task = "ท่อระบายน้ำหรือชักโครกที่อุดตัน";
            return setupShadows(group);
        }

        function createTargetFirewood() {
            const group = new THREE.Group();
            const log = new THREE.Mesh(new THREE.CylinderGeometry(0.6, 0.6, 2), matWood);
            log.rotation.x = Math.PI / 2;
            group.add(log);
            group.userData.task = "ท่อนไม้ขนาดใหญ่ที่ต้องผ่าเป็นฟืน";
            return setupShadows(group);
        }

        function createTargetShelf() {
            const group = new THREE.Group();
            const shelf = new THREE.Mesh(new THREE.BoxGeometry(3, 0.2, 1), matWood);
            shelf.rotation.z = Math.PI / 16; // Uneven
            group.add(shelf);
            group.userData.task = "ชั้นวางของที่ติดตั้งแล้วเอียง ไม่ได้ระดับ";
            return setupShadows(group);
        }

        function createTargetIkea() {
            const group = new THREE.Group();
            const wood = new THREE.Mesh(new THREE.BoxGeometry(2, 2, 0.5), matWood);
            const hole = new THREE.Mesh(new THREE.CylinderGeometry(0.1, 0.1, 0.52, 6), new THREE.MeshStandardMaterial({color: 0x111111}));
            hole.rotation.x = Math.PI / 2;
            group.add(wood, hole);
            group.userData.task = "เฟอร์นิเจอร์ประกอบที่มีรูน็อตหกเหลี่ยม";
            return setupShadows(group);
        }

        function createTargetDoorHinge() {
            const group = new THREE.Group();
            const edge = new THREE.Mesh(new THREE.BoxGeometry(1, 3, 0.5), matWood);
            const groove = new THREE.Mesh(new THREE.BoxGeometry(0.2, 1, 0.55), new THREE.MeshStandardMaterial({color: 0x5c3a21}));
            groove.position.set(-0.4, 0, 0);
            group.add(edge, groove);
            group.userData.task = "ขอบบานประตูไม้ที่ต้องเซาะร่องใส่บานพับ";
            return setupShadows(group);
        }

        function createTargetCrate() {
            const group = new THREE.Group();
            const board1 = new THREE.Mesh(new THREE.BoxGeometry(3, 0.5, 1), matWood);
            const board2 = new THREE.Mesh(new THREE.BoxGeometry(3, 0.5, 1), matWood);
            board2.position.set(0, 0.5, 0);
            const nail1 = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.08, 1.2), matMetal);
            nail1.position.set(-1, 0.5, 0);
            const nail2 = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.08, 1.2), matMetal);
            nail2.position.set(1, 0.5, 0);
            group.add(board1, board2, nail1, nail2);
            group.userData.task = "ลังไม้เก่าที่ถูกตอกปิดตายต้องการงัดเปิด";
            return setupShadows(group);
        }

        function createTargetTiles() {
            const group = new THREE.Group();
            const tile1 = new THREE.Mesh(new THREE.BoxGeometry(1.4, 1.4, 0.2), matCeramic);
            tile1.position.set(-0.75, 0, 0);
            const tile2 = new THREE.Mesh(new THREE.BoxGeometry(1.4, 1.4, 0.2), matCeramic);
            tile2.position.set(0.75, 0, 0);
            const gap = new THREE.Mesh(new THREE.BoxGeometry(0.1, 1.4, 0.18), new THREE.MeshStandardMaterial({color: 0x111111}));
            group.add(tile1, tile2, gap);
            group.userData.task = "รอยต่อกระเบื้องห้องน้ำที่มีช่องโหว่และน้ำรั่วซึม";
            return setupShadows(group);
        }

        function createTargetUnstrippedWire() {
            const group = new THREE.Group();
            const tube = new THREE.Mesh(new THREE.CylinderGeometry(0.2, 0.2, 2), matPlasticRed);
            tube.rotation.z = Math.PI / 2;
            const core = new THREE.Mesh(new THREE.CylinderGeometry(0.1, 0.1, 2.5), matCopper);
            core.rotation.z = Math.PI / 2;
            group.add(tube, core);
            group.userData.task = "ปลายสายไฟที่เพิ่งตัดและต้องการปอกฉนวนหุ้มออก";
            return setupShadows(group);
        }

        // ==========================================
        // 3. GAME DATA & LOGIC
        // ==========================================
        // Data structure: 10 questions (Very Hard)
        const gameData = [
            { target: createTargetCircuit, tools: [createSolderingIron, createWireStripper, createPliers], correct: 0, levelName: "ยากมาก" },
            { target: createTargetTiles, tools: [createCaulkingGun, createPaintBrush, createTrowel], correct: 0, levelName: "ยากมาก" },
            { target: createTargetUnstrippedWire, tools: [createWireStripper, createCutter, createPliers], correct: 0, levelName: "ยากมาก" },
            { target: createTargetCrate, tools: [createCrowbar, createHammer, createWrench], correct: 0, levelName: "ยากมาก" },
            { target: createTargetWall, tools: [createAllenWrench, createScrewdriver, createDrill], correct: 2, levelName: "ยากมาก" },
            { target: createTargetPipe, tools: [createWrench, createPliers, createTapeMeasure], correct: 0, levelName: "ยากมาก" },
            { target: createTargetDimension, tools: [createLevel, createTapeMeasure, createTrowel], correct: 1, levelName: "ยากมาก" },
            { target: createTargetDoorHinge, tools: [createCutter, createChisel, createScrewdriver], correct: 1, levelName: "ยากมาก" },
            { target: createTargetIkea, tools: [createScrewdriver, createAllenWrench, createDrill], correct: 1, levelName: "ยากมาก" },
            { target: createTargetBricks, tools: [createPaintBrush, createTrowel, createLevel], correct: 1, levelName: "ยากมาก" }
        ];

        // Audio using Tone.js
        let synth, errorSynth;
        function initAudio() {
            if(!synth) {
                synth = new Tone.Synth().toDestination();
                errorSynth = new Tone.MembraneSynth().toDestination();
            }
        }

        function playSuccessSound() {
            if(synth) {
                const now = Tone.now();
                synth.triggerAttackRelease("C5", "8n", now);
                synth.triggerAttackRelease("E5", "8n", now + 0.15);
                synth.triggerAttackRelease("G5", "4n", now + 0.3);
            }
        }

        function playErrorSound() {
            if(errorSynth) errorSynth.triggerAttackRelease("C2", "8n");
        }

        // UI Functions
        function updateHUD() {
            const qData = gameData[currentQuestion];
            document.getElementById('player-display').innerText = `ผู้เล่น: ${playerName}`;
            document.getElementById('level-display').innerText = `ระดับ: ${qData.levelName} (ข้อ ${currentQuestion + 1}/${gameData.length})`;
            document.getElementById('score-display').innerText = score;
        }

        // Initialize Tool Labels
        function initToolLabels() {
            const container = document.getElementById('tool-labels');
            for(let i=0; i<3; i++) {
                const el = document.createElement('div');
                el.className = "absolute text-white font-bold text-base md:text-lg bg-gray-900/90 px-5 py-2 rounded-full border-2 border-gray-500 backdrop-blur pointer-events-none shadow-xl transform -translate-x-1/2 -translate-y-1/2 transition-opacity duration-200";
                el.style.display = 'none';
                container.appendChild(el);
                toolLabels.push(el);
            }
        }

        function showMessage(title, desc, type, buttonType) {
            const overlay = document.getElementById('message-overlay');
            const box = document.getElementById('message-box');
            const titleEl = document.getElementById('message-title');
            
            overlay.classList.remove('hidden');
            box.classList.remove('animate-pop');
            void box.offsetWidth; // trigger reflow
            box.classList.add('animate-pop');

            titleEl.innerText = title;
            document.getElementById('message-desc').innerText = desc;
            
            document.getElementById('next-btn').classList.add('hidden');
            document.getElementById('restart-btn').classList.add('hidden');

            if (type === 'success') {
                box.className = "bg-green-900/95 p-8 rounded-3xl shadow-2xl border-2 border-green-500 text-center max-w-sm w-full mx-4 pointer-events-auto animate-pop text-white";
                titleEl.className = "text-4xl font-bold mb-4 text-green-300";
            } else if (type === 'error') {
                box.className = "bg-red-900/95 p-8 rounded-3xl shadow-2xl border-2 border-red-500 text-center max-w-sm w-full mx-4 pointer-events-auto animate-pop text-white";
                titleEl.className = "text-4xl font-bold mb-4 text-red-300";
            } else {
                box.className = "bg-gray-800/95 p-8 rounded-3xl shadow-2xl border-2 border-teal-500 text-center max-w-sm w-full mx-4 pointer-events-auto animate-pop text-white";
                titleEl.className = "text-4xl font-bold mb-4 text-teal-400";
            }

            if (buttonType === 'next') document.getElementById('next-btn').classList.remove('hidden');
            if (buttonType === 'restart') document.getElementById('restart-btn').classList.remove('hidden');
        }

        function hideMessage() {
            document.getElementById('message-overlay').classList.add('hidden');
        }

        // Game Loop Functions
        function loadStage() {
            // Clear previous objects
            if (targetObject) scene.remove(targetObject);
            toolObjects.forEach(t => scene.remove(t));
            toolObjects = [];

            const stageData = gameData[currentQuestion];
            correctToolIndex = stageData.correct;

            // 1. Setup Target
            targetObject = stageData.target();
            targetObject.position.set(0, 2.5, 0); // ดึงโมเดลเป้าหมายขึ้นมา
            
            // Update Task Display Text
            document.getElementById('task-text').innerText = targetObject.userData.task;
            
            scene.add(targetObject);

            // 2. Setup Tools
            const positions = [-4, 0, 4]; // ขยายระยะห่างซ้ายขวาให้กว้างขึ้น
            stageData.tools.forEach((toolFunc, index) => {
                const tool = toolFunc();
                tool.position.set(positions[index], -1.0, 0); // ดึงเครื่องมือช่างขึ้นมาไม่ให้ตกขอบล่าง
                
                // ปรับเพิ่มขนาดโมเดลเครื่องมือจาก 0.8 เป็น 1.4 ให้ใหญ่และชัดเจนขึ้น
                tool.scale.set(1.4, 1.4, 1.4);
                tool.rotation.x = Math.PI / 4;
                tool.rotation.y = Math.PI / 8;

                const bbox = new THREE.Box3().setFromObject(tool);
                const size = bbox.getSize(new THREE.Vector3());
                const hitBoxGeo = new THREE.BoxGeometry(size.x * 1.5, size.y * 1.5, size.z * 1.5);
                const hitBoxMat = new THREE.MeshBasicMaterial({visible: false});
                const hitBox = new THREE.Mesh(hitBoxGeo, hitBoxMat);
                
                hitBox.userData = { isTool: true, index: index, parentGroup: tool };
                tool.add(hitBox);

                toolObjects.push(tool);
                scene.add(tool);
                
                // Update Label Text and Show
                if(toolLabels.length > 0) {
                    toolLabels[index].innerText = tool.userData.toolName;
                    toolLabels[index].style.display = 'block';
                    toolLabels[index].style.opacity = '1';
                }
            });

            updateHUD();
            
            isInputEnabled = true;
        }

        function handleToolClick(index) {
            if (!isInputEnabled) return;
            isInputEnabled = false;

            // Hide labels when animating
            toolLabels.forEach(label => label.style.opacity = '0');

            const selectedTool = toolObjects[index];
            
            // Animate tool flying to target
            const startPos = selectedTool.position.clone();
            const targetPos = targetObject.position.clone().add(new THREE.Vector3(0, 0, 2)); // Bring in front of target
            
            let frame = 0;
            const duration = 30; // frames
            
            function animateAction() {
                frame++;
                const progress = frame / duration;
                
                selectedTool.position.lerpVectors(startPos, targetPos, progress);
                selectedTool.rotation.z += 0.1; // Spin while flying

                if (frame < duration) {
                    requestAnimationFrame(animateAction);
                } else {
                    // Reached target, evaluate
                    if (index === correctToolIndex) {
                        handleCorrect();
                    } else {
                        handleWrong(selectedTool, startPos);
                    }
                }
            }
            animateAction();
        }

        function handleCorrect() {
            playSuccessSound();
            score += 1; // อัปเดตให้ได้ 1 คะแนนต่อข้อ
            
            // Visual feedback - flash target
            const origScale = targetObject.scale.clone();
            targetObject.scale.set(origScale.x*1.2, origScale.y*1.2, origScale.z*1.2);
            setTimeout(() => { targetObject.scale.copy(origScale); }, 150);

            updateHUD();
            showMessage("ยอดเยี่ยม!", "คุณเลือกเครื่องมือได้ถูกต้อง เหมาะสมกับงาน", "success", "next");
        }

        function handleWrong(tool, origPos) {
            playErrorSound();
            
            // หักคะแนน 1 คะแนนเมื่อตอบผิด (ไม่ให้ติดลบ)
            score = Math.max(0, score - 1);
            updateHUD();
            
            // Shake tool and return
            let shake = 0;
            const shakeInterval = setInterval(() => {
                tool.position.x += (Math.random() - 0.5) * 0.5;
                shake++;
                if(shake > 10) {
                    clearInterval(shakeInterval);
                    tool.position.copy(origPos); // Return to original spot
                    tool.rotation.set(Math.PI / 4, Math.PI / 8, 0); // Reset rotation
                    showMessage("ผิดพลาด!", "เครื่องมือนี้ไม่เหมาะกับงาน (หัก 1 คะแนน แต่ไม่ต่ำกว่า 0)", "error", "next");
                }
            }, 30);
        }

        function advanceGame() {
            hideMessage();
            
            currentQuestion++;
            if (currentQuestion >= gameData.length) {
                // Game Over - สรุปผลคะแนนหาร 2
                const finalScore = score / 2;
                showMessage("จบเกม!", `คุณซ่อมงานทั้งหมดสำเร็จ!\nคะแนนที่ทำได้: ${score} คะแนน\nคะแนนสรุปผล (หาร 2): ${finalScore} คะแนน`, "normal", "restart");
                return;
            }
            
            loadStage();
        }

        function restartGame() {
            currentQuestion = 0;
            score = 0;
            hideMessage();
            loadStage();
        }

        // ==========================================
        // 4. EVENTS & INTERACTION
        // ==========================================
        document.getElementById('start-btn').addEventListener('click', async () => {
            const fname = document.getElementById('player-fname').value.trim();
            const lname = document.getElementById('player-lname').value.trim();
            const errorText = document.getElementById('login-error');

            if (!fname || !lname) {
                errorText.classList.remove('hidden');
                return;
            }
            
            errorText.classList.add('hidden');
            playerName = `${fname} ${lname}`;

            await Tone.start(); // Required by browsers to start audio context
            initAudio();
            initToolLabels(); // Initialize labels here
            document.getElementById('start-screen').classList.add('hidden');
            document.getElementById('hud').classList.remove('hidden');
            document.getElementById('task-display').classList.remove('hidden');
            document.getElementById('instruction').classList.remove('hidden');
            loadStage();
        });

        document.getElementById('next-btn').addEventListener('click', () => {
            const box = document.getElementById('message-box');
            if (box.classList.contains('bg-red-900/95')) {
                // Was error, just hide and re-enable input
                hideMessage();
                isInputEnabled = true;
                // Show labels again
                toolLabels.forEach(label => label.style.opacity = '1');
            } else {
                // Was correct, advance
                advanceGame();
            }
        });

        document.getElementById('restart-btn').addEventListener('click', restartGame);

        // Raycasting for 3D clicks
        function onPointerDown(event) {
            if (!isInputEnabled) return;
            
            // Calculate mouse position in normalized device coordinates (-1 to +1)
            mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
            mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

            raycaster.setFromCamera(mouse, camera);

            // Intersect objects
            const intersects = raycaster.intersectObjects(scene.children, true);
            
            for (let i = 0; i < intersects.length; i++) {
                if (intersects[i].object.userData.isTool) {
                    handleToolClick(intersects[i].object.userData.index);
                    break;
                }
            }
        }
        window.addEventListener('pointerdown', onPointerDown);

        // Resize handler
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });


        // ==========================================
        // 5. ANIMATION LOOP
        // ==========================================
        window.onload = function() {
            // Add subtle floating animation vars
            const clock = new THREE.Clock();
            const vector = new THREE.Vector3();

            function animate() {
                requestAnimationFrame(animate);

                const time = clock.getElapsedTime();

                // Rotate Target slowly
                if (targetObject) {
                    targetObject.rotation.y = time * 0.5;
                    targetObject.position.y = 2.5 + Math.sin(time * 2) * 0.1; // Float (แก้ Base Y ให้ตรงกับค่าเริ่มต้นใหม่)
                }

                // Animate Tools idle state (if not currently clicked/flying)
                if (isInputEnabled && toolObjects.length > 0) {
                    toolObjects.forEach((tool, i) => {
                        tool.rotation.y = (Math.PI / 8) + Math.sin(time * 1.5 + i) * 0.1;
                        
                        // Sync 2D Label Position with 3D Object Position
                        if (toolLabels[i] && toolLabels[i].style.opacity !== '0') {
                            tool.getWorldPosition(vector);
                            vector.y -= 1.6; // ปรับระยะป้ายชื่อให้ลอยอยู่ใต้โมเดลพอดี
                            vector.project(camera);
                            
                            const x = (vector.x * 0.5 + 0.5) * window.innerWidth;
                            const y = (vector.y * -0.5 + 0.5) * window.innerHeight;
                            
                            toolLabels[i].style.left = `${x}px`;
                            toolLabels[i].style.top = `${y}px`;
                        }
                    });
                }

                // Render scene
                renderer.render(scene, camera);
            }
            animate();
        };

    </script>
</body>
</html>
