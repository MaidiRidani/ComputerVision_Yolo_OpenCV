<div align="center">

<img src="../../../assets/image/banner.png" weidht="full" ></div>

# Performance Metrics Deep Dive

Di materi ini pembahasannya beralih ke cara mengevaluasi hasil training model. Selama ini aku sering melihat angka seperti mAP, Precision, Recall, atau IoU saat selesai training YOLO, tetapi belum benar-benar memahami arti masing-masing. Setelah materi ini, aku jadi lebih paham bahwa setiap metrik sebenarnya mengukur aspek performa model yang berbeda. Jadi, model yang bagus bukan berarti hanya memiliki satu nilai tinggi, tetapi harus dilihat dari berbagai metrik secara bersamaan.

## Metrik Penting pada Object Detection

Sebelum membahas output YOLO, materi memperkenalkan beberapa metrik yang paling sering digunakan untuk mengevaluasi model Object Detection.

| Metrik | Fungsi |
|--------|---------|
| **IoU (Intersection over Union)** | Mengukur seberapa besar overlap antara bounding box prediksi dan ground truth. |
| **Precision** | Mengukur seberapa banyak prediksi yang benar dari seluruh objek yang diprediksi model. |
| **Recall** | Mengukur seberapa banyak objek yang berhasil ditemukan dari seluruh objek yang sebenarnya ada. |
| **F1 Score** | Nilai keseimbangan antara Precision dan Recall. |
| **AP (Average Precision)** | Luas area di bawah kurva Precision-Recall untuk satu class. |
| **mAP (Mean Average Precision)** | Rata-rata AP dari seluruh class sehingga menjadi metrik utama performa model. |

Dari semua metrik tersebut, mAP menjadi angka yang paling sering digunakan untuk membandingkan performa antar model. Walaupun begitu, ternyata mAP saja belum cukup karena masih perlu melihat Precision, Recall, dan IoU untuk mengetahui letak kelemahan model.

## Menggunakan Validation pada YOLO

Setelah model selesai di-training, YOLO menyediakan mode **Validation** melalui `model.val()`.

Proses ini akan menjalankan model pada validation dataset, kemudian menghasilkan berbagai metrik evaluasi beserta beberapa visualisasi yang membantu memahami performa model.

Jadi, validation bukan sekadar menghitung akurasi, tetapi juga memberikan gambaran lengkap mengenai kualitas model.

## Memahami Output Validation

YOLO menampilkan hasil evaluasi untuk setiap class yang ada di dataset.

| Kolom | Keterangan |
|-------|------------|
| **Class** | Nama class. |
| **Images** | Jumlah gambar yang mengandung class tersebut. |
| **Instances** | Jumlah total objek pada class tersebut. |
| **Precision (P)** | Ketepatan prediksi model. |
| **Recall (R)** | Kemampuan menemukan seluruh objek. |
| **mAP50** | mAP pada IoU 0.50. |
| **mAP50-95** | Rata-rata mAP dari IoU 0.50 sampai 0.95. Nilai ini lebih ketat dan lebih representatif. |

Aku baru tahu kalau **mAP50** biasanya lebih tinggi karena hanya menggunakan satu threshold IoU, sedangkan **mAP50-95** jauh lebih sulit dicapai sehingga sering dijadikan acuan utama dalam benchmark.

## Speed Metrics

Selain akurasi, YOLO juga menampilkan kecepatan proses inference.
Beberapa informasi yang biasanya ditampilkan antara lain preprocessing, inference, dan postprocessing time.
Bagian ini penting terutama jika model akan dijalankan secara real-time, misalnya pada CCTV, robot, atau autonomous vehicle. Model dengan akurasi tinggi belum tentu menjadi pilihan terbaik jika proses inferencenya terlalu lambat.

## Visual Output dari Validation
Selain angka-angka evaluasi, YOLO juga menghasilkan berbagai file visual yang membantu memahami performa model.

| File | Fungsi |
|------|--------|
| **BoxF1_curve.png** | Kurva perubahan F1 Score terhadap confidence threshold. |
| **BoxPR_curve.png** | Kurva Precision dan Recall. |
| **BoxP_curve.png** | Perubahan Precision terhadap threshold. |
| **BoxR_curve.png** | Perubahan Recall terhadap threshold. |
| **confusion_matrix.png** | Menampilkan distribusi prediksi benar dan salah untuk setiap class. |
| **confusion_matrix_normalized.png** | Versi normalisasi confusion matrix sehingga lebih mudah dibandingkan antar class. |
| **val_batch_labels.jpg** | Menampilkan ground truth pada validation dataset. |
| **val_batch_pred.jpg** | Menampilkan hasil prediksi model pada validation dataset. |

