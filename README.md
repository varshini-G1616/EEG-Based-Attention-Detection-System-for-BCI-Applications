# EEG-Based-Attention-Detection-System-for-BCI-Applications
MATLAB-based EEG signal processing and machine-learning system for detecting Focused, Unfocused, and Drowsy attention states for Brain-Computer Interface (BCI) applications.

Project Overview

This project aims to develop an EEG-based attention detection system using MATLAB. The system processes multi-channel EEG signals, extracts meaningful time-domain and frequency-domain features, and applies machine-learning techniques to classify different attention states.

Objectives

Process and analyze raw EEG signals.

Extract statistical and frequency-band features from EEG data.

Classify Focused, Unfocused, and Drowsy attention states.

Evaluate the machine-learning model using standard performance metrics.

Visualize EEG signals, extracted features, and classification results.

Develop a MATLAB-based graphical user interface for displaying attention predictions.

Dataset

The project uses the EEG Data for Mental Attention State Detection dataset containing multiple EEG recordings sampled at 128 Hz.

The EEG recordings contain 14 channels:

AF3, F7, F3, FC5, T7, P7, O1, O2, P8, T8, FC6, F4, F8, AF4

The experimental protocol defines three attention states:

Focused: 0–10 minutes

Unfocused: 10–20 minutes

Drowsy: 20–30 minutes

The attention labels are based on the experimental protocol and are not inferred from the marker values in the recordings.

Methodology

Raw EEG Data
     ↓
Data Validation
     ↓
Preprocessing
     ↓
EEG Segmentation
     ↓
Feature Extraction
     ↓
Machine Learning
     ↓
Attention Classification
     ↓
Performance Evaluation
     ↓
MATLAB GUI

Features

The planned feature set includes statistical EEG features and frequency-band power features such as:

Mean

Variance

Standard deviation

RMS

Delta power

Theta power

Alpha power

Beta power

Relative band power

Technologies Used

MATLAB | Signal Processing | Machine Learning | EEG Analysis | Brain-Computer Interface

Project Status

The EEG data loading, channel extraction, sampling-frequency verification, and initial signal visualization have been completed. Preprocessing, feature extraction, machine-learning classification, evaluation, and GUI development are being developed progressively using actual experimental data.

Future Scope

The completed system will provide genuine model predictions and performance metrics and can serve as a foundation for real-time EEG-based attention monitoring and future BCI applications.
