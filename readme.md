# Laporan Proyek Machine Learning - Dany Eka Saputra

## Project Overview (Ulasan Proyek)

### Latar Belakang
Kemajuan teknologi digital membawa dampak signifikan di dunia hiburan, terutama dengan melimpahnya pilihan film. Meski demikian, banyaknya opsi membuat pengguna kesulitan menentukan film yang cocok. Oleh karena itu, sistem rekomendasi diperlukan untuk membantu pengguna menemukan film yang sesuai dengan minat mereka.

### Pentingnya Proyek
Alasan pentingnya pengembangan sistem ini meliputi:

1.  Pengalaman Pengguna yang Terpersonalisasi:
Rekomendasi membantu pengguna menemukan film favorit mereka dengan cepat.

2. Peningkatan Interaksi:
Rekomendasi yang tepat mendorong pengguna lebih sering menggunakan platform.

3. Keunggulan Kompetitif:
Sistem rekomendasi yang efektif membuat platform unggul di antara kompetitor.

### Hasil Riset dan Referensi
Penelitian sebelumnya menunjukkan efektivitas algoritma Content-Based Filtering dan Collaborative Filtering dalam sistem rekomendasi. Collaborative Filtering sebagai metode yang efektif dalam merekomendasikan item berdasarkan interaksi pengguna[1]. Sementara itu, bagaimana Content-Based Filtering dapat memanfaatkan atribut konten untuk memberikan rekomendasi yang lebih personal[2]. Dataset MovieLens 100K mendukung pengembangan sistem ini dengan menyediakan data interaksi yang lengkap.[3].

## Business Understanding

### Klarifikasi Masalah
Dalam pengembangan sistem rekomendasi film, penting untuk mengidentifikasi tantangan utama yang dihadapi pengguna dan platform. Tiga masalah kunci yang perlu ditangani adalah:

1. Terlalu Banyak Pilihan: Membuat pengguna kebingungan memilih film.
2. Cold Start: Data interaksi terbatas untuk pengguna atau film baru.
3. Data Sparse: Kurangnya rating/interaksi menyebabkan model kurang optimal.

### Problem Statements (Pernyataan Masalah)
1. Bagaimana membangun sistem rekomendasi yang akurat dan sesuai preferensi pengguna?
2. Bagaimana mengatasi cold start dan kelangkaan data dalam MovieLens 100K?

### Goals (Tujuan)
1. Membangun sistem rekomendasi berbasis Content-Based dan Collaborative Filtering.
2. Mengoptimalkan algoritma Cosine Similarity dan SVD untuk menangani cold start dan sparsity.

### Solution Approach
Untuk mencapai tujuan, dua pendekatan solusi akan diimplementasikan:

#### 1. Content-Based Filtering (CBF) dengan Cosine Similarity
Pendekatan ini berfokus pada analisis atribut konten dari film, seperti genre, aktor, sutradara, dan deskripsi film. Tahapannya: ekstraksi fitur → representasi vektor → perhitungan kemiripan dengan cosine similarity → rekomendasi film serupa.

#### 2. Collaborative Filtering (CF) dengan Singular Value Decomposition (SVD)
Pendekatan ini memanfaatkan interaksi pengguna dengan film untuk menemukan pola dan hubungan tersembunyi antara pengguna dan item. Berikut langkah-langkahnya: Matrix Factorization (Menganalisis pola interaksi pengguna-film dengan metode dekomposisi matriks) → SVD digunakan untuk memprediksi rating film yang belum diberi nilai oleh pengguna  → lalu menyarankan film dengan prediksi tertinggi.

Dengan membangun dua pendekatan, sistem rekomendasi diharapkan memberikan hasil lebih akurat dan relevan, serta mengatasi masalah cold start dan sparsity.


