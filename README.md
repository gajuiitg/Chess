<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Chess · one‑player vs AI</title>
  <style>
    * {
      box-sizing: border-box;
      font-family: 'Segoe UI', Roboto, system-ui, sans-serif;
    }
    html, body {
      overflow-x: hidden;
      width: 100%;
    }
    body {
      background: linear-gradient(145deg, #2b5a4c 0%, #1d3e34 100%);
      min-height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
      margin: 0;
      padding: 12px;
    }
    .game-container {
      background: #e8d5b5;
      padding: clamp(1rem, 4vw, 2rem) clamp(1rem, 5vw, 2.5rem) clamp(1.25rem, 5vw, 2.5rem);
      border-radius: 60px 60px 40px 40px;
      box-shadow: 0 30px 40px rgba(0,0,0,0.7);
      display: flex;
      flex-direction: column;
      align-items: center;
      border: 6px solid #b39264;
      max-width: 100%;
      width: fit-content;
    }
    h1 {
      margin: 0 0 12px 0;
      font-weight: 400;
      letter-spacing: 4px;
      color: #f7f0e0;
      text-shadow: 3px 3px 0 #5a3f2b;
      font-size: 2.1rem;
      background: #3d2c1e;
      padding: 0 30px;
      border-radius: 60px;
      box-shadow: inset 0 -3px 0 #b39264;
    }
    .board-wrapper {
      background: #b39264;
      padding: clamp(8px, 2.5vw, 18px);
      border-radius: 32px;
      box-shadow: inset 0 0 0 2px #e8d5b5, 0 16px 28px rgba(0,0,0,0.6);
      max-width: 100%;
    }
    #chess-board {
      display: grid;
      grid-template-columns: repeat(8, minmax(0, 1fr));
      grid-template-rows: repeat(8, minmax(0, 1fr));
      /* board = viewport minus body padding, container border/padding, wrapper padding */
      width: min(560px, calc(100vw - 24px - 12px - 2 * clamp(1rem, 5vw, 2.5rem) - 2 * clamp(8px, 2.5vw, 18px)));
      aspect-ratio: 1 / 1;
      border: 4px solid #3d2c1e;
      border-radius: 8px;
      overflow: hidden;
      background: #ebd0b0;
    }
    .square {
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: clamp(1.1rem, 6.5vw, 3.1rem);
      min-width: 0;
      min-height: 0;
      overflow: hidden;
      font-weight: 500;
      text-shadow: 2px 2px 4px rgba(0,0,0,0.4);
      cursor: pointer;
      transition: background 0.1s ease;
      user-select: none;
      aspect-ratio: 1 / 1;
    }
    .square.light { background: #f0d9b5; }
    .square.dark { background: #b58863; }
    .square.selected { background: #7fc97f !important; box-shadow: inset 0 0 0 3px #2b6e2b; }
    .square.valid-move { background: #b1d0b1 !important; box-shadow: inset 0 0 0 3px #2b6e2b; }
    .square.last-move { background: #d9b650 !important; }
    .status-area {
      display: flex;
      justify-content: space-between;
      align-items: center;
      width: 100%;
      margin-top: 20px;
      gap: 20px;
      flex-wrap: wrap;
    }
    .turn-indicator {
      padding: 10px 28px;
      border-radius: 40px;
      font-size: 1.3rem;
      font-weight: 600;
      letter-spacing: 1px;
      transition: background 0.25s ease, color 0.25s ease, box-shadow 0.25s ease;
      border: 2px solid transparent;
    }
    .turn-indicator.turn-white {
      background: #f7f0e0;
      color: #2b1f14;
      box-shadow: inset 0 -4px 0 #d9b650;
      border-color: #b39264;
    }
    .turn-indicator.turn-black {
      background: #1d2f26;
      color: #f0e3d0;
      box-shadow: inset 0 -4px 0 #3d6e52;
      border-color: #0f1a14;
    }
    .turn-indicator.turn-thinking {
      background: #4a2f6e;
      color: #f0e3d0;
      box-shadow: inset 0 -4px 0 #8a5fd9;
      border-color: #2e1d4a;
    }
    .turn-indicator.turn-over {
      background: #6e3d2b;
      color: #f7f0e0;
      box-shadow: inset 0 -4px 0 #d9884f;
      border-color: #4a2a1d;
    }
    .btn-group {
      display: flex;
      gap: 12px;
      flex-wrap: wrap;
    }
    .btn {
      background: #b39264;
      border: none;
      color: #1d2f26;
      font-size: 1.1rem;
      font-weight: 600;
      padding: 10px 22px;
      border-radius: 40px;
      cursor: pointer;
      box-shadow: 0 6px 0 #5a3f2b;
      transition: all 0.07s;
      letter-spacing: 0.5px;
    }
    .btn:active {
      transform: translateY(6px);
      box-shadow: none;
    }
    .btn.active-mode {
      background: #d9b650;
      box-shadow: 0 6px 0 #7a6420;
      color: #1d2f26;
    }
    .difficulty-row {
      display: flex;
      align-items: center;
      gap: 12px;
      flex-wrap: wrap;
      width: 100%;
      margin-top: 16px;
    }
    .diff-label {
      font-weight: 600;
      color: #3d2c1e;
      letter-spacing: 0.5px;
      font-size: 1rem;
    }
    .diff-btn {
      padding: 7px 18px;
      font-size: 0.95rem;
      box-shadow: 0 4px 0 #5a3f2b;
    }
    .diff-btn:active { transform: translateY(4px); box-shadow: none; }
    .diff-btn.diff-active {
      background: #7fc97f;
      box-shadow: 0 4px 0 #2b6e2b;
      color: #12240f;
    }
    .help-box {
      background: #f7f0e0;
      border: 4px solid #3d2c1e;
      border-radius: 30px;
      padding: 0 24px 16px 24px;
      margin-top: 24px;
      max-width: 700px;
      width: 100%;
      box-shadow: inset 0 0 0 2px #b39264, 0 8px 12px rgba(0,0,0,0.3);
    }
    .help-box h3 {
      background: #3d2c1e;
      color: #f0e3d0;
      margin: 0 -24px 10px -24px;
      padding: 12px 0;
      text-align: center;
      border-radius: 30px 30px 0 0;
      font-weight: 400;
      letter-spacing: 2px;
      font-size: 1.3rem;
    }
    .help-box ul {
      columns: 2;
      column-gap: 30px;
      margin: 6px 0 8px 0;
      padding-left: 22px;
      list-style-type: square;
      color: #1d2f26;
    }
    .help-box li { margin-bottom: 5px; font-size: 0.95rem; break-inside: avoid; }
    .help-box .note {
      font-weight: 500;
      background: #dac09a;
      padding: 6px 12px;
      border-radius: 60px;
      display: inline-block;
      margin-top: 4px;
    }
    @media (max-width: 700px) {
      .help-box ul { columns: 1; }
    }
    @media (max-width: 420px) {
      h1 { font-size: 1.5rem; letter-spacing: 2px; }
      .turn-indicator { font-size: 1rem; padding: 8px 16px; }
      .btn { font-size: 0.95rem; padding: 8px 16px; }
    }
  </style>
</head>
<body>
<div class="game-container">
  <h1>♚ CHESS</h1>
  <div class="board-wrapper">
    <div id="chess-board"></div>
  </div>
  <div class="status-area">
    <div class="turn-indicator turn-white" id="turnIndicator">White's turn</div>
    <div class="btn-group">
      <button class="btn active-mode" id="modeBtn">🤖 vs Computer</button>
      <button class="btn" id="resetBtn">↺ New game</button>
    </div>
  </div>
  <div class="difficulty-row" id="difficultyRow">
    <span class="diff-label">Difficulty:</span>
    <div class="btn-group">
      <button class="btn diff-btn" data-level="easy">Easy</button>
      <button class="btn diff-btn" data-level="medium">Medium</button>
      <button class="btn diff-btn diff-active" data-level="hard">Hard</button>
      <button class="btn diff-btn" data-level="hardest">Hardest</button>
    </div>
  </div>
  <div class="help-box">
    <h3>♛ Rules of Chess</h3>
    <ul>
      <li><strong>Goal:</strong> Checkmate the opponent's king.</li>
      <li><strong>White moves first</strong> – alternate turns.</li>
      <li><strong>Pawn:</strong> forward 1 (or 2 from start), capture diagonally. Promotes on last rank.</li>
      <li><strong>Knight:</strong> L‑shape (2+1). Jumps over pieces.</li>
      <li><strong>Bishop:</strong> diagonal any distance.</li>
      <li><strong>Rook:</strong> horizontal / vertical any distance.</li>
      <li><strong>Queen:</strong> bishop + rook combined.</li>
      <li><strong>King:</strong> one square any direction. Cannot move into check.</li>
      <li><strong>Castling:</strong> king + rook (neither moved, no pieces between, king not in/through check).</li>
      <li><strong>En passant:</strong> if pawn moves 2 squares from start, enemy pawn may capture as if it moved 1.</li>
      <li><strong>Pawn promotion:</strong> automatically becomes Queen.</li>
      <li><strong>Check / Checkmate</strong> – game ends on checkmate.</li>
    </ul>
    <div class="note">♔ Two‑player / vs Computer · click piece, then target</div>
  </div>
</div>
<script>
  (function() {
    // ---------- core ----------
    let board = [];
    let turn = 'white';
    let selected = null;
    let validMoves = [];
    let moveHistory = [];
    let gameOver = false;
    let lastMove = null;
    let aiEnabled = true;         // true = vs computer, false = two-player
    let aiThinking = false;

    const boardEl = document.getElementById('chess-board');
    const turnIndicator = document.getElementById('turnIndicator');
    const modeBtn = document.getElementById('modeBtn');

    const PIECES = {
      'white': { 'king': '♔', 'queen': '♕', 'rook': '♖', 'bishop': '♗', 'knight': '♘', 'pawn': '♙' },
      'black': { 'king': '♚', 'queen': '♛', 'rook': '♜', 'bishop': '♝', 'knight': '♞', 'pawn': '♟' }
    };

    function initBoard() {
      const backRank = ['rook', 'knight', 'bishop', 'queen', 'king', 'bishop', 'knight', 'rook'];
      board = Array(8).fill().map(() => Array(8).fill(null));
      for (let c = 0; c < 8; c++) {
        board[0][c] = { type: backRank[c], color: 'black', hasMoved: false };
        board[1][c] = { type: 'pawn', color: 'black', hasMoved: false };
        board[6][c] = { type: 'pawn', color: 'white', hasMoved: false };
        board[7][c] = { type: backRank[c], color: 'white', hasMoved: false };
      }
      turn = 'white';
      selected = null;
      validMoves = [];
      moveHistory = [];
      gameOver = false;
      lastMove = null;
      aiThinking = false;
    }

    function cloneBoard(b) { return b.map(row => row.map(cell => cell ? { ...cell } : null)); }

    function findKing(b, color) {
      for (let r = 0; r < 8; r++) for (let c = 0; c < 8; c++) {
        const p = b[r][c];
        if (p && p.type === 'king' && p.color === color) return { row: r, col: c };
      }
      return null;
    }

    function isInCheck(b, color) {
      const king = findKing(b, color);
      if (!king) return true;
      const enemy = color === 'white' ? 'black' : 'white';
      for (let r = 0; r < 8; r++) for (let c = 0; c < 8; c++) {
        const p = b[r][c];
        if (p && p.color === enemy) {
          // forAttackCheck=true so we never regenerate castling moves here —
          // otherwise this would recurse forever into the enemy king's move-gen.
          const moves = getPseudoLegalMoves(b, r, c, true);
          for (let m of moves) if (m.row === king.row && m.col === king.col) return true;
        }
      }
      return false;
    }

    function getPseudoLegalMoves(b, row, col, forAttackCheck) {
      const piece = b[row][col];
      if (!piece) return [];
      const moves = [];
      const { type, color } = piece;
      const enemy = color === 'white' ? 'black' : 'white';
      const dir = color === 'white' ? -1 : 1;
      const startRow = color === 'white' ? 6 : 1;
      const addSliding = (dr, dc) => {
        for (let i = 1; i < 8; i++) {
          const nr = row + dr * i, nc = col + dc * i;
          if (nr < 0 || nr > 7 || nc < 0 || nc > 7) break;
          if (b[nr][nc]) {
            if (b[nr][nc].color === enemy) moves.push({ row: nr, col: nc });
            break;
          }
          moves.push({ row: nr, col: nc });
        }
      };

      switch(type) {
        case 'pawn': {
          const nr = row + dir;
          if (nr >= 0 && nr < 8 && !b[nr][col]) {
            moves.push({ row: nr, col });
            if (row === startRow && !b[row + 2*dir][col]) moves.push({ row: row + 2*dir, col });
          }
          for (let dc of [-1, 1]) {
            const nc = col + dc;
            if (nr >= 0 && nr < 8 && nc >= 0 && nc < 8) {
              if (b[nr][nc] && b[nr][nc].color === enemy) moves.push({ row: nr, col: nc });
              if (moveHistory.length > 0) {
                const last = moveHistory[moveHistory.length-1];
                if (last && last.pieceType === 'pawn' && Math.abs(last.fromRow - last.toRow) === 2) {
                  if (last.toRow === row && last.toCol === nc) {
                    moves.push({ row: nr, col: nc, enPassant: true });
                  }
                }
              }
            }
          }
          break;
        }
        case 'knight': {
          const jumps = [[-2,-1],[-2,1],[-1,-2],[-1,2],[1,-2],[1,2],[2,-1],[2,1]];
          for (let [dr, dc] of jumps) {
            const nr = row+dr, nc = col+dc;
            if (nr>=0 && nr<8 && nc>=0 && nc<8 && (!b[nr][nc] || b[nr][nc].color === enemy)) {
              moves.push({ row: nr, col: nc });
            }
          }
          break;
        }
        case 'bishop': addSliding(1,1); addSliding(1,-1); addSliding(-1,1); addSliding(-1,-1); break;
        case 'rook': addSliding(1,0); addSliding(-1,0); addSliding(0,1); addSliding(0,-1); break;
        case 'queen': addSliding(1,1); addSliding(1,-1); addSliding(-1,1); addSliding(-1,-1); addSliding(1,0); addSliding(-1,0); addSliding(0,1); addSliding(0,-1); break;
        case 'king': {
          for (let dr of [-1,0,1]) for (let dc of [-1,0,1]) {
            if (dr===0 && dc===0) continue;
            const nr=row+dr, nc=col+dc;
            if (nr>=0 && nr<8 && nc>=0 && nc<8 && (!b[nr][nc] || b[nr][nc].color === enemy)) {
              moves.push({ row: nr, col: nc });
            }
          }
          // castling — only computed on the real "legal move" pass, never
          // while scanning for attacks (that would recurse into isInCheck forever)
          if (!forAttackCheck && !piece.hasMoved && !isInCheck(b, color)) {
            const kingRow = color === 'white' ? 7 : 0;
            if (row === kingRow && col === 4) {
              const rook = b[kingRow][7];
              if (rook && rook.type === 'rook' && rook.color === color && !rook.hasMoved &&
                  !b[kingRow][5] && !b[kingRow][6]) {
                if (!isSquareAttacked(b, kingRow, 5, color) && !isSquareAttacked(b, kingRow, 6, color)) {
                  moves.push({ row: kingRow, col: 6, castling: 'king' });
                }
              }
              const rookQ = b[kingRow][0];
              if (rookQ && rookQ.type === 'rook' && rookQ.color === color && !rookQ.hasMoved &&
                  !b[kingRow][1] && !b[kingRow][2] && !b[kingRow][3]) {
                if (!isSquareAttacked(b, kingRow, 3, color) && !isSquareAttacked(b, kingRow, 2, color)) {
                  moves.push({ row: kingRow, col: 2, castling: 'queen' });
                }
              }
            }
          }
          break;
        }
      }
      return moves;
    }

    // is (row,col) attacked by the opponent of `color`? Used only for castling's
    // "king may not pass through check" rule. Always uses forAttackCheck=true.
    function isSquareAttacked(b, row, col, color) {
      const enemy = color === 'white' ? 'black' : 'white';
      for (let r = 0; r < 8; r++) for (let c = 0; c < 8; c++) {
        const p = b[r][c];
        if (p && p.color === enemy) {
          const moves = getPseudoLegalMoves(b, r, c, true);
          if (moves.some(m => m.row === row && m.col === col)) return true;
        }
      }
      return false;
    }

    function getLegalMoves(b, row, col) {
      const piece = b[row][col];
      if (!piece) return [];
      const color = piece.color;
      const pseudo = getPseudoLegalMoves(b, row, col, false);
      const legal = [];
      for (let m of pseudo) {
        const nb = cloneBoard(b);
        const fromPiece = nb[row][col];
        if (m.enPassant) {
          const epRow = color === 'white' ? m.row + 1 : m.row - 1;
          nb[epRow][m.col] = null;
        }
        if (m.castling) {
          const kingRow = row;
          if (m.castling === 'king') { nb[kingRow][5] = nb[kingRow][7]; nb[kingRow][7] = null; }
          else { nb[kingRow][3] = nb[kingRow][0]; nb[kingRow][0] = null; }
        }
        nb[m.row][m.col] = fromPiece;
        nb[row][col] = null;
        if (fromPiece.type === 'pawn' && (m.row === 0 || m.row === 7)) {
          nb[m.row][m.col] = { type: 'queen', color: color, hasMoved: true };
        }
        if (!isInCheck(nb, color)) legal.push(m);
      }
      return legal;
    }

    function getAllLegalMovesForColor(b, color) {
      const moves = [];
      for (let r = 0; r < 8; r++) for (let c = 0; c < 8; c++) {
        const p = b[r][c];
        if (p && p.color === color) {
          const lm = getLegalMoves(b, r, c);
          for (let m of lm) moves.push({ fromRow: r, fromCol: c, ...m });
        }
      }
      return moves;
    }

    function isCheckmateOrStalemate(b, color) {
      const moves = getAllLegalMovesForColor(b, color);
      if (moves.length > 0) return null;
      return isInCheck(b, color) ? 'checkmate' : 'stalemate';
    }

    function executeMove(b, fromRow, fromCol, toRow, toCol, moveData) {
      const piece = b[fromRow][fromCol];
      let captured = b[toRow][toCol];
      if (moveData.enPassant) {
        const epRow = piece.color === 'white' ? toRow + 1 : toRow - 1;
        captured = b[epRow][toCol];
        b[epRow][toCol] = null;
      }
      if (moveData.castling) {
        const kingRow = fromRow;
        if (moveData.castling === 'king') {
          b[kingRow][5] = b[kingRow][7]; b[kingRow][7] = null;
          if (b[kingRow][5]) b[kingRow][5].hasMoved = true;
        } else {
          b[kingRow][3] = b[kingRow][0]; b[kingRow][0] = null;
          if (b[kingRow][3]) b[kingRow][3].hasMoved = true;
        }
      }
      b[toRow][toCol] = piece;
      b[fromRow][fromCol] = null;
      piece.hasMoved = true;
      let promotion = false;
      if (piece.type === 'pawn' && (toRow === 0 || toRow === 7)) {
        b[toRow][toCol] = { type: 'queen', color: piece.color, hasMoved: true };
        promotion = true;
      }
      return { captured, promotion };
    }

    // ---------- AI: minimax with alpha-beta pruning + material/positional eval ----------
    // Each level controls how many plies the AI looks ahead, and how often it
    // deliberately plays a weaker move instead of its best line (a "blunder"),
    // which is what actually makes Easy/Medium feel beatable rather than just slow.
    const DIFFICULTY_SETTINGS = {
      easy:    { depth: 1, blunderChance: 0.45 },
      medium:  { depth: 2, blunderChance: 0.18 },
      hard:    { depth: 3, blunderChance: 0 },
      hardest: { depth: 4, blunderChance: 0 }
    };
    let aiDifficulty = 'hard';

    const PIECE_VALUES = { pawn: 100, knight: 320, bishop: 330, rook: 500, queen: 900, king: 20000 };

    // Standard piece-square tables (row 0 = rank 8/black home, row 7 = rank 1/white
    // home — matches this board's indexing). White reads the table as-is; black
    // reads it mirrored (7-row) since black's "good" squares are on the other side.
    const PST = {
      pawn: [
        [0,0,0,0,0,0,0,0],
        [50,50,50,50,50,50,50,50],
        [10,10,20,30,30,20,10,10],
        [5,5,10,25,25,10,5,5],
        [0,0,0,20,20,0,0,0],
        [5,-5,-10,0,0,-10,-5,5],
        [5,10,10,-20,-20,10,10,5],
        [0,0,0,0,0,0,0,0]
      ],
      knight: [
        [-50,-40,-30,-30,-30,-30,-40,-50],
        [-40,-20,0,0,0,0,-20,-40],
        [-30,0,10,15,15,10,0,-30],
        [-30,5,15,20,20,15,5,-30],
        [-30,0,15,20,20,15,0,-30],
        [-30,5,10,15,15,10,5,-30],
        [-40,-20,0,5,5,0,-20,-40],
        [-50,-40,-30,-30,-30,-30,-40,-50]
      ],
      bishop: [
        [-20,-10,-10,-10,-10,-10,-10,-20],
        [-10,0,0,0,0,0,0,-10],
        [-10,0,5,10,10,5,0,-10],
        [-10,5,5,10,10,5,5,-10],
        [-10,0,10,10,10,10,0,-10],
        [-10,10,10,10,10,10,10,-10],
        [-10,5,0,0,0,0,5,-10],
        [-20,-10,-10,-10,-10,-10,-10,-20]
      ],
      rook: [
        [0,0,0,0,0,0,0,0],
        [5,10,10,10,10,10,10,5],
        [-5,0,0,0,0,0,0,-5],
        [-5,0,0,0,0,0,0,-5],
        [-5,0,0,0,0,0,0,-5],
        [-5,0,0,0,0,0,0,-5],
        [-5,0,0,0,0,0,0,-5],
        [0,0,0,5,5,0,0,0]
      ],
      queen: [
        [-20,-10,-10,-5,-5,-10,-10,-20],
        [-10,0,0,0,0,0,0,-10],
        [-10,0,5,5,5,5,0,-10],
        [-5,0,5,5,5,5,0,-5],
        [0,0,5,5,5,5,0,-5],
        [-10,5,5,5,5,5,0,-10],
        [-10,0,5,0,0,0,0,-10],
        [-20,-10,-10,-5,-5,-10,-10,-20]
      ],
      king: [
        [-30,-40,-40,-50,-50,-40,-40,-30],
        [-30,-40,-40,-50,-50,-40,-40,-30],
        [-30,-40,-40,-50,-50,-40,-40,-30],
        [-30,-40,-40,-50,-50,-40,-40,-30],
        [-20,-30,-30,-40,-40,-30,-30,-20],
        [-10,-20,-20,-20,-20,-20,-20,-10],
        [20,20,0,0,0,0,20,20],
        [20,30,10,0,0,10,30,20]
      ]
    };

    // Positive = good for White, negative = good for Black.
    function evaluate(b) {
      let score = 0;
      for (let r = 0; r < 8; r++) {
        for (let c = 0; c < 8; c++) {
          const p = b[r][c];
          if (!p) continue;
          const table = PST[p.type];
          const posBonus = p.color === 'white' ? table[r][c] : table[7 - r][c];
          const val = PIECE_VALUES[p.type] + posBonus;
          score += p.color === 'white' ? val : -val;
        }
      }
      return score;
    }

    // Try captures first — dramatically improves alpha-beta pruning efficiency.
    function orderMoves(b, moves) {
      moves.sort((m1, m2) => {
        const cap1 = b[m1.row][m1.col] ? PIECE_VALUES[b[m1.row][m1.col].type] : 0;
        const cap2 = b[m2.row][m2.col] ? PIECE_VALUES[b[m2.row][m2.col].type] : 0;
        return cap2 - cap1;
      });
    }

    function applySimMove(b, m) {
      const nb = cloneBoard(b);
      executeMove(nb, m.fromRow, m.fromCol, m.row, m.col, {
        enPassant: m.enPassant || false,
        castling: m.castling || null
      });
      return nb;
    }

    function minimax(b, depth, alpha, beta, colorToMove) {
      const moves = getAllLegalMovesForColor(b, colorToMove);
      if (moves.length === 0) {
        if (isInCheck(b, colorToMove)) {
          // Checkmated — huge penalty/bonus, biased toward faster mates.
          return colorToMove === 'white' ? -99000 - depth : 99000 + depth;
        }
        return 0; // stalemate
      }
      if (depth === 0) return evaluate(b);

      orderMoves(b, moves);

      if (colorToMove === 'white') {
        let maxEval = -Infinity;
        for (const m of moves) {
          const ev = minimax(applySimMove(b, m), depth - 1, alpha, beta, 'black');
          if (ev > maxEval) maxEval = ev;
          if (ev > alpha) alpha = ev;
          if (beta <= alpha) break; // black won't allow this line
        }
        return maxEval;
      } else {
        let minEval = Infinity;
        for (const m of moves) {
          const ev = minimax(applySimMove(b, m), depth - 1, alpha, beta, 'white');
          if (ev < minEval) minEval = ev;
          if (ev < beta) beta = ev;
          if (beta <= alpha) break; // white won't allow this line
        }
        return minEval;
      }
    }

    // Root search: picks the AI's best move by looking `depth` plies ahead.
    // With probability `blunderChance`, it ignores the best line and plays a
    // randomly-chosen legal move instead — this is what makes Easy/Medium weaker
    // in a human-feeling way, rather than just being a shallower perfect player.
    function getAIMove(b, color, depth, blunderChance) {
      const moves = getAllLegalMovesForColor(b, color);
      if (moves.length === 0) return null;
      orderMoves(b, moves);

      if (blunderChance > 0 && Math.random() < blunderChance) {
        return moves[Math.floor(Math.random() * moves.length)];
      }

      let alpha = -Infinity, beta = Infinity;
      let bestScore = color === 'white' ? -Infinity : Infinity;
      let bestMoves = [];

      for (const m of moves) {
        const nb = applySimMove(b, m);
        const score = minimax(nb, depth - 1, alpha, beta, color === 'white' ? 'black' : 'white');
        if (color === 'white') {
          if (score > bestScore) { bestScore = score; bestMoves = [m]; }
          else if (score === bestScore) bestMoves.push(m);
          if (score > alpha) alpha = score;
        } else {
          if (score < bestScore) { bestScore = score; bestMoves = [m]; }
          else if (score === bestScore) bestMoves.push(m);
          if (score < beta) beta = score;
        }
      }
      // small randomness among equally-good top moves so the AI isn't fully deterministic
      return bestMoves[Math.floor(Math.random() * bestMoves.length)];
    }

    // ---------- render & UI ----------
    function render() {
      boardEl.innerHTML = '';
      for (let r = 0; r < 8; r++) {
        for (let c = 0; c < 8; c++) {
          const sq = document.createElement('div');
          const isLight = (r + c) % 2 === 0;
          sq.className = `square ${isLight ? 'light' : 'dark'}`;
          const piece = board[r][c];
          if (piece) sq.textContent = PIECES[piece.color][piece.type] || '?';
          if (lastMove && ((r === lastMove.fromRow && c === lastMove.fromCol) || (r === lastMove.toRow && c === lastMove.toCol))) {
            sq.classList.add('last-move');
          }
          if (selected && selected.row === r && selected.col === c) sq.classList.add('selected');
          if (validMoves.some(m => m.row === r && m.col === c)) sq.classList.add('valid-move');
          sq.dataset.row = r; sq.dataset.col = c;
          sq.addEventListener('click', () => handleClick(r, c));
          boardEl.appendChild(sq);
        }
      }
      updateTurnDisplay();
    }

    function updateTurnDisplay() {
      turnIndicator.classList.remove('turn-white', 'turn-black', 'turn-thinking', 'turn-over');
      if (gameOver) {
        turnIndicator.classList.add('turn-over');
        const status = isCheckmateOrStalemate(board, turn);
        if (status === 'checkmate') {
          const winner = turn === 'white' ? 'Black' : 'White';
          turnIndicator.textContent = `🏆 ${winner} wins by checkmate!`;
        } else if (status === 'stalemate') {
          turnIndicator.textContent = `🤝 Stalemate – draw`;
        } else turnIndicator.textContent = `Game over`;
        return;
      }
      // Show "thinking" purely based on whose turn it is when AI is enabled —
      // not on the aiThinking flag, which briefly lags behind render() calls.
      if (aiEnabled && turn === 'black') {
        turnIndicator.classList.add('turn-thinking');
        turnIndicator.textContent = `🤖 Computer thinking...`;
      } else {
        turnIndicator.classList.add(turn === 'white' ? 'turn-white' : 'turn-black');
        turnIndicator.textContent = turn === 'white' ? "White's turn" : "Black's turn";
      }
    }

    // ---------- voice announcements ----------
    function speak(text) {
      if (!('speechSynthesis' in window)) return;
      try {
        window.speechSynthesis.cancel(); // don't let announcements queue/overlap
        const utter = new SpeechSynthesisUtterance(text);
        utter.rate = 0.95;
        utter.pitch = 1.05;
        utter.volume = 1;
        window.speechSynthesis.speak(utter);
      } catch (e) { /* speech not available — fail silently */ }
    }

    // Call right after `turn` has switched to the side about to move, and
    // `status` (from isCheckmateOrStalemate) has been computed for them.
    function announceCheckVoice(colorToMove, status) {
      if (status === 'checkmate') {
        speak('Checkmate');
      } else if (!status && isInCheck(board, colorToMove)) {
        speak('Check');
      }
    }

    function handleClick(row, col) {
      if (gameOver || aiThinking) return;
      if (aiEnabled && turn === 'black') return; // AI controls black

      const piece = board[row][col];
      if (selected) {
        const move = validMoves.find(m => m.row === row && m.col === col);
        if (move) {
          const fromRow = selected.row, fromCol = selected.col;
          const color = board[fromRow][fromCol].color;
          const moveData = { enPassant: move.enPassant || false, castling: move.castling || null };
          const result = executeMove(board, fromRow, fromCol, row, col, moveData);
          moveHistory.push({
            fromRow, fromCol, toRow: row, toCol: col,
            pieceType: board[row][col].type,
            color: color,
            captured: result.captured,
            promotion: result.promotion || false
          });
          lastMove = { fromRow, fromCol, toRow: row, toCol: col };
          turn = turn === 'white' ? 'black' : 'white';
          selected = null; validMoves = [];
          const status = isCheckmateOrStalemate(board, turn);
          announceCheckVoice(turn, status);
          if (status) { gameOver = true; render(); return; }
          render();
          if (aiEnabled && turn === 'black' && !gameOver) {
            aiThinking = true;
            setTimeout(() => doAIMove(), 2000);
          }
          return;
        }
        if (piece && piece.color === turn) {
          selected = { row, col };
          validMoves = getLegalMoves(board, row, col);
          render();
          return;
        }
        selected = null; validMoves = []; render();
        return;
      }

      if (piece && piece.color === turn) {
        const moves = getLegalMoves(board, row, col);
        if (moves.length === 0) return;
        selected = { row, col };
        validMoves = moves;
        render();
      }
    }

    function doAIMove() {
      if (!aiEnabled || gameOver || turn !== 'black') { aiThinking = false; return; }
      const settings = DIFFICULTY_SETTINGS[aiDifficulty];
      const aiMove = getAIMove(board, 'black', settings.depth, settings.blunderChance);
      if (!aiMove) { aiThinking = false; return; }
      const { fromRow, fromCol, row, col, enPassant, castling } = aiMove;
      const moveData = { enPassant: enPassant || false, castling: castling || null };
      const result = executeMove(board, fromRow, fromCol, row, col, moveData);
      moveHistory.push({
        fromRow, fromCol, toRow: row, toCol: col,
        pieceType: board[row][col].type,
        color: 'black',
        captured: result.captured,
        promotion: result.promotion || false
      });
      lastMove = { fromRow, fromCol, toRow: row, toCol: col };
      turn = 'white';
      selected = null; validMoves = [];
      const status = isCheckmateOrStalemate(board, turn);
      announceCheckVoice(turn, status);
      if (status) { gameOver = true; aiThinking = false; render(); return; }
      aiThinking = false;
      render();
    }

    // ---------- reset & mode toggle ----------
    function resetGame() {
      if (aiThinking) return;
      initBoard();
      selected = null; validMoves = []; gameOver = false; lastMove = null;
      render();
      if (aiEnabled && turn === 'black') {
        aiThinking = true;
        setTimeout(() => { if (!gameOver) doAIMove(); }, 2000);
      }
    }

    function toggleMode() {
      if (aiThinking) return;
      aiEnabled = !aiEnabled;
      modeBtn.textContent = aiEnabled ? '🤖 vs Computer' : '👥 Two‑player';
      modeBtn.classList.toggle('active-mode', aiEnabled);
      difficultyRow.style.display = aiEnabled ? 'flex' : 'none';
      resetGame();
    }

    function setDifficulty(level) {
      aiDifficulty = level;
      diffButtons.forEach(btn => btn.classList.toggle('diff-active', btn.dataset.level === level));
    }

    // ---------- init ----------
    const difficultyRow = document.getElementById('difficultyRow');
    const diffButtons = document.querySelectorAll('.diff-btn');
    diffButtons.forEach(btn => {
      btn.addEventListener('click', () => {
        if (aiThinking) return;
        setDifficulty(btn.dataset.level);
      });
    });
    initBoard();
    render();
    document.getElementById('resetBtn').addEventListener('click', resetGame);
    modeBtn.addEventListener('click', toggleMode);
  })();
</script>
</body>
</html>
