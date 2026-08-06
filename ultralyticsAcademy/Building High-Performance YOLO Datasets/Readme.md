<div align="center">

<img src="../../assets/image/banner.png" weidht="full" > </div>



# Building High-Performance YOLO Datasets Course
<div align="center"><img src="ban.avif" width="400"></div>

## Lesson 1 : Start With the Business Objective

### Hands-On

#### Task

Write your project's four-line spec on a single page: business goal, vision task, class list, success metric. Share it with the stakeholder who's paying for the model. If they push back on any line, that's the line to fix before doing anything else.

### Solution

| Item | Description |
|------|-------------|
| **Business Goal** | Membuat sistem computer vision berbasis YOLO yang dapat mengklasifikasikan kualitas batuan nikel berdasarkan karakteristik visual yang sudah diberi label oleh geologist, sehingga proses sorting menjadi lebih cepat dan konsisten. |
| **Vision Task** | Image Classification. Karena tujuan model ini adalah menentukan kualitas dari satu batu pada setiap gambar berdasarkan kelas yang sudah ditentukan. |
| **Class List** | - High-grade Ore<br>- Medium-grade Ore<br>- Low-grade Ore<br>- Waste Rock<br>Kelas-kelas tersebut tidak saling tumpang tindih, sehingga satu batu hanya bisa masuk ke satu kategori. |
| **Success Metric** | Model diharapkan mampu mencapai Precision ≥ 95% dan Recall ≥ 90% pada holdout test dataset.<br>Target precision dibuat lebih tinggi agar kemungkinan batu berkualitas renda terklasifikasi sebagai ore dapat ditekan. Sementara itu, recall tetap dibuat tinggi agar batu yang memang berkualitas tidak banyak terlewat selama proses sorting. |


## Lesson 2 : Define the Dataset Specification
### Hands-On

#### Task

Draft the dataset spec for your project as a table: 5–10 scenario rows, with quotas. Show it to someone who knows the deployment site (a foreman, a shift lead, a customer). They'll add 2–3 scenarios you missed that's the value.

### Solution

#### Dataset Specification

| No. | Scenario | Main Variations | Quota | Target Images |
|:--:|-----------|-----------------|:-----:|--------------:|
| 1 | Batuan di conveyor saat cuaca cerah (siang) | Cuaca, pencahayaan | 20% | 1.000 |
| 2 | Batuan di conveyor pada pagi atau sore hari | Waktu, pencahayaan | 10% | 500 |
| 3 | Batuan di conveyor saat kondisi mendung atau hujan ringan | Cuaca | 15% | 750 |
| 4 | Batuan dengan berbagai ukuran (kecil, sedang, besar) | Ukuran objek | 15% | 750 |
| 5 | Batuan saling berhimpitan atau sebagian tertutup batu lain | Occlusion | 15% | 750 |
| 6 | Kamera sedikit berdebu atau terdapat noise ringan | Kondisi kamera | 13% | 650 |
| 7 | Background images (conveyor kosong tanpa objek target) | Background | 2% | 100 |
| 8 | Edge case: batu tertutup lumpur, basah, atau warna antar kelas hampir mirip | Edge Case | 10% | 500 |
| **Total** |  |  | **100%** | **5.000 gambar** |

#### Target Dataset

| Item | Description |
|------|-------------|
| Total Images | ± 5.000 gambar |
| Number of Classes | 4 kelas: High-grade Ore, Medium-grade Ore, Low-grade Ore, dan Waste Rock |
| Average Objects per Image | Diasumsikan sekitar 2–3 batu per gambar |
| Estimated Labeled Instances | Kurang lebih 10.000–15.000 instances |
| Minimum Target | >10.000 labeled instances |
| Label Source | Semua batu nantinya diberi label berdasarkan hasil identifikasi geologist, bukan hasil prediksi model. |

#### Data Collection Strategy

Untuk tahap awal, rencananya dataset akan dikumpulkan sekitar 5.000 gambar dulu sebagai dataset dasar buat training model pertama. Dengan asumsi setiap gambar berisi sekitar 2–3 batu, kemungkinan total labeled instances yang didapat ada di kisaran 10.000–15.000.
Kalau model awalnya sudah jadi, proses ngumpulin data bakal dilanjutkan pakai Active Learning. Jadi model dicoba di data baru, terus gambar yang confidence-nya rendah, salah prediksi, atau kondisi yang masih jarang muncul di dataset bakal dipilih buat dilabeli lagi sama geologist.
Harapannya cara ini bisa bikin proses labeling lebih efisien karena yang ditambah bukan asal banyak gambar, tapi gambar yang memang masih dibutuhkan model. Jadi dataset bisa berkembang sedikit demi sedikit sambil memperbaiki performa model di kondisi lapangan.





## Lesson 3 : Collect High-Quality, Representative Data

### Hands-On

#### Task

Audit your last 200 collected images. For each, check: do you know the camera, the time, and the location? If not, your collection metadata pipeline needs fixing before scaling.

### Solution

#### Dataset Baseline

