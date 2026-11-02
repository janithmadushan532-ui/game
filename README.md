<!DOCTYPE html>
<html lang="si">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Car Science Racer - OL Science 3D Game</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+Sinhala:wght@400;600;700&display=swap');
  
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    overflow: hidden;
    background: #0a0a1a;
    font-family: 'Noto Sans Sinhala', 'Segoe UI', sans-serif;
    color: #fff;
    user-select: none;
    -webkit-user-select: none;
    touch-action: none;
  }
  #gameCanvas { display: block; width: 100vw; height: 100vh; }
  
  #ui {
    position: absolute;
    top: 0; left: 0; width: 100%; height: 100%;
    pointer-events: none;
    z-index: 10;
  }
  #ui > * { pointer-events: auto; }
  
  .hud {
    position: absolute;
    top: 12px; left: 12px; right: 12px;
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    gap: 10px;
  }
  .hud-box {
    background: rgba(0,0,0,0.6);
    backdrop-filter: blur(8px);
    border: 1px solid rgba(255,180,50,0.45);
    border-radius: 12px;
    padding: 8px 14px;
    font-size: 15px;
    font-weight: 600;
    box-shadow: 0 4px 15px rgba(0,0,0,0.4);
  }
  .score { color: #ffd700; }
  .lives { color: #ff6b6b; }
  .power { color: #4ade80; display: none; }
  .power.active { display: block; animation: pulse 0.6s infinite alternate; }
  
  @keyframes pulse {
    from { transform: scale(1); }
    to { transform: scale(1.06); }
  }
  
  .screen {
    position: absolute;
    inset: 0;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background: linear-gradient(160deg, #0d0221 0%, #1a0a3e 30%, #16213e 60%, #0a1628 100%);
    z-index: 20;
    text-align: center;
    padding: 20px;
  }
  .screen::before {
    content: '';
    position: absolute;
    inset: 0;
    background: radial-gradient(ellipse at 50% 0%, rgba(255,154,60,0.15) 0%, transparent 55%),
                radial-gradient(ellipse at 80% 80%, rgba(100,80,255,0.12) 0%, transparent 40%);
    pointer-events: none;
  }
  .screen.hidden { display: none !important; }
  
  .title {
    font-size: clamp(1.8rem, 6.5vw, 3rem);
    font-weight: 700;
    background: linear-gradient(90deg, #ff9a3c, #ffd700, #ff6b6b, #ff9a3c);
    background-size: 200% auto;
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    margin-bottom: 8px;
    animation: shineTitle 3s linear infinite;
    filter: drop-shadow(0 0 18px rgba(255,180,50,0.4));
  }
  @keyframes shineTitle {
    to { background-position: 200% center; }
  }
  .subtitle {
    font-size: 1.05rem;
    color: #a0c4ff;
    margin-bottom: 24px;
    max-width: 400px;
    line-height: 1.5;
  }
  
  .btn {
    background: linear-gradient(135deg, #ff9a3c, #ff6b35);
    border: none;
    color: #fff;
    font-family: inherit;
    font-size: 1.15rem;
    font-weight: 700;
    padding: 14px 40px;
    border-radius: 50px;
    cursor: pointer;
    box-shadow: 0 6px 20px rgba(255,107,53,0.5);
    transition: transform 0.15s, box-shadow 0.15s;
    margin: 8px;
  }
  .btn:hover { transform: translateY(-2px); box-shadow: 0 8px 25px rgba(255,107,53,0.7); }
  .btn:active { transform: translateY(1px); }
  
  #questionModal {
    position: absolute;
    inset: 0;
    background: rgba(0,0,0,0.78);
    display: none;
    align-items: center;
    justify-content: center;
    z-index: 30;
    padding: 16px;
  }
  #questionModal.show { display: flex; }
  
  .q-card {
    background: linear-gradient(145deg, #1a1535, #2a1a4a 50%, #1e1e3f);
    border: 2px solid #ff9a3c;
    border-radius: 20px;
    padding: 22px 18px;
    max-width: 500px;
    width: 100%;
    box-shadow: 0 20px 50px rgba(0,0,0,0.65), 0 0 50px rgba(255,154,60,0.25), inset 0 1px 0 rgba(255,255,255,0.08);
    animation: popIn 0.28s ease;
  }
  @keyframes popIn {
    from { transform: scale(0.82); opacity: 0; }
    to { transform: scale(1); opacity: 1; }
  }
  
  .q-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 14px;
  }
  .q-badge {
    background: #ff9a3c;
    color: #1a0a2e;
    font-weight: 700;
    padding: 4px 12px;
    border-radius: 20px;
    font-size: 0.85rem;
  }
  .q-timer {
    font-size: 1.25rem;
    font-weight: 700;
    color: #4ade80;
  }
  .q-timer.urgent { color: #ff6b6b; animation: blink 0.4s infinite; }
  @keyframes blink { 50% { opacity: 0.35; } }
  
  .q-text {
    font-size: 1.1rem;
    line-height: 1.55;
    margin-bottom: 18px;
    color: #f0f0ff;
  }
  
  .options { display: grid; gap: 9px; }
  .opt {
    background: rgba(255,255,255,0.08);
    border: 2px solid rgba(255,255,255,0.15);
    border-radius: 12px;
    padding: 13px 14px;
    font-size: 0.98rem;
    color: #fff;
    cursor: pointer;
    text-align: left;
    transition: all 0.15s;
    font-family: inherit;
  }
  .opt:hover { background: rgba(255,154,60,0.25); border-color: #ff9a3c; }
  .opt.correct { background: rgba(74,222,128,0.3); border-color: #4ade80; }
  .opt.wrong { background: rgba(255,107,107,0.3); border-color: #ff6b6b; }
  .opt:disabled { pointer-events: none; }
  
  .feedback {
    margin-top: 14px;
    font-size: 1.05rem;
    font-weight: 600;
    text-align: center;
    min-height: 26px;
  }
  .feedback.good { color: #4ade80; }
  .feedback.bad { color: #ff6b6b; }
  
  #mobileControls {
    position: absolute;
    bottom: 18px;
    left: 0; right: 0;
    display: none;
    justify-content: space-between;
    padding: 0 18px;
    z-index: 15;
    pointer-events: none;
  }
  .m-btn {
    width: 68px; height: 68px;
    border-radius: 50%;
    background: rgba(255,255,255,0.18);
    border: 2px solid rgba(255,255,255,0.35);
    color: #fff;
    font-size: 1.7rem;
    display: flex;
    align-items: center;
    justify-content: center;
    pointer-events: auto;
    touch-action: manipulation;
    -webkit-tap-highlight-color: transparent;
  }
  .m-btn:active { background: rgba(255,154,60,0.55); }
  
  .howto {
    background: rgba(0,0,0,0.4);
    border-radius: 14px;
    padding: 14px 18px;
    margin: 12px 0 20px;
    text-align: left;
    max-width: 360px;
    font-size: 0.92rem;
    line-height: 1.55;
  }
  .howto li { margin: 5px 0; }
  
  #powerNotify {
    position: absolute;
    top: 28%;
    left: 50%;
    transform: translateX(-50%);
    background: linear-gradient(90deg, #4ade80, #22c55e);
    color: #052e16;
    font-weight: 700;
    font-size: 1.25rem;
    padding: 11px 26px;
    border-radius: 50px;
    box-shadow: 0 0 28px rgba(74,222,128,0.6);
    opacity: 0;
    transition: opacity 0.3s;
    z-index: 25;
    pointer-events: none;
  }
  #powerNotify.show { opacity: 1; }
  
  .final-score {
    font-size: 2.1rem;
    color: #ffd700;
    margin: 10px 0 18px;
  }
  
  #loading {
    position: absolute;
    inset: 0;
    background: #0a0a1a;
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 50;
    font-size: 1.3rem;
    color: #ffd700;
  }
</style>
</head>
<body>
<div id="loading">Loading 3D Car Game... 🚗</div>
<canvas id="gameCanvas"></canvas>

<div id="ui">
  <div class="hud">
    <div class="hud-box score">⭐ Score: <span id="scoreVal">0</span></div>
    <div class="hud-box lives">❤️ <span id="livesVal">3</span></div>
    <div class="hud-box power" id="powerBox">⚡ Power!</div>
  </div>
  
  <div id="powerNotify">⚡ POWER UP! ⚡</div>
  
  <!-- Swipe area - full screen touch -->
  <div id="swipeArea" style="position:absolute;inset:0;z-index:5;"></div>
</div>

<div id="startScreen" class="screen">
  <div class="title">🚗 Car Science Racer</div>
  <div class="subtitle">OL Science Question Ball වලට වැදී හරි උත්තර දී Invisible බලය ලබාගන්න!</div>
  <ul class="howto">
    <li>🚗 ඔබේ Car ඉස්සරහට යයි</li>
    <li>👈👉 <b>Swipe</b> කරන්න (වම් / දකුණු lane)</li>
    <li>⌨️ Keyboard: A / D හෝ ← →</li>
    <li>🚙 අනෙක් cars වලට හැප්පුණොත් → Life අඩු / Out</li>
    <li>💫 Shine Question Ball එකකට වැදුණොත් Science ප්‍රශ්නයක්</li>
    <li>✅ හරි උත්තරයට <b>Invisible බලය</b> (cars ඔබව දකින්නේ නැහැ)</li>
  </ul>
  <button class="btn" id="startBtn">🎮 ආරම්භ කරන්න</button>
</div>

<div id="gameOverScreen" class="screen hidden">
  <div class="title">Game Over!</div>
  <div class="final-score">Score: <span id="finalScore">0</span></div>
  <div class="subtitle" id="gameOverMsg">නැවත උත්සාහ කරන්න!</div>
  <button class="btn" id="restartBtn">🔄 නැවත ක්‍රීඩා කරන්න</button>
</div>

<div id="questionModal">
  <div class="q-card">
    <div class="q-header">
      <span class="q-badge">OL Science</span>
      <span class="q-timer" id="qTimer">15</span>
    </div>
    <div class="q-text" id="qText"></div>
    <div class="options" id="qOptions"></div>
    <div class="feedback" id="qFeedback"></div>
  </div>
</div>

<script src="https://unpkg.com/three@0.160.0/build/three.min.js"></script>
<script>
// ==================== QUESTIONS ====================
const QUESTIONS = [
  { q: "කොළ පැහැති ශාකවල ෆොටෝසින්තේසිස් සඳහා ප්‍රධාන වර්ණකය කුමක්ද?", options: ["ක්ලෝරොෆිල්", "කැරොටීනොයිඩ්", "ඇන්තොසයනින්", "ක්සැන්තොෆිල්"], correct: 0 },
  { q: "සෛලයේ ATP නිෂ්පාදනය කරන ප්‍රධාන ඉන්ද්‍රිය කුමක්ද?", options: ["මයිටොකොන්ඩ්‍රියා", "ක්ලෝරොප්ලාස්ට්", "ගොල්ගි ශරීරය", "රයිබොසෝම"], correct: 0 },
  { q: "ජලයේ රසායනික සූත්‍රය කුමක්ද?", options: ["H₂O", "CO₂", "O₂", "H₂O₂"], correct: 0 },
  { q: "මිනිස් ශරීරයේ රුධිරයේ ඔක්සිජන් ප්‍රවාහනය කරන ප්‍රෝටීන් කුමක්ද?", options: ["හීමොග්ලොබින්", "ඉන්සියුලින්", "ඇඩ්‍රිනලින්", "කොලෙස්ටරෝල්"], correct: 0 },
  { q: "පෘථිවියේ වායුගෝලයේ වැඩිම ප්‍රතිශතයක් ඇති වායුව කුමක්ද?", options: ["නයිට්‍රජන්", "ඔක්සිජන්", "කාබන් ඩයොක්සයිඩ්", "ආගන්"], correct: 0 },
  { q: "ශාකවල ජලය හා ඛනිජ ලවණ ප්‍රවාහනය කරන පටකය කුමක්ද?", options: ["සයිලම්", "ෆ්ලෝඑම්", "කොලෙන්කයිමා", "පැරෙන්කයිමා"], correct: 0 },
  { q: "ආලෝකයේ වේගය ආසන්න වශයෙන් කුමක්ද?", options: ["3 × 10⁸ m/s", "3 × 10⁶ m/s", "3 × 10⁵ km/s", "300 m/s"], correct: 0 },
  { q: "අම්ලයක pH අගය සාමාන්‍යයෙන් කුමක්ද?", options: ["7 ට අඩු", "7 ට වැඩි", "හරියටම 7", "14"], correct: 0 },
  { q: "මිනිස් ඇසේ ආලෝකය හඳුනාගන්නා කොටස කුමක්ද?", options: ["රෙටිනා", "කෝනියා", "අයිරිස්", "ලෙන්ස්"], correct: 0 },
  { q: "ඡායාරූප සංශ්ලේෂණයේදී ශාක මුදාහරින වායුව කුමක්ද?", options: ["ඔක්සිජන්", "කාබන් ඩයොක්සයිඩ්", "නයිට්‍රජන්", "හයිඩ්‍රජන්"], correct: 0 },
  { q: "විද්‍යුත් පරිපථයක ධාරාව මනින ඒකකය කුමක්ද?", options: ["ඇම්පියර්", "වෝල්ට්", "ඕම්", "වොට්"], correct: 0 },
  { q: "මිනිස් ශරීරයේ විශාලතම අවයවය කුමක්ද?", options: ["සම", "අක්මාව", "මොළය", "පෙනහළු"], correct: 0 },
  { q: "DNA හි ප්‍රධාන කාර්යභාරය කුමක්ද?", options: ["ජාන තොරතුරු ගබඩා කිරීම", "ශක්ති නිෂ්පාදනය", "ප්‍රෝටීන් සංශ්ලේෂණය", "කොළපැහැති වීම"], correct: 0 },
  { q: "ගුරුත්වාකර්ෂණ නියමය සොයාගත්තේ කවුද?", options: ["අයිසැක් නිව්ටන්", "ඇල්බට් අයින්ස්ටයින්", "ගැලීලියෝ", "මැක්ස්වෙල්"], correct: 0 },
  { q: "පෘථිවියේ ස්වාභාවික චුම්බක ක්ෂේත්‍රයේ උතුරු ධ්‍රැවයට ආකර්ෂණය වන්නේ මාග්නට් එකේ කුමන කෙළවරද?", options: ["දකුණු කෙළවර", "උතුරු කෙළවර", "දෙකම", "කිසිවක් නැත"], correct: 0 },
  { q: "ශාකවල ආහාර ප්‍රවාහනය කරන පටකය කුමක්ද?", options: ["ෆ්ලෝඑම්", "සයිලම්", "කැම්බියම්", "එපිඩර්මිස්"], correct: 0 },
  { q: "මිනිස් හෘදයේ කුටීර කීයක් තිබේද?", options: ["4", "2", "3", "5"], correct: 0 },
  { q: "ඔස්මෝසිස් යනු කුමක්ද?", options: ["ජලයේ ප්‍රසාරණය", "වායුවේ ප්‍රසාරණය", "ලුණු දියවීම", "ආලෝකයේ වර්තනය"], correct: 0 },
  { q: "පෘථිවියේ ගුරුත්වාකර්ෂණ ත්වරණය (g) ආසන්න වශයෙන් කුමක්ද?", options: ["9.8 m/s²", "10 km/s²", "3 × 10⁸ m/s", "6.67 × 10⁻¹¹"], correct: 0 },
  { q: "කාබන් ඩයොක්සයිඩ් හි රසායනික සූත්‍රය කුමක්ද?", options: ["CO₂", "CO", "C₂O", "O₂"], correct: 0 },
  { q: "මිනිස් රුධිරයේ සාමාන්‍ය pH අගය කුමක්ද?", options: ["7.4", "5.5", "8.2", "6.0"], correct: 0 },
  { q: "ශාකවල ප්‍රභාසංස්ලේෂණය සිදුවන ප්‍රධාන කොටස කුමක්ද?", options: ["ක්ලෝරොප්ලාස්ට්", "මයිටොකොන්ඩ්‍රියා", "න්‍යෂ්ටිය", "වැකියුලය"], correct: 0 },
  { q: "විද්‍යුත් ප්‍රතිරෝධය මනින ඒකකය කුමක්ද?", options: ["ඕම්", "ඇම්පියර්", "වෝල්ට්", "වොට්"], correct: 0 },
  { q: "මිනිස් ශරීරයේ රතු රුධිරාණු නිපදවන ස්ථානය කුමක්ද?", options: ["අස්ථි මජ්ජා", "අක්මාව", "වකුගඩු", "පෙනහළු"], correct: 0 },
  { q: "නිව්ටන්ගේ තුන්වන නියමය කුමක් පවසනවාද?", options: ["ක්‍රියාවට සමාන ප්‍රතික්‍රියාවක්", "බලය = ස්කන්ධය × ත්වරණය", "ජඩත්වය", "ගුරුත්වාකර්ෂණය"], correct: 0 },
  { q: "ජලයේ තාපාංකය (සාමාන්‍ය පීඩනයේදී) කුමක්ද?", options: ["100°C", "0°C", "50°C", "212°C"], correct: 0 },
  { q: "මිනිස් ඇසේ කාචයේ (lens) කාර්යභාරය කුමක්ද?", options: ["ආලෝකය අවධානය කිරීම", "වර්ණ හඳුනාගැනීම", "ආලෝකය අවහිර කිරීම", "කන්නය ආරක්ෂා කිරීම"], correct: 0 },
  { q: "පෘථිවියේ වායුගෝලයේ ඔක්සිජන් ප්‍රතිශතය ආසන්න වශයෙන් කුමක්ද?", options: ["21%", "78%", "0.03%", "50%"], correct: 0 },
  { q: "සෛලයේ ජාන තොරතුරු ඇති ස්ථානය කුමක්ද?", options: ["න්‍යෂ්ටිය", "සයිටොප්ලාස්මය", "රයිබොසෝම", "ගොල්ගි"], correct: 0 },
  { q: "විද්‍යුත් ශක්තියේ ඒකකය කුමක්ද?", options: ["ජූල්", "වොට්", "ඇම්පියර්", "වෝල්ට්"], correct: 0 },
  { q: "ශාකවල ශ්වසනයේදී මුදාහරින වායුව කුමක්ද?", options: ["කාබන් ඩයොක්සයිඩ්", "ඔක්සිජන්", "නයිට්‍රජන්", "හයිඩ්‍රජන්"], correct: 0 },
  { q: "මිනිස් ශරීරයේ ඉන්සියුලින් හෝමෝනය නිපදවන අවයවය කුමක්ද?", options: ["පැන්ක්‍රියාස්", "තයිරොයිඩ්", "අධිවෘක්ක", "පිටියුටරි"], correct: 0 },
  { q: "ආලෝකයේ වර්තනය (refraction) සිදුවන්නේ කවදාද?", options: ["මාධ්‍යයක් වෙනස් වන විට", "ආලෝකය පරාවර්තනය වන විට", "ආලෝකය අවශෝෂණය වන විට", "ආලෝකය විසිරෙන විට"], correct: 0 },
  { q: "රුධිරයේ සුදු රුධිරාණුවල ප්‍රධාන කාර්යභාරය කුමක්ද?", options: ["රෝගාණුවලට එරෙහිව සටන් කිරීම", "ඔක්සිජන් ප්‍රවාහනය", "රුධිර කැටි ගැසීම", "පෝෂණය"], correct: 0 },
  { q: "පෘථිවිය සූර්යයා වටා එක් වටයක් යාමට ගතවන කාලය කුමක්ද?", options: ["365 දින", "30 දින", "24 පැය", "7 දින"], correct: 0 },
  { q: "අම්ල + භෂ්ම → ?", options: ["ලවණ + ජලය", "ඔක්සිජන් + හයිඩ්‍රජන්", "කාබන් + ජලය", "නයිට්‍රජන් + ඔක්සිජන්"], correct: 0 }
];

// ==================== GAME STATE ====================
let scene, camera, renderer, clock;
let playerCar, roadMeshes = [], otherCars = [], questionBalls = [];
let score = 0, lives = 3, speed = 18, baseSpeed = 18;
let isPlaying = false, isPaused = false, hasPower = false;
let powerTimer = 0, distance = 0;
let lane = 0;
let targetX = 0;
let keys = {};
let currentQuestion = null;
let qTimeLeft = 15;
let qInterval = null;
let usedQuestions = new Set();
let spawnTimer = 0;
let invincibleTimer = 0; // after hit, short shield so not instant out
let carColors = [0xe74c3c, 0x3498db, 0x2ecc71, 0x9b59b6, 0xf39c12, 0x1abc9c, 0xe67e22];

// Swipe
let touchStartX = 0;
let touchStartY = 0;
let touchStartTime = 0;

const LANE_WIDTH = 3.5;
const ROAD_WIDTH = 12;

// ==================== INIT ====================
function init() {
  if (typeof THREE === 'undefined') {
    document.getElementById('loading').textContent = 'Three.js load වුණේ නැහැ. Internet තියෙනවාද check කරන්න.';
    return;
  }

  const canvas = document.getElementById('gameCanvas');
  renderer = new THREE.WebGLRenderer({ canvas: canvas, antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;
  renderer.toneMapping = THREE.ACESFilmicToneMapping;
  renderer.toneMappingExposure = 1.05;

  scene = new THREE.Scene();
  scene.background = new THREE.Color(0x0b1020);
  scene.fog = new THREE.Fog(0x0b1020, 40, 120);

  camera = new THREE.PerspectiveCamera(65, window.innerWidth / window.innerHeight, 0.1, 250);
  camera.position.set(0, 6.5, 11);

  const ambient = new THREE.AmbientLight(0x404060, 0.55);
  scene.add(ambient);

  const sun = new THREE.DirectionalLight(0xfff0d0, 1.15);
  sun.position.set(8, 25, 12);
  sun.castShadow = true;
  sun.shadow.mapSize.set(1024, 1024);
  sun.shadow.camera.near = 2;
  sun.shadow.camera.far = 80;
  sun.shadow.camera.left = -25;
  sun.shadow.camera.right = 25;
  sun.shadow.camera.top = 25;
  sun.shadow.camera.bottom = -25;
  scene.add(sun);

  const hemi = new THREE.HemisphereLight(0x6688cc, 0x332211, 0.35);
  scene.add(hemi);

  createStars();
  createRoad();
  createPlayerCar();

  clock = new THREE.Clock();

  window.addEventListener('resize', onResize);
  window.addEventListener('keydown', function(e) {
    keys[e.code] = true;
    if (e.code === 'Space') e.preventDefault();
  });
  window.addEventListener('keyup', function(e) {
    keys[e.code] = false;
  });

  setupMobileControls();

  document.getElementById('startBtn').onclick = startGame;
  document.getElementById('restartBtn').onclick = startGame;

  document.getElementById('loading').style.display = 'none';
  animate();
}

function createStars() {
  const geo = new THREE.BufferGeometry();
  const positions = [];
  for (let i = 0; i < 400; i++) {
    positions.push(
      (Math.random() - 0.5) * 250,
      Math.random() * 60 + 15,
      (Math.random() - 0.5) * 250 - 30
    );
  }
  geo.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3));
  const mat = new THREE.PointsMaterial({ color: 0xffffff, size: 0.13, transparent: true, opacity: 0.75 });
  scene.add(new THREE.Points(geo, mat));
}

function createRoad() {
  const roadGeo = new THREE.BoxGeometry(ROAD_WIDTH, 0.3, 300);
  const roadMat = new THREE.MeshStandardMaterial({ color: 0x151525, roughness: 0.92 });
  const road = new THREE.Mesh(roadGeo, roadMat);
  road.position.set(0, -0.15, -120);
  road.receiveShadow = true;
  scene.add(road);
  roadMeshes.push(road);

  // Neon side rails
  const railGeo = new THREE.BoxGeometry(0.18, 0.35, 300);
  const railMatL = new THREE.MeshStandardMaterial({ color: 0x00e5ff, emissive: 0x00aacc, emissiveIntensity: 0.7 });
  const railMatR = new THREE.MeshStandardMaterial({ color: 0xff6b9d, emissive: 0xcc3366, emissiveIntensity: 0.7 });
  const railL = new THREE.Mesh(railGeo, railMatL);
  railL.position.set(-6.1, 0.15, -120);
  scene.add(railL);
  roadMeshes.push(railL);
  const railR = new THREE.Mesh(railGeo, railMatR);
  railR.position.set(6.1, 0.15, -120);
  scene.add(railR);
  roadMeshes.push(railR);

  const shoulderGeo = new THREE.BoxGeometry(1.0, 0.35, 300);
  const shoulderMat = new THREE.MeshStandardMaterial({ color: 0x2a2a3a, roughness: 0.85 });
  [-6.7, 6.7].forEach(function(x) {
    const s = new THREE.Mesh(shoulderGeo, shoulderMat);
    s.position.set(x, 0.05, -120);
    s.receiveShadow = true;
    scene.add(s);
    roadMeshes.push(s);
  });

  for (let z = 20; z > -280; z -= 6) {
    const line = new THREE.Mesh(
      new THREE.BoxGeometry(0.16, 0.06, 2.6),
      new THREE.MeshBasicMaterial({ color: 0xffdd44 })
    );
    line.position.set(0, 0.02, z);
    scene.add(line);
    roadMeshes.push(line);

    [-5.1, 5.1].forEach(function(x) {
      const wl = new THREE.Mesh(
        new THREE.BoxGeometry(0.1, 0.05, 3.5),
        new THREE.MeshBasicMaterial({ color: 0xeeeeff })
      );
      wl.position.set(x, 0.02, z);
      scene.add(wl);
      roadMeshes.push(wl);
    });
  }
}

function createCar(color, isPlayer) {
  const group = new THREE.Group();

  const body = new THREE.Mesh(
    new THREE.BoxGeometry(1.6, 0.55, 3.2),
    new THREE.MeshStandardMaterial({ color: color, roughness: 0.35, metalness: 0.4 })
  );
  body.position.y = 0.55;
  body.castShadow = true;
  group.add(body);

  const cabin = new THREE.Mesh(
    new THREE.BoxGeometry(1.35, 0.5, 1.6),
    new THREE.MeshStandardMaterial({ color: isPlayer ? 0x222233 : 0x1a1a28, roughness: 0.4, metalness: 0.3 })
  );
  cabin.position.set(0, 1.0, -0.15);
  cabin.castShadow = true;
  group.add(cabin);

  const winMat = new THREE.MeshStandardMaterial({ color: 0x88ccff, transparent: true, opacity: 0.55, metalness: 0.6 });
  const frontWin = new THREE.Mesh(new THREE.BoxGeometry(1.2, 0.35, 0.08), winMat);
  frontWin.position.set(0, 1.0, 0.7);
  group.add(frontWin);

  const wheelGeo = new THREE.CylinderGeometry(0.32, 0.32, 0.28, 12);
  const wheelMat = new THREE.MeshStandardMaterial({ color: 0x111111, roughness: 0.7 });
  const wheelPositions = [
    [-0.85, 0.32, 1.05], [0.85, 0.32, 1.05],
    [-0.85, 0.32, -1.05], [0.85, 0.32, -1.05]
  ];
  wheelPositions.forEach(function(p) {
    const w = new THREE.Mesh(wheelGeo, wheelMat);
    w.rotation.z = Math.PI / 2;
    w.position.set(p[0], p[1], p[2]);
    w.castShadow = true;
    group.add(w);
  });

  const lightMat = new THREE.MeshBasicMaterial({ color: 0xffffaa });
  [-0.5, 0.5].forEach(function(x) {
    const hl = new THREE.Mesh(new THREE.BoxGeometry(0.25, 0.15, 0.1), lightMat);
    hl.position.set(x, 0.55, 1.6);
    group.add(hl);
  });

  const tailMat = new THREE.MeshBasicMaterial({ color: 0xff2222 });
  [-0.5, 0.5].forEach(function(x) {
    const tl = new THREE.Mesh(new THREE.BoxGeometry(0.22, 0.12, 0.08), tailMat);
    tl.position.set(x, 0.55, -1.6);
    group.add(tl);
  });

  if (isPlayer) {
    // Headlight beams
    const headLight = new THREE.PointLight(0xfff0c0, 1.4, 18);
    headLight.position.set(0, 0.7, 2.2);
    group.add(headLight);

    const spoiler = new THREE.Mesh(
      new THREE.BoxGeometry(1.4, 0.12, 0.35),
      new THREE.MeshStandardMaterial({ color: 0x111118, metalness: 0.6, emissive: 0x220011, emissiveIntensity: 0.15 })
    );
    spoiler.position.set(0, 1.15, -1.35);
    group.add(spoiler);
  }

  return group;
}

function createPlayerCar() {
  playerCar = createCar(0xff3333, true);
  playerCar.position.set(0, 0, 0);
  scene.add(playerCar);
}

function createOtherCar(z) {
  const color = carColors[Math.floor(Math.random() * carColors.length)];
  const car = createCar(color, false);
  const laneOffset = (Math.floor(Math.random() * 3) - 1) * LANE_WIDTH;
  car.position.set(laneOffset, 0, z);
  car.userData = { type: 'car', hit: false, speedOffset: (Math.random() - 0.5) * 4 };
  scene.add(car);
  otherCars.push(car);
}

function createQuestionBall(z) {
  const group = new THREE.Group();

  // Core glowing sphere
  const ball = new THREE.Mesh(
    new THREE.SphereGeometry(0.65, 28, 28),
    new THREE.MeshStandardMaterial({
      color: 0xffe066,
      emissive: 0xff9900,
      emissiveIntensity: 1.1,
      metalness: 0.25,
      roughness: 0.2
    })
  );
  ball.castShadow = true;
  group.add(ball);

  // Outer soft glow sphere
  const glow = new THREE.Mesh(
    new THREE.SphereGeometry(0.95, 16, 16),
    new THREE.MeshBasicMaterial({ color: 0xffcc44, transparent: true, opacity: 0.22 })
  );
  group.add(glow);

  // Rings
  const ring = new THREE.Mesh(
    new THREE.TorusGeometry(1.05, 0.07, 12, 36),
    new THREE.MeshBasicMaterial({ color: 0xffee88, transparent: true, opacity: 0.85 })
  );
  ring.rotation.x = Math.PI / 2;
  group.add(ring);

  const ring2 = new THREE.Mesh(
    new THREE.TorusGeometry(1.3, 0.04, 10, 32),
    new THREE.MeshBasicMaterial({ color: 0xffffff, transparent: true, opacity: 0.5 })
  );
  ring2.rotation.x = Math.PI / 3.2;
  group.add(ring2);

  // Sparkles
  for (let i = 0; i < 8; i++) {
    const spark = new THREE.Mesh(
      new THREE.OctahedronGeometry(0.08, 0),
      new THREE.MeshBasicMaterial({ color: 0xffffff })
    );
    const angle = (i / 8) * Math.PI * 2;
    spark.position.set(Math.cos(angle) * 1.15, 0.2 + Math.sin(angle * 2) * 0.3, Math.sin(angle) * 1.15);
    group.add(spark);
  }

  // Point light for glow
  const light = new THREE.PointLight(0xffaa33, 1.8, 12);
  light.position.y = 0.3;
  group.add(light);

  const laneOffset = (Math.floor(Math.random() * 3) - 1) * LANE_WIDTH;
  group.position.set(laneOffset, 1.5, z);
  group.userData = { type: 'question', collected: false };
  scene.add(group);
  questionBalls.push(group);
}

// ==================== GAME LOGIC ====================
function startGame() {
  document.getElementById('startScreen').classList.add('hidden');
  document.getElementById('gameOverScreen').classList.add('hidden');

  score = 0;
  lives = 3;
  speed = baseSpeed;
  hasPower = false;
  powerTimer = 0;
  distance = 0;
  lane = 0;
  targetX = 0;
  usedQuestions.clear();
  spawnTimer = 0;
  invincibleTimer = 0;

  otherCars.forEach(function(c) { scene.remove(c); });
  questionBalls.forEach(function(b) { scene.remove(b); });
  otherCars = [];
  questionBalls = [];

  playerCar.position.set(0, 0, 0);
  playerCar.rotation.set(0, 0, 0);

  for (let i = 0; i < 5; i++) {
    createOtherCar(-25 - i * 18);
  }

  updateHUD();
  isPlaying = true;
  isPaused = false;
}

function updateHUD() {
  document.getElementById('scoreVal').textContent = Math.floor(score);
  document.getElementById('livesVal').textContent = lives;
  const powerBox = document.getElementById('powerBox');
  if (hasPower) {
    powerBox.classList.add('active');
    powerBox.textContent = '👻 Invisible ' + Math.ceil(powerTimer) + 's';
  } else {
    powerBox.classList.remove('active');
  }
}

function showPowerNotify(msg) {
  const el = document.getElementById('powerNotify');
  el.textContent = msg;
  el.classList.add('show');
  setTimeout(function() { el.classList.remove('show'); }, 1600);
}

function hitOtherCar() {
  // Invisible power active → cars can't hit you
  if (hasPower) return;
  // very short invincibility only to prevent one car hitting multiple times in same frame
  if (invincibleTimer > 0) return;

  lives--;
  invincibleTimer = 0.9; // short grace only
  updateHUD();
  camera.position.x += (Math.random() - 0.5) * 0.45;
  if (lives <= 0) {
    endGame();
  } else {
    showPowerNotify('💥 Hit! Lives: ' + lives);
  }
}

function endGame() {
  isPlaying = false;
  document.getElementById('finalScore').textContent = Math.floor(score);
  document.getElementById('gameOverMsg').textContent =
    score > 800 ? 'සුපිරි Driver + Science Master!' :
    score > 350 ? 'හොඳයි! තවත් පුහුණු වන්න!' :
    'නැවත උත්සාහ කරන්න!';
  document.getElementById('gameOverScreen').classList.remove('hidden');
}

function triggerQuestion() {
  isPaused = true;
  let available = [];
  for (let i = 0; i < QUESTIONS.length; i++) {
    if (!usedQuestions.has(i)) available.push(i);
  }
  if (available.length === 0) {
    usedQuestions.clear();
    available = QUESTIONS.map(function(_, i) { return i; });
  }
  const idx = available[Math.floor(Math.random() * available.length)];
  usedQuestions.add(idx);
  currentQuestion = QUESTIONS[idx];

  document.getElementById('qText').textContent = currentQuestion.q;
  const opts = document.getElementById('qOptions');
  opts.innerHTML = '';
  document.getElementById('qFeedback').textContent = '';
  document.getElementById('qFeedback').className = 'feedback';

  currentQuestion.options.forEach(function(opt, i) {
    const btn = document.createElement('button');
    btn.className = 'opt';
    btn.textContent = String.fromCharCode(65 + i) + '. ' + opt;
    btn.onclick = function() { answerQuestion(i, btn); };
    opts.appendChild(btn);
  });

  qTimeLeft = 15;
  document.getElementById('qTimer').textContent = qTimeLeft;
  document.getElementById('qTimer').classList.remove('urgent');
  document.getElementById('questionModal').classList.add('show');

  if (qInterval) clearInterval(qInterval);
  qInterval = setInterval(function() {
    qTimeLeft--;
    document.getElementById('qTimer').textContent = qTimeLeft;
    if (qTimeLeft <= 5) document.getElementById('qTimer').classList.add('urgent');
    if (qTimeLeft <= 0) {
      clearInterval(qInterval);
      answerQuestion(-1, null);
    }
  }, 1000);
}

function answerQuestion(choice, btn) {
  clearInterval(qInterval);
  const opts = document.querySelectorAll('.opt');
  for (let i = 0; i < opts.length; i++) opts[i].disabled = true;

  const correct = currentQuestion.correct;
  if (btn) {
    if (choice === correct) {
      btn.classList.add('correct');
    } else {
      btn.classList.add('wrong');
      if (opts[correct]) opts[correct].classList.add('correct');
    }
  } else {
    if (opts[correct]) opts[correct].classList.add('correct');
  }

  const feedback = document.getElementById('qFeedback');
  if (choice === correct) {
    feedback.textContent = '✅ නිවැරදියි! Invisible බලය ලැබුණා!';
    feedback.className = 'feedback good';
    score += 150;
    hasPower = true;
    powerTimer = 10; // 10 seconds invisible
    showPowerNotify('👻 INVISIBLE!');
  } else {
    feedback.textContent = choice === -1 ? '⏰ කාලය අවසන්! Life -1' : '❌ වැරදියි! Life -1';
    feedback.className = 'feedback bad';
    lives--;
    if (lives <= 0) {
      setTimeout(function() {
        document.getElementById('questionModal').classList.remove('show');
        endGame();
      }, 1200);
      updateHUD();
      return;
    }
  }
  updateHUD();

  setTimeout(function() {
    document.getElementById('questionModal').classList.remove('show');
    isPaused = false;
  }, 1300);
}

function setupMobileControls() {
  const area = document.getElementById('swipeArea');
  if (!area) return;

  area.addEventListener('touchstart', function(e) {
    if (isPaused || !isPlaying) return;
    const t = e.changedTouches[0];
    touchStartX = t.clientX;
    touchStartY = t.clientY;
    touchStartTime = Date.now();
  }, { passive: true });

  area.addEventListener('touchend', function(e) {
    if (isPaused || !isPlaying) return;
    const t = e.changedTouches[0];
    const dx = t.clientX - touchStartX;
    const dy = t.clientY - touchStartY;
    const dt = Date.now() - touchStartTime;

    // Only horizontal swipe, not too slow, minimum distance
    if (dt < 400 && Math.abs(dx) > 40 && Math.abs(dx) > Math.abs(dy) * 1.2) {
      if (dx < 0 && lane > -1) {
        lane--;
      } else if (dx > 0 && lane < 1) {
        lane++;
      }
      targetX = lane * LANE_WIDTH;
    }
  }, { passive: true });
}

function handleInput(dt) {
  if ((keys['ArrowLeft'] || keys['KeyA']) && lane > -1) {
    lane--;
    keys['ArrowLeft'] = keys['KeyA'] = false;
  }
  if ((keys['ArrowRight'] || keys['KeyD']) && lane < 1) {
    lane++;
    keys['ArrowRight'] = keys['KeyD'] = false;
  }
  targetX = lane * LANE_WIDTH;

  playerCar.position.x += (targetX - playerCar.position.x) * Math.min(1, 10 * dt);

  const lean = (targetX - playerCar.position.x) * 0.08;
  playerCar.rotation.z = -lean;
}

function update(dt) {
  if (!isPlaying || isPaused) return;

  handleInput(dt);

  const move = speed * dt;
  distance += move;
  score += move * 1.1;

  roadMeshes.forEach(function(p) {
    p.position.z += move;
    if (p.position.z > 50) p.position.z -= 300;
  });

  for (let i = otherCars.length - 1; i >= 0; i--) {
    const c = otherCars[i];
    c.position.z += (move - c.userData.speedOffset * dt);

    if (c.position.z > 15) {
      scene.remove(c);
      otherCars.splice(i, 1);
      continue;
    }

    if (!c.userData.hit &&
        Math.abs(c.position.z - playerCar.position.z) < 2.6 &&
        Math.abs(c.position.x - playerCar.position.x) < 1.7) {
      c.userData.hit = true;
      hitOtherCar();
      c.children[0].material.emissive = new THREE.Color(0xff0000);
      c.children[0].material.emissiveIntensity = 0.6;
    }
  }

  for (let i = questionBalls.length - 1; i >= 0; i--) {
    const b = questionBalls[i];
    b.position.z += move;
    b.rotation.y += dt * 2.2;
    if (b.children[2]) b.children[2].rotation.z += dt * 3;   // ring
    if (b.children[3]) b.children[3].rotation.y += dt * 2.5; // ring2
    // pulse core emissive
    if (b.children[0] && b.children[0].material) {
      b.children[0].material.emissiveIntensity = 0.9 + Math.sin(clock.elapsedTime * 6) * 0.35;
    }

    for (let s = 4; s < b.children.length - 1; s++) {
      if (b.children[s]) b.children[s].position.y = 0.25 + Math.sin(clock.elapsedTime * 5 + s) * 0.28;
    }

    if (b.position.z > 12) {
      scene.remove(b);
      questionBalls.splice(i, 1);
      continue;
    }

    if (!b.userData.collected &&
        Math.abs(b.position.z - playerCar.position.z) < 2.2 &&
        Math.abs(b.position.x - playerCar.position.x) < 1.9) {
      b.userData.collected = true;
      scene.remove(b);
      questionBalls.splice(i, 1);
      triggerQuestion();
    }
  }

  spawnTimer += dt;
  if (spawnTimer > 1.1) {
    spawnTimer = 0;
    const z = -55 - Math.random() * 20;

    if (Math.random() < 0.28) {
      createQuestionBall(z);
    } else {
      createOtherCar(z);
    }
  }

  if (hasPower) {
    powerTimer -= dt;
    if (powerTimer <= 0) {
      hasPower = false;
      speed = baseSpeed + Math.min(distance * 0.012, 12);
      updateHUD();
    }
  } else {
    speed = baseSpeed + Math.min(distance * 0.012, 14);
  }

  // Invisible power visual + short post-hit flash
  if (invincibleTimer > 0) invincibleTimer -= dt;

  if (playerCar && playerCar.children[0]) {
    const mat = playerCar.children[0].material;
    if (hasPower) {
      // Invisible look – very transparent + slight blue tint
      mat.transparent = true;
      mat.opacity = 0.28;
      mat.emissive = new THREE.Color(0x4488ff);
      mat.emissiveIntensity = 0.35;
    } else if (invincibleTimer > 0) {
      mat.transparent = true;
      mat.opacity = (Math.floor(invincibleTimer * 10) % 2 === 0) ? 0.4 : 1;
      mat.emissive = new THREE.Color(0x000000);
      mat.emissiveIntensity = 0;
    } else {
      mat.transparent = false;
      mat.opacity = 1;
      mat.emissive = new THREE.Color(0x000000);
      mat.emissiveIntensity = 0;
    }
  }

  camera.position.x += (playerCar.position.x * 0.55 - camera.position.x) * 0.07;
  camera.position.y = 6.2;
  camera.lookAt(playerCar.position.x * 0.4, 1.2, playerCar.position.z - 8);

  playerCar.position.y = Math.sin(clock.elapsedTime * 8) * 0.03;

  updateHUD();
}

function onResize() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
}

function animate() {
  requestAnimationFrame(animate);
  const dt = Math.min(clock.getDelta(), 0.05);
  update(dt);
  renderer.render(scene, camera);
}

window.addEventListener('load', function() {
  setTimeout(init, 80);
});
</script>
</body>
</html>
