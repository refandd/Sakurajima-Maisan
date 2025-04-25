<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Huruf Berbaris dan Terbang</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    html, body { width: 100%; height: 100%; background: #000; overflow: hidden; }
    canvas { display: block; }
  </style>
</head>
<body>
  <canvas id="canvas"></canvas>
  <script>
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');

    const TEXT = "halo Sakurajima Mai cintakuuu sayangkuuuu duniaaaaakuuuu ayangkuuu ceeeweweeeku tercintsaas istrikuuuu alam semesta ku oksigen kuuu jiwaakuuu hidupkuuu iloveyou Sakurajima Mai aku sayang kamuuuu Sakurajima Mai sukidaaaaaaaaaaa!";
    const FONT_SIZE = 36;
    const LINE_HEIGHT = FONT_SIZE * 1.5;
    const MAX_WIDTH = () => canvas.width * 0.9;

    const letters = [];
    const pointer = { x: -9999, y: -9999 };
    function 
     resize() {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
      initLetters();
    }

    window.addEventListener('resize', resize);
    resize();

    function initLetters() {
      letters.length = 0;
      ctx.font = `${FONT_SIZE}px sans-serif`;
      const words = TEXT.split(" ");
      let line = "", lines = [];
      
      for (let word of words) {
        const testLine = line + word + " ";
        const testWidth = ctx.measureText(testLine).width;
        if (testWidth > MAX_WIDTH()) {
          lines.push(line);
          line = word + " ";
        } else {
          line = testLine;
        }
      }
      lines.push(line);

      const startY = (canvas.height - lines.length * LINE_HEIGHT) / 2;

      for (let i = 0; i < lines.length; i++) {
        let x = (canvas.width - ctx.measureText(lines[i]).width) / 2;
        let y = startY + i * LINE_HEIGHT;

        for (let ch of lines[i]) {
          const w = ctx.measureText(ch).width;
          letters.push({
            char: ch,
            x: x,
            y: y,
            ox: x,
            oy: y,
            vx: 0,
            vy: 0
          });
          x += w;
        }
      }
    }

    canvas.addEventListener('mousemove', e => {
      pointer.x = e.clientX;
      pointer.y = e.clientY;
    });

    canvas.addEventListener('touchmove', e => {
      e.preventDefault();
      pointer.x = e.touches[0].clientX;
      pointer.y = e.touches[0].clientY;
    }, { passive: false });

    canvas.addEventListener('touchstart', e => {
      e.preventDefault();
      pointer.x = e.touches[0].clientX;
      pointer.y = e.touches[0].clientY;
    }, { passive: false });

    canvas.addEventListener('touchend', () => {
      pointer.x = -9999;
      pointer.y = -9999;
    });

    function animate() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.font = `${FONT_SIZE}px sans-serif`;
      ctx.fillStyle = '#0ff';
      ctx.textBaseline = 'middle';

      for (let L of letters) {
        let dx = L.x - pointer.x;
        let dy = L.y - pointer.y;
        let dist = Math.hypot(dx, dy);
        const R = 100;
        const FORCE = 4;
        const ROT = 2;
        const RETURN = 0.01;

        if (dist < R) {
          let ang = Math.atan2(dy, dx);
          let f = (R - dist) / R * FORCE;
          L.vx += Math.cos(ang) * f + (-Math.sin(ang) * ROT);
          L.vy += Math.sin(ang) * f + ( Math.cos(ang) * ROT);
        } else {
          L.vx += (L.ox - L.x) * RETURN;
          L.vy += (L.oy - L.y) * RETURN;
        }

        L.vx *= 0.9;
        L.vy *= 0.9;
        L.x += L.vx;
        L.y += L.vy;

        ctx.fillText(L.char, L.x, L.y);
      }

      requestAnimationFrame(animate);
    }

    animate();
  </script>
</body>
</html>

