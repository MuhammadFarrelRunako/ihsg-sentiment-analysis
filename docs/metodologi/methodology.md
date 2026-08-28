# Metodologi Proyek

## 📌 Ringkasan Eksekutif

Proyek ini mengadopsi pendekatan **quantitative research** dengan desain **time series analysis** dan **causal inference**. Data dikumpulkan dari dua sumber utama — berita ekonomi Indonesia (Kompas, CNBC Indonesia, Kontan) dan pasar saham (IHSG via Yahoo Finance) — kemudian dianalisis menggunakan teknik Natural Language Processing (NLP) dan statistik ekonometrika untuk menguji hubungan kausalitas antara sentimen media dengan pergerakan indeks saham.

---

## 🗺️ Arsitektur Alur Penelitian

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         FASE 1: DATA ACQUISITION                            │
├─────────────────────────────────────────────────────────────────────────────┤
│  📰 Sumber Berita          │  📈 Sumber Pasar Modal                        │
│  • Kompas (Ekonomi)        │  • Yahoo Finance (^JKSE)                      │
│  • CNBC Indonesia          │  • Open, High, Low, Close, Volume             │
│  • Kontan (Pasar Modal)    │  • Rentang: 1 Jan 2019 – 31 Des 2024        │
│  • Total target: >10.000   │                                               │
└─────────────────────────────────────────────────────────────────────────────┘
                                      ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                       FASE 2: DATA PREPARATION                              │
├─────────────────────────────────────────────────────────────────────────────┤
│  🧹 Text Preprocessing          │  📊 Financial Data Engineering             │
│  • Case folding                 │  • Daily Return: (Closeₜ / Closeₜ₋₁) − 1   │
│  • Regex cleaning (URL, HTML)   │  • Log Return: ln(Closeₜ / Closeₜ₋₁)       │
│  • Stopword removal (Sastrawi)  │  • Volatilitas 20H (annualized)            │
│  • Duplikasi removal            │  • RSI(14), MACD, MA(50)                   │
└─────────────────────────────────────────────────────────────────────────────┘
                                      ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                    FASE 3: NATURAL LANGUAGE PROCESSING                      │
├─────────────────────────────────────────────────────────────────────────────┤
│  🧠 Pipeline IndoBERT (Utama)              │  📏 Pipeline VADER (Baseline)   │
│  • Model: indobert-base-p1                 │  • Lexicon-based approach       │
│  • Task: Sequence Classification           │  • Compound score: [−1, +1]     │
│  • Output: POSITIVE / NEGATIVE / NEUTRAL   │  • Output: pos / neg / neu      │
│  • Confidence score: [0, 1]                │  • Perbandingan akurasi         │
└─────────────────────────────────────────────────────────────────────────────┘
                                      ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                  FASE 4: AGREGASI & MERGING DATASET                         │
├─────────────────────────────────────────────────────────────────────────────┤
│  📅 Agregasi Harian                                                         │
│  • Daily Average Sentiment Score                                            │
│  • Daily Sentiment Standard Deviation                                       │
│  • Daily News Count                                                         │
│  • Positive Ratio (proporsi berita positif)                                 │
│                                                                             │
│  🔀 Merge: Left Join IHSG (daily) ← Sentiment Index (daily) pada tanggal   │
└─────────────────────────────────────────────────────────────────────────────┘
                                      ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                   FASE 5: STATISTICAL & ECONOMETRIC ANALYSIS                │
├─────────────────────────────────────────────────────────────────────────────┤
│  1️⃣  Stasioneritas      │  2️⃣  Kointegrasi       │  3️⃣  Kausalitas       │
│  • ADF Test              │  • Johansen Test        │  • Granger Causality  │
│  • KPSS Test             │  • Trace Statistic      │  • Lag 1–5, AIC/BIC   │
│  • Differencing          │                         │  • F-test, p-value    │
│                                                                             │
│  4️⃣  VAR Model          │  5️⃣  Event Study        │  6️⃣  Regresi          │
│  • Impulse Response       │  • Window: [−5, +5]     │  • Multiple Linear    │
│  • Forecast Error Decomp  │  • CAR (Cumulative AR)   │  • R², t-statistic    │
└─────────────────────────────────────────────────────────────────────────────┘
                                      ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                    FASE 6: DELIVERABLE & REPORTING                          │
