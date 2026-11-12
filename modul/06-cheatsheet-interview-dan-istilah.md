# ⚡ Bab VI: Cheat Sheet Wawancara, Angka Penting & Kamus Istilah

> **🎯 Tujuan Bab Tambahan Ini:**
> Bab ini adalah **lembar contekan cepat (Quick Reference Cheat Sheet)** yang wajib kamu baca 15 menit sebelum masuk ke ruangan *System Design Interview*!

---

## 1. Angka Latensi yang Wajib Dihafal (*Latency Numbers Every Programmer Should Know*)

Memahami perbandingan kecepatan antara CPU Cache, RAM, SSD, dan Jaringan Internet akan membuat estimasi desainmu terdengar seperti seorang *Senior Architect*:

| Operasi | Waktu (Nanodetik / Mikrodetik / Milidetik) | Analogi jika 1 ns = 1 Detik Manusia |
| :--- | :--- | :--- |
| **Referensi L1 Cache CPU** | `0,5 ns` | 0,5 detik (Kedipan mata) |
| **Referensi L2 Cache CPU** | `7 ns` | 7 detik |
| **Baca dari Memori Utama (RAM / Redis in-memory)** | `100 ns` (`0,1 µs`) | **1,5 menit** |
| **Kompresi 1 KB dengan Snappy** | `3.000 ns` (`3 µs`) | 50 menit |
| **Baca Acak dari SSD (NVMe/SATA)** | `16.000 – 150.000 ns` (`16 – 150 µs`) | **4 jam – 1,7 hari** |
| **Kirim Paket 1 MB lewat Jaringan 1 Gbps (Di dalam 1 Data Center)** | `500.000 ns` (`0,5 ms`) | **5,8 hari** |
| **Baca 1 MB Berurutan dari SSD** | `1.000.000 ns` (`1 ms`) | **11,5 hari** |
| **Pencarian Disk Magnetik (HDD Seek)** | `10.000.000 ns` (`10 ms`) | **4 bulan** |
| **Kirim Paket Pulang-Pergi Lintas Benua (AS ke Eropa / Asia)** | `150.000.000 ns` (`150 ms`) | **4,7 tahun!** |

> **💡 Kesimpulan Kunci untuk Interview:**
> 1. **Membaca dari RAM (Redis/Memcached) ~1.000x lebih cepat daripada membaca dari SSD Database!**
> 2. **Panggilan jaringan lintas benua (`150 ms`) sangat mahal**—itulah sebabnya kita wajib menggunakan **CDN** dan **Multi-Region Deployment**!

---

## 2. Tabel Konversi Satuan Data & Waktu (Untuk Hitung Cepat *Back-of-the-Envelope*)

### Satuan Penyimpanan (Pangkat 10 / Pangkat 2)
| Satuan | Nilai Byte | Pangkat 10 | Contoh Ukuran di Dunia Nyata |
| :--- | :--- | :---: | :--- |
| **1 Byte (B)** | 8 bits | $10^0$ | 1 karakter huruf ASCII (`'A'`). |
| **1 Kilobyte (KB)** | 1.000 Bytes | $10^3$ | 1 paragraf teks / JSON payload API standar (`500 B – 2 KB`). |
| **1 Megabyte (MB)** | 1.000 KB | $10^6$ | 1 foto resolusi tinggi yang sudah dikompresi / 1 lagu MP3 pendek. |
| **1 Gigabyte (GB)** | 1.000 MB | $10^9$ | 1 film HD durasi 1 jam / Kapasitas RAM Cache standar. |
| **1 Terabyte (TB)** | 1.000 GB | $10^{12}$ | Ukuran database produksi menengah (miliaran baris teks). |
| **1 Petabyte (PB)** | 1.000 TB | $10^{15}$ | Penyimpanan media skala WhatsApp / Netflix selama bertahun-tahun. |
| **1 Exabyte (EB)** | 1.000 PB | $10^{18}$ | Skala data center global Google / AWS. |

### Rumus Pintas Menghitung RPS (*Requests Per Second*)
Ingat bahwa **1 Hari = $24 \times 3600 = 86.400 \text{ detik} \approx 100.000 \text{ detik} (10^5)$**:
- **1 Juta request / hari** $\approx \mathbf{12 \text{ RPS}}$
- **10 Juta request / hari** $\approx \mathbf{120 \text{ RPS}}$
- **100 Juta request / hari** $\approx \mathbf{1.200 \text{ RPS}}$
- **1 Miliar request / hari** $\approx \mathbf{12.000 \text{ RPS}}$
- **2 Miliar request / hari** $\approx \mathbf{24.000 \text{ RPS}}$

