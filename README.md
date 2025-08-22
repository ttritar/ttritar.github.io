<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Mini Wordle — 4 Letters (Answer: kobe)</title>
  <style>
    :root {
      --bg: #0f172a;         /* slate-900 */
      --tile: #111827;       /* gray-900 */
      --text: #e5e7eb;       /* gray-200 */
      --muted: #9ca3af;      /* gray-400 */
      --kbd: #374151;        /* gray-700 */
      --ok: #22c55e;         /* green-500 */
      --present: #eab308;    /* yellow-500 */
      --miss: #4b5563;       /* gray-600 */
      --accent: #60a5fa;     /* blue-400 */
      --error: #f87171;      /* red-400 */
    }
    * { box-sizing: border-box; }
    body {
      margin: 0; font-family: system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, Noto Sans, Helvetica, Arial, "Apple Color Emoji", "Segoe UI Emoji";
      background: radial-gradient(1200px 800px at 50% -10%, #1f2937 0%, var(--bg) 45%);
      color: var(--text);
      display: grid; place-items: center; min-height: 100vh; padding: 24px;
    }
    .app { width: 100%; max-width: 560px; display: grid; gap: 18px; }
    header { display:flex; justify-content: space-between; align-items:center; }
    h1 { font-size: clamp(20px, 2.8vw, 28px); letter-spacing: 2px; margin: 0; font-weight: 800; }
    .sub { color: var(--muted); font-size: 12px; }

    .board { display: grid; gap: 10px; }
    .row { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; }
    .tile {
      aspect-ratio: 1/1; border-radius: 12px; background: var(--tile); border: 2px solid #1f2937;
      display: grid; place-items: center; font-weight: 800; font-size: 28px; text-transform: uppercase;
      transition: transform .08s ease, background .25s ease, border-color .25s ease;
    }
    .tile.filled { border-color: #374151; }
    .tile.reveal { animation: flip .6s ease forwards; }
    .tile.ok { background: var(--ok); border-color: var(--ok); color: #052e13; }
    .tile.present { background: var(--present); border-color: var(--present); color: #3a2b04; }
    .tile.miss { background: var(--miss); border-color: var(--miss); color: #0b1220; }

    @keyframes flip {
      0% { transform: rotateX(0); }
      49% { transform: rotateX(90deg); }
      50% { transform: rotateX(90deg); }
      100% { transform: rotateX(0); }
    }

    .kbd { display:grid; gap: 8px; user-select: none; }
    .row-kbd { display: grid; grid-template-columns: repeat(20, minmax(0,1fr)); gap: 8px; }
    .key {
      grid-column: span 2; height: 48px; border-radius: 10px; background: var(--kbd); border: 1px solid #2b333d; color: var(--text);
      display:grid; place-items:center; font-weight: 700; cursor: pointer; transition: transform .04s ease, background .15s ease;
    }
    .key:active { transform: translateY(1px) scale(0.99); }
    .key.wide { grid-column: span 4; font-size: 0.95rem; }
    .key.ok { background: var(--ok); color: #052e13; }
    .key.present { background: var(--present); color: #3a2b04; }
    .key.miss { background: var(--miss); color: #0b1220; }

    .bar { display:flex; justify-content: space-between; align-items:center; gap: 12px; }
    .msg { min-height: 24px; font-size: 14px; color: var(--muted); }
    .controls { display:flex; gap: 8px; }
    button { background: #1f2937; color: var(--text); border: 1px solid #2b333d; border-radius: 999px; padding: 8px 14px; font-weight: 700; cursor: pointer; }
    button:hover { border-color: var(--accent); box-shadow: 0 0 0 2px #1e3a8a33 inset; }

    footer { text-align:center; font-size: 12px; color: var(--muted); }
    a { color: var(--accent); text-decoration: none; }
  </style>
</head>
<body>
  <div class="app" role="application" aria-label="Mini Wordle 4-letter">
    <header>
      <div>
        <h1>Mini Wordle</h1>
        <div class="sub">4 letters · 6 tries · </div>
      </div>
      <div class="controls">
        <button id="resetBtn" aria-label="Reset game">Reset</button>
      </div>
    </header>

    <main>
      <div id="board" class="board" aria-live="polite"></div>
      <div class="bar">
        <div id="msg" class="msg">Type or use the keyboard below.</div>
      </div>

      <div class="kbd" aria-label="On-screen keyboard">
        <div class="row-kbd" id="kbd1"></div>
        <div class="row-kbd" id="kbd2"></div>
        <div class="row-kbd" id="kbd3"></div>
      </div>
    </main>

    <footer>
      Built for fun. This version only ever answers <strong>kobe</strong>.
    </footer>
  </div>

  <script>
    (function(){
      const ANSWER = "kobe"; // lower-case
      const MAX_ROWS = 6, COLS = 4;

      const boardEl = document.getElementById('board');
      const msgEl = document.getElementById('msg');
      const resetBtn = document.getElementById('resetBtn');

      let state = { row: 0, col: 0, grid: Array.from({length: MAX_ROWS}, () => Array(COLS).fill('')), status: 'playing' };

      // Build board
      function buildBoard(){
        boardEl.innerHTML = '';
        for(let r=0;r<MAX_ROWS;r++){
          const row = document.createElement('div'); row.className = 'row';
          for(let c=0;c<COLS;c++){
            const tile = document.createElement('div'); tile.className = 'tile'; tile.setAttribute('data-pos', `${r}-${c}`);
            tile.setAttribute('aria-label', `Row ${r+1} column ${c+1}`);
            row.appendChild(tile);
          }
          boardEl.appendChild(row);
        }
      }

      function setMsg(t, tone='info'){
        msgEl.textContent = t;
        msgEl.style.color = tone==='error' ? 'var(--error)' : 'var(--muted)';
      }

      function render(){
        for(let r=0;r<MAX_ROWS;r++){
          for(let c=0;c<COLS;c++){
            const tile = boardEl.querySelector(`[data-pos="${r}-${c}"]`);
            const ch = state.grid[r][c];
            tile.textContent = ch.toUpperCase();
            tile.classList.toggle('filled', !!ch);
          }
        }
      }

      function evaluate(guess){
        const g = guess.toLowerCase();
        const a = ANSWER;
        // mark correctness with simple frequency handling
        const res = Array(COLS).fill('miss');
        const freq = {};
        for(const ch of a){ freq[ch] = (freq[ch]||0)+1; }
        // first pass: exact
        for(let i=0;i<COLS;i++){
          if(g[i] === a[i]){ res[i] = 'ok'; freq[g[i]]--; }
        }
        // second pass: present
        for(let i=0;i<COLS;i++){
          if(res[i] === 'ok') continue;
          const ch = g[i];
          if(freq[ch] > 0){ res[i] = 'present'; freq[ch]--; }
        }
        return res;
      }

      function revealRow(r, res){
        for(let c=0;c<COLS;c++){
          const tile = boardEl.querySelector(`[data-pos="${r}-${c}"]`);
          tile.classList.add('reveal');
          tile.addEventListener('animationend', () => tile.classList.remove('reveal'), { once: true });
          tile.classList.remove('ok','present','miss');
          tile.classList.add(res[c]);
        }
      }

      function updateKeyboard(res, guess){
        const keys = document.querySelectorAll('.key');
        const best = { miss: 0, present: 1, ok: 2 };
        for(let i=0;i<guess.length;i++){
          const ch = guess[i].toUpperCase();
          const status = res[i];
          const key = Array.from(keys).find(k => k.textContent === ch);
          if(!key) continue;
          const prev = key.dataset.status || 'none';
          if(prev==='ok') continue; // never downgrade
          if(prev==='present' && status==='miss') continue;
          key.dataset.status = status;
          key.classList.remove('ok','present','miss');
          key.classList.add(status);
        }
      }

      function submit(){
        if(state.status !== 'playing') return;
        if(state.col < COLS){ setMsg('Not enough letters', 'error'); return; }
        const guess = state.grid[state.row].join('');
        if(!/^[a-zA-Z]{4}$/.test(guess)){ setMsg('Letters only.', 'error'); return; }

        const res = evaluate(guess);
        revealRow(state.row, res);
        updateKeyboard(res, guess);

        if(guess.toLowerCase() === ANSWER){
          state.status = 'won';
          setMsg(`Nice! ${guess.toUpperCase()} is correct.`, 'info');
        } else if(state.row === MAX_ROWS - 1){
          state.status = 'lost';
          setMsg(`Out of tries. The answer was ${ANSWER.toUpperCase()}.`);
        } else {
          state.row++; state.col = 0;
          setMsg('');
        }
      }

      function press(ch){
        if(state.status !== 'playing') return;
        if(/^[a-zA-Z]$/.test(ch)){
          if(state.col < COLS){
            state.grid[state.row][state.col] = ch.toLowerCase();
            state.col++;
            render();
          }
        }
      }

      function backspace(){
        if(state.status !== 'playing') return;
        if(state.col>0){ state.col--; state.grid[state.row][state.col] = ''; render(); }
      }

      // Keyboard UI
      const layout = [
        'QWERTYUIOP',
        'ASDFGHJKL',
        'ZXCVBNM'
      ];
      function buildKbd(){
        const rows = [document.getElementById('kbd1'), document.getElementById('kbd2'), document.getElementById('kbd3')];
        rows.forEach(r => r.innerHTML='');
        layout.forEach((row, i) => {
          const container = rows[i];
          const spanStart = i===2 ? 2 : 1; // slight centering on last row
          for(const ch of row){
            const key = document.createElement('button');
            key.className = 'key';
            key.textContent = ch;
            key.type = 'button';
            key.addEventListener('click', () => press(ch));
            container.appendChild(key);
          }
          if(i===2){
            const enter = document.createElement('button'); enter.className='key wide'; enter.textContent='ENTER'; enter.type='button'; enter.addEventListener('click', submit);
            const back = document.createElement('button'); back.className='key wide'; back.textContent='⌫'; back.type='button'; back.addEventListener('click', backspace);
            container.prepend(enter); container.appendChild(back);
          }
        });
      }

      // Events
      window.addEventListener('keydown', (e) => {
        const key = e.key;
        if(key === 'Enter') submit();
        else if(key === 'Backspace') backspace();
        else if(/^[a-zA-Z]$/.test(key)) press(key.toUpperCase());
      });

      resetBtn.addEventListener('click', () => {
        state = { row: 0, col: 0, grid: Array.from({length: MAX_ROWS}, () => Array(COLS).fill('')), status: 'playing' };
        buildBoard(); render(); buildKbd(); setMsg('New game — good luck!');
        // Clear keyboard colors
        document.querySelectorAll('.key').forEach(k => { k.dataset.status='none'; k.classList.remove('ok','present','miss'); });
      });

      // Init
      buildBoard(); render(); buildKbd();
    })();
  </script>
</body>
</html>
