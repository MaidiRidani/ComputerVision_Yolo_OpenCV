# Day 1 - Object Detection Fundamentals: NMS, IoU, Precision, Recall, AP, mAP, and COCO Evaluation

# Learning Objectives

Pada pembelajaran hari pertama ini dipelajari dasar-dasar evaluasi model Object Detection yang digunakan oleh model seperti **YOLO, SSD, Faster R-CNN**, dan model deteksi objek modern lainnya.

Topik yang dipelajari meliputi:

- Bounding Box
- Non-Maximum Suppression (NMS)
- Intersection over Union (IoU)
- Confusion Matrix pada Object Detection
- Precision
- Recall
- Precision-Recall Curve
- Average Precision (AP)
- Mean Average Precision (mAP)
- Perbedaan evaluasi PASCAL VOC dan MS COCO

---

# Bounding Box

<div style="text-align:center">
    <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQuqeYWCQK6cxi5cKTFcLmUaJFH290qlgKxd8l_ADEZOA&s=10" width=400>
</div>
Bounding Box merupakan kotak yang digunakan untuk menunjukkan lokasi suatu objek pada gambar.

Sebuah bounding box umumnya direpresentasikan menggunakan koordinat:

```text
(xmin, ymin)
(xmax, ymax)
```

atau

```text
(center_x, center_y, width, height)
```

Bounding Box terdiri dari dua jenis:

- **Ground Truth Bounding Box**
  - Dibuat secara manual oleh manusia (labeling).
  - Digunakan sebagai acuan evaluasi.

- **Predicted Bounding Box**
  - Dihasilkan oleh model.
  - Memiliki nilai confidence score.

Contoh:

```text
Ground Truth
┌─────────────┐
│    Mobil    │
└─────────────┘

Prediction
┌──────────────┐
│    Mobil     │
└──────────────┘
Confidence = 0.93
```

---

# Confidence Score

Confidence Score menunjukkan tingkat keyakinan model terhadap hasil prediksi.

Nilainya berada pada rentang:

```text
0 ≤ Confidence ≤ 1
```

Semakin mendekati 1 maka model semakin yakin bahwa objek tersebut benar.

Contoh:

| Confidence | Arti |
|------------|------|
|0.98|Sangat yakin|
|0.85|Yakin|
|0.52|Kurang yakin|
|0.20|Kemungkinan salah|

Confidence digunakan sebagai dasar untuk:

- Filtering prediction
- Non-Maximum Suppression
- Perhitungan Precision-Recall Curve

---

# Mengapa YOLO Menghasilkan Banyak Bounding Box?

Satu objek dapat diprediksi oleh beberapa lokasi (grid/feature map) secara bersamaan.

Misalnya terdapat satu mobil.

YOLO dapat menghasilkan:

| Bounding Box | Confidence |
|--------------|-----------:|
|A|0.96|
|B|0.91|
|C|0.88|
|D|0.75|

Padahal semuanya mengarah pada objek yang sama.

Hal ini merupakan perilaku normal pada Object Detection.

Masalah inilah yang diselesaikan oleh **Non-Maximum Suppression (NMS).**

---

# Non-Maximum Suppression (NMS)

## Tujuan NMS

NMS digunakan untuk menghilangkan bounding box yang saling bertumpuk sehingga hanya tersisa satu bounding box terbaik untuk setiap objek.

Tanpa NMS:

```text
Mobil

□□□□□
 □□□□□
  □□□□□
```

Setelah NMS:

```text
Mobil

□□□□□
```

---

## Langkah-langkah NMS

### 1. Model menghasilkan banyak bounding box

Misalnya:

| Box | Confidence |
|------|-----------:|
|A|0.95|
|B|0.90|
|C|0.84|

---

### 2. Urutkan berdasarkan confidence

```text
0.95
0.90
0.84
```

---

### 3. Pilih confidence tertinggi

Misalnya:

```text
A = 0.95
```

---

### 4. Hitung IoU terhadap semua box lain

Jika

```text
IoU > Threshold
```

maka dianggap mendeteksi objek yang sama.

Bounding box dengan confidence lebih rendah dihapus.

---

### 5. Ulangi

Proses diulang hingga semua bounding box diperiksa.

---

# Intersection over Union (IoU)

IoU digunakan untuk mengukur tingkat tumpang tindih antara dua bounding box.

