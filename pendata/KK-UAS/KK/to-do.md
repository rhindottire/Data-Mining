## Langkah 1
### Removing Redundant Columns
* Alasan Klinis:
  - Biopsy adalah Gold Standard:
    Kolom `Biopsy` (hasil biopsi) adalah diagnosis definitif kanker serviks. Sementara `Dx:Cancer`, `Dx:CIN`, dan `Dx:HPV` adalah diagnosis klinis awal yang belum terkonfirmasi biopsi. Mempertahankannya berisiko membuat model mempelajari pola diagnosis yang tidak akurat.
    Contoh: Pasien dengan `Dx:Cancer = 1` bisa saja ternyata `Biopsy = 0` (false positive).

  - Hindari Data Leakage:
    Kolom `Hinselmann`, `Schiller`, dan `Citology` adalah hasil tes yang biasanya dilakukan setelah kecurigaan kanker. Jika dimasukkan ke model deteksi dini, ini dianggap bocoran informasi (*temporal leakage*), karena dalam skenario nyata, tes ini belum tersedia pada tahap awal screening.

* Alasan Statistik:
  - Multikolinearitas:
    Kolom `Dx:Cancer`, `Dx:CIN`, dan `Dx:HPV` berkorelasi tinggi dengan `Biopsy`. Jika tidak dihapus, model akan overfit karena mempelajari hubungan yang redundan.
    Contoh: Korelasi sempurna antara `Dx:Cancer` dan `Biopsy` akan membuat model hanya mengandalkan satu fitur.

### Discarding Irrelevant Features for Early Detection
* Alasan Klinis:
  - Fokus pada Prediksi, Bukan Riwayat:
    `STDs: Time since first diagnosis` mengukur lama pasien hidup dengan STD, yang lebih relevan untuk prognosis (perkembangan penyakit) daripada deteksi dini. Deteksi dini membutuhkan faktor yang memprediksi onset kanker, bukan durasi komplikasi.

  - Keterbatasan Data:
    Kolom ini memiliki >80% missing values karena hanya diisi jika pasien pernah didiagnosis STD. Ini membuatnya tidak informatif untuk pasien tanpa riwayat STD.

* Alasan Statistik:
  - Noise vs Signal:
    Fitur dengan missing values tinggi cenderung menjadi noise yang mengganggu pembelajaran model. Menghapusnya meningkatkan rasio signal-to-noise dalam dataset.

* Relevansi dengan Judul Penelitian:
  - Naive Bayes:
    Asumsi independensi fitur akan terganggu jika ada fitur dengan missing values besar, karena distribusi probabilitas menjadi tidak stabil.
  - Logistic Regression:
    Fitur dengan variansi rendah (karena banyak missing values) akan menghasilkan koefisien regresi yang tidak bermakna.

### Ensuring Data is Relevant to Cervical Cancer
* Kriteria Data yang Dipertahankan:
  1. Faktor Risiko yang Terbukti:
    - Usia pertama berhubungan seksual (`First sexual intercourse`): Terkait dengan paparan HPV dini.  
    - Jumlah pasangan seksual (`Number of sexual partners`): Meningkatkan risiko infeksi HPV.  
    - Penggunaan kontrasepsi hormonal (`Hormonal Contraceptives`): Berhubungan dengan perubahan sel serviks.

  2. Faktor Biomarker Tidak Langsung:
    - Riwayat HPV (`STDs:HPV`): HPV adalah penyebab 99% kanker serviks.  
    - Jumlah kehamilan (`Num of pregnancies`): Kehamilan multipel terkait risiko kanker.

* Fitur yang Dipertimbangkan untuk Deteksi Dini:

| Kategori Fitur   | Contoh Kolom                 | Relevansi                                        |
|------------------|------------------------------|--------------------------------------------------|
| Demografi        | Age                          | Usia >40 tahun meningkatkan risiko               |
| Perilaku Seksual | Number of sexual partners    | Risiko paparan HPV                               |
| Riwayat Medis    | STDs:HPV, STDs:genital herpes| Infeksi STD tertentu terkait karsinogenesis      |
| Kontrasepsi      | Hormonal Contraceptives, IUD | Penggunaan jangka panjang terkait epitel serviks |

