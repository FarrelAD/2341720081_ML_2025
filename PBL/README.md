# Deteksi Struk Palsu dari Aplikasi Mobile Banking BNI

## 1. Ringkasan

Studi kasus deteksi struk palsu dari aplikasi mobile banking BNI ini melibatkan salah satu teknik machine learning dalam upaya untuk membantu salah satu alur utama deteksinya, yaitu OCR (Optical Character Recognition). Peran dari machine learning dalam OCR adalah mampu melakukan proses klasifikasi dari data gambar yang ada.


## 2. Dataset

### 2.1 Pengumpulan Dataset

Dataset yang dikumpulkan berupa kumpulan struk asli yang dihasilkan dari aplikasi Wondr, BNI. Total dataset yang diperoleh sekitar ~75 gambar struk dari Bank BNI. Contoh sampel dari dataset ini adalah sebagai berikut:

![Dataset sample (code: 01.png)](dataset/receipts/01.png)

Khusus pada pengembangan model OCR, tidak keseluruhan gambar digunakan sepenuhnya. Hanya beberapa gambar saja yang diperlukan. Dari beberapa gambar tersebut, data gambar dipecah kembali menjadi dataset baru yang terdiri dari 1 karakter per gambar. Karakter-karakter yang dikumpulkan adalah hampir semua jenis karakter yang mencakup:
- alphabetic (a, b, c,  ..., z)
- numeric (0, 1, 2, ..., 9)
- symbols (asterisk (`*`), colon (`:`), dot (`.`), )

### 2.2 Struktur Dataset

Untuk memudahkan pemrosesan data, maka dibuatkan metadata yang mengandung informasi keseluruhan dataset. Metadata ini disusun dengan format CSV. Metadata dapat diakses pada link berikut: [metadata.csv](dataset/individual_characters/metadata.csv)

Bentuk preview dari metadata ini adalah sebagai berikut:
| filepath                     | label |
|------------------------------|-------|
| alphabetic/a/a-01.png        | a     |
| alphabetic/a/a-02.png        | a     |
| alphabetic/a/a-03.png        | a     |
| alphabetic/b/b-01.png        | b     |
| alphabetic/b/b-02.png        | b     |
| alphabetic/b/b-03.png        | b     |
| numeric/1/1-01.png        | 1     |
| numeric/1/1-02.png        | 1     |
| numeric/1/1-03.png        | 1     |

## 3. Pipeline 

### 3.1 Pipeline Sistem Deteksi Struk Palsu

1. Load Model OCR

    Melakukan loading atau memuat model OCR yang telah diproses sebelumnya.

2. Data preprocessing

    Proses data preprocessing ini menyesuaikan tahapan preprocessing yang ada pada pipeline OCR. Hal ini karena tahapan preprocessing di sini lebih mengarah preprocessing untuk gambar yang sudah terpecah menjadi 1 karakter saja.

3. Text segmentation

    Text segmentation adalah proses untuk memetakan bagian mana yang kemungkinan mengandung teks. Sebuah teks biasanya memiliki karakter gambar yang lebih menonjol daripada bagian gambar yang lain.

4. Text extraction

    Text extraction merupakan tahapan untuk melakukan ekstraksi teks dari gambar yang ada. Proses ini dilakukan secara bertahap. Ekstraksi akan diawali dengan ekstrak gambar yang mengandung 1 karakter. Ekstraksi selanjutnya adalah melakukan penggabungan karakter-karakter yang ada menjadi sebuah kata atau kalimat utuh.

5. Verifikasi autentikasi gambar

    Verifikasi autentikasi akan melakukan verifikasi seperti apakah jumlah baris yang dideteksi sama, nilai pada field struk sama, hingga struktur layout mirip dengan struk yang asli.

6. Pengujian

    Proses pengujian dilakukan dengan menguji 2 jenis struk, struk asli dari aplikasi Wondr BNI dan juga struk palsu yang memiliki tampilan berbeda dari struk asli dari aplikasi Wondr BNI.

### 3.2 Pipeline OCR

Proses pipeline OCR bertujuan untuk mampu mendeteksi karakter dari objek gambar yang ada. OCR menjadi pelengkap dari keseluruhan pipeline deteksi struk palsu.Urutan pipeline OCR adalah sebagai berikut:

1. Data preprocessing

    Tahapan preprocessing mencakup beberapa teknik pemrosesan gambar sebagai berikut:
    - konversi ke grayscale,
    - penentuan threshold,
    - menentukan bounding box,
    - melakukan resizing,
    - menambahkan padding pada gambar untuk menambal bagian yang kosong,
    - normalisasi gambar

