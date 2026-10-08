# AutoData

**Dari CSV ke screening analisis lewat percakapan AI.**

AutoData adalah asisten analisis data dan machine learning berbasis percakapan yang membantu pengguna melakukan screening analisis awal dari dataset CSV. Project ini ditujukan terutama untuk **junior data analyst** dan **small teams** yang memiliki data tetapi masih membutuhkan bantuan untuk memahami kondisi data, menentukan langkah analisis, mencoba beberapa model, dan menilai hasil model dengan konteks yang lebih jelas.

> **Status:** Prototype  
> **Hackathon:** Hacktiv8 x IBM SkillsBuild National Hackathon 2026  
> **Theme:** Productivity & Smart Business

## Tentang AutoData

Dalam proses analisis data, pengguna tidak selalu kesulitan karena tidak memiliki data. Sering kali tantangannya justru muncul ketika harus menentukan apa yang perlu diperiksa, bagaimana menangani masalah pada dataset, target apa yang relevan, model apa yang perlu dicoba, dan apakah hasil model sudah cukup berarti.

AutoData membantu menyusun tahap awal tersebut dalam satu alur percakapan. Pengguna dapat mengirimkan dataset CSV melalui **Telegram**, kemudian workflow AutoData di **IBM Langflow** menjalankan proses analisis secara bertahap.

Alur utamanya adalah:

```text
CSV
  ↓
Data Quality Check
  ↓
Clustering / Segmentation
  ↓
Cleaning Recommendation
  ↓
User Confirmation
  ↓
Target Selection
  ↓
Cleaning & Preprocessing
  ↓
Machine Learning Screening
  ↓
Baseline Comparison
  ↓
Quality Evaluation
  ↓
Result Summary
  ↓
Google Docs (optional)
```

AutoData berfungsi sebagai **screening analisis awal**, bukan pengganti data scientist dan bukan sistem production-ready.

## Fitur Utama

### Data Quality Check

Memeriksa kondisi awal dataset, termasuk:

- jumlah baris dan kolom
- nama kolom
- missing value
- duplicate rows
- tipe data
- statistik deskriptif

### Cleaning Recommendation

Memberikan rekomendasi metode cleaning berdasarkan kondisi dataset. Pengguna tetap dapat menerima rekomendasi atau mengubah pilihan sebelum analisis machine learning dilanjutkan.

### Target Recommendation

Memberikan beberapa kandidat target berdasarkan struktur dataset dan menjelaskan kegunaannya. AutoData tidak menebak target secara diam-diam.

### Clustering / Segmentation

Melakukan clustering menggunakan **K-Means** jika dataset memiliki kolom numerik yang cukup untuk melihat pola kelompok pada data.

Hasil clustering diperlakukan sebagai **pola statistik**, bukan kategori pasti.

### Machine Learning Screening

AutoData mencoba tiga model sesuai jenis task yang terdeteksi.

**Regression**

- Random Forest Regressor
- Linear Regression
- Gradient Boosting Regressor

**Classification**

- Random Forest Classifier
- Logistic Regression
- Gradient Boosting Classifier

### Baseline Comparison

Performa model dibandingkan dengan baseline sederhana sehingga metric tidak ditampilkan tanpa konteks.

Untuk klasifikasi, baseline menggunakan proporsi kelas mayoritas. Untuk regresi, baseline menggunakan rata-rata target pada data training.

### Quality Label

Hasil evaluasi model diberi label:

- **BAIK**
- **SEDANG**
- **KURANG MEMUASKAN**

Metric, baseline, dan quality label dihitung menggunakan Python. AI digunakan untuk menjelaskan dan menyampaikan hasil tersebut, bukan untuk mengubah hasil perhitungan.

### Automated Analysis Report

Setelah screening selesai, pengguna dapat meminta laporan lengkap dalam **Google Docs** yang berisi hasil analisis, cleaning, clustering, machine learning, baseline, feature importance, temuan utama, dan keterbatasan.

## Arsitektur Sistem

### Kanal Penggunaan

Telegram digunakan sebagai kanal utama untuk penggunaan dataset karena proses upload CSV melalui IBM Bob kurang praktis untuk penggunaan langsung oleh pengguna.

```text
User
  ↓
Telegram
  ↓
Telegram Bridge
  ↓
IBM Langflow
  ↓
AutoData Workflow
  ↓
Analysis Result
  ↓
Telegram
  ↓
User
```

### IBM Langflow

IBM Langflow menjadi pusat workflow dan eksekusi analisis.

