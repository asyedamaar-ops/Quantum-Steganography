# QSteg: Quantum-Keyed Image Steganography

Course project for Quantum Communication and Cryptography. It hides an AES-256-GCM encrypted message inside the DCT coefficients of an image, using a key generated from Qiskit Hadamard circuits.

![Python](https://img.shields.io/badge/Python-3.9%2B-3776AB?style=flat-square&logo=python&logoColor=white)
![Qiskit](https://img.shields.io/badge/Qiskit-AerSimulator-6929C4?style=flat-square)
![AES-256-GCM](https://img.shields.io/badge/Encryption-AES--256--GCM-00B4A0?style=flat-square)

## What it does

```
Hadamard-circuit key  ->  AES-256-GCM encryption  ->  DCT-QIM embedding  ->  PNG stego image
```

1. **Key generation:** 8 qubits are put in superposition with Hadamard gates and measured 32 times, giving 256 bits for an AES-256 key. This runs on the Qiskit Aer simulator (see Limitations).
2. **Encryption:** the message is encrypted with AES-256-GCM before hiding. A wrong key gives a MAC failure instead of garbled text.
3. **Embedding:** ciphertext bits are hidden in mid-frequency DCT coefficients of 8x8 blocks (green channel) using Quantization Index Modulation (QIM, delta = 20). The order of blocks is a permutation derived from a hash of the key.
4. **Round-trip check:** after embedding, the encoder decodes its own output and raises an error if the message cannot be recovered.

## Results (synthetic 512x512 cover, 1920-bit demo payload)

| Check | Result |
|---|---|
| Capacity | 16,384 bits (about 2,000 characters) |
| Demo payload used | 1,920 bits (11.7%) |
| PSNR | about 50 dB |
| SSIM | about 0.998 |
| Exact message recovery | Yes |
| Wrong key | Fails with an error |

### Classic LSB steganalysis

Two attacks built for LSB embedding were run: the chi-square test (Westfeld and Pfitzmann) and RS analysis (Fridrich et al.). A result counts as flagged if chi-square p > 0.05 or the RS estimate is above 0.10 in absolute value.

| Image | Flagged? |
|---|---|
| Clean cover | No |
| DCT-QIM, demo payload | No |
| DCT-QIM, near full capacity | Inconclusive (the RS estimate varied between runs) |
| Basic LSB, same 1,920-bit payload | No |
| Basic LSB, whole LSB plane | **Yes** (RS estimate about 0.93) |

What this shows and what it does not:
- Basic LSB is only flagged when most of the LSB plane carries data. At small payloads these tests see nothing, for LSB and for DCT-QIM alike.
- DCT-QIM does not create the pair-equalisation pattern that chi-square looks for.
- These tests were designed for LSB. **Not being flagged is not proof of undetectability**, and no deep-learning detector was tested.

## Run it

```bash
pip install -r requirements.txt
jupyter notebook quantum_stego_advanced.ipynb
```

The notebook generates its own synthetic cover image. To use your own, set `COVER_PATH` in Part 4. Always save stego images as PNG. The notebook exports a demo key to `key_package.json`, which is git-ignored. Never commit a real key.

## Limitations

- **Simulated quantum randomness.** AerSimulator is a classical simulator, so the key bits come from a pseudorandom generator inside it. Real quantum randomness needs IBM Quantum hardware through `qiskit-ibm-runtime`. The circuit is the same.
- **Decoding can fail on saturated images.** Clipping at 0 or 255 corrupts DCT coefficients. On six standard test photographs, five decoded correctly and one (about 24% of green pixels near black or white) failed. The encoder's round-trip check reports this instead of silently producing a bad image.
- **Weak steganalysis benchmark.** Only LSB-targeted tests were run (see above).
- **JPEG destroys the payload.** Use PNG.
- **Key distribution is not solved.** The key is exported manually. A real system would use QKD or key exchange.
- **Synthetic cover.** Results on real photographs will differ.
- **Capacity scales with image size.** About 500 characters at 256x256 and about 8,000 at 1024x1024.

## Why these design choices

- **Key from circuits, not a seeded PRNG:** the quantum step is the source of the key bits, not a decoration on a classical generator (subject to the simulator caveat above).
- **GCM:** the authentication tag gives a hard failure on a wrong key.
- **DCT-QIM, not LSB:** embedding in the frequency domain avoids the value-pair artefact that LSB tests detect.
- **Mid-frequency coefficients:** DC changes are visible, and high-frequency terms are fragile.

## Background

The project started as LSB steganography with a quantum key. Reading about steganalysis showed that LSB is easy to flag when a large share of the image is used, so I moved to the frequency domain and then tested the claim instead of assuming it.