---

## 3. Matriks Keputusan Pemilihan Teknologi (*Cheat Sheet Arsitek*)

Gunakan tabel ini ketika *interviewer* bertanya: *"Teknologi apa yang kamu pilih untuk komponen X dan mengapa?"*

| Kebutuhan / Masalah Sistem | Solusi / Komponen yang Dipilih | Contoh Teknologi Industri |
| :--- | :--- | :--- |
| **Transaksi Finansial, Saldo, Order, Relasi Ketat (ACID)** | Relational Database (RDBMS) | **PostgreSQL**, **MySQL**, Amazon Aurora, CockroachDB |
| **Pencarian Cepat Key-Value, Caching, Session, Rate Limit, Leaderboard** | In-Memory Key-Value Store | **Redis**, **Memcached**, Dragonfly |
| **Riwayat Chat Miliaran Pesan / Write-Heavy Masif** | Wide-Column NoSQL Database | **Apache Cassandra**, **ScyllaDB**, Google Bigtable |
| **Katalog Produk Fleksibel / Dokumen JSON Dinamis** | Document NoSQL Database | **MongoDB**, Amazon DocumentDB, Firestore |
| **Jejaring Pertemanan (Mutual Friends) / Fraud Detection** | Graph Database | **Neo4j**, Amazon Neptune, ArangoDB |
| **Metrik Monitoring Server, IoT Sensor, Data Waktu** | Time-Series Database (TSDB) | **InfluxDB**, **Prometheus**, TimescaleDB |
| **Pencarian Teks Lengkap (Full-Text Search), Autocomplete, Log** | Search Engine (Inverted Index) | **Elasticsearch**, OpenSearch, Meilisearch |
| **Penyimpanan Foto, Video, PDF, Audio, Backup Statis** | Object Storage + CDN | **Amazon S3** + **Cloudflare / CloudFront** |
| **Antrean Tugas Latar Belakang (Kirim Email, Proses Video)** | Message Queue (Point-to-Point) | **RabbitMQ**, **Amazon SQS**, BullMQ |
| **Event Streaming Skala Raksasa, Log Analytics, Pub/Sub** | Distributed Event Streaming | **Apache Kafka**, Google Pub/Sub, Redpanda, NATS |
| **Komunikasi Internal Antar-Microservices Cepat** | RPC Biner di atas HTTP/2 | **gRPC (Protocol Buffers)** |
| **Komunikasi Real-Time 2 Arah (Chat, Lokasi Driver, Game)** | Koneksi TCP Persisten Full-Duplex | **WebSockets** |
| **Notifikasi Satu Arah Server ke Browser (Ticker Saham, AI Stream)** | Streaming HTTP Satu Arah | **Server-Sent Events (SSE)** |
| **Pencarian Lokasi Terdekat (Driver Ojek, Restoran Sekitar)** | Indeks Spasial | **Quadtree** (In-Memory) / **Geohash** (Redis Geo / PostGIS) |

---

## 4. Sumber Belajar Lanjutan & Referensi Resmi

### Buku & Kursus Rekomendasi
1. **Designing Data-Intensive Applications (DDIA)** — *Dr. Martin Kleppmann* (Kitab suci wajib bagi setiap Backend/System Engineer).
2. **System Design Interview: An Insider's Guide (Vol. 1 & Vol. 2)** — *Alex Xu*.
3. **Kursus Asli Bahasa Inggris**: [System Design oleh Karan Pratap Singh](https://github.com/karanpratapsingh/system-design).

### Engineering Blogs Perusahaan Teknologi Kelas Dunia
Biasakan membaca bagaimana perusahaan raksasa memecahkan masalah arsitektur nyata:
- [Netflix Tech Blog](https://netflixtechblog.com)
- [Uber Engineering Blog](https://eng.uber.com)
- [Discord Engineering Blog](https://discord.com/category/engineering)
- [Cloudflare Blog](https://blog.cloudflare.com)
- [AWS Architecture Blog](https://aws.amazon.com/blogs/architecture)
- [GitHub Engineering](https://github.blog/category/engineering)
- [Martin Fowler's Architecture Guide](https://martinfowler.com)