├─────────────────────────────────────────────────────────────────────────────┤
│  🖥️ Dashboard Streamlit  │  📓 Jupyter Notebooks  │  📄 Laporan Akademik   │
│  • Time series interaktif │  • Reproducible code   │  • Abstrak–Kesimpulan  │
│  • Filter periode/media   │  • EDA + NLP + Stats   │  • Format IEEE/APA     │
│  • Real-time metrics      │  • Documented steps    │  • PDF export          │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 📐 A. Metodologi Pengumpulan Data (Data Acquisition)

### A.1 Data IHSG (Market Data)

| Aspek | Spesifikasi |
|-------|-------------|
| **Sumber** | Yahoo Finance API (`yfinance`) |
| **Instrumen** | ^JKSE (Indeks Harga Saham Gabungan) |
| **Frekuensi** | Daily (OHLCV) |
| **Periode** | 1 Januari 2019 – 31 Desember 2024 |
| **Field** | Date, Open, High, Low, Close, Adj Close, Volume |
| **Derived Metrics** | Daily Return, Log Return, Volatility(20), RSI(14), MACD, MA(50) |

**Rationale:** Yahoo Finance menyediakan data historis IHSG yang bersih, gratis, dan reliable. Penggunaan adjusted close price memastikan data tidak bias akibat stock split atau corporate actions (meskipun pada indeks jarang terjadi).

### A.2 Data Berita Ekonomi (Text Data)

| Aspek | Spesifikasi |
|-------|-------------|
| **Sumber** | Kompas (kanal Ekonomi/Bisnis), CNBC Indonesia, Kontan (kanal Pasar Modal) |
| **Metode** | Web scraping (requests + BeautifulSoup), dengan fallback `news-watch` |
| **Field** | title, content, publish_date, source, url, category |
| **Filter** | Artikel yang mengandung keyword: IHSG, saham, ekonomi, BI Rate, inflasi, rupiah |
| **Target** | >10.000 artikel unik |

**Rationale:** Ketiga media merepresentasikan spektrum jurnalisme ekonomi Indonesia — Kompas (mainstream), CNBC (finansial fokus), dan Kontan (pasar modal spesifik). Kombinasi ini memberikan cakupan sentimen yang representatif.

**Etika Scraping:** Patuhi `robots.txt`, gunakan rate limiting (delay 1–2 detik antar request), dan hanya scrape konten publik. Data digunakan untuk analisis akademik non-komersial.

---

## 🧹 B. Metodologi Data Preparation

### B.1 Text Preprocessing Pipeline

Teks berita bahasa Indonesia memerlukan preprocessing khusus sebelum masuk ke model NLP:

```
Raw Text
    ↓
[1] Case Folding → ubah ke huruf kecil
    ↓
[2] Regex Cleaning → hapus URL, email, mention, hashtag, tag HTML
    ↓
[3] Normalisasi → hapus angka, tanda baca, karakter non-ASCII
    ↓
[4] Tokenisasi → pecah kalimat jadi kata-kata
    ↓
[5] Stopword Removal → hapus kata umum (dan, yang, di, dari, dll)
    ↓
[6] (Opsional) Stemming → reduksi ke bentuk dasar (Sastrawi)
    ↓
Clean Text → siap untuk IndoBERT / VADER
```

| Langkah | Library / Teknik | Keterangan |
|---------|------------------|------------|
| Case Folding | Python `str.lower()` | Menyamakan kapitalisasi |
| Regex Cleaning | `re` module | Pattern: `http\S+`, `<.*?>` , `[^a-zA-Z\s]` |
| Tokenisasi | `nltk.word_tokenize()` atau regex split | Pecah berdasarkan spasi |
| Stopword Removal | `Sastrawi.StopWordRemover` | 500+ stopword bahasa Indonesia |
| Stemming | `Sastrawi.Stemmer` | Contoh: "meningkatkan" → "tingkat" |

