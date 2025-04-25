<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Efek Ripple & Huruf Terbang</title>
  <style>
    * { margin:0; padding:0; box-sizing:border-box; }
    html, body { width:100%; height:100%; overflow:hidden; background:#000; font-family:sans-serif; color:#fff; }
    #intro {
      position:absolute; top:0; left:0; width:100%; height:100%;
      display:flex; flex-direction:column; justify-content:center; align-items:center;
      background:rgba(0,0,0,0.85); z-index:10;
      text-align:center;
    }
    #intro textarea {
      padding:10px 15px; font-size:18px; width:80%; max-width:400px; margin-bottom:15px;
      border:none; border-radius:5px; resize:none;
    }
    #intro button {
      padding:10px 20px; font-size:18px; cursor:pointer;
      border:none; border-radius:5px; background:#0af; color:#000;
    }
    #canvasContainer { display:none; position:relative; width:100%; height:100%; }
    canvas { display:block; width:100%; height:100%; }
    .ripple {
      position:absolute;
      border-radius:50%;
      background: rgba(255,255,255,0.5);
      width:20px; height:20px;
      transform: translate(-50%, -50%) scale(0);
      animation: ripple 0.6s ease-out;
      pointer-events:none;
    }
    @keyframes ripple {
      to {
        transform: translate(-50%, -50%) scale(15);
        opacity: 0;
      }
    }
  </style>
</head>
<body>

  <div id="intro">
    <h2>ketik kalimat /teks apa saja tetus sentuh layar :</h2>
    <textarea id="textInput" placeholder="Tulis paragrafmu di sini..." rows="4"></textarea>
    <button id="startBtn">Mulai</button>
  </div>

  <div id="canvasContainer">
    <canvas id="canvas"></canvas>
  </div>

  <script>
    const intro = document.getElementById('intro');
    const input = document.getElementById('textInput');
    const btn = document.getElementById('startBtn');
    const container = document.getElementById('canvasContainer');
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');

    let TEXT = '';
    const letters = [];
    const pointer = { x:-9999, y:-9999 };
    const FONT_SIZE = 36;
    const LINE_HEIGHT = FONT_SIZE * 1.4;

    function resize() {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
      initLetters();
    }
    window.addEventListener('resize', resize);

    function initLetters() {
      letters.length = 0;
      if (!TEXT) return;
      ctx.font = `${FONT_SIZE}px sans-serif`;
      const paras = TEXT.split('\n');
      const lines = [];
      paras.forEach((para, pIdx) => {
        const words = para.split(' ');
        let line = '';
        words.forEach(w => {
          const test = line + w + ' ';
          if (ctx.measureText(test).width > canvas.width * 0.9) {
            lines.push(line);
            line = w + ' ';
          } else {
            line = test;
          }
        });
        lines.push(line.trim());
        if (pIdx < paras.length - 1) lines.push('');
      });

      const totalHeight = lines.length * LINE_HEIGHT;
      const startY = (canvas.height - totalHeight) / 2;
      lines.forEach((ln, i) => {
        let x = (canvas.width - ctx.measureText(ln).width) / 2;
        const y = startY + i * LINE_HEIGHT;
        for (const ch of ln) {
          const w = ctx.measureText(ch).width;
          letters.push({ char: ch, x, y, ox: x, oy: y, vx: 0, vy: 0 });
          x += w;
        }
      });
    }

    function createRipple(x,y) {
      const r = document.createElement('div');
      r.className = 'ripple';
      r.style.left = x + 'px';
      r.style.top = y + 'px';
      container.appendChild(r);
      r.addEventListener('animationend', () => r.remove());
    }

    btn.addEventListener('click', () => {
      TEXT = input.value.trim() || 'HELLO SPCK EDITOR!';
      intro.style.display = 'none';
      container.style.display = 'block';
      resize();
      animate();
    });

    canvas.addEventListener('mousedown', e => {
      pointer.x = e.clientX; pointer.y = e.clientY;
      createRipple(e.clientX, e.clientY);
    });
    canvas.addEventListener('touchstart', e => {
      e.preventDefault();
      const t = e.touches[0];
      pointer.x = t.clientX; pointer.y = t.clientY;
      createRipple(t.clientX, t.clientY);
    }, { passive: false });
    canvas.addEventListener('mousemove', e => {
      pointer.x = e.clientX; pointer.y = e.clientY;
    });
    canvas.addEventListener('touchmove', e => {
      e.preventDefault();
      const t = e.touches[0];
      pointer.x = t.clientX; pointer.y = t.clientY;
    }, { passive: false });
    canvas.addEventListener('touchend', () => {
      pointer.x = pointer.y = -9999;
    });

    function animate() {
      if (!letters.length) return;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.font = `${FONT_SIZE}px sans-serif`;
      ctx.fillStyle = '#0ff';
      ctx.textBaseline = 'middle';

      for (const L of letters) {
        const dx = L.x - pointer.x;
        const dy = L.y - pointer.y;
        const dist = Math.hypot(dx, dy);
        const R = 100, FORCE = 4, ROT = 2, RETURN = 0.02;

        if (dist < R) {
          const ang = Math.atan2(dy, dx);
          const f = (R - dist) / R * FORCE;
          L.vx += Math.cos(ang) * f + (-Math.sin(ang) * ROT);
          L.vy += Math.sin(ang) * f + ( Math.cos(ang) * ROT);
        } else {
          L.vx += (L.ox - L.x) * RETURN;
          L.vy += (L.oy - L.y) * RETURN;
        }
        L.vx *= 0.9; L.vy *= 0.9;
        L.x += L.vx; L.y += L.vy;
        ctx.fillText(L.char, L.x, L.y);
      }
      requestAnimationFrame(animate);
    }
  </script>
</body>
</html>
