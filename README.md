<div align="center">

<img src="app/static/img/logo.png" alt="Logo SDM Polri" width="120"/>

# RAG SDM POLRI

### Aplikasi Question & Answer berbasis **Retrieval-Augmented Generation (RAG)**
### untuk Dokumen SDM Kepolisian Negara Republik Indonesia

[![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-009688?style=flat-square&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![Docker](https://img.shields.io/badge/Docker-ready-2496ED?style=flat-square&logo=docker&logoColor=white)](https://docker.com)
[![LLM](https://img.shields.io/badge/LLM-6_Provider-412991?style=flat-square)](#-teknologi-yang-digunakan)
[![Qdrant](https://img.shields.io/badge/Vector_DB-Qdrant-DC244C?style=flat-square)](https://qdrant.tech)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

</div>

---

## Deskripsi

Prototype aplikasi **Question & Answer** yang memungkinkan personel Polri mendapatkan informasi dari dokumen SDM secara cepat dan akurat menggunakan bahasa sehari-hari — tanpa perlu membuka dokumen dan mencari halaman per halaman secara manual.

Sistem ini mengimplementasikan pipeline **RAG (Retrieval-Augmented Generation)** yang:
1. **Memproses** dokumen PDF menjadi potongan (*chunks*) terindeks
2. **Menemukan** bagian dokumen yang paling relevan dengan pertanyaan pengguna menggunakan Hybrid Retrieval (BM25 + embedding multibahasa + Reciprocal Rank Fusion)
3. **Menghasilkan** jawaban akurat berbahasa Indonesia menggunakan LLM pilihan Anda (Gemini, OpenAI, Groq, OpenRouter, Anthropic, atau Mistral), lengkap dengan referensi halaman sumber

> Sistem juga dapat menjawab pertanyaan umum di luar knowledge base sebagai asisten AI ramah.

---

## Fitur Utama

| Fitur | Keterangan |
|---|---|
| **Smart Retrieval** | BM25 + embedding multibahasa (fastembed) + RRF + re-ranking, vektor di Qdrant |
| **Dual Mode** | Mode RAG (dokumen) + Mode General (chat bebas) |
| **Multi-dokumen** | Upload PDF/TXT tak terbatas, dipilih otomatis sesuai pertanyaan |
| **Sitasi Halaman** | Setiap jawaban dilengkapi referensi `[1][2]` + nomor halaman |
| **Chunk Explorer** | Inspect semua potongan dokumen yang terindeks |
| **Score Bars** | Visualisasi skor KEYWORD · VECTOR · RERANK per sumber |
| **Cyber UI** | Tampilan terminal siber bertema SDM Polri (emas/merah/hitam) |
| **Docker Ready** | Satu perintah `docker compose up` langsung jalan |
| **Multi-Provider AI** | Tempel API key di UI, jenis penyedia dikenali otomatis (Gemini, OpenAI, Groq, OpenRouter, Anthropic, Mistral) |
| **Cepat & Tahan Gagal** | Model Gemini ditemukan otomatis, balapan model paralel (hedged request), circuit breaker 30 dtk |
| **Mode Offline** | Tanpa API key / AI down, pencarian dokumen tetap jalan (mode ekstraktif) |
| **Responsive** | Tampilan desktop & mobile |

---

## Arsitektur Sistem

```
┌──────────────────────────────────────────────────────────────┐
│                        BROWSER / CLIENT                       │
│              Cyber UI (HTML + Vanilla JS + Canvas)            │
└───────────────────────────┬──────────────────────────────────┘
                            │ HTTP/REST
┌───────────────────────────▼──────────────────────────────────┐
│                   FastAPI Backend (:8080)                      │
│  POST /api/chat   GET /api/health   GET /api/documents        │
│  POST /api/documents   DELETE /api/documents/{id}             │
│  GET /api/documents/{id}/chunks   GET /api/documents/{id}/file│
└──────┬──────────────────────────────────────────┬────────────┘
       │                                          │
┌──────▼──────────────┐              ┌────────────▼────────────┐
│   RAG Engine        │              │   LLM Router (6 provider)│
│                     │              │                          │
│  ┌───────────────┐  │              │  Gemini/OpenAI/Groq/... │
│  │  Ingest       │  │              │  ↓ hedged race + breaker │
│  │  PDF → Chunks │  │              │  ↓ mode ekstraktif      │
│  │  Laws/General │  │              │                         │
│  └───────┬───────┘  │              │                         │
│          │          │              └──────────────────────────┘
│  ┌───────▼───────┐  │
│  │  Knowledge    │  │
│  │  Index        │  │
│  │  BM25 + RRF   │  │
│  │  Re-ranking   │  │
│  └───────┬───────┘  │
│          │          │
│  ┌───────▼───────┐  │
│  │  Answer Gen   │  │
│  │  RAG / General│  │
│  │  + Sitasi     │  │
│  └───────────────┘  │
└─────────────────────┘
```

---

## Cara Menjalankan

### Prasyarat

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (Windows/Mac/Linux)
- API key LLM milik Anda sendiri — [Gemini](https://aistudio.google.com/apikey), [OpenRouter](https://openrouter.ai/settings/keys), [OpenAI](https://platform.openai.com/api-keys), [Groq](https://console.groq.com/keys) (gratis), Anthropic, atau Mistral. Lihat catatan di bawah

### 1. Clone Repository

```bash
git clone https://github.com/GENTO1337/rag-sdm-polri.git
cd rag-sdm-polri
```

### 2. Jalankan dengan Docker

```bash
sudo service docker start
docker compose up -d --build
```

Tunggu sekitar satu menit, lalu buka **http://localhost:8080**.

### 3. Tempel API key milik Anda (di browser)

Saat pertama dibuka, muncul jendela **"Aktifkan AI dengan API key Anda"** (atau klik tombol **🔑 API KEY**
di pojok kanan atas). Tempel key Anda, lalu klik **Aktifkan**. Key divalidasi ke penyedia, jenis penyedia
dikenali otomatis (OpenRouter `sk-or-v1-...`, OpenAI `sk-...`, Groq `gsk_...`, Anthropic `sk-ant-...`, Gemini `AIza...`/`AQ....`, Mistral 32 karakter), dan tersimpan di volume Docker
lokal sehingga tetap aktif setelah container di-restart. Tidak perlu terminal maupun file `.env`.

Alternatif lewat terminal (opsional):

```bash
OPENAI_API_KEY="tempel_key_anda" docker compose up -d --build            # Linux / macOS / WSL
```

```powershell
$env:OPENAI_API_KEY="tempel_key_anda"; docker compose up -d --build       # Windows PowerShell
```

> [!IMPORTANT]
> **Mengapa API key tidak ada di repository?** Penyedia API (OpenAI, Google, OpenRouter) memindai GitHub
> secara otomatis dan **langsung menonaktifkan key yang terlihat publik**. Karena itu key tidak disertakan
> di repo ini; setiap penguji memakai key miliknya sendiri. Tanpa key aplikasi tetap berjalan tanpa galat
> dalam **mode ekstraktif** (badge "AI OFFLINE", jawaban dokumen tetap muncul lengkap dengan sumber halaman).

<details>
<summary>Alternatif: skrip interaktif (key diketik tersembunyi, tidak masuk riwayat terminal)</summary>

```bash
chmod +x start.sh && ./start.sh        # Linux / macOS / WSL
```

```powershell
.\start.ps1                             # Windows PowerShell
```

Atau lewat file: `cp .env.example .env`, isi `OPENAI_API_KEY`, lalu `docker compose up -d --build`.

</details>

**Ganti key / hentikan:** jalankan ulang perintah di atas dengan key baru (container dibuat ulang otomatis),
atau `docker compose down` untuk menghentikan.

### 4. Pantau Log (opsional)

```bash
docker compose logs -f
```

Tunggu hingga log menampilkan:
```
INFO  rag.retrieval: Index siap: 131 chunk | BM25
INFO  uvicorn: Application startup complete.
```

### 4. Akses Aplikasi

Buka browser dan akses:

```
http://localhost:8080
```

---

##  Docker — Detail

### Struktur Container

```
Container: rag-sdm-polri
├── Port  : 8080 → 8080
├── Volume: rag_docs  → /app/data/docs   (dokumen PDF)
└── Volume: rag_index → /app/data/index  (BM25 & embedding cache)
```

### Perintah Berguna

```bash
# Jalankan di background
docker compose up -d

# Lihat log real-time
docker compose logs -f

# Hentikan
docker compose down

# Rebuild dari awal (setelah update kode)
docker compose up --build

# Hapus semua data (termasuk volume)
docker compose down -v
```

### Variabel Environment

| Variabel | Default | Keterangan |
|---|---|---|
| `OPENAI_API_KEY` | *(wajib)* | API Key dari platform.openai.com |
| `OPENAI_MODEL` | *(otomatis)* | Model yang dipakai; kosong = dipilih otomatis sesuai penyedia |
| `LLM_PROVIDER` | `openai` | Penyedia LLM (`openai` / `none` untuk mode ekstraktif) |
| `MIN_COVERAGE` | `0.42` | Ambang relevansi BM25 (0.0–1.0) |
| `TOP_K` | `6` | Jumlah chunk terambil per query |
| `LLM_TIMEOUT` | `25` | Timeout request ke LLM (detik, efektif maks 20) |
| `CHUNK_SIZE` | `1100` | Ukuran chunk teks (karakter) |

---

## Menjalankan Tanpa Docker (Development)

```bash
# Buat virtual environment
python -m venv .venv
.venv\Scripts\activate        # Windows
# source .venv/bin/activate   # Linux/Mac

# Install dependensi
pip install -r requirements.txt

# Jalankan server
python -m uvicorn app.main:app --host 127.0.0.1 --port 8080 --reload
```

---

## Struktur Proyek

```
rag-sdm-polri/
├── app/
│   ├── main.py          # FastAPI routes & lifespan
│   ├── rag.py           # RAG pipeline utama (ingest→retrieve→generate)
│   ├── retrieval.py     # BM25 + Hybrid Index + Re-ranking
│   ├── ingest.py        # PDF parser + chunking (Laws/General template)
│   ├── llm.py           # LLM router 6 provider (auto-detect, hedged race, circuit breaker)
│   ├── config.py        # Settings dari environment variable
│   └── static/
│       ├── index.html   # Single-page Cyber UI
│       ├── css/         # (embedded di index.html)
│       └── img/
│           └── logo.png # Logo SDM Polri
├── data/
│   ├── docs/            # Dokumen PDF knowledge base
│   ├── index/           # Cache BM25 & embedding (auto-generated)
│   └── catalog.json     # Metadata dokumen bawaan
├── scripts/
│   ├── test_retrieval.py    # Unit test retrieval (7 queries)
│   ├── test_e2e_full.py     # End-to-end test API
│   └── debug_retrieval.py   # Debug skor BM25 per query
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── .env.example
├── .gitignore
└── README.md
```

---

## 🔧 Teknologi yang Digunakan

| Komponen | Teknologi |
|---|---|
| **Bahasa & Runtime** | Python 3.11 |
| **Backend** | FastAPI, Uvicorn, Pydantic |
| **PDF Parsing** | PyMuPDF (fitz); OCR opsional RapidOCR (ONNX) untuk PDF scan |
| **Retrieval leksikal** | BM25 (implementasi sendiri) |
| **Retrieval semantik** | fastembed (ONNX, CPU) — `paraphrase-multilingual-MiniLM-L12-v2` |
| **Vector store** | Qdrant (mode lokal persisten), cadangan NumPy |
| **Fusi & Re-ranking** | Reciprocal Rank Fusion + skor coverage istilah |
| **LLM (6 provider)** | Google Gemini, OpenAI, Groq, OpenRouter, Anthropic, Mistral |
| **Optimasi LLM** | Auto-detect provider dari key, model discovery Gemini, hedged parallel request, circuit breaker |
| **HTTP Client** | HTTPX |
| **Frontend** | HTML5, CSS3, Vanilla JavaScript (tanpa build step) |
| **Containerisasi** | Docker, Docker Compose (named volume, non-root user, healthcheck) |

---|---|
| **Backend** | Python 3.11, FastAPI, Uvicorn |
| **PDF Parsing** | PyMuPDF (fitz) + OCR sidecar |
| **Retrieval** | BM25 (custom), Numpy, Reciprocal Rank Fusion |
| **Re-ranking** | Keyword coverage + phrase match + section match |
| **LLM** | OpenAI API (gpt-4o-mini) |
| **Frontend** | Vanilla JS, HTML5 Canvas (particle network) |
| **Containerisasi** | Docker, Docker Compose |
| **HTTP Client** | HTTPX (async-capable) |

---

##  Cara Penggunaan

### Bertanya dari Knowledge Base

Ketik pertanyaan langsung di kolom teks, contoh:

```
Bagaimana ketentuan penilaian kinerja anggota Polri?
Apa saja persyaratan mengikuti SBP T.A. 2027?
Berapa pangkat minimal untuk seleksi Bintara dari Tamtama lulusan SMA?
```

Sistem akan:
1. Mencari bagian dokumen yang paling relevan
2. Memberikan jawaban dengan sitasi `[1][2]` dan nomor halaman
3. Menampilkan kartu sumber dengan skor relevansi

### Upload Dokumen Baru

Klik tombol **"Tambah dokumen"** di sidebar → pilih file PDF/TXT → tunggu proses ingest selesai.

### Pertanyaan Umum

Sistem juga dapat menjawab pertanyaan di luar knowledge base:

```
Halo, siapa kamu?
Apa itu RAG?
Berapa planet di tata surya?
```

### Chunk Explorer

Klik tombol **"Chunks"** pada dokumen di sidebar untuk melihat semua potongan teks yang terindeks.

---

##  Testing

```bash
# Test retrieval (7 query — harus semua PASS)
python scripts/test_retrieval.py

# Test end-to-end API (5 skenario)
python scripts/test_e2e_full.py

# Debug skor BM25 untuk query tertentu
python scripts/debug_retrieval.py
```

---

##  Catatan

- Retrieval bersifat **hibrida** (BM25 + embedding). Model embedding diunduh sekali dari HuggingFace di latar belakang; selama belum siap, sistem memakai BM25 saja sehingga startup tidak menunggu.
- Jawaban hanya berdasarkan dokumen di knowledge base. Jika informasi tidak tersedia, sistem akan menyampaikan bahwa data tidak ditemukan.
- API key dibuat di dashboard penyedia pilihan Anda (mis. [Google AI Studio](https://aistudio.google.com/apikey)). Key disimpan lokal di volume Docker dan tidak pernah ditampilkan kembali.
- Di WSL, Docker daemon perlu dinyalakan setelah restart: `sudo service docker start`.

---

##  Dibuat untuk

**Seleksi Kemampuan Pemrograman — Tahap 2**
Topik: *Retrieval-Augmented Generation (RAG)*
by : Bripda Zul Fahmi Rizki
---

<div align="center">

**RAG SDM POLRI** — *Secure Knowledge Terminal*

Built with ❤️ using FastAPI + Qdrant + Multi-Provider LLM

</div>