**Catatan:** Untuk IndoBERT, stemming bersifat **opsional** karena model BERT sudah memahami morfologi bahasa Indonesia melalui subword tokenization (WordPiece). Namun, untuk VADER, stemming dapat membantu jika menggunakan kamus custom.

### B.2 Financial Data Engineering

| Variabel | Formula | Interpretasi |
|----------|---------|--------------|
| **Daily Return** | $R_t = \frac{Close_t - Close_{t-1}}{Close_{t-1}}$ | Profit/loss harian dalam persen |
| **Log Return** | $r_t = \ln(\frac{Close_t}{Close_{t-1}})$ | Return yang additive dan lebih stasioner |
| **Volatilitas (20H)** | $\sigma_{20} = \text{std}(r_t, r_{t-1}, ..., r_{t-19}) \times \sqrt{252}$ | Risiko pergerakan IHSG, annualized |
| **RSI(14)** | $100 - \frac{100}{1 + RS}$ | Momentum overbought (>70) / oversold (<30) |
| **MACD** | $EMA_{12} - EMA_{26}$ | Sinyal tren bullish/bearish |

### B.3 Handling Missing Data & Calendar Alignment

| Skenario | Solusi |
|----------|--------|
| Weekend / Hari Libur Bursa | IHSG tidak ada data → forward fill atau exclude |
| Hari tanpa berita | Sentiment index = 0 atau interpolasi linear |
| Duplikat antar media | Deduplikasi berdasarkan fuzzy matching judul + tanggal |
| Missing value di derived metrics | Drop baris (karena rolling window) atau backward fill |

---

## 🧠 C. Metodologi Natural Language Processing (NLP)

### C.1 Pendekatan Utama: IndoBERT

**Model:** `indobenchmark/indobert-base-p1`  
**Arsitektur:** BERT (Bidirectional Encoder Representations from Transformers)  
**Pre-training:** Corpus bahasa Indonesia (Wikipedia, news, social media)  
**Fine-tuning:** Sequence Classification 3 kelas (POSITIVE, NEGATIVE, NEUTRAL)

**Mekanisme Kerja:**

```
Input: "IHSG ditutup menguat dipicu sentimen positif pasar global"
    ↓
Tokenisasi WordPiece: ["ihsg", "di", "tutup", "##up", "..."]
    ↓
12 Layers Transformer Encoder (self-attention)
    ↓
[CLS] token representation → Classifier Layer
    ↓
Output: POSITIVE (confidence: 0.94)
```

**Kelebihan IndoBERT:**
- Memahami **konteks dwiarah** (bidirectional) — beda dengan LSTM yang hanya satu arah
- Mengenal **nuansa bahasa Indonesia** — idiom, istilah pasar modal, konotasi positif/negatif
- **Subword tokenization** menangani kata tidak baku (e.g., "nge-drop", "bullish")

**Keterbatasan:**
- Memerlukan **GPU** untuk inference batch besar (10.000+ artikel)
- Inference time lebih lambat dibanding lexicon-based
- Tidak ada model pre-trained sentiment khusus bahasa Indonesia yang umum → perlu fine-tuning sendiri atau gunakan zero-shot

**Solusi Praktis:**
Gunakan **Google Colab (T4 GPU gratis)** untuk inference IndoBERT pada 10.000+ artikel. Simpan hasil ke CSV, lalu lanjutkan analisis di lokal.

### C.2 Pendekatan Baseline: VADER

**Model:** Valence Aware Dictionary and sEntiment Reasoner  
**Tipe:** Lexicon & rule-based  
**Output:** Compound score [−1, +1], plus breakdown pos/neu/neg

**Mekanisme Kerja:**

