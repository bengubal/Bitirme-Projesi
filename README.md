# Rotterdam CT Score — Otomatik Hesaplama Sistemi

Kafa travması (TBI) hastalarında kranyal BT görüntülerinden **Rotterdam CT Score**'u otomatik olarak hesaplayan end-to-end bir deep learning + kural tabanlı hibrit pipeline.

---

## 📁 Proje Yapısı

| Dosya | Açıklama |
|---|---|
| `unet_training.ipynb` | U-Net modelinin eğitimi (ICH segmentasyonu) |
| `rotterdam-ct-score-calculator-demo.ipynb` | DICOM dosyasından Rotterdam skoru hesaplayan demo (Gradio arayüzlü) |
| `tbidetection-results.ipynb` | CQ500 dataset üzerinde pipeline değerlendirmesi |

---

## 🧠 Pipeline Bileşenleri

| Bileşen | Yöntem | Kaynak |
|---|---|---|
| Slice Seçimi | SSA — Dışbükeylik tabanlı geometrik skor | Qi et al. 2013 |
| İdeal Orta Hat | Kafatası simetri araması | Qi et al. 2013 |
| Deforme Orta Hat | Ventrikül / CSF tespiti + U-Net entegrasyonu | Qi et al. 2013 |
| MLS Ölçümü | IML–DML geometrik mesafe | — |
| ICH Tespiti | U-Net (ResNet34 encoder) | — |
| Basal Sistern | Compression Ratio (K-means) | Toledo et al. 2021 |
| Rotterdam Skoru | Maas 2005 ölçeği (2–6 arası) | Maas et al. 2005 |

---

## 🔬 1 — U-Net Eğitimi (`unet_training.ipynb`)

### Veri Seti
- **Kaynak:** [Computed Tomography Images for Intracranial Hemorrhage Detection and Segmentation v1.3.1](https://physionet.org/content/ct-ich/1.3.1/)
- **Format:** NIfTI (`.nii`) — hasta bazlı CT hacmi + maske
- **Bölme:** %80 train / %20 validasyon (hasta bazlı, slice bazlı değil)
- **Veri setinin Google Drive üzerinden bağlantısı:** https://drive.google.com/drive/folders/1u6fq9wTmPQxa4s39LMMmX9FJ4BHBEDQW?usp=sharing

### Model
- **Mimari:** U-Net — `segmentation-models-pytorch`
- **Encoder:** ResNet34 (ImageNet ağırlıkları)
- **Girdi:** 3 kanallı windowing (Brain 40/80 HU, Bone 500/3000 HU, Subdural 175/50 HU)
- **Kayıp:** Dice + weighted BCE (pos_weight=8.0)
- **Optimizer:** Adam (lr=1e-4) + ReduceLROnPlateau

### Eğitim Detayları
- WeightedRandomSampler ile kanamalı slice oversample
- Yatay/dikey flip augmentation
- 30 epoch, en iyi Val Dice'a göre model kaydedilir

---

## 🖥️ 2 — Demo (`rotterdam-ct-score-calculator-demo.ipynb`)

Gradio tabanlı interaktif arayüz. Tek bir DICOM dosyası yükleyerek tam pipeline çalıştırılır.

**Çalıştırma:**
1. Tüm hücreleri sırayla çalıştır
2. Son hücrede oluşan Gradio URL'sine tarayıcıdan git
3. `.dcm` dosyası yükle → Rotterdam skoru ve bileşen detayları görüntülenir

**Gereksinimler:**
```
pydicom, scikit-image, scipy, segmentation-models-pytorch
pylibjpeg, pylibjpeg-libjpeg, pylibjpeg-openjpeg, gdcm, gradio
```

---

## 📊 3 — Değerlendirme (`tbidetection-results.ipynb`)

CQ500 (Qure.ai Head CT) dataset üzerinde pipeline doğrulaması.
-**Veri setinin Kaggle üzerinden bağlantısı :** https://www.kaggle.com/datasets/crawford/qureai-headct 

**Değerlendirilen Bileşenler:**

| Bileşen | Ground Truth |
|---|---|
| MLS Tespiti (≥5 mm → pozitif) | Radyolog konsensüsü |
| ICH Tespiti (U-Net piksel eşiği) | Radyolog konsensüsü |
| Basal Sistern Anomalisi | MassEffect proxy *(CQ500'de doğrudan etiket yok)* |

**Çıktılar:**
- Hasta bazlı tahmin tablosu (CSV)
- Seçili slice'lar (DICOM kopyaları)
- Rotterdam skor dağılımı

---

## ⚙️ Ortam

| Platform | Notlar |
|---|---|
| Google Colab | `unet_training.ipynb` ve demo için (Drive mount gerekli) |
| Kaggle Notebooks | `tbidetection-results.ipynb` için (CQ500 dataset Kaggle'da mevcut) |
| GPU | CUDA önerilir (T4 / A100) |

---

## 📚 Kaynaklar

- Qi X. et al. (2013). *Automatic Midline Shift Detection in Brain CT Images.*
- Toledo E. et al. (2021). *Compression Ratio for Basal Cistern Assessment.*
- Maas A.I.R. et al. (2005). *Prognostic Value of the Rotterdam CT Score.* — J Neurotrauma.
- CQ500 Dataset — [Qure.ai](http://headctstudy.qure.ai/)
