# Factory Acoustic Monitor — Machine Sound Frequency Recorder

A browser-based prototype for **recording and visualizing machine sound frequencies** in an
electronics-manufacturing environment, inspired by spectral "acoustic manifold" visualizations
of bird song, adapted for industrial condition monitoring.

## Quick start

Open `index.html` in Chrome or Edge (no server or install needed).

1. Click **Start microphone** near the machine (grant mic permission), or **Demo machine sound** to preview with a synthesized motor sound.
2. Click **Set healthy baseline** while the machine runs normally — future deviations are flagged with an on-screen alert.
3. **Record** saves the raw audio to a file; **Export CSV** downloads the per-frame acoustic feature log for analysis or ML training.

All processing runs locally in the browser (Web Audio API). No data leaves the device.

## What the prototype shows

| Panel | What it is |
|---|---|
| Acoustic manifold | Rotating 3D trajectory of the sound: time × spectral centroid × amplitude, bubble color = spectral spread (like the bird-song reference) |
| Live spectrum | Real-time FFT with spectral-centroid marker |
| Spectrogram | Frequency content over time (waterfall) |
| Multi-scale radar | The machine's acoustic "signature" across 6 features — its shape changes when machine condition changes |
| Metrics + log | Numeric features, baseline deviation %, event log |

## Information captured (per ~100 ms frame, exported to CSV)

- **Level (dBFS)** — overall loudness / energy
- **Dominant frequency (Hz)** — strongest tone, e.g. motor rotation harmonics
- **Spectral centroid (Hz)** — "brightness"; shifts when bearings wear or friction increases
- **Spectral spread (Hz)** — how wide the energy is distributed; grows with rattle/looseness
- **Spectral flatness** — tonal (healthy rotating machine) vs. noisy (grinding, air leaks)
- **Crest factor** — impulsiveness; spikes indicate impacts, knocking, bearing defects
- **Rolloff 85% (Hz)** — high-frequency energy content
- **Baseline deviation %** — distance from the saved "healthy" spectrum (simple anomaly score)
- Raw **audio recording** (WAV/WebM) with timestamps

## Technologies for a real production deployment

**Edge (on the factory floor)**
- Industrial MEMS or ICP measurement microphones (flat response, up to ultrasonic 40–100 kHz for early bearing/discharge detection), plus optional accelerometers for vibration correlation
- Edge gateway per line: Raspberry Pi / NVIDIA Jetson / industrial PC running Python (`librosa`, `scipy`, `numpy`) or C++ DSP for feature extraction on-device, so only compact features stream upstream

**Transport & backend**
- MQTT or OPC-UA to integrate with existing PLC/SCADA/MES systems
- Time-series database: InfluxDB or TimescaleDB for features; object storage (S3/MinIO) for raw audio clips around alert events
- Stream processing: Kafka + Flink (or lightweight Node/Python services) for real-time rule and threshold evaluation

**Machine learning**
- Anomaly detection: autoencoders or Gaussian-mixture models trained on the "healthy" feature distribution (unsupervised — no failure examples needed to start)
- Fault classification (once labeled data accumulates): CNNs on mel-spectrograms (the industry-standard approach, e.g. as used with the MIMII industrial-sound dataset)
- Frameworks: PyTorch / TensorFlow, ONNX for edge inference

**Dashboard & alerting**
- Grafana or a custom web app (React + WebGL/Three.js for the 3D manifold view)
- Alerts to Andon boards, email, SMS, or CMMS work-order creation (e.g. SAP PM integration)

## Business benefits for an electronics manufacturer

1. **Predictive maintenance** — detect bearing wear, motor imbalance, loose fixtures, and failing fans days or weeks before breakdown, converting unplanned downtime into scheduled maintenance. Unplanned line stops in SMT/assembly are typically the single largest avoidable cost this addresses.
2. **Quality assurance** — abnormal sounds from placement machines, presses, screwdriving, or conveyors often correlate with defective output; catching acoustic drift catches quality drift early.
3. **Non-invasive retrofit** — microphones need no machine modification, no PLC changes, and work on legacy equipment where adding vibration sensors is impractical.
4. **Process verification** — confirm operations completed correctly (e.g. a press cycle's acoustic signature) as an automated in-line check.
5. **Workplace safety & compliance** — continuous dB(A) logging documents noise-exposure compliance (hearing-conservation regulations) as a free by-product.
6. **Institutional knowledge capture** — experienced technicians "hear" problems; this system digitizes that skill so it scales across shifts and sites, and builds a labeled dataset that grows more valuable over time.
7. **Energy & OEE insight** — acoustic load signatures reveal idle-but-running equipment and feed OEE availability metrics.

## Limitations of this prototype (what production adds)

- Browser mics are consumer-grade and capped near 20 kHz; production uses calibrated industrial microphones and higher sample rates.
- Baseline deviation here is a simple spectral distance; production uses trained anomaly models robust to background noise and multiple machines.
- No persistence/backend in the prototype — data lives in the page until exported.
