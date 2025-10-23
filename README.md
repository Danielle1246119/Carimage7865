<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Hyper-real 3D Car Showcase</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;800&display=swap" rel="stylesheet">
<style>
  :root{
    --w: 980px;           /* container width */
    --h: 520px;
    --bg: linear-gradient(180deg,#071018 0%, #020406 100%);
    --glass-shine: 0.25;
    --specular-intensity: 0.35;
    --shadow-opacity: 0.5;
    --perspective: 1400px;
    --rx: 0deg;           /* rotation X - updated by JS */
    --ry: 0deg;           /* rotation Y - updated by JS */
    --tilt: 6deg;         /* static tilt for composition */
  }

  html,body{height:100%; margin:0; font-family:Inter,system-ui,-apple-system,Segoe UI,Roboto,"Helvetica Neue",Arial;}
  body{
    display:flex;
    align-items:center;
    justify-content:center;
    min-height:100vh;
    background: var(--bg);
    color:#ddd;
    -webkit-font-smoothing:antialiased;
    -moz-osx-font-smoothing:grayscale;
  }

  .stage{
    width:calc(var(--w));
    max-width:92vw;
    height:calc(var(--h));
    perspective: var(--perspective);
    position:relative;
    user-select:none;
  }

  /* ground / floor */
  .floor {
    position:absolute;
    inset: 50% -10% auto -10%;
    height: 48%;
    background:
      radial-gradient(800px 120px at 50% 0%, rgba(255,255,255,0.08), transparent 30%),
      linear-gradient(180deg, rgba(0,0,0,0) 0%, rgba(0,0,0,0.65) 100%);
    transform: translateY(10%);
    filter: blur(14px) saturate(0.9);
    z-index: 1;
    border-radius: 12px;
    overflow:hidden;
  }

  /* main car wrapper that receives rotation & tilt */
  .car-wrap{
    width:100%;
    height:100%;
    transform-style:preserve-3d;
    transition: transform 400ms cubic-bezier(.2,.9,.3,1);
    transform:
      rotateX(calc(var(--rx))) 
      rotateY(calc(var(--ry)))
      rotateZ(var(--tilt));
    position:relative;
    z-index: 10;
    display:flex;
    align-items:center;
    justify-content:center;
    will-change:transform;
  }

  /* cinematic vignette */
  .stage::after{
    content:"";
    position:absolute;
    inset:0;
    pointer-events:none;
    background: radial-gradient(60% 60% at 50% 30%, rgba(0,0,0,0) 0%, rgba(0,0,0,0.45) 60%);
    z-index:50;
    mix-blend-mode:multiply;
  }

  /* car card */
  .car {
    width:86%;
    height:auto;
    max-height:86%;
    position:relative;
    transform-style:preserve-3d;
    transition: transform 400ms cubic-bezier(.2,.9,.3,1), filter 300ms;
    will-change: transform, filter;
  }

  /* main photo layer */
  .car img.main {
    display:block;
    width:100%;
    height:auto;
    border-radius:14px;
    box-shadow:
      0 28px 80px rgba(0,0,0,0.6),
      inset 0 1px 0 rgba(255,255,255,0.03);
    transform: translateZ(60px) rotateZ(-0.8deg); /* slight forward pop */
    backface-visibility:hidden;
    filter: saturate(1.05) contrast(1.02) brightness(1.02);
  }

  /* subtle environment reflection (overlay) */
  .env {
    position:absolute;
    inset:0;
    pointer-events:none;
    border-radius:14px;
    mix-blend-mode:screen;
    background:
      linear-gradient(120deg, rgba(255,255,255,0.06), rgba(255,255,255,0.02) 20%, rgba(0,0,0,0) 60%),
      radial-gradient(600px 120px at 10% 30%, rgba(255,255,255,0.12), transparent 20%),
      radial-gradient(400px 80px at 90% 20%, rgba(255,255,255,0.08), transparent 30%);
    transform: translateZ(90px) scale(1.02);
    opacity: var(--specular-intensity);
  }

  /* glass highlight (simulate windshield/specular) */
  .glass {
    position:absolute;
    inset: 8% 6% 58% 6%;
    pointer-events:none;
    border-radius:14px;
    background: linear-gradient(110deg, rgba(255,255,255,0.12), rgba(255,255,255,0.02));
    transform: translateZ(110px) rotateZ(-1deg) skewY(-1deg);
    mix-blend-mode:overlay;
    opacity: var(--glass-shine);
    filter: blur(8px);
  }

  /* rim highlight under car to mimic light hitting lower edges */
  .lower-highlight{
    position:absolute;
    inset:auto 6% 8% 6%;
    height:4.6rem;
    border-radius:6px;
    background: linear-gradient(90deg, rgba(255,255,255,0.05), rgba(255,255,255,0.01));
    transform: translateZ(40px) rotateZ(-0.6deg);
    mix-blend-mode:soft-light;
    opacity:0.9;
  }

  /* soft reflection of the car on the floor */
  .car-reflection {
    position:absolute;
    left:7%;
    right:7%;
    bottom:6%;
    height:36%;
    transform-origin:top center;
    transform: translateZ(10px) scaleY(-1) perspective(800px) rotateX(180deg);
    background-image: linear-gradient(180deg, rgba(255,255,255,0.12), rgba(255,255,255,0));
    mask-image: linear-gradient(180deg, rgba(0,0,0,1), rgba(0,0,0,0.12));
    opacity:0.28;
    filter: blur(10px) saturate(0.65);
    z-index:2;
    border-radius:12px;
  }

  /* UI caption */
  .meta {
    position:absolute;
    left:12px;
    bottom:12px;
    z-index:60;
    display:flex;
    gap:14px;
    align-items:center;
    color:#cfd8df;
    background: rgba(255,255,255,0.02);
    padding:10px 14px;
    border-radius:10px;
    backdrop-filter: blur(6px) saturate(1.1);
    border: 1px solid rgba(255,255,255,0.03);
    font-weight:600;
    font-size:13px;
  }

  .meta .badge {
    display:inline-flex;
    align-items:center;
    justify-content:center;
    height:34px;
    min-width:34px;
    padding:8px;
    border-radius:8px;
    background: linear-gradient(180deg, rgba(255,255,255,0.04), rgba(255,255,255,0.02));
  }

  /* subtle hover pop */
  .stage:hover .car {
    transform: translateZ(12px) rotateZ(0);
    filter: drop-shadow(0 40px 120px rgba(0,0,0,0.6));
  }

  /* caption below stage */
  .caption {
    margin-top:18px;
    text-align:center;
    width:100%;
    color:#9fb0bd;
    font-size:13px;
    letter-spacing:0.6px;
  }

  /* small responsive tweaks */
  @media (max-width:720px){
    :root{ --h:420px; --w:340px; }
    .meta{ font-size:12px; padding:8px; gap:8px;}
  }
</style>
</head>
<body>

<div class="stage" id="stage" aria-label="3D car showcase">
  <div class="floor" aria-hidden="true"></div>

  <div class="car-wrap" id="carWrap">
    <div class="car" role="img" aria-label="Realistic car render">
      <!-- Replace the sample URL with your high-res car render or photo -->
      <!-- Prefer a studio render PNG with transparent background if you have it -->
      <img class="main" id="carImg"
           src="https://images.unsplash.com/photo-1603733828913-3c235b0f0f6a?auto=format&fit=crop&w=1600&q=80"
           alt="Photorealistic sports car">
      <div class="env" aria-hidden="true"></div>
      <div class="glass" aria-hidden="true"></div>
      <div class="lower-highlight" aria-hidden="true"></div>
      <div class="car-reflection" aria-hidden="true" style="background-image: url('https://images.unsplash.com/photo-1603733828913-3c235b0f0f6a?auto=format&fit=crop&w=1600&q=80'); background-size:cover; background-position:center;"></div>
    </div>
  </div>

  <div class="meta" aria-hidden="true">
    <div class="badge">✦</div>
    <div>
      <div style="font-weight:800; font-size:14px; color:#eff6fb">Studio Render — V1</div>
      <div style="font-weight:400; font-size:12px; color:#9fb0bd">HDR lighting · Cinematic reflection</div>
    </div>
  </div>
</div>

<p class="caption">Move your mouse across the image to tilt the car and see the parallax 3D effect.</p>

<script>
  /* Tiny mouse-driven parallax -> updates CSS vars for transform */
  (function(){
    const stage = document.getElementById('stage');
    const root = document.documentElement;
    let rect = stage.getBoundingClientRect();

    function updateRect(){ rect = stage.getBoundingClientRect(); }

    window.addEventListener('resize', updateRect);
    stage.addEventListener('mousemove', function(e){
      const cx = rect.left + rect.width/2;
      const cy = rect.top + rect.height/2;
      const dx = (e.clientX - cx) / (rect.width/2);  // -1 .. 1
      const dy = (e.clientY - cy) / (rect.height/2); // -1 .. 1

      // sensitivity
      const maxRy = 10; // deg
      const maxRx = 6;  // deg

      const ry = (-dx * maxRy).toFixed(2) + 'deg';
      const rx = (dy * maxRx).toFixed(2) + 'deg';

      root.style.setProperty('--ry', ry);
      root.style.setProperty('--rx', rx);
    });

    stage.addEventListener('mouseleave', function(){
      // return to neutral slowly
      root.style.setProperty('--ry', '0deg');
      root.style.setProperty('--rx', '0deg');
    });
  }());
</script>

</body>
</html>
