<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>эхо</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  html, body {
    background: #000;
    color: #c8cce0;
    min-height: 100vh;
    overflow: hidden;
    font-family: 'Georgia', 'Times New Roman', serif;
  }

  /* Пульсирующая тьма */
  .bg-pulse {
    position: fixed; inset: 0; z-index: 0;
    background: radial-gradient(ellipse at center,
      rgba(15,18,25,0.7) 0%,
      rgba(5,6,10,0.95) 60%,
      #000 100%);
    animation: bgBreathe 8s ease-in-out infinite;
  }
  @keyframes bgBreathe {
    0%, 100% { opacity: 0.7; } 50% { opacity: 1; }
  }

  .vignette {
    position: fixed; inset: 0; z-index: 2;
    pointer-events: none;
    background: radial-gradient(ellipse at center,
      transparent 30%, rgba(0,0,0,0.6) 70%, rgba(0,0,0,0.95) 100%);
  }

  .film-noise {
    position: fixed; inset: 0; z-index: 3;
    pointer-events: none;
    background-image: repeating-linear-gradient(0deg,
      rgba(255,255,255,0.02) 0px,
      rgba(255,255,255,0.02) 1px,
      transparent 1px, transparent 3px);
    opacity: 0.4;
    animation: noiseMove 0.08s steps(2) infinite;
  }
  @keyframes noiseMove {
    0% { transform: translate(0,0); }
    50% { transform: translate(1px,1px); }
    100% { transform: translate(0,0); }
  }

  .flicker {
    position: fixed; inset: 0; z-index: 4;
    pointer-events: none;
    background: rgba(255,255,255,0.008);
    animation: flickAnim 0.15s infinite;
  }
  @keyframes flickAnim {
    0% { opacity: 0.4; } 50% { opacity: 0.7; } 100% { opacity: 0.5; }
  }

  /* Частицы */
  .particle {
    position: fixed;
    width: 2px; height: 2px;
    background: rgba(180,190,220,0.5);
    border-radius: 50%;
    pointer-events: none;
    z-index: 5;
    animation: particleFloat linear infinite;
  }
  @keyframes particleFloat {
    0% { transform: translateY(0); opacity: 0; }
    10% { opacity: 0.6; }
    90% { opacity: 0.6; }
    100% { transform: translateY(-100vh); opacity: 0; }
  }

  /* Проблеск силуэта */
  .shadow-flash {
    position: fixed;
    z-index: 6;
    width: 100px; height: 200px;
    background: radial-gradient(ellipse at center bottom,
      rgba(0,0,0,0.95) 0%, transparent 80%);
    border-radius: 50% 50% 20% 20%;
    filter: blur(8px);
    opacity: 0;
    pointer-events: none;
  }
  .shadow-flash.flash { animation: shadowFlash 0.5s forwards; }
  @keyframes shadowFlash {
    0% { opacity: 0; }
    40% { opacity: 0.7; }
    100% { opacity: 0; }
  }

  /* Сцена */
  .stage {
    position: relative;
    z-index: 10;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 40px 30px 80px;
    text-align: center;
  }

  /* ТЕКСТ — слоями */
  #text {
    max-width: 640px;
    width: 100%;
    font-size: 26px;
    line-height: 1.7;
    letter-spacing: 4px;
    font-weight: 300;
    color: #c8cce0;
    text-shadow:
      0 0 20px rgba(140,160,220,0.4),
      0 0 40px rgba(100,120,200,0.2);
    margin-bottom: 30px;
    /* Сдвиг чуть выше центра */
    margin-top: -40px;
  }

  .line {
    display: block;
    opacity: 0;
    transition: opacity 1.2s ease, transform 1.2s ease;
    transform: translateY(10px);
    margin-bottom: 14px;
    min-height: 1em;
  }

  .line.active {
    opacity: 1;
    transform: translateY(0);
  }

  .line.old {
    opacity: 0.35;
    transform: translateY(0);
    transition: opacity 2s ease;
    font-size: 24px;
  }

  .cursor {
    display: inline-block;
    width: 2px;
    height: 24px;
    background: #c8cce0;
    box-shadow: 0 0 12px rgba(180,190,220,0.8);
    animation: cursorPulse 1.4s ease-in-out infinite;
    vertical-align: middle;
    margin-left: 4px;
  }
  @keyframes cursorPulse {
    0%, 100% { opacity: 1; transform: scaleY(1); }
    50% { opacity: 0.3; transform: scaleY(0.85); }
  }

  /* Тонкая декоративная линия */
  .divider {
    width: 60px;
    height: 1px;
    background: linear-gradient(90deg,
      transparent 0%,
      rgba(180,190,220,0.4) 50%,
      transparent 100%);
    margin: 40px auto 0;
    opacity: 0;
    transition: opacity 2s;
  }
  .divider.visible { opacity: 1; }

  /* Кнопка */
  #start-btn {
    display: none;
    margin-top: 60px;
    background: transparent;
    border: 1px solid rgba(140,150,190,0.4);
    color: #c8cce0;
    font-family: 'Georgia', serif;
    font-size: 15px;
    letter-spacing: 8px;
    padding: 18px 55px;
    cursor: pointer;
    transition: all 0.6s;
    opacity: 0;
    text-shadow: 0 0 12px rgba(140,160,220,0.3);
  }
  #start-btn.visible {
    display: block;
    animation: btnAppear 3s forwards;
  }
  @keyframes btnAppear {
    0% { opacity: 0; transform: scale(0.95); letter-spacing: 14px; }
    100% { opacity: 1; transform: scale(1); letter-spacing: 8px; }
  }
  #start-btn:hover {
    border-color: #c8cce0;
    color: #fff;
    box-shadow:
      0 0 40px rgba(140,160,220,0.5),
      inset 0 0 20px rgba(140,160,220,0.1);
    letter-spacing: 12px;
  }

  .white-flash {
    position: fixed; inset: 0;
    background: #fff;
    z-index: 999;
    opacity: 0;
    pointer-events: none;
  }
  .white-flash.active { animation: whiteFlash 0.9s forwards; }
  @keyframes whiteFlash {
    0% { opacity: 0; }
    30% { opacity: 0.9; }
    100% { opacity: 0; }
  }

  #sound-btn {
    position: fixed;
    bottom: 20px; right: 20px;
    background: transparent;
    border: 1px solid rgba(140,150,190,0.3);
    color: rgba(180,190,220,0.5);
    font-family: 'Georgia', serif;
    font-size: 10px;
    letter-spacing: 3px;
    padding: 8px 14px;
    cursor: pointer;
    transition: all 0.3s;
    z-index: 20;
  }
  #sound-btn.on {
    border-color: rgba(140,190,140,0.5);
    color: rgba(140,190,140,0.8);
  }
