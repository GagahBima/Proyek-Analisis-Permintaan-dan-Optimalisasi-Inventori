# Proyek Analisis Permintaan dan Simulasi Inventori Prediktif

Repositori ini berisi analisis data penjualan untuk:
- mengestimasi sensitivitas permintaan terhadap harga,
- membangun model peramalan permintaan harian, dan
- mensimulasikan level inventori untuk mengidentifikasi risiko kekurangan stok (*stockout*) secara proaktif.

Fokus proyek ini adalah **membangun kerangka kerja analitis** yang rasional dan transparan untuk mendukung pengambilan keputusan harga dan inventori, bukan (belum) sebuah solusi enterprise yang sepenuhnya otomatis.

---

## Latar Belakang & Tujuan Proyek

Manajemen inventori yang reaktif sering kali menyebabkan dua masalah utama:

1. **Kehilangan penjualan** karena stok habis saat permintaan sedang tinggi.
2. **Biaya penyimpanan yang tinggi** karena stok berlebih yang tidak perlu.

Proyek ini bertujuan untuk menyusun sebuah *Proof of Concept (PoC)* berbasis data yang dapat:

1. **Mengukur Dampak Harga (Elastisitas Lokal)**  
   Mengkuantifikasi bagaimana perubahan harga berkorelasi dengan perubahan volume penjualan, dengan kontrol terhadap faktor musiman dan ketersediaan stok.

2. **Meramalkan Permintaan Harian**  
   Membangun model peramalan yang lebih akurat daripada baseline sederhana (misalnya *naive* 7-hari), sebagai dasar perencanaan stok.

3. **Mendukung Keputusan Inventori Prediktif**  
   Menggunakan hasil peramalan untuk mensimulasikan pergerakan stok ke depan, mengestimasi risiko *stockout*, dan memberi sinyal kapan reorder perlu dilakukan.

> **Catatan penting:**  
> Seluruh analisis di repo ini dilakukan pada **satu SKU** dan periode data tertentu. Angka-angka spesifik (misalnya nilai elastisitas) bersifat **lokal**, sedangkan yang dimaksud untuk “ditransfer” ke produk lain adalah **kerangka kerjanya**, bukan nilai parameternya.

---

## Proses Analisis

Analisis dibagi menjadi beberapa tahap utama:

1. **Pembersihan Data (*Data Cleaning*)**  
   - Melengkapi tanggal yang hilang (membuat time series harian yang kontinu).  
   - Menangani nilai anomali (misalnya harga <= 0).  
   - Menyiapkan variabel-variabel turunan yang diperlukan untuk analisis lanjutan.

2. **Dekomposisi Time Series**  
   Menggunakan metode **STL (Seasonal-Trend decomposition)** untuk memisahkan:
   - tren jangka panjang,
   - pola musiman mingguan,
   - dan komponen residual.  
   Dekomposisi ini membantu memvalidasi adanya pola musiman yang kemudian dikontrol di tahap modeling.

3. **Estimasi Elastisitas Harga (Pendekatan Panel Sederhana)**  
   - Model awal (OLS log-log sederhana) menunjukkan koefisien harga yang **positif**, indikasi kuat adanya bias (misalnya karena musiman dan stok).  
   - Untuk mengurangi bias tersebut, digunakan spesifikasi dengan:
     - **Fixed effects waktu** (misalnya dummy Year-Month) untuk mengontrol tren & musiman tingkat agregat, dan
     - **Filter “stok tinggi”** sehingga regresi hanya menggunakan hari-hari di mana stok awal cukup besar, sehingga penjualan lebih mendekati *unconstrained demand*.  
   - Hasil akhirnya adalah estimasi **elastisitas harga negatif di sekitar -1.6** untuk SKU dan periode ini. Nilai ini harus dibaca sebagai **estimasi praktis/kuasi-kausal lokal**, bukan angka universal.

4. **Peramalan Permintaan (Forecasting)**  
   - Dibangun beberapa model peramalan, termasuk baseline sederhana (misalnya *naive* 7-hari) dan model berbasis **machine learning** (`HistGradientBoostingRegressor`).  
   - Pada data yang dianalisis, model ML menunjukkan **penurunan error sekitar 40%** dibanding baseline, misalnya:
     - MASE model ML ≈ 0.97  
     - MASE baseline ≈ 1.69  
   - Angka ini bersifat spesifik untuk dataset dan horizon uji yang digunakan, tetapi menunjukkan bahwa model yang lebih kaya fitur dapat memberikan dasar yang lebih baik untuk keputusan inventori.

5. **Simulasi Inventori ke Depan**  
   - Menggunakan hasil forecast untuk membangun dua jalur utama:
     - **Permintaan potensial (demand)**: apa yang terjadi jika stok selalu cukup.  
     - **Penjualan terealisasi (realized sales)**: penjualan yang dibatasi oleh stok (menggunakan fungsi `min(demand, stock)` pada tiap hari).  
   - Dengan mensimulasikan stok awal dan kebijakan pemesanan tertentu, kita dapat:
     - mengidentifikasi titik waktu di mana stok akan habis jika tidak ada intervensi,  
     - mengukur potensi *lost sales* pada periode tersebut.

