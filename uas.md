# 🎭 NARRATIVE SCRIPT PRESENTASI UAS DATA WAREHOUSE (TECHNICAL DEEP-DIVE VERSION)
### 📌 Judul Proyek: "Implementation of an ETL Pipeline-Based Data Warehouse at Kompas.com News with Sentiment Analysis, Trending Topics, and Multidimensional OLAP Visualization"
### 👥 Tim Presenter (NIM 084_087_213):
*   **Dimas Rafi Izzulhaq** (Presenter 1 - **Lead Technical & Architect**)
*   **Rizqi Aqilah Cahyani Yuniarto** (Presenter 2 - **Business Analyst & Opening**)
*   **Elvira Tiara Suci** (Presenter 3 - **Database Administrator & DWH Modeler**)
### 🗣️ Gaya Bahasa: Bilingual (Bindo-Bing Mix), Professional, Detail, & Easy to Pronounce.
### ⏱️ Total Durasi: 10 - 12 Menit
### 🎯 PPT Highlight: **Insight Analitis, OLAP Operations, & Hasil Query**

---

## 🗺️ MAP PRESENTASI & RUN-DOWN
| No | Bagian | Presenter | Durasi |
|----|--------|-----------|--------|
| 1 | Opening, Problem Statement, & Project Objectives | Rizqi | ~1.5 min |
| 2 | Technical Deep-Dive: Data Extraction & NLP Pipeline | Dimas | ~2.5 min |
| 3 | Database Loading Phase & Star Schema Architecture | Elvira | ~1.5 min |
| 4 | **🔥 Technical Deep-Dive: DB Optimization & pgvector Benchmark** | **Dimas** | ~1.5 min |
| 5 | **🔥 Technical Deep-Dive: OLAP Cube, Query Operations & Live Demo** | **Dimas** | ~3 min |
| 6 | **🔥 Empirical Analytical Insights & Closing** | Rizqi + Elvira | ~2 min |

---

## 🎬 SCRIPT PRESENTASI

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
**(Presenter 1 - Dimas | Durasi: ~2.5 Menit)**

> *"Thank you, Rizqi. Now, let's look at the technical implementations behind our **Extraction and Transformation (NLP) Phase**.*
> 
> *Pertama, proses **Data Extraction** dilakukan secara modular.*

