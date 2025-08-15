<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png">
<link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png">
<link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png">
<link rel="manifest" href="/site.webmanifest">
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Pencatatan Stok Barang</title>
  <link rel="manifest" href="manifest.json">
  <link rel="stylesheet" href="style.css">
  <meta name="theme-color" content="#4CAF50"/>
</head>
<body>
  <header>
    <h1>📦 Pencatatan Stok Barang</h1>
  </header>

  <main>
    <form id="barangForm">
      <input type="text" id="namaBarang" placeholder="Nama barang" required>
      <input type="number" id="jumlahBarang" placeholder="Jumlah" required>
      <button type="button" id="voiceBtn">🎤</button>
      <button type="submit">Tambah</button>
    </form>

    <h2>Daftar Barang</h2>
    <table>
      <thead>
        <tr>
          <th>Nama</th>
          <th>Jumlah</th>
          <th>Aksi</th>
        </tr>
      </thead>
      <tbody id="barangList"></tbody>
    </table>
  </main>

  <script src="app.js"></script>
</body>
</html>
body {
  font-family: sans-serif;
  margin: 0;
  background: #f4f4f4;
}

header {
  background: #4CAF50;
  padding: 1rem;
  color: white;
  text-align: center;
}

form {
  display: flex;
  gap: 5px;
  padding: 10px;
}

input, button {
  padding: 8px;
  font-size: 1rem;
}

table {
  width: 100%;
  border-collapse: collapse;
  background: white;
  margin-top: 10px;
}

th, td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: center;
}

button {
  cursor: pointer;
}

#voiceBtn {
  background: #ff9800;
  color: white;
  border: none;
}
// Load data dari localStorage
let barang = JSON.parse(localStorage.getItem("barangList")) || [];

const form = document.getElementById("barangForm");
const namaBarang = document.getElementById("namaBarang");
const jumlahBarang = document.getElementById("jumlahBarang");
const barangList = document.getElementById("barangList");
const voiceBtn = document.getElementById("voiceBtn");

// Render data
function renderBarang() {
  barangList.innerHTML = "";
  barang.forEach((item, index) => {
    let row = `<tr>
      <td>${item.nama}</td>
      <td>${item.jumlah}</td>
      <td><button onclick="hapusBarang(${index})">Hapus</button></td>
    </tr>`;
    barangList.innerHTML += row;
  });
}

// Tambah data
form.addEventListener("submit", (e) => {
  e.preventDefault();
  barang.push({ nama: namaBarang.value, jumlah: parseInt(jumlahBarang.value) });
  localStorage.setItem("barangList", JSON.stringify(barang));
  namaBarang.value = "";
  jumlahBarang.value = "";
  renderBarang();
});

// Hapus data
function hapusBarang(index) {
  barang.splice(index, 1);
  localStorage.setItem("barangList", JSON.stringify(barang));
  renderBarang();
}

// Input suara
voiceBtn.addEventListener("click", () => {
  if ('webkitSpeechRecognition' in window) {
    let recognition = new webkitSpeechRecognition();
    recognition.lang = "id-ID";
    recognition.start();
    recognition.onresult = function(event) {
      namaBarang.value = event.results[0][0].transcript;
    };
  } else {
    alert("Browser tidak mendukung input suara.");
  }
});

// Panggil render pertama
renderBarang();

// PWA Service Worker
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('service-worker.js');
}
{
  "name": "Pencatatan Stok Barang",
  "short_name": "StokBarang",
  "start_url": ".",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#4CAF50",
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
}
const CACHE_NAME = "stok-barang-cache-v1";
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