## Langkah 2
### Why Missing Values ​​Need to be Addressed?
* Dampak pada Model:
  - Naive Bayes:
    Algoritma ini menghitung probabilitas berdasarkan data yang ada. Missing values akan mengganggu perhitungan probabilitas posterior, terutama jika banyak data hilang.
  - Logistic Regression:
    Tidak bisa bekerja dengan missing values. Jika tidak diatasi, seluruh baris data akan dihapus (*listwise deletion*), berisiko mengurangi jumlah sampel signifikan.

### Missing Values ​​Handling Strategy
* A. Menghapus Kolom dengan Missing Values Tinggi (>40%)
  - `STDs: Time since first diagnosis` (hanya 71 dari 835 data yang valid)
  - `STDs: Time since last diagnosis`
  - Alasan Penghapusan:
    - Tidak Informatif: Data ini hanya tersedia untuk pasien yang pernah didiagnosis STD sebelumnya. Untuk deteksi dini pada populasi umum, fitur ini tidak relevan 
    - Noise Dominan Missing values >80% membuat kolom ini lebih banyak mengandung noise daripada sinyal prediktif 

* B. Mengisi Missing Values pada Kolom Lain
  1. Data Numerik (`Age`, `Number of sexual partners`):
    - Metode: *Median Imputation*
    - Alasan:
      - Median lebih robust terhadap outlier daripada mean (usia ekstrem atau jumlah pasangan seksual yang tidak realistis).
      - Mempertahankan distribusi data asli, penting untuk Naive Bayes yang mengasumsikan distribusi normal.

  2. Data Kategorikal (`Smokes`, `Hormonal Contraceptives`):
    - Metode: *Mode Imputation* (mengisi dengan nilai paling sering muncul)
    - Alasan:
      - Kolom ini merepresentasikan status biner (0/1). Mengisi dengan modus mempertahankan proporsi kategori mayoritas.
      - Contoh: Jika 70% pasien tidak merokok (`Smokes = 0`), nilai kosong diisi dengan 0.

### Why Not Use Another Method?
1. Prediksi Berdasarkan Model (regresi):
   - Berisiko overfitting karena jumlah sampel terbatas.
   - Tidak praktis untuk dataset dengan 36 kolom.

2. Menghapus Baris dengan Missing Values:
   - Berpotensi menghilangkan 100+ sampel (terutama jika missing values tersebar di banyak kolom).
   - Bisa mengurangi kemampuan deteksi dini, terutama jika kasus kanker langka.

### Relevance to Research
1. Naive Bayes:
   - Sensitivitas Distribusi: Imputasi median menjaga distribusi data numerik, sehingga perhitungan probabilitas *likelihood* tetap valid.
   - Independensi Fitur: Missing values yang tidak diatasi akan merusak asumsi independensi karena data hilang mungkin terkait pola tertentu.

2. Logistic Regression:
   - Stabilitas Koefisien: Imputasi mode pada data kategorikal mencegah pergeseran koefisien regresi yang tidak stabil.
   - Konsistensi Skala: Imputasi median memastikan fitur numerik tetap terstandardisasi, penting untuk optimasi *gradient descent*.

3. Deteksi Dini:
   - Keterandalan Faktor Risiko: Contoh: Jika `Number of sexual partners` banyak yang hilang, model mungkin gagal menangkap hubungan antara aktivitas seksual dan risiko kanker.
   - Generalisasi ke Populasi: Imputasi memungkinkan penggunaan data dari pasien yang tidak lengkap isiannya (mirip dengan kondisi riil di klinik).

### Noted!
Perbandingan dua pasien:
- Pasien A: Usia 28, `Number of sexual partners` = NaN, `STDs:HPV` = 1.
- Pasien B: Usia 45, `Number of sexual partners` = 5, `STDs:HPV` = 0.

