# 🎭 NARRATIVE SCRIPT PRESENTASI UAS DATA WAREHOUSE (TECHNICAL DEEP-DIVE VERSION)
### 📌 Judul Proyek: "Implementation of an ETL Pipeline-Based Data Warehouse at Kompas.com News with Sentiment Analysis, Trending Topics, and Multidimensional OLAP Visualization"
### 👥 Tim Presenter (NIM 084_087_213):
*   **Dimas Rafi Izzulhaq** (Presenter 1 - **Lead Technical & Architect**)
*   **Rizqi Aqilah Cahyani Yuniarto** (Presenter 2 - **Business Analyst & Opening**)
*   **Elvira Tiara Suci** (Presenter 3 - **Database Administrator & DWH Modeler**)
### 🗣️ Gaya Bahasa: Bilingual (Bindo-Bing Mix), Professional, Detail, & Easy to Pronounce.
### ⏱️ Total Durasi: 10 - 12 Menit

---

## 🗺️ MAP PRESENTASI & RUN-DOWN
*   **Bagian 1: Opening, Problem Statement, & Project Objectives** (Presenter 2 - Rizqi)
*   **Bagian 2: Technical Deep-Dive: Data Extraction & NLP Preprocessing Pipeline** (Presenter 1 - Dimas)
*   **Bagian 3: Database Loading Phase & Star Schema Architecture** (Presenter 3 - Elvira)
*   **Bagian 4: Technical Deep-Dive: Database Performance Optimization & pgvector Benchmarking** (Presenter 1 - Dimas)
*   **Bagian 5: Technical Deep-Dive: Multidimensional OLAP Cube with Atoti** (Presenter 1 - Dimas)
*   **Bagian 6: Empirical Analytical Findings (UAS Insights) & Closing** (Presenter 2 - Rizqi & Presenter 3 - Elvira)

---

## 🎬 SCRIPT PRESENTASI NARRATIVE BARIS-DEMI-BARIS

### 🗣️ BAGIAN 1: OPENING, PROBLEM STATEMENT, & PROJECT OBJECTIVES
**(Presenter 2 - Rizqi | Durasi: ~1.5 Menit)**

> *"Selamat pagi/siang Bapak/Ibu Dewan Penguji dan Dosen Pengampu Mata Kuliah Data Warehouse.*
> 
> *Hari ini, kelompok kami yang beranggotakan saya Rizqi Aqilah Cahyani, bersama rekan saya Dimas Rafi Izzulhaq, dan Elvira Tiara Suci, akan mempresentasikan final project Data Warehouse kami yang berjudul:* **'Implementation of an ETL Pipeline-Based Data Warehouse at Kompas.com News with Sentiment Analysis, Trending Topics, and Multidimensional OLAP Visualization.'**
> 
> *Let's talk about the background first. Kompas.com is one of the largest online news portals in Indonesia, producing hundreds of articles daily. Namun, tantangan terbesarnya adalah: **news content is highly unstructured, scattered across thousands of HTML pages, and there is no official public API available**.*
> 
> *Selain itu, untuk menganalisis opini publik secara longitudinal, kita memerlukan pemrosesan NLP yang kompleks untuk konten berbahasa Indonesia, seperti analisis sentimen, ekstraksi tokoh dan instansi, hingga representasi makna semantik berita.*
> 
> *To address these challenges, we designed an end-to-end Data Warehouse architecture* yang mengotomatisasi ekstraksi artikel berita secara harian, memprosesnya dengan *NLP Pipeline*, menyimpannya ke dalam *Star Schema di PostgreSQL Supabase*, dan menyajikannya secara interaktif menggunakan *Atoti Multidimensional OLAP Cube* yang terintegrasi secara persisten.
> 
> *Next, our Lead Technical Architect, Dimas, will explain the complex pipeline of our ETL system. Dimas, the stage is yours."*
> 
> **[Tunjukkan Halaman 1, 2, & 3 di PDF Laporan]**

---

### 🗣️ BAGIAN 2: TECHNICAL DEEP-DIVE: DATA EXTRACTION & NLP PREPROCESSING PIPELINE
**(Presenter 1 - Dimas | Durasi: ~3.5 Menit)**

