# Implementation and Evaluation of Deep Neural Network (DNN) Based Calibration of High-Precision Electrometer on FPGA

## 📌 Overview

This repository contains the implementation, training, and deployment of a lightweight digital calibration framework for high-resolution analog-to-digital converters (ADCs). The calibration is achieved using a quantized deep neural network (DNN) trained in QKeras and deployed onto a Xilinx Zynq-7020 FPGA via the hls4ml framework. An alternative LUT-based correction method using residual error interpolation is also explored.

This DNN is focused on improving real-time accuracy of electrometer ADC outputs used in beam diagnostics.

---

## Project Summary

High-resolution electrometers at NSLS-II utilize 20-bit ADCs for precision beam monitoring. However, these systems are susceptible to gain errors, offsets, and subtle nonlinearities. Traditional polynomial calibration methods lack flexibility and are performed offline.

This project proposes:
- A quantized, single-layer DNN trained to correct ADC errors in real-time.
- Deployment of this model to FPGA using hls4ml.
- Evaluation of performance compared to polynomial fits and LUT correction.

---

## 🧠 Model Architecture

- **Type:** Fully connected neural network  
- **Layers:** Single dense layer with 1 input and 1 output  
- **Activation:** Linear  
- **Loss:** Mean Squared Error (MSE)  
- **Optimizer:** Adam  
- **Training Epochs:** 600  
- **Normalization:** Z-score normalization on both input and output  
- **Frameworks:** Keras + QKeras for quantization

---

## Software and Toolchain Versions Used

| Tool/Framework       | Version        |
|----------------------|----------------|
| Python               | 3.9.21         |
| TensorFlow           | 2.11.1         |
| Keras                | 2.11.0         |
| QKeras               | 0.9.0          |
| hls4ml               | 0.8.1          |
| Vivado HLx           | 2019.2         |
| PYNQ                 | v2.5 / v2.6    |
| Target FPGA          | Zynq-7020 (xc7z020clg400-1) |

---

## Methodology

1. **Data Collection**
   - High-precision current sweeps across six ranges (±100 nA to ±10 mA) were performed using a Keithley 6221 current source and a quadEM electrometer.
   - 101 samples per range per channel were collected and stored as CSV files.

2. **Baseline Calibration**
   - First-order polynomial fits were used to calculate gain and offset.
   - This method lacked precision in capturing ADC nonlinearities.

3. **DNN Training & Quantization**
   - A simple DNN was trained on normalized data.
   - Post-training, weights were denormalized and quantized using QKeras for FPGA deployment.

4. **hls4ml Conversion & Deployment**
   - The model was converted to HLS using `hls4ml` and deployed onto the PYNQ-Z2 board via the VivadoAccelerator backend.
   - An AXI DMA interface was used for test input/output.

5. **Evaluation**
   - Simulated predictions were compared between:
     - Floating-point Keras model
     - Fixed-point HLS model
     - Real FPGA-deployed model
   - All achieved <1 µA RMS error in the ±1 mA range.

---

## Simulation Results

| Model Version            | RMS Error (µA) |
|--------------------------|----------------|
| Linear Fit (baseline)    | 0.1839         |
| Floating-point DNN       | 0.1841         |
| HLS (fixed-point) DNN    | 0.3750         |
| HLS vs. Keras difference | 0.3246         |
| LUT-Corrected Model      | **0.0033**     |

> The LUT-based method, using cubic interpolation of the residuals and a 20-bit upsampled LUT, achieved a 5.79× improvement in RMS error but was not feasible to deploy on hardware due to memory constraints.

<img width="583" height="393" alt="Residual comparison between DNN and linear models" src="https://github.com/user-attachments/assets/107de5ea-eae4-45bf-a5b7-e7e7375131fb" />
**Figure:** Residual error comparison between the DNN and linear calibration models across the full current range.

<img width="590" height="290" alt="Absolute error per sample" src="https://github.com/user-attachments/assets/bc5447b0-3c21-46e8-b9c3-d3cc5c337d89" />
**Figure:** Absolute error per test sample between the floating-point Keras model and the fixed-point HLS model.

---

## Notebooks

- `nsls2em_dnn_lin_calibration.ipynb`: Trains and evaluates a minimal DNN on electrometer data.
- `adc_deployment.ipynb`: Converts the trained model to HLS using hls4ml and tests DMA communication with the FPGA.
- `LUT_Correction.ipynb`: Simulates residual error correction using a 20-bit interpolated LUT for comparison.
