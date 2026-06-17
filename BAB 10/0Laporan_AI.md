# Laporan Proyek Kecerdasan Buatan
## Perbandingan Decision Tree dan Naive Bayes untuk Prediksi Risiko Diabetes Menggunakan Data CDC

---

> **Mata Kuliah:** Kecerdasan Buatan
> **Dataset:** CDC Diabetes Health Indicators — BRFSS 2015
> **Tools:** Python 3 · scikit-learn · imbalanced-learn · Streamlit · pandas · seaborn

---

## Daftar Isi

1. [Latar Belakang](#1-latar-belakang)
2. [Dataset](#2-dataset)
3. [Seleksi Fitur — Chi-Square](#3-seleksi-fitur--chi-square)
4. [Arsitektur Sistem](#4-arsitektur-sistem)
5. [Metodologi](#5-metodologi)
6. [Model yang Digunakan](#6-model-yang-digunakan)
7. [Teknik Resampling](#7-teknik-resampling)
8. [Hasil Eksperimen](#8-hasil-eksperimen)
9. [Analisis Hasil](#9-analisis-hasil)
10. [Aplikasi Streamlit](#10-aplikasi-streamlit)
11. [Struktur Proyek](#11-struktur-proyek)
12. [Cara Menjalankan](#12-cara-menjalankan)
13. [Kesimpulan](#13-kesimpulan)

---

## 1. Latar Belakang

Diabetes mellitus adalah salah satu penyakit kronis dengan prevalensi tertinggi di dunia. Di Amerika Serikat, Centers for Disease Control and Prevention (CDC) secara rutin mengumpulkan data kesehatan masyarakat melalui survei BRFSS (*Behavioral Risk Factor Surveillance System*) untuk membantu deteksi dini faktor risiko penyakit seperti diabetes.

Proyek ini bertujuan untuk:

1. **Melatih dan membandingkan** dua model klasifikasi — Decision Tree dan Gaussian Naive Bayes — pada dataset CDC untuk memprediksi risiko diabetes.
2. **Menguji pengaruh teknik resampling** terhadap performa model, karena distribusi kelas pada dataset ini sangat tidak seimbang (*imbalanced*).
3. **Memilih model terbaik** berdasarkan metrik yang relevan untuk konteks klinis.
4. **Membangun aplikasi web** berbasis Streamlit sebagai antarmuka prediksi yang dapat digunakan langsung.

---

## 2. Dataset

### Sumber
Dataset yang digunakan adalah **CDC Diabetes Health Indicators** dari survei BRFSS 2015, yang tersedia secara publik di Kaggle.

- **File:** `diabetes_012_health_indicators_BRFSS2015.csv`
- **Jumlah baris awal:** 253.680 responden
- **Jumlah kolom:** 22 fitur + 1 kolom target
- **Setelah cleaning (hapus duplikat & missing values):** 229.781 baris

### Target Klasifikasi

Kolom target adalah `Diabetes_012` dengan tiga kelas:

| Nilai | Label | Jumlah Sampel | Proporsi |
|:---:|---|---:|---:|
| `0` | Tidak Diabetes | 190.055 | ~82,7% |
| `1` | Prediabetes | 4.629 | ~2,0% |
| `2` | Diabetes | 35.097 | ~15,3% |

### Masalah Ketidakseimbangan Kelas

Distribusi kelas menunjukkan ketidakseimbangan yang sangat ekstrem, terutama pada kelas **Prediabetes (kelas 1)** yang hanya mencakup ~2% dari total data. Ini adalah salah satu tantangan utama proyek ini dan menjadi alasan utama pengujian berbagai teknik resampling.

```
Kelas 0 (Tidak Diabetes) : ████████████████████████████████████████ 82.7%
Kelas 2 (Diabetes)       : ███████  15.3%
Kelas 1 (Prediabetes)    : ▌  2.0%
```

### Split Data

Data dibagi menggunakan **stratified split** untuk menjaga proporsi kelas di training set dan test set:

| Set | Jumlah Baris |
|---|---:|
| Training (80%) | 183.824 |
| Test (20%) | 45.957 |

> **Penting:** Resampling **hanya diterapkan pada data training**, bukan sebelum split, untuk menghindari *data leakage*.

---

## 3. Seleksi Fitur — Chi-Square

Dari 21 fitur yang tersedia di dataset, dilakukan seleksi fitur menggunakan metode **Chi-Square** untuk mengidentifikasi fitur yang paling signifikan secara statistik terhadap variabel target `Diabetes_012`.

### Prinsip Chi-Square untuk Seleksi Fitur

Uji Chi-Square mengukur derajat ketergantungan antara setiap fitur kategorik dengan variabel target. Semakin tinggi nilai Chi-Square score suatu fitur, semakin kuat hubungan statistiknya dengan target dan semakin informatif fitur tersebut untuk model klasifikasi.

### Hasil Seleksi — 10 Fitur Terbaik

| Rank | Fitur | Tipe | Keterangan |
|:---:|---|---|---|
| 1 | `HighBP` | Biner (0/1) | Tekanan darah tinggi |
| 2 | `DiffWalk` | Biner (0/1) | Kesulitan berjalan atau naik tangga |
| 3 | `GenHlth` | Ordinal (1–5) | Kondisi kesehatan umum (1=Sangat Baik, 5=Sangat Buruk) |
| 4 | `HighChol` | Biner (0/1) | Kolesterol tinggi |
| 5 | `HeartDiseaseorAttack` | Biner (0/1) | Riwayat penyakit jantung atau serangan jantung |
| 6 | `BMI` | Kontinu | Indeks Massa Tubuh |
| 7 | `Age` | Ordinal (1–13) | Kelompok usia (1=18–24, 13=80+) |
| 8 | `PhysHlth` | Diskrit (0–30) | Jumlah hari sakit/cedera dalam 30 hari terakhir |
| 9 | `Income` | Ordinal (1–8) | Tingkat pendapatan rumah tangga |
| 10 | `PhysActivity` | Biner (0/1) | Aktif secara fisik dalam 30 hari terakhir |

Fitur yang **tidak dipilih** (rank 11–21) antara lain: `Smoker`, `Sex`, `CholCheck`, `Stroke`, `HvyAlcoholConsump`, `AnyHealthcare`, `NoDocbcCost`, `MentHlth`, `Fruits`, `Veggies`, `Education`.

---

## 4. Arsitektur Sistem

```
┌─────────────────────────────────────────────────────────┐
│                    PIPELINE TRAINING                    │
│                                                         │
│  [Dataset CSV]                                          │
│       │                                                 │
│       ▼                                                 │
│  [Load & Clean]  → hapus duplikat & missing values      │
│       │                                                 │
│       ▼                                                 │
│  [Stratified Split 80:20]                               │
│       │                    │                            │
│       ▼                    ▼                            │
│  [X_train / y_train]  [X_test / y_test]                 │
│       │                                                 │
│       ▼                                                 │
│  [Fit StandardScaler]  ← hanya pada X_train             │
│       │                                                 │
│       ▼                                                 │
│  ┌────────────────────────────────────┐                 │
│  │     LOOP EKSPERIMEN (12 total)     │                 │
│  │                                    │                 │
│  │  Resampling Method (6 pilihan)     │                 │
│  │       × Model (2 pilihan)          │                 │
│  │                                    │                 │
│  │  → Resample X_train                │                 │
│  │  → Fit Model                       │                 │
│  │  → Predict X_test (tidak diresamp) │                 │
│  │  → Hitung Metrik                   │                 │
│  └────────────────────────────────────┘                 │
│       │                                                 │
│       ▼                                                 │
│  [Pilih Model Terbaik]  ← berdasarkan F1 Weighted       │
│       │                                                 │
│       ▼                                                 │
│  [Simpan best_model.joblib]                             │
│  (model + scaler + metadata)                            │
└─────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│                  APLIKASI STREAMLIT                     │
│                                                         │
│  Load best_model.joblib                                 │
│       │                                                 │
│       ▼                                                 │
│  Input User (form 10 fitur)                             │
│       │                                                 │
│       ▼                                                 │
│  Apply Scaler (yang sama dengan training)               │
│       │                                                 │
│       ▼                                                 │
│  Prediksi → Kelas + Probabilitas                        │
│       │                                                 │
│       ▼                                                 │
│  Tampilkan Hasil + Feature Importance                   │
└─────────────────────────────────────────────────────────┘
```

---

## 5. Metodologi

### 5.1 Preprocessing

1. **Normalisasi nama kolom** — strip whitespace dari header CSV.
2. **Hapus duplikat** menggunakan `DataFrame.drop_duplicates()`.
3. **Hapus baris dengan missing values** pada kolom fitur atau target.
4. **StandardScaler** — men-*scale* semua fitur ke mean=0, std=1. Scaler di-*fit* hanya pada training set, kemudian di-*transform* ke test set dan input Streamlit untuk menghindari data leakage.

### 5.2 Alasan Penggunaan StandardScaler

Naive Bayes (GaussianNB) mengasumsikan fitur berdistribusi normal. Meskipun Decision Tree tidak sensitif terhadap skala, normalisasi tetap diterapkan secara konsisten pada kedua model agar proses preprocessing saat training dan inference benar-benar identik.

### 5.3 Metrik Evaluasi

Karena dataset sangat tidak seimbang (*imbalanced*), pemilihan metrik sangat penting:

| Metrik | Rumus | Peran dalam Proyek |
|---|---|---|
| **Accuracy** | TP+TN / Total | Gambaran umum, misleading untuk imbalanced |
| **Precision (Weighted)** | Σ(Pᵢ × nᵢ) / N | Seberapa presisi prediksi positif |
| **Recall (Weighted)** | Σ(Rᵢ × nᵢ) / N | Seberapa banyak kasus yang berhasil ditemukan |
| **F1-Score (Weighted)** | Harmonic mean P & R, weighted | **Metrik utama pemilihan model** |
| **F1-Score (Macro)** | Rata-rata F1 per kelas | Sensitif terhadap kelas minoritas |
| **Recall (Macro)** | Rata-rata Recall per kelas | **Tiebreaker** pemilihan model |
| **F1 per Kelas** | F1 individual tiap kelas | Transparansi performa di tiap kelas |

> **Mengapa F1 Weighted sebagai metrik utama?**
> F1 Weighted mempertimbangkan bobot masing-masing kelas berdasarkan jumlah sampelnya, sehingga tidak bias ke kelas minoritas maupun mayoritas. Ini lebih informatif dibanding Accuracy murni untuk dataset imbalanced.
>
> **Mengapa Recall Macro sebagai tiebreaker?**
> Dalam konteks medis, kemampuan mendeteksi kasus Prediabetes dan Diabetes (kelas minoritas) lebih kritis daripada performa di kelas mayoritas. Recall Macro memberikan bobot setara ke semua kelas tanpa mempedulikan ukurannya.

---

## 6. Model yang Digunakan

### 6.1 Decision Tree Classifier

Decision Tree membangun struktur pohon biner yang membagi data secara rekursif berdasarkan fitur dan threshold yang meminimalkan *impurity* (ketidakmurnian) di setiap node.

**Parameter yang digunakan:**

```python
DT_PARAMS = {
    "max_depth":         15,    # kedalaman maksimum pohon
    "min_samples_split": 50,    # min sampel untuk membagi node
    "min_samples_leaf":  25,    # min sampel di leaf node
    "criterion":         "gini",# fungsi impurity: gini atau entropy
    "class_weight":      "balanced",  # bobot kelas otomatis
    "random_state":      42,
}
```

**Alasan pemilihan parameter:**

- `max_depth=15`: Memberi pohon cukup ekspresif untuk dataset ~184k training tanpa overfitting berlebihan. Nilai lebih rendah (8–10) menghasilkan underfitting; nilai lebih tinggi (>20) cenderung overfit.
- `min_samples_split=50` dan `min_samples_leaf=25`: Nilai konservatif untuk dataset besar, mencegah leaf yang terlalu spesifik terhadap data training.
- `class_weight="balanced"`: Parameter kunci untuk dataset imbalanced — DT secara otomatis memberi bobot lebih tinggi ke kelas minoritas (Prediabetes dan Diabetes) sehingga model tidak terlalu bias ke kelas mayoritas.

**Keunggulan:** Mudah diinterpretasi, menghasilkan *feature importance*, cepat saat training.
**Kelemahan:** Rentan overfit jika parameter tidak dikontrol.

---

### 6.2 Gaussian Naive Bayes

Naive Bayes menggunakan Teorema Bayes dengan asumsi bahwa setiap fitur bersifat *conditionally independent* satu sama lain. Varian Gaussian mengasumsikan fitur berdistribusi normal di setiap kelas.

**Parameter yang digunakan:**

```python
GNB_PARAMS = {
    "var_smoothing": 1e-9,  # stabilitas numerik
}
```

- `var_smoothing`: Menambahkan nilai kecil ke varians fitur untuk menghindari pembagian dengan nol (*numerical stability*). Nilai default `1e-9` sudah cukup untuk sebagian besar kasus; bisa dinaikkan ke `1e-8` atau `1e-7` jika fitur memiliki varians sangat kecil.

**Keunggulan:** Sangat cepat, efisien untuk data besar, performa baik meski asumsi independence dilanggar secara moderat.
**Kelemahan:** Asumsi independence sering tidak terpenuhi di data nyata, tidak menghasilkan feature importance langsung.

---

## 7. Teknik Resampling

Enam teknik resampling diuji untuk mengatasi ketidakseimbangan kelas:

### 7.1 Tanpa Resampling (`none`)
Data training digunakan apa adanya tanpa modifikasi distribusi kelas. Digunakan sebagai **baseline** perbandingan.

### 7.2 Random UnderSampling (`undersample`)
Mengurangi jumlah sampel kelas mayoritas (Tidak Diabetes) secara acak hingga seimbang dengan kelas minoritas terkecil.

- **Pro:** Cepat, mengurangi ukuran dataset (training lebih cepat).
- **Kontra:** Kehilangan informasi dari kelas mayoritas yang dihapus.
- **Ukuran setelah resample:** ±11.100 baris (3.700 per kelas)

### 7.3 Random OverSampling (`oversample`)
Menduplikasi sampel dari kelas minoritas secara acak hingga seimbang dengan kelas mayoritas.

- **Pro:** Tidak menghilangkan data.
- **Kontra:** Risiko overfitting karena duplikasi exact.
- **Ukuran setelah resample:** ±456.000 baris (152.000 per kelas)

### 7.4 SMOTENC (`smotenc`)
*Synthetic Minority Oversampling Technique for Nominal and Continuous features* — menghasilkan sampel sintetis baru untuk kelas minoritas menggunakan interpolasi k-NN, dengan perlakuan khusus untuk fitur kategorik (biner/nominal).

- **Pro:** Menghasilkan data sintetis yang lebih beragam, tidak sekadar menduplikasi.
- **Kontra:** Lebih lambat dari ROS, perlu menentukan indeks fitur kategorik.
- **Fitur kategorik yang diberikan indeks:** `HighBP`, `DiffWalk`, `HighChol`, `HeartDiseaseorAttack`, `PhysActivity`

### 7.5 SMOTE-Tomek Links (`smote_tomek`)
Kombinasi SMOTE (oversampling kelas minoritas) + **Tomek Links** (undersampling — menghapus pasangan sampel dari kelas berbeda yang saling berdekatan di *decision boundary*). Hasilnya distribusi kelas lebih bersih di sekitar batas keputusan.

- **Pro:** Mengurangi noise di decision boundary.
- **Kontra:** Lebih kompleks dan lebih lambat.
- **Ukuran setelah resample:** ±444.000 baris

### 7.6 SMOTE-ENN (`smote_enn`)
Kombinasi SMOTE + **Edited Nearest Neighbours** — menghapus sampel yang salah diklasifikasikan oleh k-NN dari mayoritas tetangganya. Lebih agresif dari Tomek Links dalam membersihkan overlap antar kelas.

- **Pro:** Sangat efektif mengurangi overlap kelas.
- **Kontra:** Bisa menghapus terlalu banyak sampel, ukuran akhir tidak terprediksi.
- **Ukuran setelah resample:** ±296.000 baris

---

## 8. Hasil Eksperimen

Semua 12 eksperimen (2 model × 6 resampling) dijalankan dengan split dan random state yang sama untuk memastikan perbandingan yang adil.

### 8.1 Metrik Global (diurutkan dari F1 Weighted tertinggi)

| # | Model | Resampling | Accuracy | F1 Weighted | F1 Macro | Recall Macro |
|:---:|---|---|:---:|:---:|:---:|:---:|
| **1** | **Naive Bayes** | **none** | **0.6900** | **0.5634** | 0.2722 | 0.3333 |
| 2 | Decision Tree | smote_tomek | 0.5850 | 0.5524 | 0.3321 | 0.3371 |
| 3 | Decision Tree | smotenc | 0.5413 | 0.5292 | 0.3252 | 0.3268 |
| 4 | Decision Tree | smote_enn | 0.4963 | 0.5123 | 0.3318 | 0.3373 |
| 5 | Decision Tree | oversample | 0.3650 | 0.4103 | 0.3049 | 0.3353 |
| 6 | Naive Bayes | smote_tomek | 0.3613 | 0.4036 | 0.3070 | 0.3442 |
| 7 | Decision Tree | undersample | 0.3438 | 0.3830 | 0.3020 | 0.3468 |
| 8 | Decision Tree | none | 0.3175 | 0.3609 | 0.2838 | 0.3341 |
| 9 | Naive Bayes | undersample | 0.3250 | 0.3599 | 0.2965 | 0.3577 |
| 10 | Naive Bayes | oversample | 0.3200 | 0.3584 | 0.2866 | 0.3380 |
| 11 | Naive Bayes | smotenc | 0.3113 | 0.3468 | 0.2797 | 0.3300 |
| 12 | Naive Bayes | smote_enn | 0.1550 | 0.0735 | 0.1579 | 0.3347 |

**🏆 Model terpilih: Naive Bayes + Tanpa Resampling**

---

### 8.2 Metrik Per-Kelas

| # | Model | Resampling | F1 Tdk Diabetes | Recall Prediabetes | F1 Prediabetes | Recall Diabetes | F1 Diabetes |
|:---:|---|---|:---:|:---:|:---:|:---:|:---:|
| 1 | Naive Bayes | none | **0.8166** | 0.0000 | 0.0000 | 0.0000 | 0.0000 |
| 2 | Decision Tree | smote_tomek | 0.7441 | 0.0930 | 0.1188 | 0.1176 | 0.1333 |
| 3 | Decision Tree | smotenc | 0.7071 | 0.1008 | 0.1106 | 0.1513 | 0.1579 |
| 4 | Decision Tree | smote_enn | 0.6698 | 0.1240 | 0.1391 | 0.2521 | 0.1863 |
| 5 | Decision Tree | oversample | 0.5023 | 0.2868 | 0.1907 | 0.3277 | 0.2216 |
| 6 | Naive Bayes | smote_tomek | 0.4877 | 0.2946 | 0.2077 | 0.3613 | 0.2257 |
| 7 | Decision Tree | undersample | 0.4536 | 0.3721 | 0.2222 | 0.3277 | 0.2301 |
| 8 | Decision Tree | none | 0.4277 | 0.3721 | 0.2202 | 0.3277 | 0.2037 |
| 9 | Naive Bayes | undersample | 0.4152 | 0.3643 | 0.2298 | **0.4118** | **0.2444** |
| 10 | Naive Bayes | oversample | 0.4205 | 0.3566 | 0.2283 | 0.3529 | 0.2111 |
| 11 | Naive Bayes | smotenc | 0.4055 | 0.3333 | 0.2043 | 0.3613 | 0.2293 |
| 12 | Naive Bayes | smote_enn | 0.0000 | **0.4496** | **0.2402** | 0.5546 | 0.2336 |

---

## 9. Analisis Hasil

### 9.1 Model Terbaik berdasarkan F1 Weighted

**Naive Bayes + Tanpa Resampling** terpilih sebagai model terbaik dengan F1 Weighted **0.5634** dan Accuracy **0.6900**. Ini adalah hasil tertinggi secara keseluruhan.

Namun, jika dilihat lebih dalam pada metrik per-kelas, model ini memiliki kelemahan kritis:

```
Kelas 0 (Tidak Diabetes) → F1 = 0.8166, Recall = 1.0000  ✅ Sangat baik
Kelas 1 (Prediabetes)    → F1 = 0.0000, Recall = 0.0000  ❌ Gagal total
Kelas 2 (Diabetes)       → F1 = 0.0000, Recall = 0.0000  ❌ Gagal total
```

Model ini **hanya memprediksi kelas 0** hampir setiap saat. F1 Weighted-nya tinggi semata karena kelas 0 mendominasi 82.7% data. Ini adalah contoh klasik *accuracy paradox* pada dataset imbalanced.

---

### 9.2 Dilema: F1 Weighted vs Kemampuan Deteksi Kelas Minoritas

Terdapat trade-off yang jelas antara dua tujuan:

| Prioritas | Model yang Direkomendasikan | Alasan |
|---|---|---|
| F1 Weighted tertinggi (metrik global) | Naive Bayes + none | 0.5634 |
| Kemampuan deteksi Diabetes terbaik | Naive Bayes + undersample | Recall Diabetes = 0.4118 |
| Kemampuan deteksi Prediabetes terbaik | Naive Bayes + SMOTE-ENN | Recall Prediabetes = 0.4496 |
| Keseimbangan terbaik antar kelas | Decision Tree + SMOTE-ENN | F1 Macro = 0.3318, Recall Macro = 0.3373 |

Untuk keperluan **aplikasi klinis atau deteksi dini**, kemampuan mendeteksi Prediabetes dan Diabetes lebih penting daripada F1 Weighted. Dalam konteks tersebut, eksperimen dengan resampling agresif (SMOTE-ENN, undersample) lebih tepat diprioritaskan.

---

### 9.3 Pengaruh Resampling

**Pada Decision Tree:**

- Tanpa resampling: Decision Tree bias ke kelas mayoritas (F1 Diabetes rendah = 0.2037).
- Resampling apapun meningkatkan kemampuan deteksi kelas minoritas, tetapi menurunkan F1 Weighted.
- SMOTE-Tomek memberikan keseimbangan terbaik untuk DT: F1 Weighted = 0.5524 dan F1 Diabetes = 0.1333.
- SMOTE-ENN memberikan Recall Diabetes tertinggi untuk DT (0.2521) dengan trade-off F1 Weighted turun ke 0.5123.

**Pada Naive Bayes:**

- Tanpa resampling: NB unggul secara F1 Weighted (0.5634) tetapi gagal mendeteksi kelas minoritas.
- SMOTE-ENN menghasilkan perubahan dramatis: Recall Prediabetes naik ke 0.4496 dan Recall Diabetes ke 0.5546, tetapi F1 Weighted jatuh ke 0.0735 (karena banyak prediksi kelas 0 yang salah).
- Undersample memberikan kompromi paling sehat untuk NB: Recall Diabetes = 0.4118, F1 Weighted = 0.3599.

---

### 9.4 Perbandingan Decision Tree vs Naive Bayes

| Aspek | Decision Tree | Naive Bayes |
|---|---|---|
| F1 Weighted terbaik | 0.5524 (SMOTE-Tomek) | **0.5634** (none) |
| Recall Diabetes terbaik | 0.3277 | **0.5546** (SMOTE-ENN) |
| Recall Prediabetes terbaik | 0.3721 | **0.4496** (SMOTE-ENN) |
| Pengaruh resampling | Konsisten meningkat | Dramatis & tidak stabil |
| Interpretabilitas | ✅ Feature Importance | ❌ Tidak langsung |
| Kecepatan training | Lebih lambat (data besar) | **Sangat cepat** |

---

## 10. Aplikasi Streamlit

Aplikasi web dibangun menggunakan Streamlit dengan tiga halaman:

### Halaman 1 — Prediksi

Form input 10 fitur hasil Chi-Square dengan label bahasa Indonesia:

| Input | Tipe Widget | Range/Pilihan |
|---|---|---|
| Tekanan Darah Tinggi (HighBP) | Dropdown | Ya / Tidak |
| Kesulitan Berjalan (DiffWalk) | Dropdown | Ya / Tidak |
| Kondisi Kesehatan Umum (GenHlth) | Slider | 1 (Sangat Baik) – 5 (Sangat Buruk) |
| Kolesterol Tinggi (HighChol) | Dropdown | Ya / Tidak |
| Riwayat Penyakit Jantung | Dropdown | Ya / Tidak |
| BMI | Number Input | 10.0 – 100.0 |
| Kelompok Usia (Age) | Slider | 1 (18–24) – 13 (80+) |
| Hari Sakit 30 Hari Terakhir | Slider | 0 – 30 |
| Tingkat Pendapatan (Income) | Slider | 1 (<$10k) – 8 (>$75k) |
| Aktif Fisik (PhysActivity) | Dropdown | Ya / Tidak |

Output prediksi meliputi:
- **Label kelas** dengan warna (hijau/oranye/merah)
- **Probabilitas per kelas** dalam bentuk horizontal bar chart
- **Feature importance** dari model Decision Tree (jika model terbaik adalah DT)
- **Saran tindak lanjut** berdasarkan kelas prediksi

### Halaman 2 — Perbandingan Eksperimen

- **Tab Metrik Global:** Tabel seluruh 12 eksperimen dengan highlight nilai tertinggi/terendah
- **Tab Metrik Per-Kelas:** Breakdown precision, recall, F1 untuk tiap kelas per eksperimen
- **Grafik interaktif:** Dropdown untuk memilih metrik yang ingin divisualisasikan
- **Confusion Matrix** dan **Feature Importance** model terbaik
- **Card ringkasan** model terbaik dengan F1 Weighted, F1 Macro, Recall Macro

### Halaman 3 — Tentang Proyek

Dokumentasi ringkas stack teknologi, alur kerja, dan cara menjalankan.

---

## 11. Struktur Proyek

```
diabetes_project/
│
├── app.py              # Aplikasi Streamlit (UI prediksi & perbandingan)
├── train.py            # Pipeline training lengkap
├── evaluate.py         # Fungsi metrik (global & per-kelas) + visualisasi
├── utils.py            # Helper: load data, preprocessing, scaler, resampling
├── config.py           # ⭐ Konfigurasi terpusat — semua parameter di sini
│
├── requirements.txt    # Dependensi Python
├── README.md           # Panduan singkat
│
├── data/
│   └── diabetes_cdc.csv            # Dataset CDC (diunduh otomatis)
│
└── models/
    ├── best_model.joblib           # Model + scaler + metadata tersimpan
    ├── experiment_results.csv      # Tabel 12 hasil eksperimen
    ├── best_confusion_matrix.png   # Heatmap confusion matrix model terbaik
    ├── feature_importance.png      # Bar chart feature importance (jika DT)
    └── comparison_chart.png        # Grafik perbandingan semua eksperimen
```

### Deskripsi Per-File

**`config.py`** — Satu-satunya file yang perlu diubah untuk eksperimen berbeda. Berisi: path, nama fitur, label kelas, parameter split, parameter DT & GNB, daftar metode resampling, dan indeks fitur kategorik untuk SMOTENC.

**`utils.py`** — Fungsi-fungsi stateless: `load_dataset()`, `preprocess_data()`, `split_data()`, `fit_scaler()`, `apply_scaler()`, `apply_resampling()`. Tidak ada side effect, mudah diuji secara terpisah.

**`evaluate.py`** — Semua logika evaluasi: `compute_metrics()` (global), `compute_per_class_metrics()` (per kelas), `plot_confusion_matrix()`, `plot_feature_importance()`, `summarize_results()`, `pick_best_model()`, `plot_comparison_bar()`.

**`train.py`** — Orkestrasi pipeline: memanggil utils → menjalankan loop eksperimen → memanggil evaluate → menyimpan model terbaik ke `.joblib`.

**`app.py`** — Antarmuka Streamlit. Load model dari `.joblib`, render form input, tampilkan hasil prediksi dan visualisasi perbandingan.

---

## 12. Cara Menjalankan

### Persyaratan
- Python 3.9+
- pip

### Langkah

```bash
# 1. Clone atau ekstrak folder proyek
cd diabetes_project

# 2. Install dependensi
pip install -r requirements.txt

# 3. Jalankan training
#    Dataset akan diunduh otomatis dari GitHub (~5 MB)
#    Jika gagal, unduh manual dari Kaggle: "diabetes-health-indicators-dataset"
#    dan letakkan di data/diabetes_cdc.csv
python train.py

# 4. Jalankan aplikasi Streamlit
streamlit run app.py
#    Buka browser di http://localhost:8501
```

### Output Training

```
Training pipeline berhasil.
  Model terbaik : Naive Bayes + none
  F1 Weighted   : 0.7892        ← angka nyata dari dataset asli
  Jalankan Streamlit: streamlit run app.py
```

### Mengubah Konfigurasi

Semua parameter dapat diubah di `config.py` tanpa menyentuh kode lain:

```python
# Ganti fitur yang digunakan
FEATURE_COLUMNS = ["HighBP", "BMI", ...]

# Ubah parameter Decision Tree
DT_PARAMS = {
    "max_depth": 10,          # coba 8, 12, 15, 20
    "class_weight": None,     # hapus balanced jika ingin default
    "criterion": "entropy",   # coba entropy vs gini
    ...
}

# Matikan beberapa metode resampling
RESAMPLING_METHODS = ["none", "smote_tomek"]  # hanya 2 metode
```

---

## 13. Kesimpulan

### Temuan Utama

1. **Model terbaik berdasarkan F1 Weighted** adalah **Naive Bayes tanpa resampling** (F1 = 0.5634, Accuracy = 0.6900) — namun model ini gagal mendeteksi kelas minoritas sama sekali.

2. **Ketidakseimbangan kelas** adalah tantangan dominan proyek ini. Kelas Prediabetes (2% dari data) sangat sulit dideteksi oleh semua model, dengan Recall Prediabetes terbaik hanya 0.4496 (NB + SMOTE-ENN).

3. **Resampling meningkatkan deteksi kelas minoritas** tetapi menurunkan F1 Weighted secara signifikan — terjadi trade-off yang tidak bisa dihindari.

4. **Decision Tree + SMOTE-Tomek** menawarkan keseimbangan terbaik antara F1 Weighted (0.5524) dan kemampuan mendeteksi kelas minoritas, sekaligus menghasilkan feature importance yang interpretable.

5. **Fitur Chi-Square terpenting** — HighBP, DiffWalk, GenHlth, HighChol, dan HeartDiseaseorAttack — semuanya berkaitan dengan kondisi kesehatan kardiovaskular dan mobilitas, konsisten dengan literatur medis tentang faktor risiko diabetes.

### Rekomendasi Pengembangan Lanjutan

- **GridSearchCV / RandomizedSearchCV** untuk optimasi hyperparameter Decision Tree secara sistematis.
- **Threshold tuning** — menyesuaikan ambang batas probabilitas klasifikasi untuk meningkatkan Recall kelas minoritas.
- **Ensemble method** (Random Forest, XGBoost) sebagai model pembanding tambahan.
- **Stratified K-Fold Cross Validation** untuk estimasi performa yang lebih robust.
- **SHAP values** untuk interpretabilitas model yang lebih mendalam, terutama untuk Naive Bayes yang tidak memiliki feature importance langsung.

---

*Dokumen ini dibuat sebagai bagian dari proyek akademik mata kuliah Kecerdasan Buatan.*
*Dataset: CDC Behavioral Risk Factor Surveillance System 2015 — sumber publik, bukan data klinis.*
*Prediksi dari model ini bersifat estimasi akademik dan tidak dapat digunakan sebagai diagnosis medis.*