## Data Understanding
Dataset yang digunakan dalam proyek ini adalah MovieLens100K, yang berisi 100.000 penilaian dari pengguna untuk berbagai film, dataset ini terdiri dari 4 komponen data yang berbeda yaitu Links Dataset, Movies Dataset, Ratings Dataset, dan Tags Dataset. Dataset diambil dari [https://grouplens.org/datasets/movielens/100k/]
Dataset MovieLens 100K terdiri dari 4 bagian:
1. Links Dataset (links.csv)
   - Jumlah Data: 9742 baris dan 3 kolom.
   - Uraian Fitur:
     - movieId: ID unik untuk setiap film.
     - imdbId: ID film di IMDB.
     - tmdbId: ID film di TMDB.
   - Kondisi Data:
     - Missing Values: Terdapat 8 nilai kosong pada kolom tmdbId.
     - Duplikat: Tidak ada data duplikat.
     - Outlier: Tidak dilakukan analisis outlier secara spesifik pada tahap ini.
3. Movies Dataset (movies.csv):
   - Jumlah Data: 9742 baris dan 3 kolom.
   - Uraian Fitur:
     - movieId: ID unik untuk setiap film.
     - title: Judul film (termasuk tahun rilis dalam format string).
     - genres: Genre film (dipisahkan oleh karakter |).
    - Kondisi Data:
      - Missing Values: Tidak ada nilai kosong.
      - Duplikat: Tidak ada data duplikat.
      - Outlier: Tidak dilakukan analisis outlier secara spesifik pada tahap ini.
5. Ratings Dataset (ratings.csv):
   - Jumlah Data: 100836 baris dan 4 kolom.
   - Uraian Fitur:
     - userId: ID unik untuk setiap pengguna.
     - movieId: ID unik untuk setiap film yang diberi rating.
     - rating: Rating yang diberikan pengguna untuk film (skala 0.5 hingga 5.0).
     - timestamp: Waktu pemberian rating dalam format Unix timestamp.
   - Kondisi Data:
     - Missing Values: Tidak ada nilai kosong.
     - Duplikat: Tidak ada data duplikat.
     - Outlier: Tidak dilakukan analisis outlier secara spesifik pada tahap ini. Skala rating yang digunakan adalah 0.5 hingga 5.0.
7. Tags Dataset (tags.csv)
   - Jumlah Data: 3683 baris dan 4 kolom.
   - Uraian Fitur:
     - userId: ID unik untuk setiap pengguna.
     - movieId: ID unik untuk setiap film yang diberi tag.
     - tag: Label atau kata kunci yang ditambahkan pengguna untuk film.
     - timestamp: Waktu pemberian tag dalam format Unix timestamp.
     - Kondisi Data:
     - Missing Values: Tidak ada nilai kosong.
     - Duplikat: Tidak ada data duplikat.
     - Outlier: Tidak dilakukan analisis outlier secara spesifik pada tahap ini.

## Data Preparation
Pada tahap data preparation dilakukan transformasi berupa pembersihan judul film yang masih mengandung tahun rilis film, melakukan pembuatan kolom baru bernama features dengan cara menggabungkan judul film dan genre.

1. Penggabungan Data Awal dan Penanganan Missing Values:
   - Dataset movies.csv digabungkan dengan kolom tmdbId dari links.csv menggunakan movieId sebagai kunci gabungan (merge dengan    how='left').
   - Baris yang memiliki nilai tmdbId kosong (NaN) dihapus dari dataframe hasil gabungan (movies_df.dropna(subset=['tmdbId'])). Ini mengatasi 8 missing values yang teridentifikasi pada kolom tmdbId di links.csv.

2. Pembersihan dan Transformasi Fitur pada movies_df:
   - Ekstraksi Tahun Rilis: Kolom year dibuat dengan mengekstrak tahun (4 digit angka) dari kolom title.
   - Pembersihan Judul Film: Tahun rilis dihapus dari kolom title untuk mendapatkan judul film yang bersih.
   - Penanganan Missing Values pada year: Nilai kosong pada kolom year (jika ada setelah ekstraksi) diisi dengan modus (nilai yang paling sering muncul) dari kolom year, kemudian tipe datanya diubah menjadi integer.
   - Transformasi Genre: Karakter | pada kolom genres diganti dengan spasi untuk persiapan proses TF-IDF.
   - Pembuatan Kolom features (untuk Content-Based Filtering): Kolom baru bernama features dibuat dengan menggabungkan isi dari kolom title (yang sudah dibersihkan) dan kolom genres (yang sudah ditransformasi).
     
3. Penghapusan Kolom timestamp:
   - Kolom timestamp dihapus dari dataframe ratings_df.
   - Kolom timestamp dihapus dari dataframe tags_df.
   - Catatan: Konversi timestamp ke datetime tidak dilakukan secara eksplisit karena kolom ini dihapus sebelum digunakan dalam pemodelan.

4. Penggabungan Dataframe Utama (df):
   - Dataframe ratings_df (tanpa timestamp) digabungkan dengan movies_df (yang sudah diproses) menggunakan movieId sebagai kunci (merge dengan how='inner').
   - Hasil gabungan tersebut kemudian digabungkan lagi dengan tags_df (tanpa timestamp) menggunakan userId dan movieId sebagai kunci (merge dengan how='left').
5. Penghapusan Baris Kosong pada tag:
   - Baris yang memiliki nilai tag kosong (NaN) setelah penggabungan terakhir dihapus dari dataframe df (df.dropna(subset=['tag'])).

## Data Preparation untuk Modelling Content Based Filtering
   - Menggunakan TF-IDF (Term Frequency-Inverse Document Frequency) untuk mengubah kolom features (gabungan judul dan genre) pada movies_df menjadi representasi vektor numerik. Ini dilakukan dengan TfidfVectorizer dari sklearn.
   - Menghitung Cosine Similarity antar film berdasarkan matriks TF-IDF untuk menemukan kemiripan konten antar film.

## Data Preparation untuk Modelling Collaborative Filtering  
   - Menggunakan data rating pengguna (userId), movieId, dan rating dari ratings_df sebagai input.
   - Pengaturan Skala Rating: Skala rating didefinisikan secara eksplisit antara 0.5 hingga 5.0 menggunakan objek Reader dari pustaka Surprise (Reader(rating_scale=(0.5, 5.0))). Ini penting untuk memastikan model SVD memahami rentang nilai rating yang mungkin.
   - Dataset dimuat ke dalam format yang sesuai untuk pustaka Surprise menggunakan Dataset.load_from_df.
   - Melakukan pembagian data menjadi data latih (trainset) dan data uji (testset) dengan proporsi 80:20 menggunakan fungsi train_test_split dari pustaka Surprise.

#### Cara Kerja dan Parameter
Langkah-langkah pada *content-based filtering*:
-  Cosine Similarity : Untuk menghitung kesamaan antarfilm, digunakan *cosine similarity*, yang mengukur sudut antarvektor dalam ruang multidimensi. Hasil nilai *cosine similarity* berkisar antara 0 hingga 1, dengan nilai 1 menandakan film sangat mirip dan 0 menandakan tidak ada kesamaan.
 
#### Alur Kerja Fungsi `recommend_movies`
1.  Input Film : Pengguna memberikan judul film (misalnya "Inception") sebagai input.
2.  Pencarian Indeks : Sistem mencari indeks film berdasarkan judul input di dataframe `movies`.
3.  Perhitungan Similarity : Cosine similarity digunakan untuk mengukur kesamaan antara film input dan seluruh film dalam dataset. Cosine similarity menghitung kemiripan dua vektor dengan membandingkan sudut di antara keduanya, di mana nilai 1 berarti sangat mirip dan 0 berarti tidak mirip.
4.  Penyortiran Skor Kesamaan : Semua film diurutkan berdasarkan skor kesamaan dengan film input, dari yang paling mirip hingga yang paling rendah.
5.  Pemilihan Top-10 Rekomendasi : Sistem mengabaikan film input dan memilih 10 film paling mirip berdasarkan skor tertinggi.
6.  Output : Mengembalikan judul, ID, dan genre dari 10 film teratas sebagai rekomendasi.

#### Rekomendasi
Setelah menghitung kesamaan antarfilm, saya mengurutkan film berdasarkan skor kesamaan tertinggi dan merekomendasikan 10 film teratas yang paling mirip dengan film input. Berikut adalah contoh hasil rekomendasi berbasis *content-based filtering* untuk film  "Jumanji" ,  dan  "Toy Story" :

![jumanji](https://github.com/user-attachments/assets/9c294436-8024-456e-8c5a-a40e0b7eadc1)

![toy story](https://github.com/user-attachments/assets/46ee332e-8149-4b6f-9b64-c7be4a28bd51)

### Collaborative Filtering
Model *collaborative filtering* dalam proyek ini dilatih menggunakan data rating pengguna terhadap film. Model mempelajari pola preferensi dengan menguraikan matriks rating menjadi beberapa faktor laten (fitur yang tidak terlihat secara langsung) yang merepresentasikan hubungan antara pengguna dan film.

#### Cara Kerja dan Parameter yang Digunakan

Langkah-langkah dalam membangun dan mengevaluasi model ini adalah sebagai berikut:

#### Alur Kerja Sistem Rekomendasi
1.  Pemodelan dengan SVD :  
   - SVD adalah teknik dekomposisi matriks yang memecah matriks besar (user-item ratings) menjadi representasi lebih kecil, menangkap hubungan laten antara pengguna dan item.  
   - Setelah model dilatih pada data training, SVD menghasilkan faktor-faktor laten yang menggambarkan preferensi pengguna dan karakteristik film.

2.  Prediksi Rating :  
   - Setelah pelatihan, model digunakan untuk memprediksi rating yang mungkin diberikan oleh pengguna pada film yang belum mereka tonton.

3.  Fungsi `get_recommendations` :  
   - Fungsi ini menerima  user_id  dan dataframe film sebagai input.  
   - Pertama, sistem mengidentifikasi film yang sudah dirating oleh pengguna dan memisahkan film yang belum pernah ditonton (unrated).  
   - Untuk setiap film yang belum dirating, sistem membuat prediksi menggunakan metode `model.predict()`.  
   - Prediksi ini diurutkan dari yang tertinggi ke terendah, dan 10 film teratas dipilih sebagai rekomendasi.

4.  Output :  
   - Fungsi mengembalikan judul dan genre dari 10 film terbaik sebagai rekomendasi untuk pengguna tersebut.

### Teknologi yang Digunakan
-  SVD (Singular Value Decomposition) :  
  SVD digunakan untuk memetakan pengguna dan film ke dalam ruang laten, memprediksi rating bahkan ketika sebagian besar data rating tidak tersedia (sparse matrix). Algoritma ini sangat efektif untuk rekomendasi karena dapat menangkap pola-pola tersembunyi dalam data interaksi pengguna dan film.

Pendekatan  Collaborative Filtering dengan SVD  ini unggul karena tidak memerlukan fitur eksplisit dari item (film), melainkan hanya berdasarkan pola interaksi pengguna. Dengan demikian, sistem dapat merekomendasikan film yang relevan meskipun pengguna dan item baru tidak memiliki deskripsi atau fitur yang lengkap.


### 4. Rekomendasi Top N Film
Berikut adalah rekomendasi untuk pengguna dengan  ID 331,  ID 1  berdasarkan prediksi rating tertinggi yang dihasilkan oleh model SVD:

![user 331](https://github.com/user-attachments/assets/896f9ec0-f67b-4e78-aecb-019fa6e06e61)

![user 1](https://github.com/user-attachments/assets/31ca92de-b198-4aae-b838-ebe6eeedddd8)



### Kelebihan dan Kekurangan

#### Content-Based Filtering
- **Kelebihan:**  
  - Memberikan rekomendasi yang personal berdasarkan karakteristik film (genre, judul).  
  - Cocok untuk pengguna baru tanpa riwayat interaksi.

- **Kekurangan:**  
  - Rekomendasi kurang bervariasi karena hanya berdasarkan konten yang mirip.  
  - Kesulitan menangani item baru tanpa metadata lengkap.  
  - Terbatas pada preferensi yang sudah diketahui dari pengguna.

#### Collaborative Filtering (SVD)
- **Kelebihan:**  
  - Dapat mengungkap pola hubungan kompleks antar pengguna dan film.  
  - Efisien dalam menangani data besar melalui reduksi dimensi.

- **Kekurangan:**  
  - Tidak cocok untuk pengguna baru tanpa data interaksi (cold start).  
  - Kinerja bisa menurun pada data yang sparse.  
  - Memerlukan sumber daya komputasi tinggi untuk dekomposisi matriks.

> **Kesimpulan:**  
Kedua pendekatan saling melengkapi. Menggabungkan keduanya dapat mengatasi kelemahan masing-masing dan meningkatkan kualitas rekomendasi secara keseluruhan.

## Evaluation

### Content Based Filtering
### Metrik Evaluasi yang Digunakan
Berikut adalah metrik evaluasi yang digunakan untuk mengukur kinerja sistem rekomendasi:

1. *Precision@K*
2. *Recall@K*

#### 1. Precision@K
Precision@K mengukur proporsi rekomendasi yang relevan di antara K rekomendasi teratas yang diberikan oleh sistem. Precision tinggi menunjukkan bahwa sistem mampu memberikan rekomendasi yang relevan untuk pengguna.

- *Formula Precision@K*:
   (Jumlah item yang relevan dalam top K) / K</span>

- *Cara Kerja Precision@K*:
  Precision@K bekerja dengan menghitung berapa banyak film yang relevan di antara K film yang direkomendasikan pertama. Jika K diatur ke 10, maka Precision@10 akan menunjukkan persentase film relevan dari 10 rekomendasi teratas.

#### 2. Recall@K
Recall@K mengukur proporsi item relevan yang direkomendasikan oleh sistem dari seluruh item relevan yang tersedia. Metrik ini membantu memastikan bahwa sistem tidak melewatkan item yang relevan.

- *Formula Recall@K*:
(Jumlah item yang relevan dalam top K) / (Jumlah total item relevan)

- *Cara Kerja Recall@K*:
  Recall@K menghitung persentase item relevan yang direkomendasikan dari seluruh item relevan untuk setiap judul. Misalnya, jika ada 20 film yang relevan untuk "Toy Story" dan sistem merekomendasikan 10 di antaranya di posisi teratas, maka Recall@10 akan menunjukkan seberapa banyak item relevan yang ditemukan dari seluruh item relevan.

### Hasil Evaluasi Proyek Berdasarkan Metrik
Evaluasi sistem rekomendasi ini dilakukan pada beberapa judul film dengan ground truth yang telah ditentukan. Berikut hasil evaluasinya dengan menggunakan nilai \( K = 10 \) pada metrik di atas:

- *Precision@10*: Seberapa banyak film yang relevan dalam K rekomendasi teratas. Precision yang tinggi menunjukkan bahwa rekomendasi yang diberikan cukup relevan dengan input pengguna.

- *Recall@10*: Seberapa banyak film relevan berhasil ditemukan dari total yang tersedia. Recall yang tinggi menunjukkan bahwa sistem mampu menemukan sebagian besar film relevan untuk setiap judul.

### 3. Hasil Evaluasi
Berdasarkan hasil evaluasi, didapat performa sistem rekomendasi pada dua film input, *"Toy Story"* dan *"Jumanji"*:

- *Toy Story:* Precision@5 = 74.06%, Recall@5 = 71.55%  
- *Jumanji:* Precision@5 = 12.85%, Recall@5 = 100%

  Untuk film *Toy Story*, *precision* sebesar 74.06% menunjukkan bahwa sebagian besar dari 5 rekomendasi teratas memiliki kesamaan yang cukup kuat dengan film input. *Recall* sebesar 71.55% menunjukkan bahwa dari seluruh rekomendasi yang relevan, model telah berhasil menangkap sekitar 71% film mirip dalam rekomendasi teratas. 

  Untuk *Jumanji*, hasil *precision* yang rendah sebesar 12.85% menunjukkan bahwa model tidak memberikan rekomendasi yang secara ketat relevan di antara *top 5*. Namun, nilai *recall* 100% menunjukkan bahwa meskipun ketepatan rekomendasi di antara *top 5* kurang, model berhasil menangkap seluruh film yang relevan yang mungkin mirip dengan *Jumanji* dalam *cosine similarity*.

Metrik evaluasi *precision* dan *recall* digunakan untuk memastikan bahwa sistem rekomendasi memiliki relevansi dan cakupan yang baik, sesuai dengan kebutuhan pengguna. Dalam proyek ini, hasil menunjukkan bahwa untuk film tertentu seperti *Toy Story*, sistem berhasil memberikan rekomendasi yang cukup relevan dengan tingkat akurasi dan cakupan yang seimbang. Namun, untuk film seperti *Jumanji*, meskipun rekomendasi memiliki cakupan penuh (*recall* tinggi), ketepatan dalam memilih film-film yang sangat mirip masih memerlukan peningkatan.

### Collaborative Filtering
### Metrik Evaluasi
#### 1. Root Mean Squared Error (RMSE)
 RMSE  adalah  Mengukur seberapa besar deviasi rata-rata prediksi dari nilai asli. RMSE dihitung dengan rumus berikut:

RMSE = √( (1 / N) * Σ(y_i - ŷ_i)^2 )

Nilai RMSE yang lebih rendah menunjukkan bahwa model lebih akurat dalam memprediksi rating.

#### 2. Mean Absolute Error (MAE)
 MAE  adalah Rata-rata perbedaan absolut antar prediksi dan nilai asli. MAE dihitung dengan rumus berikut:

MAE = (1 / N) * Σ|y_i - ŷ_i|

Nilai MAE yang lebih rendah menandakan bahwa model memiliki kinerja prediksi yang lebih baik.

### Alasan Penggunaan RMSE dan MAE
RMSE dan MAE sama-sama penting untuk mengevaluasi kinerja model prediksi rating film. Perbedaan utama antara keduanya adalah RMSE lebih menekankan pada kesalahan besar, sehingga lebih sensitif terhadap prediksi yang jauh dari nilai sebenarnya. Sementara itu, MAE memberikan gambaran yang lebih stabil tentang kesalahan prediksi, sehingga membantu dalam menilai konsistensi prediksi.

### Hasil Proyek Berdasarkan Metrik Evaluasi
-  RMSE : 0.8795
-  MAE : 0.6765

Nilai RMSE sebesar 0.8795 menunjukkan bahwa rata-rata kesalahan kuadrat dari prediksi model relatif kecil, yang menunjukkan bahwa model cukup akurat dalam memberikan prediksi rating. Nilai MAE sebesar 0.6765 menunjukkan rata-rata kesalahan absolut yang rendah, yang berarti model menghasilkan prediksi yang konsisten. Kedua metrik ini mengindikasikan bahwa model dapat memberikan rekomendasi film dengan cukup baik.

### Fungsi Rekomendasi
Fungsi `get_recommendations` digunakan untuk memberikan rekomendasi film kepada pengguna tertentu. Fungsi ini bekerja dengan:
1. Mengambil daftar film yang belum ditonton oleh pengguna.
2. Menghitung prediksi rating untuk film yang belum ditonton.
3. Mengurutkan film berdasarkan prediksi rating secara menurun dan memilih film dengan prediksi rating tertinggi.

Hasil dari fungsi ini adalah daftar 10 rekomendasi film teratas untuk pengguna berdasarkan preferensi pengguna serupa lainnya.
Sistem rekomendasi film berbasis collaborative filtering menggunakan SVD berhasil dibangun dan dievaluasi dengan RMSE dan MAE. Metrik evaluasi menunjukkan bahwa model memiliki akurasi prediksi yang baik, menjadikannya solusi yang efektif untuk memberikan rekomendasi film yang relevan kepada pengguna.

## Kesimpulan  
Proyek ini berhasil membangun dua pendekatan sistem rekomendasi:
1. Mengatasi pilihan yang berlebihan dengan sistem berbasis konten  
2. Menangani cold start dan sparsity menggunakan SVD  
3. Mengoptimalkan algoritma cosine similarity dan SVD untuk akurasi lebih tinggi  
4. Menghasilkan rekomendasi yang lebih personal dan relevan  

Dua pendekatan ini saling melengkapi, menciptakan sistem rekomendasi film yang kuat dan efektif.

## Referensi
[1] B. Sarwar et al., "Item-based collaborative filtering recommendation algorithms," Proc. 10th Int. Conf. World Wide Web, Hong Kong, 2001, pp. 285-295.
[2] M. Pazzani dan D. Billsus, "Content-based recommendation systems," dalam The Adaptive Web, Springer, 2007, pp. 325-341.[https://link.springer.com/chapter/10.1007/978-3-540-72079-9_10]
[3] MovieLens Dataset. (n.d.). Retrieved from [https://grouplens.org/datasets/movielens/100k/]
