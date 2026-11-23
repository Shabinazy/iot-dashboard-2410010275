# Dashboard Monitoring dan Kontrol IoT Berbasis Node-RED dan REST API

Proyek praktikum **Pemrograman Internet of Things — Pertemuan 9: REST API & Antarmuka Web IoT**. Aplikasi ini mensimulasikan pengiriman data sensor dari ESP32 menggunakan Python dan MQTT, mengolahnya di Node-RED sebagai backend REST API, serta menampilkannya pada dashboard web berbasis HTML, CSS, dan JavaScript.

## Identitas Mahasiswa

| Keterangan | Data |
| --- | --- |
| **Nama** | **Firda Noor Shabina** |
| **NPM** | **2410010275** |
| **Kelas** | **5M Reguler BJM** |
| **Mata Kuliah** | Pemrograman Internet of Things |

## Fitur yang Dikerjakan

- **Monitoring suhu dan kelembaban:** data simulasi MQTT diterima Node-RED dan ditampilkan pada dashboard.
- **Status sensor:** indikator `ONLINE` ketika API menyediakan data suhu dan `OFFLINE` ketika data tidak tersedia atau permintaan gagal.
- **Riwayat sensor:** Node-RED menyimpan hingga **50 data terakhir** dalam *flow context*; dashboard menampilkan **10 data terakhir**.
- **Kontrol perangkat:** tombol **Nyalakan Lampu** dan **Matikan Lampu** mengirim permintaan REST API, kemudian Node-RED menerbitkan perintah ke topik MQTT `iot/kontrol`.
- **Log aktivitas:** menampilkan aktivitas dashboard, hasil pengiriman perintah, serta pesan kesalahan apabila terjadi kegagalan.

> **Ruang lingkup:** Sensor ESP32 dan pembacaan sensor pada proyek ini **disimulasikan dengan Python**, bukan dari perangkat ESP32 fisik. Kontrol lampu berupa pengiriman perintah melalui MQTT; tampilan log `Perintah terkirim` tidak dengan sendirinya membuktikan lampu fisik menyala. Penyimpanan riwayat masih menggunakan *flow context*, belum SQLite/InfluxDB. Fitur pengembangan tambahan seperti threshold, API key, WebSocket, dan Chart.js tidak diklaim sebagai fitur yang sudah diimplementasikan pada versi pengumpulan ini.

## Teknologi yang Digunakan

| Teknologi | Peran |
| --- | --- |
| Python + `paho-mqtt` | Simulator pengirim data sensor |
| MQTT / Mosquitto | Penghubung simulator dan Node-RED |
| Node-RED | Backend REST API dan pengolahan pesan MQTT |
| HTML, CSS, JavaScript | Antarmuka web monitoring dan kontrol |
| Fetch API | Mengambil data sensor dan mengirim perintah kontrol |

## Struktur Berkas

```text
.
├── README.md
├── simulasi_esp32.py
├── flows_2410010275.json
├── dasboard.html
├── HASIL UJI_2410010275.pdf
└── flow_rest_api_iot.json
```

**Keterangan berkas:**

- `simulasi_esp32.py` — simulator yang menerbitkan suhu dan kelembaban acak tiap 3 detik.
- `flows_2410010275.json` — **flow utama untuk diimpor ke Node-RED**, termasuk konfigurasi broker MQTT.
- `dasboard.html` — website dashboard (nama berkas dipertahankan sesuai file pengumpulan).
- `HASIL UJI_2410010275.pdf` — dokumentasi hasil pengujian website, riwayat/log aktivitas, flow, dan simulator.
- `flow_rest_api_iot.json` — berkas flow referensi/alternatif. **Tidak perlu diimpor bersamaan** dengan `flows_2410010275.json` karena endpoint yang digunakan sama.

## Cara Menjalankan

### 1. Persiapan

Pastikan sudah tersedia **Python 3**, **Node-RED**, dan broker MQTT **Mosquitto** di komputer. Instal pustaka Python yang diperlukan:

```bash
python -m pip install paho-mqtt
```

### 2. Jalankan broker MQTT

Pastikan Mosquitto berjalan pada `localhost:1883`. Jika belum berjalan sebagai layanan Windows, jalankan broker sesuai instalasi di komputer, misalnya:

```bash
mosquitto -v
```

Jika Mosquitto sudah aktif sebagai layanan, tidak perlu menjalankan proses kedua pada port yang sama.

