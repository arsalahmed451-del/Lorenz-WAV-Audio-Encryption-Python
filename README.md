# Lorenz Attractor Based Audio Encryption and Decryption

## Project Overview

This project implements a chaotic audio encryption and decryption system in Python using the **Lorenz attractor** and the **XOR operation**.

The system generates a chaotic sequence using the three Lorenz state variables **X, Y, and Z**. These floating-point chaotic values are converted into 16-bit fixed-point representations using the **Q8.8 fixed-point format**. The three resulting 16-bit values are then combined using XOR to generate a 16-bit chaotic key stream.

Each 16-bit PCM audio sample is XORed with its corresponding 16-bit chaotic key to produce the encrypted audio. The same Lorenz parameters and initial conditions are used during decryption, allowing the same chaotic key stream to be regenerated and used to recover the original audio.

The current implementation is specifically designed for **16-bit uncompressed PCM WAV audio**.

---

## Encryption Method

The complete encryption process is:

```text
16-bit PCM WAV Audio
          ↓
     Audio Validation
          ↓
     Audio Samples
          ↓
     Lorenz Attractor
          ↓
       X, Y, Z
          ↓
     Q8.8 Conversion
          ↓
   X16    Y16    Z16
      \     |     /
       \    |    /
        XOR Operations
             ↓
    16-bit Chaotic Key
             ↓
     Audio Sample XOR Key
             ↓
      Encrypted Audio
```

### Key Generation

The final chaotic key is generated as:

```text
Final Key = X16 XOR Y16 XOR Z16
```

where:

* `X16` = 16-bit Q8.8 representation of Lorenz X
* `Y16` = 16-bit Q8.8 representation of Lorenz Y
* `Z16` = 16-bit Q8.8 representation of Lorenz Z

The generated key stream contains one 16-bit key for each audio sample.

---

## Decryption Method

The decryption process uses the same chaotic key stream:

```text
Encrypted Sample XOR Same Key = Original Sample
```

The XOR property used is:

```text
A XOR B XOR B = A
```

Therefore:

```text
Original Sample XOR Key = Encrypted Sample

Encrypted Sample XOR Key = Original Sample
```

The decrypted samples are then written back into a WAV file using the original number of channels and sample rate.

---

## Lorenz Chaotic System

The Lorenz system is defined by the following differential equations:

```text
dx/dt = σ(y - x)

dy/dt = x(ρ - z) - y

dz/dt = xy - βz
```

The implementation uses the following standard Lorenz parameters:

```text
σ = 10.0

ρ = 28.0

β = 8/3
```

The initial conditions are:

```text
X0 = 1.0

Y0 = 1.0

Z0 = 1.0
```

The numerical integration time step is:

```text
dt = 0.01
```

Euler numerical integration is used to calculate the next Lorenz state:

```text
x_new = x + dx × dt

y_new = y + dy × dt

z_new = z + dz × dt
```

---

## Warm-Up Process

The implementation discards the first **1000 Lorenz iterations** before generating the key stream.

```text
WARMUP_STEPS = 1000
```

The purpose of the warm-up period is to discard the initial transient portion of the trajectory and begin key generation after the system has evolved from its initial state.

The process is:

```text
Initial Conditions
        ↓
1000 Lorenz Iterations
        ↓
Discard Initial Values
        ↓
Generate Chaotic Sequence
```

---

## Q8.8 Fixed-Point Conversion

The Lorenz attractor produces floating-point values.

To generate a 16-bit representation, the system uses the **Q8.8 fixed-point format**.

Q8.8 contains:

```text
8 bits → Integer portion

8 bits → Fractional portion

Total → 16 bits
```

The scaling factor is:

```text
2^8 = 256
```

Therefore:

```text
Scaled Value = round(Lorenz Value × 256)
```

The resulting integer is stored as a 16-bit unsigned representation.

The same conversion is applied independently to:

```text
X → X16

Y → Y16

Z → Z16
```

---

## Chaotic Key Generation

After converting the Lorenz variables to 16-bit values:

```text
X → X16
Y → Y16
Z → Z16
```

the key is generated using XOR:

```text
XY Key = X16 XOR Y16

Final Key = XY Key XOR Z16
```

which is equivalent to:

```text
Final Key = X16 XOR Y16 XOR Z16
```

The final result is a **16-bit chaotic key stream**.

Each generated key corresponds to one audio sample.

---

## Audio Input Requirements

The current implementation accepts only:

