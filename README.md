# Digital Modulator (ASK, BPSK, FSK) in Verilog

## Overview

This project implements a simple **Digital Modulator** in Verilog that supports three common digital modulation schemes:

* **Amplitude Shift Keying (ASK)**
* **Binary Phase Shift Keying (BPSK)**
* **Frequency Shift Keying (FSK)**

The design uses precomputed sine-wave lookup tables to generate carrier signals and modulates them based on the input data stream.

---

## Features

* 8-bit sine-wave carrier generation
* 16-sample lookup table per cycle
* Supports three modulation techniques
* Synchronous operation using a system clock
* Active-high asynchronous reset

---

## Module Interface

```verilog
module digital_modulator(
    input wire clk,
    input wire rst,
    input wire [1:0] mod_sel,
    input wire data_in,
    output reg [7:0] mod_out
);
```

### Inputs

| Signal    | Width | Description                |
| --------- | ----- | -------------------------- |
| `clk`     | 1     | System clock               |
| `rst`     | 1     | Active-high reset          |
| `mod_sel` | 2     | Modulation scheme selector |
| `data_in` | 1     | Binary data input          |

### Output

| Signal    | Width | Description               |
| --------- | ----- | ------------------------- |
| `mod_out` | 8     | Modulated output waveform |

---

## Modulation Selection

| `mod_sel` | Modulation Type |
| --------- | --------------- |
| `2'b00`   | ASK             |
| `2'b01`   | BPSK            |
| `2'b10`   | FSK             |
| `2'b11`   | Undefined       |

---

## Carrier Generation

Two carrier signals are generated:

### Carrier at Frequency `fc`

```verilog
carrier_fc
```

### Carrier at Frequency `2fc`

```verilog
carrier_2fc
```

Both carriers are represented using 8-bit unsigned samples.

A 4-bit phase accumulator advances through 16 discrete samples:

```verilog
phase <= phase + 1;
```

---

## Sine Wave Lookup Table Formula

The carrier samples are generated using the following equation:

[
\text{Sample}(n) = 128 + \text{round}\left(128 \times \sin\left(\frac{2\pi n}{16}\right)\right)
]

where:

* (n = 0,1,2,\dots,15)
* 128 is the DC offset
* 128 is the amplitude scaling factor
* 16 is the number of samples per cycle

Equivalent code:

```text
sample[n] = 128 + round(128 * sin(n * 2π / 16))
```

### Generated Values

| n  | Value |
| -- | ----- |
| 0  | 128   |
| 1  | 176   |
| 2  | 218   |
| 3  | 246   |
| 4  | 255   |
| 5  | 246   |
| 6  | 218   |
| 7  | 176   |
| 8  | 128   |
| 9  | 79    |
| 10 | 37    |
| 11 | 9     |
| 12 | 0     |
| 13 | 9     |
| 14 | 37    |
| 15 | 79    |

These values form one complete sinusoidal cycle.

---

## Modulation Schemes

### 1. Amplitude Shift Keying (ASK)

For ASK:

```verilog
mod_out = data_in ? carrier_fc : 8'd128;
```

* Data = `1` → Carrier transmitted
* Data = `0` → Carrier suppressed (DC level)

Mathematically:

[
s(t) =
\begin{cases}
A\sin(2\pi f_ct), & b=1\
0, & b=0
\end{cases}
]

---

### 2. Binary Phase Shift Keying (BPSK)

For BPSK:

```verilog
mod_out = data_in ? carrier_fc : ~carrier_fc;
```

* Data = `1` → Normal carrier
* Data = `0` → Phase-inverted carrier (180° shift)

Mathematically:

[
s(t)=A\sin(2\pi f_ct+\phi)
]

where

[
\phi=
\begin{cases}
0^\circ, & b=1\
180^\circ, & b=0
\end{cases}
]

---

### 3. Frequency Shift Keying (FSK)

For FSK:

```verilog
mod_out = data_in ? carrier_2fc : carrier_fc;
```

* Data = `0` → Frequency = `fc`
* Data = `1` → Frequency = `2fc`

Mathematically:

[
s(t)=A\sin(2\pi f_it)
]

where

[
f_i=
\begin{cases}
f_c, & b=0\
2f_c, & b=1
\end{cases}
]

---

## Internal Architecture

```text
          +----------------+
data_in -->|                |
           | Modulation     |----> mod_out
mod_sel -->| Selection Logic|
           |                |
           +--------+-------+
                    |
                    |
      +-------------+-------------+
      |                           |
+-----------+             +-----------+
| carrier_fc|             |carrier_2fc|
+-----------+             +-----------+
      ^                           ^
      |                           |
      +-----------+---------------+
                  |
             Phase Counter
```

---

## Simulation Notes

* Reset initializes the phase counter and carrier values.
* Carrier waveforms repeat every 16 clock cycles.
* FSK uses a second carrier table representing twice the carrier frequency.
* Output samples are unsigned 8-bit values ranging from 0 to 255.

---

## Example Applications

* FPGA-based digital communication experiments
* Modulation and demodulation coursework
* DSP and communication systems labs
* SDR (Software Defined Radio) prototyping
* Educational demonstrations of ASK, BPSK, and FSK

---

## Future Improvements

* Parameterized lookup-table size
* DDS/NCO-based carrier generation
* QPSK support
* QAM support
* Configurable carrier frequencies
* FIR filtering for smoother waveform generation

---

## License

This project is released under the MIT License.
