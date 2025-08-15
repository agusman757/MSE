<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Manajemen Stok Barang (AI Voice)</title>
  <link rel="manifest" href="manifest.json" />
  <link rel="stylesheet" href="style.css" />
  <meta name="theme-color" content="#3f51b5"/>
  <link rel="apple-touch-icon" href="icon-192.png">
</head>
<body>
  <header>
    <h1>📦 Stok Barang Suara</h1>
    <p style="font-size:1.1em;">Catat barang masuk & keluar pakai suara. Mudah diakses di HP!</p>
  </header>

  <main>
    <form id="transaksiForm" autocomplete="off">
      <select id="jenisTransaksi" required>
        <option value="masuk">Masuk</option>
        <option value="keluar">Keluar</option>
      </select>
      <input type="text" id="namaBarang" placeholder="Nama barang" required autocomplete="off"/>
      <input type="number" id="jumlahBarang" placeholder="Jumlah" min="1" required/>
      <button type="button" id="voiceBtn" title="Isi dengan suara">🎤</button>
      <button type="submit">Tambah</button>
    </form>

    <section id="filter">
      <label style="font-size:.9em">Tanggal: </label>
      <input type="date" id="filterStart" /> -
      <input type="date" id="filterEnd" />
      <button type="button" id="resetFilter">Reset</button>
      <button type="button" id="exportExcel">Ekspor Excel</button>
    </section>

    <section>
      <h2>Riwayat</h2>
      <table>
        <thead>
          <tr>
            <th>Tanggal</th>
            <th>Jenis</th>
            <th>Barang</th>
            <th>Jumlah</th>
            <th></th>
          </tr>
        </thead>
        <tbody id="transaksiList"></tbody>
      </table>
    </section>

    <section>
      <h2>Rekap Stok</h2>
      <table>
        <thead>
          <tr>
            <th>Barang</th>
            <th>Stok</th>
          </tr>
        </thead>
        <tbody id="rekapList"></tbody>
      </table>
    </section>
  </main>

  <footer>
    <small>© 2025 agusman757 — PWA Stok Barang Suara</small>
  </footer>
  <script src="https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js"></script>
  <script src="app.js"></script>
  <script>
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.register('service-worker.js');
    }
  </script>
