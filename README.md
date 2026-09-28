# PWA Tempahan Makanan HKL

Frontend statik (GitHub Pages) + backend Google Apps Script + Google Sheets. Kos RM0.

## Fail

| Fail | Fungsi |
|---|---|
| `Code.gs` | Backend API (Apps Script) + `setup()` cipta tab |
| `index.html` | App penuh 3 peranan. `API_URL` kosong = mod demo (localStorage) |
| `manifest.json`, `sw.js`, `icon*` | Keperluan PWA (boleh "Add to Home Screen") |

## Langkah pasang

### Langkah asas (wajib, kedua-dua pilihan)
1. **Google Sheet** — buat Sheet baharu, contoh `DB_Tempahan_Makanan_HKL`.
2. **Apps Script** — Extensions > Apps Script > padam kod asal > tampal `Code.gs` > Save.
3. **Setup** — pilih fungsi `setup` > Run > benarkan akses (Sheets + hantar emel). 5 tab dicipta dengan 4 akaun contoh, kata laluan `Demo1234`: A001 Pentadbir, P001, D001, S001. Tukar kata laluan akaun ini segera melalui menu **Akaun**.
   > Jika pernah jalankan `setup()` versi lama, padam tab `Pengguna` dahulu (lajurnya sudah berubah).

### Pilihan A — Semua dalam Apps Script (paling mudah, tiada GitHub)
4. Dalam editor Apps Script: **+ > HTML** → namakan `Index` (tanpa .html) → padam isi asal → tampal **keseluruhan** `index.html` → Save.
5. `CONFIG.API_URL` biarkan kosong (`''`). App mengesan sendiri ia berjalan dalam Apps Script.
6. **Deploy > New deployment > Web app** — Execute as: **Me**, Who has access: **Anyone** → buka URL `/exec`. Siap.

Kekangan: URL panjang (`script.google.com/...`), ada jalur "This application was created by a Google Apps Script user" di atas, dan tidak boleh "install" sebagai app sebenar (boleh tambah pintasan ke skrin utama sahaja).

### Pilihan B — GitHub Pages (PWA penuh, disyorkan untuk guna sebenar)
4. **Deploy > New deployment > Web app** — Execute as: **Me**, Who has access: **Anyone** → salin URL `/exec`.
5. Dalam `index.html`, isi `CONFIG.API_URL: 'https://script.google.com/macros/s/.../exec'`.
6. Repo GitHub baharu → upload `index.html`, `manifest.json`, `sw.js`, `icon.svg`, `icon-192.png`, `icon-512.png`, `icon-512-maskable.png` → Settings > Pages > Branch `main` / root → Save.
7. Buka `https://<username>.github.io/<repo>/` di telefon → **Add to Home Screen**. App dibuka penuh skrin tanpa bar pelayar.

`README.md` dan `Code.gs` tidak perlu dinaikkan ke GitHub (Code.gs tinggal dalam Apps Script sahaja).

> Setiap kali ubah `Code.gs` atau fail `Index`: Deploy > Manage deployments > Edit (ikon pensel) > Version: **New version** > Deploy. URL kekal sama.

## Struktur data

Ikut dokumen asal, dengan 3 tambahan:
- **Tab `Pengguna`** (baharu) — `ID_Staf | Nama | Peranan | Kata_Laluan | Lokasi_Wad | Jawatan | No_Tel | Emel | Emel_Disahkan | Status_Akaun | Tarikh_Daftar | Tarikh_Tamat | Diluluskan_Oleh`
  - Peranan: `STAF`, `PENGUSAHA`, `DIETETIK`, `ADMIN`
  - `Status_Akaun`: Menunggu → Aktif / Ditolak / Digantung. Status **Tamat** dikira automatik bila `Tarikh_Tamat` lepas.
- **`Pesanan_Masuk`** — tambah lajur `Sesi_Makan` dan `Nama_Staf` di hujung (untuk dapur).
- Status yang digunakan:
  - `Status_Pengesahan`: Draf → Disahkan / Ditolak
  - `Status_Tuntut`: Belum → Dituntut
  - `Status_Siap`: Baru → Disediakan → Siap → Dihantar (atau Dibatalkan)

## Pendaftaran & kata laluan

