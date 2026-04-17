<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>The CFO's Dilemma — An Auburn Accounting Game</title>
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;700;900&family=DM+Mono:wght@400;500&family=DM+Sans:wght@300;400;500;600&display=swap" rel="stylesheet">
<style>
  :root {
    --ink: #1a0800;
    --paper: #faf4ed;
    --cream: #f0e6d8;
    --auburn: #8B2500;
    --auburn-mid: #a83010;
    --auburn-light: #c84820;
    --auburn-pale: #f5ddd4;
    --red: #8B2500;
    --green: #2a6b3c;
    --green-light: #e8f4ec;
    --red-light: #fbeaea;
    --blue: #1a3a5c;
    --blue-light: #e8f0f8;
    --muted: #6b5248;
    --border: #d4b8aa;
    --shadow: rgba(26,8,0,0.14);
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    font-family: 'DM Sans', sans-serif;
    background: var(--paper);
    color: var(--ink);
    min-height: 100vh;
    position: relative;
    overflow-x: hidden;
  }

  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background:
      repeating-linear-gradient(0deg, transparent, transparent 27px, rgba(139,37,0,0.06) 28px),
      repeating-linear-gradient(90deg, transparent, transparent 27px, rgba(139,37,0,0.03) 28px);
    pointer-events: none;
    z-index: 0;
  }

  .noise {
    position: fixed; inset: 0; opacity: 0.025;
    background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)'/%3E%3C/svg%3E");
    pointer-events: none; z-index: 0;
  }

  .container { max-width: 900px; margin: 0 auto; padding: 20px; position: relative; z-index: 1; }

  .screen { display: none; animation: fadeIn 0.4s ease; }
  .screen.active { display: block; }
  @keyframes fadeIn { from { opacity: 0; transform: translateY(12px); } to { opacity: 1; transform: translateY(0); } }

  /* ── MASTHEAD ── */
  .masthead { text-align: center; padding: 48px 20px 32px; border-bottom: 2px solid var(--ink); margin-bottom: 40px; position: relative; }
  .masthead::after { content: ''; display: block; height: 4px; background: repeating-linear-gradient(90deg, var(--auburn) 0, var(--auburn) 8px, transparent 8px, transparent 16px); margin-top: 12px; }
  .eyebrow { font-family: 'DM Mono', monospace; font-size: 11px; letter-spacing: 3px; text-transform: uppercase; color: var(--auburn); margin-bottom: 12px; }
  .game-title { font-family: 'Playfair Display', serif; font-size: clamp(36px, 7vw, 64px); font-weight: 900; line-height: 1.05; color: var(--ink); }
  .game-title span { color: var(--auburn); }
  .subtitle { font-size: 15px; color: var(--muted); margin-top: 10px; font-weight: 300; font-style: italic; }

  /* ── PROGRESS ── */
  .progress-wrap { background: var(--cream); border: 1px solid var(--border); border-radius: 4px; height: 8px; margin-bottom: 28px; overflow: hidden; }
  .progress-fill { height: 100%; background: linear-gradient(90deg, var(--auburn), var(--auburn-light)); transition: width 0.5s cubic-bezier(0.4,0,0.2,1); border-radius: 4px; }
  .progress-label { font-family: 'DM Mono', monospace; font-size: 11px; color: var(--muted); margin-bottom: 8px; letter-spacing: 1px; display: flex; justify-content: space-between; }

  /* ── GAME LAYOUT with sidebar ── */
  .game-layout { display: grid; grid-template-columns: 1fr 220px; gap: 20px; align-items: start; }
  @media (max-width: 720px) { .game-layout { grid-template-columns: 1fr; } .stats-panel { order: -1; } }

  /* ── STATS PANEL ── */
  .stats-panel {
    background: var(--ink);
    color: var(--paper);
    border-radius: 6px;
    padding: 18px;
    position: sticky;
    top: 20px;
    font-family: 'DM Mono', monospace;
    font-size: 12px;
  }
  .stats-panel-title {
    font-family: 'Playfair Display', serif;
    font-size: 14px;
    color: var(--auburn-pale);
    margin-bottom: 14px;
    padding-bottom: 10px;
    border-bottom: 1px solid rgba(255,255,255,0.1);
    letter-spacing: 0.5px;
  }
  .stat-row { display: flex; justify-content: space-between; align-items: baseline; margin-bottom: 9px; gap: 6px; }
  .stat-label { opacity: 0.6; font-size: 10px; letter-spacing: 1px; line-height: 1.4; flex: 1; }
  .stat-value { color: #ffd6c4; font-size: 13px; font-weight: 500; text-align: right; white-space: nowrap; }
  .stat-divider { border: none; border-top: 1px solid rgba(255,255,255,0.08); margin: 10px 0; }
  .stat-highlight { color: var(--auburn-pale); font-size: 10px; opacity: 0.5; letter-spacing: 1px; margin-top: 14px; padding-top: 10px; border-top: 1px solid rgba(255,255,255,0.08); text-align: center; }
  .stat-note { opacity: 0.45; font-size: 9px; letter-spacing: 0.5px; line-height: 1.5; margin-top: 8px; font-style: italic; }

  /* ── CARD ── */
  .card { background: #fff; border: 1px solid var(--border); border-radius: 6px; padding: 32px; box-shadow: 4px 4px 0 var(--border); margin-bottom: 24px; position: relative; }
  .card-accent { position: absolute; top: 0; left: 0; width: 4px; height: 100%; background: var(--auburn); border-radius: 6px 0 0 6px; }
  .scenario-label { font-family: 'DM Mono', monospace; font-size: 10px; letter-spacing: 3px; text-transform: uppercase; color: var(--auburn); margin-bottom: 10px; }
  .question-text { font-family: 'Playfair Display', serif; font-size: clamp(17px, 3vw, 21px); line-height: 1.45; color: var(--ink); margin-bottom: 8px; }
  .context-text { font-size: 14px; color: var(--muted); line-height: 1.7; margin-bottom: 20px; padding: 14px 16px; background: var(--cream); border-left: 3px solid var(--auburn); border-radius: 0 4px 4px 0; font-style: italic; }

  /* ── DATA TABLE ── */
  .data-table { width: 100%; border-collapse: collapse; font-size: 13px; margin: 16px 0; font-family: 'DM Mono', monospace; }
  .data-table th { background: var(--ink); color: var(--paper); padding: 8px 12px; text-align: left; font-weight: 500; font-size: 11px; letter-spacing: 1px; }
  .data-table td { padding: 8px 12px; border-bottom: 1px solid var(--border); }
  .data-table tr:nth-child(even) td { background: var(--cream); }
  .data-table .num { text-align: right; }
  .data-table .highlight { color: var(--auburn); font-weight: 600; }

  /* ── ANSWERS ── */
  .answers-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-top: 20px; }
  @media (max-width: 560px) { .answers-grid { grid-template-columns: 1fr; } }
  .answer-btn { background: #fff; border: 2px solid var(--border); border-radius: 6px; padding: 14px 18px; text-align: left; cursor: pointer; font-family: 'DM Sans', sans-serif; font-size: 14px; line-height: 1.5; color: var(--ink); transition: all 0.2s; position: relative; overflow: hidden; }
  .answer-btn::before { content: ''; position: absolute; top: 0; left: 0; width: 0; height: 100%; background: var(--auburn); opacity: 0.06; transition: width 0.3s; }
  .answer-btn:hover:not(:disabled)::before { width: 100%; }
  .answer-btn:hover:not(:disabled) { border-color: var(--auburn); transform: translateY(-2px); box-shadow: 2px 4px 12px var(--shadow); }
  .answer-btn.correct { border-color: var(--green); background: var(--green-light); color: var(--green); }
  .answer-btn.wrong { border-color: var(--red); background: var(--red-light); color: var(--red); text-decoration: line-through; opacity: 0.7; }
  .answer-btn:disabled { cursor: default; transform: none !important; }
  .answer-letter { font-family: 'DM Mono', monospace; font-weight: 600; font-size: 11px; color: var(--auburn); display: block; margin-bottom: 4px; letter-spacing: 1px; }

  /* ── FEEDBACK ── */
  .feedback-box { display: none; margin-top: 20px; border-radius: 6px; overflow: hidden; animation: slideUp 0.35s ease; }
  .feedback-box.visible { display: block; }
  @keyframes slideUp { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
  .feedback-header { padding: 12px 20px; font-family: 'DM Mono', monospace; font-size: 12px; letter-spacing: 2px; font-weight: 600; display: flex; align-items: center; gap: 8px; }
  .feedback-header.correct { background: var(--green); color: #fff; }
  .feedback-header.wrong { background: var(--auburn); color: #fff; }
  .feedback-body { padding: 20px; font-size: 14px; line-height: 1.8; color: var(--ink); }
  .feedback-body.correct { background: var(--green-light); border: 1px solid #acd8b8; border-top: none; }
  .feedback-body.wrong { background: var(--auburn-pale); border: 1px solid #e8c4b4; border-top: none; }
  .feedback-body strong { color: var(--blue); }
  .feedback-body .formula { font-family: 'DM Mono', monospace; background: rgba(255,255,255,0.85); padding: 10px 14px; border-radius: 4px; margin: 12px 0; font-size: 13px; border-left: 3px solid var(--auburn); white-space: pre-wrap; line-height: 1.7; }
  .feedback-body .insight { margin-top: 12px; padding: 12px; background: rgba(255,255,255,0.6); border-radius: 4px; font-style: italic; font-size: 13px; color: var(--muted); }
  .insight-icon { font-style: normal; }

  /* ── BUTTONS ── */
  .btn-primary { display: inline-block; background: var(--auburn); color: #fff; border: none; padding: 14px 32px; font-family: 'DM Sans', sans-serif; font-size: 15px; font-weight: 600; border-radius: 4px; cursor: pointer; transition: all 0.2s; letter-spacing: 0.5px; position: relative; overflow: hidden; }
  .btn-primary::after { content: ''; position: absolute; bottom: 0; left: 0; width: 100%; height: 3px; background: rgba(255,255,255,0.25); }
  .btn-primary:hover { background: var(--auburn-mid); transform: translateY(-2px); box-shadow: 0 6px 20px var(--shadow); }
  .btn-primary:active { transform: translateY(0); }
  .btn-outline { display: inline-block; background: transparent; color: var(--ink); border: 2px solid var(--ink); padding: 12px 28px; font-family: 'DM Sans', sans-serif; font-size: 14px; font-weight: 600; border-radius: 4px; cursor: pointer; transition: all 0.2s; }
  .btn-outline:hover { background: var(--ink); color: var(--paper); }
  .btn-back { display: none; background: transparent; color: var(--muted); border: 1.5px solid var(--border); padding: 8px 18px; font-family: 'DM Mono', monospace; font-size: 11px; letter-spacing: 1px; border-radius: 4px; cursor: pointer; transition: all 0.2s; }
  .btn-back:hover { border-color: var(--auburn); color: var(--auburn); }
  .btn-back.visible { display: inline-block; }
  .btn-next { margin-top: 20px; display: none; }
  .btn-next.visible { display: inline-block; }
  .nav-row { display: flex; justify-content: space-between; align-items: center; margin-top: 8px; flex-wrap: wrap; gap: 10px; }

  /* ── CALC INPUT ── */
  .calc-input-wrap { display: flex; gap: 12px; align-items: flex-end; margin-top: 16px; flex-wrap: wrap; }
  .calc-input { flex: 1; min-width: 140px; padding: 12px 16px; border: 2px solid var(--border); border-radius: 4px; font-family: 'DM Mono', monospace; font-size: 18px; color: var(--ink); background: #fff; outline: none; transition: border-color 0.2s; text-align: right; }
  .calc-input:focus { border-color: var(--auburn); }
  .calc-input.correct { border-color: var(--green); background: var(--green-light); }
  .calc-input.wrong { border-color: var(--auburn); background: var(--auburn-pale); }
  .input-label { font-size: 13px; color: var(--muted); font-family: 'DM Mono', monospace; margin-bottom: 8px; display: block; }
  .input-group { flex: 1; min-width: 140px; }
  .hint-box { margin-top: 14px; padding: 10px 14px; background: var(--blue-light); border: 1px solid #bcd4ec; border-radius: 4px; font-size: 13px; color: var(--blue); display: none; }
  .hint-box.visible { display: block; }

  /* ── INTRO ── */
  .intro-card { background: var(--auburn); color: #fff; border-radius: 8px; padding: 40px; margin-bottom: 24px; position: relative; overflow: hidden; }
  .intro-card::before { content: '§'; position: absolute; top: -20px; right: 20px; font-family: 'Playfair Display', serif; font-size: 180px; opacity: 0.07; color: #fff; line-height: 1; }
  .intro-card h2 { font-family: 'Playfair Display', serif; font-size: 26px; color: #ffe0d4; margin-bottom: 16px; }
  .intro-card p { font-size: 15px; line-height: 1.8; opacity: 0.9; margin-bottom: 12px; }
  .objectives-list { list-style: none; margin: 20px 0; }
  .objectives-list li { padding: 10px 16px; background: var(--cream); border-radius: 4px; margin-bottom: 8px; font-size: 14px; line-height: 1.6; border-left: 4px solid var(--auburn); display: flex; gap: 10px; }
  .objectives-list li .num { font-family: 'DM Mono', monospace; font-size: 11px; color: var(--auburn); font-weight: 600; flex-shrink: 0; padding-top: 2px; }
  .chapter-nav { display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; margin: 20px 0; }
  @media (max-width: 480px) { .chapter-nav { grid-template-columns: 1fr; } }
  .chapter-pill { padding: 12px; border: 1px solid var(--border); border-radius: 4px; text-align: center; font-size: 12px; color: var(--muted); background: #fff; }
  .chapter-pill strong { display: block; font-family: 'DM Mono', monospace; font-size: 10px; color: var(--auburn); letter-spacing: 1px; margin-bottom: 4px; }

  /* ── CHAPTER HEADER ── */
  .chapter-header { display: flex; align-items: center; gap: 12px; margin-bottom: 20px; }
  .chapter-badge { font-family: 'DM Mono', monospace; font-size: 10px; letter-spacing: 2px; background: var(--auburn); color: #fff; padding: 4px 10px; border-radius: 2px; white-space: nowrap; }
  .chapter-title-text { font-family: 'Playfair Display', serif; font-size: 18px; color: var(--ink); }

  /* ── RESULTS ── */
  .results-hero { text-align: center; padding: 40px 20px; border-bottom: 2px dashed var(--border); margin-bottom: 28px; }
  .results-grade { font-family: 'Playfair Display', serif; font-size: 96px; font-weight: 900; line-height: 1; }
  .results-grade.A { color: var(--green); }
  .results-grade.B { color: var(--blue); }
  .results-grade.C { color: var(--auburn); }
  .results-grade.D { color: var(--auburn-mid); }
  .results-label { font-family: 'DM Mono', monospace; font-size: 12px; letter-spacing: 3px; color: var(--muted); margin-top: 8px; }
  .results-score-num { font-family: 'Playfair Display', serif; font-size: 28px; margin-top: 8px; color: var(--ink); }
  .results-breakdown { display: grid; grid-template-columns: repeat(2, 1fr); gap: 12px; margin-bottom: 28px; }
  @media (max-width: 480px) { .results-breakdown { grid-template-columns: 1fr; } }
  .result-chapter { padding: 16px; border: 1px solid var(--border); border-radius: 4px; background: #fff; }
  .result-chapter .chap-label { font-family: 'DM Mono', monospace; font-size: 10px; letter-spacing: 2px; color: var(--muted); margin-bottom: 4px; }
  .result-chapter .chap-title { font-size: 13px; font-weight: 600; margin-bottom: 8px; }
  .result-chapter .chap-score { font-family: 'DM Mono', monospace; font-size: 18px; font-weight: 600; }
  .result-chapter .chap-score.perfect { color: var(--green); }
  .result-chapter .chap-score.good { color: var(--blue); }
  .result-chapter .chap-score.ok { color: var(--auburn); }
  .result-chapter .chap-score.low { color: var(--auburn-mid); }
  .mini-bar { height: 4px; background: var(--border); border-radius: 2px; margin-top: 10px; overflow: hidden; }
  .mini-fill { height: 100%; border-radius: 2px; background: var(--auburn); }
  .key-takeaway { background: var(--auburn); color: #fff; border-radius: 6px; padding: 24px; margin-bottom: 24px; }
  .key-takeaway h3 { font-family: 'Playfair Display', serif; color: #ffd6c8; margin-bottom: 12px; font-size: 18px; }
  .key-takeaway p { font-size: 14px; line-height: 1.8; opacity: 0.9; }
  .citation-box { border: 1px solid var(--border); border-radius: 4px; padding: 16px 20px; font-size: 12px; color: var(--muted); font-family: 'DM Mono', monospace; line-height: 1.8; margin-top: 24px; }
  .citation-box strong { color: var(--ink); font-weight: 600; }

  /* ── REVIEW MODE BANNER ── */
  .review-banner { background: var(--blue-light); border: 1px solid #bcd4ec; border-radius: 4px; padding: 8px 14px; font-size: 12px; font-family: 'DM Mono', monospace; color: var(--blue); margin-bottom: 12px; display: none; }
  .review-banner.visible { display: block; }

  @media (max-width: 600px) { .card { padding: 20px; } .intro-card { padding: 24px; } .answer-btn { padding: 12px 14px; font-size: 13px; } }
</style>
</head>
<body>
<div class="noise"></div>
<div class="container">

<!-- ══════════ INTRO ══════════ -->
<div id="screen-intro" class="screen active">
  <div class="masthead">
    <div class="eyebrow">MBA Cost Accounting · Interactive Simulation</div>
    <h1 class="game-title">The CFO's<br><span>Dilemma</span></h1>
    <p class="subtitle">A Cost-Volume-Profit &amp; Managerial Accounting Mastery Game</p>
  </div>

  <div class="intro-card">
    <h2>Welcome, Associate.</h2>
    <p>You've just joined Meridian Consumer Goods as a Senior Financial Analyst. The CFO has one hour before the Board meeting — and they need answers.</p>
    <p>Work through <strong>four chapters</strong> of real business scenarios. A running reference panel keeps Meridian's key numbers on screen, and you can always go back to review an earlier question.</p>
  </div>

  <div class="card" style="margin-bottom:20px;">
    <div class="card-accent"></div>
    <div class="scenario-label">📋 Learning Objectives</div>
    <ul class="objectives-list">
      <li><span class="num">01</span>Distinguish fixed costs, variable costs, and mixed costs — and explain why it matters for every financial decision.</li>
      <li><span class="num">02</span>Calculate contribution margin per unit, CM ratio, and operating profit at various volume levels.</li>
      <li><span class="num">03</span>Determine break-even point in both units and sales dollars, and apply target profit analysis.</li>
      <li><span class="num">04</span>Apply the margin of safety concept and the Cost-Volume-Profit (CVP) graph framework to assess risk and performance.</li>
      <li><span class="num">05</span>Use CVP to evaluate strategic decisions: pricing changes, operating leverage, and multi-product sales mix.</li>
      <li><span class="num">06</span>Apply managerial decision-making frameworks: relevant costs, special orders, make-or-buy, and segment analysis.</li>
    </ul>
  </div>

  <div class="card">
    <div class="card-accent"></div>
    <div class="scenario-label">🗺 Game Structure — 4 Chapters</div>
    <div class="chapter-nav">
      <div class="chapter-pill"><strong>CHAPTER 1</strong>Cost Fundamentals<br><small style="color:var(--muted)">Q1–7 · Fixed, Variable, Mixed, CM</small></div>
      <div class="chapter-pill"><strong>CHAPTER 2</strong>Break-Even &amp; CVP<br><small style="color:var(--muted)">Q8–14 · Core Analysis</small></div>
      <div class="chapter-pill"><strong>CHAPTER 3</strong>Strategic Decisions<br><small style="color:var(--muted)">Q15–18 · Pricing &amp; Leverage</small></div>
      <div class="chapter-pill"><strong>CHAPTER 4</strong>Managerial Decisions<br><small style="color:var(--muted)">Q19–24 · Relevant Costs</small></div>
    </div>
    <p style="font-size:13px; color:var(--muted); margin-top:10px;">A reference panel on the right keeps Meridian's key numbers visible throughout. Use the Back button to revisit any prior question.</p>
  </div>

  <div style="text-align:center; padding: 8px 0 40px;">
    <button class="btn-primary" onclick="startGame()">Begin Simulation →</button>
  </div>
</div>

<!-- ══════════ GAME ══════════ -->
<div id="screen-game" class="screen">
  <div class="progress-label">
    <span id="progress-text">Question 1 of 24</span>
    <span id="score-display">Score: 0 pts</span>
  </div>
  <div class="progress-wrap"><div class="progress-fill" id="progress-bar" style="width:0%"></div></div>

  <div class="review-banner" id="review-banner">📖 REVIEW MODE — You are revisiting a previous question. Your score will not change.</div>

  <div class="game-layout">
    <div id="question-area"></div>

    <!-- STATS SIDEBAR -->
    <div class="stats-panel" id="stats-panel">
      <div class="stats-panel-title">📊 Meridian Reference</div>

      <div class="stat-row">
        <span class="stat-label">SELLING PRICE</span>
        <span class="stat-value">$22.00/unit</span>
      </div>
      <div class="stat-row">
        <span class="stat-label">VARIABLE COST</span>
        <span class="stat-value">$7.00/unit</span>
      </div>
      <div class="stat-row">
        <span class="stat-label">CONTRIB. MARGIN</span>
        <span class="stat-value">$15.00/unit</span>
      </div>
      <div class="stat-row">
        <span class="stat-label">CM RATIO</span>
        <span class="stat-value">68.18%</span>
      </div>
      <hr class="stat-divider">
      <div class="stat-row">
        <span class="stat-label">FIXED COSTS / MO</span>
        <span class="stat-value">$62,167</span>
      </div>
      <div class="stat-row">
        <span class="stat-label">BREAK-EVEN (units)</span>
        <span class="stat-value">4,145 units</span>
      </div>
      <div class="stat-row">
        <span class="stat-label">BREAK-EVEN ($)</span>
        <span class="stat-value">$91,174</span>
      </div>
      <hr class="stat-divider">
      <div class="stat-row">
        <span class="stat-label">ACTUAL SALES / MO</span>
        <span class="stat-value">6,200 units</span>
      </div>
      <div class="stat-row">
        <span class="stat-label">MARGIN OF SAFETY</span>
        <span class="stat-value">2,055 units</span>
      </div>
      <div class="stat-row">
        <span class="stat-label">SAFETY % </span>
        <span class="stat-value">33.1%</span>
      </div>
      <hr class="stat-divider">
      <div class="stat-row">
        <span class="stat-label">MONTHLY PROFIT</span>
        <span class="stat-value">$30,833</span>
      </div>
      <p class="stat-note">Fixed costs: Rent $45k + Salaries $9,167 + Depreciation $5k + Marketing $3k</p>
    </div>
  </div>

  <div class="nav-row">
    <button class="btn-back" id="btn-back" onclick="goBack()">← Back</button>
    <button class="btn-primary btn-next" id="btn-next" onclick="nextQuestion()">Next Question →</button>
  </div>
</div>

<!-- ══════════ RESULTS ══════════ -->
<div id="screen-results" class="screen">
  <div class="masthead">
    <div class="eyebrow">SIMULATION COMPLETE</div>
    <h1 class="game-title">Your <span>Report</span> Card</h1>
  </div>
  <div class="results-hero">
    <div class="results-grade" id="result-grade">A</div>
    <div class="results-label">FINAL GRADE</div>
    <div class="results-score-num" id="result-score">— / 240 points</div>
    <p id="result-headline" style="margin-top:12px; font-style:italic; color:var(--muted); font-size:15px;"></p>
  </div>
  <div class="results-breakdown" id="result-breakdown"></div>
  <div class="key-takeaway">
    <h3>The CFO's Summary</h3>
    <p id="result-takeaway"></p>
  </div>
  <div style="text-align:center; margin: 24px 0;">
    <button class="btn-primary" onclick="restartGame()" style="margin-right:12px;">Replay Simulation ↺</button>
    <button class="btn-outline" onclick="showIntro()">← Back to Overview</button>
  </div>
  <div class="citation-box">
    <strong>Sources &amp; References</strong><br>
    Garrison, R., Noreen, E., &amp; Brewer, P. (2021). <em>Managerial Accounting</em> (17th ed.). McGraw-Hill Education.<br>
    Horngren, C., Datar, S., &amp; Rajan, M. (2020). <em>Cost Accounting: A Managerial Emphasis</em> (17th ed.). Pearson.<br>
    Weygandt, J., Kimmel, P., &amp; Kieso, D. (2021). <em>Managerial Accounting</em> (4th ed.). Wiley.<br>
    All scenario data is fictional and created for educational purposes only.
  </div>
</div>

</div><!-- /container -->

<script>
// ══════════════════════════════════════════════════════════════
//  QUESTION BANK — 24 QUESTIONS, 4 CHAPTERS
// ══════════════════════════════════════════════════════════════
const questions = [

// ─────────────────────────────────────────
//  CHAPTER 1: COST FUNDAMENTALS  (Q1–7)
// ─────────────────────────────────────────

{
  chapter:"Chapter 1", chapterTitle:"Cost Fundamentals", type:"mcq",
  label:"COST CLASSIFICATION",
  question:"Meridian's finance team is preparing next quarter's cost model. Which of the following best describes a <strong>fixed cost</strong>?",
  context:"Meridian manufactures premium water bottles. Its monthly costs include: rent for the factory ($45,000), raw materials ($4.20 per bottle), sales commissions (8% of revenue), and executive salaries ($110,000/year).",
  answers:[
    {letter:"A",text:"Raw materials at $4.20 per bottle — cost rises proportionally with each unit produced."},
    {letter:"B",text:"Factory rent of $45,000/month — total cost stays constant regardless of production volume."},
    {letter:"C",text:"Sales commissions at 8% of revenue — cost varies directly with sales dollars."},
    {letter:"D",text:"Utility bills — these increase slightly as the factory runs more machines."}
  ], correct:1,
  feedback:{
    correct:`<strong>Exactly right.</strong> Factory rent is a textbook fixed cost: the landlord's invoice is $45,000 whether Meridian produces 1,000 bottles or 100,000 bottles. The total amount never changes with volume — only the <em>per-unit</em> share changes.<div class="formula">Fixed Cost per Unit = Total Fixed Cost ÷ Units Produced
At 10,000 units: $45,000 ÷ 10,000 = $4.50/unit
At 45,000 units: $45,000 ÷ 45,000 = $1.00/unit</div><div class="insight"><span class="insight-icon">💡</span> This is why scaling production is so powerful — fixed costs become increasingly diluted per unit, improving margins without changing the cost itself. Understanding fixed cost behavior is the foundation of all CVP analysis.</div>`,
    wrong:`<strong>The correct answer is B — Factory rent.</strong> A fixed cost remains constant in <em>total</em> regardless of activity level. Raw materials (A) and sales commissions (C) are variable costs — they scale with volume. Utilities (D) that increase with machine use are mixed or variable costs.<div class="formula">Key Test:
If activity doubles and the total cost stays the same → Fixed Cost
If activity doubles and the total cost doubles → Variable Cost</div><div class="insight"><span class="insight-icon">💡</span> Misclassifying a fixed cost as variable means you'll project the wrong profit impact when volume changes — a cascading forecasting error that compounds with every unit.</div>`
  }
},

{
  chapter:"Chapter 1", chapterTitle:"Cost Fundamentals", type:"mcq",
  label:"VARIABLE COST BEHAVIOR",
  question:"Meridian's CFO asks: if production increases from 20,000 to 30,000 units next quarter, what happens to the <strong>variable cost per unit</strong>?",
  context:"Variable costs include raw materials ($4.20/bottle), packaging ($0.80/bottle), and direct labor ($2.00/bottle). Total variable cost = $7.00 per bottle.",
  answers:[
    {letter:"A",text:"Variable cost per unit increases — higher production puts cost pressure on suppliers."},
    {letter:"B",text:"Variable cost per unit decreases — spreading costs over more units reduces the per-unit rate."},
    {letter:"C",text:"Variable cost per unit stays the same — it's constant by definition, regardless of volume."},
    {letter:"D",text:"Variable cost per unit is unpredictable — it depends on market conditions."}
  ], correct:2,
  feedback:{
    correct:`<strong>Correct.</strong> This is the defining characteristic of variable costs: the <em>per-unit rate is constant</em>. At $7.00/bottle, Meridian pays $7.00 whether it makes the 1st bottle or the 100,000th (within the relevant range).<div class="formula">Total Variable Cost = Unit Variable Cost × Volume
20,000 units: $7.00 × 20,000 = $140,000
30,000 units: $7.00 × 30,000 = $210,000
Per-unit rate: $7.00 in both cases ✓</div><div class="insight"><span class="insight-icon">💡</span> Compare this to fixed costs, where the per-unit rate falls as volume rises. Understanding both behaviors lets you model how total costs — and therefore profits — change with any volume scenario.</div>`,
    wrong:`<strong>The correct answer is C.</strong> Variable cost <em>per unit</em> stays constant within the relevant range. Total variable cost varies (increases with volume), but the rate per unit does not.<div class="formula">Don't confuse these two concepts:
• Total Variable Cost → changes with volume
• Variable Cost per Unit → stays constant</div><div class="insight"><span class="insight-icon">💡</span> Answer B describes how fixed costs behave per unit — not variable costs. That distinction is critical: fixed cost per unit falls as volume rises; variable cost per unit stays flat.</div>`
  }
},

{
  chapter:"Chapter 1", chapterTitle:"Cost Fundamentals", type:"mcq",
  label:"MIXED COSTS",
  question:"The factory electricity bill is $8,000/month at zero production and increases by $0.15 per bottle produced. How should this cost be classified?",
  context:"Understanding cost behavior is critical before building any CVP model. Electricity powers both facility systems (always on) and manufacturing equipment (variable with output).",
  answers:[
    {letter:"A",text:"Purely fixed — the $8,000 base makes it a fixed cost."},
    {letter:"B",text:"Purely variable — any cost that changes with production is variable."},
    {letter:"C",text:"Mixed (semi-variable) — it has both a fixed component and a variable component."},
    {letter:"D",text:"Step cost — it jumps in fixed increments at certain production thresholds."}
  ], correct:2,
  feedback:{
    correct:`<strong>Well done.</strong> Mixed costs — sometimes called semi-variable costs — contain both a fixed portion and a variable portion. Electricity is the classic example:<div class="formula">Total Electricity = $8,000 (fixed) + $0.15 × Units Produced
At 20,000 units: $8,000 + $3,000 = $11,000
At 40,000 units: $8,000 + $6,000 = $14,000</div>Accountants separate mixed costs using the <strong>High-Low Method</strong> or <strong>regression analysis</strong> — splitting the fixed and variable components so each can be modeled correctly.<div class="insight"><span class="insight-icon">💡</span> Misclassifying a mixed cost as purely fixed or purely variable will systematically distort your projections at every volume level. Always check whether a cost has a baseline plus a usage-driven component.</div>`,
    wrong:`<strong>The correct answer is C — Mixed cost.</strong> This bill can't be purely fixed (it changes with volume) or purely variable (it has a $8,000 base even at zero production).<div class="formula">General Form: y = Fixed Component + (Variable Rate × Activity)
Electricity: y = $8,000 + $0.15x</div><div class="insight"><span class="insight-icon">💡</span> Mixed costs are everywhere in practice: phone bills, salesperson salaries (base + commission), equipment maintenance. Recognizing them prevents forecasting errors that compound as volume scales.</div>`
  }
},

{
  chapter:"Chapter 1", chapterTitle:"Cost Fundamentals", type:"mcq",
  label:"CONTRIBUTION MARGIN",
  question:"Meridian sells each water bottle for <strong>$22.00</strong>. Variable costs are <strong>$7.00/unit</strong>. What is the contribution margin per unit — and what does it tell you?",
  context:"The CFO: 'Before we talk about break-even, we need to know what each sale actually contributes toward covering our fixed costs and generating profit.'",
  answers:[
    {letter:"A",text:"$15.00 — the amount each unit contributes toward covering fixed costs and generating profit."},
    {letter:"B",text:"$7.00 — the variable cost of each unit sold."},
    {letter:"C",text:"$22.00 — the full selling price before any costs are deducted."},
    {letter:"D",text:"$15.00 — the net profit per unit after all costs, fixed and variable."}
  ], correct:0,
  feedback:{
    correct:`<strong>Correct — and the distinction vs. answer D matters enormously.</strong> The contribution margin is <em>not</em> net profit per unit. It's what each sale contributes toward fixed costs first — only after all fixed costs are covered does the surplus become profit.<div class="formula">Contribution Margin per Unit = Selling Price − Variable Cost per Unit
= $22.00 − $7.00 = $15.00

CM Ratio = CM per Unit ÷ Selling Price = $15 ÷ $22 ≈ 68.2%</div>For every $1.00 in revenue, $0.682 goes toward fixed costs (and eventually profit). The other $0.318 reimburses variable costs.<div class="insight"><span class="insight-icon">💡</span> A high CM ratio is the signature of a scalable business — once fixed costs are covered, most of each additional revenue dollar drops to the bottom line. Think software companies vs. restaurants.</div>`,
    wrong:`<strong>The correct answer is A.</strong> CM = Selling Price − Variable Costs per Unit = $22 − $7 = $15.00. Answer D is tempting but wrong — contribution margin is <em>not</em> net profit per unit because fixed costs haven't been subtracted yet.<div class="formula">Revenue − Variable Costs = Contribution Margin
Contribution Margin − Fixed Costs = Operating Profit

$22 − $7 = $15 CM → then subtract fixed costs to get profit</div><div class="insight"><span class="insight-icon">💡</span> Thinking in CM flips the mental model: ask "how much does each sale contribute toward costs I've already committed to?" That's the CVP mindset — and it's how managers evaluate every incremental decision.</div>`
  }
},

{
  chapter:"Chapter 1", chapterTitle:"Cost Fundamentals", type:"calc",
  label:"CM RATIO CALCULATION",
  question:"Calculate Meridian's <strong>Contribution Margin (CM) Ratio</strong> as a percentage.",
  context:"The CM ratio tells you what fraction of every revenue dollar contributes toward fixed costs and profit. It's used to compute break-even in dollars and to quickly estimate profit impact from revenue changes. Selling price: $22.00. Variable cost per unit: $7.00.",
  tableData:{
    headers:["Item","Amount"],
    rows:[
      ["Selling Price","$22.00 / unit"],
      ["Variable Cost per Unit","$7.00 / unit"],
      ["Contribution Margin per Unit","$15.00 / unit"],
      ["Formula","CM per Unit ÷ Selling Price × 100"]
    ]
  },
  inputLabel:"CM Ratio (enter as a %, e.g. 68.18):",
  correctAnswer:68.18, tolerance:0.5,
  hint:"CM Ratio = (CM per Unit ÷ Selling Price) × 100 = ($15.00 ÷ $22.00) × 100",
  feedback:{
    correct:`<strong>Exactly right.</strong> The CM Ratio is one of the most useful single numbers in CVP analysis.<div class="formula">CM Ratio = CM per Unit ÷ Selling Price
= $15.00 ÷ $22.00 = 0.6818 = 68.18%</div>What it means in practice: for every $1.00 of revenue Meridian generates, $0.68 goes toward covering fixed costs and — once those are fully covered — generating profit. Only $0.32 is consumed by variable costs.<div class="insight"><span class="insight-icon">💡</span> The CM ratio is the lens you use for any revenue-based scenario: "If revenue increases by $10,000, how much does operating profit increase?" Answer: $10,000 × 68.18% = $6,818 — assuming fixed costs don't change. This is why high-CM businesses are so attractive to investors.</div>`,
    wrong:`<strong>Let's work through it:</strong><div class="formula">CM Ratio = CM per Unit ÷ Selling Price
= $15.00 ÷ $22.00 = 0.6818 = 68.18%</div>The most common mistake is dividing by variable cost instead of selling price, or forgetting to multiply by 100 to express as a percentage.<div class="insight"><span class="insight-icon">💡</span> The CM Ratio has a direct complement: the Variable Cost Ratio = 1 − CM Ratio = 31.82%. Together they must equal 100% of revenue. Knowing one tells you the other instantly — useful for quick mental math in a boardroom setting.</div>`
  }
},

{
  chapter:"Chapter 1", chapterTitle:"Cost Fundamentals", type:"calc",
  label:"VARIABLE COST TOTAL",
  question:"Meridian produced <strong>4,200 bottles</strong> in a given month. What is the <strong>total variable cost</strong> for that month?",
  context:"Variable costs are $7.00 per bottle (materials $4.20 + packaging $0.80 + labor $2.00). Understanding how to calculate total variable cost at any volume is essential before computing profit or break-even.",
  tableData:{
    headers:["Variable Cost Component","Per Unit"],
    rows:[
      ["Raw Materials","$4.20"],
      ["Packaging","$0.80"],
      ["Direct Labor","$2.00"],
      ["Total Variable Cost","$7.00"]
    ]
  },
  inputLabel:"Total Variable Cost for 4,200 bottles ($):",
  correctAnswer:29400, tolerance:0,
  hint:"Total Variable Cost = Variable Cost per Unit × Number of Units = $7.00 × 4,200",
  feedback:{
    correct:`<strong>Correct.</strong> Total variable cost scales exactly with units produced — that's what makes it "variable."<div class="formula">Total Variable Cost = VC per Unit × Units Produced
= $7.00 × 4,200 = $29,400</div>Notice that the per-unit rate stays constant at $7.00, while the total changes with volume. This is the defining trait of variable costs — and it's what separates them from fixed costs, which stay flat regardless of units.<div class="insight"><span class="insight-icon">💡</span> This calculation is the building block for the contribution margin income statement: Revenue ($22 × 4,200 = $92,400) minus Total Variable Costs ($29,400) = Total Contribution Margin ($63,000). Then subtract fixed costs to get operating profit.</div>`,
    wrong:`<strong>The formula is straightforward:</strong><div class="formula">Total Variable Cost = VC per Unit × Units Produced
= $7.00 × 4,200 = $29,400</div>A common mistake is using selling price ($22) instead of variable cost ($7) per unit — always start from the cost side. Another is adding fixed costs into the calculation, which would give total costs, not variable costs alone.<div class="insight"><span class="insight-icon">💡</span> At 4,200 units — which is above the 4,145-unit break-even — the contribution margin is $63,000 (= 4,200 × $15). Subtract the $62,167 fixed costs and you get a slim $833 operating profit. Just 55 units above break-even produces almost no profit — a reminder of why the margin of safety matters.</div>`
  }
},

{
  chapter:"Chapter 1", chapterTitle:"Cost Fundamentals", type:"calc",
  label:"OPERATING PROFIT CALCULATION",
  question:"In a month when Meridian sells <strong>5,100 units</strong>, what is the <strong>monthly operating profit</strong>?",
  context:"Operating profit (also called operating income) is calculated on the contribution margin income statement: Revenue − Variable Costs = Contribution Margin, then CM − Fixed Costs = Operating Profit. Fixed costs: $62,167/month. Selling price: $22.00. Variable cost: $7.00. CM per unit: $15.00.",
  tableData:{
    headers:["Item","Calculation","Amount"],
    rows:[
      ["Revenue","5,100 × $22.00","$112,200"],
      ["Variable Costs","5,100 × $7.00","$35,700"],
      ["Contribution Margin","5,100 × $15.00","$76,500"],
      ["Fixed Costs","(given)","$62,167"],
      ["Operating Profit","CM − Fixed Costs","?"]
    ]
  },
  inputLabel:"Monthly Operating Profit ($):",
  correctAnswer:14333, tolerance:5,
  hint:"Operating Profit = (Units × CM per Unit) − Fixed Costs = (5,100 × $15.00) − $62,167",
  feedback:{
    correct:`<strong>Exactly right.</strong><div class="formula">Revenue:          5,100 × $22.00  =  $112,200
Variable Costs:   5,100 ×  $7.00  =   $35,700
                               ────────────
Contribution Margin:           =   $76,500
Fixed Costs:                   =  ($62,167)
                               ────────────
Operating Profit:              =   $14,333 ✓</div>At 5,100 units — 955 units above break-even — Meridian earns $14,333. Each of those 955 "above break-even" units adds exactly $15.00 of profit: 955 × $15 = $14,325 ≈ $14,333 (rounding). This confirms the logic: once break-even is crossed, every additional unit contributes pure profit equal to the CM per unit.<div class="insight"><span class="insight-icon">💡</span> Compare to 6,200 units: profit = (6,200 × $15) − $62,167 = $30,833. The difference: 1,100 more units × $15 = $16,500 more profit. This proportional relationship between incremental volume and incremental profit is the core insight of CVP analysis.</div>`,
    wrong:`<strong>Let's build it step by step:</strong><div class="formula">Revenue:          5,100 × $22.00  =  $112,200
Variable Costs:   5,100 ×  $7.00  =   $35,700
Contribution Margin:               =   $76,500
Fixed Costs:                       =  ($62,167)
Operating Profit:                  =   $14,333</div>Common mistakes: forgetting to subtract fixed costs, or using total revenue instead of contribution margin as the starting point for profit.<div class="insight"><span class="insight-icon">💡</span> The contribution margin income statement format (Revenue → CM → Operating Profit) is the preferred format for internal management reporting because it makes the volume-profit relationship transparent. The traditional format (Gross Profit → Operating Profit) obscures this relationship by mixing fixed and variable manufacturing costs in COGS.</div>`
  }
},

// ─────────────────────────────────────────
//  CHAPTER 2: BREAK-EVEN & CVP  (Q8–14)
// ─────────────────────────────────────────

{
  chapter:"Chapter 2", chapterTitle:"Break-Even & CVP Analysis", type:"calc",
  label:"BREAK-EVEN UNITS",
  question:"Calculate Meridian's <strong>break-even point in units</strong> per month.",
  context:"Monthly fixed costs: Factory rent $45,000 + Executive salaries $9,167 + Depreciation $5,000 + Marketing retainer $3,000 = $62,167 total. Selling price: $22.00. Variable cost: $7.00/unit. Contribution margin: $15.00/unit.",
  tableData:{
    headers:["Item","Amount"],
    rows:[
      ["Selling Price","$22.00 / unit"],
      ["Variable Cost","$7.00 / unit"],
      ["Contribution Margin","$15.00 / unit"],
      ["Total Fixed Costs (monthly)","$62,167"]
    ]
  },
  inputLabel:"Break-Even Point (units/month):",
  correctAnswer:4145, tolerance:5,
  hint:"Break-Even Units = Total Fixed Costs ÷ Contribution Margin per Unit = $62,167 ÷ $15.00",
  feedback:{
    correct:`<strong>Exactly right.</strong> At 4,145 units per month, Meridian's total contribution margin exactly equals its fixed costs — it neither profits nor loses.<div class="formula">Break-Even Units = Fixed Costs ÷ CM per Unit
= $62,167 ÷ $15.00 = 4,144.47 → round up to 4,145 units

Verification: 4,145 × $15.00 = $62,175 ≥ $62,167 ✓</div>Below 4,145 units → operating loss. Above 4,145 → every additional bottle adds $15.00 of pure profit (fixed costs already covered).<div class="insight"><span class="insight-icon">💡</span> Always round <em>up</em> on break-even units — you need to fully cover fixed costs, so you can't sell a fraction of a bottle. This small detail matters in exams and real-world financial modeling.</div>`,
    wrong:`<strong>Let's work through it:</strong><div class="formula">Break-Even Units = Total Fixed Costs ÷ Contribution Margin per Unit
= $62,167 ÷ $15.00 = 4,144.47 → round up to 4,145 units</div>Common mistakes: using selling price ($22) instead of CM ($15) in the denominator, or forgetting to round up. At exactly 4,144 units you'd still be $8.33 short of covering fixed costs.<div class="insight"><span class="insight-icon">💡</span> The denominator (CM per unit) is the key lever. Raise price or cut variable costs → CM rises → break-even falls → faster path to profit. This is why pricing strategy and cost control are inseparable in CVP analysis.</div>`
  }
},

{
  chapter:"Chapter 2", chapterTitle:"Break-Even & CVP Analysis", type:"calc",
  label:"BREAK-EVEN IN DOLLARS",
  question:"The CFO wants the break-even point expressed in <strong>monthly revenue dollars</strong>.",
  context:"Some managers find dollar-based break-even more actionable than units — it translates directly to a sales team revenue target. Monthly fixed costs: $62,167. CM Ratio: 68.18%.",
  tableData:{
    headers:["Variable","Value"],
    rows:[
      ["Total Fixed Costs","$62,167"],
      ["CM Ratio (CM ÷ Price)","68.18%"],
      ["Method","Fixed Costs ÷ CM Ratio"]
    ]
  },
  inputLabel:"Break-Even Revenue ($/month):",
  correctAnswer:91174, tolerance:200,
  hint:"Break-Even $ = Fixed Costs ÷ CM Ratio = $62,167 ÷ 0.6818",
  feedback:{
    correct:`<strong>Right on the money.</strong><div class="formula">Break-Even Revenue = Fixed Costs ÷ CM Ratio
= $62,167 ÷ 0.6818 = $91,174

Cross-check: 4,145 units × $22.00 = $91,190 ✓ (minor rounding difference)</div>Meridian's sales team needs to generate at least $91,174 in monthly revenue before the company earns a single dollar of profit.<div class="insight"><span class="insight-icon">💡</span> The CM Ratio approach is especially useful when a company sells multiple products at different price points. Instead of tracking units by SKU, you analyze total revenue against a blended CM Ratio — a powerful tool for multi-product businesses.</div>`,
    wrong:`<strong>Here's the approach:</strong><div class="formula">Break-Even Revenue = Fixed Costs ÷ CM Ratio
CM Ratio = $15.00 ÷ $22.00 = 0.6818 (68.18%)
Break-Even Revenue = $62,167 ÷ 0.6818 = $91,174</div>This should approximately equal Break-Even Units × Selling Price (4,145 × $22 = $91,190 — the small difference is rounding). Both approaches should reconcile.<div class="insight"><span class="insight-icon">💡</span> The CM Ratio approach is preferred when working with revenue targets rather than unit counts — useful for setting quarterly sales quotas or evaluating store-level performance where units aren't easily tracked.</div>`
  }
},

{
  chapter:"Chapter 2", chapterTitle:"Break-Even & CVP Analysis", type:"mcq",
  label:"MARGIN OF SAFETY",
  question:"Meridian currently sells 6,200 units/month. Break-even is 4,145 units. What is the <strong>margin of safety</strong> — and what does it mean strategically?",
  context:"The margin of safety quantifies how much sales can fall before the company hits break-even. It's a core risk metric: a larger margin means more buffer against revenue decline.",
  answers:[
    {letter:"A",text:"2,055 units or $45,210 — sales can drop by this amount before Meridian reaches break-even. It represents a ~33% safety cushion."},
    {letter:"B",text:"4,145 units — the break-even itself is the margin of safety."},
    {letter:"C",text:"6,200 units — total current sales define the safety margin."},
    {letter:"D",text:"2,055 units, but it's meaningless — only break-even matters, not the distance from it."}
  ], correct:0,
  feedback:{
    correct:`<strong>Correct.</strong> The margin of safety is the cushion between current sales and break-even:<div class="formula">Margin of Safety (Units) = Actual Sales − Break-Even Sales
= 6,200 − 4,145 = 2,055 units

Margin of Safety ($) = 2,055 × $22.00 = $45,210

Margin of Safety % = 2,055 ÷ 6,200 = 33.1%</div>Sales could decline by one-third before Meridian starts losing money.<div class="insight"><span class="insight-icon">💡</span> During the pandemic, many hospitality businesses discovered margins of safety below 10% — a single month of revenue decline wiped out profitability. The margin of safety is your early warning system, not just an accounting metric. Boards and lenders use it to gauge financial resilience.</div>`,
    wrong:`<strong>The correct answer is A.</strong> Margin of Safety = Actual Sales − Break-Even Sales.<div class="formula">= 6,200 − 4,145 = 2,055 units
= 2,055 × $22 = $45,210 in revenue
As a %: 2,055 ÷ 6,200 = 33.1%</div>Answer D is wrong — the margin of safety is critical for risk assessment. Ignoring how much room exists before a loss is how companies get caught off-guard by demand downturns.<div class="insight"><span class="insight-icon">💡</span> A startup with $1M in sales and a $950K break-even has almost no margin of safety — that's a risky investment profile even if the business is technically profitable.</div>`
  }
},

{
  chapter:"Chapter 2", chapterTitle:"Break-Even & CVP Analysis", type:"mcq",
  label:"TARGET PROFIT ANALYSIS",
  question:"The CFO sets a monthly operating profit target of <strong>$30,000</strong>. How many units must Meridian sell to hit that target?",
  context:"Target profit analysis extends break-even thinking: instead of finding the volume to cover fixed costs alone, you find the volume to cover fixed costs plus a specified profit amount.",
  answers:[
    {letter:"A",text:"4,145 units — the break-even point already achieves this."},
    {letter:"B",text:"6,145 units — add the profit target to fixed costs, then divide by CM per unit."},
    {letter:"C",text:"2,000 units — only profit divided by CM is needed."},
    {letter:"D",text:"8,478 units — dividing the revenue target by CM ratio."}
  ], correct:1,
  feedback:{
    correct:`<strong>Correct.</strong> Target profit analysis simply adds the desired profit to fixed costs in the numerator:<div class="formula">Target Volume = (Fixed Costs + Target Profit) ÷ CM per Unit
= ($62,167 + $30,000) ÷ $15.00
= $92,167 ÷ $15.00
= 6,144.5 → round up to 6,145 units

Verify: 6,145 × $15 = $92,175 − $62,167 = $30,008 ≈ $30,000 ✓</div><div class="insight"><span class="insight-icon">💡</span> This formula is the foundation of sales target-setting. The sales VP doesn't arbitrarily pick a quota — the CFO calculates the volume required to hit the profit plan, and that becomes the target. Now you know where quotas come from.</div>`,
    wrong:`<strong>The correct answer is B — 6,145 units.</strong> The key is including the target profit in the formula:<div class="formula">Target Volume = (Fixed Costs + Target Profit) ÷ CM per Unit
= ($62,167 + $30,000) ÷ $15.00 = 6,145 units</div>Answer A is wrong — break-even produces $0 profit, not $30,000. Answer C ignores that fixed costs must be covered first. The logic: after break-even (4,145 units), each additional unit adds $15 of profit. You need 2,000 more units: 2,000 × $15 = $30,000.<div class="insight"><span class="insight-icon">💡</span> You can verify: 6,145 − 4,145 = 2,000 units above break-even × $15 CM = $30,000 profit. The math always checks out — use this cross-check on exams to confirm your answer.</div>`
  }
},

{
  chapter:"Chapter 2", chapterTitle:"Break-Even & CVP Analysis", type:"mcq",
  label:"COST-VOLUME-PROFIT (CVP) GRAPH",
  question:"On a Cost-Volume-Profit (CVP) graph — which plots total revenue and total cost against volume — where is the <strong>break-even point</strong> located?",
  context:"CVP graphs are powerful visual tools. The x-axis shows units produced/sold; the y-axis shows dollars. Two lines are plotted: Total Revenue (starting at the origin) and Total Cost (starting above the origin at the fixed cost level). The visual immediately reveals loss zones, the break-even point, and profit zones.",
  answers:[
    {letter:"A",text:"Where the total revenue line intersects the variable cost line."},
    {letter:"B",text:"Where the total revenue line intersects the total cost line — the point where profit = $0."},
    {letter:"C",text:"Where the fixed cost line intersects the variable cost line."},
    {letter:"D",text:"Where the total contribution margin line reaches its maximum value."}
  ], correct:1,
  feedback:{
    correct:`<strong>Correct.</strong> The break-even point on a CVP graph is the intersection of the total revenue line and the total cost line:<div class="formula">Total Revenue Line:   y = $22 × Units (starts at origin)
Total Cost Line:      y = $62,167 + $7 × Units (starts above origin)

At break-even: $22x = $62,167 + $7x
              $15x = $62,167
               x   = 4,145 units</div>Left of the intersection → loss zone (costs exceed revenue). Right → profit zone (revenue exceeds costs). The vertical distance between the two lines at any volume equals operating profit (or loss).<div class="insight"><span class="insight-icon">💡</span> The CVP graph is a powerful executive communication tool — a single slide conveys the volume-profit relationship to non-accountants instantly. The steeper the revenue line relative to the cost line, the faster profits accumulate post break-even. That slope difference is exactly the CM per unit — geometry reflecting the math.</div>`,
    wrong:`<strong>The correct answer is B.</strong> On a CVP graph, break-even is where Total Revenue = Total Cost — the intersection of those two lines. At that point, profit is exactly $0.<div class="formula">Left of intersection  → Loss zone (costs > revenue)
Intersection point    → Break-even (profit = $0)
Right of intersection → Profit zone (revenue > costs)</div>Answer A is wrong — variable cost and revenue lines don't determine break-even alone. Answer C involves fixed costs, which appear as a horizontal line on the graph. Answer D is incorrect — contribution margin keeps growing with volume in a CVP graph; it doesn't reach a "maximum."<div class="insight"><span class="insight-icon">💡</span> The total cost line starts at the fixed cost level on the y-axis (not at the origin) because even at zero units, fixed costs are still incurred. This visual explains why businesses can lose money even before making a single sale.</div>`
  }
},

{
  chapter:"Chapter 2", chapterTitle:"Break-Even & CVP Analysis", type:"mcq",
  label:"DIRECT vs. INDIRECT COSTS",
  question:"Meridian's production manager earns $80,000/year overseeing the bottle-making line. A portion of factory rent is also allocated to the bottling department. How should these two costs be classified?",
  context:"Before making product-level decisions, managers must understand which costs can be traced to a cost object (direct) and which must be allocated using a cost driver (indirect/overhead). Both types appear in the cost model.",
  answers:[
    {letter:"A",text:"Both are direct costs — they both relate to manufacturing and can be traced to the product."},
    {letter:"B",text:"The manager's salary is a direct fixed cost (traceable to the department); allocated rent is an indirect cost (requires a cost driver for allocation)."},
    {letter:"C",text:"Both are indirect costs — any cost that doesn't vary with volume must be allocated."},
    {letter:"D",text:"The manager's salary is a period cost; rent is a product cost."}
  ], correct:1,
  feedback:{
    correct:`<strong>Correct.</strong> The key is traceability — can the cost be directly traced to a specific cost object without allocation?<div class="formula">Direct Cost:   Can be traced to a cost object with precision
               → Production manager salary ($80,000/year)
               → Directly tied to the bottling department

Indirect Cost: Requires allocation using a cost driver
               → Factory rent allocated to bottling
               → Also serves other departments; must be split</div>The manager's salary is a direct fixed cost to the department. The rent requires an allocation key (e.g., floor space used) — making it indirect.<div class="insight"><span class="insight-icon">💡</span> Direct costs give you cleaner decision data. Indirect (allocated) costs should never be the sole basis for a drop-or-keep decision — the allocation disappears from the product but not from the company. Always distinguish between costs that truly go away vs. costs that get redistributed.</div>`,
    wrong:`<strong>The correct answer is B.</strong> Traceability is the deciding factor — not whether the cost is fixed or variable.<div class="formula">Direct:   Traceable to a cost object without allocation
          → Production manager salary

Indirect: Requires a cost driver to allocate
          → Factory rent shared across departments</div>Answer A is wrong — rent requires allocation, so it's indirect. Answer C is wrong — direct vs. indirect is about traceability, not variability. Answer D misidentifies both: the manager's salary is a product/manufacturing cost (inventoried), not a period cost.<div class="insight"><span class="insight-icon">💡</span> This distinction matters enormously for product costing accuracy. Including too many allocated indirect costs in a product's cost profile can make profitable products appear unprofitable — and vice versa.</div>`
  }
},

{
  chapter:"Chapter 2", chapterTitle:"Break-Even & CVP Analysis", type:"mcq",
  label:"INTERPRETING PROFIT ZONES",
  question:"At the current sales level of 6,200 units/month, Meridian earns $30,833 in operating profit. If sales drop to 4,000 units next month, what is the financial outcome — and by how much?",
  context:"Understanding profit zones is central to CVP analysis. Break-even is 4,145 units. At any volume below that, the company operates at a loss — not because it is doing anything wrong operationally, but because fixed costs cannot be fully covered.",
  answers:[
    {letter:"A",text:"Operating profit of $120 — 4,000 units is close enough to break-even to still be profitable."},
    {letter:"B",text:"Operating loss of $2,167 — at 4,000 units, contribution margin falls $2,167 short of fixed costs."},
    {letter:"C",text:"Operating profit of $60,000 — revenue of 4,000 × $22 = $88,000 exceeds variable costs."},
    {letter:"D",text:"Break-even — 4,000 units is approximately the break-even level."}
  ], correct:1,
  feedback:{
    correct:`<strong>Correct.</strong> At 4,000 units, Meridian falls below its break-even of 4,145 units and operates at a loss:<div class="formula">Revenue:          4,000 × $22.00  =  $88,000
Variable Costs:   4,000 ×  $7.00  =  $28,000
                                     ────────
Contribution Margin: 4,000 × $15  =  $60,000
Fixed Costs:                       = ($62,167)
                                     ────────
Operating LOSS:                    =  ($2,167)</div>The company is 145 units short of break-even. Each missing unit = $15 of unrecovered CM, so 145 × $15 = $2,175 ≈ $2,167 (rounding confirms the logic).<div class="insight"><span class="insight-icon">💡</span> This illustrates the concept of "loss zone" on the CVP graph — not a catastrophic failure, but a predictable consequence of operating below the fixed cost threshold. The margin of safety (2,055 units above break-even at normal volume) is exactly the buffer that prevents this scenario during normal operations.</div>`,
    wrong:`<strong>The correct answer is B — an operating loss of $2,167.</strong> 4,000 units is below the 4,145-unit break-even:<div class="formula">CM at 4,000 units = 4,000 × $15 = $60,000
Fixed Costs = $62,167
Operating Loss = $60,000 − $62,167 = ($2,167)</div>Answer A incorrectly calculates near-break-even as a profit. Answer C confuses revenue with operating profit — revenue exceeding variable costs only gives you CM, not profit. Fixed costs must also be covered. Answer D is off — 4,000 ≠ 4,145.<div class="insight"><span class="insight-icon">💡</span> This is precisely why the margin of safety matters: Meridian's 33% cushion means sales must fall by more than 2,055 units before losses begin. Without that buffer, any minor downturn immediately hits the bottom line.</div>`
  }
},

// ─────────────────────────────────────────
//  CHAPTER 3: STRATEGIC DECISIONS  (Q15–18)
// ─────────────────────────────────────────

{
  chapter:"Chapter 3", chapterTitle:"Strategic Decision-Making", type:"mcq",
  label:"PRICING STRATEGY",
  question:"A competitor cuts prices. The VP of Sales proposes Meridian drop its price to <strong>$19.00/unit</strong>. How does this affect break-even — and is it the right move?",
  context:"Current state: Price $22, Variable Cost $7, CM $15, Fixed Costs $62,167, Break-even 4,145 units. Proposed: Price drops to $19. Variable costs unchanged at $7.",
  answers:[
    {letter:"A",text:"Break-even drops to ~3,600 units — lower price drives volume and reduces break-even."},
    {letter:"B",text:"Break-even rises to ~5,164 units — lower price reduces CM, requiring more unit sales to cover the same fixed costs."},
    {letter:"C",text:"Break-even stays the same — price doesn't affect the break-even calculation, only volume does."},
    {letter:"D",text:"Break-even drops slightly — the volume increase from lower prices compensates for the smaller margin."}
  ], correct:1,
  feedback:{
    correct:`<strong>Excellent strategic thinking.</strong> Lower price → lower CM per unit → higher break-even:<div class="formula">New CM per Unit = $19.00 − $7.00 = $12.00
New Break-Even = $62,167 ÷ $12.00 = 5,180.6 → 5,181 units

Old Break-Even: 4,145 units
Additional units needed just to break even: +1,036 units (a 25% increase)</div>The $3 price cut requires Meridian to sell 1,036 more units before it earns a single dollar of profit. The question then becomes: will that demand increase materialize? That's an elasticity question — and it's often a losing bet.<div class="insight"><span class="insight-icon">💡</span> This is why price wars are so dangerous: both competitors cut margins and raise their break-even points simultaneously. CVP analysis makes this trade-off crystal clear — it turns a "feel-good" sales initiative into an accountable financial decision.</div>`,
    wrong:`<strong>The correct answer is B.</strong> Price decreases reduce CM, which pushes break-even higher — the opposite of what most salespeople intuit.<div class="formula">New CM = $19 − $7 = $12 (was $15)
New Break-Even = $62,167 ÷ $12 = 5,181 units (was 4,145)
That's 1,036 more units needed just to break even.</div>Answer A is backward — lower prices never reduce break-even unless variable costs fall proportionally. Answer C is wrong — selling price is the primary driver of CM, which directly determines break-even.<div class="insight"><span class="insight-icon">💡</span> The CFO's counter-move: before any price cut, calculate how many additional units are needed to maintain the same profit level. Often that number is unrealistically large — exactly what CVP analysis reveals in seconds.</div>`
  }
},

{
  chapter:"Chapter 3", chapterTitle:"Strategic Decision-Making", type:"mcq",
  label:"OPERATING LEVERAGE",
  question:"Meridian considers automating production: replace $3/unit variable labor with a $50,000/month equipment lease (fixed cost). At current volume of 6,200 units/month, what is the correct assessment?",
  context:"Operating leverage describes the relationship between fixed and variable costs. Higher fixed costs = higher operating leverage = profits grow faster above break-even, but losses deepen faster below it. Always calculate the new break-even before deciding.",
  answers:[
    {letter:"A",text:"No — adding fixed costs always increases risk and should be avoided regardless of volume."},
    {letter:"B",text:"No — at 6,200 units, automation actually produces a net operating loss under the new cost structure."},
    {letter:"C",text:"Yes — the new structure reduces variable cost and increases CM, so it is always the better choice."},
    {letter:"D",text:"It doesn't matter — total costs are the same at any volume level."}
  ], correct:1,
  feedback:{
    correct:`<strong>Correct.</strong> This requires computing the new break-even before drawing any conclusion:<div class="formula">Current Structure (6,200 units):
  CM = $15/unit | Fixed = $62,167
  Profit = (6,200 × $15) − $62,167 = $30,833

Automated Structure:
  VC = $7 − $3 = $4/unit | CM = $22 − $4 = $18/unit
  Fixed = $62,167 + $50,000 = $112,167
  New Break-Even = $112,167 ÷ $18 = 6,232 units

At 6,200 units (current volume):
  Profit = (6,200 × $18) − $112,167 = −$567 (a LOSS)</div>At 6,200 units, automation produces a loss of $567 — Meridian is 32 units below the new break-even. The decision should only be made if sustained volume above 6,232 units is virtually certain.<div class="insight"><span class="insight-icon">💡</span> This is operating leverage at work: the automated structure earns more profit per unit above break-even, but requires a higher volume threshold to get there. Always compute the new break-even before committing to any fixed cost increase. The gain is real — but so is the risk.</div>`,
    wrong:`<strong>The correct answer is B.</strong> At 6,200 units, automation produces a net loss under the new cost structure.<div class="formula">New CM = $18/unit | New Fixed Costs = $112,167
New Break-Even = $112,167 ÷ $18 = 6,232 units

At 6,200 units:
(6,200 × $18) − $112,167 = $111,600 − $112,167 = −$567 (LOSS)</div>Answer A is overstated — fixed costs aren't inherently bad; it depends entirely on sustained volume. Answer C skips the break-even check entirely, which is the critical error managers make. Answer D is demonstrably wrong — cost structure profoundly affects profit sensitivity to volume changes.<div class="insight"><span class="insight-icon">💡</span> The right question to ask: "At what volume does automation become better than the current structure?" Answer: above 6,232 units. If Meridian is confident it will consistently sell above that level, automation is the right long-run choice. Below it, the current variable-heavy structure is safer.</div>`
  }
},

{
  chapter:"Chapter 3", chapterTitle:"Strategic Decision-Making", type:"mcq",
  label:"MULTI-PRODUCT CVP",
  question:"Meridian launches a premium bottle at $38.00 (VC: $12.00) alongside the standard at $22.00 (VC: $7.00). Products sell in a <strong>3:1 ratio</strong> (standard:premium). Fixed costs remain $62,167. What is the <strong>weighted average CM per unit</strong>?",
  context:"When a company sells multiple products, CVP analysis requires a weighted average contribution margin based on the sales mix. This weighted CM replaces the single-product CM in all break-even formulas.",
  answers:[
    {letter:"A",text:"$15.00 — only the standard product's CM, since it sells most."},
    {letter:"B",text:"$20.50 — simple (unweighted) average: ($15 + $26) ÷ 2."},
    {letter:"C",text:"$17.75 — weighted average: (3 × $15 + 1 × $26) ÷ 4."},
    {letter:"D",text:"$26.00 — only the premium product's CM, since it's most profitable per unit."}
  ], correct:2,
  feedback:{
    correct:`<strong>Excellent.</strong> Weighted average CM accounts for the proportion of each product in the sales mix:<div class="formula">Standard CM = $22 − $7 = $15 | Weight = 3/4 = 75%
Premium CM  = $38 − $12 = $26 | Weight = 1/4 = 25%

Weighted Avg CM = (3 × $15 + 1 × $26) ÷ 4
= ($45 + $26) ÷ 4 = $71 ÷ 4 = $17.75

Multi-Product Break-Even = $62,167 ÷ $17.75 = 3,503 total units
(2,627 standard + 876 premium)</div><div class="insight"><span class="insight-icon">💡</span> Sales mix is a hidden profit lever. If Meridian shifts toward premium (higher CM), break-even falls and profits rise — with no price or cost changes. This is why product mix decisions are fundamentally financial decisions, not just marketing ones.</div>`,
    wrong:`<strong>The correct answer is C — $17.75.</strong> Each product's CM must be weighted by its proportion of total sales:<div class="formula">Standard: CM $15 × 3 units = $45
Premium:  CM $26 × 1 unit  = $26
Total:    ($45 + $26) ÷ 4 units = $17.75</div>Answer A and D each use only one product's CM — ignoring the mix entirely. Answer B uses a simple average — it treats each product equally regardless of how many are sold.<div class="insight"><span class="insight-icon">💡</span> In real companies, the product/channel mix shift is often the most powerful profit improvement lever — it requires no price increases, no cost cuts, and no new investment. Just strategic selling.</div>`
  }
},

{
  chapter:"Chapter 3", chapterTitle:"Strategic Decision-Making", type:"mcq",
  label:"SENSITIVITY ANALYSIS",
  question:"Meridian's fixed costs increase by $9,000/month (a new lease). Selling price and variable cost are unchanged. What is the new break-even in units — and what does this imply?",
  context:"Sensitivity analysis tests how break-even and profit respond to changes in key assumptions. It's a standard board-level tool: 'If X changes, what happens to our bottom line?'",
  answers:[
    {letter:"A",text:"4,145 units — fixed cost changes don't affect break-even."},
    {letter:"B",text:"4,745 units — the $9,000 increase ÷ $15 CM = 600 additional units needed at break-even."},
    {letter:"C",text:"5,500 units — add $9,000 to both numerator and denominator of the break-even formula."},
    {letter:"D",text:"3,545 units — the new lease provides operational efficiency that lowers break-even."}
  ], correct:1,
  feedback:{
    correct:`<strong>Correct.</strong><div class="formula">New Fixed Costs = $62,167 + $9,000 = $71,167
New Break-Even = $71,167 ÷ $15.00 = 4,744.5 → 4,745 units

Change: 4,745 − 4,145 = 600 additional units needed

Quick check: $9,000 increase ÷ $15 CM = 600 units ✓</div>The "quick check" is a powerful mental shortcut: any fixed cost increase of $X raises the break-even by exactly $X ÷ CM per unit. No need to redo the full formula every time.<div class="insight"><span class="insight-icon">💡</span> Sensitivity analysis runs these scenarios systematically — what if rent goes up 10%? What if a sales rep is added? Each assumption change translates directly to a break-even shift. Managers who internalize this shortcut can evaluate decisions in real time without a spreadsheet.</div>`,
    wrong:`<strong>The correct answer is B — 4,745 units.</strong><div class="formula">New Fixed Costs = $62,167 + $9,000 = $71,167
New Break-Even = $71,167 ÷ $15.00 = 4,745 units

Shortcut: Fixed cost increase ÷ CM = Break-even increase
$9,000 ÷ $15 = 600 more units needed ✓</div>Answer A is wrong — fixed cost changes directly affect break-even (it's in the numerator). Answer C incorrectly changes the denominator too. Answer D confuses a lease (fixed cost) with an operational efficiency (variable cost reduction).<div class="insight"><span class="insight-icon">💡</span> The shortcut ($9,000 ÷ $15 = 600) is worth memorizing. Every $1 of additional fixed cost raises break-even by 1/CM units. At a CM of $15, it takes 67 cents of fixed cost to add one unit to the break-even requirement.</div>`
  }
},

// ─────────────────────────────────────────
//  CHAPTER 4: MANAGERIAL DECISIONS  (Q19–24)
// ─────────────────────────────────────────

{
  chapter:"Chapter 4", chapterTitle:"Managerial Decision-Making", type:"mcq",
  label:"RELEVANT COSTS — MAKE VS. BUY",
  question:"Meridian currently makes bottle caps in-house. A supplier offers to provide them at $0.40/cap. In-house costs per cap: direct materials $0.15, direct labor $0.10, variable overhead $0.05, allocated fixed overhead $0.18. What is the <strong>relevant cost to make</strong> — and what should Meridian do?",
  context:"Relevant cost analysis focuses only on costs that actually differ between alternatives. Sunk costs and allocated fixed overhead that won't change regardless of the decision are irrelevant — including them leads to poor choices.",
  answers:[
    {letter:"A",text:"$0.48/cap — full in-house cost including allocated fixed overhead. Outsource since $0.48 > $0.40."},
    {letter:"B",text:"$0.30/cap — only avoidable variable costs. Continue making in-house since $0.30 < $0.40."},
    {letter:"C",text:"$0.18/cap — only the fixed overhead portion, since that's the 'extra' cost."},
    {letter:"D",text:"$0.40/cap — the outsource price anchors the comparison, so costs are irrelevant."}
  ], correct:1,
  feedback:{
    correct:`<strong>Correct.</strong> The $0.18 allocated fixed overhead will exist regardless of whether Meridian makes or buys — it's irrelevant to this decision.<div class="formula">Relevant (avoidable) costs to make:
  Direct Materials:    $0.15
  Direct Labor:        $0.10
  Variable Overhead:   $0.05
                      ───────
  Total Relevant Cost: $0.30 per cap

Make ($0.30) vs. Buy ($0.40) → MAKE saves $0.10/cap

At 6,200 units/month: $0.10 × 6,200 = $620/month saved</div>Unless making caps ties up capacity that could generate more than $620/month in value elsewhere, in-house production is the right choice.<div class="insight"><span class="insight-icon">💡</span> Allocated fixed overhead is the #1 trap in make-vs-buy analysis. It overstates the cost to make, pushing companies toward outsourcing decisions that destroy rather than create value. The mantra: only include costs that actually go away if you change the decision.</div>`,
    wrong:`<strong>The correct answer is B — $0.30/cap.</strong> Only variable costs are avoidable: $0.15 + $0.10 + $0.05 = $0.30. The $0.18 allocated fixed overhead won't disappear if Meridian outsources — it just gets reallocated.<div class="formula">Relevant Cost to Make = Avoidable Costs Only = $0.30
Buy Price = $0.40
Decision: MAKE (saves $0.10/cap)</div>Answer A includes allocated overhead — a fatal error in relevant cost analysis. The overhead stays whether or not the caps are made in-house.<div class="insight"><span class="insight-icon">💡</span> Qualitative factors also matter: outsourcing creates supplier dependency, quality control challenges, and potential IP exposure. The $0.10/cap saving must be weighed against those strategic risks — but the financial baseline is $0.30 vs. $0.40.</div>`
  }
},

{
  chapter:"Chapter 4", chapterTitle:"Managerial Decision-Making", type:"mcq",
  label:"SPECIAL ORDER DECISION",
  question:"A hotel chain offers to buy 3,000 bottles/month at $14/bottle — well below Meridian's standard $22 price. Meridian has 10,000 units of idle capacity. Fixed costs won't change. Should Meridian accept?",
  context:"Special order analysis evaluates one-time, non-recurring orders at non-standard prices. The financial question is: does the order generate positive incremental contribution? The key comparison is price vs. variable cost — not price vs. regular selling price.",
  answers:[
    {letter:"A",text:"No — accepting below-standard pricing would set a dangerous precedent and undermine regular customers' trust."},
    {letter:"B",text:"No — the $14 price is below the $22 selling price, making the order unprofitable."},
    {letter:"C",text:"Yes — at $14, the order contributes $7/unit above variable cost ($7.00), generating $21,000 of incremental profit."},
    {letter:"D",text:"Yes — but only if Meridian can negotiate $22/bottle in the next contract with the hotel."}
  ], correct:2,
  feedback:{
    correct:`<strong>Correct.</strong> With idle capacity and unchanged fixed costs, any price above variable cost is financially accretive:<div class="formula">Special Order CM = $14.00 − $7.00 = $7.00/unit
Total Incremental Profit = 3,000 × $7.00 = $21,000/month

Regular market profit (unchanged):
  (6,200 × $15) − $62,167 = $30,833
New total profit:
  $30,833 + $21,000 = $51,833</div>Fixed costs are incurred regardless — the $21,000 is pure incremental profit from otherwise idle capacity.<div class="insight"><span class="insight-icon">💡</span> Answer A raises a legitimate qualitative concern: if regular customers discover the discount, pricing power could be damaged. Special orders should be isolated from regular markets (different geography, private-label branding, or distinct customer type) to protect pricing integrity.</div>`,
    wrong:`<strong>The correct answer is C.</strong> The relevant comparison is special price vs. variable cost — not vs. the regular selling price. Fixed costs don't change, so they're irrelevant to the incremental decision.<div class="formula">Financial test: Special Price ($14) > Variable Cost ($7)?
YES → Accept → Incremental CM = $7/unit × 3,000 = $21,000</div>Answer B compares to the regular price ($22) — which is the wrong benchmark when fixed costs don't change. Answer A raises a real strategic concern but the financial answer is to accept.<div class="insight"><span class="insight-icon">💡</span> This logic explains why airlines discount last-minute seats, hotels offer late-night rates, and theaters sell half-price tickets at the box office — the marginal cost of an empty seat or bottle is essentially zero. Any contribution above variable cost improves total profitability.</div>`
  }
},

{
  chapter:"Chapter 4", chapterTitle:"Managerial Decision-Making", type:"mcq",
  label:"SEGMENT ELIMINATION",
  question:"Meridian's online retail channel shows a $15,000 operating loss: it contributes $85,000 CM but is allocated $100,000 in corporate fixed overhead. The $100,000 won't decrease if the channel is eliminated. Should it be dropped?",
  context:"Segment elimination decisions require identifying which costs are truly avoidable. Allocated common fixed costs that won't disappear are irrelevant to the drop-or-keep decision — just as in make-vs-buy analysis.",
  answers:[
    {letter:"A",text:"Yes — eliminating a loss-making segment always improves overall company profitability."},
    {letter:"B",text:"No — the online channel contributes $85,000 toward common fixed costs that persist regardless. Eliminating it worsens total profit by $85,000."},
    {letter:"C",text:"Yes — the allocated overhead will be redistributed to more profitable segments, improving their margins."},
    {letter:"D",text:"No — but only because the channel might improve in the future."}
  ], correct:1,
  feedback:{
    correct:`<strong>Correct.</strong> The $100,000 allocated overhead is irrelevant — it stays no matter what:<div class="formula">If Online Channel is KEPT:
  Company CM = $85,000 (channel) + other segments
  Allocated overhead: still $100,000 elsewhere in total

If Online Channel is DROPPED:
  $85,000 CM disappears
  $100,000 overhead: still exists, now fully on other segments
  Net impact: company profit falls by $85,000</div>Dropping the channel creates a real $85,000 hole in total company profit. The "loss" is an accounting artifact of the overhead allocation — not an economic reality.<div class="insight"><span class="insight-icon">💡</span> This is one of the most important management accounting insights: a segment showing an accounting loss may be making a real economic contribution. Always ask "what costs actually disappear if this segment is eliminated?" If the answer is few or none, keep the segment.</div>`,
    wrong:`<strong>The correct answer is B — keep the channel.</strong> The $100,000 overhead doesn't go away — it gets reallocated to remaining segments, increasing their cost burdens.<div class="formula">Drop Online → lose $85,000 CM → lose $85,000 in company profit
The $100,000 overhead persists → net result: company worse off by $85,000</div>Answer A reflects a dangerous intuition: "loss = eliminate." The correct framework asks "what costs actually go away?" — and allocated overhead rarely does.<div class="insight"><span class="insight-icon">💡</span> Many companies have discovered this lesson too late: they eliminate "loss-making" divisions only to find that the remaining divisions' unit costs rise (due to unabsorbed overhead) and total profit falls. The real question is always avoidable cost vs. avoidable revenue.</div>`
  }
},

{
  chapter:"Chapter 4", chapterTitle:"Managerial Decision-Making", type:"mcq",
  label:"CONSTRAINED RESOURCE DECISION",
  question:"Meridian has 10,000 machine-hours available per month. Standard bottles require 1 hour each (CM: $15). Premium bottles require 2.5 hours each (CM: $26). To maximize profit given the constraint, which should Meridian prioritize?",
  context:"When a binding constraint exists — a scarce resource limiting total output — the optimal product mix is determined by contribution margin per unit of the constraint, NOT contribution margin per unit of product. This is one of the most counterintuitive insights in managerial accounting.",
  answers:[
    {letter:"A",text:"Premium bottles — they generate a higher CM per unit ($26 vs. $15), so they're clearly more profitable."},
    {letter:"B",text:"Standard bottles — they generate a higher CM per machine-hour ($15.00 vs. $10.40), making them more valuable given the scarcity."},
    {letter:"C",text:"An equal mix — diversification minimizes risk and maximizes customer satisfaction."},
    {letter:"D",text:"Whichever sells the most volume — the product with the highest demand should always be prioritized."}
  ], correct:1,
  feedback:{
    correct:`<strong>Correct — this is one of the most important and counterintuitive insights in cost accounting.</strong><div class="formula">Standard:  CM per Machine-Hour = $15.00 ÷ 1.0 hr  = $15.00/hr
Premium:   CM per Machine-Hour = $26.00 ÷ 2.5 hrs = $10.40/hr

At 10,000 machine-hours — all Standard:
  10,000 hrs ÷ 1 hr × $15 = $150,000 total CM

At 10,000 machine-hours — all Premium:
  10,000 hrs ÷ 2.5 hrs × $26 = $104,000 total CM

Standard generates $46,000 MORE profit despite lower unit CM</div>Standard wins — not because it's a better product per unit, but because it uses the scarce resource more efficiently.<div class="insight"><span class="insight-icon">💡</span> This principle is called the Theory of Constraints (TOC). Toyota, lean manufacturers, and operations researchers all center their scheduling logic around it: identify the binding constraint, maximize its throughput, subordinate everything else to that goal. Every production scheduling decision flows from this insight.</div>`,
    wrong:`<strong>The correct answer is B — Standard bottles.</strong> When a constraint exists, optimize CM per unit of the scarce resource — not CM per unit of product.<div class="formula">Standard: $15 ÷ 1 hr  = $15.00/machine-hour
Premium:  $26 ÷ 2.5 hrs = $10.40/machine-hour

Standard produces 44% more profit per scarce hour</div>Answer A is the most common mistake: higher unit CM doesn't help if it consumes disproportionately more of the scarce resource.<div class="insight"><span class="insight-icon">💡</span> This explains why Toyota obsesses over cycle time reduction on bottleneck machines — every minute saved on the constraint directly increases profit. It also explains why airlines prioritize routes by revenue per seat-mile, not just revenue per seat.</div>`
  }
},

{
  chapter:"Chapter 4", chapterTitle:"Managerial Decision-Making", type:"mcq",
  label:"SELL OR PROCESS FURTHER",
  question:"Meridian has 2,000 partially completed bottles that can be sold as-is for $9/unit, or processed further at a cost of $3/unit to sell as finished bottles for $15/unit. What should Meridian do — and why?",
  context:"Sell-or-process-further decisions require ignoring all joint costs and prior costs already incurred (sunk). Only the incremental revenue and incremental cost from this decision point forward are relevant.",
  answers:[
    {letter:"A",text:"Sell now at $9 — additional processing adds risk of defects and ties up machinery."},
    {letter:"B",text:"Process further — incremental revenue of $6/unit exceeds incremental cost of $3/unit, yielding $3/unit of additional profit."},
    {letter:"C",text:"Process further only if the original manufacturing cost was below $9/unit."},
    {letter:"D",text:"Sell now — the $3 processing cost eats into the $9 current value, reducing net proceeds."}
  ], correct:1,
  feedback:{
    correct:`<strong>Correct.</strong> The original manufacturing cost is sunk — completely irrelevant. Only incremental economics matter from this point forward:<div class="formula">Incremental Revenue = Finished price − Current price
= $15.00 − $9.00 = $6.00/unit

Incremental Cost of processing = $3.00/unit

Incremental Profit = $6.00 − $3.00 = $3.00/unit

Total additional profit: 2,000 × $3.00 = $6,000</div>Always process further when incremental revenue > incremental cost. The original cost is gone regardless of the choice.<div class="insight"><span class="insight-icon">💡</span> Oil refineries, chemical plants, and food processors face this decision constantly — crude oil → gasoline → jet fuel, wheat → flour → bread. Each value-adding step is evaluated purely on incremental economics. Factoring in sunk costs leads to systematic underinvestment in value-creating activities.</div>`,
    wrong:`<strong>The correct answer is B — process further.</strong> The sunk cost (original manufacturing) is irrelevant:<div class="formula">Incremental Revenue: $15 − $9 = $6/unit
Incremental Cost: $3/unit
Net benefit: $6 − $3 = $3/unit → always process further

Total gain: 2,000 × $3 = $6,000</div>Answer C introduces the original manufacturing cost — a classic sunk cost error. Answer D incorrectly compares the $3 processing cost to the $9 selling price instead of to the $6 of incremental revenue.<div class="insight"><span class="insight-icon">💡</span> The rule is clean and universal: if the incremental revenue from further processing exceeds the incremental cost → process further. Sunk costs cannot be recovered regardless of the forward decision — including them leads to leaving profitable value on the table.</div>`
  }
},

{
  chapter:"Chapter 4", chapterTitle:"Managerial Decision-Making", type:"mcq",
  label:"OPPORTUNITY COST",
  question:"Meridian's CFO is evaluating a new product line that will use factory space currently rented out to a tenant for $40,000/month. This $40,000 must be included in the financial analysis as:",
  context:"Opportunity cost is the value of the best alternative foregone when a resource is committed to a specific use. It never appears in traditional accounting records — but it is just as real as any cash cost, and omitting it leads to systematically overestimating the attractiveness of new initiatives.",
  answers:[
    {letter:"A",text:"A sunk cost — the factory was already built, so its current use represents a past decision."},
    {letter:"B",text:"An opportunity cost — the $40,000 in foregone rental income is a real economic cost of using the space for the new product."},
    {letter:"C",text:"Irrelevant — opportunity costs are theoretical constructs that don't belong in financial analysis."},
    {letter:"D",text:"A fixed overhead allocation — it should be spread across all products using a cost driver."}
  ], correct:1,
  feedback:{
    correct:`<strong>Correct.</strong> The $40,000/month in foregone rent is a concrete opportunity cost — the value of the next-best use of the factory space that Meridian is giving up.<div class="formula">New Product Line Analysis (with opportunity cost):
  Revenue from new product:           $X
  Less: Variable costs:              ($Y)
  Less: Incremental fixed costs:     ($Z)
  Less: Opportunity cost (rent):  ($40,000)
                                    ──────
  True Incremental Profit:            $?</div>Omitting the $40,000 makes the new product look $40,000/month more attractive than it actually is — a systematic bias toward new initiatives that don't clear the real hurdle.<div class="insight"><span class="insight-icon">💡</span> Warren Buffett's concept of a "hurdle rate" is fundamentally about opportunity cost: any investment must beat the return you could earn on the best alternative. If the new product can't outperform the $40,000/month rent, keep subletting — the economically rational choice, even if accounting records don't flag it.</div>`,
    wrong:`<strong>The correct answer is B — opportunity cost.</strong> The $40,000/month is the concrete value of an alternative Meridian is forgoing. It must be included in any honest financial analysis of the new product.<div class="formula">Without opportunity cost: New product looks $40,000/month better than reality.
With opportunity cost: True hurdle rate is correctly captured.</div>Answer A confuses sunk costs (past, unchangeable decisions like building the factory) with opportunity costs (current, foregone alternatives). Answer C is wrong — opportunity costs are real and must be quantified. Answer D (overhead allocation) is a completely different concept.<div class="insight"><span class="insight-icon">💡</span> Opportunity costs are invisible to accounting systems — they don't appear in any journal entry. This is why rigorous financial analysis always supplements accounting records with economic reasoning. The CFO who only reads ledgers will consistently misallocate resources.</div>`
  }
}

]; // end questions

// ══════════════════════════════════════════════════════════════
//  STATE
// ══════════════════════════════════════════════════════════════
let currentQ = 0;
let score = 0;
let answered = false;
let reviewMode = false; // true when user goes Back to a prior question
let answeredState = []; // store answered status per question so Back works cleanly

const chaptersConfig = [
  { key:"Chapter 1", title:"Cost Fundamentals",       desc:"Fixed, variable, mixed costs; CM per unit; CM ratio; operating profit" },
  { key:"Chapter 2", title:"Break-Even & CVP Analysis", desc:"BEP units, BEP$, margin of safety, target profit, CVP graph" },
  { key:"Chapter 3", title:"Strategic Decision-Making", desc:"Pricing, leverage, multi-product CVP, sensitivity analysis" },
  { key:"Chapter 4", title:"Managerial Decision-Making", desc:"Relevant costs, special orders, segment analysis, constraints, sell-or-process, opportunity cost" }
];

function initChapterScores() {
  const cs = {};
  chaptersConfig.forEach(c => cs[c.key] = { earned: 0, total: 0 });
  return cs;
}
let chapterScores = initChapterScores();

// ══════════════════════════════════════════════════════════════
//  NAVIGATION
// ══════════════════════════════════════════════════════════════
function showScreen(id) {
  document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
  document.getElementById(id).classList.add('active');
  window.scrollTo(0, 0);
}

function startGame() {
  currentQ = 0; score = 0; answered = false; reviewMode = false;
  answeredState = questions.map(() => ({ answered: false, wasCorrect: false, recorded: false }));
  chapterScores = initChapterScores();
  showScreen('screen-game');
  renderQuestion();
}

function showIntro() { showScreen('screen-intro'); }
function restartGame() { startGame(); }

function goBack() {
  if (currentQ <= 0) return;
  currentQ--;
  reviewMode = true;
  renderQuestion();
  window.scrollTo({ top: 0, behavior: 'smooth' });
}

function nextQuestion() {
  reviewMode = false;
  currentQ++;
  if (currentQ >= questions.length) {
    showResults();
  } else {
    renderQuestion();
    window.scrollTo({ top: 0, behavior: 'smooth' });
  }
}

// ══════════════════════════════════════════════════════════════
//  RENDER QUESTION
// ══════════════════════════════════════════════════════════════
function renderQuestion() {
  answered = false;
  const q = questions[currentQ];
  const st = answeredState[currentQ];
  const pct = (currentQ / questions.length) * 100;

  document.getElementById('progress-bar').style.width = pct + '%';
  document.getElementById('progress-text').textContent = `Question ${currentQ + 1} of ${questions.length}`;
  document.getElementById('score-display').textContent = `Score: ${score} pts`;
  document.getElementById('btn-next').classList.remove('visible');
  document.getElementById('btn-back').classList.toggle('visible', currentQ > 0);

  // Review banner
  const rb = document.getElementById('review-banner');
  rb.classList.toggle('visible', reviewMode);

  // Track total per chapter (only count first visit)
  if (!st.recorded) {
    chapterScores[q.chapter].total += 10;
    st.recorded = true;
  }

  let html = `
    <div class="chapter-header">
      <span class="chapter-badge">${q.chapter.toUpperCase()}</span>
      <span class="chapter-title-text">${q.chapterTitle}</span>
    </div>
    <div class="card">
      <div class="card-accent"></div>
      <div class="scenario-label">📌 ${q.label}</div>
      <div class="question-text">${q.question}</div>
      <div class="context-text">${q.context}</div>
  `;

  if (q.tableData) {
    html += `<table class="data-table"><thead><tr>`;
    q.tableData.headers.forEach(h => html += `<th>${h}</th>`);
    html += `</tr></thead><tbody>`;
    q.tableData.rows.forEach(r => {
      html += `<tr>`;
      r.forEach((cell, i) => html += `<td class="${i > 0 ? 'num' : ''}">${cell}</td>`);
      html += `</tr>`;
    });
    html += `</tbody></table>`;
  }

  if (q.type === 'mcq') {
    html += `<div class="answers-grid">`;
    q.answers.forEach((a, i) => {
      html += `<button class="answer-btn" onclick="checkMCQ(${i})" id="ans-${i}">
        <span class="answer-letter">Option ${a.letter}</span>${a.text}
      </button>`;
    });
    html += `</div>`;
  } else if (q.type === 'calc') {
    html += `
      <div class="hint-box" id="hint-box">💡 Hint: ${q.hint}</div>
      <div class="calc-input-wrap">
        <div class="input-group">
          <label class="input-label">${q.inputLabel}</label>
          <input type="number" step="any" class="calc-input" id="calc-answer" placeholder="Enter your answer"
            onkeydown="if(event.key==='Enter') checkCalc()">
        </div>
        <div>
          <button class="btn-primary" onclick="checkCalc()" style="padding:12px 24px;">Check →</button>
        </div>
      </div>
      <div style="margin-top:10px;">
        <button class="btn-outline" id="hint-btn" onclick="showHint()" style="font-size:13px; padding:8px 16px;">Need a hint?</button>
      </div>
    `;
  }

  html += `<div class="feedback-box" id="feedback-box"></div>`;
  html += `</div>`;
  document.getElementById('question-area').innerHTML = html;

  // If this question was previously answered (Back button scenario), replay the result
  if (st.answered) {
    answered = true;
    if (q.type === 'mcq') {
      document.querySelectorAll('.answer-btn').forEach((btn, i) => {
        btn.disabled = true;
        if (i === q.correct) btn.classList.add('correct');
      });
    }
    showFeedback(st.wasCorrect, q);
    document.getElementById('btn-next').classList.add('visible');
  }
}

// ══════════════════════════════════════════════════════════════
//  CHECK ANSWERS
// ══════════════════════════════════════════════════════════════
function checkMCQ(selected) {
  if (answered) return;
  answered = true;
  const q = questions[currentQ];
  const st = answeredState[currentQ];
  const isCorrect = selected === q.correct;

  document.querySelectorAll('.answer-btn').forEach((btn, i) => {
    btn.disabled = true;
    if (i === q.correct) btn.classList.add('correct');
    else if (i === selected && !isCorrect) btn.classList.add('wrong');
  });

  // Only award points on first answer (not review mode)
  if (isCorrect && !st.answered && !reviewMode) {
    score += 10;
    chapterScores[q.chapter].earned += 10;
  }

  st.answered = true;
  st.wasCorrect = isCorrect;

  showFeedback(isCorrect, q);
  document.getElementById('score-display').textContent = `Score: ${score} pts`;
  document.getElementById('btn-next').classList.add('visible');
}

function checkCalc() {
  if (answered) return;
  const q = questions[currentQ];
  const st = answeredState[currentQ];
  const inp = document.getElementById('calc-answer');
  const val = parseFloat(inp.value.replace(/,/g, ''));
  if (isNaN(val)) { inp.style.borderColor = 'var(--auburn)'; return; }

  answered = true;
  const isCorrect = Math.abs(val - q.correctAnswer) <= q.tolerance;
  inp.classList.add(isCorrect ? 'correct' : 'wrong');
  inp.disabled = true;

  const hintBtn = document.getElementById('hint-btn');
  if (hintBtn) hintBtn.disabled = true;

  if (isCorrect && !st.answered && !reviewMode) {
    score += 10;
    chapterScores[q.chapter].earned += 10;
  }

  st.answered = true;
  st.wasCorrect = isCorrect;

  showFeedback(isCorrect, q);
  document.getElementById('score-display').textContent = `Score: ${score} pts`;
  document.getElementById('btn-next').classList.add('visible');
}

function showHint() {
  const hb = document.getElementById('hint-box');
  if (hb) hb.classList.add('visible');
}

function showFeedback(isCorrect, q) {
  const fb = document.getElementById('feedback-box');
  const status = isCorrect ? 'correct' : 'wrong';
  const icon = isCorrect ? '✓' : '✗';
  const headline = isCorrect ? 'CORRECT' : 'NOT QUITE';
  fb.innerHTML = `
    <div class="feedback-header ${status}">${icon} ${headline}</div>
    <div class="feedback-body ${status}">${isCorrect ? q.feedback.correct : q.feedback.wrong}</div>
  `;
  fb.classList.add('visible');
  setTimeout(() => fb.scrollIntoView({ behavior: 'smooth', block: 'nearest' }), 100);
}

// ══════════════════════════════════════════════════════════════
//  RESULTS
// ══════════════════════════════════════════════════════════════
function showResults() {
  showScreen('screen-results');
  const total = questions.length * 10;
  const pct = score / total;

  let grade, gradeClass, headline, takeaway;
  if (pct >= 0.90) {
    grade = 'A'; gradeClass = 'A';
    headline = "Outstanding. The CFO is impressed — you're ready for the boardroom.";
    takeaway = "You demonstrated mastery across all four chapters of MBA Cost Accounting. From cost classification and CVP analysis through strategic pricing decisions and managerial relevant cost frameworks, you applied accounting concepts with precision and sound judgment. These skills compound over your career — every pricing discussion, investment decision, and budget conversation will be sharper for understanding them deeply.";
  } else if (pct >= 0.75) {
    grade = 'B'; gradeClass = 'B';
    headline = "Solid work. Strong fundamentals — a few areas worth revisiting.";
    takeaway = "You have a strong command of the core CVP framework. Review the chapters where you lost points, particularly the managerial decision-making questions in Chapter 4. Relevant cost analysis, constrained resource optimization, and opportunity cost are high-frequency topics in both exams and real financial roles — they reward precision and clear conceptual framing.";
  } else if (pct >= 0.60) {
    grade = 'C'; gradeClass = 'C';
    headline = "Building the right foundations — the mechanics need more reinforcement.";
    takeaway = "You're developing the right intuitions, but the quantitative application needs more practice. Focus on the core formulas first — CM per unit, CM ratio, break-even units and dollars, operating profit — until the logic (not just the math) is clear. Then revisit the decision-making questions in Chapters 3 and 4; they become intuitive once the foundations are solid.";
  } else {
    grade = 'D'; gradeClass = 'D';
    headline = "The CFO recommends a deeper review before the board presentation.";
    takeaway = "CVP analysis connects every unit sold to profit and loss — it's the financial language of operational decision-making. Start with cost classification (fixed vs. variable vs. mixed), build to contribution margin, then break-even. Each concept is a stepping stone. Use the Back button to revisit every question you missed — the detailed feedback is specifically designed to close those gaps.";
  }

  document.getElementById('result-grade').textContent = grade;
  document.getElementById('result-grade').className = `results-grade ${gradeClass}`;
  document.getElementById('result-score').textContent = `${score} / ${total} points`;
  document.getElementById('result-headline').textContent = headline;
  document.getElementById('result-takeaway').textContent = takeaway;

  let breakdownHTML = '';
  chaptersConfig.forEach(ch => {
    const s = chapterScores[ch.key];
    const p = s.total > 0 ? s.earned / s.total : 0;
    const cls = p >= 0.9 ? 'perfect' : p >= 0.7 ? 'good' : p >= 0.5 ? 'ok' : 'low';
    breakdownHTML += `
      <div class="result-chapter">
        <div class="chap-label">${ch.key.toUpperCase()}</div>
        <div class="chap-title">${ch.title}</div>
        <div style="font-size:12px; color:var(--muted); margin-bottom:8px;">${ch.desc}</div>
        <div class="chap-score ${cls}">${s.earned} / ${s.total} pts</div>
        <div class="mini-bar"><div class="mini-fill" style="width:${p * 100}%"></div></div>
      </div>`;
  });
  document.getElementById('result-breakdown').innerHTML = breakdownHTML;
}
</script>
</body>
</html>
