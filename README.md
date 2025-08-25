// ====== Storage helpers ======
const STORAGE_KEY = 'zamfara_grants_apps_v1';
const $ = (sel, root=document) => root.querySelector(sel);
const $$ = (sel, root=document) => Array.from(root.querySelectorAll(sel));

function loadApps(){
  try{ return JSON.parse(localStorage.getItem(STORAGE_KEY)) || []; }catch{ return []; }
}
function saveApps(list){ localStorage.setItem(STORAGE_KEY, JSON.stringify(list)); }
function genId(){ return 'A' + Date.now().toString(36) + Math.random().toString(36).slice(2,6).toUpperCase(); }

// ====== CSV / Download helpers ======
function toCSV(rows){
  if(!rows.length) return '';
  const headers = Object.keys(rows[0]);
  const esc = v => {
    if(v==null) return '';
    const s = String(v).replace(/"/g,'""');
    return /[",\n]/.test(s) ? `"${s}"` : s;
  };
  const lines = [headers.map(esc).join(',')];
  for(const r of rows){ lines.push(headers.map(h=>esc(r[h])).join(',')); }
  return lines.join('\n');
}
function download(filename, content, mime='text/plain;charset=utf-8'){
  const blob = new Blob([content], {type:mime});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url; a.download = filename; document.body.appendChild(a); a.click(); a.remove();
  URL.revokeObjectURL(url);
}

function yearStamp(){ const y = new Date().getFullYear(); $(`#year`).textContent = y; }

// ====== Build receipt HTML (for applicant self-download) ======
function buildReceiptHTML(app){
  return `<!doctype html>
<html><head><meta charset="utf-8"><meta name="viewport" content="width=device-width, initial-scale=1">
<title>Application Receipt – ${app.name}</title>
<style>
  body{font-family:system-ui,-apple-system,Segoe UI,Roboto,Inter,Arial,sans-serif;background:#f8fafc;color:#0f172a;padding:16px}
  .wrap{max-width:720px;margin:0 auto;background:#fff;border:1px solid #e2e8f0;border-radius:16px;padding:16px}
  .header{display:flex;align-items:center;justify-content:space-between;gap:12px;margin-bottom:12px}
  .logo{width:56px;height:56px;object-fit:contain;border-radius:14px;background:#ecfdf5;border:1px solid #e2e8f0}
  .gov{width:72px;height:72px;border-radius:50%;object-fit:cover;box-shadow:0 4px 12px rgba(0,0,0,.08)}
  h1{margin:0;font-size:20px}
  .slogan{color:#059669;font-weight:600;margin:0}
  .grid{display:grid;grid-template-columns:160px 1fr;gap:8px}
  .row{display:contents}
  .label{color:#64748b}
  .photo{margin-top:8px}
  img.pass{width:120px;height:120px;object-fit:cover;border:1px solid #e2e8f0;border-radius:12px}
  .muted{color:#64748b}
</style></head>
<body>
<div class="wrap">
  <div class="header">
    <img class="logo" src="zamfara-logo.png" alt="Logo">
    <div><h1>Zamfara State Government</h1><p class="slogan">Financial Grant – Application Receipt</p></div>
    <img class="gov" src="zamfara-governor.jpg" alt="Governor" />
  </div>
  <p class="muted">Application ID: ${app.id} · Submitted: ${app.submittedAt}</p>
  <div class="grid">
    <div class="row"><div class="label">Name</div><div>${app.name||''}</div></div>
    <div class="row"><div class="label">Date of birth</div><div>${app.dob||''}</div></div>
    <div class="row"><div class="label">Phone</div><div>${app.phone||''}</div></div>
    <div class="row"><div class="label">Ward</div><div>${app.ward||''}</div></div>
    <div class="row"><div class="label">State</div><div>${app.state||'Zamfara'}</div></div>
    <div class="row"><div class="label">Local government</div><div>${app.lga||''}</div></div>
  </div>
  <div class="photo"><div class="label">Passport photo</div>
    ${app.passport ? `<img class="pass" src="${app.passport}" alt="Passport" />` : '<div class="muted">N/A</div>'}
  </div>
  <div style="margin-top:8px"><div class="label">Advice</div><div>${(app.advice||'').replace(/</g,'&lt;')}</div></div>
</div>
</body></html>`;
}

// ====== APPLY PAGE LOGIC ======
function initApply(){
  const form = $('#apply-form');
  if(!form) return;

  const previewWrap = $('#passport-preview');
  const previewImg = $('#passport-img');
  yearStamp();

  form.addEventListener('change', (e)=>{
    const t = e.target;
    if(t.name === 'passport' && t.files && t.files[0]){
      if(!t.files[0].type.startsWith('image/')){ alert('Please upload an image file.'); t.value=''; return; }
      const reader = new FileReader();
      reader.onload = ()=>{
        previewImg.src = reader.result;
        previewWrap.classList.remove('hidden');
        previewWrap.dataset.value = reader.result;
      };
      reader.readAsDataURL(t.files[0]);
    }
  });

  form.addEventListener('submit', (e)=>{
    e.preventDefault();

    const data = Object.fromEntries(new FormData(form).entries());
    if(!previewWrap.dataset.value){ alert('Passport is required.'); return; }
    if(!data.lga){ alert('Local Government is required.'); return; }

    const app = {
      id: genId(),
      submittedAt: new Date().toISOString(),
      name: (data.name||'').trim(),
      passport: previewWrap.dataset.value || null,
      dob: data.dob||'',
      phone: (data.phone||'').trim(),
      ward: (data.ward||'').trim(),
      state: 'Zamfara',
      lga: data.lga||'',
      advice: (data.advice||'').trim(),
    };

    const list = [app, ...loadApps()];
    saveApps(list);

    // Show confirmation / receipt
    $('#apply-section').classList.add('hidden');
    const receiptBox = $('#receipt');
    receiptBox.innerHTML = `
      <div class="row"><div><strong>Name</strong></div><div>${app.name}</div></div>
      <div class="row"><div><strong>Date of birth</strong></div><div>${app.dob}</div></div>
      <div class="row"><div><strong>Phone</strong></div><div>${app.phone}</div></div>
      <div class="row"><div><strong>Ward</strong></div><div>${app.ward}</div></div>
      <div class="row"><div><strong>State</strong></div><div>${app.state}</div></div>
      <div class="row"><div><strong>Local government</strong></div><div>${app.lga}</div></div>
      <div class="row"><div><strong>Advice</strong></div><div>${app.advice || '<span class="muted">(none)</span>'}</div></div>
      <div class="row"><div><strong>Application ID</strong></div><div>${app.id}</div></div>
      <div class="row"><div><strong>Submitted</strong></div><div>${app.submittedAt}</div></div>
      <div class="row"><div><strong>Passport</strong></div><div>${app.passport ? `<img class="pass" src="${app.passport}" alt="Passport" />` : 'N/A'}</div></div>
    `;
    $('#confirmation').classList.remove('hidden');

    // Self-downloads
    $('#download-json').onclick = ()=>{
      download(`my_application_${app.id}.json`, JSON.stringify(app, null, 2), 'application/json');
    };
    $('#download-csv').onclick = ()=>{
      const csv = toCSV([{Name:app.name, DateOfBirth:app.dob, Phone:app.phone, Ward:app.ward, State:app.state, LocalGovernment:app.lga, Advice:app.advice, ApplicationID:app.id, SubmittedAt:app.submittedAt}]);
      download(`my_application_${app.id}.csv`, csv, 'text/csv');
    };
    $('#download-receipt').onclick = ()=>{
      const html = buildReceiptHTML(app);
      download(`application_receipt_${app.id}.html`, html, 'text/html;charset=utf-8');
    };
  });
}

// ====== ADMIN PAGE LOGIC ======
function initAdmin(){
  const tableBody = document.querySelector('#apps-table tbody');
  if(!tableBody) return;
  yearStamp();

  const search = $('#search');
  const filterLGA = $('#filter-lga');

  function render(){
    const apps = loadApps();
    const q = (search.value||'').toLowerCase();
    const f = (filterLGA.value||'').toLowerCase();
    const rows = apps.filter(a=>{
      const okName = a.name?.toLowerCase().includes(q);
      const okLga = f ? (a.lga||'').toLowerCase() === f : true;
      return okName && okLga;
    });

    tableBody.innerHTML = rows.map(a=>`
      <tr>
        <td>${a.passport ? `<img src="${a.passport}" alt="Passport" style="width:48px;height:48px;object-fit:cover;border-radius:8px;border:1px solid #e2e8f0;"/>` : 'N/A'}</td>
        <td>${a.name||''}</td>
        <td>${a.phone||''}</td>
        <td>${a.lga||''}</td>
        <td>${(a.advice||'').replace(/</g,'&lt;')}</td>
        <td>${a.submittedAt||''}</td>
      </tr>
    `).join('');
  }

  render();
  search.addEventListener('input', render);
  filterLGA.addEventListener('input', render);

  $('#download-csv-all').onclick = ()=>{
    const apps = loadApps();
    const rows = apps.map(a=>({
      ID:a.id, SubmittedAt:a.submittedAt, Name:a.name, DateOfBirth:a.dob, Phone:a.phone, Ward:a.ward, State:a.state, LocalGovernment:a.lga, Advice:a.advice
    }));
    download(`zamfara_grant_apps_${new Date().toISOString().slice(0,10)}.csv`, toCSV(rows), 'text/csv');
  };
  $('#download-json-all').onclick = ()=>{
    download(`zamfara_grant_apps_${new Date().toISOString().slice(0,10)}.json`, JSON.stringify(loadApps(), null, 2), 'application/json');
  };
  $('#clear-all').onclick = ()=>{
    if(confirm('This will delete ALL locally stored applications on this device. Continue?')){
      saveApps([]); render();
    }
  };
}

// ====== Boot ======
document.addEventListener('DOMContentLoaded', ()=>{
  const page = document.body.dataset.page;
  if(page==='apply') initApply();
  if(page==='admin') initAdmin();
});