</body>
</html>body {
  font-family: 'Segoe UI', Arial, sans-serif;
  margin: 0;
  background: #f8f9fa;
  color: #222;
}
header {
  background: #3f51b5;
  color: #fff;
  padding: 1.3rem 1rem 1rem 1rem;
  text-align: center;
  box-shadow: 0 2px 8px #0001;
}
main {
  max-width: 420px;
  margin: 1.5rem auto;
  background: #fff;
  border-radius: 10px;
  box-shadow: 0 2px 12px #0002;
  padding: 1.2rem 0.5rem;
}
form {
  display: flex;
  gap: 0.4rem;
  flex-wrap: wrap;
  align-items: center;
  margin-bottom: 1rem;
}
select, input, button {
  padding: 0.45rem 0.6rem;
  font-size: 1em;
  border-radius: 7px;
  border: 1px solid #ddd;
}
select, input {
  flex: 1 1 40%;
}
button[type="submit"] {
  background: #3f51b5;
  color: #fff;
  border: none;
  min-width: 80px;
}
#voiceBtn {
  background: #ff9800;
  color: #fff;
  border: none;
  cursor: pointer;
  font-size: 1.19em;
  min-width: 2.5em;
}
#voiceBtn:active {
  background: #ffb74d;
}
#exportExcel {
  background: #388e3c;
  color: #fff;
  border: none;
  cursor: pointer;
  font-weight: bold;
}
#exportExcel:active {
  background: #66bb6a;
}
section {
  margin-bottom: 1.3rem;
}
h2 {
  margin-top: 0;
  color: #3f51b5;
  font-size: 1.13em;
}
table {
  width: 100%;
  border-collapse: collapse;
  background: #fff;
  font-size: 0.98em;
}
th, td {
  border: 1px solid #eee;
  padding: 0.55em 0.25em;
  text-align: center;
}
th {
  background: #f1f1f9;
}
tr:nth-child(even) td {
  background: #fafbfc;
}
footer {
  text-align: center;
  color: #888;
  margin: 1.1rem 0 0.5rem 0;
  font-size: 0.95rem;
}
#filter {
  margin-bottom: 1.1rem;
  display: flex;
  gap: 0.3em;
  flex-wrap: wrap;
  align-items: center;
  font-size: 1em;
}
@media (max-width: 500px) {
  main { padding: .6rem 0.1rem;}
  form, #filter { flex-direction: column; gap: 0.7em;}
  th, td { font-size: 1em; }
}body {
  font-family: 'Segoe UI', Arial, sans-serif;
  margin: 0;
  background: #f8f9fa;
  color: #222;
}
header {
  background: #3f51b5;
  color: #fff;
  padding: 1.3rem 1rem 1rem 1rem;
  text-align: center;
  box-shadow: 0 2px 8px #0001;
}
main {
  max-width: 420px;
  margin: 1.5rem auto;
  background: #fff;
  border-radius: 10px;
  box-shadow: 0 2px 12px #0002;
  padding: 1.2rem 0.5rem;
}
form {
  display: flex;
  gap: 0.4rem;
  flex-wrap: wrap;
  align-items: center;
  margin-bottom: 1rem;
}
select, input, button {
  padding: 0.45rem 0.6rem;
  font-size: 1em;
  border-radius: 7px;
  border: 1px solid #ddd;
}
select, input {
  flex: 1 1 40%;
}
button[type="submit"] {
  background: #3f51b5;
  color: #fff;
  border: none;
  min-width: 80px;
}
#voiceBtn {
  background: #ff9800;
  color: #fff;
  border: none;
  cursor: pointer;
  font-size: 1.19em;
  min-width: 2.5em;
}
#voiceBtn:active {
  background: #ffb74d;
}
#exportExcel {
  background: #388e3c;
  color: #fff;
  border: none;
  cursor: pointer;
  font-weight: bold;
}
#exportExcel:active {
  background: #66bb6a;
}
section {
  margin-bottom: 1.3rem;
}
h2 {
  margin-top: 0;
  color: #3f51b5;
  font-size: 1.13em;
}
table {
  width: 100%;
  border-collapse: collapse;
  background: #fff;
  font-size: 0.98em;
}
th, td {
  border: 1px solid #eee;
  padding: 0.55em 0.25em;
  text-align: center;
}
th {
  background: #f1f1f9;
}
tr:nth-child(even) td {
  background: #fafbfc;
}
footer {
  text-align: center;
  color: #888;
  margin: 1.1rem 0 0.5rem 0;
  font-size: 0.95rem;
}
#filter {
  margin-bottom: 1.1rem;
  display: flex;
  gap: 0.3em;
  flex-wrap: wrap;
  align-items: center;
  font-size: 1em;
}
@media (max-width: 500px) {
  main { padding: .6rem 0.1rem;}
  form, #filter { flex-direction: column; gap: 0.7em;}
  th, td { font-size: 1em; }
}{
  "name": "Manajemen Stok Barang Suara",
  "short_name": "StokBarang",
  "start_url": ".",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#3f51b5",
  "icons": [
    {
      "src": "icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "icon-512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}const CACHE_NAME = "stok-barang-cache-v2";
const urlsToCache = [
  '/',
  '/index.html',
  '/style.css',
  '/app.js',
  '/manifest.json',
  '/icon-192.png',
  '/icon-512.png'
];

self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(CACHE_NAME).then(cache => cache.addAll(urlsToCache))
  );
});

self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request).then(response => response || fetch(event.request))
  );
});