**📂 BUKA FILE: [scraper/sitemap_parser.py](file:///e:/UAS/scraper/sitemap_parser.py)**
> *Di file `sitemap_parser.py`, we dynamically parse the monthly XML sitemaps of Kompas.com.* Logika pemindaian sitemap utama diatur pada fungsi `parse_sitemap_index()` **di baris 34-49**, dan untuk sitemap berita bulanan diproses di fungsi `parse_sub_sitemap()` **di baris 52-93**.
> 
> Jika ada tanggal yang tidak tercakup dalam sitemap, sistem kami secara otomatis melakukan *fallback* ke halaman indeks harian Kompas.com di fungsi `collect_urls_from_indeks(target_date)` **di baris 128-168**. *To maintain a balanced daily sample, we limit the collection to a maximum of 50 random articles per day* di `collect_all_urls()` **baris 201-203**.

**📂 BUKA FILE: [scraper/article_scraper.py](file:///e:/UAS/scraper/article_scraper.py)**
> *Kedua, berkas `article_scraper.py` will crawl and extract key attributes from the HTML content.* Kami mengekstrak judul menggunakan selector BeautifulSoup `article__title`, penulis dengan `read__author`, sub-kategori, dan konten artikel utama di fungsi `scrape_article(url)`.

> *Ketiga, data mentah tersebut masuk ke dalam **Transformation Phase (NLP Pipeline)** yang berjalan secara sekuensial melalui 6 langkah utama:*
> 
> 1. **Text Cleaning**: [preprocessor/text_cleaner.py](file:///e:/UAS/preprocessor/text_cleaner.py) → fungsi `clean_text(text)` — *strip HTML tags, boilerplates* ('Baca juga', 'KOMPAS.com -'), hapus artikel < 20 kata.
> 2. **Sentiment Analysis**: [preprocessor/sentiment_analyzer.py](file:///e:/UAS/preprocessor/sentiment_analyzer.py) → fungsi `analyze_articles(articles)` — model **IndoBERT** via HuggingFace `transformers`, output: label `positive/neutral/negative` + skor.
> 3. **Embedding Generation**: [preprocessor/embedding_generator.py](file:///e:/UAS/preprocessor/embedding_generator.py) → fungsi `generate_article_embeddings(articles)` — **384-dim vector** via `sentence-transformers/all-MiniLM-L6-v2`.
> 4. **Named Entity Recognition (NER)**: [preprocessor/ner_extractor.py](file:///e:/UAS/preprocessor/ner_extractor.py) → fungsi `extract_article_entities(articles)` — tipe: `PERSON`, `ORGANIZATION`, `LOCATION`.
> 5. **Named Entity Linking (NEL)**: [preprocessor/nel_linker.py](file:///e:/UAS/preprocessor/nel_linker.py) → fungsi `link_article_entities(articles)` — link ke **Wikidata & Wikipedia API**.
> 6. **Trending Topics Detection**: [preprocessor/trending_detector.py](file:///e:/UAS/preprocessor/trending_detector.py) → fungsi `process_trending(articles, date)` — hitung `skor_trending` harian.

**📂 BUKA FILE: [dags/kompas_etl_dag.py](file:///e:/UAS/dags/kompas_etl_dag.py)**
> *Untuk mengorkestrasi seluruh tugas ini, kami menggunakan Airflow DAG.* **Baris 325-389** mendefinisikan 9 task menggunakan `ExternalPythonOperator` yang berjalan di virtualenv subprocess terpisah untuk menjaga kestabilan resource."

---

### 🗣️ BAGIAN 3: DATABASE LOADING PHASE & STAR SCHEMA ARCHITECTURE
**(Presenter 3 - Elvira | Durasi: ~1.5 Menit)**

> *"Thank you, Dimas. Now, let's transition to the **Loading Phase and our Star Schema Database Design**.*

**📂 BUKA FILE: [feeder/loader.py](file:///e:/UAS/feeder/loader.py)**
> *Seluruh proses pengunggahan data dimuat di berkas `loader.py`. We implement an **UPSERT (Insert on Conflict DO UPDATE) pattern** to guarantee idempotency.*
> 
> *Skema Data Warehouse kami di Supabase dirancang menggunakan **Star Schema** — 2 Tabel Fakta dan 5 Tabel Dimensi:*
> 
> | No | Tabel di Supabase | Tipe | Fungsi Loader | Baris Kode |
> |----|-------------------|------|---------------|------------|
> | 1 | `fact_artikel` | Fakta Utama | `load_article()` | **L211-L243** |
> | 2 | `fact_trending` | Fakta Pendukung | `load_trending()` | **L271-L288** |
> | 3 | `dim_waktu` | Dimensi | `upsert_dim_waktu()` | **L30-L61** |
> | 4 | `dim_kategori` | Dimensi | `upsert_dim_kategori()` | **L63-L82** |
> | 5 | `dim_penulis` | Dimensi | `upsert_dim_penulis()` | **L84-L104** |
> | 6 | `dim_sentimen` | Dimensi | `get_sentimen_id()` | **L105-L116** |
> | 7 | `dim_entitas` | Dimensi | `upsert_dim_entitas()` | **L117-L169** |
> | 8 | `bridge_artikel_entitas` | Bridge M:N | dalam `load_article()` | **L245-L268** |
> 
> *Di akhir batch, fungsi `refresh_materialized_views()` **baris 318-333** otomatis me-refresh 3 Materialized Views: `mv_artikel_per_kategori_bulan`, `mv_sentimen_harian`, dan `mv_top_entitas`.*"
> 
> **[Tunjukkan Halaman 4, 6 & 7 di PDF Laporan — Diagram Star Schema]**

---

### 🔥🗣️ BAGIAN 4: TECHNICAL DEEP-DIVE: DB PERFORMANCE OPTIMIZATION & PGVECTOR BENCHMARK
**(Presenter 1 - Dimas | Durasi: ~1.5 Menit)**

> *"Thank you, Elvira. Now, let's look at our **database optimization results**.*
> 
> *Kami menguji performa query menggunakan `EXPLAIN ANALYZE` langsung di Supabase PostgreSQL.*

**📊 TUNJUKKAN SLIDE PPT: Tabel Benchmark Optimasi (Halaman 5 PDF)**

> | Teknik Optimasi | Before | After | Speedup |
> |----------------|--------|-------|---------|
> | **Materialized View** (`mv_artikel_per_kategori_bulan`) | 2230.9 ms | 10.5 ms | **210.9x** |
> | **Partition Pruning** (fact_artikel per bulan) | — | 1.059 ms | Pruning Active: ✅ |
> | **B-Tree Index** (category_id, sentiment_id) | Sequential | Index Scan | — |
> | **pgvector IVFFlat** (embedding cosine `<=>`) | 3687.2 ms | 176.4 ms | **20.9x** |

> *Yang paling krusial adalah **pgvector IVFFlat Indexing** untuk pencarian semantik berbasis AI.*

**📂 BUKA FILE: [atoti_analysis/olap_cube.py](file:///e:/UAS/atoti_analysis/olap_cube.py) → scroll ke baris 286-312**
> *Di fungsi `semantic_search(query_text, top_k)` **baris 286-312**, kita bisa lihat kueri SQL yang menggunakan operator `<=>` dari pgvector untuk menghitung cosine similarity antara embedding query user dengan embedding artikel di database. Sebelum ada index, kueri ini butuh 3687 ms. Setelah kita buat IVFFlat index, turun jadi 176 ms — **20.9x lebih cepat**!*"

```python
# olap_cube.py baris 295-301 — Kueri pgvector semantic search
cur.execute("""
    SELECT judul, url, 1 - (embedding <=> %s::vector) AS similarity
    FROM fact_artikel
    WHERE embedding IS NOT NULL
    ORDER BY embedding <=> %s::vector
    LIMIT %s
""", (emb_str, emb_str, top_k))
```

---

### 🔥🗣️ BAGIAN 5: TECHNICAL DEEP-DIVE: OLAP CUBE, QUERY OPERATIONS & LIVE DEMO
**(Presenter 1 - Dimas | Durasi: ~3 Menit)**

> *"Mari kita masuk ke bagian inti: **Atoti OLAP Cube dan hasil query-nya**.*

**📂 BUKA FILE: [atoti_analysis/olap_cube.py](file:///e:/UAS/atoti_analysis/olap_cube.py) → scroll ke baris 60-206**
> *Seluruh pemodelan Cube dibangun di fungsi `create_cube()` **baris 60-206**. Session Atoti berjalan persisten di **port 11875** dengan dashboard tersimpan di folder `content/`.*

```python
# olap_cube.py baris 68-73 — Inisialisasi Atoti Session
session = tt.Session.start(
    tt.SessionConfig(
        port=11875,
        user_content_storage=content_dir
    )
)
```

> *Kami merancang **Dual-Cube Architecture**:*
> * **KompasNewsCube** (baris 138): Kubus utama — artikel, sentimen, kategori, waktu.
> * **TrendingCube** (baris 185): Kubus pendukung — keyword trending harian.

> *Dimensi **Waktu** bertingkat:* `Tahun` → `Kuartal` → `Bulan` → `Tanggal` **(baris 142-147)**:

```python
# olap_cube.py baris 142-147 — Hierarki Waktu
h["Waktu"] = {
    "Tahun": artikel_table["tahun"],
    "Kuartal": artikel_table["kuartal"],
    "Bulan": artikel_table["nama_bulan"],
    "Tanggal": artikel_table["tanggal"],
}
```

> *Custom measures bisnis **(baris 165-181)**:*

```python
# olap_cube.py baris 167-181 — Definisi Measures
m["Jumlah Artikel"] = tt.agg.count_distinct(artikel_table["artikel_id"])
m["Rata-rata Sentimen"] = tt.agg.mean(artikel_table["sentimen_score"])
m["Rata-rata Kata"] = tt.agg.mean(artikel_table["jumlah_kata"])
m["Persen Positif"] = tt.filter(m["Jumlah Artikel"], h["Sentimen"]["Label"] == "positive") / m["Jumlah Artikel"]
m["Persen Negatif"] = tt.filter(m["Jumlah Artikel"], h["Sentimen"]["Label"] == "negative") / m["Jumlah Artikel"]
```

> *Sekarang, mari kita lihat **4 Operasi OLAP** dan hasil query-nya:*

---

#### 🟢 OPERASI 1: ROLL-UP (baris 209-226)
**📂 Scroll ke baris 209 di `olap_cube.py`**

> *Roll-up: Agregasi dari level Tanggal → Bulan → Tahun per Kategori.*

```python
# olap_cube.py baris 216 — Query Roll-up Granular (Tanggal)
df_tanggal = cube.query(m["Jumlah Artikel"], levels=[h["Kategori"]["Nama Kategori"], h["Waktu"]["Tanggal"]])

# olap_cube.py baris 220 — Query Roll-up ke Bulan
df_bulan = cube.query(m["Jumlah Artikel"], levels=[h["Kategori"]["Nama Kategori"], h["Waktu"]["Bulan"]])

# olap_cube.py baris 224 — Query Roll-up ke Tahun
df_tahun = cube.query(m["Jumlah Artikel"], levels=[h["Kategori"]["Nama Kategori"], h["Waktu"]["Tahun"]])
```

> *Hasilnya, data yang tadinya sangat granular per tanggal, bisa kita lihat ringkasannya per bulan dan per tahun.*

---

#### 🟢 OPERASI 2: DRILL-DOWN (baris 228-249)
**📂 Scroll ke baris 228 di `olap_cube.py`**

> *Drill-down: Kebalikan dari Roll-up — dari Tahun 2026, turun ke Bulan, lalu turun lagi ke Tanggal harian.*

```python
# olap_cube.py baris 235-239 — Drill-down ke Bulan di Tahun 2026
df_bulan = cube.query(
    m["Jumlah Artikel"],
    levels=[h["Kategori"]["Nama Kategori"], h["Waktu"]["Bulan"]],
    filter=(h["Waktu"]["Tahun"] == tahun)
)

# olap_cube.py baris 243-248 — Drill-down ke Tanggal di Bulan Mei 2026
df_hari = cube.query(
    m["Jumlah Artikel"], m["Rata-rata Sentimen"],
    levels=[h["Waktu"]["Tanggal"]],
    filter=((h["Waktu"]["Tahun"] == tahun) & (h["Waktu"]["Bulan"] == bulan))
)
```

> *Dari sini kita bisa spot anomali — contohnya **lonjakan 3.600+ artikel di Mei 2026**.*

---

#### 🟢 OPERASI 3: SLICE (baris 252-264)
**📂 Scroll ke baris 252 di `olap_cube.py`**

> *Slice: Memotong satu dimensi saja — contoh: hanya menampilkan data untuk Kategori `'Finansial'`.*

```python
# olap_cube.py baris 258-263 — Slice pada Kategori Finansial
df_slice = cube.query(
    m["Jumlah Artikel"], m["Rata-rata Sentimen"],
    levels=[h["Waktu"]["Tahun"], h["Waktu"]["Bulan"]],
    filter=(h["Kategori"]["Nama Kategori"] == kategori)
)
```

> *Hasilnya: Rubrik Finansial punya **1.100 artikel** dengan rata-rata sentimen stabil di **0.83** — artinya Kompas sangat objektif dan minim clickbait di rubrik ekonomi.*

---

#### 🟢 OPERASI 4: DICE (baris 267-283)
**📂 Scroll ke baris 267 di `olap_cube.py`**

> *Dice: Memotong multi-dimensi sekaligus — Kategori `'Bandung'` AND Tahun `2026` AND Sentimen `'neutral'`.*

```python
# olap_cube.py baris 273-282 — Dice multi-dimensi
df_dice = cube.query(
    m["Jumlah Artikel"], m["Rata-rata Sentimen"],
    levels=[h["Waktu"]["Tanggal"]],
    filter=(
        (h["Kategori"]["Nama Kategori"] == kategori) &
        (h["Waktu"]["Tahun"] == tahun) &
        (h["Sentimen"]["Label"] == sentimen)
    )
)
```

> *Hasilnya: Dari 335 artikel Bandung, **303 di antaranya netral** — jurnalisme regional Kompas sangat informatif dan tidak sensasional.*

---

#### 🟢 LIVE DEMO ATOTI UI
**🌐 BUKA BROWSER: http://localhost:11875**

> *"Bapak/Ibu penguji, sekarang mari kita demo langsung Atoti UI:*
> 1. *Drag dimensi **Kategori** ke baris, **Waktu > Bulan** ke kolom — langsung terlihat cross-tab jumlah artikel.*
> 2. *Klik kanan pada sel → **Drill-down** ke Tanggal — data langsung breakdown per hari.*
> 3. *Switch ke **Pie Chart** → terlihat komposisi sentimen per kategori.*
> 4. *Dashboard pages yang sudah kami rancang (bar chart tren bulanan, heatmap aktivitas) tersimpan secara persisten di folder `content/content.mv.db`.*"

---

### 🔥🗣️ BAGIAN 6: EMPIRICAL ANALYTICAL INSIGHTS & CLOSING
**(Presenter 2 - Rizqi & Presenter 3 - Elvira | Durasi: ~2 Menit)**

**📊 TUNJUKKAN SLIDE PPT: Key Findings**

> **(Elvira)**:
> *"Berdasarkan pengujian OLAP Cube terhadap **15.228 baris data artikel** dan **5.273 kata kunci trending**:*
> 
> | Insight | Data | Sumber Query |
> |---------|------|-------------|
> | Kategori terbanyak | **Finansial: 1.100 artikel** | `olap_slice(kategori="Finansial")` |
> | Sentimen stabil | Rata-rata **0.83** | `m["Rata-rata Sentimen"]` pada cube |
> | Regional netral | **Bandung: 303/335 netral** | `olap_dice(kategori="Bandung")` |
> | Distribusi global | **80-85% Netral** | `olap_rollup()` per Sentimen |
> 
> *Ini menunjukkan **Kompas.com menganut peace journalism** — sangat objektif dan berimbang.*"

> **(Rizqi)**:
> *"Namun, terdapat anomali menarik:*
> 
> | Anomali | Data | Penjelasan Ilmiah |
> |---------|------|-------------------|
> | **Cek Fakta**: negatif ~50% | Query Slice pada Kategori Cek Fakta | Memang bertugas membongkar hoaks & kejahatan siber |
> | **Katanetizen**: negatif ~75% | Query Slice pada Kategori Katanetizen | Wadah kritik warganet terhadap kebijakan publik |
> | **Data Spike Mei 2026**: 3.600+ artikel | Query Drill-down ke Bulan Mei 2026 | Rata-rata sentimen turun ke **0.795** — **Law of Large Numbers** |
> 
> *Fenomena Data Spike ini membuktikan secara empiris berlakunya **Hukum Bilangan Besar**: ketika sampel membesar secara masif, spektrum topik melebar (kriminal, bencana, politik), dan rata-rata sentimen menormalkan ke arah populasi yang sesungguhnya — lebih realistis dan kritis.*
> 
> *Demikian presentasi dari kelompok kami. Semoga pipeline ETL dan OLAP Cube ini dapat menjadi fondasi kokoh untuk analisis media digital di masa depan.*
> 
> *Thank you very much. We are now open for Q&A session."*

---

## 📋 QUICK REFERENCE: FILE MANA YANG HARUS DIBUKA SAAT PRESENTASI

| Urutan | Bagian Presentasi | File yang Dibuka | Baris Kode Penting |
|--------|-------------------|------------------|-------------------|
| 1 | Extraction | `scraper/sitemap_parser.py` | L34-49 (sitemap index), L128-168 (fallback indeks), L201-203 (limit 50/day) |
| 2 | Extraction | `scraper/article_scraper.py` | Fungsi `scrape_article(url)` |
| 3 | NLP Pipeline | `preprocessor/text_cleaner.py` | Fungsi `clean_text(text)` |
| 4 | NLP Pipeline | `preprocessor/sentiment_analyzer.py` | Fungsi `analyze_articles(articles)` |
| 5 | NLP Pipeline | `preprocessor/embedding_generator.py` | Fungsi `generate_article_embeddings()` |
| 6 | NLP Pipeline | `preprocessor/ner_extractor.py` | Fungsi `extract_article_entities()` |
| 7 | NLP Pipeline | `preprocessor/nel_linker.py` | Fungsi `link_article_entities()` |
| 8 | NLP Pipeline | `preprocessor/trending_detector.py` | Fungsi `process_trending()` |
| 9 | Airflow DAG | `dags/kompas_etl_dag.py` | L325-389 (definisi semua tasks) |
| 10 | Star Schema | `feeder/loader.py` | L30-169 (dim upserts), L211-268 (fact load), L318-333 (refresh MV) |
| 11 | **🔥 DB Benchmark** | `atoti_analysis/olap_cube.py` | **L286-312** (`semantic_search()` — pgvector `<=>`) |
| 12 | **🔥 OLAP Cube Init** | `atoti_analysis/olap_cube.py` | **L60-206** (`create_cube()` — session, hierarchies, measures) |
| 13 | **🔥 Roll-up** | `atoti_analysis/olap_cube.py` | **L209-226** (`olap_rollup()`) |
| 14 | **🔥 Drill-down** | `atoti_analysis/olap_cube.py` | **L228-249** (`olap_drilldown()`) |
| 15 | **🔥 Slice** | `atoti_analysis/olap_cube.py` | **L252-264** (`olap_slice()`) |
| 16 | **🔥 Dice** | `atoti_analysis/olap_cube.py` | **L267-283** (`olap_dice()`) |
| 17 | **🔥 Live Demo** | Browser: `http://localhost:11875` | Drag-drop, drill-down, chart switch |

---
**== END OF SCRIPT ==**