> *"Thank you, Rizqi. Now, let's look at the technical implementations behind our **Extraction and Transformation (NLP) Phase**.*
> 
> *Pertama, proses **Data Extraction** dilakukan secara modular.* Di file [scraper/sitemap_parser.py](file:///e:/UAS/scraper/sitemap_parser.py), *we dynamically parse the monthly XML sitemaps of Kompas.com.* Logika pemindaian sitemap utama diatur pada fungsi `parse_sitemap_index()`, dan untuk sitemap berita bulanan diproses di fungsi `parse_sub_sitemap()`.
> 
> Jika ada tanggal yang tidak tercakup dalam sitemap, sistem kami secara otomatis melakukan *fallback* ke halaman indeks harian Kompas.com di fungsi `collect_urls_from_indeks(target_date)`. *To maintain a balanced daily sample, we limit the collection to a maximum of 50 random articles per day in our main wrapper function* `collect_all_urls()`.
> 
> *Kedua, berkas* [scraper/article_scraper.py](file:///e:/UAS/scraper/article_scraper.py) *will crawl and extract key attributes from the HTML content.* Kami mengekstrak judul menggunakan selector tag BeautifulSoup `article__title`, penulis dengan `read__author`, sub-kategori, dan konten artikel utama di fungsi `scrape_article(url)`.
> 
> *Ketiga, data mentah tersebut masuk ke dalam **Transformation Phase (NLP Pipeline)** yang berjalan secara sekuensial melalui 6 langkah utama:*
> 
> 1. **Text Cleaning**: Di berkas [preprocessor/text_cleaner.py](file:///e:/UAS/preprocessor/text_cleaner.py) pada fungsi `clean_text(text)`, *we clean the raw text using regex to strip HTML tags, special characters, and typical Kompas.com boilerplates* seperti kalimat 'Baca juga', 'Dapatkan update berita', dan mengecualikan artikel di bawah 20 kata.
> 2. **Sentiment Analysis**: Di file [preprocessor/sentiment_analyzer.py](file:///e:/UAS/preprocessor/sentiment_analyzer.py) pada fungsi `analyze_articles(articles)`, *we employ the **IndoBERT** model* melalui pipeline `transformers` HuggingFace untuk memprediksi label sentimen `positive`, `neutral`, atau `negative` beserta skor kepercayaannya.
> 3. **Embedding Generation**: Di berkas [preprocessor/embedding_generator.py](file:///e:/UAS/preprocessor/embedding_generator.py) pada fungsi `generate_article_embeddings(articles)`, *we generate a **384-dimensional vector embedding** per article* menggunakan model *SentenceTransformer* berbasis `sentence-transformers/all-MiniLM-L6-v2` untuk representasi makna semantik berita.
> 4. **Named Entity Recognition (NER)**: Di file [preprocessor/ner_extractor.py](file:///e:/UAS/preprocessor/ner_extractor.py) pada fungsi `extract_article_entities(articles)`, *we extract three key entity types* yaitu `PERSON`, `ORGANIZATION`, dan `LOCATION`.
> 5. **Named Entity Linking (NEL)**: Di file [preprocessor/nel_linker.py](file:///e:/UAS/preprocessor/nel_linker.py) pada fungsi `link_article_entities(articles)`, *we enrich the NER results by linking them to Wikidata & Wikipedia API* untuk menarik deskripsi tambahan secara otomatis.
> 6. **Trending Topics Detection**: Di berkas [preprocessor/trending_detector.py](file:///e:/UAS/preprocessor/trending_detector.py) pada fungsi `process_trending(articles, date)`, *we calculate a **trending score** per keyword per day* dengan membandingkan frekuensi harian kata kunci terhadap rata-rata historisnya.
> 
> *Untuk mengorkestrasi seluruh tugas berkabel ini, kami menggunakan Airflow DAG di* [dags/kompas_etl_dag.py](file:///e:/UAS/dags/kompas_etl_dag.py). *We use **ExternalPythonOperator** at lines 325-389 to run each task in a virtual environment subprocess*, memastikan isolasi modul NLP yang stabil."
> 
> **[Tunjukkan Halaman 3 & 4 di PDF Laporan]**

---

### 🗣️ BAGIAN 3: DATABASE LOADING PHASE & STAR SCHEMA ARCHITECTURE
**(Presenter 3 - Elvira | Durasi: ~1.5 Menit)**

> *"Thank you, Dimas. Now, let's transition to the **Loading Phase and our Star Schema Database Design**.*
> 
> *Seluruh proses pengunggahan data hasil pengayaan NLP dimuat secara modular di berkas* [feeder/loader.py](file:///e:/UAS/feeder/loader.py). *We implement an **UPSERT (Insert on Conflict DO UPDATE) pattern** to guarantee idempotency.* Artinya, jika pipeline Airflow terpaksa dijalankan ulang pada tanggal yang sama, sistem tidak akan menduplikasi baris data melainkan menimpanya dengan versi terbaru.
> 
> *Skema Data Warehouse kami di Supabase (PostgreSQL) dirancang menggunakan **Star Schema** yang terdiri atas 2 Tabel Fakta dan 5 Tabel Dimensi:*
> 
> 1. **fact_artikel** (Tabel Fakta Utama): Menyimpan ukuran kuantitatif utama. Kami memuat baris data ini pada [feeder/loader.py:L211-L243](file:///e:/UAS/feeder/loader.py#L211-L243) menggunakan fungsi `load_article()`, menyimpan metrik `sentimen_score` dan `jumlah_kata`. *To optimize query performance, we partitioned this table monthly from 2024 to 2026 based on the `tanggal_publikasi` column.*
> 2. **fact_trending** (Tabel Fakta Pendukung): Menyimpan keyword trending harian, frekuensi, dan `skor_trending`. Dimuat pada [feeder/loader.py:L271-L288](file:///e:/UAS/feeder/loader.py#L271-L288) menggunakan fungsi `load_trending()`.
> 3. **dim_waktu** (Dimensi Waktu): Secara dinamis memecah tanggal publikasi berita menjadi atribut `tanggal`, `hari`, `minggu`, `bulan`, `nama_bulan`, `kuartal`, dan `tahun`. Kode dimuat pada [feeder/loader.py:L30-L61](file:///e:/UAS/feeder/loader.py#L30-L61) menggunakan fungsi `upsert_dim_waktu()`.
> 4. **dim_kategori** (Dimensi Kategori): Menyimpan rubrik kategori berita Kompas. Dimuat pada [feeder/loader.py:L63-L82](file:///e:/UAS/feeder/loader.py#L63-L82) menggunakan fungsi `upsert_dim_kategori()`.
> 5. **dim_penulis** (Dimensi Penulis): Menyimpan nama jurnalis. Dimuat pada [feeder/loader.py:L84-L104](file:///e:/UAS/feeder/loader.py#L84-L104) menggunakan fungsi `upsert_dim_penulis()`.
> 6. **dim_sentimen** (Dimensi Sentimen): Menyimpan kategori label sentimen (`positive`, `neutral`, `negative`). Dirujuk pada [feeder/loader.py:L105-L116](file:///e:/UAS/feeder/loader.py#L105-L116) menggunakan fungsi `get_sentimen_id()`.
> 7. **dim_entitas** (Dimensi Entitas): Menyimpan entitas nama tokoh, instansi, lokasi beserta deskripsi Wikidata dan skor NEL. Dimuat pada [feeder/loader.py:L117-L169](file:///e:/UAS/feeder/loader.py#L117-L169) menggunakan fungsi `upsert_dim_entitas()`.
> 8. **bridge_artikel_entitas**: Tabel perantara *Many-to-Many* untuk menghubungkan satu artikel dengan banyak entitas beserta frekuensi kemunculannya. Dimuat pada [feeder/loader.py:L245-L268](file:///e:/UAS/feeder/loader.py#L245-L268) di dalam fungsi `load_article()`.
> 
> *Selain itu, di akhir pemuatan batch, loader kami secara otomatis menjalankan penyegaran basis data menggunakan trigger **`REFRESH MATERIALIZED VIEW`** di* [feeder/loader.py:L318-L333](file:///e:/UAS/feeder/loader.py#L318-L333) *pada fungsi `refresh_materialized_views()`. Hal ini mencakup views `mv_artikel_per_kategori_bulan`, `mv_sentimen_harian`, dan `mv_top_entitas`.*"
> 
> **[Tunjukkan Halaman 4, 6 & 7 di PDF Laporan]**

---

### 🗣️ BAGIAN 4: TECHNICAL DEEP-DIVE: DATABASE PERFORMANCE OPTIMIZATION & PGVECTOR BENCHMARKING
**(Presenter 1 - Dimas | Durasi: ~2 Menit)**

> *"Thank you, Elvira. Now, let's look at the database optimizations.*
> 
> *Untuk mendukung analisis big data berskala puluhan ribu baris, kami menguji performa query menggunakan perintah **`EXPLAIN ANALYZE`** langsung pada database Supabase PostgreSQL.*
> 
> *We implemented 4 main optimization techniques:*
> 
> 1. **Materialized View Optimization**: *We compared direct aggregation queries vs queries using our materialized view* `mv_artikel_per_kategori_bulan`. Tanpa optimasi, kueri membutuhkan waktu 2230.9 ms. *With our materialized view, it executes in just 10.5 ms! That is **210.9x times faster**!*
> 2. **Partition Pruning**: Dengan membagi tabel `fact_artikel` per bulan, *PostgreSQL successfully performed partition pruning with status Pruning Active: True.* Query pencarian range tanggal hanya memindai partisi bulan yang cocok, dengan kecepatan kueri drop drastis hingga **1.059 ms**!
> 3. **B-Tree Indexing**: Kami membuat indeks B-Tree pada kolom kunci `category_id` dan `sentiment_id` untuk mempercepat pemfilteran agregasi data silang.
> 4. **pgvector IVFFlat Indexing**: Ini bagian yang paling krusial untuk pencarian kemiripan semantik berbasis kecerdasan buatan. Kami menyimpan embedding berita menggunakan ekstensi `pgvector`. Kueri kesamaan kosinus biasa dengan operator `<=>` memakan waktu 3687.2 ms. *By building an **IVFFlat index** (Index Vector Cosine Similarity), search performance jumped to 176.4 ms! This is a massive **20.9x times faster** vector search!*
> 
> *Semua optimasi ini dipanggil ketika user melakukan kueri pencarian semantik artikel di berkas* [atoti_analysis/olap_cube.py:L286-L312](file:///e:/UAS/atoti_analysis/olap_cube.py#L286-L312) *melalui fungsi `semantic_search(query_text, top_k)` menggunakan operator `<=>` pgvector.*"
> 
> **[Tunjukkan Halaman 5 di PDF Laporan]**

---

### 🗣️ BAGIAN 5: TECHNICAL DEEP-DIVE: MULTIDIMENSIONAL OLAP CUBE WITH ATOTI
**(Presenter 1 - Dimas | Durasi: ~2.5 Menit)**

> *"Mari kita lanjut ke bagian visualisasi dan analisis multidimensi: **Atoti OLAP Cube**.*
> 
> *Seluruh pemodelan logika Cube kami bangun di file* [atoti_analysis/olap_cube.py](file:///e:/UAS/atoti_analysis/olap_cube.py). *Our session starts persistently on **port 11875**.* Pada fungsi `create_cube()` di baris 48-207, kami menginisialisasi sesi dengan `tt.Session.start` dan menghubungkan folder dashboard `content/` ke parameter `user_content_storage`.
> 
> *We designed a **Dual-Cube Architecture** to isolate separate analytical concerns:*
> *   **KompasNewsCube**: Kubus utama untuk menganalisis relasi fakta artikel berita, penulis, kategori, sentimen, dan waktu.
> *   **TrendingCube**: Kubus pendukung untuk memetakan ledakan frekuensi kata kunci trending harian.
> 
> *Di dalam KompasNewsCube, kami mendefinisikan dimensi **Waktu** terstruktur yang bertingkat:* `Tahun` -> `Kuartal` -> `Bulan` -> `Tanggal` di baris [atoti_analysis/olap_cube.py:L142-L152](file:///e:/UAS/atoti_analysis/olap_cube.py#L142-L152). *This hierarchical Time dimension is crucial for interactive Drill-Down and Roll-Up.*
> 
> *Kami juga merancang kalkulasi metrik bisnis secara bermakna di baris* [atoti_analysis/olap_cube.py:L165-L200](file:///e:/UAS/atoti_analysis/olap_cube.py#L165-L200), *seperti: `Jumlah Artikel`, `Rata-rata Sentimen`, `Rata-rata Kata`, `Persen Positif`, dan `Persen Negatif`.*
> 
> *Di sisi kode, kami menguji 4 Operasi OLAP secara programmatic:*
> 1. **Roll-up**: Mengelompokkan statistik artikel dari tanggal harian yang sangat granular naik ke tingkat Bulan dan Tahun pada fungsi `olap_rollup(cube)` [atoti_analysis/olap_cube.py:L209-L227](file:///e:/UAS/atoti_analysis/olap_cube.py#L209-L227).
> 2. **Drill-down**: Menurunkan dimensi waktu Tahun 2026 untuk merinci rincian distribusi artikel per Bulan hingga tanggal harian pada fungsi `olap_drilldown(cube)` [atoti_analysis/olap_cube.py:L228-L251](file:///e:/UAS/atoti_analysis/olap_cube.py#L228-L251).
> 3. **Slice**: Memotong sub-cube satu dimensi, contohnya menyaring seluruh fakta hanya untuk Kategori khusus `'Finansial'` pada fungsi `olap_slice(cube)` [atoti_analysis/olap_cube.py:L252-L266](file:///e:/UAS/atoti_analysis/olap_cube.py#L252-L266).
> 4. **Dice**: Mengiris sub-cube multi-dimensi sekaligus, contohnya menyaring data dengan filter Kategori `'Bandung'` AND Tahun `2026` AND Sentimen `'neutral'` pada fungsi `olap_dice(cube)` [atoti_analysis/olap_cube.py:L267-L285](file:///e:/UAS/atoti_analysis/olap_cube.py#L267-L285).
> 
> *Bapak/Ibu penguji, sekarang mari kita lihat dan demonstrasikan secara langsung visualisasi dashboard interaktif yang sudah kami rancang di browser!*"
> 
> **[Tunjukkan / Pindah ke Layar Atoti UI http://localhost:11875]**

---

### 🗣️ BAGIAN 6: EMPIRICAL ANALYTICAL FINDINGS (UAS INSIGHTS) & CLOSING
**(Presenter 2 - Rizqi & Presenter 3 - Elvira | Durasi: ~2 Menit)**

> **(Elvira)**:
> *"Terima kasih banyak, Dimas. Berdasarkan pengujian multidimensi pada OLAP Cube terhadap **15.228 baris data artikel** dan **5.273 kata kunci trending**, berikut adalah temuan analitis kami secara empiris:*
> 
> *Pertama, **Kategori Finansial** merupakan kolom rubrik berita paling produktif dengan total **1.100 artikel** dan memiliki rata-rata sentimen global yang sangat stabil di angka **0.83**. Ini menunjukkan jurnalisme ekonomi Kompas.com menyajikan berita bisnis secara objektif, factual, kredibel, dan minim bias clickbait.*
> 
> *Kedua, **Kanal Regional (seperti Bandung dengan 335 artikel)** sangat didominasi oleh berita berlabel Netral (**303 artikel**). Ini membuktikan fokus jurnalisme daerah yang mengedepankan informasi layanan publik, seremoni pemkot, dan pembangunan infrastruktur kota."*
> 
> **(Rizqi)**:
> *"Ketiga, dari aspek **Objectivity & Peace Journalism**, porsi sentimen berita secara global didominasi penuh oleh label **Netral (80-85%)**, disusul positif (~10-15%) dan negatif (~3-5%). Hal ini membuktikan secara empiris bahwa jurnalisme Kompas.com sangat berimbang dan menghindari konflik.*
> 
> *Namun, terdapat anomali unik yang sangat logis secara ilmiah pada kolom **Cek Fakta (sentimen negatif mencapai hampir 50%)** dan kolom **Katanetizen (sentimen negatif menyentuh ~75%)**. Hal ini terjadi karena Cek Fakta memang bertugas menginvestigasi hoaks, penipuan online, dan kejahatan siber, sementara Katanetizen menampung wadah kritik pedas dan keluhan netizen media sosial terkait kebijakan pemerintah.*
> 
> *Terakhir, kami mendeteksi **lonjakan volume data (Data Spike)** yang sangat drastis pada **Mei 2026** hingga menyentuh lebih dari **3.600 artikel**.*
> 
> *Secara statistik, peningkatan ukuran sampel yang masif ini memicu normalisasi rata-rata sentimen global ke titik terendahnya yaitu **0.795**.*
> *This scientifically proves the **Law of Large Numbers (Hukum Bilangan Besar)**.* Ketika sampel melebar, spectrum keragaman topik berita (seperti kriminal, bencana, politik) menyebar luas, sehingga menormalkan rata-rata sentimen ke arah nilai populasi berita harian yang sesungguhnya yang cenderung lebih realistis, kritis, dan objektif.
> 
> *Demikian presentasi dari kelompok kami. Semoga pipeline ETL dan OLAP Cube ini dapat menjadi fondasi kokoh untuk analisis media digital di masa depan.*
> 
> *Thank you very much. We are now open for Q&A session."*

---
**== END OF SCRIPT ==**
