
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SISTEMA MARSOS + SROS + SAMS/MHD - SIMULADOR 3D ARMADURA</title>
    <style>
        :root {
            --red-metallic: #8b0000;
            --gold-anodized: #d4af37;
            --titanium-silver: #a8b2c1;
            --hud-blue: #00f0ff;
            --reactor-cyan: #00ffff;
            --bg-dark: #030712;
            --panel-bg: rgba(7, 16, 34, 0.88);
            --border-hud: rgba(0, 240, 255, 0.4);
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', -apple-system, BlinkMacSystemFont, Roboto, sans-serif;
            user-select: none;
        }

        body {
            background-color: var(--bg-dark);
            color: #d1d5db;
            height: 100vh;
            overflow: hidden;
            display: flex;
            flex-direction: column;
        }

        /* HEADER */
        header {
            height: 50px;
            background: linear-gradient(90deg, rgba(139,0,0,0.4) 0%, rgba(3,7,18,0.9) 50%, rgba(212,175,55,0.3) 100%);
            border-bottom: 1px solid var(--border-hud);
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 0 20px;
            z-index: 10;
        }

        header h1 {
            font-size: 1rem;
            color: var(--hud-blue);
            letter-spacing: 2px;
            text-shadow: 0 0 10px var(--hud-blue);
        }

        .sys-badges {
            display: flex;
            gap: 10px;
            font-size: 0.7rem;
        }

        .badge {
            padding: 3px 8px;
            border-radius: 3px;
            border: 1px solid var(--border-hud);
            background: rgba(0, 240, 255, 0.1);
            color: #fff;
        }

        /* LAYOUT PRINCIPAL */
        .app-container {
            display: grid;
            grid-template-columns: 320px 1fr 360px;
            height: calc(100vh - 50px);
            position: relative;
        }

        .panel {
            background: var(--panel-bg);
            border-right: 1px solid var(--border-hud);
            padding: 15px;
            display: flex;
            flex-direction: column;
            gap: 15px;
            overflow-y: auto;
            z-index: 5;
            backdrop-filter: blur(10px);
        }

        .panel-right {
            border-right: none;
            border-left: 1px solid var(--border-hud);
        }

        .panel-title {
            font-size: 0.85rem;
            color: var(--gold-anodized);
            border-bottom: 1px solid var(--border-hud);
            padding-bottom: 5px;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        /* GRUPOS DE BOTÕES E CONTROLES */
        .btn-grid {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 6px;
        }

        button {
            background: rgba(0, 240, 255, 0.05);
            border: 1px solid var(--border-hud);
            color: var(--hud-blue);
            padding: 8px 5px;
            font-size: 0.7rem;
            font-weight: bold;
            cursor: pointer;
            border-radius: 4px;
            transition: all 0.2s;
            text-transform: uppercase;
        }

        button:hover, button.active {
            background: var(--hud-blue);
            color: #000;
            box-shadow: 0 0 10px var(--hud-blue);
        }

        button.btn-mode {
            border-color: var(--gold-anodized);
            color: var(--gold-anodized);
        }

        button.btn-mode:hover, button.btn-mode.active {
            background: var(--gold-anodized);
            color: #000;
            box-shadow: 0 0 10px var(--gold-anodized);
        }

        /* CALCULADORAS / METRICAS */
        .calc-box {
            background: rgba(0, 0, 0, 0.4);
            border: 1px solid rgba(212, 175, 55, 0.3);
            border-radius: 4px;
            padding: 10px;
            font-size: 0.75rem;
        }

        .calc-box label {
            display: block;
            color: var(--titanium-silver);
            margin-bottom: 3px;
        }

        .calc-box input[type="range"] {
            width: 100%;
            margin-bottom: 8px;
            accent-color: var(--hud-blue);
        }

        .calc-output {
            color: var(--reactor-cyan);
            font-family: monospace;
            font-weight: bold;
            display: flex;
            justify-content: space-between;
        }

        /* VIEWPORT 3D */
        #viewport {
            width: 100%;
            height: 100%;
            position: relative;
        }

        /* HOVER OVERLAY HOLOGRÁFICO DA TELA */
        .viewport-hud {
            position: absolute;
            top: 15px;
            left: 15px;
            pointer-events: none;
            font-family: monospace;
            font-size: 0.75rem;
            color: var(--hud-blue);
            line-height: 1.4;
            background: rgba(3, 7, 18, 0.6);
            padding: 10px;
            border-radius: 4px;
            border: 1px solid var(--border-hud);
        }

        /* ACIONADORES DE FUNÇÕES */
        .func-toggle {
            display: flex;
            align-items: center;
            justify-content: space-between;
            font-size: 0.75rem;
            background: rgba(255, 255, 255, 0.03);
            padding: 6px 8px;
            border-radius: 3px;
            margin-bottom: 4px;
            border: 1px solid rgba(255,255,255,0.05);
        }

        .switch {
            position: relative;
            display: inline-block;
            width: 34px;
            height: 18px;
        }

        .switch input { opacity: 0; width: 0; height: 0; }

        .slider {
            position: absolute; cursor: pointer; top: 0; left: 0; right: 0; bottom: 0;
            background-color: #333; transition: .3s; border-radius: 18px;
        }

        .slider:before {
            position: absolute; content: ""; height: 12px; width: 12px; left: 3px; bottom: 3px;
            background-color: white; transition: .3s; border-radius: 50%;
        }

        input:checked + .slider { background-color: var(--hud-blue); }
        input:checked + .slider:before { transform: translateX(16px); }

        /* SCROLLBAR CUSTOM */
        ::-webkit-scrollbar { width: 5px; }
        ::-webkit-scrollbar-track { background: rgba(0,0,0,0.2); }
        ::-webkit-scrollbar-thumb { background: var(--border-hud); border-radius: 3px; }
    </style>

    <!-- Three.js e OrbitControls -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