---

## Temuan Utama (Untuk SKU & Periode yang Dianalisis)

### 1. Estimasi Elastisitas Harga Lokal (~ -1.6)

Model dengan kontrol musiman waktu dan filter stok tinggi menghasilkan estimasi elastisitas harga sekitar **-1.6**.  
Secara interpretasi:

> Kenaikan harga 1% berkorelasi dengan penurunan volume penjualan sekitar 1.6%,  
> **dengan asumsi** bulan yang sama, ketersediaan stok memadai, dan faktor lain relatif konstan.

Elastisitas ini:

- **tidak dimaksudkan sebagai angka universal**,  
- dan tidak setara dengan hasil dari eksperimen terkontrol atau model instrumental variable,  
namun cukup informatif untuk digunakan sebagai **order-of-magnitude** dalam diskusi strategi harga SKU ini.

---

### 2. Model Peramalan Lebih Akurat dari Baseline

Model peramalan berbasis ML:

- menunjukkan error yang lebih rendah dibanding baseline naive harian,  
- sehingga memungkinkan kita merancang kebijakan inventori dengan **safety stock yang lebih tipis** untuk service level yang sama (secara teoritis, safety stock meningkat seiring variabilitas error).

Detail teknis lengkap (pemilihan fitur, metrik evaluasi, dan perbandingan antar model) tersedia di notebook `Main__Analysis.ipynb`.

---

### 3. Simulasi Mengungkap Risiko Stockout pada Skenario Tertentu

Simulasi ke depan (single-path) dengan asumsi:

- stok awal tertentu,
- lead time tetap,
- dan kebijakan pemesanan yang tidak adaptif terhadap lonjakan demand,

menunjukkan bahwa:

- **jalur permintaan potensial** tetap tinggi pada periode tertentu,  
- sedangkan **jalur penjualan terealisasi** jatuh ke nol ketika stok habis.

Dalam skenario yang dianalisis, hal ini terjadi di sekitar **pertengahan September**:  
jika perusahaan tidak melakukan reorder lebih awal, stok diproyeksikan habis sementara permintaan pasar masih ada.

![Grafik Simulasi Inventori dan Proyeksi Stockout](images/Screenshot.png)

> **Penting:**  
> Tanggal dan angka spesifik bergantung pada:
> - asumsi stok awal,
> - lead time,
> - dan profil permintaan pada data historis.  
> Simulasi ini dimaksudkan sebagai ilustrasi risiko dan alat diskusi, bukan prediksi tanggal stockout yang pasti.

---

## Batasan & Pekerjaan Lanjutan

Proyek ini adalah **Proof of Concept** dengan beberapa batasan penting:

- Analisis dilakukan pada **satu SKU** dengan data historis tertentu. Untuk ratusan SKU, diperlukan strategi tambahan (misalnya model hierarkis atau pooling).
- Estimasi elastisitas tidak menggunakan *instrumental variable* atau desain eksperimen, sehingga tetap bergantung pada asumsi bahwa variasi harga yang tersisa setelah kontrol cukup mendekati eksogen.
- Simulasi inventori saat ini bersifat **single-path**.  
  Untuk penggunaan di dunia nyata, disarankan menambahkan:
  - **Simulasi Monte Carlo** (ribuan skenario) dengan demand dan lead time acak,
  - sehingga output berupa **profil risiko** (probabilitas stockout per rentang tanggal), bukan satu angka tanggal stockout saja.

Meski demikian, kerangka kerja ini sudah cukup untuk:

- menunjukkan nilai tambah pendekatan **prediktif** dibanding kebijakan inventori yang sepenuhnya reaktif, dan  
- menjadi dasar diskusi dan iterasi lanjut dengan tim bisnis & operasional.

---

## Tools & Library yang Digunakan

- **Bahasa:** Python 3  
- **Data Analysis:** `pandas`, `numpy`  
- **Statistical Modeling:** `statsmodels`  
- **Machine Learning:** `scikit-learn` (termasuk `HistGradientBoostingRegressor`)  
- **Visualization:** `matplotlib`, `seaborn`  
- **Environment:** Jupyter Notebook  

---

## Cara Menjalankan Proyek
1.  **Clone repositori ini:**
    ```bash
    git clone [https://github.com/GagahBima/Proyek-Analisis-Permintaan-dan-Optimalisasi-Inventori]
    ```
2.  **Install dependensi yang dibutuhkan:**
    ```bash
    pip install pandas numpy matplotlib seaborn statsmodels scikit-learn
    ```
3.  **Jalankan notebook secara berurutan:**
    * Mulai dengan `Data_Cleaning.ipynb`.
    * Lanjutkan dengan notebook analisis utama `Main__Analysis.ipynb`.
