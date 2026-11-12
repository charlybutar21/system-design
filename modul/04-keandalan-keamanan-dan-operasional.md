# 🛡️ Bab IV: Komponen Sistem Lanjut, Keandalan (Resiliency) & Keamanan

> **🎯 Tujuan Belajar Bab Ini:**
> Apa yang membedakan aplikasi tugas kuliah dengan sistem kelas industri (Production-Grade)? Jawabannya ada di bab ini: kemampuan mencari lokasi spasial secara cepat (**Geohashing & Quadtrees**), bertahan saat ada komponen yang gagal (**Circuit Breaker, Rate Limiting, Idempotency, Disaster Recovery**), menemukan layanan secara dinamis (**Service Discovery & Service Mesh**), serta mengamankan identitas dan trafik pengguna (**OAuth 2.0, OIDC, SSO, SAML, TLS/mTLS**).

---

## Daftar Isi Bab IV
1. [Geohashing dan Quadtrees (Pencarian Lokasi Spasial)](#1-geohashing-dan-quadtrees)
2. [Circuit Breaker (Pemutus Arus Kegagalan Berantai)](#2-circuit-breaker)
3. [Rate Limiting (Pembatasan Laju Request)](#3-rate-limiting)
4. [Idempotency (Keamanan Eksekusi Berulang)](#4-idempotency-idempotensi)
5. [Service Discovery & Service Mesh](#5-service-discovery--service-mesh)
6. [SLA, SLO, dan SLI](#6-sla-slo-dan-sli)
7. [Disaster Recovery (Pemulihan Bencana: RTO & RPO)](#7-disaster-recovery-pemulihan-bencana)
8. [Virtual Machines (VMs) vs Containers](#8-virtual-machines-vms-dan-containers)
9. [OAuth 2.0 dan OpenID Connect (OIDC)](#9-oauth-20-dan-openid-connect-oidc)
10. [Single Sign-On (SSO) dan SAML](#10-single-sign-on-sso-dan-saml)
11. [SSL, TLS, dan mTLS](#11-ssl-tls-dan-mtls)

---

## 1. Geohashing dan Quadtrees

Bagaimana cara aplikasi seperti **Gojek, Grab, Uber, Google Maps, atau Tinder** mencari *"Pengemudi atau Restoran terdekat dalam radius 1 km dari titik GPS saya"* di antara jutaan titik di seluruh dunia dalam waktu kurang dari 10 milidetik?

Jika kita menggunakan query SQL biasa:
```sql
SELECT * FROM drivers WHERE lat BETWEEN X-R AND X+R AND long BETWEEN Y-R AND Y+R;
```
Query dua dimensi (`lat` DAN `long`) di atas **sangat lambat pada jutaan baris data** karena index B-Tree tradisional hanya optimal untuk pencarian satu dimensi! Untuk mengatasinya, kita menggunakan **Geohashing** atau **Quadtrees**.

### 1. Geohashing
**Geohashing** (diciptakan oleh Gustavo Niemeyer, 2008) adalah metode menyandikan koordinat geografis dua dimensi (`Latitude, Longitude`) menjadi **satu string pendek alfanumerik (Base-32)**.
- *Contoh*: Koordinat San Francisco `37.7564, -122.4016` diubah menjadi string Geohash **`9q8yy9mf`**.

#### Bagaimana Cara Kerja Geohashing?
Peta seluruh dunia dibagi menjadi **32 kotak (sel)** besar, di mana setiap kotak diberi 1 karakter pertama. Kemudian kotak tersebut dibagi lagi menjadi 32 kotak yang lebih kecil (karakter kedua), dan begitu seterusnya secara rekursif!

![geohashing](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-IV/geohashing-and-quadtrees/geohashing.png)

- **Sifat Ajaib Geohashing**: Semakin panjang **awalan (*prefix*) string yang sama** antara dua lokasi, semakin dekat jarak kedua lokasi tersebut!
  - Contoh: Driver di `9q8yy9mf` dan Penumpang di `9q8yy9vx` memiliki 6 karakter awal yang sama (`9q8yy9`), artinya mereka berada di dalam blok jalan yang sama!
  - Pencarian lokasi terdekat berubah menjadi pencarian string biasa (`WHERE geohash LIKE '9q8yy9%'`) yang **super cepat** diindeks oleh database apa pun (Redis, MySQL, PostgreSQL, DynamoDB)!

#### Tabel Ukuran Presisi Geohash Berdasarkan Panjang Karakter

| Panjang Geohash | Lebar Area (Width) | Tinggi Area (Height) | Contoh Skala Area |
| :---: | :--- | :--- | :--- |
| **1** | 5.000 km | 5.000 km | Benua |
| **2** | 1.250 km | 1.250 km | Negara besar |
| **3** | 156 km | 156 km | Provinsi |
| **4** | 39,1 km | 19,5 km | Kota metropolitan |
| **5** | 4,89 km | 4,89 km | Kecamatan |
| **6** | **1,22 km** | **0,61 km** | **Kelurahan / Radius cari ojek (~1 km)** ⭐ |
| **7** | 153 m | 153 m | Blok gedung / jalan |
| **8** | 38,2 m | 19,1 m | Satu bangunan rumah |
| **9** | 4,77 m | 4,77 m | Posisi mobil di parkiran |
| **10 – 12** | 1,19 m s/d 3,7 cm | 0,59 m s/d 1,8 cm | Titik presisi sangat tinggi |

### 2. Quadtrees
**Quadtree** adalah struktur data pohon (*tree*) di dalam memori di mana setiap node internal memiliki **tepat 4 anak (4 kuadran: Barat Laut/NW, Timur Laut/NE, Barat Daya/SW, Tenggara/SE)**.

![quadtree](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-IV/geohashing-and-quadtrees/quadtree.png)

#### Mengapa Quadtree Lebih Pintar dari Geohash di Area Padat vs Sepi?
Pada Geohash, ukuran kotak selalu tetap tanpa peduli apakah area itu adalah **tengah laut Pasifik (0 pengemudi)** atau **Bundaran HI Jakarta saat jam pulang kantor (5.000 pengemudi)**.
Sebaliknya, **Quadtree bersifat adaptif**:
- Sebuah kotak hanya akan dibelah menjadi 4 kuadran kecil **jika jumlah titik (pengemudi/tempat) di dalam kotak tersebut melebihi batas tertentu** (misal > 500 titik).
- Di tengah hutan/laut yang sepi, 1 node Quadtree bisa mencakup area ratusan kilometer persegi (hemat memori!). Di pusat kota yang padat, Quadtree otomatis membelah diri hingga menjadi kotak-kotak berukuran beberapa meter saja!

![quadtree-subdivision](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-IV/geohashing-and-quadtrees/quadtree-subdivision.png)

---

## 2. Circuit Breaker

Dalam sistem Microservices, Service A sering memanggil Service B melalui jaringan. Bagaimana jika Service B sedang mengalami gangguan berat dan butuh 30 detik sebelum *timeout*? Jika ada 10.000 *request* masuk ke Service A, maka seluruh *thread* dan memori di Service A akan ikut habis karena menunggu Service B! Akibatnya, Service A ikut mati, lalu Service C yang memanggil Service A juga ikut mati—terjadilah bencana **Cascading Failure (Kegagalan Berantai / Efek Domino)**!

**Circuit Breaker** adalah pola desain yang membungkus panggilan ke layanan eksternal, memantau jumlah kegagalan, dan **memutus arus panggilan secara otomatis (*Fail Fast*)** ketika jumlah error melewati batas ambang (*threshold*).

![circuit-breaker](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-IV/circuit-breaker/circuit-breaker.png)

> **💡 Analogi Pemula:**
> Persis seperti **MCB / Sekring Listrik di Rumahmu**: Saat terjadi korsleting di setrika, sekring langsung "jepret" turun (memutus arus) dalam sepersekian detik agar **seluruh rumah tidak kebakaran**. Setelah setrika yang rusak dicabut dan dingin, barulah sekring dinaikkan kembali.

### 3 Status (States) pada Circuit Breaker
1. **Closed (Tertutup — Kondisi Normal)**:
   - Arus lalu lintas berjalan normal; semua *request* diteruskan ke Service tujuan.
   - Circuit Breaker menghitung jumlah error/timeout. Jika error melewati batas (misal > 50% gagal dalam 10 detik terakhir), status langsung berpindah ke **Open**.
2. **Open (Terbuka — Arus Diputus!)**:
   - Semua *request* baru langsung **ditolak seketika (*fail fast*)** atau dialihkan ke *fallback response* (misal menampilkan data dari cache) **tanpa menyentuh Service yang sedang sakit sama sekali**! Ini memberi waktu bagi Service yang sakit untuk bernapas dan pulih.
   - Setelah waktu tunggu (*cooldown timeout*, misal 30 detik) berlalu, status berpindah ke **Half-Open**.
3. **Half-Open (Setengah Terbuka — Uji Coba Pemulihan)**:
   - Circuit Breaker mengizinkan **sejumlah kecil *request* percobaan** (misal 5 request) untuk masuk ke Service tujuan.
   - **Jika berhasil**: Berarti Service sudah sembuh → kembali ke status **Closed** (normal).
   - **Jika masih gagal**: Berarti Service masih sakit → langsung kembali ke status **Open** dan mulai hitung mundur *timeout* lagi!

---

## 3. Rate Limiting

**Rate Limiting** adalah mekanisme pertahanan untuk membatasi jumlah *request* yang boleh dilakukan oleh seorang pengguna, alamat IP, atau aplikasi dalam rentang waktu tertentu (misalnya: *"Maksimal 100 request per menit per User"*). Jika melebihi batas, server menolak request dengan status **`HTTP 429 Too Many Requests`**.

![rate-limiting](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-IV/rate-limiting/rate-limiting.png)

### Mengapa Rate Limiting Sangat Penting?
1. **Mencegah Serangan DDoS, Brute-Force Password, & Web Scraping**: Menghentikan bot jahat yang menembak endpoint login ribuan kali per detik.
2. **Mengendalikan Biaya Operasional Cloud**: Mencegah *auto-scaling* membengkak tak terkendali akibat bug *infinite loop* di aplikasi klien yang bisa menyebabkan tagihan Cloud ratusan juta rupiah.
3. **Menjaga Ketersediaan (Availability) Bersama**: Mencegah satu pengguna rakus (*noisy neighbor*) menghabiskan seluruh kapasitas server.

### 5 Algoritma Rate Limiting (Wajib Dikuasai untuk Interview!)

| Algoritma | Cara Kerja & Analogi | Kelebihan | Kekurangan |
| :--- | :--- | :--- | :--- |
| **1. Token Bucket** ⭐ *(Paling Populer: AWS, Stripe)* | Bayangkan sebuah ember yang menampung maksimal $B$ koin token. Setiap detik, sistem mengisi ulang $R$ koin ke ember. Setiap *request* yang datang wajib mengambil **1 koin**. Jika koin habis, *request* ditolak (`429`). | Sangat hemat memori, mudah dibuat, dan **mengizinkan lonjakan trafik singkat (*burst traffic*)** selama koin di ember masih ada. | Perlu menyetel 2 parameter (ukuran ember & laju pengisian). |
| **2. Leaky Bucket** *(Shopify, NGINX)* | Bayangkan ember yang bagian bawahnya berlubang kecil sehingga air menetes keluar dengan **kecepatan konstan yang tetap**. *Request* yang masuk ditampung di antrean ember; jika ember meluap, *request* baru dibuang. | Mengubah trafik yang fluktuatif menjadi **aliran beban yang sangat stabil dan rata (*smooth outflow*)** ke server backend. | Jika ada lonjakan trafik sah, *request* baru di antrean belakang harus menunggu lama. |
| **3. Fixed Window Counter** | Membagi waktu menjadi jendela tetap (misal tiap 1 menit: `10:00–10:01`). Setiap request menaikkan *counter* `+1`. Jika melewati batas (misal > 100), ditolak sampai menit berikutnya. | Paling sederhana dan sangat hemat memori (cukup 1 angka *counter* di Redis dengan TTL). | **Kelemahan di Perbatasan Waktu (*Boundary Spike*)**: Jika user mengirim 100 request pada detik `10:00:59` dan 100 request lagi pada `10:01:01`, maka dalam 2 detik server menerima **200 request** (2x lipat batas)! |
| **4. Sliding Window Log** | Menyimpan catatan *timestamp* tepat untuk setiap request di dalam *Sorted Set* (misal Redis `ZSET`). Saat request baru datang, hapus log yang lebih tua dari 1 menit lalu, lalu hitung jumlah log yang tersisa. | **100% Akurat** di setiap titik waktu; tidak ada celah di perbatasan menit. | **Boros Memori**: Karena harus menyimpan ribuan *timestamp* untuk setiap user aktif. |
| **5. Sliding Window Counter** ⭐ *(Cloudflare)* | Pendekatan hibrida cerdas: menggabungkan *counter* menit lalu dan menit sekarang dengan **bobot persentase waktu berjalan** (misal di detik ke-15 menit baru: `75% × counter_menit_lalu + counter_menit_ini`). | Sangat hemat memori seperti *Fixed Window*, namun berhasil meredam celah lonjakan di perbatasan waktu! | Merupakan pendekatan estimasi (namun akurasinya >99,9% di dunia nyata). |

### Tantangan Rate Limiting di Sistem Terdistribusi
1. **Inconsistency Antar Node**: Jika kita punya 5 server API Gateway yang masing-masing menghitung batas di memori lokalnya sendiri, pengguna bisa mengakali batas 100 req/menit menjadi 500 req/menit!
   - **Solusi**: Gunakan penyimpanan terpusat super cepat seperti **Redis** untuk menyimpan *counter* seluruh gateway.
2. **Race Condition (Kondisi Balapan)**: Jika dua request dari user yang sama masuk pada milidetik yang persis sama dan kita melakukan pola naif *"Baca nilai counter dari Redis (misal 99) → tambah 1 di kode → Simpan 100 ke Redis"*, keduanya akan sama-sama membaca `99` dan menyimpan `100` (padahal seharusnya `101`)!
   - **Solusi Arsitek**: Jangan gunakan *get-then-set* biasa! Gunakan perintah atomik Redis seperti **`INCR`** (atau **Redis Lua Script**) yang menjalankan penambahan dan pengecekan batas dalam **satu operasi atomik tunggal** tanpa perlu *distributed lock* yang lambat.

---

## 4. Idempotency (Idempotensi)

Sebuah operasi disebut **Idempotent** jika **menjalankannya berkali-kali memberikan hasil akhir (kondisi state server) yang persis sama seperti menjalankannya satu kali saja**.

> **💡 Analogi Pemula:**
> - **Contoh Idempotent**: Menekan tombol **panggil lift ke lantai 5** atau memberi perintah `"Ubah alamat email Budi menjadi budi@mail.com"`. Mau kamu tekan tombol lift itu 1 kali atau 50 kali karena tidak sabar, lift tetap hanya akan datang 1 kali ke lantai 5!
> - **Contoh TIDAK Idempotent**: Perintah `"Kurangi saldo Budi sebesar Rp100.000"`. Jika dijalankan 3 kali karena koneksi internet patah-patah (*retry*), saldo Budi akan terpotong **Rp300.000**!

### Mengapa Idempotensi Adalah Aturan Wajib Sistem Finansial & Terdistribusi?
Bayangkan pelanggan menekan tombol **"Bayar Sekarang"**. Permintaan sampai ke server bank dan saldo berhasil dipotong, tetapi **tepat saat server bank mengirim balasan "Sukses", sinyal internet HP pelanggan putus (*timeout*)!**
Aplikasi HP pelanggan tidak tahu apakah uangnya sudah terpotong atau belum, sehingga otomatis melakukan **Retry (mengirim ulang request pembayaran)**. Tanpa **Idempotency**, pelanggan akan **terdebet dua kali (*double charge*)**!

### Tabel Idempotensi pada Metode HTTP

| Metode HTTP | Idempotent? | Penjelasan |
| :--- | :---: | :--- |
| **`GET`** | **Ya** ✅ | Hanya membaca data tanpa mengubah state di server. |
| **`PUT`** | **Ya** ✅ | Menimpa seluruh resource dengan isi yang sama; diulang 10x hasilnya tetap sama. |
| **`DELETE`** | **Ya** ✅ | Menghapus data yang sama berulang kali membuat data tersebut tetap terhapus di server (meski respons kedua bisa `404`). |
| **`POST`** | **Tidak** ❌ | Secara default membuat data/transaksi baru setiap kali dipanggil. **Wajib dipasang `Idempotency-Key`!** |
| **`PATCH`** | **Tidak** ❌ | Bisa tidak idempotent jika operasinya bersifat relatif (misal *"tambah stok +5"*). |

### Cara Kerja `Idempotency-Key` (Standar Stripe, GoPay, & Perbankan)
Untuk membuat operasi `POST /payments` menjadi aman di-*retry*:
1. **Client men-generate UUID unik** (`Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000`) saat tombol bayar ditekan, lalu mengirimkannya di HTTP Header.
2. **Server memeriksa kunci tersebut secara Atomik** di database/Redis (misal menggunakan `SET key "IN_PROGRESS" NX EX 86400` atau *Unique Constraint* di tabel database):
   - **Jika Kunci Belum Pernah Ada**: Server memproses pembayaran, menyimpan hasil respons (`200 OK` beserta detail struk) terhadap kunci tersebut, lalu mengembalikan respons ke klien.
   - **Jika Kunci Sedang Diproses (`IN_PROGRESS`)**: Jika retry datang saat request pertama masih berjalan, server mengembalikan `409 Conflict` (atau menahan sampai selesai).
   - **Jika Kunci Sudah Selesai Diproses**: Server **langsung mengembalikan respons sukses yang tersimpan sebelumnya tanpa memotong saldo lagi!**
3. **Request Fingerprint**: Server juga menyimpan *hash* dari isi *body request*. Jika klien mengirim `Idempotency-Key` yang sama tapi nominal uangnya berbeda, server menolak dengan error `422 Unprocessable Entity`.

### Idempotent Consumer pada Message Queue
Karena hampir semua Message Broker (Kafka, RabbitMQ, SQS) menggunakan pengiriman **At-Least-Once**, pesan yang sama pasti suatu saat akan terkirim 2 kali ke *Consumer*. Agar aman:
- Simpan `message_id` ke dalam **Tabel Deduplikasi** di dalam transaksi database yang sama dengan perubahan bisnis, ATAU
- Gunakan nomor versi (*optimistic locking / versioning*) sehingga event lama diabaikan.
- Kombinasi **At-Least-Once Delivery + Idempotent Consumer** menghasilkan apa yang disebut **Effectively-Once Processing**!

---

## 5. Service Discovery & Service Mesh

Di era Cloud dan Kubernetes, *microservices* berjalan di dalam *containers* yang terus-menerus hidup, mati, dan berpindah mesin (*auto-scaling*). Akibatnya, **alamat IP dan Port setiap service berubah setiap saat**. Lalu bagaimana *Order Service* tahu ke alamat IP mana ia harus menghubungi *Payment Service*? Jawabannya adalah **Service Discovery**.

### 2 Pola Utama Service Discovery

#### 1. Client-Side Discovery
Klien bertanya langsung ke **Service Registry** (buku telepon pusat yang menyimpan daftar IP semua service yang sedang aktif), lalu klien memilih salah satu IP (misal dengan algoritma *Round-Robin*) dan menghubunginya langsung.

![client-side-service-discovery](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-IV/service-discovery/client-side-service-discovery.png)

#### 2. Server-Side Discovery *(Paling Umum di Cloud & Kubernetes)*
Klien cukup mengirim request ke sebuah **Load Balancer / Router**, dan Load Balancer itulah yang bertanya ke *Service Registry* lalu meneruskan request ke instance service yang sehat. Klien tidak perlu tahu-menahu soal daftar IP!

![server-side-service-discovery](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-IV/service-discovery/server-side-service-discovery.png)

### Cara Pendaftaran Service (*Service Registration*)
- **Self-Registration**: Saat *container* service baru menyala, ia mendaftarkan dirinya sendiri ke *Service Registry* dan mengirimkan sinyal *heartbeat* berkala.
- **Third-Party Registration**: Komponen pengawas (seperti Kubernetes *Control Plane*) memantau container yang baru menyala atau mati, lalu otomatis memperbarui *Service Registry*.
- **Contoh Teknologi Service Registry**: **Consul**, **etcd**, **Apache ZooKeeper**, **Kubernetes DNS**.

### Service Mesh (Istio, Envoy, Linkerd)
Ketika jumlah microservices mencapai puluhan atau ratusan, mengatur *retry*, *circuit breaking*, *tracing*, dan enkripsi **mTLS** di dalam kode masing-masing bahasa pemrograman sangat merepotkan.
**Service Mesh** menyelesaikannya dengan menaruh proxy pendamping kecil (**Sidecar Proxy**, seperti **Envoy**) di sebelah setiap container microservice untuk menangani seluruh komunikasi jaringan, keamanan, dan observabilitas secara transparan di luar kode bisnis!

---

## 6. SLA, SLO, dan SLI

Ketiga istilah ini adalah fondasi dari **Site Reliability Engineering (SRE)** dan kontrak layanan Cloud:

> **💡 Analogi Pemula (Pesan Pizza Delivery):**
> - **SLI (*Service Level Indicator* — Angka Kenyataan di Lapangan)**: Catatan stopwatch aktual berapa menit kurir pizza sampai di rumah pelanggan bulan ini (misal: rata-rata **24 menit**, atau **99,95%** pesanan sampai di bawah 30 menit).
> - **SLO (*Service Level Objective* — Target Internal Tim Teknis)**: Target yang dipasang oleh Manajer Restoran untuk para kurirnya: *"Minimal **99,9%** pengantaran pizza harus sampai di bawah 30 menit!"*
> - **SLA (*Service Level Agreement* — Kontrak Hukum & Kompensasi ke Pelanggan)**: Janji resmi di brosur ke pelanggan: *"Jika pengantaran kami di bawah **99,0%** tepat waktu bulan ini, **uang Anda kembali / gratis voucher pizza**!"*

| Istilah | Kepanjangan | Siapa yang Membuat? | Definisi & Fokus Utama |
| :--- | :--- | :--- | :--- |
| **SLI** | *Service Level Indicator* | Tim Engineering / SRE | **Metrik terukur yang nyata** (misal: *Uptime = 99,98%*, *Latensi P99 = 120ms*, *Error Rate = 0,01%*). |
| **SLO** | *Service Level Objective* | Tim Engineering & Product | **Target angka yang wajib dicapai oleh SLI** (misal: *Target Uptime $\ge$ 99,95%*). Biasanya dibuat **lebih ketat daripada SLA** sebagai sabuk pengaman (*Error Budget*). |
| **SLA** | *Service Level Agreement* | Tim Bisnis & Legal dengan Klien | **Perjanjian kontrak resmi dengan pelanggan** yang mencantumkan **konsekuensi denda/kompensasi finansial** jika target layanan gagal dipenuhi (misal: *Kontrak Uptime $\ge$ 99,9%*). |

---

## 7. Disaster Recovery (Pemulihan Bencana)

**Disaster Recovery (DR)** adalah strategi dan proses untuk memulihkan akses serta fungsionalitas infrastruktur IT setelah terjadi bencana besar—seperti kebakaran data center, banjir, gempa bumi, atau serangan siber.

![disaster-recovery](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-IV/disaster-recovery/disaster-recovery.png)

### 2 Metrik Kunci Disaster Recovery: RTO vs RPO (Sering Keluar di Interview!)

1. **RPO (*Recovery Point Objective* — Berapa Banyak Data Masa Lalu yang Boleh Hilang?)**:
   - Mengukur **jarak waktu sejak backup terakhir sebelum bencana terjadi**.
   - *Contoh*: Jika **RPO = 1 jam**, maka sistem wajib mem-backup data minimal setiap 1 jam sekali, sehingga jika data center meledak jam `14:00`, data yang hilang maksimal hanya data dari jam `13:00–14:00`. Untuk bank, RPO mendekati **0 detik** (replikasi sinkron).
2. **RTO (*Recovery Time Objective* — Berapa Lama Waktu yang Dibutuhkan Sampai Sistem Nyala Kembali?)**:
   - Mengukur **durasi maksimal layanan boleh mati (*downtime*) setelah bencana terjadi hingga pulih normal**.
   - *Contoh*: Jika **RTO = 15 menit**, maka dalam waktu maksimal 15 menit sejak data center utama lumpuh, sistem cadangan harus sudah online melayani pengguna!

### Strategi Disaster Recovery (Dari Termurah hingga Terbaik)
1. **Backup & Restore**: Hanya menyimpan salinan *backup* di lokasi lain (misal Amazon S3 lintas region). Paling murah, tetapi RTO/RPO paling lama (butuh berjam-jam untuk menyalakan server dan me-restore data).
2. **Cold Site (Situs Dingin)**: Menyediakan ruang/infrastruktur dasar di lokasi kedua, tetapi server belum dinyalakan dan data belum disinkronkan secara aktif.
3. **Warm Standby (Situs Hangat)**: Salinan sistem berskala kecil sudah menyala dan menerima sinkronisasi data berkala di lokasi kedua; saat bencana terjadi tinggal di-*scale up*.
4. **Hot Site / Multi-Region Active-Active (Situs Panas)**: Lokasi kedua memiliki infrastruktur lengkap dengan data yang disinkronkan secara *real-time* setiap saat. Biaya paling mahal, namun menghasilkan **RTO dan RPO mendekati nol detik**!

---

## 8. Virtual Machines (VMs) dan Containers

![virtualization-vs-containerization](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-IV/virtual-machines-and-containers/virtualization-vs-containerization.png)

> **💡 Analogi Pemula:**
> - **Virtual Machine (VM)** ibarat **Rumah Tapak Komplek**: Setiap rumah memiliki fondasi sendiri, genset listrik sendiri, pompa air sendiri (**Guest OS lengkap**). Sangat terisolasi dengan kuat, tetapi membangunnya lama dan memakan lahan sangat luas.
> - **Container (Docker)** ibarat **Kamar Apartemen Modern**: Seluruh kamar berbagi satu fondasi gedung, pipa air, dan instalasi listrik utama yang sama (**Berbagi Kernel OS Host**), tetapi setiap kamar tetap terkunci rapat dan punya perabotan (*code & dependencies*) masing-masing. Sangat ringan, hemat tempat, dan bisa disiapkan dalam 1 detik!

### Tabel Perbandingan Virtual Machines (VMs) vs Containers

| Aspek | Virtual Machines (VMs) | Containers (Docker / containerd) |
| :--- | :--- | :--- |
| **Level Virtualisasi** | Memvirtualisasikan **Perangkat Keras Fisik (Hardware)** menggunakan **Hypervisor** (VMware, KVM, Hyper-V). | Memvirtualisasikan **Sistem Operasi (OS)**; semua container berbagi *OS Kernel* milik mesin induk. |
| **Sistem Operasi Tamu** | Setiap VM membawa **Guest OS utuh** (memakan ukuran Bergigabyte). | Hanya membawa kode aplikasi dan *library*-nya (ukuran hanya Megabyte). |
| **Waktu Booting / Startup** | Lambat ( puluhan detik hingga beberapa menit). | **Sangat Cepat** (milidetik hingga 1–2 detik). |
| **Penggunaan RAM & CPU** | Berat (*high overhead*). | **Sangat Ringan (*lightweight*)**. |
| **Isolasi Keamanan** | Isolasi level *hardware* (sangat ketat). | Isolasi level *proses OS* (*namespaces & cgroups*). |
| **Portabilitas** | Lebih sulit dipindah antar cloud provider berbeda. | **Sangat Portabel** (*"Jalan di laptop saya = Jalan di server produksi mana pun"*). |

---

## 9. OAuth 2.0 dan OpenID Connect (OIDC)

Sebelum membahas protokolnya, jangan pernah tertukar antara dua istilah ini:
- **Authentication (Autentikasi / `AuthN`)**: Memverifikasi **SIAPA KAMU** (Contoh: Login dengan username, password, dan OTP untuk membuktikan bahwa kamu benar-benar Budi).
- **Authorization (Otorisasi / `AuthZ`)**: Memverifikasi **APA YANG BOLEH KAMU LAKUKAN / AKSES** (Contoh: Setelah login sebagai Budi, apakah Budi boleh mengakses halaman Admin atau membaca kontak Google?).

### 1. OAuth 2.0 (Fokus pada Authorization / Izin Akses)
**OAuth 2.0** adalah standar industri untuk **memberikan izin akses terbatas (*delegated authorization*)** kepada aplikasi pihak ketiga atas sumber daya pengguna **tanpa pernah memberikan password pengguna ke aplikasi tersebut**.

#### Entitas Penting dalam OAuth 2.0:
- **Resource Owner**: Pengguna pemilik data (kamu).
- **Client**: Aplikasi yang meminta akses (misal aplikasi edit foto Canva yang ingin mengimpor foto dari Google Drive-mu).
- **Authorization Server**: Server yang memverifikasi persetujuan pengguna dan menerbitkan **Access Token** (misal `accounts.google.com`).
- **Resource Server**: Server API yang menyimpan data dan memeriksa validitas *Access Token* (misal API Google Drive).
- **Scopes**: Batasan izin spesifik yang diminta (misal `drive.readonly` — hanya boleh baca file, tidak boleh hapus file).
- **Access Token & Refresh Token**: *Access Token* berumur pendek (misal 15–60 menit) untuk mengakses API; *Refresh Token* digunakan untuk meminta *Access Token* baru tanpa menyuruh pengguna login ulang. *(Catatan Keamanan: Di aplikasi Web, simpan Refresh Token hanya di dalam **`HttpOnly Secure Cookie`**, jangan pernah di `LocalStorage` agar aman dari serangan XSS!).*

#### Alur Kerja OAuth 2.0 (Authorization Code Flow)

![oauth2](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-IV/oauth2-and-openid-connect/oauth2.png)

1. **Client** mengarahkan pengguna ke **Authorization Server** dengan menyertakan `client_id`, `redirect_uri`, dan `scope`.
2. **Authorization Server** memverifikasi identitas Client dan menampilkan layar persetujuan (*consent screen*) ke pengguna.
3. **Resource Owner (Pengguna)** login di halaman Authorization Server dan menekan tombol *"Izinkan (Allow)"*.
4. **Authorization Server** mengarahkan kembali ke Client dengan membawa kode satu kali pakai (**Authorization Code**), yang kemudian ditukar oleh backend Client menjadi **Access Token** (dan *Refresh Token*).
5. **Client** menggunakan **Access Token** tersebut untuk mengambil data dari **Resource Server**.

### 2. OpenID Connect / OIDC (Menambahkan Authentication di Atas OAuth 2.0)
Karena OAuth 2.0 murni hanya dirancang untuk *memberi izin akses API* (bukan untuk mengetahui identitas profil orang yang sedang login), maka lahirlah **OpenID Connect (OIDC)** sebagai lapisan tipis di atas OAuth 2.0.
- Ketika kita menggunakan tombol **"Login with Google / Apple / GitHub"**, protokol yang bekerja adalah **OIDC**!
- Selain memberikan *Access Token*, server OIDC (**Identity Provider / IdP**) juga mengembalikan **ID Token** berformat **JWT (*JSON Web Token*)** yang berisi informasi identitas pengguna (seperti `sub/user_id`, `name`, `email`, `picture`) serta menyediakan `UserInfo Endpoint`.

---

## 10. Single Sign-On (SSO) dan SAML

**Single Sign-On (SSO)** adalah mekanisme autentikasi di mana seorang pengguna cukup **login satu kali saja dengan satu akun utama**, lalu otomatis mendapatkan akses ke berbagai aplikasi berbeda tanpa perlu login berulang-ulang.
- *Contoh Sehari-hari*: Saat kamu login sekali ke akun Google (`accounts.google.com`), kamu otomatis sudah login saat membuka **Gmail, YouTube, Google Drive, Google Calendar, dan Google Meet**! Di perusahaan besar, karyawan cukup login sekali ke **Okta / Azure AD** untuk mengakses **Jira, Confluence, Slack, GitLab, dan Portal HR**.

### 3 Komponen Utama SSO
1. **Identity Provider (IdP)**: Sistem terpusat yang menyimpan akun, memverifikasi password/MFA pengguna, dan menerbitkan bukti identitas (Contoh: **Okta, Auth0, Keycloak, Google Workspace, Microsoft Entra ID**).
2. **Service Provider (SP)**: Aplikasi tujuan yang ingin diakses pengguna (misal aplikasi Slack atau Jira).
3. **Identity Broker**: Perantara yang menjembatani banyak *Service Provider* dengan berbagai *Identity Provider* yang berbeda protokol.

### Bagaimana Alur Kerja SSO?

![sso](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-IV/single-sign-on/sso.png)

1. Pengguna membuka aplikasi tujuan (**Service Provider**).
2. Aplikasi mendeteksi pengguna belum login, lalu me-*redirect* browser pengguna ke **Identity Provider (IdP)** pusat.
3. Pengguna memasukkan kredensial (atau jika sudah login sebelumnya di IdP, langkah ini otomatis dilewati!).
4. **IdP** mengirimkan bukti autentikasi bertanda tangan kriptografi (*SSO Assertion / Token*) kembali ke aplikasi.
5. Aplikasi memverifikasi tanda tangan tersebut dan mengizinkan pengguna masuk!

### Perbandingan SAML vs OAuth 2.0 / OpenID Connect (OIDC)

| Aspek | SAML 2.0 (*Security Assertion Markup Language*) | OAuth 2.0 & OpenID Connect (OIDC) |
| :--- | :--- | :--- |
| **Format Pesan** | Berbasis **XML** (berukuran besar & verbose). | Berbasis **JSON & JWT** (ringan & modern). |
| **Target Utama** | Aplikasi **Enterprise / Korporat (B2B)** berbasis browser web (seperti Salesforce, Workday, Jira Enterprise). | **Aplikasi Mobile (iOS/Android), Single Page Apps (React/Vue), dan API Modern (B2C & B2B)**. |
| **Kemudahan Implementasi** | Cukup rumit dikonfigurasi dan dikelola. | Sangat ramah developer (*developer-friendly*) dan mudah diintegrasikan via REST API. |

---

## 11. SSL, TLS, dan mTLS

### 1. SSL (Secure Sockets Layer) & TLS (Transport Layer Security)
- Dahulu data di web dikirim dalam bentuk teks telanjang (*plaintext* `HTTP`) sehingga siapa pun di Wi-Fi publik bisa menyadap password dan nomor kartu kreditmu.
- **SSL** diciptakan pada tahun 1995 untuk mengenkripsi komunikasi internet, namun kini **SSL sudah usang (*deprecated*) dan telah digantikan sepenuhnya oleh TLS (*Transport Layer Security*)** (seperti TLS 1.2 dan TLS 1.3). *(Meskipun begitu, di industri orang masih sering menyebut sertifikat TLS dengan istilah "Sertifikat SSL").*
- **3 Jaminan Utama TLS (`HTTPS`)**:
  1. **Encryption (Enkripsi)**: Menyandikan data agar tidak bisa dibaca oleh penyadap di tengah jalan (*Man-in-the-Middle*).
  2. **Authentication (Autentikasi Server)**: Memastikan bahwa server yang kamu hubungi benar-benar server asli milik bank/website tersebut melalui sertifikat digital.
  3. **Integrity (Integritas Data)**: Memastikan isi paket data tidak diubah atau dipalsukan sedikit pun selama perjalanan.

### 2. mTLS (Mutual TLS) — *Keamanan Zero-Trust Antar-Microservices*
- Pada **TLS biasa (Satu Arah)**: Hanya **Client (Browser)** yang memverifikasi sertifikat milik **Server**, sedangkan Server tidak meminta sertifikat TLS dari browser pengguna.
- Pada **mTLS (*Mutual TLS* — Dua Arah)**: **Baik Client maupun Server sama-sama wajib menunjukkan Sertifikat TLS dan memverifikasi kunci privat satu sama lain!**
- **Kapan mTLS Digunakan?**
  - Dalam arsitektur **Zero-Trust Microservices** (biasanya diotomasi oleh *Service Mesh* seperti Istio/Linkerd) agar *Payment Service* hanya mau menerima panggilan API dari *Order Service* yang memiliki sertifikat internal yang sah, serta untuk perangkat **IoT** dan integrasi **B2B Perbankan (Open Banking SNAP BI)**.