```
Input: "IHSG anjlok akibat kekhawatiran resesi global"
    ↓
Lexicon Lookup: "anjlok" = −0.8, "kekhawatiran" = −0.5, "resesi" = −0.9
    ↓
Rule-based Adjustment: intensifier ("sangat"), negation ("tidak")
    ↓
Output: compound = −0.85 (highly negative)
```

**Kelebihan VADER:**
- **Cepat** — ribuan artikel selesai dalam detik
- **Tidak butuh training** — plug and play
- **Mengenal intensifier & negasi** — "sangat bagus" vs "tidak bagus"

**Keterbatasan untuk Bahasa Indonesia:**
- Lexicon bawaan adalah **bahasa Inggris**
- Akurasi pada teks Indonesia mentah: **~60–75%** (studies menunjukkan perlu translate atau lexicon custom)
- Tidak memahami konteks panjang atau istilah pasar modal spesifik Indonesia

**Strategi dalam Proyek Ini:**
- Gunakan VADER sebagai **baseline** — hasilnya jadi patokan minimum
- Bandingkan dengan IndoBERT — selisih akurasi menunjukkan nilai tambah model bahasa Indonesia
- Validasi manual pada 200 sampel untuk hitung precision/recall masing-masing

### C.3 Agregasi Sentimen Harian

Setiap artikel punya skor sentimen. Untuk analisis time series, perlu diagregasi per hari:

| Metrik Agregasi | Formula | Interpretasi |
|-----------------|---------|--------------|
| **Avg Sentiment** | $\bar{S}_t = \frac{1}{n} \sum_{i=1}^{n} S_{i,t}$ | Sentimen rata-rata hari itu |
| **Sentiment Std** | $\sigma_{S,t} = \sqrt{\frac{1}{n-1} \sum (S_{i,t} - \bar{S}_t)^2}$ | Ketidakpastian / polarisasi berita |
| **News Count** | $N_t = \text{count}(\text{artikel pada tanggal } t)$ | Volume informasi |
| **Positive Ratio** | $P_t = \frac{\text{count}(positive)}{N_t}$ | Proporsi berita optimis |

---

## 📊 D. Metodologi Statistik & Ekonometrika

### D.1 Uji Stasioneritas (Unit Root Test)

**Mengapa penting:** Granger Causality dan VAR mensyaratkan data **stasioner** (mean & variance konstan sepanjang waktu). Harga saham umumnya **non-stasioner**, sehingga perlu diubah ke return atau differencing.

| Uji | Hipotesis Nol | Keputusan |
|-----|---------------|-----------|
| **ADF (Augmented Dickey-Fuller)** | Data memiliki unit root (non-stasioner) | p < 0.05 → Tolak H0 → Stasioner |
| **KPSS** | Data stasioner | p < 0.05 → Terima H0 → Non-stasioner |

**Prosedur:**
1. Uji ADF pada IHSG Close → kemungkinan non-stasioner
2. Uji ADF pada IHSG Log Return → harusnya stasioner
3. Uji ADF pada Daily Average Sentiment → cek apakah perlu differencing
4. Jika non-stasioner → first differencing: $\Delta y_t = y_t - y_{t-1}$

### D.2 Granger Causality Test

**Tujuan:** Menguji apakah sentimen berita secara statistik **membantu memprediksi** pergerakan IHSG lebih baik daripada hanya menggunakan data historis IHSG sendiri.

**Model:**

$$R_t = \alpha_0 + \sum_{i=1}^{p} \alpha_i R_{t-i} + \sum_{j=1}^{q} \beta_j S_{t-j} + \epsilon_t$$

Dimana:
- $R_t$ = Return IHSG pada hari t
- $S_t$ = Average Sentiment pada hari t
- $p, q$ = lag order (1–5 hari)

**Hipotesis:**
- **H0:** $\beta_1 = \beta_2 = ... = \beta_q = 0$ → Sentimen **tidak** Granger-menyebabkan IHSG
- **H1:** Minimal satu $\beta_j \neq 0$ → Sentimen **Granger-menyebabkan** IHSG

