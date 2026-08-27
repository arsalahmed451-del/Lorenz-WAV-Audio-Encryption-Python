# Lorenz Attractor Based Audio Encryption and Decryption in Python

## Project Overview

This project implements audio encryption and decryption using the Lorenz chaotic attractor and XOR operation in Python.

The implementation extends the Lorenz-based encryption approach from RGB image data to digital audio. A Lorenz chaotic system is used to generate a pseudo-random key sequence, which is combined with 16-bit PCM audio samples using XOR.

The current implementation is designed specifically for **16-bit PCM WAV audio files**.

The project demonstrates:

1. Lorenz attractor generation
2. Chaotic X, Y, and Z sequence generation
3. 16-bit chaotic key generation
4. 16-bit PCM WAV audio processing
5. XOR-based audio encryption
6. XOR-based audio decryption
7. Encrypted and decrypted WAV generation
8. Test-vector generation
9. Original-versus-decrypted verification
10. SHA-256 verification

---

## Input Requirements

The current implementation accepts:

| Parameter     | Requirement       |
| ------------- | ----------------- |
| File format   | WAV               |
| Encoding      | PCM               |
| Bit depth     | 16-bit            |
| Channels      | Mono or Stereo    |
| Sample format | Signed 16-bit PCM |

The program validates the audio file before encryption.

### Supported

```text
16-bit PCM WAV → Accepted
```

### Rejected

```text
8-bit WAV  → Rejected
24-bit WAV → Rejected
32-bit WAV → Rejected
MP3        → Rejected
AAC        → Rejected
FLAC       → Rejected
```

If an unsupported file is uploaded, the program stops the encryption process and displays an appropriate error message.

---

## Methodology

The encryption process follows:

```text
Input WAV Audio
       ↓
Audio Validation
       ↓
Read 16-bit PCM Samples
       ↓
Generate Lorenz Chaotic Sequence
       ↓
Generate 16-bit Chaotic Key
       ↓
XOR Audio Samples with Key
       ↓
Encrypted WAV Audio
```

The decryption process uses the same chaotic key sequence:

```text
Encrypted WAV Audio
       ↓
Read Encrypted Samples
       ↓
Generate Same Lorenz Key
       ↓
XOR Encrypted Samples with Key
       ↓
Decrypted WAV Audio
```

Because XOR is reversible:

```text
Plaintext XOR Key = Ciphertext

Ciphertext XOR Key = Plaintext
```

## Key Generation

The Lorenz system initially produces floating-point values.

For compatibility with 16-bit audio samples, the chaotic values are converted into 16-bit integer values.

The conversion is based on:

```text
16-bit value = round(Lorenz value × 256)
```

The resulting value is represented as a 16-bit unsigned integer.

The generated chaotic sequence is then used to create a key stream having the same number of elements as the audio sample stream.

---

## Audio Representation

A 16-bit PCM audio sample contains:

```text
16 bits
```

and has a signed range of:

```text
-32768 to 32767
```

The samples are represented using NumPy `int16`.

For the XOR operation, the same 16-bit data is viewed in its unsigned representation. This allows the complete 16-bit binary pattern to participate in the XOR operation without changing the underlying sample bits.

---

## XOR Encryption

Each audio sample is XORed with the corresponding chaotic key:

```text
Encrypted Sample = Original Sample XOR Chaotic Key
```

For decryption:

```text
Original Sample = Encrypted Sample XOR Chaotic Key
```

The same key sequence must therefore be regenerated during decryption.

---

## Audio Validation

Before encryption, the program checks:

1. Whether the file is a valid WAV file
2. Whether the encoding is PCM
3. Whether the sample width is 16-bit
4. Number of channels
5. Sample rate
6. Number of audio frames

A 24-bit or 32-bit audio file is rejected rather than silently converted.

This prevents unwanted changes in the original audio representation and keeps the encryption experiment consistent.

---

## Repository Structure

```text
Lorenz-Audio-Encryption/
│
├── README.md
├── Lorenz_Audio_Encryption.ipynb
│
├── data/
│   └── test_sample-6s.wav
│
└── output/
    ├── encrypted_audio.wav
    ├── decrypted_audio.wav
    ├── wave form compare.png
    └── test_vectors.csv
```

The generated audio files and test vectors are placed in the `output/` directory.

---

## How to Run

### Google Colab

1. Open `Lorenz_Audio_Encryption.ipynb`.
2. Open the notebook in Google Colab.
3. Run the notebook from the beginning.
4. Upload a **16-bit PCM WAV** audio file when prompted.
5. The program validates the audio format.
6. The Lorenz chaotic sequence is generated.
7. A chaotic key stream is generated.
8. The original audio samples are encrypted using XOR.
9. The encrypted WAV file is generated.
10. The encrypted samples are decrypted using the same key.
11. The decrypted WAV file is generated.
12. The original and decrypted samples are compared.
13. SHA-256 hashes are calculated to verify the result.

---

## Output Files

The `output/` directory contains:

### `encrypted_audio.wav`

The encrypted version of the original WAV audio.

### `decrypted_audio.wav`

The audio obtained after applying the same chaotic key stream to the encrypted samples.

### `lorenz_attractor.png`

A visualization of the Lorenz chaotic trajectory.

### `test_vectors.csv`

A CSV file containing sample-level encryption test data.

---

## Test Vectors

The test-vector file can contain information such as:

```text
Sample Index
Original Sample
Lorenz X
Lorenz Y
Lorenz Z
Chaotic Key
Encrypted Sample
Decrypted Sample
Verification Result
```

These test vectors provide a transparent way to inspect the encryption and decryption process.

---

## Verification

The decrypted audio is verified against the original audio samples.

The following checks are performed:

| Verification              | Expected Result |
| ------------------------- | --------------- |
| Encryption completed      | PASS            |
| Decryption completed      | PASS            |
| Sample count preserved    | PASS            |
| Original = Decrypted      | PASS            |
| Maximum sample difference | 0               |
| SHA-256 verification      | PASS            |

Successful decryption should produce exactly the same audio sample data as the original input.

---

