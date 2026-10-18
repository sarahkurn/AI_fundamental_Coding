# Prediksi Risiko Pelanggaran SLA Tiket Insiden IT

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/sarahkurn/AI_fundamental_Coding/blob/main/AI_Fundamental_Coding_Challenge_Workbook_SARAH.ipynb)

Model *early warning* untuk memprediksi apakah tiket insiden IT akan **melanggar SLA**, hanya dengan informasi yang tersedia **saat tiket pertama kali dibuka**. Project ini menggunakan data nyata dari sistem ServiceNow sebuah perusahaan IT.

> Dikerjakan sebagai AI Fundamentals Coding Challenge pada Bootcamp Perempuan Inovasi GENerasi AI (Markoding × IBM SkillsBuild).

---

## Latar Belakang

Di operasional IT support, risiko pelanggaran SLA biasanya baru terlihat saat tiket sudah mendekati atau melewati batas waktu, sehingga sudah terlambat untuk ditangani. Project ini menguji apakah risiko tersebut bisa dideteksi lebih awal, sehingga tim service desk dapat memprioritaskan tiket berisiko sejak tiket masuk.

## Hasil Utama

| Metrik (data test) | Dummy (selalu tebak "aman") | Logistic Regression | **Random Forest (final)** |
|---|---:|---:|---:|
| Accuracy | 0,634 | 0,713 | **0,712** |
| Balanced Accuracy | 0,500 | 0,705 | **0,710** |
| Recall (SLA terlanggar) | 0,000 | 0,674 | **0,703** |
| Precision | 0,000 | 0,596 | **0,589** |
| ROC-AUC | 0,500 | 0,780 | **0,787** |

- Model final mendeteksi **70% tiket yang benar-benar melanggar SLA** (1.282 dari 1.823 tiket di data test).
- Accuracy 71% dibandingkan dengan baseline 63,4%, yaitu accuracy yang didapat hanya dengan selalu menebak "SLA terpenuhi".
- Tidak ada indikasi overfitting (selisih accuracy train–test hanya 0,021).

## Dataset

**Incident Management Process Enriched Event Log**: 141.712 event dari 24.918 insiden, berasal dari audit log ServiceNow. Nama tim, kategori, dan lokasi sudah dianonimkan.

> Amaral, C., Fantinato, M., & Peres, S. (2018). *Incident management process enriched event log* [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C57S4H

Lisensi: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Dataset tidak disertakan di repo ini; notebook mengunduhnya langsung dari UCI.

## Pendekatan

1. **Meringkas event log**: satu insiden tercatat rata-rata 5,7 kali (setiap update tiket). Data diringkas menjadi satu baris per insiden. Fitur diambil dari **event pertama** (kondisi saat tiket dibuka), sedangkan target diambil dari **event terakhir** (status SLA akhir).
2. **Mencegah data leakage**: kolom yang baru diketahui setelah tiket berjalan, seperti durasi penyelesaian, jumlah reassignment, dan waktu resolved, **tidak** dipakai sebagai fitur.
3. **Validasi target**: median durasi tiket yang melanggar SLA adalah 201,4 jam, dibandingkan 1,1 jam untuk tiket yang memenuhi SLA. Ini mengonfirmasi arti kolom `made_sla`.
4. **Pemodelan**: `Pipeline` scikit-learn dengan `SimpleImputer` + `OneHotEncoder` untuk 9 fitur kategori dan 2 fitur waktu. `class_weight='balanced'` dipakai karena data tidak seimbang (36,6% tiket melanggar SLA).
5. **Interpretasi**: permutation importance pada 11 fitur asli.
6. **Penjelasan dengan AI generatif**: hasil prediksi dijelaskan dalam bahasa awam menggunakan Gemini API, dengan prompt yang dibatasi pada fakta historis dari data.

## Insight

- **Assignment group adalah fitur terpenting.** Jika fitur ini diacak, balanced accuracy turun sekitar 8 poin.
- **Priority terlihat kuat saat eksplorasi data, tetapi hampir tidak berpengaruh di model.** Tiket High (97,1%) dan Critical (80,5%) memang jauh lebih sering melanggar SLA, tetapi 95% tiket berprioritas Moderate, sehingga priority hanya membedakan sebagian kecil tiket.
- **Tiket yang belum memiliki assignment group saat dibuka cenderung berisiko**, bahkan untuk priority Moderate.

## Keterbatasan dan Responsible AI

- Model hanya melihat kondisi saat tiket dibuka, sehingga tidak mengetahui kejadian setelahnya (reassignment, menunggu pengguna, eskalasi ke vendor). Sekitar 30% tiket yang melanggar SLA tetap tidak terdeteksi.
- Data berasal dari satu organisasi pada tahun 2016, sehingga perlu divalidasi ulang sebelum diterapkan di tempat lain.
- Model ditujukan sebagai **alat bantu prioritisasi**, bukan untuk menilai kinerja tim atau individu. Assignment group dengan tingkat pelanggaran tinggi belum tentu kinerjanya buruk; bisa jadi tim tersebut menangani masalah yang lebih sulit.
- Penjelasan dari Gemini diberi label sebagai output AI dan perlu ditinjau manusia sebelum digunakan.

## Cara Menjalankan

1. Klik tombol **Open in Colab** di atas.
2. Simpan Gemini API key di **Colab Secrets** (ikon 🔑) dengan nama `GEMINI_API_KEY`, lalu aktifkan *Notebook access*. API key bisa dibuat gratis di [Google AI Studio](https://aistudio.google.com/).
3. Pilih **Runtime → Run all**.

Semua proses acak menggunakan `RANDOM_STATE = 1082026`, sehingga hasil dapat direproduksi.

## Tools

Python · pandas · scikit-learn · matplotlib · seaborn · Google Gemini API (`google-genai`) · Google Colab

## Penulis

**Sarah Kurn**
GitHub: [@sarahkurn](https://github.com/sarahkurn)