Perbandingan imputasi:
- Tanpa imputasi: Data Pasien A dihapus → model kehilangan kasus HPV positif yang kritis.
- Dengan imputasi: `Number of sexual partners` Pasien A diisi median.

Pertimbangan Etis: Kolom `Biopsy` tidak boleh diimputasi karena merupakan variabel dependen. Jika ada missing values di sini, sampel harus dihapus.

## langkah 3
### Apa Itu Ketidakseimbangan Kelas?
- Definisi: Ketidakseimbangan kelas terjadi ketika jumlah sampel di satu kategori target (misal: non-kanker) jauh lebih banyak daripada kategori lainnya (kanker).  
- Contoh: Jika dari 835 sampel, hanya 85 pasien yang positif kanker (`Biopsy = 1`), rasio kelasnya adalah 10:1.

### Mengapa Ini Masalah untuk Deteksi Kanker Serviks?
1. Dampak pada Model:
   - `Naive Bayes`: Cenderung memprediksi kelas mayoritas karena perhitungan probabilitas prior $ P(y=0) $ jauh lebih besar daripada $ P(y=1) $.

   - `Logistic Regression`: Loss function akan dioptimalkan untuk meminimalkan kesalahan pada kelas mayoritas, sehingga mengabaikan minoritas.

2. Konsekuensi Klinis:
   - False Negative (Pasien kanker diprediksi sehat): Sangat berbahaya karena pasien tidak mendapat penanganan dini.
   - False Positive (Pasien sehat diprediksi kanker): Berisiko menyebabkan kecemasan dan biaya tes lanjutan tidak perlu.


### Apa yang Akan Dilakukan pada Langkah Ini?
1. Mengecek Distribusi Kelas Target
Memahami seberapa parah ketidakseimbangan kelas.
   * Metode:
      - Hitung persentase pasien kanker (`Biopsy = 1`) vs non-kanker (`Biopsy = 0`).
      - Jika rasio > 1:5 (20% kanker), perlu penanganan khusus.

   * Alasan:
      - Deteksi dini kanker serviks adalah kasus high-stakes di mana false negative harus diminimalkan.
      - Model dengan akurasi 95% tapi recall 0% (gagal mendeteksi semua kanker) tidak berguna secara klinis.

2. Memilih Strategi Penyeimbangan
Pilihan Metode:
   * A. Oversampling Kelas Minoritas (SMOTE):
      - Cara Kerja: Menghasilkan sampel sintetis kanker dengan interpolasi fitur.
      - Alasan Dipilih:
        - Mempertahankan informasi dari semua sampel asli.
        - Cocok untuk dataset kecil (835 sampel) karena tidak menghapus data.

   * B. Class Weighting:
      - Cara Kerja: Memberikan bobot lebih tinggi pada kesalahan klasifikasi kelas minoritas.
      - Alasan Dipilih untuk Logistic Regression:
        - Lebih etis karena tidak membuat data sintetis palsu.
        - Memaksa model untuk "lebih menghukum" kesalahan prediksi kanker.
      - Mengapa Tidak Menggunakan Undersampling?
        - Menghapus sampel non-kanker berisiko kehilangan pola penting dari kelas mayoritas.
        - Tidak sesuai untuk dataset dengan jumlah sampel sudah relatif kecil.

   * C. Mengevaluasi Dampak Penyeimbangan
      Memastikan penyeimbangan tidak merusak pola data asli.
      Metode:
      - Bandingkan distribusi fitur (misal: usia, jumlah pasangan seksual) sebelum dan setelah penyeimbangan.
      - Pastikan sampel sintetis tidak menciptakan outlier yang tidak realistis.

### Relevance Title
1. Naive Bayes:
   - Kelemahan: Tidak bisa menggunakan class weighting.
   - Solusi: SMOTE lebih cocok untuk meningkatkan recall tanpa mengubah struktur model.

2. Logistic Regression:
   - Fleksibilitas: Bisa menggunakan SMOTE atau class weighting.
   - Perbandingan: Akan dievaluasi mana yang lebih baik untuk konteks medis.

