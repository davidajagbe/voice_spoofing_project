# Voice Detection System for Identifying AI-Generated Sound or Audio

An end-to-end speech anti-spoofing and audio deepfake detection system designed to classify uploaded voice recordings as either **bonafide** (authentic human speech) or **spoofed** (artificially synthesized or converted speech). This project represents an undergraduate final-year research initiative in the **Department of Cyber Security, Faculty of Computing and Digital Technologies, Redeemer's University (RUN), Ede, Osun State, Nigeria**.

---

**Authors and Academic Metadata**

* **Ajagbe David Temiloluwa** — Matric No: `RUN/CYB/24/17741`
* **Onofua Uyoyooghene Elizabeth** — Matric No: `RUN/CYB/23/15417`
* **Supervisor:** Mr. S. J. Agbolade
* **Degree Program:** B.Sc. Cyber Security
* **Academic Session:** 2025 / 2026
* **Repository:** https://github.com/davidajagbe/voice_spoofing_project

---

## Project Overview and Scope

Rapid advancements in generative deep learning enable modern Text-to-Speech (TTS) and Voice Conversion (VC) algorithms to synthesize speech with high acoustic fidelity. While valuable in accessibility applications and conversational agents, synthetic speech introduces security vulnerabilities in Automatic Speaker Verification (ASV) systems and voice-based authentication protocols.

This project designs, evaluates, and deploys a lightweight, reproducible machine-learning countermeasure to identify digital voice spoofing attacks:

* **Threat Model:** Focuses strictly on **Logical Access (LA)** attacks, encompassing synthetic speech (TTS) and voice conversion (VC) transmitted digitally via network endpoints or file uploads.
* **Input Audio Standardization:** Accepts digital audio recordings (`.wav`, `.flac`, `.mp3`) resampled to uniform **16 kHz single-channel mono**, volume-normalized, and padded or trimmed to a fixed clip duration to prevent models from learning clip length instead of vocal quality.
* **Feature Representation:** Extracts an **80-dimensional acoustic feature vector** per clip using Librosa, composed of the mean (40 values) and standard deviation (40 values) of 40 Mel-Frequency Cepstral Coefficients across time frames.
* **Classification Engine:** Deploys a **Random Forest Classifier** as an interpretable student baseline, benchmarked against advanced deep learning architectures such as AASIST and RawNet2.
* **Web Deployment:** A lightweight, interactive **FastAPI** REST backend paired with a web frontend that computes and displays real-time spoof probability scores.

---

## System Architecture and Workflow

The system operates as an end-to-end pipeline spanning audio upload, standardization, acoustic feature extraction, classification, and client-side presentation:

1. **User Audio Upload:** Accepts audio files (WAV, FLAC, MP3, M4A) via a web dashboard.
2. **Audio Preprocessing:** Resamples audio to 16,000 Hz mono, normalizes peak amplitude, and pads or trims clips to a fixed 3.0-second window.
3. **Acoustic Feature Extraction:** Computes 40 MFCCs using Librosa across time frames to generate an 80-dimensional vector (40 means + 40 standard deviations).
4. **Machine Learning Classification:** Evaluates features using a Random Forest classifier to predict spoof probability scores between 0.0 and 1.0.
5. **Result Delivery:** Returns a structured JSON response displaying classification labels, confidence scores, risk tiers, and latency metrics.

---

## Benchmark Datasets

To ensure experimental rigor and eliminate data leakage, models are trained, tuned, and evaluated on disjoint public benchmark partitions:

1. **ASVspoof 2019 Logical Access (LA):** Primary training and development database covering 17 TTS, VC, and hybrid generation systems across 19 algorithms (A01–A19).
2. **ASVspoof 2021 Logical Access (LA) and Speech Deepfake (DF):** Benchmarks model resilience against telephony transmission channels, lossy codecs, and network compression.
3. **Fake-or-Real (FoR):** Contains ~198,000 utterances (87k synthetic, 111k real) balanced across genders and normalized across recording channels.
4. **WaveFake and In-the-Wild Datasets:** WaveFake (117,985 clips across six generator architectures) and In-the-Wild corpora are utilized for cross-dataset generalization testing.

