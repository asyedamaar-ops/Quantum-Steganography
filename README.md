<div align="center">

# 🔐 QSteg — Quantum-Secured Image Steganography

**Hide encrypted secrets inside images. Undetectable by classical steganalysis.**

[![Python](https://img.shields.io/badge/Python-3.9%2B-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Qiskit](https://img.shields.io/badge/Qiskit-AerSimulator-6929C4?style=flat-square&logo=ibm&logoColor=white)](https://qiskit.org/)
[![AES-256](https://img.shields.io/badge/Encryption-AES--256--GCM-00B4A0?style=flat-square&logo=lock&logoColor=white)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)
[![QCC Project](https://img.shields.io/badge/Course-Quantum%20Communication%20%26%20Cryptography-8A2BE2?style=flat-square)]()

</div>

---

## The Problem with Classical Steganography

Every LSB (Least Significant Bit) steganography tutorial online is broken.

An attacker runs a chi-square test on the pixel histogram and detects the hidden data — instantly, without knowing what it says. The "invisible" message was never really invisible.

**QSteg fixes this.** It combines three layers:

```
Real quantum randomness  →  AES-256-GCM encryption  →  Frequency-domain DCT hiding
```

The result is a system that is both **quantum-secured** and **statistically invisible** to classical steganalysis tools.

---

## How It Works

### Layer 1 — Quantum Key Generation
Instead of seeding a classical RNG, the AES-256 key is generated entirely from quantum circuits. 8 qubits in Hadamard superposition are measured 32 times, producing 256 physically random bits — no pseudorandomness in the loop.

```python
qc = QuantumCircuit(8, 8)
for i in range(8):
    qc.h(i)          # superposition — 50/50 on measurement
qc.measure(range(8), range(8))
```

### Layer 2 — AES-256-GCM Encryption
The message is encrypted *before* hiding it. Even if an attacker detects and extracts the stego data, they get encrypted ciphertext. GCM mode adds an authentication tag — wrong key = hard error, not garbled text.

### Layer 3 — DCT Steganography with QIM
Data is embedded in the frequency domain using Discrete Cosine Transform (the same domain JPEG uses), specifically in mid-frequency coefficients via Quantization Index Modulation. The order of blocks used for embedding is determined by a hash of the quantum key — so the embedding pattern itself is key-dependent.

```
Pixel domain   →   DCT   →   embed in mid-freq coefficients   →   IDCT   →   Stego image
                              (order determined by quantum key)
```

---

## Results

| Metric | QSteg (DCT) | Basic LSB |
|---|---|---|
| **Chi-square p-value** | > 0.05 ✅ UNDETECTED | ~0.000000 ❌ CAUGHT |
| **RS Analysis score** | < 0.1 ✅ CLEAN | > 0.5 ❌ DETECTED |
| **PSNR** | > 42 dB (better than JPEG 90%) | ~51 dB |
| **SSIM** | > 0.999 | ~0.999 |
| **Message security** | AES-256-GCM encrypted | Plaintext in bits |

---

## Project Structure

```
quantum_stego_advanced.ipynb   ← Main notebook — run this
├── Part 1: Quantum Key Generation (Hadamard circuits, 256-bit AES key)
├── Part 2: AES-256-GCM Encryption layer
├── Part 3: DCT + QIM steganography engine
├── Part 4: Encode / Decode demo with synthetic cover image
├── Part 5: Visual quality metrics (PSNR, SSIM, diff maps)
├── Part 6: Steganalysis tests (chi-square + RS analysis vs. LSB baseline)
└── Part 7: Decode + verify + key export
```

---

## Quickstart

### 1. Clone and install dependencies

```bash
git clone https://github.com/<your-username>/qsteg.git
cd qsteg
pip install -r requirements.txt
```

### 2. Open the notebook

```bash
jupyter notebook quantum_stego_advanced.ipynb
```

### 3. Run all cells

The notebook is self-contained. A synthetic 512×512 cover image is generated automatically so no external files are needed. To use your own image:

```python
COVER_PATH = 'your_photo.png'   # change this line in Part 4
```

> **Note:** Always save stego images as PNG. JPEG compression modifies DCT coefficients and will destroy the hidden data.

---

## Requirements

```
qiskit
qiskit-aer
pillow
scipy
pycryptodome        # NOT pycrypto — they conflict
numpy
matplotlib
scikit-image
```

See `requirements.txt` for pinned versions.

---

## Design Decisions

**Why generate the AES key from quantum circuits, not just seed a PRNG with one?**
Seeding a classical RNG with a quantum output still runs pseudorandom operations — the quantum step becomes cosmetic. QSteg uses quantum measurement outcomes as the key bits directly.

**Why GCM mode for AES?**
GCM adds an authentication tag. A wrong key produces a MAC verification error, not garbled output. Hard failures are unambiguous; garbled text could be mistaken for a partial success.

**Why DCT and not just better LSB?**
LSB touches pixel values directly and always produces a detectable histogram signature. DCT embedding operates in the frequency domain where natural image variation provides cover, and QIM doesn't create the paired-value artifact that chi-square exploits.

**Why embed in mid-frequency coefficients?**
DC coefficients control overall brightness (too visible). High-frequency corners are discarded by JPEG compression (not robust). Mid-frequency is the sweet spot — perceptually masked by texture and compression-stable in PNG.

---

## Limitations — Being Upfront

**AerSimulator ≠ real quantum hardware.**
The simulator accurately models quantum gate operations, but the underlying randomness is classical. For genuine quantum randomness, swap `AerSimulator` for `qiskit-ibm-runtime` with an IBM Quantum backend — the circuit code is identical.

**JPEG breaks it.**
Save stego images as PNG only. JPEG's lossy re-quantization corrupts the embedded bits.

**Key distribution is unsolved here.**
The quantum key must reach the receiver securely via a separate channel. This project exports it manually. A production system would use QKD or asymmetric key exchange.

**ML-based steganalysis is out of scope.**
QSteg defeats chi-square and RS analysis. It has not been evaluated against deep-learning detectors like SRNet or Zhu-Net, which are trained on frequency-domain steganography. Defeating those is an active research problem.

**Capacity scales with image resolution.**
A 256×256 image holds ~500 characters. A 1024×1024 holds ~8,000.

---

## Background

This was my semester project for the Quantum Communication & Cryptography course. The motivation came from a question during the QKD lecture: if quantum mechanics gives us truly random numbers, why does every security demo still use `random.seed(42)`?

The project started as basic LSB steganography with a quantum key. After reading about steganalysis, I found that any free online tool can detect LSB with a chi-square test — so the whole design had to change. This is the result.

---

## License

MIT — use it, modify it, cite it if it helps.

---

<div align="center">
<sub>Built with Qiskit · AES-256-GCM · DCT-QIM · Python</sub>
</div>
