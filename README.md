# Studio Proyek 3 – Fine-Tuning Transformer Pretrained untuk Klasifikasi Sentimen Bahasa Indonesia (SmSA)

**Mata kuliah:** Pembelajaran Mendalam (IF25-40401) – Teknik Informatika, Institut Teknologi Sumatera
**Kelompok:** 16

| No | Nama | NIM |
|----|------|-----|
| 1 | Silvia | 123140133 |
| 2 | Tri Putri Sormin | 123140013 |

## Ringkasan

Repositori ini berisi eksperimen fine-tuning model Transformer pretrained untuk klasifikasi sentimen teks Bahasa Indonesia (negatif, netral, positif) pada dataset **SmSA** dari benchmark IndoNLU.

Eksperimen membandingkan **4 backbone × 3 strategi × 3 seed = 36 run**:

- **Backbone:** mBERT (`bert-base-multilingual-cased`), DistilBERT multilingual (`distilbert-base-multilingual-cased`), IndoBERT (`indobenchmark/indobert-base-p1`), XLM-R (`xlm-roberta-base`)
- **Strategi:** full fine-tuning, partial freezing (embedding + separuh lapis awal), linear probing. Head klasifikasi diganti menjadi 3 kelas di semua run.
- **Seed:** 42, 123, 2026

## Protokol eksperimen (identik di semua run)

| Variabel | Nilai |
|----------|-------|
| Data latih | 6.000 sampel, stratified per kelas (seed 42) |
| Data validasi | Seluruh split resmi setelah pembersihan (1.246 baris), hanya untuk memilih epoch terbaik |
| Data uji | Seluruh uji resmi (500 kalimat), dievaluasi sekali per run |
| max_length | 128 |
| Batch size | 32 (latih), 64 (evaluasi) |
| Epoch | 3 |
| Learning rate | full 2e-5, partial 2e-5, probing 1e-3 |
| Optimizer | AdamW, weight decay 0,01, warmup linear 10%, grad clip 1,0 |
| Pembersihan data | Buang duplikat, teks berlabel bertentangan, dan teks validasi yang bocor ke data latih |

Kode konfigurasi eksperimen: `c7e6d734`.

## Temuan utama

Semua angka berupa rata-rata ± simpangan baku antar-seed. Nilai persis ada di berkas CSV; angka di bawah dibulatkan.

- **IndoBERT full fine-tuning** memberi F1-makro validasi tertinggi (sekitar 0,91). **XLM-R full** hampir sama baiknya, dengan selisih di dalam simpangan antar-seed, tetapi parameternya lebih dari dua kali lipat.
- **mBERT** dan **DistilBERT** tertinggal dari IndoBERT. DistilBERT adalah yang tercepat saat inferensi (sekitar 4–5 ms/kalimat, dibanding sekitar 8–10 ms untuk model 12 lapis).
- **Partial freezing** pada IndoBERT hampir setara dengan full fine-tuning dengan waktu latih lebih singkat. Pada backbone lain partial turun secara bermakna.
- **Linear probing** selalu di bawah full fine-tuning. Pada mBERT, hasilnya mendekati tebakan acak.
- **Rekomendasi:** IndoBERT dengan partial freezing sebagai keseimbangan akurasi dan biaya latih. Alternatif dan alasannya ada di bagian Kesimpulan notebook.

## Cara menjalankan ulang

1. Buka notebook di Google Colab atau Kaggle dan aktifkan GPU (T4).
2. Jalankan sel secara berurutan dari atas. Dataset SmSA diunduh otomatis dari repositori IndoNLU.
3. Seed dikunci lewat `set_seed()`, jadi hasil seharusnya konsisten. Latensi inferensi bisa berbeda tipis antar-sesi atau antar-mesin.
4. Matriks penuh memakan waktu sekitar 2 jam di satu GPU T4. Pekerjaan bisa dibagi antar sesi dengan mengatur `MEMBER_TAG` dan `RUN_BACKBONES`, lalu digabung dengan sel "Gabungkan hasil antar-anggota".


