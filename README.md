# JCDSAH-026_Beta
# Prediksi Subscription Term Deposit — Bank Marketing Campaign

Model klasifikasi untuk memprediksi nasabah mana yang akan subscribe term deposit, sekaligus analisis kondisi kampanye yang efektif — dasar penyusunan call list terprioritas untuk tim Campaign Manager.

## Problem Statement

Dari 41.176 panggilan telemarketing, hanya 11,3% berakhir dengan subscription. Masalahnya bukan kurang volume panggilan, melainkan menelepon orang yang kurang tepat, pada waktu yang kurang tepat, lewat channel yang kurang efektif. Proyek ini menjawab: **bagaimana memprediksi nasabah mana yang akan subscribe, dan kondisi apa yang membuat kampanye efektif**, sehingga call list bisa diprioritaskan dan panggilan sia-sia berkurang tanpa kehilangan subscription.

## Dataset

`bank-additional-full.csv` (Moro, Cortez & Rita, 2014) — 41.188 baris, Mei 2008–Nov 2010. Kolom tahun kalender tidak tersedia di data asli, tapi berhasil direkonstruksi dari pola kuartalan `nr.employed`: conversion rate naik dari **4,84% (2008) → 19,48% (2009) → 52,14% (2010)**, mengikuti jatuhnya suku bunga euribor (korelasi -0,886). Temuan ini jadi dasar seluruh analisis: beberapa pola populer di dataset ini (mis. keunggulan nasabah usia lanjut) ternyata cuma berlaku kondisional pada periode suku bunga rendah, bukan aturan permanen.

## Pendekatan

1. Rekonstruksi dimensi waktu + EDA (distribusi, korelasi, missing value, deteksi data leakage pada kolom `duration`)
2. Preprocessing: tangani sentinel `pdays=999`, pertahankan kategori `unknown` sebagai sinyal, 18 fitur model final
3. Analisis deskriptif + 4 uji hipotesis (chi-square, Mann-Whitney, Cochran-Armitage trend test)
4. Modeling: baseline → Logistic Regression vs Gradient Boosting, tuning via `RandomizedSearchCV`, dua skema validasi (stratified CV + **temporal split**: latih 2008-2009, uji 2010)
5. Kuantifikasi dampak finansial (3 skenario biaya/margin ilustratif)

## Hasil Utama

- Model final: **Gradient Boosting**, PR-AUC 0,64 pada data uji 2010 (temporal split)
- Ambang batas 0,5 tidak dipakai — distribution shift antar tahun membuatnya tidak bermakna; keputusan call list pakai **kurva lift/ranking** skor
- Indikator makro (`emp_var_rate`, `euribor3m`, dll) menyumbang **68% feature importance** — model lebih banyak menangkap "kapan waktunya tepat" dibanding "siapa nasabahnya"

**Rekomendasi (dampak skenario tengah, dari 41.176 panggilan historis):**

| Rekomendasi | Dampak | Status |
|---|---|---|
| Batasi maksimum 7 panggilan/nasabah | +€21.602 | Kuat |
| Migrasi telephone → cellular | +€40.927 | Kuat |
| Sesuaikan intensitas kampanye mengikuti suku bunga | — (strategis) | Kuat |
| Call list berbasis skor model | +€21.602 s/d -€75.659 | **Kondisional** — hanya untung saat conversion rate dasar rendah |
| Prioritaskan usia 60+ & `poutcome=success` | tercakup di atas | Robust arah, kondisional besaran |