```text
                    IBM Langflow
                         │
                AI Agent + Gemini
                         │
        ┌────────────────┼────────────────┐
        │                │                │
load_files_path   run_python_repl   Google Docs
        │                │                │
        │       Data Analysis & ML        │
        │                │                │
        └────────────────┼────────────────┘
                         │
                    Chat Output
```

### IBM Bob dan MCP

IBM Bob terhubung ke workflow Langflow melalui **Model Context Protocol (MCP)**.

Dalam AutoData, IBM Bob digunakan sebagai bagian dari integrasi serta pengujian kemampuan AI Agent dalam mengakses dan memanggil workflow Langflow. Bob bukan kanal utama untuk upload dataset pengguna.

```text
IBM Bob
   ↕
  MCP
   ↕
IBM Langflow
   ↓
AutoData Workflow
```

## Penggunaan IBM Langflow

Langflow digunakan untuk membangun dan menjalankan workflow AutoData secara terstruktur.

Component utama yang digunakan adalah:

- **AI Agent** untuk mengatur penggunaan tools berdasarkan instruksi pengguna
- **Gemini** sebagai model yang mendukung AI Agent
- **load_files_path** untuk mengambil path file CSV
- **run_python_repl** untuk menjalankan proses analisis dengan Python
- **Google Docs** untuk membuat laporan analisis
- **Chat Output** untuk mengembalikan ringkasan hasil

Workflow Python dibagi menjadi dua tahap utama dalam satu sesi analisis:

```text
Call 1
EDA + Data Quality Check + Clustering

Call 2
Cleaning + Preprocessing + Machine Learning Screening
```

Setiap tahap dijalankan satu kali dan tidak menggunakan automatic retry.

## Penggunaan IBM Bob

IBM Bob digunakan untuk mengakses dan menguji workflow Langflow melalui MCP.

Bob dapat mengirimkan tool call ke Langflow, kemudian Langflow menjalankan workflow yang dipanggil dan mengembalikan hasilnya.

```text
IBM Bob
   ↓
MCP
   ↓
IBM Langflow
   ↓
Workflow AutoData
   ↓
MCP
   ↓
IBM Bob
```

Penggunaan Bob pada project ini berfokus pada **integrasi dan pengujian Agent**, bukan sebagai tempat utama pengguna mengunggah dataset CSV.

## Telegram Bridge

Telegram Bridge menghubungkan Telegram dengan Langflow.

Alur dasarnya:

```text
CSV dari User
     ↓
Telegram Bot
     ↓
Telegram Bridge
     ↓
Langflow File API
     ↓
AutoData Workflow
```

Bridge menerima file dari Telegram, mengambil file tersebut melalui Telegram API, kemudian mengunggahnya ke Langflow agar dapat digunakan oleh workflow AutoData. Source code bridge tidak disertakan di repository ini.

## Trust Layer

AutoData dirancang agar hasil machine learning tidak disajikan hanya berdasarkan angka metric.

Sistem membandingkan model dengan baseline sederhana terlebih dahulu, kemudian menentukan quality label berdasarkan aturan evaluasi yang telah ditetapkan.

Perhitungan dilakukan di Python secara deterministik. AI Agent hanya menyampaikan dan menjelaskan hasilnya.

Feature importance juga tidak dianggap sebagai hubungan sebab-akibat, melainkan sebagai informasi mengenai kontribusi statistik fitur terhadap model.

## Responsible AI

Beberapa prinsip yang diterapkan pada prototype:

- pengguna tetap menentukan target
- pengguna dapat menerima atau mengubah rekomendasi cleaning
- hasil metric dihitung menggunakan Python
- quality label ditentukan berdasarkan aturan yang sudah ditetapkan
- baseline digunakan untuk memberikan konteks terhadap hasil model
- sistem tidak mengarang angka yang tidak tersedia
- clustering dijelaskan sebagai pola statistik
- credential dan secret tidak disimpan di repository
- Data/bank_transactions_data_edited.csv adalah data contoh

Hasil analisis tetap perlu ditinjau oleh pengguna sebelum digunakan untuk keputusan bisnis atau keputusan penting lainnya.

## Contoh Penggunaan

1. Pengguna mengirimkan file CSV melalui Telegram.
2. AutoData memeriksa kondisi awal dataset.
3. AutoData memberikan ringkasan dan rekomendasi cleaning.
4. AutoData menampilkan kandidat target.
5. Pengguna memilih target dan mengonfirmasi atau mengubah cleaning.
6. AutoData menjalankan preprocessing dan machine learning screening.
7. Hasil model dibandingkan dengan baseline.
8. AutoData memberikan ringkasan hasil dan quality label.
9. Pengguna dapat meminta laporan lengkap di Google Docs.

