<div align="center">

<img src="../../assets/image/banner.png" weidht="full" > </div>

# Building High-Performance YOLO Datasets Course

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



