# Preterm-Bradycardia-Prediction-from-ECG-Using-a-Convolutional-Neural-Network
MATLAB pipeline for predicting bradycardia in preterm infants from ECG signals using a 1D Fully Convolutional Network. The project preprocesses PICS recordings, creates balanced control and pre-bradycardia windows, and evaluates the model with subject-wise validation.

# Preterm Bradycardia Prediction from ECG Using a 1D Fully Convolutional Network

This project implements a MATLAB pipeline for identifying ECG patterns preceding bradycardia events in preterm infants. The pipeline processes neonatal ECG recordings, creates balanced control and pre-bradycardia windows, and trains a one-dimensional Fully Convolutional Network to classify the two conditions.

## Objective

Bradycardia is a frequent clinical event in preterm infants and may be associated with cardiorespiratory instability. The objective of this project is to investigate whether ECG morphology and temporal dynamics contain information that can distinguish normal ECG segments from segments immediately preceding a bradycardia event.

The model was trained using ECG recordings acquired in the Intensive Care Unit of a hospital in Cesena, Italy, together with publicly available recordings from the Preterm Infant Cardio-Respiratory Signals (PICS) Database hosted on PhysioNet.

The model receives three-minute ECG windows and classifies each window as:

**Control:** ECG activity recorded away from annotated bradycardia events.
**Pre-bradycardia:** ECG activity recorded during the three minutes immediately preceding a bradycardia onset.

## Dataset

For each infant, the generated CSV files contain:
sample, ecg, brady


or, when R-peak annotations are included:
sample, ecg, brady, rpeak


Where:

* `sample` is the ECG sample index.
* `ecg` is the ECG amplitude.
* `brady` identifies annotated bradycardia onsets.
* `rpeak` identifies detected or annotated R peaks.

## Project Structure

preterm-bradycardia-prediction-1d-fcn/
│
├── code/
│   ├── 01_ecg_visualization.m
│   ├── 02_preprocessing.m
│   ├── 03_data_preparation.m
│   └── 04_train_fcn.m
├── README.md

```

## Pipeline Overview

Raw ECG recordings
        ↓
ECG visualization and quality inspection
        ↓
Signal preprocessing
        ↓
Three-minute window generation
        ↓
Balanced control and pre-bradycardia dataset
        ↓
Subject-wise chronological split
        ↓
1D Fully Convolutional Network
        ↓
Validation predictions and performance metrics
```

## 1. ECG Visualization

The ECG visualization module is used to inspect the neonatal recordings before model development.

Its main purposes are:

* Plot representative ECG segments.
* Visualize the position of annotated bradycardia events.
* Visualize R-peak annotations when available.
* Check ECG amplitude and signal morphology.
* Detect noisy, flat, saturated, or corrupted signal intervals.
* Compare ECG activity before and during clinically annotated events.

A representative visualization may include the ECG waveform together with vertical markers showing bradycardia onsets and R peaks.

## 2. ECG Preprocessing

The preprocessing module prepares each ECG window before it is provided to the neural network.

The main preprocessing operations are:

### Band-pass filtering

A band-pass filter between approximately **0.5 and 40 Hz** is applied to reduce:

* Baseline drift.
* Slow non-physiological fluctuations.
* High-frequency noise.
* Part of the muscular and acquisition artefacts.

### Power-line interference removal

A notch filter centred at **50 Hz** can be applied to reduce electrical interference from the power supply.

### Resampling

All ECG windows are resampled to 250 Hz

### Normalization

Each ECG window is independently standardized using z-score normalization:

## 3. Data Preparation

The data-preparation module converts the continuous ECG recordings into observations suitable for deep learning.

### Pre-bradycardia windows

For every valid bradycardia annotation, the pipeline extracts the ECG interval immediately preceding the event. Each window has a duration of 3 minutes

### Control windows

Control windows are extracted from ECG intervals located away from bradycardia annotations. A safety margin is used to prevent control windows from overlapping with, or being too close to, a bradycardia event.

## 4. Training and Validation Split

The dataset is divided separately for each infant to preserve the chronological organization of the recordings.

For each subject:

* The first 70% of the event-control pairs are assigned to training.
* The remaining 30% are assigned to validation.

The control and pre-bradycardia windows belonging to the same pair are always assigned to the same partition. This prevents one member of a matched pair from appearing in training while the other appears in validation. 

## 5. Fully Convolutional Network

The classification model is a one-dimensional Fully Convolutional Network designed to learn temporal features directly from the ECG waveform.

The network architecture is:

Input ECG sequence
        ↓
Conv1D: 1 → 128 filters, kernel size 8
        ↓
Batch normalization
        ↓
ReLU
        ↓
Conv1D: 128 → 256 filters, kernel size 5
        ↓
Batch normalization
        ↓
ReLU
        ↓
Conv1D: 256 → 128 filters, kernel size 3
        ↓
Batch normalization
        ↓
ReLU
        ↓
Average pooling
        ↓
Global average pooling
        ↓
Fully connected layer
        ↓
Softmax classification



## Training Configuration

The current MATLAB implementation uses:

```text
Optimizer: Adam
Learning rate: 0.001
Maximum epochs: 50
Mini-batch size: 1
Execution: Serial
```

The small mini-batch size reduces GPU memory requirements when processing three-minute ECG windows containing 45,000 samples.

## Model Outputs

For each validation window, the model produces:

* Predicted class.
* Control probability.
* Pre-bradycardia probability.
* Infant identifier.
* Event-control pair number.

## Performance Metrics

The model is evaluated using several complementary metrics:

### Accuracy

### Precision

### Recall

### F1-score

### Confusion matrix

The confusion matrix reports:

* Correctly classified control windows.
* Control windows classified as pre-bradycardia.
* Pre-bradycardia windows classified as control.
* Correctly classified pre-bradycardia windows.

## Requirements

The project requires:

* MATLAB.
* Deep Learning Toolbox.
* Signal Processing Toolbox.
* WFDB Toolbox for MATLAB for the initial annotation and recording conversion.