3. Deteksi Dini:
   - Prioritas: Recall tinggi (>80%) untuk memastikan sedikit mungkin kanker terlewat.
   - Trade-off: Precision mungkin turun, tetapi ini diterima selama pasien "false positive" dikonfirmasi biopsi.

### Contoh Kasus Tanpa Penyeimbangan vs Dengan Penyeimbangan
| Scenario            | Akurasi | Recall (Kanker) | Konsekuensi Klinis                            |
|---------------------|---------|-----------------|-----------------------------------------------|
| Tanpa Penyeimbangan | 95%     | 10%             | 90% pasien kanker tidak terdeteksi            |
| Dengan SMOTE        | 85%     | 75%             | 25% pasien kanker terlewat, tetapi lebih baik |

### Pertimbangan Etis
1. Data Sintetis: SMOTE menghasilkan data palsu. Pastikan ini tidak dilaporkan sebagai "data pasien nyata" dalam penelitian.
2. Generalizability: Model yang dilatih dengan data sintetis harus divalidasi pada data riil untuk memastikan tidak overfit.

## Langkah 4
### **Apa Tujuan Split Data 80-20?**
1. **Training Set (80%)**:  
   - Digunakan untuk **membangun model**:  
     - Naive Bayes menghitung probabilitas likelihood.  
     - Logistic Regression mengoptimasi koefisien regresi.  
   - Jumlah sampel: ~668 data (80% dari 835).  

2. **Testing Set (20%)**:  
   - Digunakan untuk **menguji generalisasi model** pada data yang belum pernah dilihat.  
   - Jumlah sampel: ~167 data (20% dari 835).  
   - Simulasi kondisi nyata: Model diuji seolah-olah menghadapi pasien baru di klinik.

### **Mengapa Harus 80-20?**
1. **Keseimbangan antara Pembelajaran dan Evaluasi**:  
   - 80% cukup untuk menangkap pola kompleks dalam data medis (misal: hubungan non-linear antara usia dan risiko kanker).  
   - 20% memastikan evaluasi **statistikal yang signifikan**:  
     - Jika hanya 5% testing (~42 data), hasil evaluasi berisiko noise tinggi.  

2. **Konsistensi dengan Literatur**:  
   - Rasio 80-20 adalah standar di penelitian *machine learning* untuk dataset berukuran kecil-sedang (<10,000 sampel).  

3. **Efisiensi Komputasi**:  
   - Dataset kecil (835 sampel) tidak memerlukan validasi kompleks seperti 10-fold cross-validation.  

### **Mengapa Split Data Dilakukan *Setelah* Penanganan Missing Values & SMOTE?**
1. **Mencegah Data Leakage**:  
   - Jika split dilakukan **sebelum** imputasi missing values atau SMOTE:  
     - Informasi dari testing set "bocor" ke training set (misal: median dihitung dari seluruh data).  
     - Model menjadi overestimate performa karena sudah "melihat" pola testing set.  

2. **Stratifikasi Penting untuk Data Tidak Seimbang**:  
   - Parameter `stratify=y` dalam `train_test_split` memastikan proporsi kelas kanker/non-kanker **sama** di training dan testing set.  
   - Contoh: Jika 10% data adalah kanker, maka:  
     - Training set: 10% kanker (534 non-kanker + 66 kanker).  
     - Testing set: 10% kanker (134 non-kanker + 17 kanker).  

### **Detail Kritis yang Sering Terlewatkan**
1. **Random State**:  
   - Penggunaan `random_state=42` memastikan hasil split **reproducible**.  
   - Tanpa ini, split acak berbeda tiap eksekusi → hasil evaluasi tidak konsisten.  

2. **Karakteristik Testing Set**:  
   - Testing set **tidak boleh disentuh** selama:  
     - Pelatihan model.  
     - Penyeimbangan kelas (SMOTE hanya dilakukan pada training set).  
   - Ini mensimulasikan skenario riil: model hanya belajar dari data historis, lalu diuji pada pasien baru.  

