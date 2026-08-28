# Hipotesis Proyek

## H1: Korelasi Sentimen & IHSG Return
- **H0**: Tidak ada korelasi signifikan (ρ = 0)
- **H1**: Ada korelasi positif signifikan (ρ &gt; 0)
- **Metode**: Pearson/Spearman Correlation Test
- **α**: 0.05

## H2: Granger Causality
- **H0**: Sentimen tidak Granger-menyebabkan IHSG
- **H1**: Sentimen Granger-menyebabkan IHSG (lag 1–3 hari)
- **Metode**: Granger Causality Test (F-test)
- **α**: 0.05

## H3: Event Study (BI Rate)
- **H0**: Tidak ada abnormal return signifikan (CAR = 0)
- **H1**: Ada abnormal return signifikan pada event window [-5, +5]
- **Metode**: Event Study dengan t-test
- **α**: 0.05

## H4: Perbandingan NLP
- **H0**: Akurasi IndoBERT = Akurasi VADER
- **H1**: Akurasi IndoBERT &gt; Akurasi VADER
- **Metode**: Manual validation 200 sampel
- **α**: 0.05