## Struktur Repository

```text
AutoData/
├── README.md
├── About/
│   ├── Project Overview - AutoData - Hackaton IBM x Hacktiv8.docx
│   └── Technical Documentation - Project AutoData.docx
├── Component/
│   └── agent-instructions.md
├── Data/
│   └── sales_data.csv
├── Flow/
│   └── AutoData Flows Langflow.json
├── IBM Bob/
│   └── mcp.example.json
└── Output/
    └── AutoData Analysis Report - Bank Transactions.docx
```

## Requirements

Untuk menjalankan prototype, siapkan environment dan layanan yang digunakan oleh project, termasuk:

- IBM Langflow 1.12.0
- API key model bahasa (OpenRouter atau Gemini, sesuaikan dengan yang benar-benar kamu pakai)
- API key Composio dengan koneksi Google Docs

Credential dan konfigurasi lokal tidak disimpan di repository.

## Coba AutoData

**1. Lewat bot Telegram (atas permintaan)**
Bot Telegram AutoData tidak dibagikan sebagai source code dan hanya dinyalakan atas permintaan. Hubungi saya lewat [LinkedIn / email], sebutkan waktu (beserta zona waktu) yang kamu inginkan, nanti saya nyalakan botnya dan kirim link-nya.

**2. Analisis manual lewat Playground Langflow**
  1. Import `Flow/AutoData Flows Langflow.json` ke Langflow (versi 1.12.0).
  2. Isi API key model dan koneksi Google Docs (Composio) milik kamu sendiri.
  3. Buka Playground, upload CSV lewat komponen Read File, lalu mulai chat.

**3. Integrasi IBM Bob (opsional)**
Salin `IBM Bob/mcp.example.json` ke `.bob/mcp.json`, lalu isi API key Langflow dan project ID milikmu.

## Testing

Pengujian project mencakup beberapa bagian utama:

- upload dan penerimaan file CSV
- data quality check
- rekomendasi cleaning
- target selection
- machine learning screening
- baseline comparison
- quality label
- output hasil analisis
- pembuatan Google Docs
- integrasi Langflow dan IBM Bob melalui MCP
- error handling

Bukti pengujian dan hasil run tersedia pada dokumentasi serta Contoh hasil analisis tersedia di folder Output/, dan detail pengujian ada di dokumentasi teknis.

## Limitations

AutoData masih berupa prototype dan memiliki beberapa keterbatasan:

- belum ditujukan untuk data skala production
- model machine learning masih terbatas pada beberapa model
- hasil machine learning merupakan screening awal
- quality label tidak menggantikan evaluasi model yang lebih lengkap
- clustering digunakan untuk melihat pola awal
- proses setup beberapa layanan masih membutuhkan konfigurasi manual
- belum terdapat automated retention policy untuk data dan laporan

## Security

Repository ini tidak menyimpan:

```text
API keys
Bot tokens
Passwords
Access tokens
Google credentials
Private keys
Sensitive user datasets
```

Gunakan environment variable atau konfigurasi lokal untuk menyimpan credential.

## Documentation

Dokumentasi teknis yang lebih lengkap mencakup:

- Problem dan target user
- Solution dan differentiation
- System architecture
- Workflow AutoData
- Technical implementation
- Trust Layer
- Testing dan validation
- Responsible AI
- Limitations
- MCP integration
- Environment dan configuration notes

Dokumentasi tersedia di:

```text
About/Technical Documentation - Project AutoData.docx
```

## Project Status

**Prototype**

AutoData telah mengimplementasikan workflow analisis awal dari dataset CSV, data quality check, clustering, rekomendasi cleaning dan target, machine learning screening, baseline comparison, serta optional Google Docs report.

Project ini masih memiliki keterbatasan dan belum ditujukan untuk deployment production.

## Hackathon

**Hacktiv8 x IBM SkillsBuild National Hackathon 2026**  
**Theme:** Productivity & Smart Business  
**Project:** AutoData  
**Tagline:** Dari CSV ke screening analisis lewat percakapan AI.

## Disclaimer

AutoData membantu proses screening analisis awal. Hasil analisis, clustering, machine learning, dan rekomendasi tetap perlu ditinjau oleh pengguna sebelum digunakan sebagai dasar pengambilan keputusan.

## Kontak & Lisensi

Dibuat oleh Alditia Muhamad Pirmansyah. Hubungi saya lewat
LinkedIn: https://www.linkedin.com/in/alditia-mp/
Email: aldityamufir3@gmail.com