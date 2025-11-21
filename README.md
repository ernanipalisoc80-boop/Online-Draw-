<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Online Draw — Random Picker</title>
  <style>
    :root{--bg:#0f172a;--card:#0b1220;--accent:#06b6d4;--muted:#94a3b8;--glass: rgba(255,255,255,0.03)}
    *{box-sizing:border-box;font-family:Inter,ui-sans-serif,system-ui,Arial}
    html,body{height:100%;margin:0;background:linear-gradient(180deg,#071029 0%, #071124 60%);color:#e6eef6}
    .container{max-width:900px;margin:32px auto;padding:24px;background:linear-gradient(180deg, rgba(255,255,255,0.02), transparent);border-radius:12px;border:1px solid rgba(255,255,255,0.04);box-shadow:0 6px 30px rgba(2,6,23,0.6)}
    h1{margin:0 0 12px;font-size:20px}
    .grid{display:grid;grid-template-columns:1fr 360px;gap:20px}
    textarea{width:100%;height:360px;padding:12px;border-radius:8px;border:1px solid rgba(255,255,255,0.04);background:var(--card);color:inherit;resize:vertical}
    .controls{background:var(--glass);padding:16px;border-radius:8px;border:1px solid rgba(255,255,255,0.03)}
    label{display:block;margin-bottom:8px;color:var(--muted);font-size:13px}
    input[type=number]{width:72px;padding:8px;border-radius:6px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:inherit}
    button{padding:10px 14px;border-radius:8px;border:0;background:var(--accent);color:#012;cursor:pointer;font-weight:600}
    button.secondary{background:transparent;border:1px solid rgba(255,255,255,0.06);color:var(--muted);font-weight:600}
    .display{height:220px;display:flex;align-items:center;justify-content:center;border-radius:8px;background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));border:1px solid rgba(255,255,255,0.03);margin-bottom:12px}
    .name{font-size:28px;letter-spacing:1px}
    .winners{margin-top:12px;max-height:240px;overflow:auto;padding:10px;border-radius:8px;background:rgba(255,255,255,0.01);border:1px solid rgba(255,255,255,0.02)}
    .winner-item{padding:6px 8px;border-radius:6px;margin-bottom:6px;background:linear-gradient(90deg, rgba(255,255,255,0.02), transparent)}
    footer{margin-top:12px;color:var(--muted);font-size:13px}
    .small{font-size:13px;color:var(--muted)}
  </style>
</head>
<body>
  <div class="container">
    <h1>Online Draw — Random Picker</h1>
    <div class="grid">
      <div>
        <label for="names">Names (one per line)</label>
        <textarea id="names"></textarea>
        <div style="display:flex;gap:8px;margin-top:8px">
          <button id="loadDefault">Load provided list</button>
          <button id="clear" class="secondary">Clear</button>
          <button id="shuffle" class="secondary">Shuffle list</button>
          <button id="export" class="secondary">Export CSV</button>
        </div>
        <div class="small" style="margin-top:10px">Tip: You can paste or edit names directly in the box. Names are treated exactly as typed.</div>
      </div>
      <div class="controls">
        <div class="display"><div class="name" id="current">—</div></div>
        <div style="display:flex;gap:8px;margin-bottom:8px">
          <button id="start">Start</button>
          <button id="stop" class="secondary" disabled>Stop</button>
          <button id="pick" class="secondary">Pick Now</button>
        </div>
        <div style="display:flex;align-items:center;gap:8px;margin-bottom:8px">
          <label for="numWinners" style="margin:0">Winners:</label>
          <input id="numWinners" type="number" min="1" value="1" />
          <button id="pickMultiple" class="secondary">Pick N</button>
        </div>
        <div style="display:flex;gap:8px;margin-bottom:8px">
          <button id="reset" class="secondary">Reset</button>
          <button id="copy" class="secondary">Copy winners</button>
        </div>
        <div class="winners" id="winners"></div>
        <footer>Made for quick online draws. Refresh to fully reset.</footer>
      </div>
    </div>
  </div>

  <script>
    // Provided names exactly as the user supplied
    const provided = `ARANETA
LISING
ABLOLA
ALDEA
ANTONIO
ΒΕΝJAΜΙΝ
CALIWARA
CARATING
DEMILLO
DUMLAO
ECULLADA
GALANG
GALVEZ
LANDINGIN
LIMPIN
LOTA
LUSTADO
MANOS
MENDOZA
MONTENEGRO
NARVAEZ
NUQUI
PALISOC
GARBO
LIMATEN
FAT
ALFONSO
DONIZA
HILVANO
SERRANO
QUINIMON
VALENCIA`;

    const namesEl = document.getElementById('names');
    const loadDefaultBtn = document.getElementById('loadDefault');
    const clearBtn = document.getElementById('clear');
    const shuffleBtn = document.getElementById('shuffle');
    const exportBtn = document.getElementById('export');

    const startBtn = document.getElementById('start');
    const stopBtn = document.getElementById('stop');
    const pickBtn = document.getElementById('pick');
    const pickMultipleBtn = document.getElementById('pickMultiple');
    const numWinnersInput = document.getElementById('numWinners');
    const currentEl = document.getElementById('current');
    const winnersEl = document.getElementById('winners');
    const resetBtn = document.getElementById('reset');
    const copyBtn = document.getElementById('copy');

    let pool = [];
    let running = false;
    let ticker = null;
    let lastPicked = [];

    function parseNames(text){
      return text.split('\n').map(s=>s.trim()).filter(Boolean);
    }

    function refreshPool(){
      pool = parseNames(namesEl.value);
    }

    function pickRandom(){
      if(pool.length===0) return null;
      const i = Math.floor(Math.random()*pool.length);
      return pool[i];
    }

    function startSpin(){
      refreshPool();
      if(pool.length===0) { alert('No names in the list.'); return; }
      running = true;
      startBtn.disabled = true;
      stopBtn.disabled = false;
      ticker = setInterval(()=>{ currentEl.textContent = pickRandom() }, 60);
    }

    function stopSpin(){
      if(!running) return;
      clearInterval(ticker);
      running = false;
      startBtn.disabled = false;
      stopBtn.disabled = true;
      // final pick is whatever is currently shown; ensure it's a real name
      const final = currentEl.textContent;
      if(final && pool.includes(final)) addWinner(final);
    }

    function addWinner(name){
      lastPicked.push(name);
      // remove from pool so it won't be picked again
      pool = pool.filter(n=>n!==name);
      renderWinners();
    }

    function renderWinners(){
      winnersEl.innerHTML = '';
      lastPicked.forEach((w,i)=>{
        const d = document.createElement('div'); d.className='winner-item';
        d.textContent = `${i+1}. ${w}`;
        winnersEl.appendChild(d);
      })
    }

    function pickNow(){
      refreshPool();
      if(pool.length===0) { alert('No names available to pick.'); return; }
      const pick = pickRandom();
      currentEl.textContent = pick;
      addWinner(pick);
    }

    function pickN(n){
      refreshPool();
      if(pool.length===0) { alert('No names available.'); return; }
      n = Math.min(n, pool.length);
      // pick without replacement
      const picks = [];
      const working = pool.slice();
      for(let i=0;i<n;i++){
        const idx = Math.floor(Math.random()*working.length);
        picks.push(working.splice(idx,1)[0]);
      }
      // show picks one-by-one quickly
      let i=0;
      const interval = setInterval(()=>{
        currentEl.textContent = picks[i];
        i++;
        if(i>=picks.length){ clearInterval(interval); picks.forEach(addWinner); }
      }, 400);
    }

    function shuffleArray(){
      refreshPool(); if(pool.length===0) return; let a=pool; for(let i=a.length-1;i>0;i--){const j=Math.floor(Math.random()*(i+1));[a[i],a[j]]=[a[j],a[i]];} namesEl.value=a.join('\n'); refreshPool();
    }

    function exportCSV(){
      refreshPool(); const all = parseNames(namesEl.value);
      const csv = all.map(s=>`"${s.replace(/"/g,'""')}"`).join('\n');
      const blob = new Blob([csv],{type:'text/csv;charset=utf-8;'});
      const url = URL.createObjectURL(blob);
      const a=document.createElement('a'); a.href=url; a.download='names.csv'; document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url);
    }

    function resetAll(){
      lastPicked = [];
      renderWinners();
      currentEl.textContent='—';
      refreshPool();
    }

    function copyWinners(){
      if(lastPicked.length===0) { alert('No winners yet.'); return; }
      navigator.clipboard.writeText(lastPicked.join('\n')).then(()=>{ alert('Winners copied to clipboard.') }).catch(()=>{ alert('Unable to copy.'); });
    }

    // wire up
    loadDefaultBtn.addEventListener('click', ()=>{ namesEl.value = provided; refreshPool(); resetAll(); });
    clearBtn.addEventListener('click', ()=>{ namesEl.value=''; refreshPool(); resetAll(); });
    shuffleBtn.addEventListener('click', ()=>{ shuffleArray(); resetAll(); });
    exportBtn.addEventListener('click', exportCSV);

    startBtn.addEventListener('click', startSpin);
    stopBtn.addEventListener('click', stopSpin);
    pickBtn.addEventListener('click', pickNow);
    pickMultipleBtn.addEventListener('click', ()=>{ const n = parseInt(numWinnersInput.value)||1; pickN(n); });
    resetBtn.addEventListener('click', resetAll);
    copyBtn.addEventListener('click', copyWinners);

    // initialize
    namesEl.value = provided;
    refreshPool();
  </script>
</body>
</html>
