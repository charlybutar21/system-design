# 🌐 Bab I: Fundamental Jaringan & Infrastruktur Sistem

> **🎯 Tujuan Belajar Bab Ini:**
> Sebelum merancang sistem berskala besar seperti WhatsApp, Netflix, atau Gojek, seorang *Software Engineer* wajib memahami bagaimana sebuah *request* dari perangkat pengguna (HP/Laptop) bisa sampai ke server kita dengan cepat, aman, dan tidak mati meskipun diakses jutaan orang bersamaan.

---

## Daftar Isi Bab I
1. [Pengantar: Apa itu System Design?](#1-apa-itu-system-design)
2. [IP (Internet Protocol)](#2-ip-internet-protocol)
3. [Model OSI (7 Layer Jaringan)](#3-model-osi-open-systems-interconnection)
4. [TCP dan UDP](#4-tcp-dan-udp)
5. [Domain Name System (DNS)](#5-domain-name-system-dns)
6. [Load Balancing (Penyeimbang Beban)](#6-load-balancing)
7. [Clustering (Pengelompokan Server)](#7-clustering)
8. [Caching (Penyimpanan Cepat Sementara)](#8-caching)
9. [Content Delivery Network (CDN)](#9-content-delivery-network-cdn)
10. [Proxy (Forward Proxy vs Reverse Proxy)](#10-proxy)
11. [Availability (Ketersediaan Sistem & Tabel "The Nines")](#11-availability-ketersediaan-sistem)
12. [Scalability (Skalabilitas Vertikal vs Horizontal)](#12-scalability-skalabilitas)
13. [Storage (Penyimpanan Data: RAID, Block, File, Object)](#13-storage-penyimpanan-data)

---

## 1. Apa itu System Design?

**System Design** (Perancangan Sistem) adalah proses mendefinisikan **arsitektur**, **komponen**, **antarmuka (API)**, dan **aliran data** untuk sebuah sistem agar dapat memenuhi kebutuhan bisnis maupun teknis tertentu.

> **💡 Analogi Pemula:**
> Bayangkan kamu diminta membangun gedung perkantoran:
> - **Coding / Algoritma** ibarat keahlian memasang batu bata, menyambung pipa air, atau memasang kabel listrik.
> - **System Design** ibarat **cetak biru arsitektur gedung**: Berapa lantai gedungnya? Di mana letak tangga darurat? Bagaimana agar lift tidak macet saat jam masuk kantor? Bagaimana jika listrik PLN mati (apakah ada genset cadangan)?
> Tanpa cetak biru yang matang, gedung setinggi 50 lantai akan roboh meskipun batu batanya dipasang dengan rapi.

### Mengapa System Design Sangat Penting?
1. **Keputusan Awal Sulit Diubah Belakangan**: Mengubah fondasi arsitektur saat aplikasi sudah dipakai jutaan pengguna jauh lebih mahal dan berisiko dibanding merancangnya dengan benar sejak awal.
2. **Menjembatani Kebutuhan Bisnis & Teknis**: System design menerjemahkan keinginan bisnis (misal: *"Aplikasi harus tetap jalan saat Flash Sale 12.12"*) menjadi solusi teknis (*Auto-scaling, Load Balancer, Caching, Message Queue*).
3. **Kunci Utama Naik Level Karier**: Di hampir semua perusahaan teknologi, kemampuan System Design adalah pembeda utama antara *Junior/Mid Engineer* dengan *Senior/Principal Engineer*.

---

## 2. IP (Internet Protocol)

**Alamat IP (*IP Address*)** adalah alamat unik yang mengidentifikasi sebuah perangkat di internet atau jaringan lokal. **IP (*Internet Protocol*)** adalah seperangkat aturan yang mengatur format data yang dikirim melalui internet.

> **💡 Analogi Pemula:**
> Alamat IP ibarat **alamat rumah lengkap beserta kode pos**. Jika kamu ingin mengirim paket ke temanmu, kurir butuh alamat tujuan (IP penerima) dan alamat pengirim (IP kamu) agar paket tidak nyasar.

### Versi Alamat IP

| Fitur | IPv4 | IPv6 |
| :--- | :--- | :--- |
| **Tahun Diperkenalkan** | 1981 (Standar awal internet) | 1998 (Mulai diadopsi luas pertengahan 2000-an) |
| **Ukuran Bit** | **32-bit** (4 angka desimal dipisah titik) | **128-bit** (8 blok heksadesimal dipisah titik dua) |
| **Jumlah Alamat Unik** | $\sim 4,3 \text{ Miliar}$ alamat ($2^{32}$) | $\sim 340 \text{ Undesiliun}$ ($3,4 \times 10^{38}$ atau $2^{128}$) |
| **Contoh Format** | `102.22.192.181` | `2001:0db8:85a3:0000:0000:8a2e:0370:7334` |
| **Status Saat Ini** | Sudah hampir habis secara global (dibantu teknologi NAT) | Masa depan internet karena jumlah perangkat IoT & HP melonjak |

### Jenis-Jenis Alamat IP

1. **Public IP (IP Publik)**:
   - Alamat IP utama yang diberikan oleh penyedia internet (ISP seperti IndiHome, Biznet, Telkomsel) ke *router* kamu.
   - Alamat inilah yang dikenali oleh dunia luar (internet). Seluruh perangkat di satu Wi-Fi rumah biasanya berbagi 1 IP Publik yang sama saat mengakses internet.
2. **Private IP (IP Privat)**:
   - Alamat IP internal yang diberikan oleh *router* Wi-Fi rumah/kantor ke masing-masing perangkat di dalamnya (laptop, HP, Smart TV, printer).
   - Contoh rentang IP Privat: `192.168.1.x`, `10.0.0.x`, `172.16.x.x`. IP ini tidak bisa diakses langsung dari internet luar.
3. **Static IP (IP Statis)**:
   - Alamat IP yang **tidak pernah berubah** dan dikonfigurasi secara manual/khusus.
   - Lebih mahal, namun wajib digunakan untuk **server produksi, DNS server, VPN kantor, atau layanan pembayaran (whitelist IP)** agar alamat server tidak berubah-ubah.
4. **Dynamic IP (IP Dinamis)**:
   - Alamat IP yang **berubah secara berkala** dan diberikan otomatis oleh server **DHCP (*Dynamic Host Configuration Protocol*)**.
   - Lebih hemat biaya bagi ISP karena alamat IP yang sedang tidak terpakai (misal modem dimatikan) bisa dipinjamkan ke pelanggan lain. Umum dipakai untuk internet rumahan/konsumen.

---

## 3. Model OSI (Open Systems Interconnection)

**Model OSI** adalah kerangka konseptual yang membagi proses komunikasi jaringan menjadi **7 lapisan (layer)** abstrak, dari lapisan fisik (kabel) hingga aplikasi yang dilihat pengguna.

![osi-model](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/osi-model/osi-model.png)

> **💡 Analogi Pemula & Cara Cepat Menghafal:**
> Bayangkan seorang Direktur di lantai 7 gedung A ingin mengirim surat rahasia ke Direktur di lantai 7 gedung B:
> - **L7 (Application)**: Direktur menulis isi surat.
> - **L6 (Presentation)**: Sekretaris menerjemahkan surat ke bahasa Inggris & memasukkannya ke amplop bersegel (enkripsi) serta melipatnya (kompresi).
> - **L5 (Session)**: Bagian resepsionis menelepon gedung B untuk memastikan kantor mereka sedang buka.
> - **L4 (Transport)**: Bagian ekspedisi memotong dokumen tebal menjadi beberapa bundel bernomor urut (Segmen) agar kalau ada bundel yang hilang bisa diminta kirim ulang.
> - **L3 (Network)**: Kurir kota menempelkan alamat jalan & kode pos gedung B (IP Address) dan memilih rute jalan raya tercepat (Routing).
> - **L2 (Data Link)**: Kurir gedung memindahkan paket antar-pos di dalam satu komplek yang sama (MAC Address).
> - **L1 (Physical)**: Jalan aspal, ban motor kurir, atau kabel serat optik yang membawa sinyal listrik/cahaya `1` dan `0`.
>
> **Jembatan Keledai (Dari Layer 1 ke 7):** **P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way (*Physical, Data Link, Network, Transport, Session, Presentation, Application*).

### Penjelasan 7 Layer OSI (Dari Atas ke Bawah)

| Layer | Nama Lapisan | Unit Data | Protokol / Contoh | Tugas Utama |
| :---: | :--- | :--- | :--- | :--- |
| **Layer 7** | **Application** | Data | `HTTP`, `HTTPS`, `SMTP`, `DNS`, `WebSocket` | Berinteraksi langsung dengan aplikasi pengguna (browser, aplikasi email). |
| **Layer 6** | **Presentation** | Data | `TLS/SSL`, `JPEG`, `JSON`, `Gzip` | **Translasi** format data, **Enkripsi/Dekripsi**, dan **Kompresi**. |
| **Layer 5** | **Session** | Data | `NetBIOS`, `RPC`, `PPTP` | Membuka, menjaga, menyinkronkan (*checkpoint*), dan menutup sesi komunikasi antar dua perangkat. |
| **Layer 4** | **Transport** | Segment (TCP) / Datagram (UDP) | `TCP`, `UDP` | Komunikasi *end-to-end* antar proses, memecah data menjadi segmen, mengatur *port* (`80`, `443`), kontrol aliran & perbaikan error. |
| **Layer 3** | **Network** | Packet | `IP (IPv4/IPv6)`, `ICMP`, Router | Mentransfer paket antar jaringan yang berbeda dan menentukan jalur fisik terbaik (**Routing**). |
| **Layer 2** | **Data Link** | Frame | `Ethernet`, `Wi-Fi`, Switch, MAC Address | Mentransfer data antar dua perangkat di dalam **satu jaringan yang sama**. |
| **Layer 1** | **Physical** | Bit (`1` & `0`) | Kabel LAN (UTP), Fiber Optik, Gelombang Radio | Perangkat keras fisik yang mengubah data menjadi aliran bit sinyal listrik/cahaya. |

> **🎯 Tips Mentor untuk Interview:**
> Dalam *System Design Interview*, kamu hampir selalu akan ditanya perbedaan **Layer 4 (Transport)** dan **Layer 7 (Application)**—terutama saat membahas **Load Balancer** atau **Firewall**. Ingat:
> - **Layer 4** hanya tahu **IP Address + Port** (`TCP/UDP`). Dia sangat cepat karena tidak membuka isi paket.
> - **Layer 7** tahu **isi HTTP Request** (URL path `/api/users`, Header, Cookie, JSON body). Dia lebih pintar tapi butuh komputasi sedikit lebih besar.

---

## 4. TCP dan UDP

Keduanya berada di **Layer 4 (Transport Layer)** dan bertugas mengirimkan data melalui jaringan, tetapi memiliki filosofi yang bertolak belakang: **Keandalan (TCP)** vs **Kecepatan (UDP)**.

### TCP (Transmission Control Protocol)

TCP adalah protokol **berorientasi koneksi (*connection-oriented*)**. Sebelum mengirim data, pengirim dan penerima wajib melakukan "jabat tangan" (**TCP 3-Way Handshake**: `SYN` → `SYN-ACK` → `ACK`).

![tcp](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/tcp-and-udp/tcp.png)

- **Karakteristik**:
  - Menjamin semua paket data sampai secara **utuh** dan **sesuai urutan** (*ordered delivery*).
  - Jika ada paket yang hilang di jalan, TCP otomatis meminta pengiriman ulang (*retransmission*).
  - Memiliki kontrol kemacetan jaringan (*congestion control*).
- **Konsekuensi (*Trade-off*)**: Memiliki *overhead* (beban tambahan) lebih besar dan latensi lebih tinggi dibanding UDP.

### UDP (User Datagram Protocol)

UDP adalah protokol **tanpa koneksi (*connectionless*)** yang bekerja dengan prinsip *"kirim saja terus tanpa peduli sampai atau tidak"* (*fire and forget*).

![udp](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/tcp-and-udp/udp.png)

- **Karakteristik**:
  - Tidak ada *handshake* pembuka koneksi.
  - Tidak ada pengiriman ulang jika paket hilang.
  - Sangat ringan dan memiliki **latensi paling rendah**.
- **Kapan memilih UDP dibanding TCP?** Ketika **data yang datang terlambat lebih buruk daripada data yang hilang sedikit**. Contoh: Saat *video call* WhatsApp/Zoom atau *game online* (Mobile Legends/Valorant), kalau ada 1 frame gambar macet 2 detik lalu, kamu tidak butuh frame basi tersebut dikirim ulang—kamu butuh gambar detik ini juga!

### Tabel Perbandingan TCP vs UDP

| Fitur | TCP (Transmission Control Protocol) | UDP (User Datagram Protocol) |
| :--- | :--- | :--- |
| **Koneksi** | Wajib membuat koneksi dulu (*Connection-oriented*) | Tanpa koneksi (*Connectionless*) |
| **Jaminan Sampai** | **Ya** (Setiap paket dikonfirmasi dengan `ACK`) | **Tidak** (Paket bisa hilang tanpa pemberitahuan) |
| **Urutan Data** | Dijamin berurutan sesuai pengiriman | Tidak dijamin berurutan |
| **Pengiriman Ulang** | Paket hilang otomatis dikirim ulang (*Retransmission*) | Tidak ada pengiriman ulang |
| **Kecepatan / Latensi** | Lebih lambat (karena *handshake* & pengecekan error) | **Sangat cepat** (overhead minimal) |
| **Broadcasting** | Tidak mendukung *broadcast* (hanya *one-to-one*) | Mendukung *broadcast* & *multicast* (*one-to-many*) |
| **Contoh Penggunaan** | Web (`HTTP/HTTPS`), Email (`SMTP`), Transfer File (`FTP`), Database Query | *Live Video Streaming*, *Voice/Video Call* (`VoIP`), *Online Gaming*, `DNS` lookup |

---

## 5. Domain Name System (DNS)

Manusia mudah mengingat nama seperti `tokopedia.com` atau `google.com`, sedangkan komputer hanya mengerti alamat IP seperti `142.250.192.232`. **DNS** adalah sistem penamaan hierarkis dan terdesentralisasi yang menerjemahkan nama domain manusia menjadi alamat IP komputer.

> **💡 Analogi Pemula:**
> DNS adalah **Buku Kontak di HP kamu**. Saat kamu ingin menelepon "Ibu", kamu cukup mengetik nama "Ibu" di layar. HP kamu yang akan mencari nomor telepon `0812-xxxx-xxxx` di balik nama tersebut lalu melakukan panggilan.

### Bagaimana Cara Kerja DNS? (8 Langkah Resolusi DNS)

![how-dns-works](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/domain-name-system/how-dns-works.png)

Ketika kamu mengetik `example.com` di browser (dan belum ada di *cache*):
1. **Browser → DNS Resolver**: Browser mengirim pertanyaan *"Berapa IP dari `example.com`?"* ke **DNS Recursive Resolver** (biasanya milik ISP atau layanan seperti Cloudflare `1.1.1.1` / Google `8.8.8.8`).
2. **Resolver → Root Server**: Karena Resolver belum tahu, ia bertanya ke **DNS Root Nameserver** (`.`): *"Siapa yang mengurus domain `.com`?"*
3. **Root Server → Resolver**: Root Server menjawab: *"Saya tidak tahu IP `example.com`, tapi ini alamat **TLD Nameserver** untuk `.com`."*
4. **Resolver → TLD Server**: Resolver bertanya ke **TLD Server `.com`**: *"Siapa yang mengurus `example.com`?"*
5. **TLD Server → Resolver**: TLD Server menjawab: *"Ini alamat **Authoritative Nameserver** milik domain `example.com`."*
6. **Resolver → Authoritative Server**: Resolver bertanya langsung ke **Authoritative Nameserver**: *"Berapa IP untuk `example.com`?"*
7. **Authoritative Server → Resolver**: Authoritative Nameserver melihat catatan resminya (*A Record*) dan menjawab: *"IP-nya adalah `93.184.216.34`."*
8. **Resolver → Browser**: Resolver menyimpan jawaban tersebut di memori (*cache*) untuk sementara waktu, lalu memberikan IP `93.184.216.34` ke browser. Browser pun langsung menghubungi server `93.184.216.34` untuk menampilkan halaman web!

### 4 Komponen Server Utama DNS
1. **DNS Recursive Resolver**: Perantara pertama yang menerima permintaan dari perangkat pengguna dan bekerja keras mencari jawaban ke server-server DNS lainnya (atau langsung menjawab jika sudah ada di *cache*).
2. **Root Nameserver**: Puncak hierarki DNS yang mengarahkan pencarian ke TLD yang sesuai. Ada 13 jenis Root Server di dunia (dilambangkan huruf A sampai M) yang diawasi oleh **ICANN**, namun secara fisik memiliki ratusan salinan server di seluruh dunia menggunakan teknologi **Anycast Routing**.
3. **TLD (Top-Level Domain) Nameserver**: Mengelola semua domain dengan akhiran yang sama, dikelola oleh **IANA** (cabang ICANN). Dibagi menjadi dua:
   - **gTLD (*Generic TLD*)**: `.com`, `.org`, `.net`, `.edu`, `.gov`.
   - **ccTLD (*Country Code TLD*)**: `.id` (Indonesia), `.sg` (Singapura), `.jp` (Jepang), `.uk` (Inggris).
4. **Authoritative Nameserver**: Server terakhir yang memegang data resmi (*zone file*) dari domain tersebut. Jika domain tidak ditemukan, ia mengembalikan pesan error `NXDOMAIN` (*Non-Existent Domain*).

### 3 Jenis Query DNS
- **Recursive Query**: Klien meminta DNS Resolver memberikan jawaban tuntas (berupa IP yang dicari atau pesan error jika tidak ada). Resolver tidak boleh menyuruh klien bertanya sendiri ke server lain.
- **Iterative Query**: Resolver bertanya ke Root/TLD server, dan jika server tersebut tidak punya jawabannya langsung, ia memberikan rujukan (*referral*): *"Coba tanya ke server X."*
- **Non-Recursive Query**: Terjadi ketika DNS Resolver **sudah tahu jawabannya** di dalam *cache* lokalnya, sehingga langsung membalas ke klien tanpa perlu bertanya ke server lain.

### Jenis-Jenis DNS Record yang Wajib Diketahui

| Jenis Record | Kepanjangan | Fungsi & Contoh Penggunaan |
| :--- | :--- | :--- |
| **`A`** | *Address Record* | Menyimpan alamat **IPv4** dari sebuah domain (misal: `example.com` → `93.184.216.34`). |
| **`AAAA`** | *IPv6 Address Record* | Menyimpan alamat **IPv6** dari sebuah domain. |
| **`CNAME`** | *Canonical Name* | Mengarahkan satu domain/subdomain (alias) ke **nama domain lain**, bukan ke IP (misal: `www.example.com` → `example.com`). |
| **`MX`** | *Mail Exchanger* | Mengarahkan email yang dikirim ke domain tersebut menuju server email tujuan. |
| **`TXT`** | *Text Record* | Menyimpan catatan teks untuk verifikasi kepemilikan domain dan keamanan email (`SPF`, `DKIM`, `DMARC`). |
| **`NS`** | *Name Server* | Menentukan server DNS mana yang berwenang (*authoritative*) atas domain tersebut. |
| **`SOA`** | *Start of Authority* | Menyimpan informasi administratif domain (email admin, nomor seri pembaruan zona). |
| **`SRV`** | *Service Location* | Menentukan *host* dan *port* spesifik untuk layanan tertentu (misal VoIP/XMPP). |
| **`PTR`** | *Pointer Record* | Kebalikan dari `A Record`: digunakan untuk **Reverse DNS Lookup** (mencari nama domain dari alamat IP, sering dipakai server email untuk menangkal spam). |
| **`CERT`** | *Certificate Record* | Menyimpan sertifikat kunci publik (*public key certificates*). |

### Konsep Penting Lainnya di DNS
- **Subdomain**: Bagian tambahan di depan domain utama untuk memisahkan fungsi layanan, misalnya `blog.example.com`, `api.example.com`, atau `karir.example.com`.
- **DNS Zone**: Bagian administratif dari ruang nama domain yang didelegasikan ke satu entitas/organisasi untuk dikelola sendiri.
- **DNS Caching & TTL (*Time-To-Live*)**: Agar tidak perlu melakukan 8 langkah pencarian setiap kali membuka halaman web, hasil pencarian DNS disimpan sementara di **OS komputer, Browser, dan ISP Resolver** selama durasi **TTL** (dalam satuan detik). Jika TTL habis (mencapai 0), barulah pencarian DNS dilakukan ulang.
- **Contoh Layanan Managed DNS**: Amazon Route53, Cloudflare DNS, Google Cloud DNS, Azure DNS, NS1.

---

## 6. Load Balancing

Ketika aplikasi kita tumbuh dari 100 pengguna menjadi 10 juta pengguna, satu server saja pasti akan kewalahan (*overload*) dan mati. Kita perlu menambah banyak server di belakang layar, dan di sinilah **Load Balancer (Penyeimbang Beban)** berperan.

**Load Balancer** adalah komponen yang berada di depan sekumpulan server untuk mendistribusikan trafik jaringan yang masuk secara merata ke server-server yang sehat (*online*).

![load-balancing](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/load-balancing/load-balancer.png)

> **💡 Analogi Pemula:**
> Bayangkan antrean kasir di supermarket besar saat akhir pekan. Jika hanya ada 1 kasir, antrean akan mengular sampai ke pintu keluar. Maka supermarket membuka **10 meja kasir**, dan menaruh **1 petugas pengatur antrean (Load Balancer)** di depan yang mengarahkan pelanggan: *"Silakan ke Kasir 3 karena sedang kosong, Kasir 2 sedang istirahat (offline) jadi jangan ke sana."*

Untuk ketahanan dan skalabilitas maksimal, kita bisa memasang Load Balancer di **setiap lapisan sistem** (antara User → Web Server, Web Server → App Server, dan App Server → Database):

![load-balancing-layers](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/load-balancing/load-balancer-layers.png)

### Cara Distribusi Beban Kerja (*Workload Distribution*)
- **Host-based**: Mengarahkan *request* berdasarkan nama domain/host yang diminta (misal `api.toko.com` ke grup server A, `web.toko.com` ke grup server B).
- **Path-based**: Mengarahkan *request* berdasarkan jalur URL (misal `/pembayaran` ke server khusus transaksi, `/katalog` ke server produk).
- **Content-based**: Memeriksa isi pesan *request* (seperti parameter query atau HTTP header) sebelum menentukan server tujuan.

### Lapisan Operasi Load Balancer: Layer 4 vs Layer 7

| Aspek | Network Load Balancer (Layer 4) | Application Load Balancer (Layer 7) |
| :--- | :--- | :--- |
| **Basis Routing** | Hanya melihat **IP Address** dan **Port TCP/UDP**. | Membaca keseluruhan isi HTTP/HTTPS (**Header, URL Path, Cookie, Body**). |
| **Kecepatan** | Sangat cepat (tidak perlu mendekripsi/membaca paket aplikasi). | Sedikit lebih lambat dari L4, namun jauh lebih cerdas. |
| **Content Routing** | Tidak bisa merutekan berdasarkan URL `/video` vs `/gambar`. | Bisa melakukan routing pintar berdasarkan URL, header, atau tipe pengguna. |
| **Contoh** | AWS NLB, HAProxy (mode TCP). | AWS ALB, Nginx, Envoy, Traefik. |

### Jenis Load Balancer
1. **Software Load Balancer**: Berupa perangkat lunak yang diinstal di server atau layanan Cloud. Sangat fleksibel, hemat biaya, dan mudah dikonfigurasi (Contoh: **Nginx, HAProxy, Envoy, AWS ELB, GCP Load Balancing**).
2. **Hardware Load Balancer**: Perangkat fisik khusus (*dedicated appliance* seperti F5 BIG-IP) yang dipasang di ruang data center. Mampu menangani trafik raksasa dengan sangat cepat, tetapi harganya sangat mahal dan kurang fleksibel.
3. **DNS Load Balancing**: Mengonfigurasi satu domain agar mengembalikan beberapa alamat IP server secara bergantian. **Kelemahan utamanya**: DNS tidak tahu apakah sebuah server sedang mati (*down*), dan karena adanya *DNS caching* di perangkat pengguna, trafik bisa tetap dikirim ke server yang sudah mati!

### 6 Algoritma Routing pada Load Balancer
1. **Round-Robin**: Membagikan *request* secara bergiliran satu per satu (Server 1 → Server 2 → Server 3 → kembali ke Server 1). Cocok jika semua server punya spesifikasi identik.
2. **Weighted Round-Robin**: Sama seperti Round-Robin, tetapi setiap server diberi **bobot (*weight*)** sesuai kapasitas CPU/RAM-nya. Server berspesifikasi 2x lebih besar akan menerima 2x lebih banyak *request*.
3. **Least Connections**: Mengirim *request* baru ke server yang saat ini memiliki **jumlah koneksi aktif paling sedikit**. Sangat cocok jika durasi tiap *request* berbeda-beda (misal koneksi panjang).
4. **Least Response Time**: Mengirim *request* ke server dengan kombinasi **waktu respons tercepat** dan **koneksi aktif paling sedikit**.
5. **Least Bandwidth**: Mengirim *request* ke server yang sedang menggunakan trafik bandwidth (Mbps) paling kecil saat itu.
6. **Hashing (IP Hash / URL Hash)**: Menghitung nilai *hash* dari IP klien atau URL *request* untuk menentukan server tujuan. Memastikan pengguna yang sama cenderung diarahkan ke server yang sama.

### Menghindari Single Point of Failure: Redundant Load Balancers
Bagaimana jika Load Balancer itu sendiri yang mati? Jika hanya ada 1 Load Balancer, seluruh sistem tetap akan lumpuh (**Single Point of Failure / SPOF**)!
Solusinya adalah memasang minimal **2 Load Balancer dalam mode Cluster (Active-Passive atau Active-Active)** yang saling memantau detak jantung (*heartbeat*). Jika Load Balancer utama (*Active*) mati, Load Balancer cadangan (*Passive*) langsung mengambil alih IP mengambang (*Floating IP*).

![redundant-load-balancing](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/load-balancing/redundant-load-balancer.png)

### Fitur-Fitur Penting Load Balancer Modern
- **Health Checks**: Mengecek kesehatan server secara berkala (misal memanggil endpoint `/health` tiap 5 detik). Jika server gagal merespons, Load Balancer otomatis berhenti mengirim trafik ke server tersebut.
- **Autoscaling**: Menambah server baru secara otomatis saat trafik melonjak, dan mematikan server saat trafik sepi.
- **SSL/TLS Termination**: Mengambil alih tugas dekripsi enkripsi HTTPS di Load Balancer sehingga server backend tidak terbebani komputasi kriptografi.
- **Sticky Sessions (Session Affinity)**: Mengarahkan pengguna yang sama ke server yang sama terus-menerus. *(Catatan Arsitek: Sebaiknya hindari Sticky Sessions! Rancang server secara **Stateless** dan simpan session di **Redis** agar jika satu server mati, user tidak ter-logout).*

---

## 7. Clustering

**Cluster komputer** adalah sekumpulan dua atau lebih komputer (*nodes*) yang terhubung dalam jaringan dan bekerja secara paralel untuk mencapai satu tujuan bersama, sehingga dari luar terlihat seolah-olah hanya sebagai **satu sistem tunggal**.

![cluster](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/clustering/cluster.png)

Biasanya terdapat minimal satu node yang ditunjuk sebagai **Leader Node** yang bertugas menerima pekerjaan, membagikannya ke node pekerja (*worker nodes*), dan menggabungkan hasilnya.

### Kategori Cluster
1. **High Availability (HA) / Fail-over Cluster**: Menjamin layanan tetap hidup meskipun ada mesin yang rusak.
2. **Load Balancing Cluster**: Membagi beban kerja ke banyak node agar performa dan *throughput* meningkat.
3. **High-Performance Computing (HPC) Cluster**: Menggabungkan kekuatan CPU dan RAM ratusan mesin untuk komputasi ilmiah/AI yang sangat berat.

### Konfigurasi High Availability Cluster

#### 1. Active-Active Cluster
Semua node (minimal 2 node) sama-sama aktif melayani trafik secara bersamaan di bawah koordinasi Load Balancer.
- **Kelebihan**: Kapasitas dan kecepatan meningkat karena semua mesin bekerja; tidak ada mesin yang menganggur.

![active-active](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/clustering/active-active.png)

#### 2. Active-Passive Cluster
Node utama (**Active**) melayani seluruh trafik, sedangkan node cadangan (**Passive**) bersiaga (*standby*) dan baru akan aktif menggantikan jika node utama mati (*failover*).
- **Kelebihan**: Konsistensi data lebih mudah dijaga karena hanya ada satu node aktif yang memodifikasi data pada satu waktu.

![active-passive](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/clustering/active-passive.png)

### Apa Bedanya Load Balancing vs Clustering?
- **Load Balancing**: Server-server di belakang Load Balancer **tidak saling mengenal** satu sama lain. Mereka hanya merespons perintah yang dikirim oleh Load Balancer.
- **Clustering**: Server-server di dalam sebuah cluster **saling menyadari keberadaan satu sama lain** (*heartbeats*, *gossip protocol*, *leader election*), berbagi status, dan bekerja sama sebagai satu kesatuan (Contoh: **Kubernetes Cluster, Redis Cluster, Apache Cassandra, MongoDB Replica Set**).

---

## 8. Caching

> *"Ada dua hal paling sulit dalam Ilmu Komputer: Cache Invalidation dan memberi nama variabel."* — Phil Karlton

**Cache** adalah lapisan penyimpanan data berkecepatan sangat tinggi (biasanya di dalam memori **RAM**) yang menyimpan sebagian data sementara waktu agar permintaan data berikutnya dapat dilayani jauh lebih cepat tanpa harus membaca ke database utama (disk) yang lambat.

![caching](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/caching/caching.png)

Caching memanfaatkan prinsip **Locality of Reference**: *"Data yang baru saja diminta memiliki kemungkinan besar untuk diminta kembali dalam waktu dekat."*

> **💡 Analogi Pemula:**
> - **Database (SSD/Harddisk)** ibarat **Perpustakaan Kota** yang menyimpan jutaan buku lengkap, tapi butuh waktu 30 menit perjalanan untuk mengambil satu buku.
> - **Cache (RAM/Redis)** ibarat **Meja Belajar di kamarmu**. Buku-buku yang paling sering kamu baca minggu ini kamu taruh di atas meja belajar, sehingga kamu bisa mengambilnya hanya dalam **1 detik**!

### Hierarki Memori & Cache Hit vs Cache Miss
Di dalam perangkat keras komputer maupun sistem terdistribusi, cache disusun bertingkat mulai dari **L1, L2, L3** (di dalam prosesor) hingga **RAM / In-Memory Cache (Redis/Memcached)**.

- **Cache Hit**: Data yang dicari **ditemukan** di dalam cache!
  - **Hot Cache**: Data dibaca dengan kecepatan paling maksimal (dari memori terdepan/L1 atau cache yang sudah terisi penuh oleh data populer).
  - **Warm Cache**: Data ditemukan di level memori menengah (L2/L3), lebih cepat dari database tapi belum secepat *hot cache*.
  - **Cold Cache**: Kondisi ketika cache baru saja dinyalakan/kosong atau berada di level paling lambat sehingga hampir semua request awal harus mengambil dari database terlebih dahulu.
- **Cache Miss**: Data yang dicari **tidak ditemukan** di dalam cache, sehingga sistem terpaksa mengambilnya dari database utama lalu menyimpannya ke cache untuk permintaan berikutnya.

### Strategi Penulisan & Cache Invalidation
Jika data di database berubah (misal harga barang diupdate), data di dalam cache menjadi basi (*stale*). Berikut adalah **4 pola utama arsitektur caching**:

#### 1. Cache-Aside (Lazy Loading) — *Paling Populer di Industri*
Aplikasi mengecek Cache terlebih dahulu. Jika *Cache Miss*, aplikasi membaca dari Database, lalu aplikasi menulis hasilnya ke Cache untuk request berikutnya. Saat ada update data, aplikasi mengupdate Database lalu **menghapus (invalidate)** entri di Cache.

#### 2. Write-Through Cache
Setiap kali ada data baru/perubahan, data ditulis ke **Cache dan Database secara bersamaan (sinkron)**.

![write-through-cache](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/caching/write-through-cache.png)

- **Kelebihan**: Data di cache dan database selalu 100% konsisten; pembacaan selalu cepat.
- **Kekurangan**: Operasi tulis (*write*) menjadi lebih lambat karena harus menunggu penulisan di dua tempat selesai.

#### 3. Write-Around Cache
Operasi tulis (*write*) langsung masuk ke **Database saja**, melewati (*bypass*) Cache. Cache hanya diisi ketika ada pembacaan (*read*) yang mengalami *Cache Miss*.

![write-around-cache](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/caching/write-around-cache.png)

- **Kelebihan**: Cache tidak dipenuhi oleh data yang baru ditulis tapi jarang dibaca.
- **Kekurangan**: Pembacaan pertama setelah data ditulis pasti mengalami *Cache Miss* dan terasa lambat.

#### 4. Write-Back (Write-Behind) Cache
Aplikasi hanya menulis data ke **Cache**, dan langsung dianggap selesai (*sukses*)! Selanjutnya, Cache yang akan menyinkronkan data tersebut ke Database secara **asinkron** di belakang layar (misal secara berkala/*batch*).

![write-back-cache](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/caching/write-back-cache.png)

- **Kelebihan**: Kecepatan tulis (*write throughput*) luar biasa cepat! Cocok untuk *workload* tulis masif (seperti *counter* jumlah *view* atau *like*).
- **Kekurangan**: **Berisiko kehilangan data** jika server Cache tiba-tiba mati sebelum sempat menyalin data ke Database.

### 6 Kebijakan Penghapusan Cache (*Cache Eviction Policies*)
Karena ukuran RAM terbatas dan mahal, ketika Cache penuh kita harus memilih data mana yang harus dibuang:
1. **LRU (*Least Recently Used*)** ⭐ *(Paling sering dipakai)*: Membuang data yang **paling lama tidak disentuh/diakses**.
2. **LFU (*Least Frequently Used*)**: Menghitung berapa kali tiap data diakses, lalu membuang data yang **frekuensi aksesnya paling jarang**.
3. **FIFO (*First In First Out*)**: Membuang data yang paling pertama masuk ke cache tanpa peduli seberapa sering data itu dipakai.
4. **LIFO (*Last In First Out*)**: Membuang data yang paling terakhir masuk ke cache.
5. **MRU (*Most Recently Used*)**: Kebalikan LRU, membuang data yang baru saja diakses (berguna untuk pola akses di mana data yang sudah dilihat tidak akan dilihat lagi dalam waktu dekat).
6. **RR (*Random Replacement*)**: Memilih dan membuang data secara acak saat ruang penuh.

### Distributed Cache vs Global Cache
- **Distributed Cache**: Menggabungkan RAM dari banyak server komputer menjadi satu klaster cache raksasa (menggunakan *Consistent Hashing*), sehingga kapasitas cache bisa diperbesar tanpa batas.

![distributed-cache](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/caching/distributed-cache.png)

- **Global Cache**: Satu ruang cache terpusat yang dipakai bersama oleh seluruh node aplikasi. Jika data tidak ada di Global Cache, cache itu sendiri yang bertanggung jawab mengambil data dari database.

![global-cache](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/caching/global-cache.png)

### Kapan TIDAK Boleh Menggunakan Caching?
- Ketika waktu akses ke cache sama lamanya dengan akses ke penyimpanan utama.
- Ketika permintaan data sangat acak (jarang ada data yang diminta berulang kali).
- Ketika data berubah setiap detik dan menuntut konsistensi mutlak tanpa toleransi jeda.

> **Contoh Teknologi Caching**: **Redis**, **Memcached**, Amazon ElastiCache, Aerospike.

---

## 9. Content Delivery Network (CDN)

**CDN** adalah jaringan server yang tersebar secara geografis di berbagai belahan dunia (*Edge Servers / Points of Presence*) yang bekerja sama untuk mengirimkan konten internet (terutama aset statis seperti HTML, CSS, JavaScript, Gambar, dan Video) dari lokasi terdekat dengan pengguna.

![cdn-map](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/content-delivery-network/cdn-map.png)

> **💡 Analogi Pemula:**
> Bayangkan pabrik pusat sepatu ada di **Amerika Serikat (Origin Server)**. Jika setiap pembeli di Jakarta harus menunggu kiriman langsung dari gudang Amerika, paket baru sampai 2 minggu kemudian (latensi tinggi). Maka perusahaan membuka **Gudang Cabang (Edge Server CDN) di Jakarta, Singapura, dan Surabaya** yang menyimpan stok sepatu populer. Pembeli di Jakarta cukup dikirim dari gudang Jakarta dalam waktu 1 jam!

### Bagaimana Cara Kerja CDN?

![cdn](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/content-delivery-network/cdn.png)

1. Server utama kita disebut **Origin Server** (menyimpan versi asli seluruh konten).
2. CDN menyimpan salinan (*cache*) aset statis di banyak lokasi geografis yang disebut **Edge Locations**.
3. Saat pengguna di Indonesia membuka website yang server aslinya di Amerika Serikat, DNS mengarahkan permintaan aset statis ke **Edge Server terdekat** (misal di Jakarta atau Singapura).
4. Hasilnya: Waktu muat halaman jauh lebih cepat, beban *Origin Server* turun drastis, dan biaya bandwidth lintas benua lebih hemat.

### 2 Jenis CDN: Push CDN vs Pull CDN

| Karakteristik | Push CDN | Pull CDN |
| :--- | :--- | :--- |
| **Cara Kerja** | Developer/Server kita **mengunggah (*push*)** konten baru secara aktif ke CDN setiap kali ada perubahan. | CDN **mengambil sendiri (*pull*)** aset dari *Origin Server* hanya ketika ada pengguna pertama yang memintanya (*Cache Miss*). |
| **Biaya & Trafik** | Hemat trafik jaringan, tetapi memakan ruang penyimpanan lebih besar. | Sangat mudah dikelola (*low maintenance*), hanya menyimpan konten yang benar-benar diakses pengguna. |
| **Cocok Untuk** | Website dengan trafik kecil atau konten yang jarang berubah. | **Website trafik tinggi** dengan jutaan aset (seperti e-commerce atau media sosial). |

### Kekurangan CDN
- **Biaya Tambahan**: Bisa cukup mahal untuk layanan dengan trafik video/unduhan masif.
- **Pemblokiran Wilayah**: Beberapa negara/organisasi kadang memblokir IP CDN tertentu.
- **Lokasi**: Jika audiens kita berada di wilayah yang tidak memiliki *Edge Server* CDN tersebut, rute data justru bisa memutar lebih jauh.

> **Contoh Layanan CDN**: **Cloudflare CDN**, **Amazon CloudFront**, **Google Cloud CDN**, **Fastly**, **Akamai**.

---

## 10. Proxy

**Proxy Server** adalah perangkat keras/lunak perantara yang duduk di antara **Client (Pengguna)** dan **Server Backend**. Proxy menerima *request*, lalu dapat memfilter, mencatat log, mengenkripsi, mengompresi, atau meneruskannya ke server tujuan.

Ada dua jenis Proxy yang **sangat sering ditanyakan saat interview** karena arah kerjanya berkebalikan:

### 1. Forward Proxy (Mewakili Sisi Client)
**Forward Proxy** duduk **di depan sekumpulan Client** (misal komputer-komputer karyawan di kantor). Ketika klien mengakses internet, Forward Proxy mencegat permintaan tersebut dan menghubungi server di internet **atas nama klien**.

![forward-proxy](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/proxy/forward-proxy.png)

- **Intinya**: Server di internet **tidak pernah tahu siapa klien aslinya**; server hanya melihat IP milik Forward Proxy.
- **Kegunaan Utama**:
  - Memblokir akses karyawan/siswa ke situs tertentu (misal memblokir situs judi/sosmed di jaringan kantor).
  - Menyembunyikan identitas/IP asli pengguna (anonimitas).
  - Menembus pembatasan geografis (*geo-restriction*).

### 2. Reverse Proxy (Mewakili Sisi Server)
**Reverse Proxy** duduk **di depan satu atau lebih Web Server kita**. Ketika pengguna di internet mengakses website kita, permintaan mereka dicegat oleh Reverse Proxy, lalu diteruskan ke server backend yang sesuai.

![reverse-proxy](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/proxy/reverse-proxy.png)

- **Intinya**: Pengguna di internet **tidak pernah berkomunikasi langsung dengan Origin Server kita**; mereka hanya tahu alamat Reverse Proxy.
- **Kegunaan Utama**:
  - **Keamanan Tinggi**: Menyembunyikan IP asli server backend dari serangan DDoS langsung.
  - **SSL Termination & Kompresi**: Menangani enkripsi HTTPS dan kompresi `Gzip/Brotli`.
  - **Caching**: Menyimpan halaman/aset statis agar tidak perlu membebani server aplikasi.
  - **Load Balancing**: Membagi trafik ke banyak server di belakangnya.

### Pertanyaan Klasik Interview: "Apa Bedanya Reverse Proxy vs Load Balancer?"
> **🎯 Jawaban Terbaik saat Interview:**
> - **Load Balancer** baru berguna jika kita memiliki **dua atau lebih server** yang menjalankan fungsi yang sama untuk membagi beban trafik.
> - **Reverse Proxy** tetap sangat berguna **meskipun kita hanya punya 1 server saja** (untuk menyembunyikan IP server, menangani SSL/HTTPS, caching, dan kompresi).
> - Dalam praktiknya, **Reverse Proxy modern (seperti Nginx, HAProxy, Envoy, Traefik) hampir selalu dapat berfungsi sekaligus sebagai Load Balancer**, tetapi tidak semua Load Balancer (seperti L4 Hardware Load Balancer murni) memiliki fitur lengkap Reverse Proxy.

---

## 11. Availability (Ketersediaan Sistem)

**Availability** adalah persentase waktu suatu sistem tetap beroperasi secara normal untuk menjalankan fungsinya dalam periode waktu tertentu.

$$
\text{Availability} = \frac{\text{Uptime}}{\text{Uptime} + \text{Downtime}} \times 100\%
$$

### Tabel "The Nines of Availability" (Wajib Tahu!)
Dalam industri Cloud & Backend, tingkat ketersediaan diukur dari jumlah angka **9 ("Nines")**. Standar emas untuk sistem *enterprise/production* umumnya adalah **"Four Nines" (99.99%)** atau **"Five Nines" (99.999%)**.

| Availability (%) | Sebutan | Maksimal Downtime / Tahun | Maksimal Downtime / Bulan | Maksimal Downtime / Minggu |
| :--- | :--- | :--- | :--- | :--- |
| **90%** | *One Nine* | 36,53 hari | 72 jam | 16,8 jam |
| **99%** | *Two Nines* | 3,65 hari | 7,20 jam | 1,68 jam |
| **99.9%** | *Three Nines* | 8,77 jam | 43,8 menit | 10,1 menit |
| **99.99%** | *Four Nines* ⭐ | **52,6 menit** | **4,32 menit** | **1,01 menit** |
| **99.999%** | *Five Nines* 🏆 | **5,25 menit** | **25,9 detik** | **6,05 detik** |
| **99.9999%** | *Six Nines* | 31,56 detik | 2,59 detik | 604,8 milidetik |
| **99.99999%** | *Seven Nines* | 3,15 detik | 263 milidetik | 60,5 milidetik |
| **99.999999%** | *Eight Nines* | 315,6 milidetik | 26,3 milidetik | 6 milidetik |
| **99.9999999%** | *Nine Nines* | 31,6 milidetik | 2,6 milidetik | 0,6 milidetik |

### Menghitung Availability: Komponen Berurutan (Sequence) vs Paralel (Parallel)

Jika sistem kita terdiri dari dua komponen (`Foo` dan `Bar`), bagaimana total *availability*-nya?

#### 1. Disusun Berurutan (Sequence / Seri) — *Availability Menurun!*
Jika `Foo` membutuhkan `Bar` untuk bekerja (jika salah satu mati, layanan gagal):

$$
\text{Availability (Total)} = \text{Availability (Foo)} \times \text{Availability (Bar)}
$$

- **Contoh**: Jika `Foo` = `99.9%` dan `Bar` = `99.9%`, maka totalnya dalam seri adalah $0.999 \times 0.999 = 99.8\%$. Semakin banyak komponen yang saling bergantung secara sinkron, semakin turun total *availability* sistem!

#### 2. Disusun Paralel (Redundan) — *Availability Meningkat Drastis!*
Jika `Foo` dan `Bar` adalah dua server kembar yang berjalan paralel (layanan tetap hidup selama minimal salah satu masih nyala):

$$
\text{Availability (Total)} = 1 - (1 - \text{Availability (Foo)}) \times (1 - \text{Availability (Bar)})
$$

- **Contoh**: Jika dua server yang masing-masing hanya punya availability `99.9%` dipasang paralel dengan failover otomatis, total availability-nya melonjak menjadi:
  $$1 - (1 - 0.999) \times (1 - 0.999) = 1 - 0.000001 = 99.9999\% \text{ (Six Nines!)}$$

### Perbedaan Istilah Penting
- **Availability vs Reliability**:
  - **Reliability (Keandalan)** adalah probabilitas sistem berjalan **tanpa kegagalan sedikit pun** selama periode tertentu.
  - Jika sistem *reliable*, pasti *available*. Namun sistem yang *tidak reliable* (sering crash) tetap bisa mencapai *high availability* jika kita memasang banyak server cadangan yang mengambil alih dalam milidetik!
- **High Availability vs Fault Tolerance**:
  - **High Availability (HA)** meminimalkan waktu gangguan (*minimal downtime*, misal butuh beberapa detik untuk *failover* ke server cadangan).
  - **Fault Tolerance** menjamin **nol gangguan sama sekali (*zero downtime*)** ketika komponen rusak (misal mesin pesawat terbang yang memiliki 3 komputer identik yang menyala bersamaan), namun biayanya jauh lebih mahal.

---

## 12. Scalability (Skalabilitas)

**Scalability** adalah kemampuan sebuah sistem untuk menangani peningkatan beban kerja dengan cara menambahkan sumber daya (*resources*).

![scalability](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-I/scalability/scalability.png)

> **💡 Analogi Pemula:**
> Bayangkan bisnis kateringmu kebanjiran pesanan:
> - **Vertical Scaling (*Scale Up*)**: Kamu mengganti 1 koki biasa dengan **1 MasterChef super cepat** dan memberinya kompor raksasa. (Simpel, tapi ada batas kemampuan fisik 1 manusia, dan kalau koki itu sakit, katering tutup total!).
> - **Horizontal Scaling (*Scale Out*)**: Kamu merekrut **10 koki tambahan** yang memasak bersama-sama di dapur menggunakan buku resep yang sama. (Kalau 1 koki sakit, 9 koki lain tetap masak!).

### Perbandingan Vertical Scaling vs Horizontal Scaling

| Aspek | Vertical Scaling (Scale Up) | Horizontal Scaling (Scale Out) |
| :--- | :--- | :--- |
| **Definisi** | Menambah kapasitas perangkat keras pada **1 mesin yang sudah ada** (Upgrade CPU, RAM, SSD). | Menambah **jumlah mesin/instance server baru** ke dalam kumpulan server (*pool*). |
| **Kemudahan Implementasi** | Sangat mudah (tidak perlu mengubah kode aplikasi). | Lebih kompleks (server harus *Stateless*, butuh Load Balancer & sinkronisasi data). |
| **Batas Maksimal** | Ada batas fisik maksimal spesifikasi 1 mesin (dan harganya makin mahal secara eksponensial). | Hampir **tak terbatas** (bisa menambah ribuan server murah sesuai kebutuhan). |
| **Downtime & SPOF** | Sering butuh *restart* saat upgrade hardware, dan menjadi **Single Point of Failure (SPOF)**. | **Zero downtime** saat menambah server, dan tahan banting (*fault-tolerant*). |
| **Konsistensi Data** | Mudah karena semua data berada di 1 mesin yang sama. | Menantang karena data tersebar di banyak mesin. |

> **🎯 Prinsip Arsitektur Utama:**
> Untuk sistem berskala besar, **selalu rancang layanan aplikasi secara Stateless** agar siap melakukan **Horizontal Scaling** kapan saja di belakang Load Balancer.

---

## 13. Storage (Penyimpanan Data)

Pemahaman tentang lapisan penyimpanan fisik maupun virtual sangat penting untuk memilih teknologi penyimpanan yang tepat bagi database, log, maupun file media.

### 1. RAID (Redundant Array of Independent Disks)
**RAID** adalah teknologi menggabungkan beberapa *hard disk* atau SSD fisik menjadi satu kesatuan logis untuk meningkatkan **kecepatan baca/tulis** dan/atau **keamanan data jika ada disk yang rusak**.

- **RAID 0 (Striping)**: Memecah data secara merata ke semua disk. Sangat cepat, kapasitas 100% terpakai, tetapi **nol toleransi kegagalan** (jika 1 disk rusak, seluruh data hilang!).
- **RAID 1 (Mirroring)**: Menyalin data yang persis sama (*mirror*) ke minimal 2 disk. Sangat aman jika 1 disk rusak, tapi kapasitas efektif terpotong menjadi **50%**.
- **RAID 5 (Striping with Parity)**: Memecah data beserta informasi pemulihan (*parity*) ke minimal 3 disk. Tahan terhadap kerusakan **1 disk**.
- **RAID 6 (Striping with Double Parity)**: Seperti RAID 5 tetapi menyimpan 2 blok *parity* (minimal 4 disk). Tahan terhadap kerusakan **2 disk sekaligus**.
- **RAID 10 (RAID 1+0: Striping + Mirroring)**: Menggabungkan kecepatan RAID 0 dan keamanan RAID 1 (minimal 4 disk). Pilihan terbaik untuk **Database berkinerja tinggi**.

| Fitur | RAID 0 | RAID 1 | RAID 5 | RAID 6 | RAID 10 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Teknik Utama** | *Striping* | *Mirroring* | *Striping + 1 Parity* | *Striping + 2 Parity* | *Striping + Mirroring* |
| **Jumlah Disk Minimal** | 2 | 2 | 3 | 4 | 4 |
| **Performa Baca (Read)** | Tinggi | Tinggi | Tinggi | Tinggi | **Sangat Tinggi** |
| **Performa Tulis (Write)** | Tinggi | Sedang | Tinggi | Tinggi | Sedang - Tinggi |
| **Toleransi Kerusakan** | **Tidak ada** (0 disk) | 1 disk rusak | 1 disk rusak | 2 disk rusak | 1 disk di tiap sub-array |
| **Kapasitas Efektif** | 100% | 50% | 67% - 94% | 50% - 80% | 50% |

### 2. Perbandingan File Storage vs Block Storage vs Object Storage

Dalam dunia Cloud Computing (AWS, GCP, Azure), terdapat 3 jenis penyimpanan utama yang wajib kamu bedakan saat wawancara:

| Jenis Penyimpanan | Cara Kerja | Analogi | Kapan Digunakan? | Contoh Layanan |
| :--- | :--- | :--- | :--- | :--- |
| **File Storage** | Menyimpan data sebagai file di dalam struktur folder/direktori hierarkis (`/folder/sub/file.txt`). Bisa diakses bersama lewat jaringan (NFS/SMB). | Lemari arsip kantor dengan map bertingkat yang bisa dibuka banyak staf. | Ketika banyak server membutuhkan akses bersama (*shared filesystem*) ke folder yang sama. | **Amazon EFS**, Azure Files, Google Cloud Filestore, **NAS**. |
| **Block Storage** | Memecah data menjadi blok-blok berukuran tetap dengan ID unik. Dipasang ke server seolah-olah seperti *hard disk* lokal. | Hard disk / SSD internal yang dicolok langsung ke komputer. | Untuk **menyimpan Sistem Operasi VM dan Database Transaksional (SQL/NoSQL)** yang butuh latensi sangat rendah. | **Amazon EBS**, Azure Managed Disks, Google Persistent Disk. |
| **Object Storage** | Menyimpan file utuh sebagai **Objek** datar (*flat*) beserta *Metadata* dan *Unique ID*, diakses melalui REST API (`HTTP GET/PUT`). | Gudang penitipan barang raksasa: kamu menyerahkan koper, lalu diberi **nomor tiket (URL)** untuk mengambilnya lagi nanti. | Untuk menyimpan **file statis dan media berukuran masif** (Foto profil, Video Netflix, Backup DB, File PDF). Tidak cocok untuk diedit baris per baris. | **Amazon S3**, Google Cloud Storage (GCS), Azure Blob Storage, MinIO. |

### 3. NAS dan HDFS
- **NAS (*Network Attached Storage*)**: Perangkat penyimpanan khusus yang terhubung ke jaringan lokal kantor/data center sehingga pengguna resmi dapat menyimpan dan mengambil file dari satu lokasi terpusat.
- **HDFS (*Hadoop Distributed File System*)**: Sistem penyimpanan file terdistribusi yang dirancang untuk menyimpan file berukuran raksasa (Terabyte hingga Petabyte) di atas ratusan komputer standar (*commodity hardware*). HDFS memecah setiap file besar menjadi blok-blok besar (misal 128 MB per blok) dan mereplikasi setiap blok ke beberapa mesin agar tahan terhadap kerusakan perangkat keras.