3. **Dampak pada Deteksi Dini**:  
   - Jika testing set tidak stratifikasi, mungkin tidak ada kasus kanker sama sekali → recall = 0% (model gagal total).  
   - Contoh: Jika 20% testing set (167 data) hanya berisi 1 kasus kanker, metrik evaluasi menjadi tidak stabil.

### **Mengapa Ini Relevan untuk Naive Bayes vs Logistic Regression?**
1. **Naive Bayes**:  
   - Cenderung lebih stabil pada dataset kecil.  
   - Jika testing set terlalu kecil (misal 10%), perbedaan performa kedua model sulit diukur.  

2. **Logistic Regression**:  
   - Membutuhkan lebih banyak data untuk mengestimasi koefisien.  
   - Split 80-20 memastikan training set cukup besar (~668 data) untuk model kompleks.  

3. **Perbandingan yang Adil**:  
   - Kedua model diuji pada **testing set yang sama** → perbedaan performa murni karena algoritma, bukan keberuntungan split data.  

---

### **Apa yang Terjadi Jika Split Data Salah?**
- **Kasus 1**: Tidak stratifikasi → testing set hanya berisi non-kanker.  
  - Model Naive Bayes: Akurasi 100%, Recall 0% (gagal total).  
  - Konsekuensi: Pasien kanker tidak terdeteksi.  

- **Kasus 2**: Data leakage → model melihat pola testing set selama training.  
  - AUC-ROC tinggi (misal 95%), tetapi performa drop drastis di dunia nyata.  
  - Contoh: Model "mengingat" pasien di testing set, bukan belajar pola umum.  

### **Relevansi dengan Judul Penelitian**
1. **Deteksi Dini**:  
   - Split 80-20 menguji kemampuan model untuk **menggeneralisasi** pola risiko kanker ke populasi baru.  

2. **Analisis Performa**:  
   - Perbandingan AUC-ROC dan recall kedua model hanya valid jika diuji pada **testing set yang sama**.  

3. **Kredibilitas Hasil**:  
   - Metodologi split data yang ketat meningkatkan kepercayaan hasil penelitian di komunitas medis.  

## Langkah 5
### Kategorisasi Jenis Fitur
Berdasarkan `df.info()`, fitur terbagi menjadi:
   - **Numerik Kontinu** (float64): `Age`, `Number of sexual partners`, `First sexual intercourse`, `Num of pregnancies`, `Smokes (years)`, dll.
   - **Numerik Kategorikal/Biner** (int64/float64 yang seharusnya kategori): `Smokes`, `Hormonal Contraceptives`, `IUD`, `STDs`, dan semua kolom `STDs:...` (kecuali `STDs: Number of diagnosis`).
   - **Target** (`Biopsy`): Sudah biner (0/1).

#### Transformasi dan Konversi Tipe Data Kategorikal
- `Smokes` (822 non-null float64)
- `STDs:condylomatosis` (735 non-null float64)

**Aksi**:
- Ubah ke **integer** (0/1) karena merepresentasikan status biner:
  ```python
  df['Smokes'] = df['Smokes'].astype(int)
  ```
**Alasan**:
- Kesalahan interpretasi model: Jika tetap float, model mungkin menganggap nilai 0.0 vs 1.0 sebagai kontinu.
- Khusus untuk Naive Bayes, probabilitas likelihood akan lebih akurat dengan data diskrit.

#### Standarisasi Fitur Numerik Kontinu
- `Age` (rentang: 15-80 tahun)
- `Number of sexual partners` (rentang: 1-25)
- `Smokes (packs/year)` (rentang: 0.1-2.5)

**Aksi**:  
- Gunakan **StandardScaler** untuk mengubah nilai ke skala Z-score (mean=0, std=1).  

**Alasan**:  
- **Logistic Regression**:  
  Sangat sensitif terhadap skala fitur. Tanpa standarisasi, fitur dengan rentang besar (misal `Age`) akan mendominasi perhitungan koefisien.  
- **Naive Bayes**:  
  Tidak memerlukan standarisasi, tetapi dilakukan untuk memastikan perbandingan adil antara kedua model.