| Parameter             | Requirement       |
| --------------------- | ----------------- |
| File format           | WAV               |
| Encoding              | Uncompressed PCM  |
| Bit depth             | 16-bit            |
| Channels              | Mono or Stereo    |
| Sample representation | Signed 16-bit PCM |

### Supported

```text
16-bit PCM WAV
```

### Rejected

```text
8-bit WAV
24-bit WAV
32-bit WAV
MP3
AAC
FLAC
OGG
Other unsupported formats
```

The program validates the file **before encryption begins**.

If an unsupported file is uploaded, the program raises an error and stops processing.

For example, a 24-bit WAV file produces an error indicating that the implementation requires 16-bit PCM WAV audio.

---

## Why 16-bit PCM WAV?

The current implementation deliberately uses 16-bit PCM WAV because each audio sample can be represented directly using a 16-bit value.

This creates a simple and controlled relationship:

```text
16-bit Audio Sample
        +
16-bit Chaotic Key
        ↓
      XOR
        ↓
16-bit Encrypted Sample
```

This also avoids complications associated with compressed audio formats and different sample representations.

---

## Audio Processing

The Python `wave` module is used to read and write WAV files.

The raw PCM audio bytes are converted into NumPy signed 16-bit integers:

```text
WAV Bytes
   ↓
NumPy int16
   ↓
Audio Samples
```

For encryption, the signed `int16` samples are viewed as `uint16` values.

This preserves the exact 16-bit binary representation of each sample while allowing the XOR operation to operate on all 16 bits.

After encryption, the resulting 16-bit data is viewed again as signed `int16` and written to the output WAV file.

---

# Program Features

The implementation includes the following features:

### 1. Audio Validation

Checks:

* WAV extension
* Valid WAV structure
* PCM encoding
* 16-bit sample width
* Mono/stereo channels

### 2. Lorenz Sequence Generation

Generates:

```text
X sequence
Y sequence
Z sequence
```

after a 1000-step warm-up.

### 3. Q8.8 Conversion

Converts the Lorenz floating-point values into 16-bit representations.

### 4. Chaotic Key Generation

Generates:

```text
X16 XOR Y16 XOR Z16
```

### 5. Audio Encryption

Performs 16-bit XOR encryption.

### 6. Audio Decryption

Uses the same chaotic key to recover the original audio.

### 7. Key Stream Analysis

Calculates:

* Total keys
* Unique keys
* Minimum key
* Maximum key
* Zero keys
* Repeated keys

### 8. Encryption Change Analysis

Calculates:

* Total samples
* Changed samples
* Unchanged samples
* Percentage of changed samples

### 9. Correlation Analysis

Calculates the correlation between:

```text
Original Audio
       vs
Encrypted Audio
```

### 10. SHA-256 Verification

Calculates SHA-256 hashes for:

```text
Original Samples
Decrypted Samples
```

A successful decryption should produce identical hashes.

### 11. Sample-by-Sample Verification

The program directly compares:

```text
Original Samples
       vs
Decrypted Samples
```

using NumPy.

### 12. Waveform Comparison

Generates a waveform comparison showing:

```text
Original Audio Waveform

Encrypted Audio Waveform

Decrypted Audio Waveform
```

### 13. Audio Playback

The notebook provides playback for:

* Original audio
* Encrypted audio
* Decrypted audio

### 14. Test Vector Generation

The program generates detailed CSV files containing Lorenz values, keys, original samples, encrypted samples, and decrypted samples.

---

# Verification and Results

The implementation performs multiple verification checks.

## Sample-by-Sample Verification

The program checks:

```text
Original Samples == Decrypted Samples
```

Expected result:

```text
True
```

---

## SHA-256 Verification

The SHA-256 hash of the original sample data is compared with the SHA-256 hash of the decrypted sample data.

Expected result:

```text
SHA-256 match: True
```

This provides an additional verification that the decrypted sample data is identical to the original sample data.

---

## Encryption Change Analysis

The program counts how many samples changed after encryption.

It reports:

```text
Total samples
Changed samples
Unchanged samples
Percentage changed
```

This provides a basic measurement of how extensively the encryption operation changes the sample values.

---

## Correlation Analysis

The program calculates the Pearson correlation coefficient between the original and encrypted audio sample sequences.

```text
Correlation =
corr(Original Samples, Encrypted Samples)
```

A lower correlation indicates that the encrypted signal has a weaker linear relationship with the original signal.

Correlation is used here as a basic statistical observation and is not by itself a proof of cryptographic security.

---

# Generated Files

The program generates the following files:

```text
encrypted_audio.wav
decrypted_audio.wav
audio_test_vectors.csv
lorenz_key_stream.csv
waveform_comparison.png
```

## `encrypted_audio.wav`

Contains the encrypted audio samples.

## `decrypted_audio.wav`

Contains the audio recovered after decryption.

## `audio_test_vectors.csv`

Contains sample-level encryption and decryption information.

The CSV includes:

```text
Sample_Index
X
Y
Z
X16_Q8.8
Y16_Q8.8
Z16_Q8.8
Final_Key
Original_Sample
Encrypted_Sample
Decrypted_Sample
```

## `lorenz_key_stream.csv`

Contains the complete Lorenz sequence and generated key stream.

It includes:

```text
Index
X
Y
Z
X16_Q8.8
Y16_Q8.8
Z16_Q8.8
Final_Key
```

## `waveform_comparison.png`

Contains three waveform plots:

```text
Original
Encrypted
Decrypted
```

---

# Test Vectors

The notebook displays the first five test vectors for easy inspection.

For each sample, the following information is displayed:

```text
Original Sample
X16
Y16
Z16
Final Key
Encrypted Sample
Decrypted Sample
```

A typical encryption relationship is:

```text
Original Sample XOR Final Key
          =
Encrypted Sample
```

and:

```text
Encrypted Sample XOR Final Key
          =
Decrypted Sample
```

The complete test-vector dataset is saved to:

```text
audio_test_vectors.csv
```

---

# Repository Structure

```text
Lorenz-Audio-Encryption/
│
├── README.md
├── Lorenz_Audio_Encryption.ipynb
├── requirements.txt
│
├── input/
│   └── README.md
│
└── output/
    ├── README.md
    ├── encrypted_audio.wav
    ├── decrypted_audio.wav
    ├── audio_test_vectors.csv
    ├── lorenz_key_stream.csv
    └── waveform_comparison.png
```

The generated output files are created after executing the notebook.

---

# How to Run

## Google Colab

### Step 1

Open:

```text
Lorenz_Audio_Encryption.ipynb
```

in Google Colab.

### Step 2

Run the notebook from the beginning.

### Step 3

When prompted, upload **one 16-bit PCM WAV audio file**.

### Step 4

The program validates:

```text
File format
PCM encoding
Bit depth
Channels
Sample rate
Number of frames
```

### Step 5

The Lorenz attractor generates the chaotic X, Y, and Z sequences.

### Step 6

The X, Y, and Z sequences are converted to 16-bit Q8.8 values.

### Step 7

The final chaotic key stream is generated:

```text
X16 XOR Y16 XOR Z16
```

### Step 8

The audio is encrypted:

```text
Audio Sample XOR Chaotic Key
```

### Step 9

The encrypted audio is saved as:

```text
encrypted_audio.wav
```

### Step 10

The encrypted audio is decrypted using the same key stream.

### Step 11

The decrypted audio is saved as:

```text
decrypted_audio.wav
```

### Step 12

The program performs:

* Sample-by-sample verification
* SHA-256 verification
* Correlation analysis
* Encryption change analysis
* Key-stream analysis
* Waveform comparison

### Step 13

The generated output files are downloaded automatically.

---

# Example Processing Flow

```text
                INPUT
                  │
                  ▼
          16-bit PCM WAV
                  │
                  ▼
         Validate Audio File
                  │
                  ▼
           Read PCM Samples
                  │
                  ▼
          Lorenz Attractor
                  │
          ┌───────┼───────┐
          ▼       ▼       ▼
          X       Y       Z
          │       │       │
          ▼       ▼       ▼
         X16     Y16     Z16
          │       │       │
          └───────┼───────┘
                  ▼
            XOR Combination
                  │
                  ▼
         16-bit Chaotic Key
                  │
                  ▼
        Sample XOR Key Stream
                  │
                  ▼
          Encrypted WAV
                  │
                  ▼
        Same Key XOR Again
                  │
                  ▼
          Decrypted WAV
                  │
                  ▼
             Verification
```

---

# Configuration

The main configurable parameters are:

```text
SIGMA = 10.0
RHO = 28.0
BETA = 8.0 / 3.0

X0 = 1.0
Y0 = 1.0
Z0 = 1.0

DT = 0.01

WARMUP_STEPS = 1000

FIXED_POINT_SCALE = 256
```

Display settings include:

```text
LORENZ_DISPLAY_STEPS = 5

TEST_VECTOR_DISPLAY = 5

WAVEFORM_SAMPLES = 5000
```

These display settings control the amount of information shown in the notebook and do not change the encryption method.

---