Rumus:

\[
IoU=\frac{Intersection}{Union}
\]

dengan

- Intersection = area yang saling bertumpuk
- Union = total area kedua box

Rumus union:

\[
Union = Area_A + Area_B - Intersection
\]

Sehingga:

\[
IoU=
\frac{Intersection}
{Area_A+Area_B-Intersection}
\]

---

## Rentang Nilai IoU

| IoU | Arti |
|------|------|
|0|Tidak bertumpuk|
|0.25|Sedikit overlap|
|0.50|Overlap sedang|
|0.75|Overlap tinggi|
|1|Bounding box identik|

Semakin besar IoU berarti prediksi semakin mendekati Ground Truth.

---

## Contoh Perhitungan IoU

Area A

```text
10000
```

Area B

```text
10000
```

Intersection

```text
8000
```

Union

\[
10000+10000-8000
=
12000
\]

Maka

\[
IoU=
\frac{8000}{12000}
=
0.667
\]

---

# IoU Threshold

IoU Threshold menentukan apakah prediksi dianggap benar atau salah.

Contoh:

Threshold:

```text
0.5
```

Jika

```text
IoU = 0.73
```

Maka

```text
True Positive
```

Jika

```text
IoU = 0.28
```

Maka

```text
False Positive
```

---

# Pengaruh Threshold pada NMS

## Threshold terlalu kecil

Misalnya:

```text
0.2
```

Akibatnya:

- Banyak bounding box dibuang
- NMS menjadi agresif
- Berpotensi menghapus objek yang sebenarnya berbeda

---

## Threshold terlalu besar

Misalnya:

```text
0.9
```

Akibatnya:

- Banyak bounding box dipertahankan
- Satu objek dapat memiliki beberapa bounding box

---

# Confusion Matrix pada Object Detection

Empat kemungkinan hasil prediksi:

| | Ground Truth Positif | Ground Truth Negatif |
|---|---|---|
|Prediksi Positif|True Positive (TP)|False Positive (FP)|
|Prediksi Negatif|False Negative (FN)|True Negative (TN)|

---

## True Positive (TP)

Model berhasil mendeteksi objek dengan benar.

---

## False Positive (FP)

Model mendeteksi objek yang sebenarnya tidak ada.

Disebut juga **False Alarm**.

---

## False Negative (FN)

Model gagal mendeteksi objek yang sebenarnya ada.

---

## True Negative (TN)

Model mengatakan objek tidak ada dan memang benar objek tidak ada.

Pada Object Detection, TN hampir tidak pernah digunakan karena jumlah kemungkinan background sangat besar dan tidak terdefinisi dengan jelas.

---

# Precision

Precision mengukur ketepatan prediksi positif model.

Rumus:

\[
Precision=
\frac{TP}
{TP+FP}
\]

Artinya:

> Dari seluruh prediksi positif yang dibuat model, berapa banyak yang benar?

Contoh:

TP = 90

FP = 10

\[
Precision=
\frac{90}{90+10}
=
0.9
\]

---

# Recall

Recall mengukur kemampuan model menemukan seluruh objek yang ada.

Rumus:

\[
Recall=
\frac{TP}
{TP+FN}
\]

Artinya:

> Dari seluruh objek yang benar-benar ada, berapa banyak yang berhasil ditemukan model.

Contoh:

TP = 90

FN = 10

\[
Recall=
\frac{90}{100}
=
0.9
\]

---

# Perbedaan Precision dan Recall

| Precision | Recall |
|-----------|--------|
|Fokus pada kualitas prediksi|Fokus pada jumlah objek yang berhasil ditemukan|
|Menggunakan FP|Menggunakan FN|

---

# Precision-Recall Curve

Precision dan Recall dihitung untuk setiap confidence threshold.

Setiap hasil menjadi satu titik pada grafik.

Sumbu:

```text
Y = Precision

X = Recall
```

Grafik ini disebut **Precision-Recall Curve (PR Curve).**

---

# Average Precision (AP)

Average Precision (AP) merupakan luas area di bawah Precision-Recall Curve.

**AP bukan rata-rata dari seluruh nilai Precision.**

Pada PASCAL VOC digunakan metode:

```text
11 Point Interpolation
```

Recall yang digunakan:

```text
0.0

0.1

0.2

...

1.0
```

Sebanyak:

