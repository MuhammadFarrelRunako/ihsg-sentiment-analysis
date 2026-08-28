# Metodologi

## NLP Pipeline
1. **Preprocessing**: Case folding, regex cleaning, stopword removal (Sastrawi)
2. **IndoBERT**: `indobenchmark/indobert-base-p1` → 3 kelas sentiment
3. **VADER**: Compound score sebagai baseline
4. **Agregasi**: Rata-rata skor sentimen harian

## Statistical Pipeline
1. **ADF Test**: Cek stasioneritas
2. **Differencing**: Jika data non-stasioner
3. **Granger Causality**: Lag 1–5, pilih optimal via AIC
4. **VAR Model**: Impulse Response Function
5. **Event Study**: CAR analysis pada pengumuman BI Rate

## Tools
- Python 3.10+, VS Code, Google Colab (GPU)
- Streamlit untuk dashboard
- Git + GitHub untuk version control

## Flowchart Sederhana