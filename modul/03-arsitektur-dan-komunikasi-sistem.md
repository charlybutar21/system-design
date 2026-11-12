# 🏗️ Bab III: Pola Arsitektur Aplikasi & Komunikasi Antar-Layanan

> **🎯 Tujuan Belajar Bab Ini:**
> Setelah memahami jaringan dan database, kini saatnya merancang bagaimana kode aplikasi kita disusun (N-Tier, Clean Architecture, Monolith vs Microservices) dan bagaimana layanan-layanan tersebut saling berbicara—baik secara **sinkron** (REST, GraphQL, gRPC), **asinkron berbasis pesan** (Message Queue, Pub/Sub, Event-Driven, Event Sourcing, CQRS), maupun **real-time** (Long Polling, WebSockets, SSE).

---

## Daftar Isi Bab III
1. [N-Tier Architecture & Clean Architecture](#1-n-tier-architecture)
2. [Message Brokers (Perantara Pesan)](#2-message-brokers)
3. [Message Queues (Antrean Pesan Point-to-Point)](#3-message-queues)
4. [Publish-Subscribe (Pub/Sub)](#4-publish-subscribe-pubsub)
5. [Enterprise Service Bus (ESB)](#5-enterprise-service-bus-esb)
6. [Monolith, Modular Monolith, dan Microservices](#6-monolith-modular-monolith-dan-microservices)
7. [Event-Driven Architecture (EDA)](#7-event-driven-architecture-eda)
8. [Event Sourcing](#8-event-sourcing)
9. [Command and Query Responsibility Segregation (CQRS)](#9-command-and-query-responsibility-segregation-cqrs)
10. [API Gateway & Pola Backend For Frontend (BFF)](#10-api-gateway--backend-for-frontend-bff)
11. [REST vs GraphQL vs gRPC](#11-rest-vs-graphql-vs-grpc)
12. [Komunikasi Real-Time: Long Polling, WebSockets, dan Server-Sent Events (SSE)](#12-long-polling-websockets-dan-server-sent-events-sse)

---

## 1. N-Tier Architecture

**N-Tier Architecture** membagi sebuah aplikasi menjadi **Lapisan Logis (*Logical Layers*)** dan **Tingkatan Fisik (*Physical Tiers*)** untuk memisahkan tanggung jawab (*Separation of Concerns*) serta mengelola ketergantungan antar bagian kode.

![n-tier-architecture](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-III/n-tier-architecture/n-tier-architecture.png)

- **Layer (Lapisan Logis)**: Pembagian tanggung jawab di dalam struktur kode. Layer yang lebih atas boleh memanggil layanan di layer bawahnya, tetapi tidak sebaliknya.
  - **Closed Layer Architecture**: Suatu layer hanya boleh memanggil layer tepat satu tingkat di bawahnya (meminimalkan ketergantungan/coupling).
  - **Open Layer Architecture**: Suatu layer boleh memanggil layer mana pun di bawahnya secara langsung (mengurangi loncatan perantara).
- **Tier (Tingkatan Fisik)**: Mesin/server fisik tempat kode dijalankan. Memisahkan *tier* ke mesin berbeda meningkatkan skalabilitas dan keamanan, namun menambah sedikit latensi jaringan.

### Jenis-Jenis N-Tier Architecture
1. **1-Tier (Single-Tier)**: Seluruh antarmuka, logika bisnis, dan database berjalan di dalam 1 komputer yang sama (contoh: aplikasi desktop Microsoft Excel atau game offline).
2. **2-Tier**: Aplikasi klien (Presentation) berkomunikasi langsung dengan server Database tanpa perantara server aplikasi terpisah.
3. **3-Tier Architecture** *(Paling Umum)*:
   - **Presentation Layer (Tier 1 — Frontend)**: Menangani tampilan UI dan interaksi pengguna (Web Browser, Aplikasi Mobile iOS/Android).
   - **Business Logic / Application Layer (Tier 2 — Backend)**: Menerima permintaan dari UI, memvalidasi aturan bisnis, melakukan perhitungan, dan mengatur alur kerja.
   - **Data Access Layer (Tier 3 — Database/Storage)**: Menyimpan dan mengambil data dari Database, Cache, atau Object Storage.

> **🎯 Catatan Arsitek (Hexagonal / Clean Architecture):**
> Di dalam *Business Logic Layer*, praktik terbaik modern adalah menerapkan **Clean / Hexagonal Architecture (Ports & Adapters)**: pisahkan **Logika Bisnis Inti (*Domain*)** agar tidak bergantung langsung pada framework HTTP atau jenis database tertentu. Dengan begitu, logika bisnis mudah diuji (*Unit Test*) dan database bisa diganti tanpa merombak aturan bisnis!

---

## 2. Message Brokers

**Message Broker** adalah perangkat lunak perantara yang memungkinkan berbagai aplikasi, sistem, dan *microservices* untuk saling berkomunikasi dan bertukar pesan secara **asinkron**, meskipun masing-masing layanan ditulis menggunakan bahasa pemrograman atau platform yang berbeda.

![message-broker](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-III/message-brokers/message-broker.png)

Message Broker bertugas memvalidasi, menyimpan sementara, merutekan (*routing*), dan mengirimkan pesan ke tujuan yang tepat. Pengirim pesan (*Producer*) tidak perlu tahu di mana penerima (*Consumer*) berada atau apakah penerima sedang sibuk/mati.

### 2 Pola Utama Distribusi Pesan
1. **Point-to-Point Messaging (*Message Queue*)**: Hubungan **1-ke-1** di mana satu pesan di dalam antrean hanya akan diambil dan diproses oleh **tepat satu Consumer**.
2. **Publish-Subscribe Messaging (*Pub/Sub*)**: Hubungan **1-ke-Banyak** di mana satu pesan yang dikirim ke sebuah *Topic* akan disiarkan (*broadcast*) ke **semua Subscriber** yang berlangganan topik tersebut.

### Perbedaan Penting: Message Broker vs Event Streaming Platform

| Aspek | Message Broker Tradisional (RabbitMQ, ActiveMQ) | Event Streaming Platform (Apache Kafka, Pulsar) |
| :--- | :--- | :--- |
| **Perlakuan pada Pesan** | Pesan **dihapus dari antrean** segera setelah berhasil diproses (*acknowledged*) oleh Consumer. | Pesan disimpan sebagai **Log Berurutan (*Append-only Log*)** di dalam disk selama periode tertentu (misal 7 hari atau selamanya), sehingga **bisa diputar ulang (*replay*)**! |
| **Kecerdasan Routing** | *Smart Broker, Dumb Consumer*: Broker bisa melakukan routing kompleks dan melacak status tiap pesan per consumer. | *Dumb Broker, Smart Consumer*: Broker fokus pada kecepatan *throughput* raksasa; Consumer sendiri yang mencatat posisi baca (*offset*). |
| **Skala & Throughput** | Sangat baik untuk antrean tugas (*task queue*) puluhan ribu pesan/detik. | Luar biasa masif (ratusan ribu hingga jutaan *event* per detik). |

---

## 3. Message Queues

**Message Queue (Antrean Pesan)** memfasilitasi komunikasi **asinkron** antar layanan: *Producer* menaruh pekerjaan (*job/message*) ke dalam antrean lalu langsung lanjut melayani pengguna tanpa harus menunggu pekerjaan berat tersebut selesai!

![message-queue](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-III/message-queues/message-queue.png)

> **💡 Analogi Pemula:**
> Bayangkan kamu memesan kopi di Starbucks:
> - **Tanpa Message Queue (Sinkron)**: Kasir menerima pembayaranmu, lalu kasir itu sendiri pergi menggiling kopi, menyeduh susu, dan membuat kopi selama 5 menit. Selama 5 menit itu, pelanggan di belakangmu tidak bisa dilayani!
> - **Dengan Message Queue (Asinkron)**: Kasir menerima pesananmu dalam 10 detik, menempelkan **struk pesanan ke meja antrean (Message Queue)**, lalu langsung melayani pelanggan berikutnya! Di belakang, para **Barista (Worker/Consumer)** mengambil struk satu per satu dan meracik kopi secara paralel.

### Mengapa Message Queue Wajib untuk Proses Lambat?
Semua proses yang memakan waktu lama—seperti **mengirim email/WA OTP, men-generate laporan PDF, memproses/mengompresi video, atau memanggil API pihak ketiga yang lambat**—WAJIB dipisahkan dari *thread* HTTP utama menggunakan Message Queue agar respons API ke pengguna tetap instan!

### Fitur-Fitur Penting Message Queue
1. **Push vs Pull Delivery**: Consumer bisa menarik pesan sendiri saat siap (*Pull*) atau dikirimi langsung oleh antrean (*Push*).
2. **FIFO (*First-In-First-Out*)**: Pesan yang masuk paling awal diproses paling awal.
3. **Schedule / Delay Queue**: Menunda pengiriman pesan selama waktu tertentu (misal: *"Cek apakah order sudah dibayar setelah 15 menit; jika belum, otomatis batalkan order"*).
4. **Jaminan Pengiriman (*Delivery Guarantees*)**:
   - **At-Most-Once**: Pesan dikirim maksimal 1 kali (bisa hilang jika jaringan gagal, tapi tidak pernah duplikat).
   - **At-Least-Once** *(Paling Umum)*: Pesan dijamin sampai minimal 1 kali (jika tidak ada `ACK`, pesan dikirim ulang). *Catatan: Consumer wajib bersifat **Idempotent** karena pesan yang sama bisa diterima lebih dari sekali!*
   - **Exactly-Once**: Pesan diproses tepat 1 kali tanpa duplikasi (paling sulit dan membutuhkan koordinasi/deduplikasi otomatis).
5. **Dead-Letter Queue (DLQ)**: Antrean khusus tempat memindahkan pesan-pesan yang **terus-menerus gagal diproses** setelah beberapa kali percobaan (*retry*). Dengan DLQ, pesan error tidak menyumbat antrean utama dan bisa dianalisis oleh developer.
6. **Poison-Pill Messages**: Pesan rusak yang tidak bisa diproses, atau pesan khusus yang sengaja dikirim untuk memberi sinyal kepada *Consumer* agar menghentikan pekerjaannya dengan aman (*graceful shutdown*).
7. **Backpressure (Tekanan Balik)**: Jika jumlah pesan yang masuk jauh lebih cepat daripada kemampuan *Consumer* memprosesnya, memori antrean bisa meluap. *Backpressure* membatasi kapasitas antrean dan mengembalikan status `HTTP 503 Service Unavailable (Server Busy)` ke pengirim agar melakukan percobaan ulang nanti dengan strategi **Exponential Backoff** (menunggu 1 detik, 2 detik, 4 detik, 8 detik, dst.).

> **Contoh Teknologi Message Queue**: **RabbitMQ**, **Amazon SQS**, **ActiveMQ**, **ZeroMQ**, **BullMQ (Redis)**.

---

## 4. Publish-Subscribe (Pub/Sub)

Dalam model **Publish-Subscribe (Pub/Sub)**, pengirim pesan (**Publisher**) mempublikasikan pesan ke sebuah saluran bernama **Topic**, dan pesan tersebut langsung didorong (*pushed*) secara serentak ke **semua penerima (Subscribers)** yang berlangganan *Topic* tersebut.

![publish-subscribe](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-III/publish-subscribe/publish-subscribe.png)

> **💡 Analogi Pemula: Message Queue vs Pub/Sub**
> - **Message Queue** ibarat **Tumpukan Cucian Piring di Dapur Restoran**: Jika ada 3 pencuci piring, piring nomor 1 yang diambil oleh Pencuci A tidak perlu dicuci lagi oleh Pencuci B dan C.
> - **Pub/Sub** ibarat **Siaran Radio / Grup WhatsApp Pengumuman**: Saat penyiar berbicara di frekuensi 101 FM (*Topic*), **semua orang** yang menyetel radionya ke 101 FM (*Subscribers*) akan mendengar suara yang sama secara bersamaan, dan masing-masing pendengar bisa melakukan reaksi yang berbeda!

### Fitur & Keunggulan Utama Pub/Sub
- **Fanout (Penyebaran Paralel)**: Satu kejadian bisnis (misal: `OrderPaidEvent` saat pelanggan membayar pesanan) dikirim ke 1 Topic, lalu otomatis digandakan (*fanout*) ke:
  1. *Notification Service* (mengirim email struk pembayaran),
  2. *Warehouse Service* (menyiapkan pengiriman barang),
  3. *Analytics Service* (mencatat omzet harian), dan
  4. *Loyalty Service* (menambah poin reward pelanggan) — **semuanya berjalan paralel tanpa saling menunggu!**
- **Message Filtering**: Subscriber dapat memasang kebijakan filter agar hanya menerima pesan yang relevan dari sebuah Topic.
- **Menghilangkan Polling**: Pengiriman berbasis *push* instan tanpa perlu cek berkala.
- **Contoh Layanan Pub/Sub**: **Google Cloud Pub/Sub**, **Amazon SNS**, **Apache Kafka**, **Redis Pub/Sub**, **NATS**.

---

## 5. Enterprise Service Bus (ESB)

**Enterprise Service Bus (ESB)** adalah pola arsitektur era *Service-Oriented Architecture (SOA)* di mana terdapat satu komponen perangkat lunak terpusat yang sangat pintar untuk menghubungkan berbagai aplikasi di perusahaan—mulai dari mengubah format data (XML ke JSON), mengonversi protokol komunikasi, hingga mengatur alur orkestrasi bisnis.

![enterprise-service-bus](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-III/enterprise-service-bus/enterprise-service-bus.png)

- **Mengapa ESB Mulai Ditinggalkan di Era Microservices?**
  - Karena ESB menaruh terlalu banyak logika bisnis dan transformasi di satu pipa tengah (*Smart Pipes, Dumb Endpoints*), ESB akhirnya menjadi **bottleneck raksasa**, **Single Point of Failure**, sangat mahal, dan sulit di-update lintas tim.
  - Arsitektur Microservices modern beralih ke filosofi kebalikannya: **"Smart Endpoints and Dumb Pipes"** (gunakan Message Broker ringan seperti Kafka/RabbitMQ atau REST/gRPC yang hanya bertugas mengantar pesan, sementara logika bisnis tetap berada di masing-masing service).

---

## 6. Monolith, Modular Monolith, dan Microservices

Memilih antara **Monolith**, **Modular Monolith**, dan **Microservices** adalah salah satu keputusan arsitektur paling krusial.

### 1. Monolith (Arsitektur Monolitik)
Seluruh fitur aplikasi (Autentikasi, Profil, Katalog Produk, Keranjang, Pembayaran, Notifikasi) dibangun di dalam **satu repositori kode tunggal**, dikompilasi menjadi **satu unit aplikasi**, dan dijalankan sebagai **satu proses server** yang terhubung ke satu database.

![monolith](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-III/monoliths-microservices/monolith.png)

- **Kelebihan**:
  - Sangat mudah dikembangkan, dijalankan di laptop lokal, di-test, dan di-*deploy* saat awal membangun produk.
  - Komunikasi antar fungsi terjadi di dalam memori (*in-memory function call*) tanpa latensi jaringan.
  - Sangat mudah menjaga konsistensi data dengan **transaksi ACID (`BEGIN ... COMMIT`)** karena semuanya berada di satu database.
- **Kekurangan**:
  - Ketika kode membesar hingga jutaan baris dan dikerjakan puluhan developer, kode mudah menjadi kusut (*spaghetti code*).
  - Perubahan satu baris kode saja mengharuskan kita men-*deploy* ulang seluruh aplikasi.
  - Satu *bug* fatal (misal *memory leak* di fitur cetak laporan PDF) bisa membuat seluruh aplikasi mati total termasuk fitur pembayaran!

### 2. Modular Monolith ⭐ *(Rekomendasi Terbaik untuk Startup & Sistem Baru)*
**Modular Monolith** adalah jalan tengah yang sangat cerdas: aplikasi tetap di-*deploy* sebagai **satu unit tunggal (Monolith)**, tetapi di dalam kodenya dipecah secara ketat menjadi **modul-modul independen** sesuai domain bisnis (misal modul `user`, modul `order`, modul `payment`) yang tidak boleh saling mengintip tabel database secara sembarangan.
- **Keuntungan**: Kamu mendapatkan kesederhanaan operasional Monolith (tidak pusing mengurus ratusan server dan jaringan terdistribusi), sekaligus mendapatkan kerapian kode ala Microservices. Jika suatu saat satu modul butuh dipisah menjadi Microservice mandiri, proses ekstraksinya sangat mudah!

### 3. Microservices Architecture
Aplikasi dipecah menjadi sekumpulan **layanan kecil yang otonom (*autonomous services*)**, di mana setiap service hanya fokus menangani **satu kapabilitas bisnis spesifik (*Bounded Context*)**, memiliki codebase sendiri, dikelola oleh tim kecil mandiri, di-*deploy* secara terpisah, dan **wajib memiliki database sendiri (*Database-per-Service*)**.

![microservices](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-III/monoliths-microservices/microservices.png)

- **Kelebihan**:
  - **Independent Deployment & Scaling**: Jika hanya layanan *Flash Sale* yang kebanjiran trafik, kita cukup memperbanyak server *Flash Sale Service* tanpa perlu memperbanyak server *User Profile*.
  - **Fault Isolation**: Jika *Recommendation Service* mati, pengguna masih tetap bisa mencari barang dan membayar.
  - **Otonomi Tim & Fleksibilitas Teknologi**: Tim A bisa menulis *Payment Service* dengan Go/Java, sementara Tim B menulis *AI Search Service* dengan Python.
- **Kekurangan**:
  - Kompleksitas sistem terdistribusi sangat tinggi (butuh *Service Discovery, Distributed Tracing, API Gateway, Circuit Breaker*).
  - Menjaga konsistensi data lintas database sangat menantang (harus memakai *Saga Pattern / Eventual Consistency*).
  - Biaya infrastruktur dan pemeliharaan DevOps lebih mahal.

### ⚠️ Bahaya "Distributed Monolith" (Monolit Terdistribusi)
Jebakan paling mematikan saat tim buru-buru mengadopsi Microservices adalah menciptakan **Distributed Monolith**: secara fisik terpisah menjadi 20 service berbeda, tetapi:
1. Semua service masih **membaca/menulis ke 1 database yang sama (*shared database*)**, ATAU
2. Setiap *request* harus memanggil 10 service lain secara berantai dan sinkron, sehingga jika 1 service mati atau di-*deploy*, ke-19 service lainnya ikut lumpuh!
- *Distributed Monolith* menggabungkan **semua keburukan Monolith** dan **semua kerumitan Microservices** tanpa mendapatkan keuntungannya sama sekali.

### Kapan Kamu TIDAK Butuh Microservices?

![architecture-range](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-III/monoliths-microservices/architecture-range.png)

> **🎯 Nasihat Senior Engineer:**
> *"Jangan meniru arsitektur Netflix di hari pertama jika jumlah penggunamu dan ukuran tim engineer-mu belum sebesar Netflix!"*
> Microservices pada dasarnya memecahkan **masalah skala organisasi (ratusan engineer yang bekerja paralel)** dan **skala beban tidak merata**. Saat memulai sistem baru di dunia nyata, mulailah dengan **Modular Monolith yang bersih (Clean/Hexagonal Architecture)**, dan pecah menjadi Microservices hanya ketika benar-benar dibutuhkan! *(Namun dalam sesi System Design Interview untuk perusahaan besar, tetap gunakan pendekatan Microservices/komponen terdistribusi jika skala soalnya mencapai puluhan juta pengguna).*

---

## 7. Event-Driven Architecture (EDA)

**Event-Driven Architecture (EDA)** adalah paradigma arsitektur di mana komponen-komponen sistem berkomunikasi dengan cara memproduksi dan merespons **Events (Kejadian Perubahan Status)** secara asinkron.

- **Apa itu Event?** Sebuah fakta yang mencatat bahwa *"sesuatu baru saja terjadi di dalam sistem"* pada waktu lampau (biasanya diberi nama bentuk lampau seperti `UserRegistered`, `OrderPlaced`, `PaymentSucceeded`, `DriverArrived`).
- Event **tidak memberi perintah** kepada siapa pun tentang apa yang harus dilakukan; Event hanya mengumumkan fakta perubahan status.

### 3 Komponen Utama EDA
1. **Event Producer**: Layanan yang mendeteksi adanya aksi/perubahan lalu menerbitkan *event* ke router.
2. **Event Router (Broker / Bus)**: Menyaring dan menyalurkan *event* ke konsumen yang tertarik (Contoh: Apache Kafka, Amazon EventBridge, RabbitMQ, Google Pub/Sub).
3. **Event Consumer**: Layanan yang menerima *event* dan menjalankan reaksi bisnis masing-masing.

![event-driven-architecture](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-III/event-driven-architecture/event-driven-architecture.png)

- **Keunggulan**: Sangat *loosely coupled* (Producer dan Consumer tidak saling kenal); menambah fitur baru sangat mudah (cukup tambahkan Consumer baru yang mendengarkan event yang sudah ada tanpa menyentuh kode Producer sama sekali!).
- **Tantangan**: Alur sistem lebih sulit dilacak (*debugging* butuh *Correlation ID / Distributed Tracing*), serta harus menangani urutan event (*ordering*) dan duplikasi event (*idempotency*).

---

## 8. Event Sourcing

Pada database tradisional (CRUD), kita hanya menyimpan **kondisi terakhir (*current state*)** dari sebuah data dengan menimpa nilai lama (`UPDATE accounts SET balance = 150000 WHERE id = 1`). Akibatnya, kita kehilangan sejarah bagaimana saldo tersebut bisa menjadi Rp150.000!

**Event Sourcing** mengambil pendekatan radikal yang berbeda: **Kita tidak menimpa state terakhir, melainkan menyimpan seluruh urutan kejadian (*events*) yang pernah terjadi sejak awal di dalam penyimpanan *Append-Only (Event Store)***.

![event-sourcing](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-III/event-sourcing/event-sourcing.png)

> **💡 Analogi Pemula (Buku Tabungan Bank):**
> Event Sourcing persis seperti **Mutasi Rekening Bank / Buku Besar Akuntansi**:
> Bank tidak hanya menulis satu angka *"Saldo Budi = Rp150.000"* lalu menghapus angka kemarin. Bank mencatat daftar mutasi yang tidak pernah boleh dihapus:
> 1. `AccountOpened (+Rp0)`
> 2. `MoneyDeposited (+Rp200.000)`
> 3. `MoneyWithdrawn (-Rp50.000)`
> Dari deretan *event* di atas, kapan pun kita menjumlahkannya dari awal (*replay events*), kita pasti mendapatkan saldo saat ini yaitu **Rp150.000**!

### Perbedaan Event-Driven Architecture (EDA) vs Event Sourcing
- **EDA**: Menggunakan *event* sebagai **cara komunikasi antar-service** melewati jaringan.
- **Event Sourcing**: Menggunakan *event* sebagai **cara menyimpan data di dalam database (*state storage*)** pengganti `UPDATE/DELETE` tradisional.

### Kelebihan & Kekurangan Event Sourcing
- **Kelebihan**:
  - **100% Audit Trail Akurat**: Sangat ideal untuk sistem perbankan, dompet digital, akuntansi, dan medis yang wajib memiliki bukti histori lengkap yang tidak bisa dimanipulasi.
  - **Time Travel & Debugging**: Kita bisa merekonstruksi kondisi sistem pada detik mana pun di masa lalu hanya dengan memutar ulang (*replay*) event sampai titik waktu tersebut!
- **Kekurangan**:
  - Untuk mengetahui status saat ini dari akun yang sudah punya 10.000 transaksi, menjumlahkan dari awal akan lambat—sehingga membutuhkan mekanisme **Snapshot** berkala (misal menyimpan saldo akhir tiap tutup hari) dan pola **CQRS**.
  - Mengubah format/skema event lama membutuhkan *Schema Registry* dan versi event yang rapi.

---

## 9. Command and Query Responsibility Segregation (CQRS)

**CQRS** (diperkenalkan oleh Greg Young) adalah pola arsitektur yang **memisahkan model dan jalur untuk mengubah data (Command / Write)** dari **model dan jalur untuk membaca data (Query / Read)**.

![command-and-query-responsibility-segregation](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-III/command-and-query-responsibility-segregation/command-and-query-responsibility-segregation.png)

- **Command (Perintah Tulis)**: Instruksi untuk mengubah status sistem (misal `CreateOrder`, `UpdateAddress`). Memvalidasi aturan bisnis yang ketat, menulis ke **Write Database** (misal PostgreSQL yang ternormalisasi 3NF atau Event Store), dan tidak mengembalikan data selain status berhasil/gagal.
- **Query (Permintaan Baca)**: Permintaan untuk melihat data tanpa mengubah apa pun. Membaca langsung dari **Read Database** khusus (misal Elasticsearch, Redis, atau tabel SQL/MongoDB yang sudah sangat ter-denormalisasi / *Materialized View*) sehingga super cepat tanpa `JOIN`!

### Mengapa Menggabungkan CQRS + Event Sourcing?
Pada sistem *Event Sourcing*, menulis event baru sangat cepat, tetapi melakukan query pencarian (misal *"Tampilkan semua pengguna yang saldonya di atas Rp10 juta"*) sangat sulit jika hanya punya daftar mutasi event.
Dengan **CQRS + Event Sourcing**:
1. **Write Side** menyimpan mutasi di *Event Store*.
2. Setiap ada event baru, *Event Store* menyiarkan event tersebut ke **Read Side** untuk memperbarui tampilan data siap saji (*Denormalized Read View*).
3. Aplikasi yang butuh membaca data tinggal mengambil dari *Read View* dalam hitungan milidetik!

> **⚠️ Catatan Trade-off CQRS:**
> Karena pembaruan dari *Write DB* ke *Read DB* biasanya berjalan secara asinkron melalui event, akan ada jeda **Eventual Consistency** (beberapa milidetik). Jangan gunakan CQRS untuk aplikasi CRUD sederhana karena akan menambah kompleksitas yang tidak perlu.

---

## 10. API Gateway & Backend For Frontend (BFF)

### 1. API Gateway
Dalam arsitektur Microservices yang memiliki 50 layanan berbeda, tidak mungkin aplikasi HP pengguna harus menghafal alamat 50 server dan memanggilnya satu per satu.

**API Gateway** adalah **pintu gerbang tunggal (*Single Entry Point*)** yang duduk di antara klien luar dan seluruh *microservices* di belakang layar.

![api-gateway](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-III/api-gateway/api-gateway.png)

#### Tanggung Jawab Utama API Gateway:
- **Routing & Reverse Proxy**: Mengarahkan `/users/*` ke *User Service*, `/orders/*` ke *Order Service*.
- **Authentication & Authorization**: Memverifikasi token login (JWT / OAuth) di gerbang depan sehingga setiap microservice di belakang tidak perlu memvalidasi ulang dari nol.
- **Rate Limiting & Throttling**: Mencegah serangan spam/DDoS dan membatasi kuota request per pengguna.
- **SSL/TLS Termination**: Menangani enkripsi HTTPS dari publik.
- **API Composition / Aggregation**: Menggabungkan respons dari beberapa microservice menjadi 1 paket respons ke klien.
- **Caching, Logging, Distributed Tracing (Correlation ID), dan Circuit Breaking**.
- **Contoh Teknologi API Gateway**: **Kong**, **Amazon API Gateway**, **Apigee (Google Cloud)**, **Azure API Management**, **KrakenD**, **Envoy**.

### 2. Pola Backend For Frontend (BFF)
Terkadang, kebutuhan tampilan **Aplikasi Mobile (HP)** sangat berbeda dengan **Aplikasi Web Desktop** maupun **Smartwatch/TV**:
- Layar HP kecil dan koneksinya terbatas, sehingga butuh data ringkas.
- Layar Web Desktop lebar dan butuh tabel analitik lengkap dari banyak service.

Jika kita memaksakan hanya 1 API Gateway umum untuk semua jenis perangkat, kode gateway tersebut akan penuh dengan percabangan `if (device == mobile) ... else if (device == web)`.
Solusinya adalah pola **Backend For Frontend (BFF)** (diperkenalkan oleh Sam Newman): kita membuat **layanan perantara (BFF) terpisah khusus untuk masing-masing antarmuka frontend** (1 BFF khusus Mobile, 1 BFF khusus Web Desktop).

![backend-for-frontend](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-III/api-gateway/backend-for-frontend.png)

> *Tips: **GraphQL** sangat populer digunakan sebagai lapisan BFF karena memungkinkan frontend meminta field yang dibutuhkan saja!*

---

## 11. REST vs GraphQL vs gRPC

Memilih protokol komunikasi API yang tepat adalah pertanyaan wajib dalam setiap sesi System Design Interview.

### 1. REST (Representational State Transfer)
Gaya arsitektur API paling populer di dunia web (diperkenalkan oleh Roy Fielding tahun 2000) yang berpusat pada konsep **Resource (Sumber Daya, misal `/users`, `/products`)** yang dimanipulasi menggunakan metode standar **HTTP Verbs**.

- **HTTP Verbs Utama**:
  - `GET`: Membaca/mengambil data resource (Aman & Idempotent).
  - `POST`: Membuat resource baru atau memicu aksi pemrosesan.
  - `PUT`: Mengganti seluruh representasi resource secara utuh (Idempotent).
  - `PATCH`: Mengubah sebagian atribut dari resource.
  - `DELETE`: Menghapus resource (Idempotent).
- **Kelas HTTP Status Codes**:
  - `1xx`: Informasi sementara.
  - `2xx`: **Sukses** (`200 OK`, `201 Created`, `204 No Content`).
  - `3xx`: **Redirection / Pengalihan** (`301 Moved Permanently`, `302 Found`, `304 Not Modified`).
  - `4xx`: **Kesalahan dari sisi Client** (`400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `429 Too Many Requests`).
  - `5xx`: **Kesalahan dari sisi Server** (`500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`, `504 Gateway Timeout`).
- **Kelebihan REST**: Sangat sederhana, mudah dibaca manusia (JSON), didukung oleh semua browser/bahasa, dan memiliki dukungan **HTTP Caching** bawaan yang luar biasa.
- **Kekurangan REST**:
  - **Over-fetching**: Klien hanya butuh `id` dan `name`, tapi endpoint `GET /users/1` mengembalikan 50 kolom lengkap.
  - **Under-fetching & Multiple Round-Trips**: Untuk menampilkan halaman profil beserta daftar pesanan dan ulasan, klien harus memanggil 3 endpoint berbeda secara berurutan.

### 2. GraphQL
Bahasa query untuk API (dikembangkan oleh Facebook/Meta, 2015) yang memungkinkan klien **meminta struktur dan field data persis seperti yang dibutuhkan—tidak lebih dan tidak kurang—dalam 1 kali request tunggal**.

```graphql
# Contoh Query dari Klien (Hanya meminta id, name, dan city)
{
  getUser {
    id
    name
    city
  }
}
```

- **Kelebihan GraphQL**: Menghilangkan *over-fetching* dan *under-fetching*, menghemat kuota/bandwidth aplikasi mobile, memiliki skema bertipe kuat (*Strongly Typed Schema*).
- **Kekurangan GraphQL**:
  - Memindahkan kompleksitas ke sisi server.
  - Rawan terkena masalah **N+1 Query** pada resolver jika tidak memakai *DataLoader*.
  - Lebih sulit melakukan *HTTP Caching* standar karena semua query biasanya dikirim lewat `POST /graphql`.

### 3. gRPC (Google Remote Procedure Call)
Framework RPC modern berkinerja super tinggi buatan Google yang berjalan di atas **HTTP/2** dan menggunakan **Protocol Buffers (Protobuf)** sebagai format serialisasi data biner.

```protobuf
service HelloService {
  rpc SayHello (HelloRequest) returns (HelloResponse);
}
```

- **Kelebihan gRPC**:
  - **Performa & Kecepatan Luar Biasa**: Karena Protobuf dikompilasi menjadi format **biner** (bukan teks JSON yang gemuk), ukuran paketnya jauh lebih kecil dan proses serialisasinya hingga **5–10x lebih cepat dari JSON REST**!
  - **Code Generation Otomatis**: Dari satu file kontrak `.proto`, kita langsung bisa men-generate kode klien & server di Go, Java, Python, C++, Rust, maupun TypeScript.
  - **Mendukung Bi-directional Streaming** penuh berkat HTTP/2.
- **Kekurangan gRPC**: Format biner tidak bisa dibaca langsung oleh mata manusia saat *debugging*, dan dukungan langsung dari browser web masih terbatas (membutuhkan proxy *gRPC-Web*).

### Tabel Perbandingan Lengkap: REST vs GraphQL vs gRPC

| Parameter | REST | GraphQL | gRPC |
| :--- | :--- | :--- | :--- |
| **Format Data** | Teks (`JSON` / `XML`) | Teks (`JSON`) | **Biner (`Protocol Buffers`)** |
| **Coupling (Keterikatan)** | Rendah | Sedang | Tinggi (kontrak `.proto` ketat) |
| **Chattiness (Bolak-balik Request)** | Tinggi (sering butuh banyak call) | **Rendah** (1 query bisa ambil banyak relasi) | Sedang (mendukung *streaming*) |
| **Performa & Kecepatan** | Baik | Baik | **Sangat Cepat (Terbaik)** 🏆 |
| **Kemudahan HTTP Caching** | **Sangat Mudah (Bawaan)** 🏆 | Sulit (Butuh mekanisme kustom) | Sulit (Kustom) |
| **Code Generation** | Butuh tool tambahan (OpenAPI) | Baik | **Bawaan (Sangat Kuat)** 🏆 |
| **Rekomendasi Penggunaan Utama** | **Public API** & komunikasi standar Browser/Mobile ke Server. | **Backend For Frontend (BFF)** & aplikasi Mobile dengan relasi data kompleks. | **Komunikasi Internal Antar-Microservices** yang butuh latensi super rendah! |

---

## 12. Long Polling, WebSockets, dan Server-Sent Events (SSE)

Secara tradisional, HTTP bekerja searah: **Server hanya bisa menjawab jika ditanya terlebih dahulu oleh Client**. Lalu bagaimana cara membuat fitur *real-time* seperti **chat WhatsApp, pelacakan lokasi driver ojek online, atau notifikasi skor sepak bola langsung** di mana Server harus mengirim pembaruan ke Client kapan saja?

Ada **3 teknik utama** yang wajib kamu pahami:

### 1. HTTP Long Polling

![long-polling](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-III/long-polling-websockets-server-sent-events/long-polling.png)

- **Cara Kerja**:
  1. Client mengirim HTTP request ke Server: *"Ada pesan baru belum?"*
  2. Berbeda dengan polling biasa yang langsung menjawab *"Belum ada"*, pada **Long Polling**, Server **menahan koneksi tetap terbuka (menggantung)** sampai benar-benar ada data baru masuk atau batas waktu (*timeout*, misal 30 detik) habis.
  3. Begitu ada pesan baru, Server langsung membalas request tersebut.
  4. Segera setelah menerima balasan, Client langsung mengirim request baru lagi untuk menunggu pesan berikutnya.
- **Kelebihan**: Sangat mudah dibuat dan berjalan di semua browser lawas.
- **Kekurangan**: Tidak efisien untuk skala besar karena terus-menerus membuka-tutup koneksi HTTP baru setiap kali pesan terkirim.

### 2. WebSockets ⭐ *(Standar Utama untuk Komunikasi 2 Arah Real-Time)*

**WebSocket** menyediakan saluran komunikasi **Full-Duplex (Dua Arah Secara Bersamaan)** melalui satu koneksi TCP tunggal yang terus terbuka (*persistent connection*).

![websockets](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-III/long-polling-websockets-server-sent-events/websockets.png)

- **Cara Kerja**:
  1. Client mengirim HTTP request awal yang menyertakan header `Connection: Upgrade` dan `Upgrade: websocket` (**WebSocket Handshake**).
  2. Server menyetujui (`HTTP 101 Switching Protocols`), dan protokol berubah dari `http://` menjadi `ws://` (atau `wss://` yang terenkripsi TLS).
  3. Koneksi tetap terbuka! Kini baik **Client maupun Server bebas saling mengirim pesan kapan saja dengan latensi sangat rendah** tanpa overhead header HTTP yang besar.
- **Kapan Menggunakan WebSockets?** Aplikasi **Chatting (WhatsApp/Telegram)**, **Multiplayer Online Game**, **Kolaborasi Dokumen Real-time (Google Docs/Figma)**, **Live Location Tracking (Uber/Gojek)**.

### 3. Server-Sent Events (SSE)

**Server-Sent Events (SSE)** adalah standar HTTP yang memungkinkan **Server mendorong (*push*) aliran data secara terus-menerus ke Client secara Satu Arah (Unidirectional: Server → Client saja)**.

![server-sent-events](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-III/long-polling-websockets-server-sent-events/server-sent-events.png)

- **Cara Kerja**: Client membuka satu koneksi HTTP bertipe `text/event-stream`, lalu Server dapat mengirimkan *stream* event teks berkali-kali melalui koneksi tersebut.
- **Kelebihan**: Jauh lebih ringan dan sederhana dibanding WebSockets jika Client hanya butuh **mendengarkan** tanpa perlu mengirim banyak pesan balik; memiliki fitur *auto-reconnect* bawaan di browser dan berjalan mulus melewati firewall HTTP standar.
- **Kekurangan**: Hanya **satu arah** (Server ke Client) dan tidak mendukung pengiriman data biner mentah.
- **Kapan Menggunakan SSE?** **Streaming jawaban AI (seperti efek ketikan ChatGPT)**, **Live Ticker Harga Saham**, **Update Feed Notifikasi Sosial Media**, **Live Skor Pertandingan**.

### Tabel Ringkasan: Kapan Memilih Long Polling vs WebSockets vs SSE?

| Kriteria | HTTP Long Polling | Server-Sent Events (SSE) | WebSockets |
| :--- | :--- | :--- | :--- |
| **Arah Komunikasi** | Request-Response (Ditahan) | **Satu Arah** (*Server → Client*) | **Dua Arah Penuh (*Full-Duplex*)** |
| **Protokol** | HTTP Standar | HTTP (`text/event-stream`) | `ws://` atau `wss://` di atas TCP |
| **Format Data** | Teks / JSON | Hanya Teks (UTF-8) | **Teks maupun Biner** |
| **Beban Server & Latensi** | Tinggi (buka-tutup koneksi) | Rendah | **Sangat Rendah (Paling Cepat)** |
| **Contoh Kasus Terbaik** | Sistem legacy sederhana atau pengecekan status pembayaran pendek. | **Streaming teks AI (ChatGPT)**, notifikasi dashboard, harga saham live. | **Aplikasi Chat (WhatsApp)**, **Game Online**, **Lokasi Driver Gojek/Uber**. |
