# 🗄️ Bab II: Database, Konsistensi Data & Skalabilitas Penyimpanan

> **🎯 Tujuan Belajar Bab Ini:**
> Hampir setiap aplikasi adalah sistem pengolah data. Di bab ini kita mempelajari cara memilih database yang tepat (SQL vs NoSQL), cara mempercepat query (Indexing, Materialized View, Denormalisasi), menjaga integritas transaksi (ACID, BASE, CAP, PACELC, 2PC, Saga), serta menskalakan database ke jutaan pengguna per detik (Replikasi, Sharding, Consistent Hashing, Federation).

---

## Daftar Isi Bab II
1. [Database dan DBMS](#1-database-dan-dbms)
2. [Database SQL (Relasional)](#2-database-sql-relasional)
3. [Database NoSQL (Non-Relasional)](#3-database-nosql-non-relasional)
4. [Perbandingan SQL vs NoSQL](#4-perbandingan-sql-vs-nosql)
5. [Replikasi Database (Master-Slave & Master-Master)](#5-replikasi-database-database-replication)
6. [Database Indexes (Dense vs Sparse Index)](#6-database-indexes)
7. [Normalisasi dan Denormalisasi](#7-normalisasi-dan-denormalisasi)
8. [Model Konsistensi ACID dan BASE](#8-model-konsistensi-acid-dan-base)
9. [Teorema CAP (CAP Theorem)](#9-teorema-cap-cap-theorem)
10. [Teorema PACELC (PACELC Theorem)](#10-teorema-pacelc-pacelc-theorem)
11. [Transaksi Database (Database Transactions)](#11-transaksi-database-transactions)
12. [Transaksi Terdistribusi (2PC, 3PC, dan Saga Pattern)](#12-transaksi-terdistribusi-distributed-transactions)
13. [Sharding (Partisi Database Horizontal)](#13-sharding-partisi-database)
14. [Consistent Hashing (Hash Ring & Virtual Nodes)](#14-consistent-hashing)
15. [Database Federation (Partisi Fungsional)](#15-database-federation)

---

## 1. Database dan DBMS

- **Database (Basis Data)**: Kumpulan informasi atau data terorganisir yang disimpan secara elektronik di dalam sistem komputer.
- **DBMS (*Database Management System*)**: Perangkat lunak yang menjadi perantara antara pengguna/aplikasi dengan data di dalam disk. DBMS bertugas menerima query, mengoptimalkan pencarian, mengatur hak akses keamanan, memonitor performa, hingga melakukan *backup & recovery* (Contoh DBMS: PostgreSQL, MySQL, MongoDB, Redis).

### Komponen Dasar Database
1. **Schema (Skema)**: Cetak biru atau aturan struktur yang mendefinisikan bentuk data dan tipe data apa yang boleh masuk ke mana.
2. **Table (Tabel)**: Kumpulan data yang tersusun seperti lembar *spreadsheet* (baris dan kolom), mewakili satu entitas bisnis (misal tabel `users` atau `orders`).
3. **Column (Kolom / Atribut)**: Menyimpan satu jenis nilai data spesifik untuk setiap baris (misal `email VARCHAR`, `created_at TIMESTAMP`).
4. **Row (Baris / Record / Tuple)**: Satu entri data utuh di dalam tabel (misal data milik satu pelanggan bernama Budi).

![database-types](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-II/databases-and-dbms/database-types.png)

### Tantangan Mengelola Database Berskala Besar
- Menyerap lonjakan volume data yang masif (jutaan transaksi atau log sensor per detik).
- Menjaga keamanan data (*data security*) dan privasi (enkripsi data sensitif/PII).
- Memenuhi tuntutan akses *real-time* dengan latensi milidetik.
- Kepatuhan regulasi lokasi data (*Data Residency / Data Sovereignty*, misal aturan pemerintah bahwa data warga negara wajib disimpan di data center dalam negeri).

---

## 2. Database SQL (Relasional)

**Database SQL (Relasional / RDBMS)** menyimpan data dalam tabel-tabel yang memiliki **relasi yang telah ditentukan sebelumnya** menggunakan **Primary Key** (pengenal unik baris) dan **Foreign Key** (penghubung ke tabel lain). Database SQL mematuhi jaminan transaksi **ACID** secara ketat.

### Konsep Penting pada SQL Database

#### 1. Materialized Views
- **View Biasa**: Hanyalah *query* SQL yang disimpan sebagai nama alias; setiap kali dipanggil, database tetap menjalankan *query* berat dari awal.
- **Materialized View**: Hasil *query* kompleks (misal laporan penjualan bulanan hasil `JOIN` 5 tabel) **dihitung di awal (*pre-computed*) dan disimpan secara fisik di disk** sebagai tabel bayangan.
- **Keuntungan**: Membaca *Materialized View* berkali-kali lipat lebih cepat dibanding menjalankan ulang query `JOIN` dan agregasi (`SUM`, `COUNT`) pada jutaan baris data.

#### 2. Masalah Query N+1 (*N+1 Query Problem*) ⚠️ *(Wajib Dihindari!)*
Masalah klasik yang sangat sering terjadi saat developer menggunakan **ORM** (seperti Hibernate/JPA, Prisma, GORM) atau **GraphQL**:
- **Bagaimana terjadinya?** Misalnya kita ingin menampilkan 50 artikel beserta nama penulisnya:
  1. Aplikasi menjalankan **1 query utama**: `SELECT * FROM articles LIMIT 50;`
  2. Lalu di dalam perulangan (`for loop`), aplikasi menjalankan **50 query tambahan (N query)** untuk mengambil penulis tiap artikel: `SELECT * FROM users WHERE id = ?;` (dieksekusi 50 kali!).
  3. Total ada **1 + 50 = 51 query** bolak-balik ke database! Jika `N = 1000`, database langsung lumpuh!
- **Solusi Arsitek**:
  - Gunakan **Eager Loading / `JOIN FETCH`** dalam 1 query tunggal: `SELECT * FROM articles a JOIN users u ON a.author_id = u.id LIMIT 50;`
  - Atau gunakan **Batching / DataLoader** (`WHERE id IN (1, 2, 3, ..., 50)`).

### Kelebihan & Kekurangan Database SQL
- **Kelebihan**: Konsistensi data sangat kuat (ACID), mendukung query kompleks (`JOIN`, agregasi), skema terstruktur rapi dan tervalidasi.
- **Kekurangan**: Sulit melakukan *Horizontal Scaling* (umumnya mengandalkan *Vertical Scaling*), migrasi skema (`ALTER TABLE`) pada tabel ratusan juta baris membutuhkan kehati-hatian tinggi, biaya `JOIN` antar tabel raksasa bisa memperlambat performa.
- **Contoh**: **PostgreSQL**, **MySQL**, **MariaDB**, **Amazon Aurora**, **CockroachDB / Google Cloud Spanner** (NewSQL).

---

## 3. Database NoSQL (Non-Relasional)

**NoSQL** (*Not Only SQL*) mencakup semua jenis database yang tidak menggunakan tabel relasional tradisional. Data tidak dipaksa mengikuti skema kaku (*schemaless/dynamic schema*) dan dirancang sejak awal untuk mudah di-*scale out* secara horizontal mengikuti model **BASE**.

Berikut adalah **6 jenis utama Database NoSQL** beserta karakteristik dan contohnya:

### 1. Document Database
Menyimpan data dalam bentuk dokumen mirip objek **JSON / BSON**. Setiap dokumen bersifat mandiri dan bisa memiliki atribut yang berbeda-beda.
- **Kelebihan**: Sangat intuitif bagi programmer (mirip objek di kode), skema fleksibel, mudah di-*scale* horizontal.
- **Kekurangan**: Tidak dirancang untuk `JOIN` kompleks antar banyak koleksi; rawan duplikasi data.
- **Contoh**: **MongoDB**, **Amazon DocumentDB**, **CouchDB**, **Firestore**.

### 2. Key-Value Database
Bentuk NoSQL paling sederhana dan paling cepat: menyimpan data sebagai pasangan **Kunci (`Key`)** dan **Nilai (`Value`)** layaknya *Hash Map / Dictionary*.
- **Kelebihan**: Kecepatan baca/tulis luar biasa tinggi ($O(1)$ lookup), sangat ideal untuk **Caching, Session Management, Shopping Cart, Rate Limiting, dan Leaderboard**.
- **Kekurangan**: Hanya bisa mencari berdasarkan `Key`; tidak bisa melakukan *filter* kompleks pada isi `Value`.
- **Contoh**: **Redis**, **Memcached**, **Amazon DynamoDB**, **Aerospike**.

### 3. Graph Database
Menyimpan data menggunakan struktur graf yang terdiri dari **Nodes (Titik/Entitas)**, **Edges (Garis Hubungan)**, dan **Properties**.
- **Kelebihan**: Sangat cepat untuk menelusuri hubungan berlapis-lapis (misal: *"Cari teman dari temannya Budi yang menyukai restoran Jepang"*). Di SQL biasa, ini butuh berkali-kali `JOIN` yang lambat; di Graph DB, hubungan sudah tersimpan langsung secara fisik.
- **Kekurangan**: Bahasa query belum seragam (Cypher, Gremlin), lebih kompleks untuk kasus CRUD biasa.
- **Use Cases**: **Jejaring Sosial (Mutual Friends)**, **Mesin Rekomendasi**, **Deteksi Penipuan Finansial (*Fraud Detection*)**, Pemetaan Jaringan.
- **Contoh**: **Neo4j**, **Amazon Neptune**, **ArangoDB**, **JanusGraph**.

### 4. Time-Series Database (TSDB)
Dioptimalkan khusus untuk data yang selalu memiliki **stempel waktu (*timestamp*)** dan terus bertambah setiap detik (*append-only*).
- **Kelebihan**: Kecepatan tulis (*ingestion*) sangat tinggi, kompresi data waktu sangat efisien, mendukung query agregasi waktu otomatis (misal rata-rata per 5 menit).
- **Use Cases**: **Sensor IoT**, **Monitoring Server & Aplikasi (CPU/RAM metrics)**, **Grafik Harga Saham/Kripto**.
- **Contoh**: **InfluxDB**, **Prometheus**, **TimescaleDB**, **Apache Druid**.

### 5. Wide-Column Store (Column-Family)
Menyimpan data dalam *column families* yang terdistribusi, bukan baris-dan-kolom kaku. Dirancang untuk menangani volume tulis raksasa hingga skala **Petabyte** tanpa *Single Point of Failure*.
- **Kelebihan**: Skalabilitas horizontal luar biasa tinggi, kecepatan tulis (*write throughput*) sangat kencang, cocok untuk *real-time big data*.
- **Kekurangan**: Pola query harus dirancang sejak awal (tidak fleksibel untuk query *ad-hoc* sembarangan).
- **Use Cases**: Menyimpan riwayat pesan chat miliaran pengguna (Discord/WhatsApp), log aktivitas, analitik skala besar.
- **Contoh**: **Apache Cassandra**, **ScyllaDB**, **Google Cloud Bigtable**, **Apache HBase**.

### 6. Multi-Model Database
Menggabungkan beberapa model database (Relasional, Dokumen, Graf, Key-Value) ke dalam satu mesin backend terpadu.
- **Contoh**: **ArangoDB**, **Azure Cosmos DB**, **Couchbase**.

---

## 4. Perbandingan SQL vs NoSQL

| Dimensi | Database SQL (Relasional) | Database NoSQL (Non-Relasional) |
| :--- | :--- | :--- |
| **Model Penyimpanan** | Tabel dengan Baris (*Rows*) dan Kolom (*Columns*). | Dokumen JSON, Key-Value, Graph, Wide-Column, atau Time-Series. |
| **Skema Data** | **Kaku (*Strict/Fixed Schema*)**: Kolom harus didefinisikan di awal; perubahan butuh *migration*. | **Dinamis (*Flexible/Schemaless*)**: Field baru bisa ditambahkan kapan saja tanpa migrasi tabel. |
| **Bahasa Query** | Standar universal menggunakan **SQL** (sangat kuat untuk `JOIN` & analitik). | Berbeda-beda tergantung produk (fokus pada koleksi/key, minim dukungan `JOIN`). |
| **Skalabilitas** | Umumnya **Vertikal** (menambah spesifikasi mesin). Skalabilitas horizontal membutuhkan *sharding* manual yang rumit. | Dirancang untuk **Horizontal Scaling** otomatis sejak awal di atas banyak mesin murah. |
| **Jaminan Transaksi** | **ACID Compliant** (Integritas dan konsistensi data terjamin 100%). | Umumnya **BASE** (Mengorbankan konsistensi instan demi performa dan ketersediaan tinggi, meski beberapa kini mendukung transaksi terbatas). |

### 🎯 Panduan Arsitek: Kapan Memilih SQL vs NoSQL?
- **Pilih SQL (PostgreSQL / MySQL) sebagai DEFAULT ketika:**
  - Data memiliki relasi yang jelas dan membutuhkan integritas ketat.
  - Aplikasi menangani **transaksi keuangan, saldo dompet digital, akuntansi, atau inventori stok barang** (Wajib ACID!).
  - Membutuhkan query kompleks dan `JOIN` antar banyak entitas.
- **Pilih NoSQL ketika:**
  - Struktur data sangat dinamis/tidak terstruktur (misal katalog produk dengan ribuan atribut berbeda untuk pakaian vs elektronik).
  - Laju penulisan (*write throughput*) sangat ekstrem (puluhan ribu pesan chat atau log sensor IoT per detik → pilih **Cassandra/ScyllaDB/InfluxDB**).
  - Membutuhkan akses *key-value* sub-milidetik untuk *caching/session* (pilih **Redis**).

---

## 5. Replikasi Database (Database Replication)

**Replikasi** adalah proses menyalin dan menjaga sinkronisasi data di beberapa server database yang berbeda untuk meningkatkan **ketersediaan (*availability*)**, **keandalan (*fault-tolerance*)**, dan **kecepatan baca (*read performance*)**.

### 1. Master-Slave (Leader-Follower / Primary-Replica) Replication

Satu node database bertindak sebagai **Master (Primary)** yang menerima seluruh operasi **Tulis (`WRITE`: Insert, Update, Delete)**, lalu menyalin perubahan tersebut ke satu atau lebih **Slave (Read Replicas)** yang hanya melayani operasi **Baca (`READ`: Select)**.

![master-slave-replication](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-II/database-replication/master-slave-replication.png)

- **Kelebihan**:
  - Sangat efektif untuk aplikasi **Read-Heavy** (banyak baca, sedikit tulis): kita bisa menambah banyak *Read Replica* agar Master tidak kewalahan.
  - Proses *backup* database bisa dilakukan di node *Slave* tanpa mengganggu performa *Master*.
- **Kekurangan**:
  - Jika *Master* mati, sistem tidak bisa menerima penulisan baru (*read-only mode*) sampai salah satu *Slave* dipromosikan menjadi *Master* baru (*Failover*).
  - Semakin banyak *Read Replica*, semakin besar risiko terjadinya **Replication Lag** (jeda waktu beberapa milidetik/detik di mana data yang baru ditulis di Master belum sempat sampai ke Slave).

### 2. Master-Master (Multi-Leader / Active-Active) Replication

Dua atau lebih node database sama-sama bertindak sebagai **Master** yang bisa menerima operasi **Baca (`READ`) sekaligus Tulis (`WRITE`)**, serta saling menyinkronkan data satu sama lain.

![master-master-replication](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-II/database-replication/master-master-replication.png)

- **Kelebihan**:
  - Membagi beban penulisan (*write load*) ke beberapa node.
  - Jika salah satu Master mati, Master lainnya langsung melayani baca dan tulis tanpa jeda *downtime*.
- **Kekurangan**:
  - **Masalah Konflik Penulisan (*Write Conflict*)**: Bagaimana jika dua pengguna mengubah baris data yang sama secara bersamaan di Master A dan Master B? Membutuhkan mekanisme *Conflict Resolution* yang rumit (seperti *Last-Write-Wins / LWW* atau *CRDT*).

### Synchronous vs Asynchronous Replication

| Jenis Replikasi | Cara Kerja | Kelebihan | Kekurangan |
| :--- | :--- | :--- | :--- |
| **Synchronous (Sinkron)** | Saat aplikasi menulis ke Master, Master **menunggu sampai Slave selesai menyalin data** sebelum memberi respons "Sukses" ke aplikasi. | **Zero Data Loss**: Data di Master dan Slave dijamin selalu 100% sama setiap saat. | **Latensi Tulis Tinggi**: Jika jaringan ke Slave lambat atau Slave mati, proses tulis di Master ikut terhambat. |
| **Asynchronous (Asinkron)** | Master langsung memberi respons "Sukses" ke aplikasi begitu data tersimpan di Master, lalu menyalinnya ke Slave di belakang layar. | **Sangat Cepat**: Performa tulis di Master tidak terpengaruh oleh jarak atau kondisi Slave. | Risiko **Replication Lag** (baca data lama di Slave) dan potensi kehilangan sedikit data terbaru jika Master tiba-tiba terbakar sebelum sempat menyalin ke Slave. |

---

## 6. Database Indexes

**Index** pada database adalah struktur data khusus (umumnya berbentuk pohon **B-Tree** atau **B+ Tree**) yang berfungsi mempercepat operasi pencarian data (`SELECT ... WHERE ...`) tanpa harus memeriksa setiap baris satu per satu dari awal sampai akhir tabel (*Full Table Scan*).

![indexes](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-II/indexes/indexes.png)

> **💡 Analogi Pemula:**
> Bayangkan kamu punya buku ensiklopedia setebal 1.000 halaman dan ingin mencari pembahasan tentang **"Fotosintesis"**:
> - **Tanpa Index (*Full Table Scan*)**: Kamu harus membuka halaman 1, halaman 2, halaman 3... satu per satu sampai halaman 1.000. Sangat lambat ($O(N)$)!
> - **Dengan Index**: Kamu langsung membuka halaman **Indeks Abjad** di bagian belakang buku, mencari huruf **F → Fotosintesis → Halaman 412**, lalu langsung membuka halaman 412 ($O(\log N)$)!

### Trade-off Penggunaan Index (Wajib Disebutkan Saat Interview!)
- **Keuntungan**: Mempercepat operasi **Baca (`SELECT`, `WHERE`, `JOIN`, `ORDER BY`)** secara drastis.
- **Harga yang Harus Dibayar (*Trade-off*)**:
  1. **Memakan Ruang Penyimpanan Tambahan** (RAM dan Disk) untuk menyimpan struktur pohon index.
  2. **Memperlambat Operasi Tulis (`INSERT`, `UPDATE`, `DELETE`)** karena setiap kali data di tabel berubah, database juga wajib memperbarui dan menyeimbangkan ulang pohon index-nya!
  - *Aturan Praktis*: Pasang index pada kolom **Foreign Key**, kolom pencarian unik (**Email, Username, Kode Transaksi**), dan kolom yang sering masuk klausa `WHERE`/`ORDER BY`. Jangan mengindeks semua kolom secara membabi buta!

### Dense Index vs Sparse Index

#### 1. Dense Index (Indeks Padat)
Membuat catatan index (berisi nilai kunci pencarian + pointer alamat baris) untuk **setiap baris data** yang ada di dalam tabel.

![dense-index](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-II/indexes/dense-index.png)

- **Kelebihan**: Pencarian data sangat cepat (cukup *binary search* pada index langsung ketemu alamat barisnya); data di tabel utama tidak wajib berurutan.
- **Kekurangan**: Memakan memori/disk lebih besar dan beban pemeliharaan saat *write* lebih berat.

#### 2. Sparse Index (Indeks Jarang)
Hanya membuat catatan index untuk **sebagian baris data saja** (misalnya hanya 1 entri index untuk setiap blok/halaman data yang sudah terurut).

![sparse-index](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-II/indexes/sparse-index.png)

- **Kelebihan**: Sangat hemat memori dan proses *insert/update/delete* lebih ringan.
- **Kekurangan**: Pencarian sedikit lebih lambat dibanding Dense Index (setelah menemukan blok terdekat di index, database harus memindai beberapa baris di dalam blok tersebut), serta mewajibkan data fisik di tabel tersimpan secara berurutan (*sorted*).

---

## 7. Normalisasi dan Denormalisasi

### Istilah Kunci (Keys) dalam Database Relasional
- **Primary Key (PK)**: Satu kolom (atau gabungan kolom) yang secara unik mengidentifikasi setiap baris dalam tabel (tidak boleh `NULL` dan tidak boleh kembar).
- **Composite Key**: *Primary Key* yang terbentuk dari gabungan dua atau lebih kolom.
- **Super Key**: Himpunan satu atau lebih kolom yang jika digabungkan dapat mengidentifikasi baris secara unik.
- **Candidate Key**: *Super Key* paling minimal yang memenuhi syarat untuk dipilih menjadi *Primary Key* (misal: `id`, `nik`, atau `email`).
- **Alternate Key**: *Candidate Key* yang tidak terpilih menjadi *Primary Key*.
- **Foreign Key (FK)**: Kolom pada suatu tabel yang merujuk pada *Primary Key* di tabel lain untuk membentuk relasi.
- **Surrogate Key**: Kunci buatan yang di-generate oleh sistem (seperti `AUTO_INCREMENT` angka atau `UUID`) ketika tidak ada kolom alami yang cocok menjadi Primary Key.

### Jenis Ketergantungan (*Dependencies*)
- **Functional Dependency ($X \to Y$)**: Nilai kolom $X$ menentukan nilai kolom $Y$ (misal `id_karyawan` menentukan `nama_karyawan`).
- **Partial Dependency**: Terjadi pada *Composite Primary Key* $(A, B)$ ketika sebuah kolom non-kunci hanya bergantung pada sebagian kunci saja (misal hanya bergantung pada $A$, tidak butuh $B$).
- **Transitive Dependency**: Terjadi ketika kolom non-kunci $Y$ bergantung pada kolom non-kunci lain $Z$ ($X \to Y \to Z$).

### 3 Anomali Database (Akibat Tabel Tidak Dinormalisasi)
Perhatikan tabel karyawan yang **belum dinormalisasi** berikut:

| ID | Nama | Jabatan | Tim | Lokasi Gedung Tim |
| :---: | :--- | :--- | :---: | :--- |
| 1 | Peter | Software Engineer | A | Gedung Utara |
| 2 | Brian | DevOps Engineer | B | Gedung Selatan |
| 3 | Hailey | Product Manager | C | Gedung Timur |
| 4 | Hailey | Product Manager | A | Gedung Utara |

Jika semua data digabung dalam satu tabel datar seperti di atas, akan muncul 3 masalah (**Anomali**):
1. **Insertion Anomaly (Anomali Penyisipan)**: Kita tidak bisa menambahkan data "Tim D yang berlokasi di Gedung Barat" ke dalam database sebelum ada minimal 1 karyawan yang masuk ke Tim D!
2. **Update Anomaly (Anomali Pembaruan)**: Jika "Tim A" pindah kantor ke "Gedung Pusat", kita harus mencari dan mengubah semua baris karyawan yang berada di Tim A. Jika ada 1 baris saja yang terlewat, data menjadi tidak konsisten!
3. **Deletion Anomaly (Anomali Penghapusan)**: Jika Brian (ID 2) mengundurkan diri dan barisnya dihapus, maka informasi bahwa "Tim B berlokasi di Gedung Selatan" ikut terhapus selamanya dari database!

### Normalisasi & Bentuk-Bentuk Normal (*Normal Forms*)

**Normalisasi** adalah proses mengorganisir tabel dan kolom dalam database relasional untuk **meminimalkan duplikasi data (redundansi)** dan menghilangkan anomali di atas.

| Bentuk Normal | Syarat & Aturan Utama |
| :--- | :--- |
| **1NF (*First Normal Form*)** | 1. Setiap kolom hanya boleh menyimpan **1 nilai tunggal (*atomic*)**—tidak boleh ada array/daftar nilai berulang dalam 1 sel.<br>2. Setiap baris diidentifikasi secara unik oleh *Primary Key*.<br>3. Tidak boleh mencampur tipe data berbeda dalam satu kolom. |
| **2NF (*Second Normal Form*)** | 1. Sudah memenuhi **1NF**.<br>2. **Tidak boleh ada *Partial Dependency***: Semua kolom non-kunci harus bergantung sepenuhnya pada keseluruhan *Primary Key* (jika PK berupa gabungan kolom, pisahkan data yang hanya bergantung pada sebagian kunci ke tabel tersendiri). |
| **3NF (*Third Normal Form*)** | 1. Sudah memenuhi **2NF**.<br>2. **Tidak boleh ada *Transitive Dependency***: Kolom non-kunci tidak boleh bergantung pada kolom non-kunci lainnya (pisahkan ke tabel referensi tersendiri). *Catatan: Database yang mencapai 3NF sudah dianggap ternormalisasi dengan baik di industri.* |
| **BCNF (*Boyce-Codd Normal Form / 3.5NF*)** | Versi lebih ketat dari 3NF: Untuk setiap ketergantungan fungsional $X \to Y$, $X$ **wajib merupakan sebuah Super Key**. |

### Denormalisasi

**Denormalisasi** adalah teknik optimasi database di mana kita **secara sengaja menambahkan data redundan (duplikat)** ke satu atau beberapa tabel untuk **menghindari operasi `JOIN` yang mahal** dan mempercepat pembacaan (*read performance*).

> **🎯 Perbandingan Trade-off Normalisasi vs Denormalisasi:**
> - **Normalisasi**: Hemat ruang penyimpanan, konsistensi data terjaga, `INSERT/UPDATE` cepat, tetapi **`SELECT` lebih lambat** karena harus melakukan banyak `JOIN`. Cocok untuk sistem **OLTP (*Online Transaction Processing*)** yang banyak melakukan tulis/update.
> - **Denormalisasi**: **`SELECT` (pembacaan) sangat cepat** dan query sederhana tanpa `JOIN`, tetapi memakan ruang lebih besar dan `UPDATE` lebih lambat karena harus mengubah data di beberapa tempat sekaligus. Sangat cocok untuk sistem **Read-Heavy, Pelaporan, atau Data Warehouse (OLAP)** serta database yang sudah di-*sharding* lintas server.

---

## 8. Model Konsistensi ACID dan BASE

### ACID (Standar Database Relasional / SQL)
Untuk menjaga keutuhan data dalam pemrosesan transaksi (seperti transfer uang di bank), database relasional mematuhi 4 sifat **ACID**:

1. **A — Atomicity (Atomisitas / "Semua atau Tidak Sama Sekali")**:
   - Seluruh operasi di dalam satu transaksi harus berhasil semuanya. Jika ada 1 langkah saja yang gagal di tengah jalan, seluruh perubahan yang sudah telanjur dilakukan wajib dibatalkan (**Rollback**) kembali ke kondisi awal.
   - *Contoh*: Saat Budi mentransfer Rp100.000 ke Siti, ada 2 operasi: (1) Potong saldo Budi Rp100.000, dan (2) Tambah saldo Siti Rp100.000. Jika langkah ke-2 gagal karena error server, maka potongan saldo Budi di langkah ke-1 wajib dikembalikan!
2. **C — Consistency (Konsistensi Struktur & Aturan)**:
   - Setiap transaksi harus membawa database dari satu kondisi valid ke kondisi valid lainnya sesuai seluruh aturan skema (*constraints, foreign keys, triggers*). Misalnya aturan "Saldo tidak boleh negatif".
3. **I — Isolation (Isolasi Antar Transaksi Bersamaan)**:
   - Jika ada banyak transaksi yang berjalan secara bersamaan (*concurrent*), mereka tidak boleh saling mengacaukan. Hasil akhirnya harus sama seolah-olah transaksi-transaksi tersebut dijalankan mengantre satu per satu.
4. **D — Durability (Daya Tahan Permanen)**:
   - Begitu sebuah transaksi dinyatakan berhasil (**Committed**), perubahannya telah ditulis ke penyimpanan non-volatil (Disk / *Write-Ahead Log*) dan **tidak akan hilang** meskipun detik berikutnya listrik data center padam!

### BASE (Standar Database NoSQL Terdistribusi)
Untuk mencapai skalabilitas ekstrem dan ketersediaan tinggi di ratusan server, banyak sistem NoSQL melonggarkan aturan ACID dan mengadopsi model **BASE**:

1. **BA — Basic Availability (Ketersediaan Dasar)**:
   - Database menjamin akan selalu merespons setiap permintaan (tetap hidup dan bisa diakses setiap saat) meskipun ada sebagian node yang sedang gangguan.
2. **S — Soft-state (Kondisi Lunak)**:
   - Kondisi data di dalam sistem dapat berubah seiring waktu (karena proses sinkronisasi latar belakang antar replika) meskipun sedang tidak ada input baru dari pengguna.
3. **E — Eventual Consistency (Konsistensi pada Akhirnya)**:
   - Saat data baru ditulis ke satu node, node lain mungkin belum langsung memiliki data terbaru tersebut dalam beberapa milidetik pertama. Namun sistem menjamin bahwa **pada akhirnya (*eventually*)** semua node replika akan menerima pembaruan tersebut dan menjadi konsisten.

> **💡 Analogi Pemula ACID vs BASE:**
> - **ACID** seperti **Transfer Saldo Bank**: Saat uang berpindah, angka di rekening pengirim dan penerima wajib langsung tepat detik itu juga. Tidak boleh uangnya "hilang sebentar di udara".
> - **BASE (Eventual Consistency)** seperti **Jumlah Like / View di Postingan Instagram atau YouTube**: Kalau kamu melihat jumlah *like* artis 1.000.005 sementara temanmu di kota lain melihat 1.000.002, lalu 3 detik kemudian angkanya sama-sama menjadi 1.000.010, tidak ada yang dirugikan! Sebagai gantinya, Instagram bisa melayani ratusan juta pengguna tanpa lemot.

---

## 9. Teorema CAP (CAP Theorem)

**Teorema CAP** (diperkenalkan oleh Eric Brewer) menyatakan bahwa dalam sebuah **sistem data terdistribusi**, kita hanya bisa menjamin maksimal **2 dari 3** karakteristik berikut pada saat yang bersamaan:

![cap-theorem](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-II/cap-theorem/cap-theorem.png)

1. **C — Consistency (Konsistensi)**:
   - Setiap pembacaan data di node mana pun selalu mengembalikan data paling baru yang terakhir ditulis (atau mengembalikan error). Semua klien melihat data yang persis sama pada waktu yang sama.
2. **A — Availability (Ketersediaan)**:
   - Setiap permintaan (*request*) ke node yang tidak mati pasti mendapatkan respons sukses (bukan pesan error/penolakan), meskipun tanpa jaminan bahwa data tersebut adalah versi paling baru.
3. **P — Partition Tolerance (Toleransi Partisi Jaringan)**:
   - Sistem tetap terus beroperasi meskipun kabel jaringan antar node terputus (*network partition*) sehingga node-node tidak bisa saling berkomunikasi.

### Mengapa di Dunia Nyata Kita Hanya Bisa Memilih CP atau AP?
Di dunia nyata, **jaringan komputer pasti suatu saat bisa mengalami gangguan atau putus (*Network Partition*)**. Kita tidak bisa mencegah kabel bawah laut putus atau switch jaringan macet. Oleh karena itu, setiap sistem terdistribusi **WAJIB mendukung P (Partition Tolerance)**.

Ketika jaringan antar Server A dan Server B terputus (*Partition terjadi*), sang Arsitek harus mengambil keputusan sulit:
- **Pilih CP (*Consistency + Partition Tolerance*)**:
  - Demi menjaga data tetap konsisten dan tidak terjadi konflik, sistem memilih **menolak permintaan tulis/baca pada node yang terisolasi** sampai jaringan pulih kembali (mengorbankan *Availability*).
  - **Kapan dipilih?** Sistem perbankan, pembayaran, pemesanan kursi bioskop/tiket pesawat, stok gudang e-commerce.
  - **Contoh Database CP**: MongoDB (konfigurasi default primary), Apache HBase, Redis Cluster / Zookeeper / etcd.
- **Pilih AP (*Availability + Partition Tolerance*)**:
  - Demi menjaga layanan tetap hidup, semua node tetap menerima baca dan tulis meskipun kabel antar node sedang putus (mengorbankan *Consistency* sementara waktu karena node yang terpisah bisa mengembalikan data versi lama). Saat jaringan pulih, data akan disinkronkan kembali.
  - **Kapan dipilih?** Feed sosial media, katalog produk, keranjang belanja Amazon, metrik analitik.
  - **Contoh Database AP**: Apache Cassandra, ScyllaDB, Amazon DynamoDB, CouchDB.
- **Bagaimana dengan CA (*Consistency + Availability*)?**
  - Hanya mungkin terjadi jika **tidak ada partisi jaringan**, yaitu pada database **Single-Node (1 server tunggal)** seperti PostgreSQL/MySQL standar tanpa klaster terdistribusi.

---

## 10. Teorema PACELC (PACELC Theorem)

**Teorema PACELC** (diperkenalkan oleh Daniel J. Abadi) menyempurnakan Teorema CAP. Mengapa? Karena Teorema CAP hanya menjelaskan apa yang terjadi **saat jaringan sedang putus (Partition)**, padahal 99,9% waktu sistem berjalan dalam keadaan **normal tanpa partisi jaringan**!

![pacelc-theorem](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-II/pacelc-theorem/pacelc-theorem.png)

Rumusan **PACELC** berbunyi:
- **PAC**: **Jika ada Partisi Jaringan (P)**, bagaimana sistem memilih antara **Availability (A)** dan **Consistency (C)**?
- **ELC**: **Else (E — Jika jaringan normal/tidak ada partisi)**, bagaimana sistem memilih antara **Latency / Kecepatan (L)** dan **Consistency (C)**?

> **💡 Penjelasan Intuitif PACELC:**
> Bayangkan jaringan sedang normal (tidak putus). Saat ada data baru masuk ke Server Utama:
> - Apakah kita langsung membalas ke pengguna dalam **2 milidetik (Pilih L = Low Latency)** sebelum menyalin data ke 3 server replika lainnya?
> - Ataukah kita menyuruh pengguna menunggu **50 milidetik (Pilih C = Consistency)** sampai ke-3 server replika selesai menyalin data secara sinkron?
> Itulah *trade-off* **L vs C** saat kondisi normal!

### Klasifikasi Database Berdasarkan PACELC
- **PC/EC** (*Partition → Consistency; Else → Consistency*): Selalu mengutamakan konsistensi baik saat jaringan putus maupun saat jaringan normal (Contoh: **Google Cloud Spanner, CockroachDB, VoltDB, HBase**).
- **PA/EL** (*Partition → Availability; Else → Latency*): Saat jaringan putus memilih tetap hidup (A), dan saat jaringan normal memilih respons secepat mungkin (L) dengan replikasi asinkron (Contoh: **Apache Cassandra, Amazon DynamoDB, Cosmos DB mode Eventual**).
- **PA/EC** (*Partition → Availability; Else → Consistency*): Contoh: **MongoDB** (tergantung konfigurasi *write/read concern*).

---

## 11. Transaksi Database (Transactions)

**Transaksi Database** adalah sekumpulan operasi baca/tulis database yang diperlakukan sebagai **satu unit pekerjaan tunggal (*single unit of work*)**. Semuanya harus berhasil bersama, atau gagal bersama.

### 6 Siklus Status Transaksi (*Transaction States*)

![transaction-states](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-II/transactions/transaction-states.png)

1. **Active**: Status awal ketika transaksi sedang dieksekusi dan menjalankan perintah baca/tulis di memori.
2. **Partially Committed**: Perintah terakhir dalam transaksi selesai dijalankan, namun perubahan masih berada di *buffer* memori dan sedang menunggu proses penulisan permanen (*flush*) ke disk.
3. **Committed**: Seluruh pemeriksaan lolos dan perubahan telah tertulis permanen di disk. Transaksi dinyatakan berhasil secara sah!
4. **Failed**: Terjadi kesalahan (misal melanggar *constraint*, saldo kurang, atau listrik mati saat status *Active/Partially Committed*) sehingga transaksi tidak bisa dilanjutkan.
5. **Aborted**: Setelah masuk status *Failed*, *Recovery Manager* database melakukan **Rollback** (membatalkan seluruh perubahan untuk mengembalikan database ke kondisi semula sebelum transaksi dimulai). Setelah *Aborted*, sistem dapat memilih untuk mengulang (*restart*) atau menghentikan (*kill*) transaksi.
6. **Terminated**: Siklus hidup transaksi resmi berakhir (baik setelah *Committed* maupun setelah selesai *Aborted*), dan sistem siap melayani transaksi baru.

---

## 12. Transaksi Terdistribusi (Distributed Transactions)

Ketika arsitektur kita menggunakan **Microservices** (di mana Layanan Order, Layanan Pembayaran, dan Layanan Gudang masing-masing punya database sendiri yang terpisah), bagaimana cara memastikan bahwa saat pelanggan membeli barang, stok berkurang DAN saldo terpotong secara konsisten di 3 database berbeda? Kita tidak bisa memakai `BEGIN ... COMMIT` SQL biasa!

Berikut adalah **3 solusi utama Transaksi Terdistribusi**:

### 1. Two-Phase Commit (2PC)

Protokol algoritma terdistribusi yang menggunakan satu **Coordinator Node** untuk memimpin proses *commit* atau *abort* di seluruh database peserta (*Participant Nodes*) melalui **2 Tahap**:

![two-phase-commit](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-II/distributed-transactions/two-phase-commit.png)

1. **Tahap 1 — Prepare Phase (Pemungutan Suara)**:
   - Coordinator bertanya ke semua node database peserta: *"Apakah kalian semua siap melakukan commit transaksi ini?"*
   - Setiap node memeriksa aturan dan mengunci (*lock*) baris data terkait, lalu menjawab `YES (Prepared)` atau `NO (Abort)`.
2. **Tahap 2 — Commit / Abort Phase (Keputusan Akhir)**:
   - **Jika SEMUA node menjawab `YES`**: Coordinator mengirim perintah `COMMIT` ke semua node.
   - **Jika ADA SATU SAJA node yang menjawab `NO` (atau timeout)**: Coordinator mengirim perintah `ROLLBACK (ABORT)` ke seluruh node agar membatalkan transaksi.

**Kelemahan Besar 2PC**:
- **Bersifat Blocking (Mengunci Sumber Daya)**: Selama menunggu jawaban dari node lain, baris database tetap terkunci (*locked*) sehingga pengguna lain tidak bisa mengaksesnya (latensi tinggi).
- **Single Point of Failure pada Coordinator**: Jika Coordinator tiba-tiba mati setelah mengirim pesan *Prepare*, seluruh node peserta akan terkatung-katung (*blocked*) menunggu aba-aba selanjutnya!

### 2. Three-Phase Commit (3PC)

Pengembangan dari 2PC yang memecah tahap kedua menjadi dua bagian untuk mengatasi masalah *blocking* tak berujung jika Coordinator mati:

![three-phase-commit](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-II/distributed-transactions/three-phase-commit.png)

1. **Prepare Phase**: Sama seperti 2PC.
2. **Pre-Commit Phase**: Coordinator mengirim pesan `Pre-Commit` untuk memberi tahu semua peserta bahwa seluruh node sudah bulat menjawab `YES` di tahap pertama.
3. **Commit Phase**: Setelah semua peserta mengakui pesan `Pre-Commit`, barulah eksekusi `Commit` dilakukan.
- **Keuntungan**: Setiap tahap memiliki batas waktu (*timeout*) yang jelas; jika sebuah node berada di tahap *Pre-Commit*, dijamin 100% bahwa seluruh node lain sudah menyelesaikan tahap *Prepare*, sehingga tidak menggantung selamanya.

### 3. Saga Pattern ⭐ *(Standar Emas Microservices Modern)*

Karena 2PC dan 3PC bersifat sinkron, lambat, dan mengunci lintas database, sistem *microservices* modern hampir selalu memilih **Saga Pattern**.

**Saga** adalah rangkaian **transaksi lokal (*local transactions*)** di masing-masing service. Setiap transaksi lokal mengubah database miliknya sendiri, lalu mengirim pesan/*event* untuk memicu transaksi lokal di service berikutnya.

![sagas](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-II/distributed-transactions/sagas.png)

**Bagaimana jika terjadi kegagalan di langkah ke-3?**
Karena langkah ke-1 dan ke-2 sudah telanjur di-*commit* di database masing-masing, Saga akan menjalankan **Compensating Transactions (Transaksi Kompensasi / Pembatalan)** secara mundur untuk menetralkan perubahan sebelumnya!
- *Contoh*: `Order Dibuat` → `Stok Dikurangi 1` → `Pembayaran Gagal (Saldo Kurang!)` → Jalankan Kompensasi: **`Kembalikan Stok +1`** → **`Ubah Status Order menjadi Dibatalkan`**.

#### 2 Cara Mengoordinasikan Saga:
1. **Choreography (Tanpa Pemimpin Pusat)**: Setiap service mempublikasikan *domain event* ke Message Broker (misal Kafka/RabbitMQ), dan service lain mendengarkan *event* tersebut untuk bekerja secara mandiri. Cocok untuk alur sederhana (2–4 service), tapi berisiko memicu ketergantungan melingkar (*cyclic dependency*) jika terlalu banyak service.
2. **Orchestration (Dengan Pemimpin Pusat / Orchestrator)**: Ada satu service khusus (*Saga Orchestrator*, misal menggunakan Temporal.io atau AWS Step Functions) yang memberi perintah satu per satu ke tiap service peserta dan memicu kompensasi jika ada yang gagal. Jauh lebih rapi dan mudah di-*debug* untuk alur bisnis kompleks!

---

## 13. Sharding (Partisi Database)

Ketika sebuah tabel database sudah berisi miliaran baris (misal puluhan Terabyte), 1 server database fisik tidak akan sanggup lagi menampungnya. Kita harus memecah database tersebut menjadi potongan-potongan yang lebih kecil (**Data Partitioning**).

### Metode Partisi Data
1. **Vertical Partitioning (Partisi Vertikal — Berdasarkan Kolom)**:
   - Memecah satu tabel lebar yang memiliki banyak kolom menjadi beberapa tabel lebih ramping di server terpisah. Contoh: Kolom profil singkat (`id`, `username`, `email`) disimpan di Tabel A yang cepat, sedangkan kolom berukuran besar (`bio_panjang`, `foto_ktp_blob`) dipisah ke Tabel B.
2. **Horizontal Partitioning / Sharding (Partisi Horizontal — Berdasarkan Baris)**:
   - Memecah baris-baris dari satu tabel raksasa ke dalam banyak tabel/server database berbeda yang disebut **Shards**. Setiap *Shard* memiliki skema dan kolom yang persis sama, tetapi menyimpan **potongan baris data yang berbeda** dan independen.

![sharding](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-II/sharding/sharding.png)

### 4 Kriteria Pembagian Shard (*Partitioning Criteria*)
1. **Hash-Based Sharding**: Memasukkan *Partition Key* (misal `user_id`) ke dalam fungsi hash untuk menentukan nomor shard tujuan. Menyebarkan data secara merata, tapi sulit saat menambah jumlah server baru jika tidak memakai *Consistent Hashing*.
2. **Range-Based Sharding**: Membagi baris berdasarkan rentang nilai yang berurutan (misal: User dengan ID `1 – 1.000.000` di Shard 1, ID `1.000.001 – 2.000.000` di Shard 2; atau berdasarkan bulan transaksi). Mudah diimplementasikan, tetapi rawan terjadi **Hotspot** (misal Shard bulan berjalan menerima 100% trafik tulis sedangkan Shard tahun lalu menganggur).
3. **List-Based (Directory/Geography) Sharding**: Membagi shard berdasarkan daftar nilai diskrit, misalnya berdasarkan negara/wilayah (`Indonesia` → Shard Asia, `Jerman` → Shard Eropa).
4. **Composite Sharding**: Menggabungkan dua teknik partisi sekaligus (misal dibagi berdasarkan Wilayah terlebih dahulu, lalu di dalam tiap wilayah dibagi lagi menggunakan Hash).

### Kelebihan & Tantangan Besar Sharding
- **Kelebihan**: Kapasitas penyimpanan dan kecepatan tulis meningkat secara horizontal tanpa batas; query lebih cepat karena setiap shard hanya mencari di sebagian kecil data; jika 1 shard gangguan, pengguna di shard lain tetap aman.
- **Tantangan & Kekurangan Sharding (Sering Ditanyakan Interviewer!)**:
  1. **Cross-Shard Joins Sangat Lambat**: Melakukan `JOIN` data yang tersebar di 5 server shard berbeda lewat jaringan sangat tidak efisien (sering kali harus diatasi dengan *Denormalisasi*).
  2. **Masalah Rebalancing & Hotspot (Celebrity Problem)**: Jika kita membagi shard secara kurang tepat (misal shard tempat akun Justin Bieber dan Cristiano Ronaldo berada), satu shard tersebut akan kebanjiran trafik sedangkan shard lain sepi. Memindahkan data antar shard (*rebalancing*) tanpa *downtime* membutuhkan **Consistent Hashing**.

---

## 14. Consistent Hashing

### Masalah pada Hashing Tradisional (`Hash(key) % N`)
Bayangkan kita punya $N = 4$ server cache/shard, dan kita memetakan data menggunakan rumus modulo sederhana:

$$
\text{Server Tujuan} = \text{Hash}(\text{key}) \bmod N
$$

![simple-hashing](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-II/consistent-hashing/simple-hashing.png)

**Apa bencana yang terjadi jika 1 server mati (sehingga $N$ berubah dari $4$ menjadi $3$) atau kita menambah 1 server baru ($N$ menjadi $5$)?**
Karena pembagi $N$ berubah, hasil dari $\text{Hash}(\text{key}) \bmod N$ untuk **hampir 100% kunci data akan berubah total!** Akibatnya, seluruh data di cache mendadak salah alamat (*Cache Miss* massal secara serentak) dan menyerbu database utama hingga mati (**Cache Stampede**)!

### Solusi: Bagaimana Cara Kerja Consistent Hashing?

**Consistent Hashing** memetakan baik **Server (Nodes)** maupun **Kunci Data (Keys)** ke atas sebuah lingkaran imajiner yang disebut **Hash Ring** (misal rentang nilai hash dari $0$ hingga $2^{32}-1$).

![consistent-hashing](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-II/consistent-hashing/consistent-hashing.png)

1. Setiap **Server** di-hash berdasarkan IP/namanya dan ditempatkan pada titik tertentu di keliling lingkaran.
2. Setiap **Kunci Data (Request/Key)** juga di-hash menggunakan fungsi yang sama sehingga jatuh pada suatu titik di lingkaran.
3. Untuk mengetahui server mana yang menyimpan kunci tersebut, kita cukup **berjalan searah jarum jam (*clockwise*)** dari titik kunci tersebut sampai menemukan **Server terdekat pertama**!

**Mengapa Consistent Hashing Luar Biasa?**
Ketika kita menambah 1 server baru atau 1 server mati di atas lingkaran *Hash Ring*, **hanya sebagian kecil data ($R = K / N$) yang perlu berpindah ke server tetangganya**, sedangkan mayoritas data lainnya tetap berada di server yang sama!

$$
R = \frac{K}{N}
$$
*(Di mana $K$ = jumlah total kunci data, dan $N$ = jumlah server).*

### Mengatasi Distribusi Tidak Merata dengan Virtual Nodes (VNodes)
Jika kita hanya menaruh sedikit server fisik di atas lingkaran, jarak antar server bisa tidak sama rata, sehingga ada satu server yang menampung porsi lingkaran terlalu besar (**Hotspot**).

Solusinya adalah **Virtual Nodes (VNodes)**:
Setiap 1 server fisik dipetakan ke **banyak titik bayangan (misal 100 titik VNode)** yang tersebar acak di seluruh keliling *Hash Ring* menggunakan $k$ fungsi hash berbeda.

![virtual-nodes](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-II/consistent-hashing/virtual-nodes.png)

- **Keuntungan VNodes**:
  - Beban data terbagi **sangat merata** di semua server fisik.
  - Server yang punya spesifikasi hardware 2x lebih kuat bisa diberi jumlah VNode 2x lebih banyak!
  - Saat 1 server fisik mati, bebannya otomatis terbagi rata ke seluruh server lain di klaster, bukan menimpa 1 server tetangga saja.

### Replikasi Data pada Hash Ring
Untuk mencegah kehilangan data ketika sebuah node mati, setiap data tidak hanya disimpan di 1 node pertama searah jarum jam, melainkan disalin ke **$N$ node berikutnya searah jarum jam** (di mana $N$ adalah **Replication Factor**, misalnya $N = 3$).
- **Contoh Pengguna Consistent Hashing**: **Apache Cassandra**, **Amazon DynamoDB**, **Discord**, **Akamai CDN**.

---

## 15. Database Federation

**Database Federation** (atau *Functional Partitioning*) adalah teknik memecah database **berdasarkan fungsi atau domain bisnisnya**, bukan memecah satu tabel yang sama.

![database-federation](https://raw.githubusercontent.com/karanpratapsingh/portfolio/master/public/static/courses/system-design/chapter-II/database-federation/database-federation.png)

- **Contoh**: Daripada menaruh seluruh tabel aplikasi e-commerce di dalam 1 database raksasa tunggal, kita membaginya menjadi:
  - **Users DB**: Khusus menyimpan tabel pengguna dan profil.
  - **Products DB**: Khusus menyimpan katalog barang dan kategori.
  - **Orders & Payments DB**: Khusus menyimpan transaksi pembelian.
- Selain itu, arsitektur *Federated Database* juga dapat menyatukan tampilan dari berbagai sumber data yang berbeda (heterogen) sehingga tampak seperti satu database logis bagi aplikasi.
- **Kelebihan**: Setiap database lebih kecil sehingga muat di memori (cache hit meningkat), beban tulis terbagi per domain bisnis, dan kegagalan pada *Products DB* tidak merusak *Users DB*.
- **Kekurangan**: Tidak bisa melakukan `JOIN` langsung antar dua database berbeda (data harus digabungkan di level kode aplikasi/backend).