---

## Repository Directory Structure

```text
voice_spoofing_project/
|
|-- data/
|   |-- raw/                          # Raw ASVspoof / FoR datasets (excluded via .gitignore)
|   |-- processed/                    # Standardized 16kHz mono audio files
|   |-- protocols/                    # Ground-truth label files (*.trn.txt, *.trl.txt)
|   `-- features/                     # Extracted 80-dim feature matrices (*.csv, *.npy)
|
|-- models/
|   |-- random_forest_baseline.joblib # Serialized Random Forest classifier
|   `-- feature_scaler.joblib         # Fitted StandardScaler parameters
|
|-- notebooks/
|   |-- 01_audio_exploration.ipynb   # Waveform, mel-spectrogram, and MFCC visualizations
|   |-- 02_preprocessing.ipynb       # Resampling, mono conversion, and duration trimming
|   |-- 03_feature_extraction.ipynb  # Batch generation of 80-dim MFCC feature matrices
|   `-- 04_model_evaluation.ipynb    # ERR, ROC-AUC, and Confusion Matrix calculations
|
|-- src/
|   |-- __init__.py
|   |-- config.py                    # Constants (SAMPLE_RATE=16000, DURATION=3.0, N_MFCC=40)
|   |-- preprocess.py                # Resampling, mono conversion, and padding routines
|   |-- extractor.py                 # Librosa 40-MFCC mean and std computation
|   |-- train.py                     # Scikit-learn training and hyperparameter tuning
|   `-- evaluate.py                  # ERR, ROC-AUC, and classification metric evaluations
|
|-- app/
|   |-- main.py                      # FastAPI REST application endpoints
|   |-- static/
|   |   |-- css/style.css            # Responsive dashboard layout (Tailwind CSS)
|   |   `-- js/app.js                # Asynchronous upload handler and DOM updater
|   `-- templates/
|       `-- index.html               # Main verification interface
|
|-- tests/
|   |-- test_preprocessing.py        # Unit tests for signal standardization
|   `-- test_api.py                  # API endpoint integration and upload tests
|
|-- requirements.txt                 # Pinned Python package dependencies
|-- .gitignore                       # Rules excluding raw audio, models, and virtual environments
`-- README.md                        # Documentation
```

---

## Execution Guide

**Step 1: Feature Extraction**
Extract the 80-dimensional feature vector across audio clips using the challenge protocols:
```bash
python src/extractor.py --audio_dir data/raw/LA/ASVspoof2019_LA_train/flac \
                        --protocol data/protocols/ASVspoof2019.LA.cm.train.trn.txt \
                        --output data/features/train_features.csv
```

**Step 2: Baseline Model Training**
Train the Random Forest classifier on the training partition and evaluate against the development partition:
```bash
python src/train.py --train_features data/features/train_features.csv \
                    --dev_features data/features/dev_features.csv \
                    --model_out models/random_forest_baseline.joblib
```

**Step 3: Model Evaluation**
Compute ERR, ROC-AUC, and confusion matrices across evaluation partitions:
```bash
python src/evaluate.py --model models/random_forest_baseline.joblib \
                       --test_features data/features/dev_features.csv
