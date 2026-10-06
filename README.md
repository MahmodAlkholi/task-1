# task-1
task 1 chest_cancer_detection_report

# Chest Cancer Detection Using Teachable Machine

*Short Project Report — Problem, Results & Path to Higher Accuracy*

> Tool: Google Teachable Machine (Image) · Source dataset: Kaggle chest CT-scan dataset

---

## 1. Project Overview

This project investigates the use of artificial intelligence to support the early detection and classification of chest (lung) cancer from CT-scan images. The work is built on top of a public Kaggle dataset containing three non-small cell lung cancer subtypes — **Adenocarcinoma**, **Large cell carcinoma**, and **Squamous cell carcinoma** — plus a class of **normal** CT scans. The training was carried out inside Google Teachable Machine, a no-code image-classification tool that wraps a MobileNet-based transfer-learning model.

---

## 2. Problem We Are Trying to Solve

Lung cancer is one of the deadliest cancers worldwide, partly because early-stage tumors are easy to miss on standard CT review. Manual diagnosis is slow, depends on radiologist experience, and is not equally accessible everywhere. The goal of this project is to:

- Automate the first-pass classification of a chest CT scan as cancerous or normal.
- Differentiate between the most common lung cancer subtypes so the patient can be routed to the right treatment pathway as early as possible.
- Provide a fast, low-cost, and accessible triage tool that can later support clinicians instead of replacing them.

In short: **build an AI model that looks at a CT image and tells the doctor — quickly and accurately — whether cancer is present, and if so, what type.**

---

## 3. Current Prediction Results

Two test images were run through the trained Teachable Machine model:

| Test Image | Expected Class | Model Prediction | Confidence |
|---|---|---|---|
| Image 1 (normal lung) | Normal | Normal CT scan | 95% |
| Image 2 (adenocarcinoma) | Adenocarcinoma | Adenocarcinoma CT scan | 100% |
| **Overall impression** | — | Both predictions match the ground-truth class with very high confidence | Strong |

*On these two samples the model behaves correctly, but two correct predictions on two images is not enough to claim the model is reliable. A proper evaluation on the held-out test set is needed before the system can be trusted.*

---

## 4. Dataset & Limitations Observed

### 4.1 Dataset snapshot

The Kaggle source dataset contains four classes:

- **Adenocarcinoma** — ~30% of all lung cancers; found in the outer lung region.
- **Large cell carcinoma** — ~10–15% of NSCLC; grows and spreads quickly.
- **Squamous cell carcinoma** — ~30% of NSCLC; usually central, linked to smoking.
- **Normal** — healthy CT scans used as the control class.

It is split into **train (70%)**, **test (20%)** and **validation (10%)**.

### 4.2 What was actually trained in Teachable Machine

From the training screenshots, only **two** classes were created:

- `normal ct scan` — 72 image samples
- `adenocarcinoma ct scan` — 72 image samples

Two of the four classes from the source dataset — **Large cell carcinoma** and **Squamous cell carcinoma** — are missing. That means the current model can only answer one question: *"Is this scan normal or adenocarcinoma?"* It cannot distinguish the other two cancer types, which is a serious gap for a real diagnostic workflow.

### 4.3 Other limitations

- **Small sample size**: 72 images per class is far below what is normally needed for deep-learning–grade medical imaging.
- **Teachable Machine uses a fixed MobileNet backbone** and a closed training loop — no ability to tune the architecture, loss function, learning rate, or class weights.
- **No visible preprocessing**: CT scans benefit from Hounsfield-unit windowing, lung-window clipping, and grayscale normalization, none of which Teachable Machine exposes.
- **No data augmentation**: rotation, flipping, zoom and brightness changes are not controllable from inside the tool.
- **The train/validation/test split is done internally** by Teachable Machine and cannot be aligned with the original 70/20/10 split from the Kaggle dataset.
- **Each image is a single 2D slice** — the 3D context of a full CT volume is lost.
- **No reporting of precision, recall, F1 or confusion matrix** was produced; *"95% confidence"* on a single image is not the same as 95% accuracy on a test set.

---

## 5. How to Improve Accuracy

The improvements below go from easy wins to bigger engineering work. Most of them require moving past Teachable Machine into a proper deep-learning framework such as **TensorFlow/Keras** or **PyTorch**.

### 5.1 Fix the data first

- **Re-include all four classes.** Without Large cell and Squamous cell carcinoma the model is not a real lung-cancer classifier.
- **Increase the dataset.** Aim for at least a few hundred images per class, ideally 1,000+. Augment aggressively (rotation, flip, zoom, brightness, elastic deformation) to multiply the effective size.
- **Balance the classes.** If one class has fewer images, use class weights or oversampling instead of letting the model favor the majority class.
- **Apply CT-specific preprocessing.** Rescale to Hounsfield units, clip to a lung window (e.g. –1000 to +400 HU), convert to single-channel grayscale, resize to a fixed shape (224×224 or 256×256).
- **Keep the original 70/20/10 split** from Kaggle so that train, validation and test come from the same patient population and never overlap.

### 5.2 Upgrade the model

- Move from Teachable Machine to a proper CNN. Start with a small **VGG-16** or **ResNet-50**, then try **EfficientNet-B0/B3** which usually wins on medical imaging with limited data.
- Use **transfer learning**: start from ImageNet weights and fine-tune the last few blocks. Freeze the early layers to avoid overfitting on small datasets.
- Add **dropout (0.3–0.5)** and **L2 regularization** to reduce overfitting.
- Use **learning-rate scheduling** (cosine decay or `ReduceLROnPlateau`) and **early stopping** based on validation loss.
- If multiple slices per patient are available, stack them as 3-channel input or use a small **3D CNN (3D ResNet)** to recover volumetric information.

### 5.3 Evaluate properly

- Report **accuracy, precision, recall, F1-score and a confusion matrix** on the held-out test set — not on individual predictions.
- In medical imaging, **recall on the cancer classes matters more than overall accuracy**. A missed cancer (false negative) is more dangerous than a false alarm.
- Use **k-fold cross-validation** to make sure the result is stable and not just lucky on one split.
- Consider **Grad-CAM** or similar saliency maps to verify that the model is actually looking at the tumor and not at scanner artifacts or background.

### 5.4 Explainability and clinical readiness

- Attach a **confidence threshold**: below a certain score, send the case to a human radiologist instead of making an automatic decision.
- Document **failure cases** — scans the model gets wrong — and analyze them. They usually reveal the next data or preprocessing improvement you need.
- If the goal is a real product, **validate on an external dataset** from a different hospital. A model that works on one scanner often fails on another (domain shift).

---

## 6. Conclusion

The current Teachable Machine experiment is a useful first prototype: it shows that a CNN can distinguish a normal CT scan from an adenocarcinoma CT scan with high confidence on the tested images. However, the model only covers two of the four intended classes, is trained on a very small dataset, and uses a fixed black-box pipeline.

To turn this prototype into a reliable diagnostic aid, the next step is to re-train on the full Kaggle dataset inside a proper deep-learning framework, apply CT-aware preprocessing and augmentation, evaluate with recall-oriented metrics, and add explainability so clinicians can trust the predictions.

---

*Note: this report is based on the two Teachable Machine prediction screenshots and the Kaggle dataset summary provided. Quantitative numbers (accuracy, F1, etc.) were not available and should be measured on the held-out test set as a next step.*
