# Scope Proyek

## Judul
Analisis Sentimen Berita Ekonomi vs Pergerakan IHSG (2019–2024)

## Pertanyaan Riset
Apakah sentimen berita ekonomi dari Kompas, CNBC Indonesia, dan Kontan 
memiliki hubungan kausalitas dengan pergerakan IHSG periode 2019–2024?

## Batasan Proyek
| Aspek | Batasan |
|-------|---------|
| Periode data | 1 Jan 2019 – 31 Des 2024 |
| Media berita | Kompas, CNBC Indonesia, Kontan |
| Kanal berita | Ekonomi, Bisnis, Pasar Modal, Moneter |
| Variabel IHSG | Open, High, Low, Close, Volume, Return |
| Metode NLP | IndoBERT (utama) + VADER (baseline) |
| Metode statistik | Granger Causality, VAR, Event Study, Korelasi |

## Risiko & Mitigasi
| Risiko | Mitigasi |
|--------|----------|
| Website berita berubah struktur HTML | Simpan HTML sample, gunakan news-watch |
| Data 5 tahun terlalu besar | Mulai dari 1 tahun dulu |
| IndoBERT lambat tanpa GPU | Gunakan Google Colab GPU gratis |
| IHSG tidak stasioner | Siapkan differencing (return log) |
| Berita duplikat antar media | Deduplikasi berdasarkan judul + tanggal |

## Success Criteria
- [ ] Dataset berita &gt; 10.000 artikel
- [ ] Minimal 1 hipotesis terbukti signifikan (p &lt; 0.05)
- [ ] Dashboard Streamlit berjalan lancar
- [ ] Laporan akademik lengkap (Abstrak – Kesimpulan)