```text
11 titik
```

---

## Interpolated Precision

Untuk setiap recall:

\[
P_{interp}(r)
=
\max_{r' \ge r}
P(r')
\]

Artinya:

Pada setiap nilai recall, diambil **precision maksimum yang berada di sebelah kanan (recall yang sama atau lebih besar).**

Tujuan interpolasi:

- Membuat kurva Precision-Recall menjadi monoton menurun.
- Menghasilkan estimasi area yang lebih stabil.

---

## Rumus AP pada PASCAL VOC

\[
AP=
\frac1{11}
\sum_{r=0}^{1}
P_{interp}(r)
\]

---

# Mean Average Precision (mAP)

mAP merupakan rata-rata Average Precision dari seluruh kelas.

Misalnya:

|Class|AP|
|-----|--|
|Dog|0.35|
|Person|0.55|
|Truck|1.00|
|Sheep|0.00|
|Teddy|0.50|

Maka

\[
mAP=
\frac{
0.35+0.55+1+0+0.50
}{5}
\]

Semakin besar mAP maka performa model semakin baik.

---

# PASCAL VOC vs MS COCO

## PASCAL VOC

Menggunakan:

- IoU = 0.5
- 11 Point Interpolation

Metrik:

```text
mAP@0.50
```

Evaluasi relatif lebih mudah karena hanya menggunakan satu threshold IoU.

---

## MS COCO

MS COCO memperkenalkan:

### 101 Point Interpolation

Recall dihitung pada:

```text
0.00

0.01

0.02

...

1.00LL
```

Sebanyak:

```text
101 titik
```

Sehingga estimasi area di bawah kurva menjadi lebih akurat.

---

### Multiple IoU Threshold

COCO tidak hanya menggunakan IoU = 0.5.

Threshold yang digunakan:

```text
0.50

0.55

0.60

0.65

0.70

0.75

0.80

0.85

0.90

0.95
```

Sebanyak:

```text
10 threshold IoU
```

Setiap threshold menghasilkan AP tersendiri.

Kemudian seluruh AP dirata-ratakan.

Metrik ini disebut:

```text
mAP@0.50:0.95
```

---

# Perbedaan mAP@0.50 dan mAP@0.50:0.95

| mAP@0.50 | mAP@0.50:0.95 |
|-----------|---------------|
|IoU tetap 0.50|IoU dari 0.50 hingga 0.95|
|Evaluasi lebih mudah|Evaluasi jauh lebih ketat|
|Mirip evaluasi PASCAL VOC|Standar MS COCO|

Karena menggunakan banyak threshold IoU, nilai:

```text
mAP@0.50:0.95
```

hampir selalu lebih rendah daripada

```text
mAP@0.50
```

---

# Pipeline Evaluasi Object Detection

Seluruh proses evaluasi model Object Detection dapat diringkas sebagai berikut:

```text
Input Image
      │
      ▼
YOLO menghasilkan banyak bounding box
      │
      ▼
Confidence Filtering
      │
      ▼
Non-Maximum Suppression (NMS)
      │
      ▼
Bounding Box Final
      │
      ▼
Bandingkan dengan Ground Truth
      │
      ▼
Hitung IoU
      │
      ▼
Tentukan TP, FP, dan FN
      │
      ▼
Hitung Precision dan Recall
      │
      ▼
Bangun Precision-Recall Curve
      │
      ▼
Hitung Average Precision (AP)
      │
      ▼
Hitung mAP (Average seluruh AP)
      │
      ▼
Evaluasi dilakukan pada IoU 0.50–0.95
```

# Kesimpulan

Pada hari pertama dipelajari dasar evaluasi Object Detection yang menjadi fondasi hampir seluruh model modern seperti YOLO, SSD, Faster R-CNN, RetinaNet, DETR, dan RT-DETR.

Konsep yang paling penting untuk dipahami adalah:

- Bounding Box
- Confidence Score
- Non-Maximum Suppression (NMS)
- Intersection over Union (IoU)
- Confusion Matrix
- Precision
- Recall
- Precision-Recall Curve
- Average Precision (AP)
- Mean Average Precision (mAP)
- Evaluasi PASCAL VOC
- Evaluasi MS COCO

Seluruh metrik tersebut saling berhubungan dan membentuk pipeline evaluasi standar pada hampir semua penelitian Object Detection modern.