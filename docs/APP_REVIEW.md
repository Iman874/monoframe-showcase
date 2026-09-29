# Review App — Monoframe

Review jujur dari sisi yang membangunnya. Bagian yang kuat dan bagian yang masih mahal.

## Yang bekerja dengan baik

### 1. Offline bukan lagi kondisi darurat

Kios tetap menerima dan menangkap foto saat jaringan hilang. Capture masuk antrean lokal, lalu disinkronkan begitu koneksi pulih. Secara bisnis artinya sederhana: tidak ada momen pelanggan yang gagal hanya karena sinyal.

Konsekuensinya tidak gratis: setiap aset harus punya identitas yang stabil sejak dibuat di mesin, bukan sejak masuk server. Kalau tidak, reupload pasca-sinkron menghasilkan duplikat atau aset yatim.

### 2. Operator bisa melihat, bukan menebak

Dashboard admin dipakai untuk hal konkret: stok kertas, printer mana yang macet, sesi mana yang gagal unggah, dan voucher yang terpakai. Fitur audit sesi + "generate ulang" yang sekarang sedang digarap menutup celah terakhir: sebelumnya aset yang gagal harus ditangani manual oleh teknisi.

### 3. Batas tanggung jawab antar service tegas

Tujuh service, tujuh repository. Aturan kepemilikannya eksplisit: satu perubahan dikerjakan di service yang punya logikanya, tidak mencampur logika lintas service dalam satu task. Ini bukan gaya-gayaan arsitektur — dengan tim efektif satu orang, batas tegas adalah satu-satunya cara supaya perubahan minggu ini tidak merusak hal lain tanpa ketahuan.

### 4. Versi dari satu sumber

`VERSIONS.ini` jadi single source of truth untuk versi layanan Go, dan build selalu lewat skrip (tidak pernah `go build` manual). Alasannya praktis: hasil build manual bisa menyisipkan metadata versi yang tidak konsisten, lalu kegagalan baru muncul di mesin kios, bukan di laptop.

## Yang masih mahal

### 1. Flutter desktop sebagai pusat kendali

Aplikasi kios memuat state aplikasi, antrean lokal, kontrol perangkat, dan render UI dalam satu proses desktop. Setiap perubahan di root widget dan lifecycle punya risiko efek samping yang tidak sekecil terlihat — gesture arena, repaint boundary, dan reaktivitas timer bereaksi lintas seluruh halaman. Karena itu aturan internal melarang mengubah tipe kelas fundamental (`StatelessWidget`/`StatefulWidget`, observer, perilaku scroll) tanpa penjelasan dampak dan persetujuan eksplisit.

### 2. Terlalu banyak route

392 endpoint REST untuk satu produk, dengan v1 dan v2 hidup berdampingan. Ini utang yang terasa: permukaan pengujian melebar, dan memahami mana yang masih dipakai butuh penelusuran manual.

### 3. Keamanan harus ditebus belakangan

Audit keamanan internal dijalankan secara rutin, dan hasilnya ditindaklanjuti. Isi laporan tidak dipublikasi di sini, bahkan kelas temuan dan lokasinya: membuka itu pada sistem yang sedang berjalan sama dengan membagikan peta serangan. Yang bisa disimpulkan dari repo ini hanya bahwa audit ada, berulang, dan memengaruhi cara fitur berikutnya dirancang.

### 4. Test kuat di area yang disentuh, tipis di area yang tidak

156 test Flutter dan 71 test Laravel terkonsentrasi di jalur kritis (cetak, sinkronisasi, sesi, pembayaran). Layar kosmetik dan tool setup masih sedikit cakupannya.

## Pelajaran teknis yang paling berharga

- **Ukur dulu, baru setel konstanta.** Ketika sebuah jalur terasa lambat, kesalahan pertamaku adalah menaikkan retry dan menurunkan timeout. Yang benar: mengukur biaya query yang kutulis sendiri — ternyata lambatnya dari query itu, bukan dari konstanta antrian.
- **Memindahkan baris kotor ke tempat yang lebih rapi bukan perbaikan.** Perbaikan berarti mengubah apa yang ditulis mulai sekarang, bukan merelabel history.
- **Batas kita dan batas API eksternal berbeda.** Pagination 1.000 objek per halaman di object storage adalah batas penyedia, bukan keputusan yang bisa "dihapus" dari kode kita.
