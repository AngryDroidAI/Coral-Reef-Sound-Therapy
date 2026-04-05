<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Coral Reef Sound Therapy</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { overflow: hidden; background: #000; font-family: 'Segoe UI', sans-serif; }
  canvas { display: block; }

  #ui {
    position: fixed; top: 20px; left: 20px; z-index: 10;
    color: #a0e8ff; pointer-events: none;
  }
  #ui h1 {
    font-size: 1.6rem; font-weight: 300; letter-spacing: 2px;
    text-shadow: 0 0 20px rgba(100,200,255,0.5);
    margin-bottom: 6px;
  }
  #ui p { font-size: 0.85rem; opacity: 0.7; max-width: 320px; line-height: 1.4; }
  #description { margin-top: 16px; max-width: 360px; }
  .desc-text { font-size: 0.78rem; opacity: 0.55; line-height: 1.5; margin-bottom: 8px; }
  .desc-text strong { color: #64c8ff; opacity: 1; }

  #controls {
    position: fixed; bottom: 30px; left: 50%; transform: translateX(-50%);
    display: flex; gap: 16px; align-items: center; z-index: 10;
  }
  .btn {
    padding: 12px 28px; border: 1px solid rgba(100,200,255,0.4);
    background: rgba(0,30,60,0.6); color: #a0e8ff; font-size: 0.95rem;
    border-radius: 30px; cursor: pointer; letter-spacing: 1px;
    transition: all 0.3s; backdrop-filter: blur(10px);
  }
  .btn:hover { background: rgba(0,60,100,0.7); border-color: rgba(100,200,255,0.8); box-shadow: 0 0 20px rgba(100,200,255,0.3); }
  .btn.active { background: rgba(0,80,140,0.7); border-color: #64c8ff; box-shadow: 0 0 25px rgba(100,200,255,0.5); }

  #stats {
    position: fixed; top: 20px; right: 20px; z-index: 10;
    color: #a0e8ff; text-align: right;
  }
  .stat { margin-bottom: 12px; }
  .stat-label { font-size: 0.7rem; opacity: 0.6; text-transform: uppercase; letter-spacing: 1px; }
  .stat-value { font-size: 1.4rem; font-weight: 300; }
  .stat-bar { width: 120px; height: 3px; background: rgba(100,200,255,0.2); border-radius: 2px; margin-top: 4px; }
  .stat-fill { height: 100%; background: linear-gradient(90deg, #0088cc, #00ffcc); border-radius: 2px; transition: width 0.5s; }

  #overlay {
    position: fixed; inset: 0; background: rgba(0,10,20,0.85);
    display: flex; align-items: center; justify-content: center;
    z-index: 100; flex-direction: column; gap: 20px;
  }
  #overlay h2 { color: #a0e8ff; font-weight: 300; font-size: 2rem; letter-spacing: 3px; }
  #overlay p { color: rgba(160,232,255,0.6); max-width: 400px; text-align: center; line-height: 1.6; }
  #overlay .btn { pointer-events: all; font-size: 1.1rem; padding: 14px 40px; }
</style>
</head>
<body>

<div id="overlay">
  <h2>CORAL REEF SOUND THERAPY</h2>
  <p>Experience how the sounds of a healthy reef can bring dying coral ecosystems back to life. Click to begin the simulation.</p>
  <button class="btn" id="startBtn">ENTER THE REEF</button>
</div>

<div id="ui">
  <h1>Coral Reef Sound Therapy</h1>
  <p>Playing healthy reef sounds to attract marine life and restore the ecosystem</p>
  <div id="description">
    <p class="desc-text">Degraded coral reefs are silent and lifeless. Healthy reefs produce a rich soundscape — crackling shrimp, popping snapping shrimp, and low-frequency fish calls — that acts as a beacon for fish larvae searching for a home.</p>
    <p class="desc-text">This simulation demonstrates how underwater acoustic enrichment can bring a dying reef back to life. Click <strong>"Play Reef Sounds"</strong> to broadcast healthy reef audio, watch sound waves ripple outward, and observe as fish are drawn to the reef and coral health gradually recovers.</p>
  </div>
</div>

<div id="stats">
  <div class="stat">
    <div class="stat-label">Reef Health</div>
    <div class="stat-value" id="healthVal">12%</div>
    <div class="stat-bar"><div class="stat-fill" id="healthBar" style="width:12%"></div></div>
  </div>
  <div class="stat">
    <div class="stat-label">Fish Species</div>
    <div class="stat-value" id="fishVal">2</div>
  </div>
  <div class="stat">
    <div class="stat-label">Sound Waves</div>
    <div class="stat-value" id="waveVal">0</div>
  </div>
</div>

<div id="controls">
  <button class="btn" id="soundBtn">Play Reef Sounds</button>
  <button class="btn" id="resetBtn">Reset Simulation</button>
</div>

<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.163.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.163.0/examples/jsm/"
  }
}
</script>