2. Data splitting

    Data yang telah dilakukan preprocessing akan dibagi menjadi 2 bagian, yaitu data training dan juga testing. Proporsi pembagiannya adalah 25% sebagai data testing dan 75% sebagai data training.

3. Pemodelan machine learning

    Pengujian model machine learning dilakukan dengan melakukan percobaan 3 model machine learning sekaligus untuk mengetahui performa model. Model yang dipilih adalah sebagai berikut:
    - Logistic regression
    - KNN
    - SVM

4. Evaluasi model

    Evaluasi model dilakukan dengan menggunakan metrik akurasi untuk mengetahui seberapa akurat hasil dari model yang telah dikembangkan. Metrik lain yang digunakan adalah precision, recall, dan juga f1_score.

5. Pengujian

    Pengujian dilakukan dengan menggunakan data testing yang sudah dibagi sebelumnya dan juga menggunakan data di luar dataset yang ada. Hal ini digunakan untuk memastikan bahwa model mampu mengklasifikasi karakter dengan baik.

## 4. Hasil Training

Hasil training dari percobaan pengembangan model OCR ini adalah sebagai berikut yang ditunjukkan bahwa model SVM mampu melakukan hasil akurasi tertinggi dengan skor $0.918182$.

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>accuracy</th>
      <th>precision</th>
      <th>recall</th>
      <th>f1_score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>KNN</th>
      <td>0.900000</td>
      <td>0.788613</td>
      <td>0.765831</td>
      <td>0.764247</td>
    </tr>
    <tr>
      <th>Logistic Regression</th>
      <td>0.909091</td>
      <td>0.720126</td>
      <td>0.738245</td>
      <td>0.716395</td>
    </tr>
    <tr>
      <th>SVM</th>
      <td>0.918182</td>
      <td>0.808886</td>
      <td>0.818182</td>
      <td>0.796402</td>
    </tr>
  </tbody>
</table>
</div>

## 5. Deployment

Pada tahap deployment, model machine learning yang digunakan hanya satu, yaitu *logistic regression*. Model ini dipilih karena mampu menghasilkan akurasi yang cukup tinggi dengan biaya komputasi yang relatif rendah. Hal tersebut menjadikannya ideal untuk kondisi sumber daya yang terbatas, terutama jika deployment ingin dilakukan secara *on-device*.

Deployment dari model OCR serta keseluruhan pipeline deteksi struk palsu dilakukan melalui dua pendekatan, yaitu deployment pada server dan *on-device*. Beberapa proses yang tergolong ringan tetap dijalankan secara *on-device* karena masih dapat diproses dengan baik pada perangkat mobile pengguna.

Sementara itu, proses yang melibatkan OCR serta beberapa tahapan *computer vision* lainnya dijalankan di sisi server menggunakan platform Firebase, dengan memanfaatkan layanan **Firebase Functions**. Implementasi deployment model pada sisi server ini dikembangkan pada repositori GitHub terpisah, yaitu:
https://github.com/Squadron-team/Rukunin-Firebase-Admin/tree/main/functions

## 6. Tantangan

Pada awalnya, deployment direncanakan untuk dijalankan sepenuhnya secara *on-device* dengan memanfaatkan tool lintas platform seperti ONNX. Namun, dalam implementasinya, pendekatan tersebut memiliki tingkat kompleksitas yang lebih tinggi dan membutuhkan waktu pengembangan yang lebih lama.

Oleh karena itu, pada tahap ini dipilih pendekatan yang lebih konvensional, yaitu melakukan deployment model ke server, agar proses pengembangan dan integrasi sistem dapat berjalan lebih efisien.


## 7. Kesimpulan

Proyek ini berhasil mengembangkan sistem deteksi struk palsu dari aplikasi mobile banking BNI dengan memanfaatkan OCR berbasis *machine learning* sebagai bagian dari pipeline utama, mulai dari pemrosesan gambar, ekstraksi teks, hingga verifikasi struktur struk. Meskipun model SVM menunjukkan akurasi tertinggi pada tahap eksperimen, model *logistic regression* dipilih untuk deployment karena lebih efisien secara komputasi dan sesuai untuk kebutuhan integrasi sistem. Deployment dilakukan dengan pendekatan hibrid, yaitu menjalankan proses ringan secara *on-device* dan proses OCR serta *computer vision* yang lebih kompleks di sisi server, sehingga sistem tetap efisien dan layak untuk dikembangkan lebih lanjut.
