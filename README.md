# Predictive Pulse

### Machines That Warn You Before They Fail

> A low-cost, offline Edge-AI sensor network that clips onto existing industrial machines, listens to vibration, temperature, motor current and sound, and warns maintenance teams of a developing fault in plain language, days before it becomes an unplanned breakdown. No cloud required.

**Avishkar 2026** | Theme: Smart Industrial Automation | Category: Engineering and Technology

![Poster](images/poster.jpg)
<!-- Replace with your clean poster image -->

---

## The Problem

- Breakdowns come without warning and production stops.
- Fixed monthly maintenance wastes money on healthy machines and still misses real faults.
- Remote sites such as mines have weak or no internet, so cloud-based monitoring does not work.
- One hour of unplanned downtime can cost **₹1.5 - 5 lakh** for an MSME (figure from the poster; add your source here).

## The Solution

A small sensor node is bolted or clamped onto a machine. It **senses**, **thinks** and **acts**:

```
SENSE  ->  THINK  ->  ACT
```

| Stage | What happens | Where it runs |
|-------|--------------|---------------|
| **Sense** | Reads vibration, temperature, motor current and (optionally) sound, then filters and extracts 14 health features per node | ESP32-S3 node |
| **Think (Level 1)** | Z-score, CUSUM and EWMA check each signal against its own recent history | ESP32-S3 node |
| **Think (Level 2)** | Fuses features from all nodes on a machine and classifies the fault with a Random Forest model | Raspberry Pi |
| **Act** | Shows a plain-language alert, logs a ticket, and learns from technician feedback | Local dashboard |

Example dashboard output:

```
Pump 01 - Warning
  Maintenance recommended within 72 hrs

Motor 01 - Normal
  No immediate action
```

## System Architecture

```
ESP32 Node 1 --\
ESP32 Node 2 ---+--MQTT--> Raspberry Pi (Edge AI) --> Local Dashboard
ESP32 Node 3 --/                  |
                                  +--> Optional: PLC / SCADA (Modbus, OPC UA)
```
<!-- Replace with an architecture diagram image if you have one -->

## Key Features

- **Works offline:** all sensing, DSP, classification and dashboard run on-site.
- **Low-cost add-on:** about ₹3,900 per node (planning figure); no change to the machine or its wiring.
- **Explainable:** alerts show which readings and trends caused them, not just a score.
- **Scalable:** one Raspberry Pi serves many nodes.
- **Continuously learning:** technician-confirmed alerts become new training data.
- **Non-intrusive:** clip-on current transformer, surface-mounted accelerometer, clamped temperature probe.
- **Multi-sensor fusion:** a fault is confirmed from more than one signal, reducing false alarms.
- **Integrates with existing systems:** Modbus RTU/TCP and OPC UA, or runs standalone.

## Hardware

| Block | Component |
|-------|-----------|
| Controller | ESP32-S3 |
| Vibration | ADXL345 3-axis accelerometer (upgrade path: IIS3DWB) |
| Temperature | DS18B20 (machine body) + BME280 (ambient) |
| Motor current | SCT-013-030 split-core current transformer |
| Sound (optional) | INMP441 I2S MEMS microphone |
| Hub | Raspberry Pi 4 (4 GB) |
| Communication | MQTT over local Wi-Fi (Mosquitto broker on the Pi) |

Approximate cost per node: **₹2,920 - ₹4,800**. Prices are indicative; verify with your supplier.

## Signal Processing (on the node)

1. **Filtering:** DC removal, anti-alias low-pass, 50 Hz notch (IIR biquads).
2. **Moving average / RMS:** overall severity, crest factor, EWMA trend.
3. **FFT:** dominant frequencies, peak amplitude and band energy (2,048-point window, ESP-DSP).
4. **Rate of change / statistical shape:** kurtosis, Z-score, CUSUM, rate of rise.

The result is **14 features per node**, sent about once per second (roughly 70 bytes in binary form instead of about 19 kB of raw vibration data).

## Machine Learning

- **Model:** Random Forest classifier (runs on the Raspberry Pi, no GPU).
- **Data:** public bearing-fault datasets (for example CWRU, NASA IMS/PRONOSTIA) plus own healthy-machine recordings.
- **Classes validated:** Normal, Inner Race Fault, Outer Race Fault, Ball Fault, Misalignment, Unbalance.
- **Training approach:** train/validation/test split by machine and time period, class balancing, cross-validated tuning with priority on recall.

### Results

| Metric | Value |
|--------|-------|
| Overall accuracy | 92.4% |
| Macro F1-score | 0.92 |
| Precision | 0.93 |
| Recall | 0.91 |

![Confusion matrix and feature importance](images/results.png)
<!-- Replace with your confusion matrix / feature importance figure -->

Vibration features (RMS, FFT peak, kurtosis) contribute most to classification, followed by temperature and current features.

> **Note:** results come from the validation run described in the project document. Add details on exactly which dataset(s) and split produced these numbers so others can reproduce them.

## Impact

- Replaces calendar-based maintenance with condition-based maintenance.
- Makes condition monitoring affordable for MSMEs and usable at remote, offline sites.
- **Projected** 35% reduction in unplanned downtime and maintenance cost. This is a projection; the calculation template is in the project document and will be replaced with measured pilot data.

## Documentation

| File | Contents |
|------|----------|
| [`docs/Predictive_Pulse_Project_Document.pdf`](docs/Predictive_Pulse_Project_Document.pdf) | Full project document: hardware, sensors and cost, communication protocol, DSP, Edge-AI hub, software stack, ML training and results, PLC/SCADA integration, impact |
| [`poster/`](poster/) | Avishkar 2026 poster |
| [`case-study/`](case-study/) | Detailed case study |
| [`images/`](images/) | Figures used in this README |

## Repository Structure

```
predictive-pulse/
|-- README.md
|-- docs/
|   `-- Predictive_Pulse_Project_Document.pdf
|-- poster/
|-- case-study/
|-- images/
|-- firmware/        # ESP32 node code (add if available)
|-- edge/            # Raspberry Pi ingestion, ML and dashboard (add if available)
|-- ml/              # Training notebooks and scripts (add if available)
`-- LICENSE
```
<!-- Delete any folders you do not have. -->

## Status and Roadmap

- [x] System design and documentation
- [x] Random Forest model trained and validated
- [ ] Field pilot on a real machine
- [ ] Measured downtime and cost savings
- [ ] Wideband accelerometer upgrade (IIS3DWB)
- [ ] LoRa / ESP-NOW options for large sites
<!-- Edit the checkboxes to match your real progress. -->

## About

Designed and built by Shivam Ambilwade for Avishkar 2026.

- LinkedIn: https://www.linkedin.com/in/shivam-ambilwade-a3a9a1245/
- Email: shivamcreations609@gmail.com

Feedback from people in manufacturing, maintenance and Edge-AI is very welcome. Please open an issue or message me.

## License
MIT License

## Tags

`edge-ai` `predictive-maintenance` `industry-4-0` `iiot` `esp32` `raspberry-pi` `mqtt` `random-forest` `signal-processing` `msme`
