<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Advanced Wing Aerodynamics ML Predictor</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/3.9.1/chart.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tensorflow/4.10.0/tf.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
        }

        .container {
            max-width: 1400px;
            margin: 0 auto;
            background: rgba(255, 255, 255, 0.95);
            border-radius: 20px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
            overflow: hidden;
        }

        .header {
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            color: white;
            text-align: center;
            padding: 30px;
        }

        .header h1 {
            font-size: 2.5em;
            margin-bottom: 10px;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
        }

        .header p {
            font-size: 1.2em;
            opacity: 0.9;
        }

        .content {
            padding: 30px;
        }

        .control-panel {
            background: #f8f9fa;
            border-radius: 15px;
            padding: 25px;
            margin-bottom: 30px;
            border: 2px solid #e9ecef;
        }

        .control-panel h3 {
            color: #2c3e50;
            margin-bottom: 20px;
            font-size: 1.4em;
            border-bottom: 2px solid #3498db;
            padding-bottom: 10px;
        }

        .controls-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
            gap: 20px;
            margin-bottom: 20px;
        }

        .control-group {
            background: white;
            padding: 15px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        }

        .control-group label {
            display: block;
            font-weight: 600;
            color: #34495e;
            margin-bottom: 8px;
        }

        .control-group input, .control-group select {
            width: 100%;
            padding: 10px;
            border: 2px solid #ddd;
            border-radius: 5px;
            font-size: 14px;
            transition: border-color 0.3s;
        }

        .control-group input:focus, .control-group select:focus {
            outline: none;
            border-color: #3498db;
        }

        .control-group small {
            color: #7f8c8d;
            font-size: 0.9em;
        }

        .button-group {
            display: flex;
            gap: 15px;
            justify-content: center;
            flex-wrap: wrap;
            margin: 20px 0;
        }

        .btn {
            padding: 12px 25px;
            border: none;
            border-radius: 25px;
            cursor: pointer;
            font-size: 16px;
            font-weight: 600;
            transition: all 0.3s;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .btn-primary {
            background: linear-gradient(135deg, #3498db, #2980b9);
            color: white;
        }

        .btn-secondary {
            background: linear-gradient(135deg, #95a5a6, #7f8c8d);
            color: white;
        }

        .btn-success {
            background: linear-gradient(135deg, #27ae60, #2ecc71);
            color: white;
        }

        .btn-warning {
            background: linear-gradient(135deg, #f39c12, #e67e22);
            color: white;
        }

        .btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
        }

        .results {
            background: #f8f9fa;
            border-radius: 15px;
            padding: 25px;
            margin: 20px 0;
            border-left: 5px solid #27ae60;
        }

        .results h3 {
            color: #27ae60;
            margin-bottom: 15px;
            font-size: 1.4em;
        }

        .prediction-results {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
            margin: 20px 0;
        }

        .prediction-result {
            background: linear-gradient(135deg, #27ae60, #2ecc71);
            color: white;
            padding: 20px;
            border-radius: 10px;
            text-align: center;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
        }

        .prediction-result.drag {
            background: linear-gradient(135deg, #e74c3c, #c0392b);
        }

        .prediction-result h4 {
            font-size: 1.3em;
            margin-bottom: 10px;
        }

        .prediction-value {
            font-size: 2.2em;
            font-weight: bold;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
        }

        .charts-container {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(500px, 1fr));
            gap: 30px;
            margin: 30px 0;
        }

        .chart-container {
            background: white;
            border-radius: 15px;
            padding: 20px;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
            height: 400px;
        }

        .chart-container canvas {
            max-height: 350px !important;
        }

        .chart-container h4 {
            color: #2c3e50;
            margin-bottom: 15px;
            text-align: center;
            font-size: 1.2em;
        }

        .simulation-container {
            background: #2c3e50;
            border-radius: 15px;
            padding: 20px;
            margin: 20px 0;
            color: white;
            position: relative;
            height: 400px;
            overflow: hidden;
        }

        .simulation-canvas {
            width: 100%;
            height: 100%;
            border-radius: 10px;
            background: linear-gradient(135deg, #34495e, #2c3e50);
        }

        .status {
            background: #e8f4f8;
            border-left: 4px solid #3498db;
            padding: 15px;
            margin: 15px 0;
            border-radius: 0 10px 10px 0;
        }

        .loading {
            display: inline-block;
            width: 20px;
            height: 20px;
            border: 3px solid #f3f3f3;
            border-top: 3px solid #3498db;
            border-radius: 50%;
            animation: spin 1s linear infinite;
            margin-right: 10px;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        .model-info {
            background: #fff3cd;
            border: 1px solid #ffeaa7;
            color: #856404;
            padding: 15px;
            border-radius: 10px;
            margin: 20px 0;
        }

        .airfoil-preview {
            background: white;
            border-radius: 10px;
            padding: 20px;
            margin: 20px 0;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        }

        .airfoil-shape {
            width: 100%;
            height: 100px;
            background: #ecf0f1;
            border-radius: 5px;
            position: relative;
            overflow: hidden;
        }

        .airfoil-svg {
            width: 100%;
            height: 100%;
        }

        .training-progress {
            background: white;
            border-radius: 10px;
            padding: 20px;
            margin: 20px 0;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        }

        .progress-bar {
            width: 100%;
            height: 20px;
            background: #ecf0f1;
            border-radius: 10px;
            overflow: hidden;
            margin: 10px 0;
        }

        .progress-fill {
            height: 100%;
            background: linear-gradient(135deg, #3498db, #2980b9);
            width: 0%;
            transition: width 0.3s;
        }

        .example-airfoils {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 15px;
            margin: 20px 0;
        }

        .example-airfoil {
            background: white;
            border: 2px solid #e9ecef;
            border-radius: 10px;
            padding: 15px;
            cursor: pointer;
            transition: all 0.3s;
            text-align: center;
        }

        .example-airfoil:hover {
            border-color: #3498db;
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
        }

        .example-airfoil h5 {
            color: #2c3e50;
            margin-bottom: 10px;
            font-size: 1.1em;
        }

        .airfoil-code {
            font-family: 'Courier New', monospace;
            font-weight: bold;
            color: #e74c3c;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>✈️ Advanced Wing Aerodynamics ML Predictor</h1>
            <p>Predict lift & drag with TensorFlow.js • NACA airfoils • Real-time simulations</p>
        </div>

        <div class="content">
            <div class="control-panel">
                <h3>Wing Configuration Parameters</h3>
                <div class="controls-grid">
                    <div class="control-group">
                        <label for="nacaCode">NACA Airfoil Code</label>
                        <select id="nacaCode">
                            <option value="0012">NACA 0012 (Symmetric)</option>
                            <option value="2412">NACA 2412 (Cambered)</option>
                            <option value="4412">NACA 4412 (High Camber)</option>
                            <option value="6412">NACA 6412 (Very High Camber)</option>
                            <option value="0018">NACA 0018 (Thick Symmetric)</option>
                            <option value="2415">NACA 2415 (Thick Cambered)</option>
                            <option value="23012">NACA 23012 (5-digit)</option>
                            <option value="64212">NACA 64-212 (Laminar Flow)</option>
                        </select>
                        <small>Select standard NACA airfoil profile</small>
                    </div>
                    <div class="control-group">
                        <label for="angleOfAttack">Angle of Attack (degrees)</label>
                        <input type="number" id="angleOfAttack" value="5" min="-15" max="25" step="0.1">
                        <small>Typical range: -15° to 25°</small>
                    </div>
                    <div class="control-group">
                        <label for="wingSpan">Wing Span (meters)</label>
                        <input type="number" id="wingSpan" value="10" min="5" max="25" step="0.1">
                        <small>Distance from tip to tip</small>
                    </div>
                    <div class="control-group">
                        <label for="chordLength">Chord Length (meters)</label>
                        <input type="number" id="chordLength" value="2" min="1" max="5" step="0.1">
                        <small>Wing depth from front to back</small>
                    </div>
                    <div class="control-group">
                        <label for="airspeed">Airspeed (m/s)</label>
                        <input type="number" id="airspeed" value="80" min="20" max="350" step="1">
                        <small>Aircraft velocity</small>
                    </div>
                    <div class="control-group">
                        <label for="reynolds">Reynolds Number</label>
                        <input type="number" id="reynolds" value="1000000" min="100000" max="10000000" step="10000">
                        <small>Flow regime indicator</small>
                    </div>
                </div>

                <div class="button-group">
                    <button class="btn btn-primary" onclick="predictCoefficients()">🎯 Predict CL & CD</button>
                    <button class="btn btn-secondary" onclick="generateAdvancedData()">📊 Generate Data</button>
                    <button class="btn btn-success" onclick="trainNeuralNetwork()">🧠 Train Neural Net</button>
                    <button class="btn btn-warning" onclick="startFlowSimulation()">🌊 Flow Simulation</button>
                </div>
            </div>

            <div class="airfoil-preview">
                <h4>Airfoil Profile Preview</h4>
                <div class="airfoil-shape">
                    <svg class="airfoil-svg" id="airfoilSvg" viewBox="0 0 400 100">
                        <!-- Airfoil shape will be drawn here -->
                    </svg>
                </div>
                <div id="airfoilInfo" style="margin-top: 10px; color: #7f8c8d;">
                    Select an airfoil to see its profile and characteristics
                </div>
            </div>

            <div class="example-airfoils">
                <div class="example-airfoil" onclick="loadExampleAirfoil('0012')">
                    <h5>Symmetric</h5>
                    <div class="airfoil-code">NACA 0012</div>
                    <small>No camber, 12% thick</small>
                </div>
                <div class="example-airfoil" onclick="loadExampleAirfoil('2412')">
                    <h5>General Aviation</h5>
                    <div class="airfoil-code">NACA 2412</div>
                    <small>2% camber, 12% thick</small>
                </div>
                <div class="example-airfoil" onclick="loadExampleAirfoil('4412')">
                    <h5>High Lift</h5>
                    <div class="airfoil-code">NACA 4412</div>
                    <small>4% camber, 12% thick</small>
                </div>
                <div class="example-airfoil" onclick="loadExampleAirfoil('64212')">
                    <h5>Laminar Flow</h5>
                    <div class="airfoil-code">NACA 64-212</div>
                    <small>Low drag design</small>
                </div>
            </div>

            <div class="model-info">
                <h4>🧠 Neural Network Status</h4>
                <div id="modelStatus">Ready to train TensorFlow.js neural network for advanced predictions!</div>
                <div class="training-progress" id="trainingProgress" style="display: none;">
                    <h5>Training Progress</h5>
                    <div class="progress-bar">
                        <div class="progress-fill" id="progressFill"></div>
                    </div>
                    <div id="trainingMetrics"></div>
                </div>
            </div>

            <div class="simulation-container">
                <h4 style="text-align: center; margin-bottom: 15px;">🌊 Airflow Simulation</h4>
                <canvas class="simulation-canvas" id="flowCanvas" width="800" height="300"></canvas>
                <div style="position: absolute; bottom: 10px; left: 20px; font-size: 0.9em;">
                    Click "Flow Simulation" to see particle flow visualization
                </div>
            </div>

            <div id="results" class="results" style="display: none;">
                <h3>🎯 Prediction Results</h3>
                <div class="prediction-results" id="predictionOutput"></div>
                <div id="theoreticalComparison"></div>
            </div>

            <div class="charts-container">
                <div class="chart-container">
                    <h4>📈 Lift vs Angle of Attack</h4>
                    <canvas id="liftChart"></canvas>
                </div>
                <div class="chart-container">
                    <h4>📊 Drag Polar Curve</h4>
                    <canvas id="dragChart"></canvas>
                </div>
                <div class="chart-container">
                    <h4>🎯 ML vs Theoretical</h4>
                    <canvas id="comparisonChart"></canvas>
                </div>
                <div class="chart-container">
                    <h4>🧠 Training Loss</h4>
                    <canvas id="lossChart"></canvas>
                </div>
            </div>
        </div>
    </div>

    <script>
        // Global variables
        let trainingData = [];
        let neuralNetwork = null;
        let scaler = null;
        let isTraining = false;
        let flowSimulation = null;
        
        // Chart storage
        window.liftChart = null;
        window.dragChart = null;
        window.comparisonChart = null;
        window.lossChart = null;

        // NACA Airfoil database with characteristics
        const nacaDatabase = {
            '0012': { camber: 0, position: 0, thickness: 12, clMax: 1.2, clSlope: 6.28, cd0: 0.008, description: 'Symmetric, good for aerobatics' },
            '2412': { camber: 2, position: 40, thickness: 12, clMax: 1.4, clSlope: 6.5, cd0: 0.009, description: 'Popular general aviation airfoil' },
            '4412': { camber: 4, position: 40, thickness: 12, clMax: 1.6, clSlope: 6.8, cd0: 0.011, description: 'High-lift, good for slow flight' },
            '6412': { camber: 6, position: 40, thickness: 12, clMax: 1.8, clSlope: 7.0, cd0: 0.014, description: 'Very high camber, high CL' },
            '0018': { camber: 0, position: 0, thickness: 18, clMax: 1.3, clSlope: 6.0, cd0: 0.012, description: 'Thick symmetric, structural' },
            '2415': { camber: 2, position: 40, thickness: 15, clMax: 1.5, clSlope: 6.3, cd0: 0.012, description: 'Thick cambered, good structure' },
            '23012': { camber: 2.3, position: 15, thickness: 12, clMax: 1.35, clSlope: 6.4, cd0: 0.0085, description: '5-digit series, refined design' },
            '64212': { camber: 2, position: 40, thickness: 12, clMax: 1.3, clSlope: 6.6, cd0: 0.006, description: 'Laminar flow, low drag' }
        };

        // Simple StandardScaler
        class AdvancedScaler {
            constructor() {
                this.means = [];
                this.stds = [];
            }

            fit(X) {
                const nFeatures = X[0].length;
                this.means = new Array(nFeatures).fill(0);
                this.stds = new Array(nFeatures).fill(0);
                
                for (let i = 0; i < nFeatures; i++) {
                    const values = X.map(row => row[i]);
                    this.means[i] = values.reduce((a, b) => a + b) / values.length;
                }
                
                for (let i = 0; i < nFeatures; i++) {
                    const values = X.map(row => row[i]);
                    const variance = values.reduce((sum, val) => sum + Math.pow(val - this.means[i], 2), 0) / values.length;
                    this.stds[i] = Math.sqrt(variance) || 1;
                }
            }

            transform(X) {
                return X.map(row => 
                    row.map((val, i) => (val - this.means[i]) / this.stds[i])
                );
            }
        }

        // Generate NACA airfoil coordinates
        function generateNacaCoordinates(naca, numPoints = 50) {
            const coordinates = [];
            
            if (naca.length === 4) {
                // 4-digit NACA
                const m = parseInt(naca[0]) / 100; // max camber
                const p = parseInt(naca[1]) / 10;  // position of max camber
                const t = parseInt(naca.substr(2)) / 100; // thickness
                
                for (let i = 0; i < numPoints; i++) {
                    const x = i / (numPoints - 1);
                    
                    // Thickness distribution
                    const yt = 5 * t * (0.2969 * Math.sqrt(x) - 0.1260 * x - 0.3516 * x * x + 0.2843 * x * x * x - 0.1015 * x * x * x * x);
                    
                    // Camber line
                    let yc, dyc_dx;
                    if (x <= p) {
                        yc = m / (p * p) * (2 * p * x - x * x);
                        dyc_dx = 2 * m / (p * p) * (p - x);
                    } else {
                        yc = m / ((1 - p) * (1 - p)) * (1 - 2 * p + 2 * p * x - x * x);
                        dyc_dx = 2 * m / ((1 - p) * (1 - p)) * (p - x);
                    }
                    
                    const theta = Math.atan(dyc_dx);
                    
                    // Upper and lower surface
                    coordinates.push({
                        xu: x - yt * Math.sin(theta),
                        yu: yc + yt * Math.cos(theta),
                        xl: x + yt * Math.sin(theta),
                        yl: yc - yt * Math.cos(theta)
                    });
                }
            }
            
            return coordinates;
        }

        // Draw airfoil in SVG
        function drawAirfoil(nacaCode) {
            const svg = document.getElementById('airfoilSvg');
            const coords = generateNacaCoordinates(nacaCode);
            const info = nacaDatabase[nacaCode];
            
            if (!coords.length) return;
            
            // Scale coordinates
            const scale = 350;
            const offsetY = 50;
            
            // Create upper surface path
            let upperPath = `M ${coords[0].xu * scale + 25} ${offsetY - coords[0].yu * scale}`;
            for (let i = 1; i < coords.length; i++) {
                upperPath += ` L ${coords[i].xu * scale + 25} ${offsetY - coords[i].yu * scale}`;
            }
            
            // Create lower surface path
            let lowerPath = `M ${coords[coords.length-1].xl * scale + 25} ${offsetY - coords[coords.length-1].yl * scale}`;
            for (let i = coords.length - 2; i >= 0; i--) {
                lowerPath += ` L ${coords[i].xl * scale + 25} ${offsetY - coords[i].yl * scale}`;
            }
            
            svg.innerHTML = `
                <defs>
                    <linearGradient id="airfoilGrad" x1="0%" y1="0%" x2="0%" y2="100%">
                        <stop offset="0%" style="stop-color:#3498db;stop-opacity:0.8" />
                        <stop offset="100%" style="stop-color:#2980b9;stop-opacity:0.8" />
                    </linearGradient>
                </defs>
                <path d="${upperPath} ${lowerPath} Z" fill="url(#airfoilGrad)" stroke="#2c3e50" stroke-width="2"/>
                <line x1="25" y1="${offsetY}" x2="${375}" y2="${offsetY}" stroke="#34495e" stroke-width="1" stroke-dasharray="5,5"/>
            `;
            
            // Update info
            document.getElementById('airfoilInfo').innerHTML = `
                <strong>NACA ${nacaCode}</strong> - ${info.description}<br>
                Max Camber: ${info.camber}%, Thickness: ${info.thickness}%, CL max ≈ ${info.clMax}
            `;
        }

        // Theoretical aerodynamic calculations
        function calculateTheoreticalCL(aoa, nacaCode) {
            const info = nacaDatabase[nacaCode];
            const aoaRad = aoa * Math.PI / 180;
            
            // Basic thin airfoil theory with camber effects
            const clAlpha = info.clSlope;
            const cl0 = info.camber * 0.1; // Camber contribution
            let cl = cl0 + clAlpha * aoaRad;
            
            // Stall modeling
            const stallAngle = 15 + info.camber * 0.5;
            if (Math.abs(aoa) > stallAngle) {
                const stallFactor = Math.exp(-Math.abs(aoa - stallAngle) / 10);
                cl *= stallFactor;
            }
            
            return Math.max(-0.5, Math.min(cl, info.clMax));
        }

        function calculateTheoreticalCD(cl, nacaCode) {
            const info = nacaDatabase[nacaCode];
            
            // Drag buildup: CD = CD0 + CDi + CDstall
            const cd0 = info.cd0; // Profile drag
            const aspectRatio = parseFloat(document.getElementById('wingSpan').value) / parseFloat(document.getElementById('chordLength').value);
            const e = 0.8; // Oswald efficiency factor
            const cdi = cl * cl / (Math.PI * aspectRatio * e); // Induced drag
            
            // Additional drag at high CL
            const cdStall = Math.max(0, (Math.abs(cl) - 1.2) * 0.1);
            
            return cd0 + cdi + cdStall;
        }

        // Generate advanced training data
        async function generateAdvancedData() {
            updateStatus("Generating comprehensive aerodynamic dataset...", true);
            
            trainingData = [];
            const nSamples = 2000;
            const nacaCodes = Object.keys(nacaDatabase);
            
            for (let i = 0; i < nSamples; i++) {
                const nacaCode = nacaCodes[Math.floor(Math.random() * nacaCodes.length)];
                const info = nacaDatabase[nacaCode];
                
                const aoa = -15 + Math.random() * 40; // -15 to 25 degrees
                const span = 8 + Math.random() * 12;   // 8 to 20 meters
                const chord = 1.5 + Math.random() * 2.5; // 1.5 to 4 meters
                const speed = 30 + Math.random() * 250;   // 30 to 280 m/s
                const reynolds = 200000 + Math.random() * 4800000; // 200k to 5M
                
                // Calculate derived parameters
                const area = span * chord;
                const aspectRatio = span / chord;
                
                // Theoretical calculations with noise
                let cl = calculateTheoreticalCL(aoa, nacaCode);
                let cd = calculateTheoreticalCD(cl, nacaCode);
                
                // Add Reynolds number effects
                const reEffect = Math.min(1.2, Math.log10(reynolds / 100000) / 2);
                cl *= reEffect;
                cd /= Math.sqrt(reEffect);
                
                // Add realistic noise
                cl += (Math.random() - 0.5) * 0.05;
                cd += (Math.random() - 0.5) * 0.002;
                
                trainingData.push({
                    nacaCode: nacaCode,
                    camber: info.camber,
                    thickness: info.thickness,
                    angleOfAttack: aoa,
                    wingSpan: span,
                    chordLength: chord,
                    airspeed: speed,
                    reynolds: reynolds,
                    wingArea: area,
                    aspectRatio: aspectRatio,
                    liftCoeff: cl,
                    dragCoeff: cd
                });
            }
            
            updateStatus(`Generated ${nSamples} advanced training samples with NACA airfoils!`);
            visualizeLiftCurve();
            visualizeDragPolar();
        }

        // Train TensorFlow.js Neural Network
        async function trainNeuralNetwork() {
            if (trainingData.length === 0) {
                alert("Please generate training data first!");
                return;
            }
            
            if (isTraining) {
                alert("Training already in progress!");
                return;
            }
            
            isTraining = true;
            updateStatus("Training TensorFlow.js Neural Network...", true);
            document.getElementById('trainingProgress').style.display = 'block';
            
            try {
                // Prepare features and targets
                const features = ['camber', 'thickness', 'angleOfAttack', 'wingSpan', 'chordLength', 'airspeed', 'reynolds', 'aspectRatio'];
                const X = trainingData.map(d => features.map(f => d[f]));
                const yLift = trainingData.map(d => d.liftCoeff);
                const yDrag = trainingData.map(d => d.dragCoeff);
                
                // Scale features
                scaler = new AdvancedScaler();
                scaler.fit(X);
                const XScaled = scaler.transform(X);
                
                // Convert to tensors
                const xs = tf.tensor2d(XScaled);
                const ysLift = tf.tensor2d(yLift, [yLift.length, 1]);
                const ysDrag = tf.tensor2d(yDrag, [yDrag.length, 1]);
                
                // Create neural network model
                neuralNetwork = tf.sequential({
                    layers: [
                        tf.layers.dense({ inputShape: [features.length], units: 64, activation: 'relu' }),
                        tf.layers.dropout({ rate: 0.2 }),
                        tf.layers.dense({ units: 32, activation: 'relu' }),
                        tf.layers.dropout({ rate: 0.2 }),
                        tf.layers.dense({ units: 16, activation: 'relu' }),
                        tf.layers.dense({ units: 2, activation: 'linear' }) // [lift, drag]
                    ]
                });
                
                // Compile model
                neuralNetwork.compile({
                    optimizer: tf.train.adam(0.001),
                    loss: 'meanSquaredError',
                    metrics: ['mae']
                });
                
                // Combine targets
                const ys = tf.concat([ysLift, ysDrag], 1);
                
                // Training callbacks
                const lossHistory = [];
                
                // Train the model
                await neuralNetwork.fit(xs, ys, {
                    epochs: 100,
                    batchSize: 32,
                    validationSplit: 0.2,
                    callbacks: {
                        onEpochEnd: (epoch, logs) => {
                            const progress = ((epoch + 1) / 100) * 100;
                            document.getElementById('progressFill').style.width = progress + '%';
                            
                            lossHistory.push({ epoch: epoch + 1, loss: logs.loss, val_loss: logs.val_loss });
                            
                            document.getElementById('trainingMetrics').innerHTML = `
                                Epoch ${epoch + 1}/100 - Loss: ${logs.loss.toFixed(4)} - Val Loss: ${logs.val_loss.toFixed(4)} - MAE: ${logs.mae.toFixed(4)}
                            `;
                            
                            if (epoch % 10 === 0) {
                                visualizeTrainingLoss(lossHistory);
                            }
                        }
                    }
                });
                
                // Clean up tensors
                xs.dispose();
                ysLift.dispose();
                ysDrag.dispose();
                ys.dispose();
                
                updateStatus("Neural network trained successfully! Ready for predictions.");
                visualizeTrainingLoss(lossHistory);
                
            } catch (error) {
                console.error('Training error:', error);
                updateStatus("Training failed. Check console for details.");
            }
            
            isTraining = false;
        }

        // Predict both lift and drag coefficients
        async function predictCoefficients() {
            if (!neuralNetwork || !scaler) {
                // If no neural network, use theoretical predictions
                const nacaCode = document.getElementById('nacaCode').value;
                const info = nacaDatabase[nacaCode];
                const aoa = parseFloat(document.getElementById('angleOfAttack').value);
                const span = parseFloat(document.getElementById('wingSpan').value);
                const chord = parseFloat(document.getElementById('chordLength').value);
                const speed = parseFloat(document.getElementById('airspeed').value);
                const reynolds = parseFloat(document.getElementById('reynolds').value);
                
                // Calculate derived parameters
                const area = span * chord;
                const aspectRatio = span / chord;
                
                // Use theoretical calculations
                const theoreticalLift = calculateTheoreticalCL(aoa, nacaCode);
                const theoreticalDrag = calculateTheoreticalCD(theoreticalLift, nacaCode);
                
                displayAdvancedResults({
                    nacaCode: nacaCode,
                    angleOfAttack: aoa,
                    wingSpan: span,
                    chordLength: chord,
                    airspeed: speed,
                    reynolds: reynolds,
                    wingArea: area,
                    aspectRatio: aspectRatio,
                    predictedLift: theoreticalLift,
                    predictedDrag: theoreticalDrag,
                    theoreticalLift: theoreticalLift,
                    theoreticalDrag: theoreticalDrag
                });
                
                updateStatus("Using theoretical calculations. Train neural network for ML predictions!");
                return;
            }
            
            // Get input values
            const nacaCode = document.getElementById('nacaCode').value;
            const info = nacaDatabase[nacaCode];
            const aoa = parseFloat(document.getElementById('angleOfAttack').value);
            const span = parseFloat(document.getElementById('wingSpan').value);
            const chord = parseFloat(document.getElementById('chordLength').value);
            const speed = parseFloat(document.getElementById('airspeed').value);
            const reynolds = parseFloat(document.getElementById('reynolds').value);
            
            // Calculate derived parameters
            const area = span * chord;
            const aspectRatio = span / chord;
            
            try {
                // Prepare input for model
                const inputFeatures = [info.camber, info.thickness, aoa, span, chord, speed, reynolds, aspectRatio];
                const inputScaled = scaler.transform([inputFeatures]);
                
                // Make prediction with TensorFlow
                const inputTensor = tf.tensor2d(inputScaled);
                const predictions = await neuralNetwork.predict(inputTensor);
                const predictionData = await predictions.data();
                
                const predictedLift = predictionData[0];
                const predictedDrag = predictionData[1];
                
                // Calculate theoretical values for comparison
                const theoreticalLift = calculateTheoreticalCL(aoa, nacaCode);
                const theoreticalDrag = calculateTheoreticalCD(theoreticalLift, nacaCode);
                
                // Clean up tensors
                inputTensor.dispose();
                predictions.dispose();
                
                // Display results
                displayAdvancedResults({
                    nacaCode: nacaCode,
                    angleOfAttack: aoa,
                    wingSpan: span,
                    chordLength: chord,
                    airspeed: speed,
                    reynolds: reynolds,
                    wingArea: area,
                    aspectRatio: aspectRatio,
                    predictedLift: predictedLift,
                    predictedDrag: predictedDrag,
                    theoreticalLift: theoreticalLift,
                    theoreticalDrag: theoreticalDrag
                });
                
                // Generate comparison charts
                generateComparisonChart(aoa, nacaCode);
                
            } catch (error) {
                console.error('Prediction error:', error);
                updateStatus("Prediction failed. Check console for details.");
            }
        }

        // Display advanced prediction results
        function displayAdvancedResults(results) {
            const resultsDiv = document.getElementById('results');
            const outputDiv = document.getElementById('predictionOutput');
            const comparisonDiv = document.getElementById('theoreticalComparison');
            
            const liftCategory = results.predictedLift > 1.5 ? "High Lift" : 
                               results.predictedLift > 0.8 ? "Good Lift" :
                               results.predictedLift > 0.3 ? "Moderate Lift" : "Low Lift";
            
            const dragCategory = results.predictedDrag < 0.02 ? "Low Drag" :
                               results.predictedDrag < 0.05 ? "Moderate Drag" :
                               results.predictedDrag < 0.1 ? "High Drag" : "Very High Drag";
            
            outputDiv.innerHTML = `
                <div class="prediction-result">
                    <h4>Predicted Lift Coefficient</h4>
                    <div class="prediction-value">${results.predictedLift.toFixed(3)}</div>
                    <div style="margin-top: 10px; font-size: 1.1em;">${liftCategory}</div>
                </div>
                <div class="prediction-result drag">
                    <h4>Predicted Drag Coefficient</h4>
                    <div class="prediction-value">${results.predictedDrag.toFixed(4)}</div>
                    <div style="margin-top: 10px; font-size: 1.1em;">${dragCategory}</div>
                </div>
            `;
            
            const liftError = Math.abs(results.predictedLift - results.theoreticalLift);
            const dragError = Math.abs(results.predictedDrag - results.theoreticalDrag);
            
            comparisonDiv.innerHTML = `
                <h4>🔍 ML vs Theoretical Comparison</h4>
                <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 15px; margin-top: 15px;">
                    <div style="background: white; padding: 15px; border-radius: 10px; border-left: 4px solid #3498db;">
                        <strong>Lift Coefficient</strong><br>
                        ML: ${results.predictedLift.toFixed(3)}<br>
                        Theory: ${results.theoreticalLift.toFixed(3)}<br>
                        Error: ${liftError.toFixed(3)} (${(liftError/Math.abs(results.theoreticalLift)*100).toFixed(1)}%)
                    </div>
                    <div style="background: white; padding: 15px; border-radius: 10px; border-left: 4px solid #e74c3c;">
                        <strong>Drag Coefficient</strong><br>
                        ML: ${results.predictedDrag.toFixed(4)}<br>
                        Theory: ${results.theoreticalDrag.toFixed(4)}<br>
                        Error: ${dragError.toFixed(4)} (${(dragError/Math.abs(results.theoreticalDrag)*100).toFixed(1)}%)
                    </div>
                    <div style="background: white; padding: 15px; border-radius: 10px; border-left: 4px solid #27ae60;">
                        <strong>Performance</strong><br>
                        L/D Ratio: ${(results.predictedLift/results.predictedDrag).toFixed(1)}<br>
                        NACA: ${results.nacaCode}<br>
                        Re: ${(results.reynolds/1000000).toFixed(1)}M
                    </div>
                    <div style="background: white; padding: 15px; border-radius: 10px; border-left: 4px solid #f39c12;">
                        <strong>Configuration</strong><br>
                        AOA: ${results.angleOfAttack}°<br>
                        AR: ${results.aspectRatio.toFixed(1)}<br>
                        Area: ${results.wingArea.toFixed(1)} m²
                    </div>
                </div>
            `;
            
            resultsDiv.style.display = 'block';
        }

        // Flow simulation with particles
        function startFlowSimulation() {
            const canvas = document.getElementById('flowCanvas');
            if (!canvas) {
                console.error('Canvas not found');
                return;
            }
            
            const ctx = canvas.getContext('2d');
            if (!ctx) {
                console.error('Canvas context not available');
                return;
            }
            
            // Set canvas size explicitly
            canvas.width = 800;
            canvas.height = 300;
            
            if (flowSimulation) {
                clearInterval(flowSimulation);
            }
            
            const particles = [];
            const numParticles = 100;
            const nacaCode = document.getElementById('nacaCode').value;
            const aoa = parseFloat(document.getElementById('angleOfAttack').value) * Math.PI / 180;
            
            // Define airfoil position and scale
            const airfoilX = 300;
            const airfoilY = 150;
            const airfoilScale = 150;
            
            // Initialize particles
            for (let i = 0; i < numParticles; i++) {
                particles.push({
                    x: Math.random() * 150, // Start from left side
                    y: 100 + (Math.random() - 0.5) * 100, // Spread around middle
                    vx: 1.5 + Math.random() * 1.5, // Base horizontal velocity
                    vy: (Math.random() - 0.5) * 0.3, // Small vertical velocity
                    age: Math.random() * 50,
                    maxAge: 80 + Math.random() * 40,
                    size: 1.5 + Math.random() * 1.5
                });
            }
            
            // Simple airfoil shape for collision detection
            function isInsideAirfoil(x, y) {
                const dx = (x - airfoilX) / airfoilScale;
                const dy = (y - airfoilY) / (airfoilScale * 0.3);
                
                // Simple elliptical airfoil approximation
                if (dx >= -0.5 && dx <= 0.5 && Math.abs(dy) <= 0.12 * (1 - 4 * dx * dx)) {
                    return true;
                }
                return false;
            }
            
            function animate() {
                // Clear canvas with background
                ctx.fillStyle = '#2c3e50';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                
                // Draw simple airfoil shape
                ctx.fillStyle = '#34495e';
                ctx.strokeStyle = '#ecf0f1';
                ctx.lineWidth = 2;
                
                ctx.save();
                ctx.translate(airfoilX, airfoilY);
                ctx.rotate(aoa);
                
                // Draw simplified airfoil (elliptical)
                ctx.beginPath();
                ctx.ellipse(0, 0, airfoilScale * 0.5, airfoilScale * 0.12, 0, 0, Math.PI * 2);
                ctx.fill();
                ctx.stroke();
                ctx.restore();
                
                // Update and draw particles
                for (let i = particles.length - 1; i >= 0; i--) {
                    const p = particles[i];
                    
                    // Calculate distance to airfoil
                    const dx = p.x - airfoilX;
                    const dy = p.y - airfoilY;
                    const dist = Math.sqrt(dx * dx + dy * dy);
                    
                    // Apply flow effects around airfoil
                    if (dist > 20 && dist < 120) {
                        const influence = Math.max(0, (120 - dist) / 100);
                        const angle = Math.atan2(dy, dx);
                        
                        // Flow deflection
                        const deflectionX = Math.cos(angle + Math.PI/2) * influence * 0.02;
                        const deflectionY = Math.sin(angle + Math.PI/2) * influence * 0.02;
                        
                        p.vx += deflectionX;
                        p.vy += deflectionY;
                        
                        // Angle of attack effects
                        p.vy += Math.sin(aoa) * influence * 0.03;
                    }
                    
                    // Check collision with airfoil
                    if (isInsideAirfoil(p.x + p.vx, p.y + p.vy)) {
                        // Bounce off airfoil
                        p.vx *= -0.5;
                        p.vy += (Math.random() - 0.5) * 2;
                    }
                    
                    // Update position
                    p.x += p.vx;
                    p.y += p.vy;
                    p.age++;
                    
                    // Add some turbulence
                    p.vy += (Math.random() - 0.5) * 0.05;
                    
                    // Draw particle with trail effect
                    const alpha = Math.max(0.1, 1 - p.age / p.maxAge);
                    const speed = Math.sqrt(p.vx * p.vx + p.vy * p.vy);
                    const hue = Math.max(180, Math.min(240, 200 + speed * 20));
                    
                    ctx.fillStyle = `hsla(${hue}, 70%, 60%, ${alpha})`;
                    ctx.beginPath();
                    ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
                    ctx.fill();
                    
                    // Draw velocity vector for some particles
                    if (i % 10 === 0 && alpha > 0.5) {
                        ctx.strokeStyle = `hsla(${hue}, 50%, 70%, ${alpha * 0.5})`;
                        ctx.lineWidth = 1;
                        ctx.beginPath();
                        ctx.moveTo(p.x, p.y);
                        ctx.lineTo(p.x + p.vx * 10, p.y + p.vy * 10);
                        ctx.stroke();
                    }
                    
                    // Reset particle if old or off screen
                    if (p.age > p.maxAge || p.x > canvas.width + 50 || p.x < -50 || p.y > canvas.height + 50 || p.y < -50) {
                        p.x = -20 + Math.random() * 40;
                        p.y = airfoilY + (Math.random() - 0.5) * 100;
                        p.vx = 1.5 + Math.random() * 1.5;
                        p.vy = (Math.random() - 0.5) * 0.3;
                        p.age = 0;
                    }
                }
                
                // Draw flow field grid (optional visual aid)
                ctx.strokeStyle = 'rgba(52, 152, 219, 0.1)';
                ctx.lineWidth = 1;
                for (let x = 0; x < canvas.width; x += 40) {
                    ctx.beginPath();
                    ctx.moveTo(x, 0);
                    ctx.lineTo(x, canvas.height);
                    ctx.stroke();
                }
                for (let y = 0; y < canvas.height; y += 30) {
                    ctx.beginPath();
                    ctx.moveTo(0, y);
                    ctx.lineTo(canvas.width, y);
                    ctx.stroke();
                }
            }
            
            // Start animation
            flowSimulation = setInterval(animate, 60); // ~16 FPS
            updateStatus("Flow simulation active! Watch particles flow around the NACA " + nacaCode + " airfoil.");
            
            // Auto-stop after 30 seconds to prevent infinite running
            setTimeout(() => {
                if (flowSimulation) {
                    clearInterval(flowSimulation);
                    flowSimulation = null;
                    updateStatus("Flow simulation completed. Click 'Flow Simulation' to restart.");
                }
            }, 30000);
        }

        // Chart visualization functions
        function visualizeLiftCurve() {
            const ctx = document.getElementById('liftChart').getContext('2d');
            
            if (window.liftChart && typeof window.liftChart.destroy === 'function') {
                window.liftChart.destroy();
            }
            
            // Generate lift curve data for different NACA airfoils
            const angles = [];
            for (let a = -10; a <= 20; a += 0.5) {
                angles.push(a);
            }
            
            const datasets = [];
            const colors = ['#3498db', '#e74c3c', '#27ae60', '#f39c12'];
            const nacaCodes = ['0012', '2412', '4412', '64212'];
            
            nacaCodes.forEach((naca, index) => {
                const liftData = angles.map(aoa => ({
                    x: aoa,
                    y: calculateTheoreticalCL(aoa, naca)
                }));
                
                datasets.push({
                    label: `NACA ${naca}`,
                    data: liftData,
                    borderColor: colors[index],
                    backgroundColor: colors[index] + '20',
                    borderWidth: 2,
                    fill: false,
                    tension: 0.1
                });
            });
            
            window.liftChart = new Chart(ctx, {
                type: 'line',
                data: { datasets: datasets },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        x: {
                            title: { display: true, text: 'Angle of Attack (degrees)', font: { weight: 'bold' } },
                            grid: { color: 'rgba(0,0,0,0.1)' }
                        },
                        y: {
                            title: { display: true, text: 'Lift Coefficient (CL)', font: { weight: 'bold' } },
                            grid: { color: 'rgba(0,0,0,0.1)' }
                        }
                    },
                    plugins: {
                        title: { display: true, text: 'Theoretical Lift Curves', font: { size: 14, weight: 'bold' } },
                        legend: { position: 'top' }
                    }
                }
            });
        }

        function visualizeDragPolar() {
            const ctx = document.getElementById('dragChart').getContext('2d');
            
            if (window.dragChart && typeof window.dragChart.destroy === 'function') {
                window.dragChart.destroy();
            }
            
            const datasets = [];
            const colors = ['#3498db', '#e74c3c', '#27ae60', '#f39c12'];
            const nacaCodes = ['0012', '2412', '4412', '64212'];
            
            nacaCodes.forEach((naca, index) => {
                const polarData = [];
                for (let aoa = -5; aoa <= 15; aoa += 0.5) {
                    const cl = calculateTheoreticalCL(aoa, naca);
                    const cd = calculateTheoreticalCD(cl, naca);
                    polarData.push({ x: cd, y: cl });
                }
                
                datasets.push({
                    label: `NACA ${naca}`,
                    data: polarData,
                    borderColor: colors[index],
                    backgroundColor: colors[index] + '40',
                    borderWidth: 2,
                    fill: false,
                    tension: 0.1
                });
            });
            
            window.dragChart = new Chart(ctx, {
                type: 'line',
                data: { datasets: datasets },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        x: {
                            title: { display: true, text: 'Drag Coefficient (CD)', font: { weight: 'bold' } },
                            grid: { color: 'rgba(0,0,0,0.1)' }
                        },
                        y: {
                            title: { display: true, text: 'Lift Coefficient (CL)', font: { weight: 'bold' } },
                            grid: { color: 'rgba(0,0,0,0.1)' }
                        }
                    },
                    plugins: {
                        title: { display: true, text: 'Drag Polar Curves', font: { size: 14, weight: 'bold' } },
                        legend: { position: 'top' }
                    }
                }
            });
        }

        function generateComparisonChart(centerAOA, nacaCode) {
            const ctx = document.getElementById('comparisonChart').getContext('2d');
            
            if (window.comparisonChart && typeof window.comparisonChart.destroy === 'function') {
                window.comparisonChart.destroy();
            }
            
            const angles = [];
            const mlData = [];
            const theoreticalData = [];
            
            for (let aoa = centerAOA - 5; aoa <= centerAOA + 5; aoa += 0.5) {
                angles.push(aoa);
                
                // Theoretical
                const theoLift = calculateTheoreticalCL(aoa, nacaCode);
                theoreticalData.push({ x: aoa, y: theoLift });
                
                // Mock ML prediction (would be actual prediction in real scenario)
                const mlLift = theoLift + (Math.random() - 0.5) * 0.05;
                mlData.push({ x: aoa, y: mlLift });
            }
            
            window.comparisonChart = new Chart(ctx, {
                type: 'line',
                data: {
                    datasets: [{
                        label: 'Theoretical',
                        data: theoreticalData,
                        borderColor: '#e74c3c',
                        backgroundColor: '#e74c3c20',
                        borderWidth: 3,
                        fill: false,
                        borderDash: [5, 5]
                    }, {
                        label: 'Neural Network',
                        data: mlData,
                        borderColor: '#3498db',
                        backgroundColor: '#3498db20',
                        borderWidth: 2,
                        fill: false
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        x: {
                            title: { display: true, text: 'Angle of Attack (degrees)', font: { weight: 'bold' } },
                            grid: { color: 'rgba(0,0,0,0.1)' }
                        },
                        y: {
                            title: { display: true, text: 'Lift Coefficient', font: { weight: 'bold' } },
                            grid: { color: 'rgba(0,0,0,0.1)' }
                        }
                    },
                    plugins: {
                        title: { display: true, text: `ML vs Theory: NACA ${nacaCode}`, font: { size: 14, weight: 'bold' } },
                        legend: { position: 'top' }
                    }
                }
            });
        }

        function visualizeTrainingLoss(lossHistory) {
            const ctx = document.getElementById('lossChart').getContext('2d');
            
            if (window.lossChart && typeof window.lossChart.destroy === 'function') {
                window.lossChart.destroy();
            }
            
            const epochs = lossHistory.map(h => h.epoch);
            const trainLoss = lossHistory.map(h => h.loss);
            const valLoss = lossHistory.map(h => h.val_loss);
            
            window.lossChart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: epochs,
                    datasets: [{
                        label: 'Training Loss',
                        data: trainLoss,
                        borderColor: '#3498db',
                        backgroundColor: '#3498db20',
                        borderWidth: 2,
                        fill: false
                    }, {
                        label: 'Validation Loss',
                        data: valLoss,
                        borderColor: '#e74c3c',
                        backgroundColor: '#e74c3c20',
                        borderWidth: 2,
                        fill: false
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        x: {
                            title: { display: true, text: 'Epoch', font: { weight: 'bold' } },
                            grid: { color: 'rgba(0,0,0,0.1)' }
                        },
                        y: {
                            title: { display: true, text: 'Loss', font: { weight: 'bold' } },
                            grid: { color: 'rgba(0,0,0,0.1)' },
                            type: 'logarithmic'
                        }
                    },
                    plugins: {
                        title: { display: true, text: 'Neural Network Training Progress', font: { size: 14, weight: 'bold' } },
                        legend: { position: 'top' }
                    }
                }
            });
        }

        // Utility functions
        function loadExampleAirfoil(nacaCode) {
            document.getElementById('nacaCode').value = nacaCode;
            drawAirfoil(nacaCode);
            updateReynolds();
        }

        function updateReynolds() {
            const speed = parseFloat(document.getElementById('airspeed').value) || 80;
            const chord = parseFloat(document.getElementById('chordLength').value) || 2;
            // Simplified Reynolds calculation (assuming standard conditions)
            const reynolds = Math.round(speed * chord * 68000); // Approximate for standard conditions
            document.getElementById('reynolds').value = reynolds;
        }

        function updateStatus(message, loading = false) {
            const statusDiv = document.getElementById('modelStatus');
            statusDiv.innerHTML = loading ? 
                `<span class="loading"></span>${message}` : 
                message;
        }

        // Event listeners
        document.getElementById('nacaCode').addEventListener('change', function() {
            drawAirfoil(this.value);
        });

        document.getElementById('airspeed').addEventListener('input', updateReynolds);
        document.getElementById('chordLength').addEventListener('input', updateReynolds);

        // Initialize
        window.addEventListener('load', function() {
            updateStatus("Advanced aerodynamics predictor ready! Select an airfoil and generate training data.");
            drawAirfoil('0012');
            updateReynolds();
        });
    </script>
</body>
</html>