<script type="module">
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

// Scene setup
const scene = new THREE.Scene();
scene.fog = new THREE.FogExp2(0x001828, 0.025);

const camera = new THREE.PerspectiveCamera(60, innerWidth / innerHeight, 0.1, 200);
camera.position.set(0, 4, 12);

const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
renderer.setSize(innerWidth, innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 0.8;
document.body.appendChild(renderer.domElement);

const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.05;
controls.maxPolarAngle = Math.PI * 0.85;
controls.minDistance = 5;
controls.maxDistance = 35;
controls.target.set(0, 1, 0);

// Lighting
const ambient = new THREE.AmbientLight(0x1a3a5c, 0.6);
scene.add(ambient);

const sun = new THREE.DirectionalLight(0x88ccff, 1.2);
sun.position.set(5, 20, 5);
scene.add(sun);

const godRays = new THREE.PointLight(0x00aaff, 0.5, 50);
godRays.position.set(0, 15, 0);
scene.add(godRays);

// Water surface
const waterGeo = new THREE.PlaneGeometry(100, 100, 64, 64);
const waterMat = new THREE.MeshPhysicalMaterial({
  color: 0x0066aa, transparent: true, opacity: 0.3,
  roughness: 0.1, metalness: 0.1, side: THREE.DoubleSide
});
const water = new THREE.Mesh(waterGeo, waterMat);
water.rotation.x = -Math.PI / 2;
water.position.y = 12;
scene.add(water);

// Seafloor
const floorGeo = new THREE.PlaneGeometry(60, 60, 128, 128);
const floorMat = new THREE.MeshStandardMaterial({
  color: 0x2a1f1a, roughness: 0.95, metalness: 0.05
});
const floor = new THREE.Mesh(floorGeo, floorMat);
floor.rotation.x = -Math.PI / 2;
floor.position.y = -1;
scene.add(floor);

// Coral classes
class Coral {
  constructor(x, z, type, size) {
    this.group = new THREE.Group();
    this.type = type;
    this.size = size;
    this.health = 0;
    this.targetHealth = 0;
    this.aliveParts = [];
    this.deadParts = [];
    this.x = x;
    this.z = z;

    this.build();
    this.group.position.set(x, -1, z);
    scene.add(this.group);
  }

  build() {
    if (this.type === 'brain') this.buildBrain();
    else if (this.type === 'staghorn') this.buildStaghorn();
    else if (this.type === 'tube') this.buildTube();
    else if (this.type === 'anemone') this.buildAnemone();
    else this.buildBrain();
  }

  buildBrain() {
    const geo = new THREE.SphereGeometry(this.size, 32, 16, 0, Math.PI * 2, 0, Math.PI * 0.6);
    const deadMat = new THREE.MeshStandardMaterial({ color: 0x8a8a8a, roughness: 0.8 });
    const aliveMat = new THREE.MeshStandardMaterial({ color: 0x22aa66, roughness: 0.6, emissive: 0x114422, emissiveIntensity: 0.2 });

    this.deadMesh = new THREE.Mesh(geo, deadMat);
    this.deadMesh.scale.set(1, 0.6, 1);
    this.group.add(this.deadMesh);
    this.deadParts.push(this.deadMesh);

    this.aliveMesh = new THREE.Mesh(geo.clone(), aliveMat.clone());
    this.aliveMesh.scale.set(1, 0.6, 1);
    this.aliveMesh.visible = false;
    this.group.add(this.aliveMesh);
    this.aliveParts.push(this.aliveMesh);
  }

  buildStaghorn() {
    const deadMat = new THREE.MeshStandardMaterial({ color: 0x9a9a9a, roughness: 0.7 });
    const aliveMat = new THREE.MeshStandardMaterial({ color: 0x44bb88, roughness: 0.5, emissive: 0x225533, emissiveIntensity: 0.15 });

    const branches = 5 + Math.floor(Math.random() * 4);
    for (let i = 0; i < branches; i++) {
      const h = this.size * (0.5 + Math.random() * 0.5);
      const geo = new THREE.CylinderGeometry(0.05 * this.size, 0.1 * this.size, h, 8);
      const dead = new THREE.Mesh(geo, deadMat.clone());
      const angle = (i / branches) * Math.PI * 2;
      dead.position.set(Math.cos(angle) * 0.3 * this.size, h * 0.3, Math.sin(angle) * 0.3 * this.size);
      dead.rotation.z = Math.cos(angle) * 0.4;
      dead.rotation.x = Math.sin(angle) * 0.4;
      this.group.add(dead);
      this.deadParts.push(dead);

      const alive = new THREE.Mesh(geo.clone(), aliveMat.clone());
      alive.position.copy(dead.position);
      alive.rotation.copy(dead.rotation);
      alive.visible = false;
      this.group.add(alive);
      this.aliveParts.push(alive);
    }
  }

  buildTube() {
    const deadMat = new THREE.MeshStandardMaterial({ color: 0x7a7a7a, roughness: 0.8 });
    const aliveMat = new THREE.MeshStandardMaterial({ color: 0x6644aa, roughness: 0.5, emissive: 0x332266, emissiveIntensity: 0.2 });

    const count = 3 + Math.floor(Math.random() * 4);
    for (let i = 0; i < count; i++) {
      const h = this.size * (0.4 + Math.random() * 0.6);
      const geo = new THREE.CylinderGeometry(0.08 * this.size, 0.12 * this.size, h, 12);
      const dead = new THREE.Mesh(geo, deadMat.clone());
      dead.position.set((Math.random() - 0.5) * 0.5 * this.size, h * 0.3, (Math.random() - 0.5) * 0.5 * this.size);
      this.group.add(dead);
      this.deadParts.push(dead);

      const alive = new THREE.Mesh(geo.clone(), aliveMat.clone());
      alive.position.copy(dead.position);
      alive.visible = false;
      this.group.add(alive);
      this.aliveParts.push(alive);
    }
  }

  buildAnemone() {
    const deadMat = new THREE.MeshStandardMaterial({ color: 0x888888, roughness: 0.7 });
    const aliveMat = new THREE.MeshStandardMaterial({ color: 0xff66aa, roughness: 0.4, emissive: 0x662233, emissiveIntensity: 0.15 });

    const baseGeo = new THREE.CylinderGeometry(0.3 * this.size, 0.4 * this.size, 0.3 * this.size, 16);
    const base = new THREE.Mesh(baseGeo, deadMat.clone());
    base.position.y = 0.15 * this.size;
    this.group.add(base);
    this.deadParts.push(base);

    const tentacles = 12;
    for (let i = 0; i < tentacles; i++) {
      const angle = (i / tentacles) * Math.PI * 2;
      const h = this.size * 0.8;
      const geo = new THREE.CylinderGeometry(0.02, 0.04, h, 6);
      const dead = new THREE.Mesh(geo, deadMat.clone());
      dead.position.set(Math.cos(angle) * 0.25 * this.size, 0.3 * this.size + h * 0.3, Math.sin(angle) * 0.25 * this.size);
      dead.rotation.z = Math.cos(angle) * 0.3;
      dead.rotation.x = Math.sin(angle) * 0.3;
      this.group.add(dead);
      this.deadParts.push(dead);

      const alive = new THREE.Mesh(geo.clone(), aliveMat.clone());
      alive.position.copy(dead.position);
      alive.rotation.copy(dead.rotation);
      alive.visible = false;
      this.group.add(alive);
      this.aliveParts.push(alive);
    }
  }

  setHealth(h) {
    this.health = h;
    this.aliveParts.forEach(p => {
      p.visible = h > 0.05;
      p.material.opacity = h;
      p.material.transparent = true;
    });
    this.deadParts.forEach(p => {
      p.material.opacity = 1 - h * 0.7;
      p.material.transparent = true;
    });
  }

  update(time) {
    this.aliveParts.forEach((p, i) => {
      if (p.visible) {
        p.position.y += Math.sin(time * 2 + i) * 0.001;
      }
    });
  }
}

// Create corals
const corals = [];
const coralPositions = [];

function createReef() {
  corals.forEach(c => scene.remove(c.group));
  corals.length = 0;
  coralPositions.length = 0;

  const types = ['brain', 'staghorn', 'tube', 'anemone'];
  for (let i = 0; i < 25; i++) {
    const angle = Math.random() * Math.PI * 2;
    const radius = 2 + Math.random() * 10;
    const x = Math.cos(angle) * radius;
    const z = Math.sin(angle) * radius;
    const type = types[Math.floor(Math.random() * types.length)];
    const size = 0.5 + Math.random() * 1.2;

    coralPositions.push({ x, z });
    const coral = new Coral(x, z, type, size);
    coral.setHealth(0.05 + Math.random() * 0.1);
    corals.push(coral);
  }
}
createReef();

// Sound speaker
const speakerGeo = new THREE.CylinderGeometry(0.4, 0.5, 0.3, 32);
const speakerMat = new THREE.MeshStandardMaterial({ color: 0x222222, metalness: 0.8, roughness: 0.3 });
const speaker = new THREE.Mesh(speakerGeo, speakerMat);
speaker.position.set(0, 0.5, 0);
scene.add(speaker);

const speakerGlow = new THREE.PointLight(0x00aaff, 0, 15);
speakerGlow.position.copy(speaker.position);
scene.add(speakerGlow);

// Sound waves
const soundWaves = [];
function createSoundWave() {
  const geo = new THREE.RingGeometry(0.5, 0.6, 64);
  const mat = new THREE.MeshBasicMaterial({
    color: 0x00ccff, transparent: true, opacity: 0.6, side: THREE.DoubleSide
  });
  const ring = new THREE.Mesh(geo, mat);
  ring.rotation.x = -Math.PI / 2;
  ring.position.set(0, 0.5, 0);
  ring.userData = { radius: 0.5, speed: 2 + Math.random(), life: 1 };
  scene.add(ring);
  soundWaves.push(ring);
  return ring;
}

// Fish
class Fish {
  constructor() {
    this.group = new THREE.Group();
    this.speed = 1 + Math.random() * 2;
    this.angle = Math.random() * Math.PI * 2;
    this.radius = 3 + Math.random() * 8;
    this.y = 1 + Math.random() * 4;
    this.wobble = Math.random() * Math.PI * 2;
    this.attracted = false;
    this.arrived = false;

    this.build();
    this.group.position.set(
      Math.cos(this.angle) * this.radius,
      this.y,
      Math.sin(this.angle) * this.radius
    );
    scene.add(this.group);
  }

  build() {
    const hue = Math.random();
    const color = new THREE.Color().setHSL(hue, 0.8, 0.5);
    const bodyMat = new THREE.MeshStandardMaterial({ color, roughness: 0.4, metalness: 0.3 });

    const bodyGeo = new THREE.SphereGeometry(0.15, 8, 6);
    bodyGeo.scale(1.8, 0.6, 0.5);
    const body = new THREE.Mesh(bodyGeo, bodyMat);
    this.group.add(body);

    const tailGeo = new THREE.ConeGeometry(0.12, 0.2, 4);
    const tail = new THREE.Mesh(tailGeo, bodyMat);
    tail.rotation.z = Math.PI / 2;
    tail.position.x = -0.3;
    this.group.add(tail);
    this.tail = tail;

    const eyeGeo = new THREE.SphereGeometry(0.03, 6, 6);
    const eyeMat = new THREE.MeshBasicMaterial({ color: 0xffffff });
    const eye = new THREE.Mesh(eyeGeo, eyeMat);
    eye.position.set(0.2, 0.04, 0.06);
    this.group.add(eye);

    const pupilGeo = new THREE.SphereGeometry(0.015, 6, 6);
    const pupilMat = new THREE.MeshBasicMaterial({ color: 0x000000 });
    const pupil = new THREE.Mesh(pupilGeo, pupilMat);
    pupil.position.set(0.22, 0.04, 0.06);
    this.group.add(pupil);
  }

  update(time, dt, soundPlaying) {
    if (soundPlaying && !this.attracted && Math.random() < 0.001) {
      this.attracted = true;
    }

    if (this.attracted && !this.arrived) {
      const target = new THREE.Vector3(0, 2, 0);
      const dir = target.clone().sub(this.group.position).normalize();
      this.group.position.add(dir.multiplyScalar(this.speed * dt * 2));

      if (this.group.position.distanceTo(target) < 2) {
        this.arrived = true;
      }
    }

    if (!this.arrived) {
      this.angle += this.speed * dt * 0.3;
      this.group.position.x = Math.cos(this.angle) * this.radius;
      this.group.position.z = Math.sin(this.angle) * this.radius;
      this.group.position.y = this.y + Math.sin(time * 2 + this.wobble) * 0.3;
    } else {
      this.group.position.y = 2 + Math.sin(time * 1.5 + this.wobble) * 0.5;
      this.angle += this.speed * dt * 0.15;
      const r = 1.5 + Math.sin(time * 0.5) * 0.5;
      this.group.position.x = Math.cos(this.angle) * r;
      this.group.position.z = Math.sin(this.angle) * r;
    }

    this.group.rotation.y = -this.angle + Math.PI / 2;
    this.tail.rotation.y = Math.sin(time * 8) * 0.3;
  }
}

const fishes = [];
function spawnFish(count) {
  for (let i = 0; i < count; i++) {
    fishes.push(new Fish());
  }
}
spawnFish(2);

// Bubbles
const bubbles = [];
function createBubble() {
  const geo = new THREE.SphereGeometry(0.03 + Math.random() * 0.05, 8, 8);
  const mat = new THREE.MeshPhysicalMaterial({
    color: 0xffffff, transparent: true, opacity: 0.4,
    roughness: 0, metalness: 0.1, transmission: 0.8
  });
  const bubble = new THREE.Mesh(geo, mat);
  bubble.position.set(
    (Math.random() - 0.5) * 20,
    -1,
    (Math.random() - 0.5) * 20
  );
  bubble.userData.speed = 0.5 + Math.random() * 1;
  scene.add(bubble);
  bubbles.push(bubble);
}
for (let i = 0; i < 50; i++) createBubble();

// Light rays
const rays = [];
for (let i = 0; i < 8; i++) {
  const geo = new THREE.CylinderGeometry(0.05, 0.3, 15, 8);
  const mat = new THREE.MeshBasicMaterial({
    color: 0x4488cc, transparent: true, opacity: 0.03, side: THREE.DoubleSide
  });
  const ray = new THREE.Mesh(geo, mat);
  ray.position.set((Math.random() - 0.5) * 15, 5, (Math.random() - 0.5) * 15);
  ray.rotation.z = (Math.random() - 0.5) * 0.3;
  scene.add(ray);
  rays.push(ray);
}

// Particles
const particleCount = 500;
const particleGeo = new THREE.BufferGeometry();
const positions = new Float32Array(particleCount * 3);
for (let i = 0; i < particleCount; i++) {
  positions[i * 3] = (Math.random() - 0.5) * 30;
  positions[i * 3 + 1] = Math.random() * 12 - 1;
  positions[i * 3 + 2] = (Math.random() - 0.5) * 30;
}
particleGeo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
const particleMat = new THREE.PointsMaterial({
  color: 0x88ccff, size: 0.05, transparent: true, opacity: 0.4
});
const particles = new THREE.Points(particleGeo, particleMat);
scene.add(particles);

// Audio context for reef sounds
let audioCtx, soundPlaying = false, waveInterval;
let totalWaves = 0;

function createReefSound() {
  audioCtx = new (window.AudioContext || window.webkitAudioContext)();

  function playCrackle() {
    if (!soundPlaying) return;
    const bufferSize = audioCtx.sampleRate * 0.1;
    const buffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate);
    const data = buffer.getChannelData(0);
    for (let i = 0; i < bufferSize; i++) {
      data[i] = (Math.random() * 2 - 1) * Math.exp(-i / (bufferSize * 0.1));
    }
    const source = audioCtx.createBufferSource();
    source.buffer = buffer;

    const filter = audioCtx.createBiquadFilter();
    filter.type = 'bandpass';
    filter.frequency.value = 2000 + Math.random() * 3000;
    filter.Q.value = 1 + Math.random() * 2;

    const gain = audioCtx.createGain();
    gain.gain.value = 0.08;

    source.connect(filter).connect(gain).connect(audioCtx.destination);
    source.start();
    setTimeout(playCrackle, 50 + Math.random() * 200);
  }

  function playHum() {
    if (!soundPlaying) return;
    const osc = audioCtx.createOscillator();
    osc.type = 'sine';
    osc.frequency.value = 80 + Math.random() * 40;

    const gain = audioCtx.createGain();
    gain.gain.value = 0.03;
    gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 2);

    osc.connect(gain).connect(audioCtx.destination);
    osc.start();
    osc.stop(audioCtx.currentTime + 2);
    setTimeout(playHum, 1000 + Math.random() * 3000);
  }

  function playPop() {
    if (!soundPlaying) return;
    const osc = audioCtx.createOscillator();
    osc.type = 'sine';
    osc.frequency.value = 800 + Math.random() * 2000;
    osc.frequency.exponentialRampToValueAtTime(100, audioCtx.currentTime + 0.1);

    const gain = audioCtx.createGain();
    gain.gain.value = 0.05;
    gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.1);

    osc.connect(gain).connect(audioCtx.destination);
    osc.start();
    osc.stop(audioCtx.currentTime + 0.1);
    setTimeout(playPop, 200 + Math.random() * 1000);
  }

  playCrackle();
  playHum();
  playPop();
}