</head>
<body>

    <header>
        <h1>ARMADURA NANOTECNOLÓGICA - MARK LXXXV COOP</h1>
        <div class="sys-badges">
            <div class="badge">M.A.R.S.O.S.: <span id="st-marsos" style="color:#4ade80">ONLINE</span></div>
            <div class="badge">SROS: <span id="st-sros" style="color:#4ade80">SYNC</span></div>
            <div class="badge">S.A.M.S./MHD: <span id="st-mhd" style="color:#4ade80">1.21 GW</span></div>
        </div>
    </header>

    <div class="app-container">
        <!-- PAINEL ESQUERDO: CONTROLES DE CÂMERA E VISUALIZAÇÃO -->
        <aside class="panel">
            <div class="panel-title">1. Vistas e Ângulos de Câmera</div>
            <div class="btn-grid">
                <button class="active" onclick="setCameraView('FRONTAL')">Frontal</button>
                <button onclick="setCameraView('POSTERIOR')">Posterior</button>
                <button onclick="setCameraView('LATERAL_DIR')">Lat. Direita</button>
                <button onclick="setCameraView('LATERAL_ESQ')">Lat. Esquerda</button>
                <button onclick="setCameraView('SUPERIOR')">Superior</button>
                <button onclick="setCameraView('INFERIOR')">Inferior</button>
            </div>

            <div class="panel-title">2. Modo de Visão Estrutural</div>
            <div class="btn-grid">
                <button id="btn-vis-ext" class="btn-mode active" onclick="setVisMode('EXTERNA')">Externa</button>
                <button id="btn-vis-int" class="btn-mode" onclick="setVisMode('INTERNA')">Interna (Raio-X)</button>
                <button id="btn-vis-hib" class="btn-mode" onclick="setVisMode('HIBRIDA')" style="grid-column: span 2">Híbrida (Transparente)</button>
            </div>

            <div class="panel-title">3. Calculadoras Operacionais em Tempo Real</div>
            
            <!-- Calculadora 1: Reator MHD -->
            <div class="calc-box">
                <label>Potência do Reator MHD (GW):</label>
                <input type="range" id="input-mhd" min="0.1" max="2.5" step="0.1" value="1.21" oninput="calcularMHD()">
                <div class="calc-output">
                    <span>Campo Escudo:</span>
                    <span id="out-escudo">121.0 Tesla</span>
                </div>
                <div class="calc-output">
                    <span>Empuxo Plasma:</span>
                    <span id="out-empuxo">84.7 kN</span>
                </div>
            </div>

            <!-- Calculadora 2: Reciclagem CO2 Carbono M.A.R.S.O.S. -->
            <div class="calc-box">
                <label>Fluxo CO2 Respiratório (L/min):</label>
                <input type="range" id="input-co2" min="0.2" max="2.0" step="0.1" value="0.8" oninput="calcularCarbono()">
                <div class="calc-output">
                    <span>Depósito Carbono:</span>
                    <span id="out-carbono">100% (Pronto)</span>
                </div>
                <div class="calc-output">
                    <span>Taxa Reparo Nanotubos:</span>
                    <span id="out-reparo">4.8 mm²/s</span>
                </div>
            </div>

            <!-- Calculadora 3: Amortecimento S.A.M.S. -->
            <div class="calc-box">
                <label>Força de Impacto Recebida (kN):</label>
                <input type="range" id="input-impacto" min="0" max="500" step="10" value="150" oninput="calcularSAMS()">
                <div class="calc-output">
                    <span>Absorção Cinética:</span>
                    <span id="out-sams">98.5%</span>
                </div>
                <div class="calc-output">
                    <span>Estresse Ósseo Astronauta:</span>
                    <span id="out-ossos">0.0 MPa (Seguro)</span>
                </div>
            </div>
        </aside>

        <!-- VIEWPORT 3D REALISTA (THREE.JS) -->
        <main id="viewport">
            <div class="viewport-hud">
                DIGITAL TWIN - ARMADURA MARK LXXXV<br>
                ANGULO: <span id="hud-angulo">FRONTAL</span><br>
                CAMADA: <span id="hud-camada">EXTERNA</span><br>
                SISTEMA COOPERATIVO: TRIPLE-CORE ONLINE
            </div>
        </main>

        <!-- PAINEL DIREITO: ESTADOS DAS FUNÇÕES DA ARMADURA -->
        <aside class="panel panel-right">
            <div class="panel-title">4. Simulação Ativa de Funções</div>

            <!-- 01. CAPACETE -->
            <div class="calc-box">
                <label style="color:var(--gold-anodized); font-weight:bold;">01. CAPACETE E CÚPULA</label>
                <div class="func-toggle">
                    <span>HUD Holográfico SROS</span>
                    <label class="switch"><input type="checkbox" id="sw-hud" checked onchange="toggleFuncao('hud')"><span class="slider"></span></label>
                </div>
                <div class="func-toggle">
                    <span>Protetor Ocular LCD</span>
                    <label class="switch"><input type="checkbox" id="sw-lcd" onchange="toggleFuncao('lcd')"><span class="slider"></span></label>
                </div>
                <div class="func-toggle">
                    <span>Alimentação/Água IA</span>
                    <label class="switch"><input type="checkbox" id="sw-nutri" checked><span class="slider"></span></label>
                </div>
            </div>

            <!-- 02. TORSO -->
            <div class="calc-box">
                <label style="color:var(--gold-anodized); font-weight:bold;">02. CHASSI TORÁCICO</label>
                <div class="func-toggle">
                    <span>Reator MHD & Escudo</span>
                    <label class="switch"><input type="checkbox" id="sw-mhd" checked onchange="toggleFuncao('mhd')"><span class="slider"></span></label>
                </div>
                <div class="func-toggle">
                    <span>M.A.R.S.O.S. Digital Twin</span>
                    <label class="switch"><input type="checkbox" id="sw-marsos" checked><span class="slider"></span></label>
                </div>
                <div class="func-toggle">
                    <span>Depósito CO2/Carbono</span>
                    <label class="switch"><input type="checkbox" id="sw-co2" checked><span class="slider"></span></label>
                </div>
            </div>

            <!-- 03. BRAÇOS -->
            <div class="calc-box">
                <label style="color:var(--gold-anodized); font-weight:bold;">03. OMBROS E BRAÇOS</label>
                <div class="func-toggle">
                    <span>Placas S.A.M.S. Cinéticas</span>
                    <label class="switch"><input type="checkbox" id="sw-sams" checked onchange="toggleFuncao('sams')"><span class="slider"></span></label>
                </div>
                <div class="func-toggle">
                    <span>Micro-injetores Carbono</span>
                    <label class="switch"><input type="checkbox" id="sw-injetor" onchange="toggleFuncao('injetor')"><span class="slider"></span></label>
                </div>
            </div>

            <!-- 04. MANOPLAS -->
            <div class="calc-box">
                <label style="color:var(--gold-anodized); font-weight:bold;">04. MANOPLAS (MÃOS)</label>
                <div class="func-toggle">
                    <span>Bobinas Faraday / Plasma</span>
                    <label class="switch"><input type="checkbox" id="sw-plasma" checked onchange="toggleFuncao('plasma')"><span class="slider"></span></label>
                </div>
                <div class="func-toggle">
                    <span>Sensoriamento Háptico</span>
                    <label class="switch"><input type="checkbox" id="sw-haptico" checked><span class="slider"></span></label>
                </div>
            </div>

            <!-- 05. PÉLVICO -->
            <div class="calc-box">
                <label style="color:var(--gold-anodized); font-weight:bold;">05. CINTURÃO PÉLVICO</label>
                <div class="func-toggle">
                    <span>Bio-Reator de Reciclagem</span>
                    <label class="switch"><input type="checkbox" id="sw-bioreator" checked><span class="slider"></span></label>
                </div>
                <div class="func-toggle">
                    <span>Tanques Hélio/Hidrogênio</span>
                    <label class="switch"><input type="checkbox" id="sw-tanques" checked><span class="slider"></span></label>
                </div>
            </div>

            <!-- 06. PERNAS -->
            <div class="calc-box">
                <label style="color:var(--gold-anodized); font-weight:bold;">06. PERNAS E COXAS</label>
                <div class="func-toggle">
                    <span>Exoesqueleto Magnético</span>
                    <label class="switch"><input type="checkbox" id="sw-exo" checked><span class="slider"></span></label>
                </div>
                <div class="func-toggle">
                    <span>Radiadores Térmicos</span>
                    <label class="switch"><input type="checkbox" id="sw-rad" checked><span class="slider"></span></label>
                </div>
            </div>

            <!-- 07. BOTAS -->
            <div class="calc-box">
                <label style="color:var(--gold-anodized); font-weight:bold;">07. BOTAS E PROPULSORES</label>
                <div class="func-toggle">
                    <span>Propulsão Plasma Ciclo Fechado</span>
                    <label class="switch"><input type="checkbox" id="sw-prop" checked onchange="toggleFuncao('prop')"><span class="slider"></span></label>
                </div>
                <div class="func-toggle">
                    <span>Âncoras Magnéticas 0G</span>
                    <label class="switch"><input type="checkbox" id="sw-ancora" onchange="toggleFuncao('ancora')"><span class="slider"></span></label>
                </div>
            </div>
        </aside>
    </div>

    <script>
        /* ==========================================================================
           CONFIGURAÇÃO THREE.JS REALISTA - ARMADURA MARK LXXXV
           ========================================================================== */
        const container = document.getElementById('viewport');
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x030712);

        // Câmera
        const camera = new THREE.PerspectiveCamera(45, container.clientWidth / container.clientHeight, 0.1, 1000);
        camera.position.set(0, 0, 8.5);

        // Renderer com sombras e tomemapping para aspecto metálico realista
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(container.clientWidth, container.clientHeight);
        renderer.setPixelRatio(window.devicePixelRatio);
        renderer.shadowMap.enabled = true;
        renderer.toneMapping = THREE.ACESFilmicToneMapping;
        renderer.toneMappingExposure = 1.2;
        container.appendChild(renderer.domElement);

        // Controls
        const controls = new THREE.OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.dampingFactor = 0.05;

        /* ILUMINAÇÃO ESTÚDIO METÁLICO */
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.8);
        scene.add(ambientLight);

        const dirLight1 = new THREE.DirectionalLight(0xffffff, 1.5);
        dirLight1.position.set(5, 10, 7);
        scene.add(dirLight1);

        const dirLight2 = new THREE.DirectionalLight(0x00f0ff, 1.0); // Rim light cyan
        dirLight2.position.set(-5, -5, -5);
        scene.add(dirLight2);

        const pointLightCore = new THREE.PointLight(0x00ffff, 2.0, 3);
        pointLightCore.position.set(0, 0.7, 0.4);
        scene.add(pointLightCore);

        /* MATERIAIS REALISTAS DA ARMADURA */
        const matRedMetallic = new THREE.MeshStandardMaterial({
            color: 0x8b0000, metalness: 0.85, roughness: 0.25, name: 'externo'
        });
        const matGoldAnodized = new THREE.MeshStandardMaterial({
            color: 0xd4af37, metalness: 0.9, roughness: 0.2, name: 'externo'
        });
        const matTitaniumSilver = new THREE.MeshStandardMaterial({
            color: 0xa8b2c1, metalness: 0.95, roughness: 0.15, name: 'externo'
        });
        const matGlowCyan = new THREE.MeshBasicMaterial({ color: 0x00ffff });
        const matGlowOrange = new THREE.MeshBasicMaterial({ color: 0xff5500 });

        // Materiais Internos (Estrutura/Sistemas)
        const matInternalMesh = new THREE.MeshStandardMaterial({
            color: 0x22252a, metalness: 0.5, roughness: 0.5, wireframe: false, name: 'interno'
        });
        const matInternalGlow = new THREE.MeshBasicMaterial({ color: 0x00f0ff, wireframe: true, name: 'interno' });

        /* MONTAGEM DA ARMADURA 3D HIERÁRQUICA */
        const armorGroup = new THREE.Group();
        const externalGroup = new THREE.Group();
        const internalGroup = new THREE.Group();
        const fxGroup = new THREE.Group();

        armorGroup.add(externalGroup);
        armorGroup.add(internalGroup);
        armorGroup.add(fxGroup);
        scene.add(armorGroup);

        // 01. CAPACETE
        const helmetExt = new THREE.Mesh(new THREE.SphereGeometry(0.38, 32, 16), matRedMetallic);
        helmetExt.position.set(0, 1.8, 0);
        helmetExt.scale.set(0.9, 1.1, 1.0);
        externalGroup.add(helmetExt);

        const facePlate = new THREE.Mesh(new THREE.ConeGeometry(0.3, 0.4, 4), matGoldAnodized);
        facePlate.position.set(0, 1.78, 0.2);
        facePlate.rotation.x = -Math.PI / 4;
        externalGroup.add(facePlate);

        // Eyes Glow / LCD
        const eyeMat = new THREE.MeshBasicMaterial({ color: 0x00ffff });
        const eyes = new THREE.Mesh(new THREE.BoxGeometry(0.25, 0.04, 0.1), eyeMat);
        eyes.position.set(0, 1.85, 0.33);
        externalGroup.add(eyes);

        // 02. TORSO / PEITO E DORSO
        const chestExt = new THREE.Mesh(new THREE.CylinderGeometry(0.7, 0.5, 1.1, 8), matRedMetallic);
        chestExt.position.set(0, 0.8, 0);
        externalGroup.add(chestExt);

        const chestGold = new THREE.Mesh(new THREE.BoxGeometry(0.8, 0.6, 0.6), matGoldAnodized);
        chestGold.position.set(0, 0.9, 0.1);
        externalGroup.add(chestGold);

        // Reator MHD Arc Core
        const coreGeo = new THREE.OctahedronGeometry(0.18, 2);
        const coreMesh = new THREE.Mesh(coreGeo, matGlowCyan);
        coreMesh.position.set(0, 0.9, 0.35);
        externalGroup.add(coreMesh);

        // 03. OMBROS E BRAÇOS
        [-0.85, 0.85].forEach((x, idx) => {
            // Ombro S.A.M.S.
            const shoulder = new THREE.Mesh(new THREE.SphereGeometry(0.3, 16, 16), matGoldAnodized);
            shoulder.position.set(x, 1.2, 0);
            externalGroup.add(shoulder);

            // Braço
            const arm = new THREE.Mesh(new THREE.CylinderGeometry(0.15, 0.12, 0.8), matRedMetallic);
            arm.position.set(x, 0.6, 0);
            externalGroup.add(arm);

            // 04. MANOPLAS
            const hand = new THREE.Mesh(new THREE.BoxGeometry(0.18, 0.22, 0.18), matGoldAnodized);
            hand.position.set(x, 0.1, 0);
            externalGroup.add(hand);

            // Repulsor Palma
            const repulsor = new THREE.Mesh(new THREE.CircleGeometry(0.05, 16), matGlowCyan);
            repulsor.position.set(x, 0.05, 0.1);
            externalGroup.add(repulsor);
        });

        // 05. CINTURÃO PÉLVICO
        const pelvis = new THREE.Mesh(new THREE.CylinderGeometry(0.5, 0.4, 0.35, 6), matTitaniumSilver);
        pelvis.position.set(0, 0.1, 0);
        externalGroup.add(pelvis);

        // 06. PERNAS E COXAS
        [-0.32, 0.32].forEach(x => {
            const thigh = new THREE.Mesh(new THREE.CylinderGeometry(0.2, 0.15, 0.9), matRedMetallic);
            thigh.position.set(x, -0.5, 0);
            externalGroup.add(thigh);

            const knee = new THREE.Mesh(new THREE.SphereGeometry(0.16, 16, 16), matGoldAnodized);
            knee.position.set(x, -0.95, 0.05);
            externalGroup.add(knee);

            const calf = new THREE.Mesh(new THREE.CylinderGeometry(0.15, 0.12, 0.9), matTitaniumSilver);
            calf.position.set(x, -1.45, 0);
            externalGroup.add(calf);

            // 07. BOTAS E PROPULSORES
            const boot = new THREE.Mesh(new THREE.BoxGeometry(0.22, 0.2, 0.4), matRedMetallic);
            boot.position.set(x, -1.95, 0.08);
            externalGroup.add(boot);

            // Bocal Plasma Bota
            const thruster = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.02, 0.15), matGlowCyan);
            thruster.position.set(x, -2.05, 0);
            externalGroup.add(thruster);
        });

        /* CONSTRUÇÃO DA ESTRUTURA INTERNA (RAIO-X / DIGITAL TWIN) */
        const spine = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.08, 2.5), matInternalGlow);
        spine.position.set(0, 0.2, -0.1);
        internalGroup.add(spine);

        const bioReactor = new THREE.Mesh(new THREE.DodecahedronGeometry(0.25), matInternalGlow);
        bioReactor.position.set(0, 0.1, 0);
        internalGroup.add(bioReactor);

        const marsosComp = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.2, 0.2), new THREE.MeshBasicMaterial({ color: 0xff0055, wireframe: true }));
        marsosComp.position.set(0, 1.1, -0.2);
        internalGroup.add(marsosComp);

        // Efeitos Dinâmicos (Escudo Deflector MHD)
        const shieldGeo = new THREE.SphereGeometry(2.4, 32, 16);
        const shieldMat = new THREE.MeshBasicMaterial({ color: 0x00f0ff, transparent: true, opacity: 0.15, wireframe: true });
        const shieldMesh = new THREE.Mesh(shieldGeo, shieldMat);
        shieldMesh.visible = false;
        fxGroup.add(shieldMesh);

        /* ==========================================================================
           CONTROLES DE CÂMERA E VISTAS
           ========================================================================== */
        function setCameraView(vista) {
            document.getElementById('hud-angulo').innerText = vista;

            // Transição suave de câmera
            const targetPos = new THREE.Vector3();
            switch (vista) {
                case 'FRONTAL': targetPos.set(0, 0, 8.5); break;
                case 'POSTERIOR': targetPos.set(0, 0, -8.5); break;
                case 'LATERAL_DIR': targetPos.set(8.5, 0, 0); break;
                case 'LATERAL_ESQ': targetPos.set(-8.5, 0, 0); break;
                case 'SUPERIOR': targetPos.set(0, 8.5, 0.1); break;
                case 'INFERIOR': targetPos.set(0, -8.5, 0.1); break;
            }

            camera.position.copy(targetPos);
            controls.target.set(0, 0, 0);
            controls.update();

            // Atualizar botões
            document.querySelectorAll('.panel .btn-grid button').forEach(btn => {
                btn.classList.remove('active');
                if (btn.innerText.toUpperCase().includes(vista.replace('_', ' '))) btn.classList.add('active');
            });
        }

        function setVisMode(modo) {
            document.getElementById('hud-camada').innerText = modo;

            document.getElementById('btn-vis-ext').classList.remove('active');
            document.getElementById('btn-vis-int').classList.remove('active');
            document.getElementById('btn-vis-hib').classList.remove('active');

            if (modo === 'EXTERNA') {
                document.getElementById('btn-vis-ext').classList.add('active');
                externalGroup.visible = true;
                internalGroup.visible = false;
                setArmourOpacity(1.0, false);
            } else if (modo === 'INTERNA') {
                document.getElementById('btn-vis-int').classList.add('active');
                externalGroup.visible = false;
                internalGroup.visible = true;
            } else if (modo === 'HIBRIDA') {
                document.getElementById('btn-vis-hib').classList.add('active');
                externalGroup.visible = true;
                internalGroup.visible = true;
                setArmourOpacity(0.35, true);
            }
        }

        function setArmourOpacity(opacity, transparent) {
            externalGroup.traverse((child) => {
                if (child.isMesh && child.material.name === 'externo') {
                    child.material.transparent = transparent;
                    child.material.opacity = opacity;
                }
            });
        }

        /* ==========================================================================
           CALCULADORAS EM TEMPO REAL
           ========================================================================== */
        function calcularMHD() {
            const gw = parseFloat(document.getElementById('input-mhd').value);
            const tesla = (gw * 100).toFixed(1);
            const empuxo = (gw * 70).toFixed(1);

            document.getElementById('out-escudo').innerText = `${tesla} Tesla`;
            document.getElementById('out-empuxo').innerText = `${empuxo} kN`;
        }

        function calcularCarbono() {
            const co2 = parseFloat(document.getElementById('input-co2').value);
            const taxa = (co2 * 6).toFixed(1);
            document.getElementById('out-reparo').innerText = `${taxa} mm²/s`;
        }

        function calcularSAMS() {
            const impacto = parseFloat(document.getElementById('input-impacto').value);
            let absorcao = 99.5 - (impacto * 0.02);
            if (absorcao < 70) absorcao = 70;
            const estresse = impacto > 350 ? ((impacto - 350) * 0.1).toFixed(1) : 0.0;

            document.getElementById('out-sams').innerText = `${absorcao.toFixed(1)}%`;
            document.getElementById('out-ossos').innerText = `${estresse} MPa (${estresse > 0 ? 'ATENÇÃO' : 'Seguro'})`;
        }

        /* SIMULAÇÕES VISUAIS DAS FUNÇÕES DA ARMADURA */
        function toggleFuncao(func) {
            if (func === 'hud') {
                const checked = document.getElementById('sw-hud').checked;
                eyes.material.color.setHex(checked ? 0x00ffff : 0x222222);
            } else if (func === 'lcd') {
                const checked = document.getElementById('sw-lcd').checked;
                facePlate.material.color.setHex(checked ? 0x111111 : 0xd4af37);
            } else if (func === 'mhd') {
                const checked = document.getElementById('sw-mhd').checked;
                shieldMesh.visible = checked;
            } else if (func === 'prop') {
                const checked = document.getElementById('sw-prop').checked;
                pointLightCore.intensity = checked ? 2.0 : 0.2;
            }
        }

        /* LOOP DE ANIMAÇÃO 60 FPS */
        function animate() {
            requestAnimationFrame(animate);

            // Rotação sutil do Reator Arc MHD
            coreMesh.rotation.z += 0.02;
            marsosComp.rotation.y += 0.03;

            // Pulsação do Escudo Deflector se ativo
            if (shieldMesh.visible) {
                shieldMesh.rotation.y += 0.005;
            }

            controls.update();
            renderer.render(scene, camera);
        }

        window.addEventListener('resize', () => {
            camera.aspect = container.clientWidth / container.clientHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(container.clientWidth, container.clientHeight);
        });

        // Inicialização
        calcularMHD();
        calcularCarbono();
        calcularSAMS();
        animate();
    </script>
</body>
</html>