Aku baru sadar ternyata folder `runs/detect/val` yang selama ini sering muncul setelah validation memang berisi berbagai visualisasi tersebut. Selama ini aku lebih sering melihat angka mAP saja tanpa memperhatikan file-file lainnya.

## Memilih Metrik Sesuai Kebutuhan

Tidak semua proyek membutuhkan metrik yang sama.

| Kondisi | Metrik yang Paling Penting |
|---------|----------------------------|
| Evaluasi performa secara umum | mAP |
| Lokasi bounding box harus sangat akurat | IoU |
| Ingin meminimalkan false positive | Precision |
| Tidak boleh melewatkan objek | Recall |
| Membutuhkan keseimbangan Precision dan Recall | F1 Score |
| Sistem real-time | Speed, FPS, dan Latency |

Jadi pemilihan metrik sebenarnya bergantung pada kebutuhan aplikasi, bukan sekadar mengejar angka mAP setinggi mungkin.

## Cara Menginterpretasikan Hasil

Materi juga memberikan beberapa contoh bagaimana membaca hasil evaluasi.

| Kondisi | Kemungkinan Penyebab | Solusi |
|---------|----------------------|--------|
| mAP rendah | Model belum belajar dengan baik | Perbaiki dataset atau training. |
| IoU rendah | Bounding box kurang presisi | Perbaiki proses localization. |
| Precision rendah | Banyak false positive | Naikkan confidence threshold atau perbaiki dataset. |
| Recall rendah | Banyak objek terlewat | Tambah variasi data atau tingkatkan kemampuan model. |
| AP salah satu class rendah | Class tersebut sulit dipelajari | Tambahkan data pada class tersebut atau ubah class weight. |

Aku suka bagian ini karena lebih praktis. Jadi ketika melihat angka validation nanti, aku punya gambaran harus memperbaiki bagian mana, bukan sekadar mencoba training ulang berkali-kali.

## Studi Kasus

Materi memberikan beberapa contoh kondisi nyata saat mengevaluasi model.

| Kondisi | Interpretasi |
|---------|--------------|
| Recall tinggi tetapi Precision rendah | Model terlalu banyak mendeteksi objek yang sebenarnya tidak ada. |
| mAP cukup baik tetapi IoU rendah | Model berhasil menemukan objek, tetapi posisi bounding box masih kurang tepat. |
| Salah satu class memiliki AP jauh lebih rendah | Class tersebut kemungkinan membutuhkan lebih banyak data atau perlu penyesuaian saat training. |

Menurutku contoh seperti ini jauh lebih mudah dipahami dibanding hanya menghafalkan definisi setiap metrik.

## Catatan Pribadi

Selama ini aku sering melihat angka mAP setelah selesai training dan langsung menganggap model sudah bagus atau belum. Setelah mempelajari materi ini, ternyata proses evaluasi jauh lebih luas dari itu.

Sekarang aku mulai memahami bahwa setiap metrik memiliki fungsi yang berbeda. Precision berbicara tentang seberapa sering model benar ketika mendeteksi objek, Recall menunjukkan seberapa banyak objek yang berhasil ditemukan, IoU mengukur ketepatan posisi bounding box, sedangkan mAP menjadi ringkasan performa model secara keseluruhan.

Aku juga baru mengetahui bahwa nilai mAP yang ditampilkan selama proses training ternyata dihitung menggunakan **validation dataset**, bukan training dataset. Hal ini masuk akal karena tujuan validation adalah mengukur kemampuan model pada data yang belum digunakan untuk belajar sehingga hasil evaluasinya lebih objektif.

Hal lain yang menurutku cukup menarik adalah kualitas annotation ternyata ikut memengaruhi hasil evaluasi. Jika ada objek yang sebenarnya ada di gambar tetapi tidak dianotasi, lalu model berhasil mendeteksinya, prediksi tersebut justru akan dianggap sebagai **False Positive**. Artinya, kesalahan pada annotation dapat menurunkan nilai Precision, F1 Score, bahkan mAP, meskipun model sebenarnya mendeteksi objek dengan benar.

Kalau diringkas, saat mengevaluasi model sebaiknya tidak hanya melihat satu angka saja. Kombinasi antara mAP, Precision, Recall, IoU, F1 Score, serta visualisasi hasil validation akan memberikan gambaran yang jauh lebih lengkap mengenai performa model.

