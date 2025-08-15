cd C:\path\to\parent-of-stok-barang
Compress-Archive -Path .\stok-barang\* -DestinationPath .\stok-barang.zip
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Pencatatan Stok Barang</title>
  <link rel="manifest" href="manifest.json" />
  <link rel="stylesheet" href="style.css" />
</head>
<body>
  <header><h1>📦 Pencatatan Stok Barang</h1></header>

  <main>
    <form id="item-form">
      <input id="item-name" type="text" placeholder="Nama barang" required />
      <input id="item-qty" type="number" placeholder="Jumlah" required />
      <button id="voice-btn" type="button">🎤</button>
      <button type="submit">Tambah</button>
    </form>

    <h2>Daftar Barang</h2>
    <table id="table">
      <thead><tr><th>Nama</th><th>Jumlah</th><th>Aksi</th></tr></thead>
      <tbody id="list"></tbody>
    </table>
  </main>

  <footer><small>Offline ready • PWA Stok Barang</small></footer>

  <script src="app.js"></script>
</body>
</html>
* { box-sizing: border-box; }
body { font-family: Arial, sans-serif; margin:0; background:#f3f4f6; color:#111; }
header { background:#16a34a; color:white; padding:14px 12px; text-align:center; }
main { padding:12px; max-width:900px; margin:0 auto; }
form { display:flex; gap:8px; flex-wrap:wrap; margin-bottom:12px; }
input[type="text"], input[type="number"] { padding:10px; font-size:16px; flex:1 1 140px; }
button { padding:10px 12px; font-size:16px; cursor:pointer; border:none; background:#0ea5e9; color:white; border-radius:6px; }
#voice-btn { background:#f59e0b; }
table { width:100%; border-collapse:collapse; background:#fff; }
th, td { padding:10px; border:1px solid #e5e7eb; text-align:center; }
footer { text-align:center; padding:12px; color:#666; }
@media (max-width:520px){ form { flex-direction:column; } }
// PWA Stok Barang — IndexedDB optional fallback to localStorage
// For simplicity we use localStorage here (works offline & persists per browser)
const STORAGE_KEY = 'stok_barang_items';

let items = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]');

const form = document.getElementById('item-form');
const inputName = document.getElementById('item-name');
const inputQty = document.getElementById('item-qty');
const listEl = document.getElementById('list');
const voiceBtn = document.getElementById('voice-btn');

function save() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(items));
  render();
}

function render() {
  listEl.innerHTML = '';
  items.forEach((it, idx) => {
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${escapeHtml(it.name)}</td><td>${it.qty}</td>
      <td><button data-i="${idx}" class="del">Hapus</button></td>`;
    listEl.appendChild(tr);
  });
  // delegate delete
  document.querySelectorAll('.del').forEach(btn=>{
    btn.onclick = e => {
      const i = parseInt(e.target.dataset.i);
      items.splice(i,1);
      save();
    };
  });
}

function escapeHtml(s){ return String(s).replace(/[&<>"']/g, (m)=>({ '&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;' }[m])); }

form.addEventListener('submit', (e)=> {
  e.preventDefault();
  const name = inputName.value.trim();
  const qty = parseInt(inputQty.value);
  if(!name || isNaN(qty)){ alert('Isi nama dan jumlah yang valid'); return; }
  // if exists, add qty to existing
  const found = items.find(it=> it.name.toLowerCase()===name.toLowerCase());
  if(found){ found.qty += qty; } else { items.push({ name, qty }); }
  save();
  form.reset();
  inputName.focus();
});

// Voice input: supports "Tambah 5 paku" or just name/qty result handling
voiceBtn.addEventListener('click', ()=>{
  const Speech = window.SpeechRecognition || window.webkitSpeechRecognition;
  if(!Speech){ alert('Browser tidak mendukung input suara. Gunakan Chrome di Android.'); return; }
  const rec = new Speech();
  rec.lang = 'id-ID';
  rec.interimResults = false;
  rec.maxAlternatives = 1;
  rec.start();
  rec.onresult = (ev) => {
    const text = ev.results[0][0].transcript.trim();
    // try parse "tambah 5 paku" or "5 paku" or "paku 5"
    const t = text.toLowerCase();
    let m = t.match(/(?:tambah|masuk)\s+(\d+)\s+(.+)/) || t.match(/(\d+)\s+(.+)/) || t.match(/(.+)\s+(\d+)/);
    if(m){
      let qty, name;
      if(m.length===3){
        // patterns: [full, num, name] or [full, name, num]
        if(/^\d+$/.test(m[1])){ qty = parseInt(m[1]); name = m[2]; }
        else if(/^\d+$/.test(m[2])){ qty = parseInt(m[2]); name = m[1]; }
      }
      if(name && qty){
        inputName.value = name;
        inputQty.value = qty;
        // auto add
        form.dispatchEvent(new Event('submit'));
        return;
      }
    }
    // fallback: put speech into name field
    inputName.value = text;
  };
  rec.onerror = ()=>{ alert('Gagal mengenali suara. Coba lagi.'); };
});

render();

// register service worker if available
if('serviceWorker' in navigator){
  navigator.serviceWorker.register('service-worker.js').catch(()=>{/* ignore */});
}
{
  "name": "Pencatatan Stok Barang",
  "short_name": "StokBarang",
  "start_url": ".",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#16a34a",
  "icons":[
    { "src":"icon-192.png","sizes":"192x192","type":"image/png" },
    { "src":"icon-512.png","sizes":"512x512","type":"image/png" }
  ]
}
const CACHE = 'stok-barang-v1';
const FILES = [
  '/',
  '/index.html',
  '/style.css',
  '/app.js',
  '/manifest.json',
  '/icon-192.png',
  '/icon-512.png'
];
self.addEventListener('install', e=>{
  e.waitUntil(caches.open(CACHE).then(c=>c.addAll(FILES)));
});
self.addEventListener('fetch', e=>{
  e.respondWith(caches.match(e.request).then(r=> r || fetch(e.request)));
});