```

**Step 4: Web Prototype Deployment**
Launch the FastAPI development server:
```bash
uvicorn app.main:app --host 127.0.0.1 --port 8000 --reload
```
Access the application locally:
* **Interactive Web Interface:** http://127.0.0.1:8000
* **OpenAPI Documentation:** http://127.0.0.1:8000/docs

---

## Evaluation Metrics and Performance Targets

* **Equal Error Rate (EER):** The threshold where the False Acceptance Rate (FAR) equals the False Rejection Rate (FRR); lower values indicate higher discrimination.
* **Area Under the ROC Curve (ROC-AUC):** Assesses threshold-independent binary discrimination.
* **Confusion Matrix & F1-Score:** Tracks specific false positive and false negative error counts.
* **Inference Latency:** Target operational latency of under **150 milliseconds** on CPU for a standardized 3-second recording.

---

## Security, Privacy and Ethical Compliance

* **Ephemeral Data Handling:** Uploaded voice recordings are processed directly in transient memory buffers and discarded immediately after inference without permanent server storage.
* **Input Sanitization:** Upload endpoints enforce strict MIME type and extension validation with a 10 MB file size limit to mitigate Denial of Service (DoS) risks.
* **Probabilistic Reporting:** Model outputs represent statistical confidence scores rather than definitive legal or forensic conclusions.
* **Data Privacy Regulations:** Research and prototype development adhere to the data handling provisions of the **Nigeria Data Protection Act (NDPA 2023)**.

---

## 12-Week Implementation Schedule

```text
Phase 1 (Weeks 1 - 2)  : Literature Review, Proposal Approval & Topic Defense
Phase 2 (Weeks 3 - 4)  : Dataset Acquisition, Protocol Parsing & 16kHz Audio Standardization
Phase 3 (Weeks 5 - 6)  : Librosa Feature Extraction (80-Dimensional MFCC Matrix CSVs)
Phase 4 (Weeks 7 - 8)  : Baseline Random Forest Training & EER/ROC-AUC Evaluation
Phase 5 (Weeks 9 - 10) : FastAPI Backend Integration & Web Dashboard Interface
Phase 6 (Weeks 11 - 12): Final Dissertation Compilation, Pre-Data Defense & Final Examination
```

---

## Key Academic References

* Alzantot, M., Wang, Z., & Srivastava, M. (2019). Deep residual neural networks for audio spoofing detection. *Proceedings of Interspeech 2019*, 1078–1082.
* Frank, J., & Schönherr, L. (2021). WaveFake: A data set to facilitate audio deepfake detection. *NeurIPS Datasets and Benchmarks 2021*.
* Jung, J., Heo, H., Tak, H., Shim, H., Chung, J. S., Lee, B., Yu, H., & Evans, N. (2022). AASIST: Audio anti-spoofing using integrated spectro-temporal graph attention networks. *IEEE ICASSP 2022*, 6367–6371.
* Liu, Y., Fan, Y., & Chen, L. (2023). A comprehensive survey on audio deepfake detection: Taxonomy, datasets, and future directions. *ACM Computing Surveys*, 55(11), 1–36.
* McFee, B., Raffel, C., Liang, D., Ellis, D. P., McVicar, M., Battenberg, E., & Nieto, O. (2015). librosa: Audio and music signal analysis in python. *Proceedings of the 14th Python in Science Conference*, 18–25.
* Müller, N. M., Czempin, P., Dieckmann, F., Froghyar, A., & Böttinger, K. (2022). Does audio deepfake detection generalize? *Proceedings of Interspeech 2022*, 3421–3425.
* Reimao, R., & Tzerpos, V. (2019). FoR: A dataset for synthetic speech detection. *Speech Communication*, 115, 12–22.
* Tak, H., Patino, J., Todisco, M., Nautsch, A., Evans, N., & Larcher, A. (2021). End-to-end anti-spoofing with RawNet2. *IEEE ICASSP 2021*, 6383–6387.
* Todisco, M., Wang, X., Evans, N., Sahidullah, M., Delgado, H., Kinnunen, T., Yamagishi, J., & Lee, K. A. (2019). ASVspoof 2019: Future horizons in spoofed and counterfeit voice detection. *Proceedings of Interspeech 2019*, 1008–1012.
* Wang, X., Yamagishi, J., Todisco, M., Delgado, H., Evans, N., Kinnunen, T., Lee, K. A., & Vestman, V. (2020). ASVspoof 2019: A large-scale public database of synthesized, converted and replayed speech. *Computer Speech & Language*, 64, 101114.
* Yamagishi, J., Wang, X., Todisco, M., Sahidullah, M., Delgado, H., Evans, N., Kinnunen, T., & Lee, K. A. (2021). ASVspoof 2021: Accelerating progress in spoofed and deepfake speech detection. *ASVspoof 2021 Workshop*, 1–8.