**Daftar (auto lulus)**
1. Tekan **Daftar akaun baharu** → isi ID staf, nama, jawatan, wad, telefon, **emel (wajib)**, kata laluan, tarikh tamat penempatan (pilihan).
2. Kod 6 digit dihantar ke emel → masukkan kod → akaun **terus Aktif**. Tiada kelulusan manual.
3. Jika tutup app sebelum sahkan: log masuk seperti biasa → sistem bawa ke skrin sahkan emel → **Hantar kod**.

**Lupa kata laluan**
1. **Lupa kata laluan?** → isi ID staf + emel berdaftar → **Hantar kod**.
2. Masukkan kod + kata laluan baharu → simpan. Semua sesi lama di peranti lain dilog keluar.
3. Pengguna tanpa emel (cth. import pukal): pentadbir guna **Jana kata laluan sementara** di tab Pengguna.

**Kawalan keselamatan kod**: sah 10 minit, 5 cubaan salah, jeda 60 saat antara permintaan, maksimum 5 kod sejam bagi setiap ID. Jawapan "lupa kata laluan" sentiasa umum (tidak mendedahkan sama ada ID/emel wujud).

Syarat kata laluan: minimum 8 aksara, ada huruf dan nombor. Disimpan sebagai hash SHA-256 + rahsia `PEPPER` (Script Properties).

**Tetapan di atas `Code.gs`**
- `AUTO_LULUS = true` — akaun aktif selepas emel disahkan. `false` = perlu kelulusan pentadbir juga.
- `SAHKAN_EMEL = true` — wajib sahkan emel. Jangan tukar ke `false` kecuali untuk ujian (tanpa pengesahan, sesiapa boleh daftar guna ID staf orang lain).
- `DOMAIN_EMEL = []` — cth. `['moh.gov.my']` untuk terima emel rasmi sahaja.
- `PELULUS = ['ADMIN','DIETETIK']` — siapa boleh gantung/lanjut akaun dan jana kata laluan sementara.

**Had emel**: Apps Script akaun Gmail biasa boleh hantar ~100 emel sehari; akaun Google Workspace ~1,500. Setiap pendaftaran/reset guna 1 emel. Untuk pilot 1–2 wad, akaun Gmail biasa memadai. Semasa `setup()`, benarkan kebenaran "Send email as you".

Import pukal: tampal terus ke tab `Pengguna` dengan `Status_Akaun = Aktif`, `Emel_Disahkan = YA` dan kata laluan sementara (teks biasa). Ia ditukar ke hash automatik pada log masuk pertama.

Had kuasa: DIETETIK hanya boleh urus akaun `STAF`. Hanya ADMIN boleh urus akaun lain dan tukar peranan. Tiada siapa boleh ubah akaun sendiri. Menggantung akaun terus melog keluar pengguna itu.

## Peraturan perniagaan (boleh ubah di atas `Code.gs`)

- Menu hanya muncul kepada staf jika **Disahkan** DAN dimasukkan ke `Menu_Harian` untuk tarikh + sesi tersebut.
- Pengusaha ubah menu yang sudah disahkan → status kembali **Draf** (perlu disahkan semula).
- `CATUAN_MAX_HIDANGAN = 1` — satu catuan tanggung 1 hidangan. Catuan ditanda `Dituntut` sebaik pesanan dihantar.
- Harga dikira di pelayan (harga dari frontend diabaikan).
- Semua tulisan guna `LockService` — selamat untuk pesanan serentak.
- Tarikh ikut zon `Asia/Kuala_Lumpur` (elak pepijat `toISOString()` yang guna UTC, ada dalam draf asal — pesanan antara 12:00 malam–8:00 pagi akan tersalah hari).

## Nota keselamatan (sebelum guna sebenar)

- Kata laluan disimpan sebagai hash SHA-256 bersama rahsia (`PEPPER` dalam Script Properties). **Jangan padam atau ubah `PEPPER`**, semua kata laluan akan jadi tidak sah.
- Jangan guna No. Kad Pengenalan sebagai ID staf (PDPA). Hadkan akses Sheet kepada pentadbir sahaja.
- Sesi log masuk tamat selepas 6 jam.
- Untuk produksi seluruh hospital: pertimbangkan log masuk Google Workspace KKM.
- Apps Script menghadkan kira-kira 30 pelaksanaan serentak — memadai untuk pilot 1–2 wad. Jika skala seluruh hospital, pantau kelewatan waktu puncak tengah hari.
