# AIPP Voice Trainer (Colab)

Notebook Google Colab untuk **melatih model suara RVC dari rekaman suara Anda sendiri** memakai [Applio](https://github.com/IAHispano/Applio). Hasilnya berupa `NamaModel.zip` (berisi `.pth` dan `.index`) yang langsung bisa dipakai di notebook **AIPP Voice Studio**.

File notebook: `AIPP_Voice_Trainer_Colab.ipynb`
Pasangannya untuk memakai model: `AIPP_Voice_Studio_Colab.ipynb` (lihat `README_AIPP_Voice_Studio_Colab.md`).

---

## Alur singkat

```
Rekaman Anda -> cek kualitas -> preprocess + ekstraksi fitur -> training
             -> uji dengar beberapa checkpoint -> pilih terbaik -> ekspor zip
```

---

## Siapkan rekaman dulu (penentu kualitas)

Kualitas model mengikuti kualitas rekaman. Ini bagian terpenting.

- **Durasi:** 30 detik **tidak cukup** untuk training. Minimal 5 menit, **15–30 menit** untuk hasil bagus. Makin banyak dan bervariasi, makin stabil.
- **Tempat:** ruangan tenang dan tidak bergema (kamar berkarpet, dekat lemari pakaian atau bantal). Matikan kipas dan AC. Tanpa musik latar.
- **Mikrofon:** satu mikrofon yang konsisten, jarak mulut sekitar 10–15 cm, volume tidak sampai pecah (clipping). Simpan sebagai WAV bila bisa.
- **Isi:** bacakan teks yang beragam dengan ekspresi: berita, cerita, kalimat tanya dan seru, angka, nada santai dan serius. Rekaman datar menghasilkan suara datar.
- **Untuk cover lagu:** sertakan 5–10 menit Anda menyanyi (tanpa musik) dengan jangkauan nada bervariasi. Model dari rekaman bicara saja sering goyang saat menyanyi.
- **Format:** WAV, MP3, M4A, FLAC, OGG, AAC, OPUS, WEBM, atau WMA. Boleh banyak file, atau satu `.zip` berisi banyak rekaman.
- Tidak perlu transkrip.
- Gunakan hanya **suara Anda sendiri** atau suara orang yang memberi izin eksplisit.

---

## Yang Anda butuhkan

1. Akun Google dan Google Colab.
2. **Runtime GPU (T4)**. Notebook akan berhenti dengan pesan jelas bila GPU tidak aktif.
3. **Ruang Google Drive yang cukup.** Folder training menyimpan hasil preprocess, fitur, dan bobot per checkpoint, sehingga bisa membesar.
4. Rekaman suara Anda.

---

## Cara pakai

1. Buka notebook di Colab. *Runtime > Change runtime type*, pilih **T4 GPU**.
2. Jalankan sel 1 sampai 9 berurutan.

| Sel | Fungsi |
|---|---|
| **1** | Pengaturan (nama model, epoch, sample rate, dll.), cek GPU, sambungkan Google Drive. |
| **2** | Memasang Applio (versi dikunci) dan model dasar. Tahap pip paling lama. Progres dan jam tiap tahap ditampilkan. |
| **3** | Mengunggah rekaman (file satuan atau zip), atau membaca dari folder Drive lewat `DATA_DRIVE`. Dikonversi ke WAV mono 44,1 kHz. |
| **4** | **Cek kualitas rekaman**: total menit, tingkat bising, dan clipping per file, lengkap dengan peringatan. |
| **5** | Preprocess (potong otomatis, filter rumble) dan ekstraksi fitur. |
| **6** | **Training.** Bagian paling lama. Jangan tutup tab. |
| **7** | Membuat file index bila belum ada (biasanya sudah otomatis di akhir training). |
| **8** | Uji dengar: 6 checkpoint terakhir diubah dari audio uji yang sama untuk dibandingkan. |
| **9** | Ekspor epoch pilihan Anda ke zip, lalu unduh dan simpan di Drive. |

3. Dengarkan hasil sel 8, pilih epoch yang paling natural, isi `CHOSEN_EPOCH` di sel 9, lalu jalankan.
4. Unggah zip hasil ke sel 2 notebook **AIPP Voice Studio**.

---

## Pengaturan di sel 1

| Variabel | Nilai awal | Keterangan |
|---|---|---|
| `MODEL_NAME` | `Irwin_v3` | Nama model, tanpa spasi atau simbol. Nama baru = folder baru, model lama tidak tertimpa. |
| `SAMPLE_RATE` | `40000` | 40000 seimbang (disarankan). 48000 kualitas lebih tinggi tetapi lebih berat. |
| `TOTAL_EPOCH` | `300` | Jangan berlebihan. Terlalu banyak epoch membuat suara pecah (overtrain). |
| `SAVE_EVERY` | `25` | Simpan bobot tiap N epoch. Nilai lebih kecil (mis. 10) mengurangi epoch yang terulang bila terputus, tetapi lebih sering menulis ke Drive. |
| `BATCH_SIZE` | `8` | Naikkan ke 12 bila memori GPU lega, turunkan ke 4 bila out-of-memory. |
| `F0_METHOD` | `rmvpe` | Metode ekstraksi pitch. |
| `VOCODER` | `HiFi-GAN` | Biarkan, agar model cocok dengan notebook AIPP Voice Studio. |
| `SAVE_TO_DRIVE` | `True` | Simpan folder kerja di Drive agar training bisa dilanjutkan. |

---

## Struktur folder di Google Drive

```
Drive Saya/AIPP_RVC/
  logs/
    NamaModel/        <- ruang kerja training: hasil preprocess, fitur,
                         checkpoint, bobot per checkpoint, file index
  hasil/
    NamaModel.zip     <- model jadi (.pth + .index) setelah sel 9
    NamaModel/        <- isi yang sama dalam bentuk file terpisah
```

- **`logs/`** membuat training bisa dilanjutkan setelah terputus. Jangan dihapus selama Anda masih mungkin melanjutkan atau membandingkan checkpoint.
- **`hasil/`** berisi keluaran akhir. Simpan zipnya, dan salin ke tempat lain sebagai cadangan.

---

## Kapan modelnya muncul

- **Otomatis, tiap `SAVE_EVERY` epoch:** file bobot seperti `NamaModel_25e_825s.pth` disimpan di `logs/NamaModel/`.
- **Otomatis, di akhir training:** training berhenti sendiri di `TOTAL_EPOCH`, bobot terakhir disimpan, dan file index dibuat.
- **Manual:** Anda yang mendengarkan checkpoint (sel 8), memilih epoch, dan mengekspor zip (sel 9).

**Pilih epoch dengan telinga, bukan angka loss.** Nilai `lowest_value` di log adalah loss generator dan tidak otomatis berarti epoch yang paling natural. Checkpoint terakhir juga belum tentu terbaik.

---

## Melanjutkan training yang terputus

Colab gratis bisa memutus sesi. Checkpoint di Drive tetap aman.

1. Jalankan **sel 1**, lalu **sel 2** (paket yang terpasang hilang saat sesi direset).
2. **Lewati sel 3, 4, dan 5.** Hasil preprocess dan fitur sudah ada di Drive.
3. Jalankan **sel 6**. Applio memuat checkpoint terakhir dan melanjutkan.

Syaratnya `MODEL_NAME`, `SAMPLE_RATE`, dan pengaturan lain sama persis seperti sebelumnya, serta `SAVE_TO_DRIVE = True`. Progres sejak penyimpanan terakhir (maksimal `SAVE_EVERY` − 1 epoch) bisa terulang.

---

## Membuat model baru

1. Di sel 1, ganti `MODEL_NAME` (mis. `Irwin_v4`).
2. Jalankan sel 1 dan 2, lalu sel 3 sampai 6 dengan rekaman untuk model baru.
3. Model lama tetap ada di `logs/NamaLama` dan `hasil/`.

Untuk memperbaiki suara yang sama dengan rekaman tambahan, gabungkan rekaman lama dan baru dalam satu dataset lalu latih sebagai model baru dengan nama baru. Menjalankan sel 5 lagi dengan `MODEL_NAME` lama tidak disarankan, karena belum dipastikan apakah data ditambahkan atau menimpa.

**Simpan rekaman asli di Drive** dan isi `DATA_DRIVE` di sel 3. Folder `/content/dataset` hilang saat sesi habis dan dihapus setiap sel 3 dijalankan.

---

## Waktu dan kuota Colab gratis

- Panel **Resource** (klik indikator RAM/Disk di kanan atas) menampilkan perkiraan sisa waktu runtime. Itu perkiraan, bukan jaminan. Pada pemakaian saat dokumen ini ditulis, tampil sekitar 3 jam 20 menit di awal sesi.
- Sesi bisa berakhir lebih awal bila tab ditutup, komputer tidur, internet putus, atau kuota GPU habis. Tidak ada jam reset yang tetap.
- Sisakan paling sedikit **20 menit** untuk sel 7 sampai 9. Bila sisa runtime menipis dan training belum selesai, pilih *Runtime > Interrupt execution*, jalankan sel 7 (bila index belum ada), lalu sel 8 dan 9 untuk mengekspor checkpoint terakhir yang tersimpan.
- Sebagai gambaran dari satu uji: dengan 33 batch per epoch di T4, kecepatannya sekitar 22–31 detik per epoch. Ini bergantung pada jumlah data dan kondisi GPU, jadi pantau kolom `training_speed` di log Anda sendiri.

---

## Pemecahan masalah

| Masalah | Penanganan |
|---|---|
| Sel 2 sangat lama | Normal sampai belasan menit karena mengunduh PyTorch dan paket besar. Bila log jam tidak bergerak jauh lebih lama, *Runtime > Interrupt execution* lalu ulangi sel 1 dan 2. |
| Teks merah "dependency resolver ... incompatible" di sel 2 | Peringatan, bukan error. Applio mengunci versi paket yang berbeda dari bawaan Colab. Paket Colab yang disebut tidak dipakai training. Lanjutkan bila akhir sel menyatakan "Prerequisites downloaded and installed successfully". |
| Error numpy atau versi paket | *Runtime > Restart session*, lalu ulangi dari sel 1. |
| GPU tidak terbaca setelah sel 2 | Restart session, jalankan sel 1, dan cek ulang `torch.cuda.is_available()`. |
| `GPU tidak aktif` di sel 1 | *Change runtime type > T4 GPU*. |
| Peringatan kualitas di sel 4 | Perbaiki rekaman (kurang durasi, bising, clipping), lalu unggah ulang sebelum training. |
| Out of memory saat training | Turunkan `BATCH_SIZE` (mis. 4). |
| Sel 8: "File index belum ada" | Jalankan sel 7 setelah training selesai. |
| Suara hasil pecah atau serak | Kemungkinan overtrain. Bandingkan checkpoint lebih awal di sel 8 dan pilih yang lebih natural. |
| Sesi terputus | Ikuti bagian "Melanjutkan training yang terputus". |

---

## Status pengujian

- **Sudah diuji:** sel konversi audio dan sel cek kualitas (dengan rekaman buatan; sel ini mendeteksi file yang pecah dan bising), serta pemeriksaan sintaks semua sel.
- **Dicocokkan dengan kode sumber:** perintah preprocess, extract, train, index, dan infer dicocokkan dengan kode Applio versi `6d5d951` (versi yang dikunci di sel 2). Perilaku melanjutkan training dari checkpoint juga diperiksa di kode sumber.
- **Dijalankan oleh pengguna di Colab:** tahap pemasangan dan training berjalan sampai ratusan step pada uji pengguna. Kualitas suara akhir tidak dapat dijamin dan baru bisa dinilai dengan mendengarkan hasilnya.

---

## Catatan

- Hasil tidak dijanjikan sempurna. Kualitas mengikuti kualitas rekaman dan jumlah datanya.
- Vocoder dibiarkan HiFi-GAN agar model kompatibel dengan notebook AIPP Voice Studio. Vocoder lain (MRF HiFi-GAN, RefineGAN) hanya berfungsi penuh di Applio.
- Gunakan model hanya untuk suara Anda sendiri atau suara orang yang memberi izin eksplisit. Jangan dipakai untuk meniru orang lain atau menipu.
- Lisensi Applio dan komponennya berlaku sesuai repositori masing-masing. Periksa sebelum penggunaan komersial.
