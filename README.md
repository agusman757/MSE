<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PWA Stok Barang</title>
    <link rel="manifest" href="manifest.json">
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h1>📦 Pencatatan Stok Barang</h1>

    <div class="input-area">
        <input type="text" id="itemName" placeholder="Nama barang">
        <input type="number" id="itemQty" placeholder="Jumlah">
        <button id="addBtn">Tambah</button>
        <button id="voiceBtn">🎤 Input Suara</button>
    </div>

    <h2>Daftar Stok</h2>
    <table>
        <thead>
            <tr>
                <th>Barang</th>
                <th>Jumlah</th>
            </tr>
        </thead>
        <tbody id="stockList"></tbody>
    </table>

    <script src="app.js"></script>
</body>
</html>
body {
    font-family: Arial, sans-serif;
    padding: 20px;
}
h1 {
    text-align: center;
}
.input-area {
    display: flex;
    gap: 10px;
    margin-bottom: 20px;
}
input, button {
    padding: 10px;
    font-size: 16px;
}
table {
    width: 100%;
    border-collapse: collapse;
}
th, td {
    border: 1px solid #ccc;
    padding: 8px;
    text-align: center;
}
let db;
const request = indexedDB.open("stokBarangDB", 1);

request.onupgradeneeded = function(e) {
    db = e.target.result;
    if (!db.objectStoreNames.contains("stok")) {
        db.createObjectStore("stok", { keyPath: "nama" });
    }
};

request.onsuccess = function(e) {
    db = e.target.result;
    loadStock();
};

document.getElementById("addBtn").addEventListener("click", addStock);
document.getElementById("voiceBtn").addEventListener("click", startVoice);

function addStock() {
    const nama = document.getElementById("itemName").value.trim();
    const qty = parseInt(document.getElementById("itemQty").value);
    if (!nama || isNaN(qty)) return;

    const tx = db.transaction("stok", "readwrite");
    const store = tx.objectStore("stok");
    store.get(nama).onsuccess = function(e) {
        let data = e.target.result || { nama, jumlah: 0 };
        data.jumlah += qty;
        store.put(data);
        loadStock();
    };
}

function loadStock() {
    const tx = db.transaction("stok", "readonly");
    const store = tx.objectStore("stok");
    const list = document.getElementById("stockList");
    list.innerHTML = "";

    store.openCursor().onsuccess = function(e) {
        const cursor = e.target.result;
        if (cursor) {
            const row = `<tr><td>${cursor.value.nama}</td><td>${cursor.value.jumlah}</td></tr>`;
            list.innerHTML += row;
            cursor.continue();
        }
    };
}

function startVoice() {
    if (!('webkitSpeechRecognition' in window)) {
        alert("Browser tidak mendukung input suara");
        return;
    }
    const recognition = new webkitSpeechRecognition();
    recognition.lang = "id-ID";
    recognition.onresult = function(event) {
        const hasil = event.results[0][0].transcript;
        const match = hasil.match(/tambah (\d+) (.+)/i);
        if (match) {
            document.getElementById("itemName").value = match[2];
            document.getElementById("itemQty").value = match[1];
            addStock();
        } else {
            alert("Format suara: 'Tambah 5 paku'");
        }
    };
    recognition.start();
}

// Registrasi service worker
if ("serviceWorker" in navigator) {
    navigator.serviceWorker.register("service-worker.js");
}
self.addEventListener("install", event => {
    event.waitUntil(
        caches.open("stok-cache").then(cache => {
            return cache.addAll([
                "/",
                "/index.html",
                "/style.css",
                "/app.js"
            ]);
        })
    );
});

self.addEventListener("fetch", event => {
    event.respondWith(
        caches.match(event.request).then(response => {
            return response || fetch(event.request);
        })
    );
});
{
    "name": "PWA Stok Barang",
    "short_name": "StokBarang",
    "start_url": ".",
    "display": "standalone",
    "background_color": "#ffffff",
    "description": "Aplikasi pencatatan stok barang offline dengan input suara",
    "icons": [
        {
            "src": "icon.png",
            "sizes": "192x192",
            "type": "image/png"
        }
    ]
}
