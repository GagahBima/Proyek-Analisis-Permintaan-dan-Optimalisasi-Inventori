# Proyek Analisis Permintaan dan Optimalisasi Inventori

Repositori ini berisi analisis lengkap data penjualan untuk mengukur elastisitas harga, melakukan peramalan permintaan, dan mensimulasikan tingkat inventori untuk mengidentifikasi risiko kekurangan stok (*stockout*) secara proaktif.

---

## Latar Belakang & Tujuan Proyek

Manajemen inventori yang reaktif sering kali menyebabkan dua masalah utama: kehilangan potensi penjualan akibat kehabisan stok, dan biaya penyimpanan yang tinggi karena kelebihan stok. Proyek ini bertujuan untuk mengatasi masalah tersebut dengan membangun sebuah kerangka kerja berbasis data untuk:

1.  **Mengukur Dampak Harga:** Memahami secara kuantitatif seberapa besar pengaruh perubahan harga terhadap volume penjualan.
2.  **Meramalkan Permintaan:** Membuat model peramalan yang akurat untuk memprediksi penjualan di masa depan.
3.  **Mengoptimalkan Inventori:** Mensimulasikan tingkat stok untuk memberikan peringatan dini terhadap potensi risiko *stockout*.

---

## Proses Analisis

Analisis ini dibagi menjadi beberapa tahap utama:

1.  **Pembersihan Data (*Data Cleaning*):** Mempersiapkan data mentah dengan mengisi tanggal yang hilang, menangani nilai anomali (seperti harga <= 0), dan menstandarisasi format untuk analisis.
2.  **Dekomposisi Time Series:** Menggunakan metode **STL** untuk mengidentifikasi komponen tren jangka panjang dan pola musiman mingguan dalam data penjualan.
3.  **Analisis Elastisitas & Peramalan:** Membangun model **Regresi OLS** untuk mengukur elastisitas harga secara akurat. Selanjutnya, model peramalan dikembangkan untuk memprediksi permintaan harian, yang kinerjanya terbukti **42% lebih baik** dari metode baseline (MASE 0.97 vs 1.69).

---

## Temuan Utama & Hasil

### 1. Elastisitas Harga Sebesar -1.68
Ditemukan hubungan negatif yang kuat antara harga dan penjualan. Ini berarti setiap kenaikan harga 1% berpotensi menurunkan penjualan sekitar 1.68%, memberikan dasar yang kuat untuk strategi penetapan harga.

### 2. Peramalan Mengungkap Risiko Stockout Kritis
Puncak dari analisis ini adalah simulasi inventori yang menggunakan hasil peramalan. Visualisasi di bawah ini merangkum keseluruhan cerita:
* **Permintaan Pasar (Garis Oranye):** Menunjukkan prediksi permintaan yang terus ada.
* **Realisasi Penjualan (Garis Hijau):** Mensimulasikan penjualan aktual yang akan terjadi dengan stok yang ada.

Grafik ini menunjukkan bahwa **tanpa intervensi, perusahaan diproyeksikan akan mengalami stockout total** pada pertengahan September, di mana penjualan akan anjlok ke nol meskipun permintaan pasar masih tinggi.

![Grafik Simulasi Inventori dan Proyeksi Stockout](images/Screenshot.png)

---

## Tools & Library yang Digunakan

* **Bahasa:** Python 3
* **Data Analysis:** `pandas`, `numpy`
* **Statistical Modeling:** `statsmodels`
* **Visualization:** `matplotlib`, `seaborn`
* **Machine Learning:** `scikit-learn`
* **Environment:** Jupyter Notebook
  
---

## Cara Menjalankan Proyek
1.  **Clone repositori ini:**
    ```bash
    git clone [URL_REPOSITORY_ANDA]
    ```
2.  **Install dependensi yang dibutuhkan:**
    ```bash
    pip install pandas numpy matplotlib seaborn statsmodels scikit-learn
    ```
3.  **Jalankan notebook secara berurutan:**
    * Mulai dengan `Data_Cleaning.ipynb`.
    * Lanjutkan dengan notebook analisis utama (`Main__Analysis.ipynb`).