</style>
</head>
<body>

<div class="bg-pulse"></div>
<div class="vignette"></div>
<div class="film-noise"></div>
<div class="flicker"></div>

<div class="stage">
  <div id="text"></div>
  <div class="divider" id="divider"></div>
  <button id="start-btn">НАЧАТЬ</button>
</div>

<button id="sound-btn">ЗВУК ВКЛ</button>
<div class="white-flash" id="whiteFlash"></div>

<script>
  const SOUNDS = {
    hum: 'hum1.mp3',
    whisper1: 'whisper1.mp3',
    whisper2: 'whisper2.mp3',
    click: 'click1.mp3',
    breath: 'breath.mp3',
    door_creak: 'door_creak.mp3'
  };

  const audio = {};
  Object.keys(SOUNDS).forEach(k => { audio[k] = new Audio(SOUNDS[k]); });
  audio.hum.loop = true;
  audio.hum.volume = 0.12;
  audio.whisper1.volume = 0.45;
  audio.whisper2.volume = 0.4;
  audio.click.volume = 0.2;
  audio.breath.volume = 0.3;
  audio.door_creak.volume = 0.3;

  let soundOn = false;
  const soundBtn = document.getElementById('sound-btn');

  soundBtn.addEventListener('click', () => {
    soundOn = !soundOn;
    if (soundOn) {
      audio.hum.play().catch(()=>{});
      soundBtn.textContent = 'ЗВУК ВЫКЛ';
      soundBtn.classList.add('on');
    } else {
      audio.hum.pause();
      soundBtn.textContent = 'ЗВУК ВКЛ';
      soundBtn.classList.remove('on');
    }
  });

  function play(name) {
    if (!soundOn || !audio[name]) return;
    audio[name].currentTime = 0;
    audio[name].play().catch(()=>{});
  }

  // ============ ЧАСТИЦЫ ============
  function spawnParticle() {
    const p = document.createElement('div');
    p.className = 'particle';
    p.style.left = Math.random() * 100 + '%';
    p.style.bottom = '-10px';
    p.style.animationDuration = (10 + Math.random() * 12) + 's';
    p.style.opacity = 0.2 + Math.random() * 0.5;
    document.body.appendChild(p);
    setTimeout(() => p.remove(), 22000);
  }
  setInterval(spawnParticle, 500);

  // ============ ПРОБЛЕСК СИЛУЭТА ============
  function flashSilhouette() {
    const s = document.createElement('div');
    s.className = 'shadow-flash';
    s.style.left = (10 + Math.random() * 80) + '%';
    s.style.top = (20 + Math.random() * 60) + '%';
    document.body.appendChild(s);
    requestAnimationFrame(() => s.classList.add('flash'));
    setTimeout(() => s.remove(), 600);
  }

  function scheduleFlash() {
    const delay = 8000 + Math.random() * 10000;
    setTimeout(() => {
      flashSilhouette();
      if (Math.random() < 0.5) play('whisper1');
      scheduleFlash();
    }, delay);
  }

  function scheduleWhisper() {
    const delay = 12000 + Math.random() * 15000;
    setTimeout(() => {
      play(Math.random() < 0.5 ? 'whisper1' : 'whisper2');
      scheduleWhisper();
    }, delay);
  }

  function scheduleAmbient() {
    const delay = 6000 + Math.random() * 8000;
    setTimeout(() => {
      if (Math.random() < 0.5) play('breath');
      else play('door_creak');
      scheduleAmbient();
    }, delay);
  }

  // ============ ТЕКСТ СЛОЯМИ ============
  const lines = [
    "Сеня.",
    "Ты здесь.",
    "Хорошо.",
    "Это игра.",
    "Но она не про игру.",
    "Пройди её. До конца.",
    "Я жду."
  ];

  const pauses = [2200, 1800, 2400, 1500, 2500, 2000, 2800];

  const textEl = document.getElementById('text');
  const startBtn = document.getElementById('start-btn');
  const divider = document.getElementById('divider');
  const whiteFlash = document.getElementById('whiteFlash');

  let idx = 0;

  function typeLine(text, lineEl) {
    let i = 0;
    const cursor = document.createElement('span');
    cursor.className = 'cursor';
    lineEl.appendChild(cursor);

    function step() {
      if (i < text.length) {
        cursor.before(text[i]);
        i++;
        if (Math.random() < 0.2) play('click');
        setTimeout(step, 75);
      } else {
        cursor.remove();
        nextLine();
      }
    }
    step();
  }

  function nextLine() {
    if (idx >= lines.length) {
      // Всё напечатано — показать кнопку
      setTimeout(() => {
        divider.classList.add('visible');
        startBtn.classList.add('visible');
        play('whisper2');
      }, 1500);
      return;
    }

    // Затемняем предыдущую строку
    const prev = textEl.lastElementChild;
    if (prev) prev.classList.add('old');

    // Создаём новую
    const lineEl = document.createElement('span');
    lineEl.className = 'line';
    textEl.appendChild(lineEl);

    setTimeout(() => {
      lineEl.classList.add('active');
      setTimeout(() => typeLine(lines[idx], lineEl), 300);
    }, 100);
  }

  startBtn.addEventListener('click', () => {
    play('whisper1');
    whiteFlash.classList.add('active');
    setTimeout(() => {
      window.location.href = 'scena2.html';
    }, 900);
  });

  window.addEventListener('load', () => {
    setTimeout(() => {
      const first = document.createElement('span');
      first.className = 'line';
      textEl.appendChild(first);
      first.classList.add('active');
      setTimeout(() => typeLine(lines[0], first), 300);
    }, 2000);

    setTimeout(scheduleFlash, 5000);
    setTimeout(scheduleWhisper, 7000);
    setTimeout(scheduleAmbient, 9000);
  });
</script>
</body>
</html>