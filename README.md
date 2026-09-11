<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Task Rail</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=IBM+Plex+Sans:wght@400;500;600&family=IBM+Plex+Mono:wght@400;500&display=swap');

  :root{
    --bg:#14171A;
    --panel:#1D2125;
    --panel-header:#20242A;
    --border:#2C3136;
    --text:#ECEAE5;
    --muted:#8B9096;
    --amber:#E2A33B;
    --moss:#5B9279;
    --danger:#C1614F;
  }

  *{ box-sizing:border-box; }

  body{
    margin:0;
    background:var(--bg);
    color:var(--text);
    font-family:'IBM Plex Sans', sans-serif;
    padding:32px 24px 64px;
    min-height:100vh;
  }

  .wrap{ max-width:980px; margin:0 auto; }

  header.top{
    display:flex;
    align-items:baseline;
    justify-content:space-between;
    border-bottom:1px solid var(--border);
    padding-bottom:16px;
    margin-bottom:24px;
    flex-wrap:wrap;
    gap:8px;
  }

  header.top h1{
    font-size:22px;
    font-weight:600;
    margin:0;
    letter-spacing:-0.01em;
  }

  header.top .tally{
    font-family:'IBM Plex Mono', monospace;
    font-size:13px;
    color:var(--muted);
  }

  .board{
    display:grid;
    grid-template-columns:repeat(3, 1fr);
    gap:16px;
  }

  @media (max-width:760px){
    .board{ grid-template-columns:1fr; }
  }

  .panel{
    background:var(--panel);
    border:1px solid var(--border);
    display:flex;
    flex-direction:column;
    min-height:220px;
    transition:border-color .15s ease;
  }

  .panel.dragover{ border-color:var(--amber); }

  .panel-head{
    background:var(--panel-header);
    border-bottom:1px solid var(--border);
    padding:12px 14px;
    display:flex;
    align-items:center;
    justify-content:space-between;
  }

  .panel-head .name{
    font-weight:600;
    font-size:14px;
  }

  .panel-head .count{
    font-family:'IBM Plex Mono', monospace;
    font-size:12px;
    color:var(--muted);
    border:1px solid var(--border);
    padding:1px 6px;
    transition:transform .12s ease;
  }

  .count.bump{ transform:scale(1.18); color:var(--amber); }

  .panel[data-status="done"] .panel-head .name{ color:var(--moss); }

  .add-row{
    display:flex;
    border-bottom:1px solid var(--border);
  }

  .add-row input{
    flex:1;
    background:transparent;
    border:none;
    color:var(--text);
    font-family:'IBM Plex Sans', sans-serif;
    font-size:13px;
    padding:10px 12px;
    outline:none;
  }

  .add-row input::placeholder{ color:var(--muted); }

  .add-row button{
    background:none;
    border:none;
    border-left:1px solid var(--border);
    color:var(--muted);
    font-family:'IBM Plex Mono', monospace;
    font-size:12px;
    padding:0 14px;
    cursor:pointer;
  }

  .add-row button:hover{ color:var(--amber); }

  .cards{
    padding:10px;
    display:flex;
    flex-direction:column;
    gap:8px;
    flex:1;
  }

  .empty{
    color:var(--muted);
    font-size:12px;
    font-family:'IBM Plex Mono', monospace;
    padding:8px 4px;
  }

  .card{
    background:var(--bg);
    border:1px solid var(--border);
    border-left:3px solid var(--amber);
    padding:10px 10px 10px 12px;
    cursor:grab;
    transition:transform .12s ease, box-shadow .12s ease, opacity .12s ease;
  }

  .panel[data-status="progress"] .card{ border-left-color:#7FA6C9; }
  .panel[data-status="done"] .card{ border-left-color:var(--moss); opacity:.72; }

  .card.dragging{ opacity:.35; }

  .card .row{
    display:flex;
    align-items:flex-start;
    justify-content:space-between;
    gap:8px;
  }

  .card .text{
    font-size:13.5px;
    line-height:1.4;
  }

  .card .meta{
    font-family:'IBM Plex Mono', monospace;
    font-size:10.5px;
    color:var(--muted);
    margin-top:6px;
  }

  .card .actions{
    display:flex;
    gap:6px;
    flex-shrink:0;
  }

  .card .actions button{
    background:none;
    border:1px solid var(--border);
    color:var(--muted);
    font-family:'IBM Plex Mono', monospace;
    font-size:11px;
    width:22px;
    height:22px;
    cursor:pointer;
    line-height:1;
  }

  .card .actions button:hover{ color:var(--text); border-color:var(--muted); }
  .card .actions button.del:hover{ color:var(--danger); border-color:var(--danger); }

  @media (prefers-reduced-motion: reduce){
    *{ transition:none !important; }
  }
</style>
</head>
<body>

<div class="wrap">
  <header class="top">
    <h1>Task Rail</h1>
    <div class="tally" id="tally">0 open</div>
  </header>

  <div class="board" id="board"></div>
</div>

<script>
  // ---- State ----
  let seq = 1;
  const panels = [
    { status: 'backlog',  name: 'Backlog' },
    { status: 'progress', name: 'In Progress' },
    { status: 'done',     name: 'Done' }
  ];

  let tasks = [
    { id: seq++, text: 'Wire up the panel event listeners', status: 'progress' },
    { id: seq++, text: 'Sketch the board layout',           status: 'done' },
    { id: seq++, text: 'Add drag-and-drop between panels',  status: 'backlog' },
  ];

  const nextStatus = { backlog: 'progress', progress: 'done', done: null };

  const board = document.getElementById('board');
  const tallyEl = document.getElementById('tally');

  // ---- Render ----
  function render(bumpStatus) {
    board.innerHTML = '';

    panels.forEach(panel => {
      const list = tasks.filter(t => t.status === panel.status);

      const panelEl = document.createElement('div');
      panelEl.className = 'panel';
      panelEl.dataset.status = panel.status;

      panelEl.innerHTML = `
        <div class="panel-head">
          <span class="name">${panel.name}</span>
          <span class="count ${bumpStatus === panel.status ? 'bump' : ''}">${list.length}</span>
        </div>
        <form class="add-row" data-status="${panel.status}">
          <input type="text" placeholder="Add to ${panel.name.toLowerCase()}…" maxlength="80" />
          <button type="submit">add</button>
        </form>
        <div class="cards"></div>
      `;

      const cardsEl = panelEl.querySelector('.cards');

      if (list.length === 0) {
        const empty = document.createElement('div');
        empty.className = 'empty';
        empty.textContent = '— nothing here —';
        cardsEl.appendChild(empty);
      }

      list.forEach(task => {
        const card = document.createElement('div');
        card.className = 'card';
        card.draggable = true;
        card.dataset.id = task.id;

        const canAdvance = nextStatus[task.status] !== null;

        card.innerHTML = `
          <div class="row">
            <div class="text">${escapeHtml(task.text)}</div>
            <div class="actions">
              ${canAdvance ? `<button class="advance" title="Move forward">›</button>` : ''}
              <button class="del" title="Delete">×</button>
            </div>
          </div>
          <div class="meta">#${String(task.id).padStart(3, '0')}</div>
        `;

        cardsEl.appendChild(card);
      });

      board.appendChild(panelEl);
    });

    const open = tasks.filter(t => t.status !== 'done').length;
    tallyEl.textContent = `${open} open`;

    if (bumpStatus) {
      setTimeout(() => {
        const c = board.querySelector(`.panel[data-status="${bumpStatus}"] .count`);
        if (c) c.classList.remove('bump');
      }, 160);
    }
  }

  function escapeHtml(str) {
    const div = document.createElement('div');
    div.textContent = str;
    return div.innerHTML;
  }

  // ---- State mutators ----
  function addTask(status, text) {
    const trimmed = text.trim();
    if (!trimmed) return;
    tasks.push({ id: seq++, text: trimmed, status });
    render(status);
  }

  function advanceTask(id) {
    const task = tasks.find(t => t.id === id);
    if (!task) return;
    const next = nextStatus[task.status];
    if (!next) return;
    task.status = next;
    render(next);
  }

  function moveTask(id, status) {
    const task = tasks.find(t => t.id === id);
    if (!task || task.status === status) return;
    task.status = status;
    render(status);
  }

  function deleteTask(id) {
    tasks = tasks.filter(t => t.id !== id);
    render();
  }

  // ---- Event delegation: form submit (add), click (advance/delete) ----
  board.addEventListener('submit', (e) => {
    e.preventDefault();
    const form = e.target.closest('form.add-row');
    if (!form) return;
    const input = form.querySelector('input');
    addTask(form.dataset.status, input.value);
    input.value = '';
    input.focus();
  });

  board.addEventListener('click', (e) => {
    const advanceBtn = e.target.closest('.advance');
    const delBtn = e.target.closest('.del');
    const card = e.target.closest('.card');
    if (!card) return;

    const id = Number(card.dataset.id);
    if (advanceBtn) advanceTask(id);
    if (delBtn) deleteTask(id);
  });

  // ---- Drag and drop between panels ----
  let draggedId = null;

  board.addEventListener('dragstart', (e) => {
    const card = e.target.closest('.card');
    if (!card) return;
    draggedId = Number(card.dataset.id);
    card.classList.add('dragging');
    e.dataTransfer.effectAllowed = 'move';
  });

  board.addEventListener('dragend', (e) => {
    const card = e.target.closest('.card');
    if (card) card.classList.remove('dragging');
    board.querySelectorAll('.panel.dragover').forEach(p => p.classList.remove('dragover'));
  });

  board.addEventListener('dragover', (e) => {
    const panel = e.target.closest('.panel');
    if (!panel) return;
    e.preventDefault();
    e.dataTransfer.dropEffect = 'move';
    board.querySelectorAll('.panel').forEach(p => p.classList.remove('dragover'));
    panel.classList.add('dragover');
  });
