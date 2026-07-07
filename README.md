[README.md](https://github.com/user-attachments/files/29724442/README.md)
# Lot Ticket — Daily Ledger

A single-file web app for pricing Pokémon card buy-ins and trades on the fly.

- Set your cash/trade % tiers by price range, plus a flat bulk rate for cheap cards
- Log cards coming in (from a customer) and going out (to a customer) in one transaction
- Get an instant "you owe them" / "they owe you" number
- Save transactions through the day, then export a CSV with everything for your records

No build step, no dependencies — it's just `index.html`. Open it directly in a browser or deploy it anywhere that serves static files (GitHub Pages, Vercel, Netlify, etc).


[index.html](https://github.com/user-attachments/files/29724444/index.html)
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Lot Ticket — Daily Ledger</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@500;700&family=JetBrains+Mono:wght@400;500;700&family=Inter:wght@400;500;600&display=swap');

  :root{
    --ink:#1B2430; --ink-2:#242F3F;
    --parchment:#F5EFE1; --parchment-line:#C9BFA6;
    --cash:#3F7859; --cash-dim:#5C8F72;
    --trade:#8C3A2B; --trade-dim:#A85242;
    --chalk:#EDE7D9; --chalk-dim:#B9B3A4;
  }
  *{ box-sizing:border-box; }
  body{ margin:0; background:var(--ink); color:var(--chalk); font-family:'Inter',sans-serif; padding:32px 16px 80px; }
  .wrap{ max-width:840px; margin:0 auto; }
  h1{ font-family:'Space Grotesk',sans-serif; font-weight:700; font-size:26px; letter-spacing:.02em; text-transform:uppercase; margin:0 0 4px; }
  .subtitle{ color:var(--chalk-dim); font-size:13px; margin-bottom:24px; }

  section{ background:var(--ink-2); border:1px solid #33405480; border-radius:4px; padding:18px 20px; margin-bottom:18px; }
  section h2{ font-family:'Space Grotesk',sans-serif; font-size:12px; letter-spacing:.12em; text-transform:uppercase; color:var(--chalk-dim); margin:0 0 12px; display:flex; align-items:center; gap:8px; }
  .h2-cash::before{ content:''; width:4px; height:4px; background:var(--cash-dim); border-radius:50%; }
  .h2-trade::before{ content:''; width:4px; height:4px; background:var(--trade-dim); border-radius:50%; }
  .h2-neutral::before{ content:''; width:4px; height:4px; background:var(--chalk-dim); border-radius:50%; }

  table{ width:100%; border-collapse:collapse; font-size:13px; }
  th{ text-align:left; font-family:'JetBrains Mono',monospace; font-weight:500; font-size:10px; text-transform:uppercase; letter-spacing:.05em; color:var(--chalk-dim); padding:0 6px 6px; border-bottom:1px solid #C9BFA633; }
  td{ padding:4px 6px; vertical-align:middle; }

  input[type=number], input[type=text]{ background:var(--ink); border:1px solid #3A4759; color:var(--chalk); font-family:'JetBrains Mono',monospace; font-size:13px; padding:5px 7px; border-radius:3px; width:100%; }
  input:focus{ outline:none; border-color:var(--cash-dim); }
  .tier-input{ width:64px; }
  .price-input{ width:90px; }

  .btn{ font-family:'Space Grotesk',sans-serif; font-weight:700; font-size:11px; text-transform:uppercase; letter-spacing:.04em; border:none; border-radius:3px; padding:8px 14px; cursor:pointer; transition:opacity .15s; }
  .btn:hover{ opacity:.85; }
  .btn-in{ background:var(--cash); color:var(--chalk); }
  .btn-out{ background:var(--trade); color:var(--chalk); }
  .btn-neutral{ background:#3A4759; color:var(--chalk); }
  .btn-save{ background:var(--chalk); color:var(--ink); font-size:12px; padding:10px 18px; }
  .btn-remove{ background:none; border:1px solid #3A4759; color:var(--chalk-dim); padding:5px 9px; font-size:10px; }
  .btn-remove:hover{ border-color:var(--trade-dim); color:var(--trade-dim); }

  .toggle-group{ display:flex; border-radius:3px; overflow:hidden; border:1px solid #3A4759; width:fit-content; }
  .toggle-opt{ padding:5px 10px; font-family:'JetBrains Mono',monospace; font-size:11px; cursor:pointer; background:var(--ink); color:var(--chalk-dim); user-select:none; white-space:nowrap; }
  .toggle-opt.active-cash{ background:var(--cash); color:var(--chalk); }
  .toggle-opt.active-trade{ background:var(--trade); color:var(--chalk); }

  .offer-cell{ text-align:center; }
  .offer-box{ padding:6px 4px; border:1px solid #3A4759; border-radius:3px; font-family:'JetBrains Mono',monospace; font-size:13px; cursor:pointer; color:var(--chalk-dim); text-align:center; transition:opacity .1s; }
  .offer-box:hover{ opacity:.8; }
  .offer-box.selected-cash{ background:var(--cash); color:var(--chalk); border-color:var(--cash); font-weight:700; }
  .offer-box.selected-trade{ background:var(--trade); color:var(--chalk); border-color:var(--trade); font-weight:700; }

  .bulk-row{ display:flex; align-items:center; gap:14px; flex-wrap:wrap; font-family:'JetBrains Mono',monospace; font-size:12px; color:var(--chalk-dim); margin-top:10px; padding-top:12px; border-top:1px dashed #3A4759; }
  .bulk-row label{ display:flex; align-items:center; gap:6px; }
  .bulk-row input[type=checkbox]{ accent-color:var(--cash); width:14px; height:14px; }

  .badge{ font-family:'JetBrains Mono',monospace; font-size:9px; text-transform:uppercase; padding:2px 6px; border-radius:2px; letter-spacing:.04em; white-space:nowrap; }
  .badge-bulk{ background:#3A475940; color:var(--chalk-dim); }
  .badge-tier{ background:#3F785930; color:var(--cash-dim); }
  .mono{ font-family:'JetBrains Mono',monospace; }
  .num{ text-align:right; }
  .dim{ color:var(--chalk-dim); }
  .note-input{ width:120px; }
  .cardnum-input{ width:70px; }

  .txn-note{ margin-bottom:12px; }
  .txn-note input{ font-family:'Inter',sans-serif; }

  /* mini summary ticket inside the transaction builder */
  .mini-ticket{ background:#00000030; border:1px dashed #3A4759; border-radius:3px; padding:12px 14px; margin-top:14px; font-family:'JetBrains Mono',monospace; font-size:12px; }
  .mini-line{ display:flex; justify-content:space-between; padding:2px 0; }
  .mini-line.final{ border-top:1px dashed #3A4759; margin-top:6px; padding-top:8px; font-size:14px; font-weight:700; }
  .final.owe-us .value{ color:var(--cash-dim); }
  .final.owe-them .value{ color:var(--trade-dim); }

  .save-row{ display:flex; justify-content:flex-end; margin-top:14px; }

  /* receipt-style ticket used for day totals, torn edge signature element */
  .ticket{ background:var(--parchment); color:var(--ink); border-radius:2px; position:relative; padding:22px 20px 18px; margin-top:6px; }
  .ticket::before{ content:''; position:absolute; top:-9px; left:0; right:0; height:10px; background:linear-gradient(135deg, var(--ink) 50%, transparent 50%) 0 0/12px 12px repeat-x; }
  .ticket-title{ font-family:'Space Grotesk',sans-serif; font-weight:700; font-size:12px; text-transform:uppercase; letter-spacing:.1em; border-bottom:1px dashed #00000033; padding-bottom:8px; margin-bottom:10px; display:flex; justify-content:space-between; }
  .ticket-line{ display:flex; justify-content:space-between; font-family:'JetBrains Mono',monospace; font-size:13px; padding:3px 0; }
  .ticket-line.total{ border-top:1px dashed #00000033; margin-top:6px; padding-top:8px; font-weight:700; font-size:14px; }
  .ticket-line .label{ color:#4a4030; }

  .day-list{ margin-top:14px; }
  .day-row{ display:flex; justify-content:space-between; align-items:center; padding:8px 0; border-bottom:1px solid #33405450; font-size:12px; }
  .day-row-left{ display:flex; gap:10px; align-items:center; }
  .day-row-time{ color:var(--chalk-dim); font-family:'JetBrains Mono',monospace; font-size:11px; width:56px; }
  .day-row-note{ font-family:'Inter'; }
  .day-row-amt{ font-family:'JetBrains Mono',monospace; }
  .day-row-amt.owe-us{ color:var(--cash-dim); }
  .day-row-amt.owe-them{ color:var(--trade-dim); }
  .empty-state{ color:var(--chalk-dim); font-size:12px; font-family:'JetBrains Mono',monospace; padding:10px 0; }

  .day-actions{ display:flex; gap:10px; margin-top:16px; flex-wrap:wrap; }
  .note{ font-size:11px; color:var(--chalk-dim); margin-top:8px; line-height:1.5; }
</style>
</head>
<body>
<div class="wrap">
  <h1>Lot Ticket — Daily Ledger</h1>
  <div class="subtitle">Two-sided transactions · running day totals · end-of-day export</div>

  <section>
    <h2 class="h2-neutral">Tier Table (buy-in only)</h2>
    <table>
      <thead><tr><th>Min $</th><th>Max $</th><th>Cash %</th><th>Trade %</th><th></th></tr></thead>
      <tbody id="tierBody"></tbody>
    </table>
    <button class="btn btn-neutral" id="addTier" style="margin-top:8px;">+ Add tier</button>
    <div class="bulk-row">
      <label><input type="checkbox" id="bulkEnabled" checked> Bulk mode</label>
      <span>Cards ≤ $</span><input type="text" inputmode="decimal" id="bulkThreshold" class="tier-input" value="20" style="width:56px;">
      <span>Cash %</span><input type="text" inputmode="decimal" id="bulkCash" class="tier-input" placeholder="—" style="width:56px;">
      <span>Trade %</span><input type="text" inputmode="decimal" id="bulkTrade" class="tier-input" placeholder="—" style="width:56px;">
    </div>
  </section>

  <section>
    <h2 class="h2-neutral">Current Transaction</h2>
    <div class="txn-note"><input type="text" id="txnNote" placeholder="Customer / note (optional)"></div>

    <h2 class="h2-cash" style="margin-top:6px;">Cards In (from customer)</h2>
    <table>
      <thead><tr><th>Market $</th><th>Note</th><th>Card #</th><th>Bracket</th><th class="offer-cell">Cash</th><th class="offer-cell">Trade</th><th></th></tr></thead>
      <tbody id="inBody"></tbody>
    </table>
    <button class="btn btn-in" id="addIn" style="margin-top:8px;">+ Add card in</button>

    <h2 class="h2-trade" style="margin-top:18px;">Cards Out (to customer)</h2>
    <table>
      <thead><tr><th>Sticker $</th><th>Note</th><th>Card #</th><th>Payment</th><th class="num">Amount</th><th></th></tr></thead>
      <tbody id="outBody"></tbody>
    </table>
    <button class="btn btn-out" id="addOut" style="margin-top:8px;">+ Add card out</button>

    <div class="mini-ticket" id="miniTicket"></div>

    <div class="save-row">
      <button class="btn btn-save" id="saveTxn">Save transaction to day →</button>
    </div>
  </section>

  <section>
    <h2 class="h2-neutral">Day Ledger</h2>
    <div class="day-list" id="dayList"><div class="empty-state">No transactions saved yet.</div></div>

    <div class="ticket">
      <div class="ticket-title"><span>Day Totals</span><span id="txnCount">0 transactions</span></div>
      <div class="ticket-line"><span class="label">Cash paid out (buy-ins)</span><span class="mono" id="dCashOut">$0.00</span></div>
      <div class="ticket-line"><span class="label">Cash received (sales/top-ups)</span><span class="mono" id="dCashIn">$0.00</span></div>
      <div class="ticket-line"><span class="label">Trade credit issued</span><span class="mono" id="dCreditIssued">$0.00</span></div>
      <div class="ticket-line"><span class="label">Trade credit redeemed</span><span class="mono" id="dCreditRedeemed">$0.00</span></div>
      <div class="ticket-line"><span class="label">Cards in / out</span><span class="mono" id="dCardCount">0 / 0</span></div>
      <div class="ticket-line total"><span class="label">Net cash flow (day)</span><span class="mono" id="dNetFlow">$0.00</span></div>
    </div>

    <div class="day-actions">
      <button class="btn btn-neutral" id="completeDay">Complete Day → Export CSV</button>
      <button class="btn btn-remove" id="newDay" style="padding:8px 14px;">Start New Day</button>
    </div>
    <div class="note">Exporting doesn't erase anything — "Start New Day" clears the ledger once you've got the CSV. Bring that CSV to Claude in chat to get your Drive inventory file updated.</div>
  </section>
</div>

<script>
let tiers = [
  {min:0, max:10, cash:60, trade:75},
  {min:10.01, max:25, cash:70, trade:75},
  {min:25.01, max:149.99, cash:75, trade:80},
  {min:150, max:null, cash:65, trade:70},
];
let inRows = [], outRows = [], dayTxns = [];
let inId=0, outId=0, txnId=0;

function fmt(n){ const s = (n<0?'-':'') + '$' + Math.abs(n).toLocaleString('en-US',{minimumFractionDigits:2,maximumFractionDigits:2}); return s; }
function nowStr(){ const d=new Date(); return d.toLocaleTimeString('en-US',{hour:'2-digit',minute:'2-digit'}); }

// Re-rendering a row rebuilds its DOM nodes, which normally steals focus mid-keystroke.
// These helpers snapshot which field was focused (and cursor position) before a re-render,
// then restore it afterward so typing feels continuous instead of "one char at a time".
function saveFocus(){
  const active = document.activeElement;
  if(!active || active.tagName !== 'INPUT' || active.dataset.id === undefined) return null;
  let start=null, end=null;
  try { start = active.selectionStart; end = active.selectionEnd; } catch(e){}
  return { id: active.dataset.id, f: active.dataset.f, start, end };
}
function restoreFocus(saved, containerId){
  if(!saved) return;
  const container = document.getElementById(containerId);
  if(!container) return;
  const el = container.querySelector(`input[data-id="${saved.id}"][data-f="${saved.f}"]`);
  if(el){
    el.focus();
    if(saved.start!=null){ try{ el.setSelectionRange(saved.start, saved.end); }catch(e){} }
  }
}
function withFocusPreserved(renderFn, containerId){
  const saved = saveFocus();
  renderFn();
  restoreFocus(saved, containerId);
}

// ---------- Tier table ----------
function renderTiers(){
  const body = document.getElementById('tierBody');
  body.innerHTML = '';
  tiers.forEach((t,i)=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td><input type="text" inputmode="decimal" class="tier-input" value="${t.min}" data-i="${i}" data-f="min"></td>
      <td><input type="text" inputmode="decimal" class="tier-input" value="${t.max===null?'':t.max}" placeholder="∞" data-i="${i}" data-f="max"></td>
      <td><input type="text" inputmode="decimal" class="tier-input" value="${t.cash}" data-i="${i}" data-f="cash"></td>
      <td><input type="text" inputmode="decimal" class="tier-input" value="${t.trade}" data-i="${i}" data-f="trade"></td>
      <td><button class="btn btn-remove" data-remove="${i}">×</button></td>`;
    body.appendChild(tr);
  });
  body.querySelectorAll('input').forEach(inp=>{
    inp.addEventListener('input', e=>{
      const i=+e.target.dataset.i, f=e.target.dataset.f;
      let v=e.target.value;
      tiers[i][f] = (f==='max' && v==='') ? null : parseFloat(v);
      renderIn();
    });
  });
  body.querySelectorAll('[data-remove]').forEach(btn=>{
    btn.addEventListener('click', e=>{ tiers.splice(+e.target.dataset.remove,1); renderTiers(); renderIn(); });
  });
}
document.getElementById('addTier').addEventListener('click', ()=>{ tiers.push({min:0,max:null,cash:50,trade:60}); renderTiers(); });

function getBracket(price){
  const bulkEnabled = document.getElementById('bulkEnabled').checked;
  const threshold = parseFloat(document.getElementById('bulkThreshold').value) || 0;
  if(bulkEnabled && price <= threshold){
    const bc = parseFloat(document.getElementById('bulkCash').value);
    const bt = parseFloat(document.getElementById('bulkTrade').value);
    return { label:'Bulk', type:'bulk', cash:isNaN(bc)?0:bc, trade:isNaN(bt)?0:bt };
  }
  for(const t of tiers){
    if(price >= t.min && (t.max===null || price <= t.max)){
      return { label:`$${t.min}\u2013${t.max===null?'\u221e':t.max}`, type:'tier', cash:t.cash, trade:t.trade };
    }
  }
  return { label:'No match', type:'none', cash:0, trade:0 };
}

// ---------- IN rows ----------
function addIn(){ inRows.push({id:inId++, price:0, note:'', cardNum:'', pays:'trade'}); renderIn(); }
document.getElementById('addIn').addEventListener('click', addIn);

function renderIn(){
  const body = document.getElementById('inBody');
  body.innerHTML='';
  inRows.forEach(r=>{
    const b = getBracket(r.price);
    const cashPayout = r.price * (b.cash/100);
    const tradePayout = r.price * (b.trade/100);
    r._payout = r.pays==='cash' ? cashPayout : tradePayout;
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td><input type="text" inputmode="decimal" class="price-input" value="${r.price}" data-id="${r.id}" data-f="price"></td>
      <td><input type="text" class="note-input" value="${r.note}" placeholder="card name" data-id="${r.id}" data-f="note"></td>
      <td><input type="text" class="cardnum-input" value="${r.cardNum}" placeholder="150/108" data-id="${r.id}" data-f="cardNum"></td>
      <td><span class="badge ${b.type==='bulk'?'badge-bulk':'badge-tier'}">${b.label}</span></td>
      <td class="offer-cell"><div class="offer-box ${r.pays==='cash'?'selected-cash':''}" data-id="${r.id}" data-pays="cash">${fmt(cashPayout)}</div></td>
      <td class="offer-cell"><div class="offer-box ${r.pays==='trade'?'selected-trade':''}" data-id="${r.id}" data-pays="trade">${fmt(tradePayout)}</div></td>
      <td><button class="btn btn-remove" data-removein="${r.id}">×</button></td>`;
    body.appendChild(tr);
  });
  body.querySelectorAll('input[data-f]').forEach(inp=>{
    inp.addEventListener('input', e=>{
      const row = inRows.find(x=>x.id===+e.target.dataset.id);
      const f = e.target.dataset.f;
      row[f] = f==='price' ? (parseFloat(e.target.value)||0) : e.target.value;
      withFocusPreserved(renderIn, 'inBody');
    });
  });
  body.querySelectorAll('.offer-box').forEach(box=>{
    box.addEventListener('click', e=>{
      const id=+e.target.dataset.id;
      inRows.find(x=>x.id===id).pays = e.target.dataset.pays;
      renderIn();
    });
  });
  body.querySelectorAll('[data-removein]').forEach(btn=>{
    btn.addEventListener('click', e=>{ inRows = inRows.filter(x=>x.id !== +e.target.dataset.removein); renderIn(); });
  });
  updateMiniTicket();
}

// ---------- OUT rows ----------
function addOut(){ outRows.push({id:outId++, sticker:0, note:'', cardNum:'', paymentType:'credit', cashAmount:0}); renderOut(); }
document.getElementById('addOut').addEventListener('click', addOut);

function renderOut(){
  const body = document.getElementById('outBody');
  body.innerHTML='';
  outRows.forEach(r=>{
    const amount = r.paymentType==='credit' ? r.sticker : r.cashAmount;
    r._amount = amount;
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td><input type="text" inputmode="decimal" class="price-input" value="${r.sticker}" data-id="${r.id}" data-f="sticker"></td>
      <td><input type="text" class="note-input" value="${r.note}" placeholder="card name" data-id="${r.id}" data-f="note"></td>
      <td><input type="text" class="cardnum-input" value="${r.cardNum}" placeholder="150/108" data-id="${r.id}" data-f="cardNum"></td>
      <td><div class="toggle-group" data-id="${r.id}" data-side="out">
            <div class="toggle-opt ${r.paymentType==='cash'?'active-trade':''}" data-pay="cash">Cash Sale</div>
            <div class="toggle-opt ${r.paymentType==='credit'?'active-cash':''}" data-pay="credit">Trade Credit</div>
          </div></td>
      <td class="num">
        ${r.paymentType==='cash'
          ? `<input type="text" inputmode="decimal" class="price-input" value="${r.cashAmount}" data-id="${r.id}" data-f="cashAmount">`
          : `<span class="mono dim">${fmt(r.sticker)} <span style="font-size:9px;">(sticker)</span></span>`}
      </td>
      <td><button class="btn btn-remove" data-removeout="${r.id}">×</button></td>`;
    body.appendChild(tr);
  });
  body.querySelectorAll('input[data-f]').forEach(inp=>{
    inp.addEventListener('input', e=>{
      const row = outRows.find(x=>x.id===+e.target.dataset.id);
      const f = e.target.dataset.f;
      if(f==='note' || f==='cardNum'){ row[f] = e.target.value; }
      else{
        row[f] = parseFloat(e.target.value)||0;
        if(f==='sticker' && row.paymentType==='cash' && row.cashAmount===0){ row.cashAmount = row.sticker; }
      }
      withFocusPreserved(renderOut, 'outBody');
    });
  });
  body.querySelectorAll('[data-side="out"] .toggle-opt').forEach(opt=>{
    opt.addEventListener('click', e=>{
      const id=+e.target.closest('.toggle-group').dataset.id;
      const row = outRows.find(x=>x.id===id);
      row.paymentType = e.target.dataset.pay;
      if(row.paymentType==='cash' && row.cashAmount===0){ row.cashAmount = row.sticker; }
      renderOut();
    });
  });
  body.querySelectorAll('[data-removeout]').forEach(btn=>{
    btn.addEventListener('click', e=>{ outRows = outRows.filter(x=>x.id !== +e.target.dataset.removeout); renderOut(); });
  });
  updateMiniTicket();
}

// ---------- Transaction totals ----------
function computeTxnTotals(){
  const sumInCash = inRows.filter(r=>r.pays==='cash').reduce((a,r)=>a+r._payout,0);
  const sumInTrade = inRows.filter(r=>r.pays==='trade').reduce((a,r)=>a+r._payout,0);
  const sumOutCash = outRows.filter(r=>r.paymentType==='cash').reduce((a,r)=>a+r._amount,0);
  const sumOutCredit = outRows.filter(r=>r.paymentType==='credit').reduce((a,r)=>a+r._amount,0);

  const netTradeBalance = sumInTrade - sumOutCredit; // >0 leftover credit owed back, <0 shortfall owed to us
  const cashToCustomer = sumInCash + (netTradeBalance>0 ? netTradeBalance : 0);
  const cashToUs = sumOutCash + (netTradeBalance<0 ? -netTradeBalance : 0);
  const finalNet = cashToUs - cashToCustomer; // >0 customer owes us, <0 we owe customer

  return { sumInCash, sumInTrade, sumOutCash, sumOutCredit, netTradeBalance, cashToCustomer, cashToUs, finalNet };
}

function updateMiniTicket(){
  const t = computeTxnTotals();
  const dir = t.finalNet > 0.004 ? 'owe-us' : (t.finalNet < -0.004 ? 'owe-them' : '');
  const label = t.finalNet > 0.004 ? 'Customer owes you' : (t.finalNet < -0.004 ? 'You owe customer' : 'Even — no cash changes hands');
  document.getElementById('miniTicket').innerHTML = `
    <div class="mini-line"><span class="dim">Cash to customer (buy-ins)</span><span class="mono">${fmt(t.sumInCash)}</span></div>
    <div class="mini-line"><span class="dim">Trade credit generated</span><span class="mono">${fmt(t.sumInTrade)}</span></div>
    <div class="mini-line"><span class="dim">Cash sales (to us)</span><span class="mono">${fmt(t.sumOutCash)}</span></div>
    <div class="mini-line"><span class="dim">Trade credit redeemed</span><span class="mono">${fmt(t.sumOutCredit)}</span></div>
    <div class="mini-line final ${dir}"><span>${label}</span><span class="value mono">${fmt(Math.abs(t.finalNet))}</span></div>`;
}

// ---------- Save transaction to day ----------
document.getElementById('saveTxn').addEventListener('click', ()=>{
  if(inRows.length===0 && outRows.length===0) return;
  const totals = computeTxnTotals();
  dayTxns.push({
    id: txnId++,
    time: nowStr(),
    note: document.getElementById('txnNote').value,
    inRows: JSON.parse(JSON.stringify(inRows)),
    outRows: JSON.parse(JSON.stringify(outRows)),
    totals
  });
  inRows = []; outRows = [];
  document.getElementById('txnNote').value = '';
  renderIn(); renderOut(); renderDay();
});

// ---------- Day ledger ----------
function renderDay(){
  const list = document.getElementById('dayList');
  if(dayTxns.length===0){
    list.innerHTML = '<div class="empty-state">No transactions saved yet.</div>';
  } else {
    list.innerHTML = dayTxns.map(tx=>{
      const dir = tx.totals.finalNet > 0.004 ? 'owe-us' : (tx.totals.finalNet < -0.004 ? 'owe-them' : '');
      const label = tx.totals.finalNet > 0.004 ? '+' : (tx.totals.finalNet < -0.004 ? '\u2212' : '');
      return `<div class="day-row">
        <div class="day-row-left">
          <span class="day-row-time">${tx.time}</span>
          <span class="day-row-note">${tx.note || `${tx.inRows.length} in / ${tx.outRows.length} out`}</span>
        </div>
        <div class="day-row-left">
          <span class="day-row-amt ${dir}">${label}${fmt(Math.abs(tx.totals.finalNet))}</span>
          <button class="btn btn-remove" data-removetxn="${tx.id}">×</button>
        </div>
      </div>`;
    }).join('');
    list.querySelectorAll('[data-removetxn]').forEach(btn=>{
      btn.addEventListener('click', e=>{ dayTxns = dayTxns.filter(x=>x.id !== +e.target.dataset.removetxn); renderDay(); });
    });
  }

  let dCashOut=0, dCashIn=0, dCreditIssued=0, dCreditRedeemed=0, cardsIn=0, cardsOut=0;
  dayTxns.forEach(tx=>{
    dCashOut += tx.totals.cashToCustomer;
    dCashIn += tx.totals.cashToUs;
    dCreditIssued += tx.totals.sumInTrade;
    dCreditRedeemed += tx.totals.sumOutCredit;
    cardsIn += tx.inRows.length;
    cardsOut += tx.outRows.length;
  });
  document.getElementById('txnCount').textContent = dayTxns.length + (dayTxns.length===1?' transaction':' transactions');
  document.getElementById('dCashOut').textContent = fmt(dCashOut);
  document.getElementById('dCashIn').textContent = fmt(dCashIn);
  document.getElementById('dCreditIssued').textContent = fmt(dCreditIssued);
  document.getElementById('dCreditRedeemed').textContent = fmt(dCreditRedeemed);
  document.getElementById('dCardCount').textContent = `${cardsIn} / ${cardsOut}`;
  document.getElementById('dNetFlow').textContent = fmt(dCashIn - dCashOut);
}

// ---------- Complete day / export ----------
function csvEscape(s){ return `"${String(s).replace(/"/g,'""')}"`; }
document.getElementById('completeDay').addEventListener('click', ()=>{
  if(dayTxns.length===0){ alert('No transactions to export yet.'); return; }
  const rows = [['Date','Time','Transaction Note','Direction','Card Note','Card #','Price','Pay Type','Amount']];
  const dateStr = new Date().toLocaleDateString('en-US');
  dayTxns.forEach(tx=>{
    tx.inRows.forEach(r=>{
      rows.push([dateStr, tx.time, tx.note, 'IN', r.note, r.cardNum, r.price.toFixed(2), r.pays, r._payout.toFixed(2)]);
    });
    tx.outRows.forEach(r=>{
      rows.push([dateStr, tx.time, tx.note, 'OUT', r.note, r.cardNum, r.sticker.toFixed(2), r.paymentType, r._amount.toFixed(2)]);
    });
  });
  rows.push([]);
  rows.push(['DAY SUMMARY']);
  rows.push(['Cash paid out (buy-ins)', document.getElementById('dCashOut').textContent]);
  rows.push(['Cash received (sales/top-ups)', document.getElementById('dCashIn').textContent]);
  rows.push(['Trade credit issued', document.getElementById('dCreditIssued').textContent]);
  rows.push(['Trade credit redeemed', document.getElementById('dCreditRedeemed').textContent]);
  rows.push(['Cards in / out', document.getElementById('dCardCount').textContent]);
  rows.push(['Net cash flow', document.getElementById('dNetFlow').textContent]);

  const csv = rows.map(r=>r.map(csvEscape).join(',')).join('\n');
  const blob = new Blob([csv], {type:'text/csv'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `lot-ticket-${dateStr.replace(/\//g,'-')}.csv`;
  document.body.appendChild(a); a.click(); document.body.removeChild(a);
  URL.revokeObjectURL(url);
});

document.getElementById('newDay').addEventListener('click', ()=>{
  if(dayTxns.length>0 && !confirm('Clear the day ledger? Make sure you exported first.')) return;
  dayTxns = []; renderDay();
});

document.getElementById('bulkEnabled').addEventListener('change', renderIn);
document.getElementById('bulkThreshold').addEventListener('input', renderIn);
document.getElementById('bulkCash').addEventListener('input', renderIn);
document.getElementById('bulkTrade').addEventListener('input', renderIn);
document.getElementById('txnNote').addEventListener('input', ()=>{});

renderTiers();
addIn(); addOut();
renderDay();
</script>
</body>
</html>