**Prosedur:**
1. Tentukan lag order optimal via **AIC** (Akaike Information Criterion) atau **BIC**
2. Estimasi model VAR/VECM pada data stasioner
3. Jalankan F-test untuk setiap lag
4. Interpretasi: p-value < 0.05 → Tolak H0 → Ada Granger Causality

**Catatan Penting:**
- Granger Causality ≠ kausalitas sebenarnya. Ini hanya menunjukkan **prediktivitas temporal**.
- Hasil bisa **bidirectional** — IHSG juga bisa Granger-menyebabkan sentimen (feedback loop).

### D.3 Vector Autoregression (VAR)

**Tujuan:** Model multivariate yang memperlakukan **kedua variabel sebagai endogen** — IHSG mempengaruhi sentimen, dan sentimen mempengaruhi IHSG.

**Model:**

$$\begin{bmatrix} R_t \\ S_t \end{bmatrix} = \begin{bmatrix} c_1 \\ c_2 \end{bmatrix} + \sum_{k=1}^{p} \begin{bmatrix} a_{11,k} & a_{12,k} \\ a_{21,k} & a_{22,k} \end{bmatrix} \begin{bmatrix} R_{t-k} \\ S_{t-k} \end{bmatrix} + \begin{bmatrix} \epsilon_{1,t} \\ \epsilon_{2,t} \end{bmatrix}$$

**Analisis Lanjutan dari VAR:**

| Teknik | Fungsi |
|--------|--------|
| **Impulse Response Function (IRF)** | Melihat respons IHSG terhadap "shock" sentimen 1 standard deviation |
| **Forecast Error Variance Decomposition (FEVD)** | Seberapa besar kontribusi sentimen dalam meramalkan IHSG |
| **Granger Causality dari VAR** | Uji F pada koefisien cross-equation |

### D.4 Event Study

**Tujuan:** Mengukur dampak spesifik peristiwa ekonomi makro terhadap IHSG dan melihat apakah sentimen berita memperkuat atau memperlemah efek tersebut.

**Peristiwa yang Diamati:**
- Pengumuman kebijakan suku bunga (BI Rate) oleh Bank Indonesia
- Pengumuman inflasi oleh BPS
- Pengumuman PDB / pertumbuhan ekonomi
- Peristiwa global (COVID-19, kenaikan Fed Rate, dll)

**Window Definition:**

| Window | Rentang | Fungsi |
|--------|---------|--------|
| **Estimation Window** | $[-120, -11]$ hari sebelum event | Estimasi expected return (normal) |
| **Event Window** | $[-5, +5]$ hari sekitar event | Mengukur abnormal return |
| **Post-Event** | $[+6, +30]$ hari | Melihat persistensi efek |

**Perhitungan:**

$$AR_{i,t} = R_{i,t} - E[R_{i,t}]$$

$$CAR_i = \sum_{t=-5}^{+5} AR_{i,t}$$

Dimana expected return $E[R_{i,t}]$ diestimasi dari **market model** selama estimation window:

$$E[R_{i,t}] = \hat{\alpha} + \hat{\beta} R_{m,t}$$

**Uji Signifikansi:**
- t-test pada CAR: $t = \frac{CAR}{\sigma_{CAR} / \sqrt{n}}$
- Jika $|t| > t_{\alpha/2, n-1}$ → CAR signifikan

### D.5 Regresi & Korelasi

| Metode | Persamaan | Tujuan |
|--------|-----------|--------|
| **Pearson Correlation** | $\rho = \frac{\text{Cov}(S, R)}{\sigma_S \sigma_R}$ | Uji hubungan linear |
| **Spearman Correlation** | Rank-based $\rho$ | Uji hubungan monotonic (non-linear) |
| **Multiple Regression** | $R_t = \beta_0 + \beta_1 S_t + \beta_2 V_t + \beta_3 \sigma_t + \epsilon_t$ | Kontrol variabel (volume, volatilitas) |

---

## 🖥️ E. Metodologi Dashboard Development

### E.1 Arsitektur Streamlit