// UI
const overlay = document.getElementById('overlay');
const startBtn = document.getElementById('startBtn');
const soundBtn = document.getElementById('soundBtn');
const resetBtn = document.getElementById('resetBtn');
const healthVal = document.getElementById('healthVal');
const healthBar = document.getElementById('healthBar');
const fishVal = document.getElementById('fishVal');
const waveVal = document.getElementById('waveVal');

let simulationStarted = false;
let reefHealth = 0.12;
let fishCount = 2;

startBtn.addEventListener('click', () => {
  overlay.style.display = 'none';
  simulationStarted = true;
});

soundBtn.addEventListener('click', () => {
  soundPlaying = !soundPlaying;
  soundBtn.classList.toggle('active', soundPlaying);
  soundBtn.textContent = soundPlaying ? 'Stop Reef Sounds' : 'Play Reef Sounds';

  if (soundPlaying) {
    if (!audioCtx) createReefSound();
    else audioCtx.resume();

    waveInterval = setInterval(() => {
      createSoundWave();
      totalWaves++;
      waveVal.textContent = totalWaves;
    }, 600);
  } else {
    clearInterval(waveInterval);
  }
});

resetBtn.addEventListener('click', () => {
  soundPlaying = false;
  soundBtn.classList.remove('active');
  soundBtn.textContent = 'Play Reef Sounds';
  clearInterval(waveInterval);

  reefHealth = 0.12;
  totalWaves = 0;
  waveVal.textContent = '0';
  fishCount = 2;
  fishVal.textContent = '2';

  corals.forEach(c => c.setHealth(0.05 + Math.random() * 0.1));

  fishes.forEach(f => {
    scene.remove(f.group);
    f.attracted = false;
    f.arrived = false;
    f.angle = Math.random() * Math.PI * 2;
    f.radius = 3 + Math.random() * 8;
  });
  fishes.length = 0;
  spawnFish(2);

  soundWaves.forEach(w => scene.remove(w));
  soundWaves.length = 0;
});