### Why is it Important to Detect Early?
1. **Naive Bayes**:  
   - **Fitur Independen**:  
     Jika ada fitur yang berkorelasi tinggi (misal: `Hormonal Contraceptives` dan `IUD`), asumsi independensi akan dilanggar → probabilitas likelihood tidak akurat.  
   - Solusi: Feature engineering untuk mengurangi korelasi.  

2. **Logistic Regression**:
   - **Multikolinearitas**:
     Fitur seperti `STDs:condylomatosis` dan `STDs:cervical condylomatosis` mungkin berkorelasi → menyebabkan estimasi koefisien tidak stabil.  
   - Solusi: Gunakan **VIF (Variance Inflation Factor)** untuk mengidentifikasi dan menghapus fitur redundan.

### Relevansi dengan Judul Penelitian
1. **Naive Bayes**:
   - Performa tergantung pada **pemisahan distribusi fitur** antara kelas kanker/non-kanker.
   - Contoh: Jika `Tahun Aktif Seksual` terstandarisasi, perbedaan mean antara kedua kelas lebih mudah terdeteksi.

2. **Logistic Regression**:
   - Membutuhkan **linearitas** antara fitur dan log-odds kanker.
   - Contoh: Transformasi `Total Pack-Years` membuat hubungan linear dengan risiko kanker lebih jelas.

3. **Keterbandingan Model**:
   - Kedua model harus diuji pada **fitur yang sama** setelah transformasi untuk memastikan perbedaan performa murni dari algoritma.

## Langkah 6
### 1. Pelatihan Model
**A. Naive Bayes (GaussianNB)**  
- **Prosedur**:  
  - Model menghitung **probabilitas prior** dari kelas target (kanker/non-kanker) berdasarkan distribusi data training.  
  - Untuk setiap fitur, dihitung **likelihood** (probabilitas fitur muncul di kelas tertentu) dengan asumsi distribusi Gaussian.  
- **Alasan**:  
  - Tidak memerlukan optimasi iteratif → cepat untuk dataset kecil (<1000 sampel).  
  - Cocok untuk eksplorasi awal sebelum model kompleks.  

**B. Logistic Regression**  
- **Prosedur**:  
  - Model mengoptimasi **koefisien regresi** untuk memaksimalkan *log-likelihood* dari data training.  
  - Menggunakan algoritma seperti *gradient descent* atau *Newton-Raphson* untuk menemukan parameter terbaik.  
- **Alasan**:  
  - Mampu menangkap hubungan non-linear melalui transformasi fitur (misal: polinomial).  
  - Menyediakan interpretasi klinis melalui koefisien (misal: risiko kanker meningkat X% per tahun usia).  

### 2. Tuning Hyperparameter
**A. Untuk Naive Bayes**:  
- **Tidak ada hyperparameter utama** karena berbasis probabilitas statis.  
- Jika menggunakan varian lain (misal: MultinomialNB), bisa diatur **alpha** (smoothing parameter).  

**B. Untuk Logistic Regression**:  
- **Regularisasi (C)**:  
  - Mengontrol kekuatan regularisasi L1/L2 untuk mencegah overfitting.  
  - Contoh: `C=0.1` → regularisasi kuat (koefisien lebih kecil).  
- **Class Weight**:  
  - Jika tidak pakai SMOTE, bisa atur `class_weight='balanced'` untuk memberi bobot lebih pada kelas minoritas.  

**Alasan Tuning**:  
- Meningkatkan generalisasi model dengan mencegah overfitting ke data training.

### 3. Evaluasi Model
**A. Metrik Evaluasi**:  
| **Metrik**      | **Formula**               | **Relevansi untuk Deteksi Dini**                          |
|-----------------|---------------------------|-----------------------------------------------------------|
| **Recall**      | TP / (TP + FN)            | Prioritas utama: Minimalkan pasien kanker yang terlewat.  |
| **Precision**   | TP / (TP + FP)            | Hindari biaya tambahan tes lanjutan untuk false positive. |
| **F1-Score**    | 2*(Precision*Recall)/(Prec+Recall)| Menyeimbangkan trade-off recall dan precision.    |
| **AUC-ROC**     | Area di bawah kurva ROC   | Mengukur kemampuan membedakan kanker/non-kanker.          |
| **Specificity** | TN / (TN + FP)            | Penting untuk menghindari alarm palsu pada pasien sehat.  |