```
app.py (Entry Point)
    ├── Sidebar: Filter kontrol
    │       ├── Rentang tanggal
    │       ├── Pilih media berita
    │       └── Pilih metode sentimen (IndoBERT / VADER)
    │
    └── Main Content
            ├── Tab 1: Overview (KPI + time series)
            ├── Tab 2: Sentiment Analysis (distribusi + word cloud)
            ├── Tab 3: IHSG Analysis (candlestick + indikator teknikal)
            ├── Tab 4: Correlation & Causality (heatmap + Granger results)
            └── Tab 5: Event Study (CAR visualization)
```

### E.2 Komponen Visualisasi

| Komponen | Library | Fungsi |
|----------|---------|--------|
| Time Series Line Chart | `plotly.express.line()` | IHSG vs Sentiment over time |
| Candlestick Chart | `plotly.graph_objects.Candlestick()` | OHLC IHSG |
| Heatmap | `seaborn.heatmap()` | Matriks korelasi |
| Word Cloud | `wordcloud.WordCloud()` | Kata dominan per sentimen |
| Distribution Plot | `plotly.histogram()` | Distribusi skor sentimen |
| Metrics Card | `st.metric()` | KPI real-time (latest IHSG, avg sentiment) |

---

## 📋 F. Validasi & Robustness Check

| Aspek | Metode | Kriteria Keberhasilan |
|-------|--------|----------------------|
| **Validasi NLP** | Manual labeling 200 sampel acak | Accuracy IndoBERT > 80%, VADER > 60% |
| **Stasioneritas** | ADF + KPSS | Semua variabel stasioner pada level atau first difference |
| **Autokorelasi** | Durbin-Watson / Ljung-Box | Tidak ada autokorelasi residual |
| **Heteroskedastisitas** | ARCH-LM test | Tidak ada heteroskedastisitas, atau gunakan robust SE |
| **Outlier** | Z-score / IQR | Identifikasi & sensitivity analysis |
| **Lag Robustness** | Uji Granger pada lag 1, 2, 3, 5 | Hasil konsisten di beberapa lag |

---

## 🎯 G. Ringkasan Keputusan Metodologi

| Aspek | Keputusan | Alasan |
|-------|-----------|--------|
| **NLP Utama** | IndoBERT | Akurasi tinggi untuk bahasa Indonesia, memahami konteks pasar modal |
| **NLP Baseline** | VADER | Perbandingan cepat, benchmark minimum |
| **Statistik Utama** | Granger Causality | Uji prediktivitas temporal, gold standard untuk time series |
| **Model Multivariate** | VAR | Melihat hubungan timbal balik + impulse response |
| **Event Analysis** | Event Study | Mengukur dampak spesifik kebijakan moneter |
| **Dashboard** | Streamlit | Python-native, gratis, deploy mudah, cukup untuk portofolio |
| **Periode** | 2019–2024 | Mencakup COVID-19, pemulihan, dan volatilitas tinggi |

---

## 📚 Referensi Metodologi

1. Mukherjee, S. (2025). *Investor Sentiment and Market Movements: A Granger Causality Perspective*. Journal of Behavioral Finance.
2. Rihaadatul'Aisy, N., & Pamungkas, D. S. (2025). *Sentiment Analysis of Indonesian News Texts Using IndoBERT and IndoRoBERTa*. IEEE Access.
3. Gujarati, D. N., & Porter, D. C. (2021). *Basic Econometrics* (6th ed.). McGraw-Hill. (Bab 22: Time Series Econometrics)
4. Hutto, C. J., & Gilbert, E. (2014). *VADER: A Parsimonious Rule-based Model for Sentiment Analysis of Social Media Text*. ICWSM.
5. Koto, F., & Rahmaningtyas, G. (2020). *IndoBERT: Pre-trained Model for Indonesian Language Understanding*. EMNLP.

---

*Dokumen ini merupakan pedoman teknis yang dapat direvisi seiring berjalannya proyek. Semua keputusan metodologi didasarkan pada best practices dalam financial econometrics dan NLP untuk bahasa Indonesia.*
