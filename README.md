# Proyek Analisis Permintaan dan Optimalisasi Inventori

Repositori ini berisi analisis lengkap data penjualan untuk mengukur elastisitas harga, melakukan peramalan permintaan, dan mensimulasikan tingkat inventori untuk mengidentifikasi risiko kekurangan stok (*stockout*) secara proaktif.

---

## 🚩 Latar Belakang & Tujuan Proyek

Manajemen inventori yang reaktif sering kali menyebabkan dua masalah utama: kehilangan potensi penjualan akibat kehabisan stok, dan biaya penyimpanan yang tinggi karena kelebihan stok. Proyek ini bertujuan untuk mengatasi masalah tersebut dengan membangun sebuah kerangka kerja berbasis data untuk:

1.  **Mengukur Dampak Harga:** Memahami secara kuantitatif seberapa besar pengaruh perubahan harga terhadap volume penjualan (elastisitas).
2.  **Meramalkan Permintaan:** Membuat model peramalan yang akurat untuk memprediksi penjualan di masa depan.
3.  **Mengoptimalkan Inventori:** Mensimulasikan tingkat stok untuk memberikan peringatan dini terhadap potensi risiko *stockout* dan merekomendasikan tindakan.

---

## 🔑 Temuan Utama & Hasil

| Metrik | Hasil | Dampak Bisnis |
| :--- | :--- | :--- |
| **Elastisitas Harga** | **-1.68** | Memberikan dasar kuantitatif untuk strategi penetapan harga dan promosi. |
| **Akurasi Forecast** | **42% lebih baik** dari baseline | Meningkatkan keandalan perencanaan dan mengurangi ketidakpastian. |
| **Analisis Risiko** | **Risiko Stockout teridentifikasi** | Memberikan peringatan dini 3 bulan sebelumnya untuk tindakan korektif. |

---

## ⚙️ Metodologi & Proses Analisis

Analisis ini dibagi menjadi beberapa tahap utama:

### 1. Pembersihan Data (*Data Cleaning*)
Fondasi dari analisis yang baik adalah data yang bersih. Proses ini dilakukan dalam notebook `Data Cleaning.ipynb` dan mencakup:
* Mengubah tipe data tanggal ke format `datetime`.
* Mengidentifikasi dan mengisi **tanggal yang hilang** untuk menjaga kontinuitas data.
* Menangani nilai anomali seperti **harga <= 0**.
* Menghilangkan entri tanggal yang duplikat.
* Mengekspor dataset yang bersih dan siap untuk dianalisis.

### 2. Dekomposisi Time Series
Untuk memahami pola dasar dalam data, digunakan metode **STL (Seasonal and Trend decomposition using Loess)**. Analisis ini mengungkap adanya:
* **Tren Pertumbuhan:** Tren penjualan yang meningkat secara signifikan sejak tahun 2015.
* **Pola Musiman Mingguan:** Siklus penjualan yang berulang setiap minggu (misalnya, puncak di akhir pekan).

![Grafik Dekomposisi STL](Grafik%20Dekomposisi%20STL.png)

### 3. Analisis Elastisitas Harga
* **Model Awal:** Regresi linear sederhana antara harga dan penjualan menunjukkan hubungan yang menyesatkan karena tidak memperhitungkan tren dan musiman.
![Scatter Plot Harga vs Penjualan Awal](Scatter%20Plot%20Harga%20vs%20Penjualan%20Awal.png)
* **Model Robust:** Dengan menggunakan **Regresi OLS** yang memasukkan komponen tren dan musiman sebagai variabel kontrol, elastisitas harga yang sebenarnya berhasil diungkap.

### 4. Peramalan & Simulasi Inventori
* Model peramalan dikembangkan untuk memprediksi permintaan harian. Kinerja model divalidasi selama 90 hari dan terbukti 42% lebih akurat daripada metode baseline, seperti yang ditunjukkan di bawah ini.

![Grafik Validasi Aktual vs Prediksi](Grafik%20Validasi%20Aktual%20vs%20Prediksi.png)

* Hasil ramalan yang valid ini kemudian dimasukkan ke dalam **simulasi inventori** dengan kebijakan **Reorder Point (ROP)** dinamis untuk memproyeksikan tingkat stok di masa depan dan menandai minggu-minggu berisiko.

---

## 🛠️ Tools & Library yang Digunakan
* **Bahasa:** Python
* **Library:**
    * `pandas` untuk manipulasi data
    * `statsmodels` untuk analisis statistik dan regresi (OLS & STL)
    * `matplotlib` untuk visualisasi data
* **Lingkungan:** Jupyter Notebook

---

## 🚀 Cara Menjalankan Proyek
1.  **Clone repositori ini:**
    ```bash
    git clone [URL_REPOSITORY_ANDA]
    ```
2.  **Install dependensi yang dibutuhkan:**
    ```bash
    pip install -r requirements.txt
    ```
3.  **Jalankan notebook secara berurutan:**
    * Mulai dengan `Data Cleaning.ipynb` untuk menghasilkan dataset yang bersih.
    * Lanjutkan dengan notebook analisis utama (`Main Analysis.ipynb` atau nama lain yang Anda gunakan).
