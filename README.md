<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>TIC TAC TOE ANGEL ROMERO</title>
  <style>
    :root{
      --bg:#0f172a;        /* slate-900 */
      --panel:#111827;     /* gray-900 */
      --muted:#94a3b8;     /* slate-400 */
      --text:#e5e7eb;      /* gray-200 */
      --accent:#22c55e;    /* green-500 */
      --accent-2:#38bdf8;  /* sky-400 */
      --danger:#ef4444;    /* red-500 */
      --cell:#1f2937;      /* gray-800 */
      --cell-hover:#2b3646;/* darker */
      --win:#16a34a;       /* green-600 */
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{
      margin:0; display:grid; place-items:center; min-height:100%;
      font-family: system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, Noto Sans, Helvetica, Arial, "Apple Color Emoji", "Segoe UI Emoji";
      background: radial-gradient(80rem 80rem at 10% -10%, #1e293b 0%, var(--bg) 55%);
      color:var(--text);
    }
    .app{width:min(980px, 92vw);}
    header{display:flex; gap:1rem; align-items:center; justify-content:space-between; margin-bottom:1rem}
    h1{font-size:clamp(1.1rem, 2.5vw + 1rem, 2rem); margin:0; letter-spacing:.4px}
    .panel{background:linear-gradient(180deg, rgba(255,255,255,.04), rgba(255,255,255,0)); border:1px solid rgba(255,255,255,.08); box-shadow: 0 10px 30px rgba(0,0,0,.35); border-radius:1.25rem; padding:1rem}

    /* Top controls */
    .controls{display:flex; flex-wrap:wrap; gap:.75rem; align-items:center}
    .select, button, .badge{
      background:var(--panel); color:var(--text); border:1px solid rgba(255,255,255,.09);
      border-radius:.9rem; padding:.6rem .9rem; font-weight:600; letter-spacing:.2px;
    }
    .select{display:flex; gap:.6rem; align-items:center}
    select{background:transparent; color:var(--text); border:none; outline:none; font:inherit}
    select option{background:#0b1220;}
    button{cursor:pointer; transition:transform .06s ease, background .2s ease;}
    button:hover{transform:translateY(-1px); background:#0b1220}
    .accent{border-color: rgba(34,197,94,.6)}

    /* Grid */
    .board-wrap{display:grid; grid-template-columns: 1fr 380px; gap:1rem}
    @media (max-width: 860px){.board-wrap{grid-template-columns:1fr}}

    .board{display:grid; grid-template-columns:repeat(3, 1fr); gap:.8rem; padding:1rem}
    .cell{
      aspect-ratio:1/1; display:grid; place-items:center; font-size:clamp(2.2rem, 8vw, 3.2rem);
      font-weight:800; border-radius:1.1rem; background:var(--cell); cursor:pointer;
      user-select:none; border:1px solid rgba(255,255,255,.06);
      transition:background .15s ease, box-shadow .15s ease, transform .06s ease;
    }
    .cell:hover{background:var(--cell-hover)}
    .cell:active{transform:scale(.98)}
    .cell.disabled{cursor:not-allowed; opacity:.6}

    .cell.x{color:var(--accent-2)}
    .cell.o{color:var(--accent)}
    .cell.win{outline:3px solid var(--win); box-shadow:0 0 0 2px rgba(22,163,74,.3) inset}

    /* Sidebar */
    .side{display:flex; flex-direction:column; gap:.8rem; padding:1rem}
    .status{font-weight:700; font-size:1.05rem}
    .sub{color:var(--muted); font-size:.95rem}
    .score{display:grid; grid-template-columns:repeat(3, 1fr); gap:.6rem}
    .score .card{background:var(--cell); border:1px solid rgba(255,255,255,.08); border-radius:1rem; padding:.8rem}
    .card h3{margin:.1rem 0; font-size:.95rem; color:var(--muted); font-weight:700}
    .card .big{font-size:1.8rem; font-weight:900; margin-top:.2rem}

    .row{display:flex; gap:.6rem; flex-wrap:wrap}
    .ghost{opacity:.7}
    .danger{border-color: rgba(239,68,68,.6)}
    .link{color:var(--accent-2); text-decoration:none}
    footer{margin-top:1rem; text-align:center; color:var(--muted); font-size:.9rem}
  </style>
</head>
<body>
  <main class="app">
    <header>
      <h1>🎮 Tic Tac Toe ANGEL ROMERO</h1>
      <div class="controls panel">
        <div class="select">
          <span>Modo:</span>
          <select id="mode">
            <option value="pvp">2 Jugadores</option>
            <option value="cpu">Vs Computadora (Imbatible)</option>
          </select>
        </div>
        <button id="newGameBtn" class="accent">🔄 Nueva partida</button>
        <button id="clearScoreBtn" class="danger">🧹 Reiniciar marcador</button>
      </div>
    </header>

    <section class="board-wrap">
      <div id="board" class="board panel" aria-label="Tablero Tres en Raya"></div>

      <aside class="side panel">
        <div class="status" id="status">Turno de <span id="turn">X</span></div>
        <div class="sub">Haz clic en una casilla vacía para jugar.</div>

        <div class="score">
          <div class="card">
            <h3>Jugador X</h3>
            <div class="big" id="scoreX">0</div>
          </div>
          <div class="card">
            <h3>Empates</h3>
            <div class="big" id="scoreT">0</div>
          </div>
          <div class="card">
            <h3>Jugador O</h3>
            <div class="big" id="scoreO">0</div>
          </div>
        </div>

        <div class="row">
          <button id="swapBtn" title="X empieza / O empieza">🔁 Cambiar jugador inicial</button>
          <button id="hintBtn" class="ghost" title="Sugerir mejor jugada">💡 Pista</button>
        </div>
      
      </aside>
    </section>

    <footer>
      Hecho por Angel Romero
    </footer>
  </main>

  <script>
    // --- Estado del juego ---
    const boardEl = document.getElementById('board');
    const statusEl = document.getElementById('status');
    const turnEl = document.getElementById('turn');
    const modeEl = document.getElementById('mode');
    const scoreEls = { X: document.getElementById('scoreX'), O: document.getElementById('scoreO'), T: document.getElementById('scoreT') };

    const newGameBtn = document.getElementById('newGameBtn');
    const clearScoreBtn = document.getElementById('clearScoreBtn');
    const swapBtn = document.getElementById('swapBtn');
    const hintBtn = document.getElementById('hintBtn');

    const LINES = [
      [0,1,2],[3,4,5],[6,7,8], // filas
      [0,3,6],[1,4,7],[2,5,8], // columnas
      [0,4,8],[2,4,6]          // diagonales
    ];

    let board, current, gameOver, starter = 'X';
    let score = { X: 0, O: 0, T: 0 };

    // Cargar marcador de localStorage (opcional)
    try {
      const saved = JSON.parse(localStorage.getItem('ttt_score'));
      if (saved) score = saved;
    } catch(err) {}

    // Render inicial
    function init(){
      board = Array(9).fill(null);
      current = starter;
      gameOver = false;
      renderBoard();
      updateStatus();
      updateScore();
      // Si el modo es CPU y empieza la CPU (starter==='O'), que juegue
      maybeCpuMove();
    }

    function renderBoard(){
      boardEl.innerHTML='';
      for(let i=0;i<9;i++){
        const btn = document.createElement('button');
        btn.className = 'cell';
        btn.setAttribute('aria-label', `Casilla ${i+1}`);
        btn.addEventListener('click', () => playAt(i));
        boardEl.appendChild(btn);
      }
      paint();
    }

    function paint(winLine){
      [...boardEl.children].forEach((cell,i)=>{
        const v = board[i];
        cell.textContent = v ? v : '';
        cell.classList.toggle('x', v==='X');
        cell.classList.toggle('o', v==='O');
        cell.classList.toggle('disabled', !!v || gameOver);
        cell.classList.toggle('win', winLine?.includes(i));
      });
    }

    function updateStatus(msg){
      if (msg){ statusEl.textContent = msg; return; }
      if (gameOver){ return; }
      statusEl.innerHTML = `Turno de <span id="turn">${current}</span>`;
    }

    function updateScore(){
      scoreEls.X.textContent = score.X;
      scoreEls.O.textContent = score.O;
      scoreEls.T.textContent = score.T;
      try { localStorage.setItem('ttt_score', JSON.stringify(score)); } catch(err) {}
    }

    function playAt(i){
      if (gameOver || board[i]) return;
      board[i] = current;
      const win = getWin(board);
      if (win){
        paint(win.line);
        gameOver = true;
        score[current]++;
        updateScore();
        updateStatus(`🎉 ¡${current} gana!`);
        return;
      }
      if (isDraw(board)){
        paint();
        gameOver = true; score.T++; updateScore();
        updateStatus('🤝 ¡Empate!');
        return;
      }
      // Cambiar turno
      current = current === 'X' ? 'O' : 'X';
      paint();
      updateStatus();
      maybeCpuMove();
    }

    function maybeCpuMove(){
      if (gameOver) return;
      const cpu = 'O';
      if (modeEl.value === 'cpu' && current === cpu){
        // Pequeño delay para naturalidad
        setTimeout(()=>{
          const move = bestMove(board, cpu);
          if (move != null) playAt(move);
        }, 220);
      }
    }

    // --- Utilidades de juego ---
    function getWin(b){
      for (const line of LINES){
        const [a,b2,c] = line;
        if (b[a] && b[a]===b[b2] && b[a]===b[c]){
          return { player: b[a], line };
        }
      }
      return null;
    }
    function isDraw(b){ return b.every(Boolean) && !getWin(b); }

    // IA: Minimax con poda simple
    function bestMove(b, ai){
      const human = ai === 'X' ? 'O' : 'X';
      let best = { idx: null, score: -Infinity };
      for (let i=0;i<9;i++){
        if (!b[i]){
          b[i] = ai;
          const score = minimax(b, 0, false, ai, human, -Infinity, Infinity);
          b[i] = null;
          if (score > best.score){ best = { idx:i, score }; }
        }
      }
      return best.idx;
    }
    function minimax(b, depth, isMax, ai, human, alpha, beta){
      const win = getWin(b);
      if (win){
        if (win.player === ai) return 10 - depth;
        if (win.player === human) return depth - 10;
      }
      if (isDraw(b)) return 0;

      if (isMax){
        let best = -Infinity;
        for (let i=0;i<9;i++) if (!b[i]){
          b[i] = ai;
          best = Math.max(best, minimax(b, depth+1, false, ai, human, alpha, beta));
          b[i] = null;
          alpha = Math.max(alpha, best);
          if (beta <= alpha) break; // poda beta
        }
        return best;
      } else {
        let best = Infinity;
        for (let i=0;i<9;i++) if (!b[i]){
          b[i] = human;
          best = Math.min(best, minimax(b, depth+1, true, ai, human, alpha, beta));
          b[i] = null;
          beta = Math.min(beta, best);
          if (beta <= alpha) break; // poda alpha
        }
        return best;
      }
    }

    // --- Controles ---
    newGameBtn.addEventListener('click', () => init());
    clearScoreBtn.addEventListener('click', () => {
      if (confirm('¿Reiniciar marcador?')){ score = { X:0, O:0, T:0 }; updateScore(); }
    });
    swapBtn.addEventListener('click', () => {
      starter = starter === 'X' ? 'O' : 'X';
      init();
    });
    modeEl.addEventListener('change', () => init());

    hintBtn.addEventListener('click', () => {
      if (gameOver) return;
      const player = current;
      const move = bestMove([...board], player);
      if (move == null) return;
      // Efecto de "pista" en la celda sugerida
      const cell = boardEl.children[move];
      cell.animate([
        { boxShadow: '0 0 0 0 rgba(56,189,248,0.0)' },
        { boxShadow: '0 0 0 10px rgba(56,189,248,0.35)' },
        { boxShadow: '0 0 0 0 rgba(56,189,248,0.0)' }
      ], { duration: 600, easing: 'ease' });
    });

    // Iniciar
    updateScore();
    init();
  </script>
</body>
</html>