### 3. Impor flow ke Node-RED

1. Jalankan Node-RED:

   ```bash
   node-red
   ```

2. Buka `http://localhost:1880` pada browser.
3. Klik menu **☰ → Import**, lalu pilih **`flows_2410010275.json`**.
4. Pastikan konfigurasi MQTT broker mengarah ke `localhost` port `1883`.
5. Klik **Deploy**. Periksa agar node **Subscribe Sensor** berstatus `connected`.

**Catatan:** Jika sudah ada flow dengan endpoint `/api/suhu`, `/api/history`, atau `/api/kontrol`, hindari mengaktifkan flow duplikat secara bersamaan.

### 4. Jalankan simulator ESP32

Buka CMD atau terminal pada folder yang berisi `simulasi_esp32.py`, lalu jalankan:

```bash
python simulasi_esp32.py
```

Simulator mengirim data suhu acak **25–32 °C** dan kelembaban **55–80%** melalui topik `iot/sensor` setiap **3 detik**. Biarkan terminal tetap berjalan selama pengujian. Hentikan dengan `Ctrl+C`.

> Versi kode simulator yang dikumpulkan menggunakan Paho MQTT Callback API **VERSION1**. Pada versi pustaka tertentu, pesan peringatan *deprecated* dapat muncul meskipun program tetap berjalan.

### 5. Buka dashboard web

Buka berkas **`dasboard.html`** menggunakan Chrome atau Microsoft Edge (misalnya dengan klik dua kali file). Jika diletakkan pada folder `C:\node-red-static\`, berkas juga dapat dibuka dari lokasi tersebut.

Dashboard menggunakan alamat API **`http://localhost:1880/api`**, sehingga **Node-RED dan simulator harus berjalan di komputer yang sama** dengan browser, atau alamat API perlu disesuaikan untuk penggunaan jaringan lain.

Saat berjalan, data suhu diperbarui setiap **3 detik**, sedangkan tabel history setiap **5 detik**.

## Endpoint REST API

| Method | Endpoint | Fungsi |
| --- | --- | --- |
| `GET` | `/api/suhu` | Mengambil data sensor terbaru |
| `GET` | `/api/history` | Mengambil riwayat sensor dari *flow context* |
| `POST` | `/api/kontrol` | Mengirim perintah kontrol perangkat ke MQTT |

Contoh respons `GET /api/suhu`:

```json
{
  "status": "success",
  "data": {
    "suhu": 28.9,
    "kelembaban": 58.6,
    "timestamp": "2026-11-23T16:46:40.819767",
    "received": "2026-11-23T08:46:40.819Z"
  },
  "server_time": "2026-11-23T08:46:43.000Z"
}
```

Contoh permintaan untuk mengontrol lampu (`POST /api/kontrol`):

```json
{
  "device": "lamp",
  "action": "on"
}
```

Untuk mematikan, gunakan `"action": "off"`. Node-RED meneruskan perintah ke topik MQTT **`iot/kontrol`**.

## Pengujian dan Hasil

Pengujian dasar didokumentasikan pada **`HASIL UJI_2410010275.pdf`**, meliputi:

1. Dashboard yang menampilkan suhu, kelembaban, status sensor, dan riwayat pembacaan.
2. Log aktivitas perintah lampu **ON/OFF** dari website.
3. Flow Node-RED untuk penerimaan MQTT dan penyediaan REST API.
4. Keluaran terminal simulator Python yang menerbitkan data sensor.

Untuk mengecek API secara langsung melalui browser, buka:

- `http://localhost:1880/api/suhu`
- `http://localhost:1880/api/history`

**Catatan teknis:** Riwayat 50 pembacaan tersimpan dalam *flow context* Node-RED. Pada konfigurasi penyimpanan context default di memori, data tersebut dapat hilang setelah Node-RED dimulai ulang. Proyek ini ditujukan untuk pengujian lokal, bukan deployment layanan publik.
**catatan** : Mohon maaf, Pak, untuk fitur tambahan saya sudah mencoba mengimplementasikannya, tetapi masih mengalami error sehingga belum berhasil. Oleh karena itu, saya mengumpulkan fitur utama yang sudah berjalan dengan baik. Terima kasih atas pengertian Bapak.
## Penyusun

**Firda Noor Shabina**  
**NPM:** 2410010275  
**Kelas:** 5M Reguler BJM