// Animation loop
const clock = new THREE.Clock();

function animate() {
  requestAnimationFrame(animate);
  const dt = clock.getDelta();
  const time = clock.getElapsedTime();

  controls.update();

  // Water animation
  const pos = waterGeo.attributes.position;
  for (let i = 0; i < pos.count; i++) {
    const x = pos.getX(i);
    const y = pos.getY(i);
    pos.setZ(i, Math.sin(x * 0.5 + time) * 0.15 + Math.cos(y * 0.3 + time * 0.7) * 0.1);
  }
  pos.needsUpdate = true;

  // Sound waves
  for (let i = soundWaves.length - 1; i >= 0; i--) {
    const wave = soundWaves[i];
    wave.userData.radius += wave.userData.speed * dt;
    wave.scale.setScalar(wave.userData.radius);
    wave.material.opacity = 0.6 * (1 - wave.userData.radius / 20);

    if (wave.userData.radius > 20) {
      scene.remove(wave);
      soundWaves.splice(i, 1);
    }
  }

  // Update corals
  if (soundPlaying && simulationStarted) {
    reefHealth = Math.min(1, reefHealth + dt * 0.02);
  }

  corals.forEach(c => {
    c.setHealth(reefHealth);
    c.update(time);
  });

  // Speaker glow
  speakerGlow.intensity = soundPlaying ? 1 + Math.sin(time * 4) * 0.3 : 0;

  // Fish spawning based on health
  const targetFish = Math.floor(reefHealth * 30);
  if (fishes.length < targetFish && simulationStarted) {
    spawnFish(1);
    fishCount = fishes.length;
    fishVal.textContent = fishCount;
  }

  // Update fish
  fishes.forEach(f => f.update(time, dt, soundPlaying));

  // Bubbles
  bubbles.forEach(b => {
    b.position.y += b.userData.speed * dt;
    b.position.x += Math.sin(time + b.position.z) * 0.005;
    if (b.position.y > 12) {
      b.position.y = -1;
      b.position.x = (Math.random() - 0.5) * 20;
      b.position.z = (Math.random() - 0.5) * 20;
    }
  });

  // Particles
  const pPos = particleGeo.attributes.position;
  for (let i = 0; i < particleCount; i++) {
    let y = pPos.getY(i);
    y += Math.sin(time * 0.5 + i) * 0.002;
    pPos.setY(i, y);
  }
  pPos.needsUpdate = true;

  // Light rays
  rays.forEach((r, i) => {
    r.material.opacity = 0.02 + Math.sin(time * 0.5 + i) * 0.01;
  });

  // Update stats
  healthVal.textContent = Math.floor(reefHealth * 100) + '%';
  healthBar.style.width = (reefHealth * 100) + '%';

  // Color shift based on health
  const hue = THREE.MathUtils.lerp(0.55, 0.35, reefHealth);
  scene.fog.color.setHSL(hue, 0.5, 0.05 + reefHealth * 0.05);
  renderer.toneMappingExposure = 0.8 + reefHealth * 0.5;

  renderer.render(scene, camera);
}

animate();

// Resize
window.addEventListener('resize', () => {
  camera.aspect = innerWidth / innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(innerWidth, innerHeight);
});
</script>
</body>
</html>
