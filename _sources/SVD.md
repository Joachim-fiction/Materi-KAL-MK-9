# Analisis Kompresi dan Klasifikasi Wajah Menggunakan Metode Singular Value Decomposition (SVD)

Dokumentasi ini disusun sebagai laporan analisis dan penjelasan matematis mengenai implementasi **Singular Value Decomposition (SVD)** dalam melakukan dekomposisi matriks gambar, reduksi dimensi, hingga perhitungan geometri untuk pengenalan wajah (*Face Recognition*).

---

## 📘 1. Teori Dasar Matematika SVD

Dalam cabang ilmu Aljabar Linier, **Singular Value Decomposition (SVD)** adalah teorema faktorisasasi yang menyatakan bahwa setiap matriks riil atau kompleks $A$ berukuran $m \times n$ dapat dipecah menjadi perkalian tiga matriks terpisah yang saling berhubungan:

$$A = U \Sigma V^T$$

Pada proyek praktikum ini, matriks data utama kita ($A$) memiliki dimensi $35.000 \times 30$, yang terbentuk dari 30 total sampel foto wajah (20 data wajah *male* dan 10 data wajah *female*) di mana masing-masing foto berukuran $200 \times 175 = 35.000$ piksel yang telah diubah menjadi vektor kolom tunggal.

### Anatomi Komponen Matriks Hasil Dekomposisi SVD:

1. **Matriks $U$ (Left Singular Vectors):**
   - Matriks ini berukuran $m \times m$ (atau $m \times n$ pada versi *Thin SVD*).
   - Kolom-kolom di dalam matriks $U$ bersifat ortonormal, memenuhi syarat matematis $U^T U = I$.
   - **Interpretasi Fisik:** Dalam sistem rekognisi wajah, kolom-kolom ini bertindak sebagai **Eigenfaces** (sering disebut wajah hantu). Setiap kolom menangkap pola variasi geometri vertikal utama—seperti letak rongga mata, struktur rahang, atau gradasi pencahayaan—yang diekstrak dari seluruh database foto training.

2. **Matriks $\Sigma$ (Singular Values):**
   - Matriks diagonal berukuran $m \times n$ (atau $n \times n$ pada versi *Reduced SVD*).
   - Nilai-nilai pada diagonal utama matriks ini disebut Nilai Singular ($\sigma_i$), yang secara sistematis diurutkan dari nilai terbesar hingga terkecil:
     $$\sigma_1 \ge \sigma_2 \ge \sigma_3 \ge \dots \ge \sigma_r > 0$$
   - **Interpretasi Fisik:** Nilai singular mencerminkan tingkat kekuatan atau besaran varians informasi yang dikandung oleh komponen basis *Eigenface* yang bersesuaian. Nilai $\sigma_1$ memuat magnitudo variasi informasi terbesar (seperti pencahayaan global), sementara nilai singular pada indeks-indeks akhir umumnya hanya merepresentasikan *noise* atau detail kecil yang tidak signifikan.

3. **Matriks $V^T$ (Right Singular Vectors - Transpose):**
   - Matriks ortonormal berukuran $n \times n$, memenuhi syarat $V^T V = I$.
   - **Interpretasi Fisik:** Matriks ini memuat koefisien bobot atau koordinat proyeksi dari masing-masing foto wajah asli terhadap ruang representasi baru (*Eigenspace*) yang dibentuk oleh matriks $U$.

---

## 📐 2. Kaitan SVD dengan Eigenvalue ($\lambda$)

Perhitungan nilai singular pada dekomposisi SVD memiliki hubungan matematis yang sangat erat dengan konsep **Eigenvalue** ($\lambda$) dan **Eigenvector** pada matriks kovarians.

Jika kita membentuk matriks persegi hasil perkalian dalam (*inner product*) dari data kita, yaitu $A^T A$ (berukuran $n \times n$), maka secara teoritis:

$$A^T A = (U \Sigma V^T)^T (U \Sigma V^T) = V \Sigma^T U^T U \Sigma V^T$$

Karena $U$ adalah matriks ortonormal ($U^T U = I$), persamaan di atas menyusut menjadi bentuk dekomposisi spektral standar:

$$A^T A = V \Sigma^2 V^T$$

Dari penurunan rumus Aljabar Linier tersebut, kita dapat menarik kesimpulan mutlak bahwa:
- Kolom-kolom dari matriks $V$ adalah **Eigenvector** dari matriks hubungan $A^T A$.
- Nilai dari **Eigenvalue** ($\lambda_i$) berbanding lurus sebagai kuadrat dari Nilai Singular ($\sigma_i$) yang kita dapatkan dari proses SVD:
  $$\lambda_i = \sigma_i^2$$

Menghitung SVD secara langsung pada matriks data $A$ jauh lebih disukai dalam komputasi numerik komputer karena algoritma SVD memiliki tingkat stabilitas dan presisi yang jauh lebih tinggi (menghindari eror pembulatan floating-point) dibandingkan jika kita harus menghitung matriks kovarians $A^T A$ secara manual terlebih dahulu.

---

## 🎯 3. Konsep Reduksi Dimensi dan Pengenalan Wajah

Melalui teorema *Eckart-Young-Mirsky*, SVD memungkinkan kita melakukan reduksi dimensi data tanpa kehilangan karakteristik utama objek gambar. Kita dapat memilih untuk hanya mengambil $k$ buah kolom pertama yang memiliki nilai singular terbesar untuk membentuk ruang bagian (*subspace*) baru yang disebut **Eigenspace** ($\Phi = U_{:, :k}$).

### Alur Logika Klasifikasi Geometri:
1. **Peta Koordinat Kelompok:** Setiap foto training diproyeksikan ke dalam ruang bagian $k$-dimensi tersebut untuk mendapatkan koordinat posisinya.
2. **Penentuan Centroid:** Sistem menghitung nilai rata-rata posisi (*mean vector/centroid*) untuk kelompok *Male* ($\mathbf{c}_{male}$) berdasarkan 20 foto pertama, dan kelompok *Female* ($\mathbf{c}_{female}$) berdasarkan 10 foto berikutnya.
3. **Pengujian Foto Baru:** Ketika ada file foto tantangan baru ($\mathbf{x}$), foto tersebut diproyeksikan menjadi vektor koordinat $\mathbf{\Omega} = \Phi^T \mathbf{x}$.
4. **Keputusan Jarak Geometris:** Jarak Euclidean dihitung dari koordinat wajah baru tersebut menuju masing-masing *centroid*:
   - $d_{male} = \|\mathbf{\Omega} - \mathbf{c}_{male}\|$
   - $d_{female} = \|\mathbf{\Omega} - \mathbf{c}_{female}\|$
   
   Sistem secara otomatis mengklasifikasikan bahwa wajah baru tersebut memiliki kemiripan gender dengan kelompok yang memiliki jarak Euclidean terkecil.

---

## 🔗 4. Link Akses Program Interaktif

Program demonstrasi interaktif *Dual-Mode* (Simulasi & Database Foto Asli 20 Laki-laki + 10 Perempuan) yang dilengkapi dengan fitur *Form Fields* visual dapat diakses langsung oleh Dosen penguji melalui tautan di bawah ini:

**[Google Colab Notebook - Praktikum SVD Interaktif](https://colab.research.google.com/drive/1_1hXeEx-2mbcqy2w4lCcM62-dSDkb1eL?usp=sharing)**