**B. Prosedur Validasi**:  
- **Holdout Validation**:  
  - Evaluasi menggunakan **testing set** yang belum pernah dilihat model selama pelatihan.  
  - Memastikan performa model generalizable.  
- **Cross-Validation (Opsional)**:  
  - Misal: 5-fold CV untuk estimasi performa yang lebih stabil.

### 4. Analisis Komparatif
**A. Perbandingan Performa**:
- Bandingkan kedua model berdasarkan:  
  - **Recall**: Model mana yang lebih baik mendeteksi kanker?  
  - **AUC-ROC**: Model mana yang lebih konsisten di semua threshold?  
  - **Kecepatan Pelatihan**: Relevan untuk implementasi real-time di klinik.  

**B. Contoh Hasil Hipotesis**:  
| **Model**               | **Recall** | **Precision** | **AUC-ROC** | **Waktu Pelatihan** |
|-------------------------|------------|---------------|-------------|---------------------|
| **Naive Bayes**         | 82%        | 45%           | 0.78        | 0.2 detik           |
| **Logistic Regression** | 75%        | 68%           | 0.85        | 1.5 detik           |

**Interpretasi**:  
- Naive Bayes unggul dalam recall (baik untuk skrining), tetapi banyak false positive.  
- Logistic Regression lebih seimbang dengan AUC lebih tinggi.

### 5. Interpretasi Hasil
**A. Untuk Naive Bayes**:
- **Fitur Paling Diskriminatif**:  
  - Fitur dengan perbedaan **likelihood** terbesar antara kelas kanker/non-kanker.  
  - Contoh: Jika `STDs:HPV` memiliki likelihood 0.8 untuk kanker vs 0.1 untuk non-kanker → HPV adalah prediktor kuat.  

**B. Untuk Logistic Regression**:  
- **Koefisien Regresi**:  
  - Nilai positif: Peningkatan fitur → peningkatan risiko kanker.  
  - Contoh: Koefisien `Age` = 0.5 → Setiap peningkatan 1 SD usia, risiko kanker meningkat 65% (e^0.5 ≈ 1.65).

### 6. Analisis Error
**A. Confusion Matrix**:  
- **False Negative (FN)**:  
  - Pasien kanker yang diprediksi sehat → Analisis pola: Apakah FN punya karakteristik khusus (misal: usia muda, tidak merokok)?  
- **False Positive (FP)**:  
  - Pasien sehat yang diprediksi kanker → Apakah FP terkait faktor risiko ambigu (misal: riwayat STD tanpa HPV)?  

**B. Contoh Temuan**:  
- Naive Bayes gagal mendeteksi kanker pada pasien dengan `Hormonal Contraceptives (years)` > 10.  
- Logistic Regression overfit pada pasien perokok berat.

### Relevansi dengan Judul Penelitian  
1. Naive Bayes:  
   - Kelebihan: Simpel dan cepat, cocok untuk skrining awal di klinik dengan sumber daya terbatas.  
   - Kekurangan: Asumsi independensi fitur tidak realistis (misal: usia dan jumlah pasangan seksual berkorelasi).  

2. Logistic Regression:  
   - Kelebihan: Interpretasi klinis melalui koefisien (misal: HPV meningkatkan risiko kanker 5x).  
   - Kekurangan: Memerlukan data lebih banyak untuk menangkap interaksi kompleks.  

3. Deteksi Dini:  
   - Jika recall Naive Bayes >80%, model ini layak jadi *first-line screening tool*.  
   - Jika AUC Logistic Regression >0.85, model ini bisa jadi *second-line validator* setelah skrining awal.