Untuk simulasi proyek ini, saya mengasumsikan dataset baseline tidak langsung diambil dari area produksi. Rencananya dimulai dengan mengumpulkan ratusan batuan yang kelas kualitasnya sudah ditentukan lebih dulu oleh geologist.
Setiap batu kemudian diberi Sample ID sebagai identitas unik. Jadi sebelum difoto sebenarnya kelas setiap batu sudah diketahui, sehingga nantinya lebih mudah ditelusuri lagi saat proses labeling.
Batu-batu tersebut kemudian disusun pada area simulasi proses sorting. Di area ini dipasang 3 kamera CCTV dari sudut yang berbeda supaya setiap batu bisa terekam dari beberapa perspektif.
Proses pengambilan gambar dilakukan dengan berbagai skenario yang sudah direncanakan pada Lesson 2, misalnya pencahayaan yang berbeda, ukuran batu yang bervariasi, batu saling berhimpitan, occlusion, kamera sedikit berdebu, batu basah, batu tertutup lumpur, dan beberapa kondisi lain yang dibuat semirip mungkin dengan kondisi sebenarnya.
Karena setiap batu sudah memiliki Sample ID dan kelas kualitas sebelum difoto, hasil akhirnya tinggal dibuat bounding box untuk setiap batu, lalu dataset tersebut digunakan untuk melatih model YOLO pertama sebagai model baseline.


#### Active Learning

Kalau model baseline sudah selesai dilatih, idenya model mulai dicoba di area produksi.
Di tahap ini tujuan pengumpulan data sudah sedikit berbeda. Bukan sekadar nambah jumlah gambar, tapi lebih mencari kondisi yang masih susah dikenali model.
Misalnya ketika model gagal mendeteksi batu, salah menentukan kelas, confidence-nya rendah, atau muncul kondisi baru yang sebelumnya belum banyak ada di dataset.
Kalau menemukan kasus seperti itu, sampel tersebut rencananya dipisahkan dulu. Batu kemudian kembali diidentifikasi oleh geologist, diberi Sample ID, difoto lagi dari beberapa sudut, lalu dianotasi sebelum ditambahkan ke dataset berikutnya.
Harapannya cara seperti ini bisa membuat proses penambahan data lebih efisien karena yang ditambahkan memang contoh-contoh yang masih sulit dipelajari model.

---

#### Sample Metadata

Setiap gambar yang diambil juga disimpan bersama metadata sederhana untuk memudahkan proses audit dataset.
Contohnya seperti berikut.

```json
{
    "filename": "20260906_01_001.jpg",
    "sample_id": "HG-015",
    "camera": "CAM_01",
    "location": "Sorting Area A",
    "timestamp": "2026-09-06T08:20:00"
}
```

#### Metadata Description

| Metadata | Keterangan |
|----------|------------|
| filename | Nama file gambar. |
| sample_id | ID batu yang sudah ditentukan kelasnya oleh geologist. |
| camera | Kamera yang mengambil gambar. |
| location | Lokasi pemasangan kamera. |
| timestamp | Waktu saat gambar diambil. |

Menurut saya metadata tersebut sudah cukup untuk memenuhi kebutuhan audit dasar seperti yang dijelaskan pada lesson ini. Saya juga menambahkan Sample ID supaya hubungan antara gambar dan batu yang difoto tetap bisa ditelusuri.


#### Metadata Audit

Sebagai simulasi, dilakukan audit terhadap 200 gambar terakhir yang terdapat pada dataset.

| Metadata | Audit Result |
|----------|--------------|
| Camera | 200 / 200 tersedia |
| Timestamp | 200 / 200 tersedia |
| Location | 200 / 200 tersedia |
| Sample ID | 200 / 200 tersedia |

#### Conclusion

Dari hasil simulasi audit, seluruh gambar sudah memiliki informasi Camera, Timestamp, Location, dan Sample ID. Jadi kalau nantinya dataset diperbesar, saya rasa proses pelacakan asal setiap gambar masih bisa dilakukan dengan cukup mudah.


#### Prediction Log (Deployment)

Kalau model nantinya mulai dipakai di area produksi, saya membayangkan sistem juga menyimpan prediction log yang terpisah dari metadata dataset.

Contohnya seperti berikut.

```json
{
    "filename": "20260915_03_012.jpg",
    "camera": "CAM_03",
    "location": "Sorting Area C",
    "timestamp": "2026-09-15T10:40:00",
    "prediction": "Medium-grade Ore",
    "confidence": 0.61,
    "ground_truth": "High-grade Ore",
    "correct": false,
    "weather": "Rainy"
}
```

Menurut saya file ini akan lebih berguna untuk melihat performa model setelah deployment, misalnya mencari kondisi apa saja yang sering membuat model salah prediksi atau kamera mana yang paling sering menghasilkan error.

Kalau ternyata banyak kesalahan muncul pada kondisi tertentu, misalnya saat hujan atau dari kamera tertentu, data tersebut bisa diprioritaskan untuk ditambahkan ke dataset pada proses Active Learning berikutnya.


Menurut saya akan lebih rapi kalau metadata dipisahkan menjadi dua file.
1. `sample_metadata.json` digunakan selama proses pembangunan dataset dan audit metadata.
2. `prediction_log.json` digunakan setelah model di-deploy untuk evaluasi performa dan membantu proses Active Learning.

Pemisahan seperti ini menurut saya membuat alur proyek lebih jelas karena metadata untuk training dan metadata untuk evaluasi memiliki tujuan yang berbeda.


