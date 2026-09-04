# Zynq-7020 Sub-Nanosecond Time-to-Digital Converter

A **hybrid coarse–fine Time-to-Digital Converter (TDC)** implemented on a **Xilinx Zynq-7020 SoC** for high-resolution optical **Time-of-Flight (ToF)** measurement and quantum LiDAR applications.

The design combines a synchronous coarse counter with asynchronous fine-time measurement using FPGA carry-chain delay lines. The prototype was implemented and experimentally tested on physical Zynq-7020 hardware, with statistical **code-density calibration** used to characterize the delay-line bins.

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [FPGA Implementation](#fpga-implementation)
4. [ARM Processing System](#arm-processing-system)
5. [Calibration](#calibration)
6. [Clock-Domain Crossing](#clock-domain-crossing)
7. [Hardware Validation](#hardware-validation)
8. [Applications](#applications)
9. [Technology Stack](#technology-stack)
10. [Key Design Parameters](#key-design-parameters)
11. [Project Status](#project-status)
12. [Research Context](#research-context)
13. [Project Focus](#project-focus)

---

## Overview

The TDC measures the time interval between two asynchronous events — a **START** pulse and a **STOP** pulse.

The measured time is reconstructed as:
T_ToF = N_coarse × T₀ + τ_start − τ_stop

text

where the coarse clock period is:
T₀ = 5 ns



corresponding to a 200 MHz reference clock.

The coarse counter tracks the large-scale timing interval, while the fine tapped delay line (TDL) determines where each asynchronous event occurred within that 5 ns clock period.
START
│
▼
┌───────────────────┐
│ 256-tap CARRY4 │
│ TDL │
└─────────┬─────────┘
│
▼
Fine Timestamp
│
▼
┌───────────────────┐
│ Coarse Counter │
│ 200 MHz / 5 ns │
└─────────┬─────────┘
│
▼
Coarse + Fine Data
│
▼
Time Reconstruction
▲
│
┌─────────┴─────────┐
│ 256-tap CARRY4 │
│ TDL │
└─────────┬─────────┘
▲
│
STOP



In essence, the architecture combines a coarse counter for large‑scale timing with a carry‑chain delay line for sub‑clock resolution.

---

## Architecture

The system uses a two‑channel tapped‑delay‑line (TDL) architecture with independent **START** and **STOP** timing paths. Each timing channel contains:

- **64 × Xilinx CARRY4 primitives**
- **256 delay taps**
- Asynchronous pulse capture
- Synchronous sampling at 200 MHz
- Thermometer‑code decoding
- Fine timestamp extraction

---

## FPGA Implementation

The time‑critical measurement path is implemented in the programmable logic (PL) of the Zynq‑7020. The FPGA handles:

- CARRY4 tapped‑delay lines
- Asynchronous timing capture
- 200 MHz sampling
- Thermometer‑code decoding
- Coarse timestamp generation
- Acquisition state machine
- Validity and error detection
- Snapshot registers
- Hardware histogramming
- Calibration support
- Hardware test generation

The START and STOP inputs are intentionally captured through the CARRY4 delay lines rather than conventional synchronizers. This preserves the sub‑clock timing information that would be lost by standard synchronization.

---

## ARM Processing System

The Zynq ARM Cortex‑A9 acts as the control and instrumentation layer, performing:

- TDC configuration
- Acquisition control
- Register readout
- Snapshot handshaking
- Calibration control
- Lookup‑table management
- Histogram collection
- UART communication
- Data logging
- Measurement analysis

This creates a clean separation:

- **FPGA** → High‑speed timing engine
- **ARM** → Control, calibration & data handling

---

## Calibration

FPGA carry‑chain delays are fast but not perfectly uniform. The fine‑time bins are characterized using **statistical code‑density calibration**.

For each TDL bin:
w_i = (N_i / N_total) × T₀

text

where:

- `N_i` = number of calibration hits in bin `i`
- `N_total` = total number of calibration hits
- `T₀` = 5 ns reference clock period

The calibrated bin widths are accumulated to construct a delay map:
τ(k) = Σ w_i for i = 0 to k-1

text

The resulting timing map is stored as a lookup table (LUT) and used during timestamp reconstruction.

**Calibration checks** include:

- Inactive / zero‑count bins
- Histogram saturation
- Monotonicity
- Differential Nonlinearity (DNL)
- Integral Nonlinearity (INL)
- Active TDL range
- Calibration quality

---

## Clock‑Domain Crossing

The system contains multiple timing domains because the asynchronous inputs, TDC logic, AXI interface, and ARM processing system do not share identical timing relationships.

Control signals crossing into the 200 MHz TDC domain are synchronized and edge‑detected where required. In contrast, the START and STOP timing inputs remain asynchronous and enter the CARRY4 TDL directly to preserve sub‑clock timing information.

**Summary:**

- Control signals → synchronize
- Timing signals → measure

Mixing the two would defeat the purpose of the TDC.

---

## Hardware Validation

The design was implemented and tested on an actual Zynq‑7020 FPGA platform. The validation workflow includes:

- FPGA timing‑capture verification
- Coarse/fine timestamp reconstruction
- Asynchronous pulse acquisition
- Code‑density calibration
- Histogram generation
- ARM‑based control and readout
- Optical Time‑of‑Flight experiments

The project currently represents an FPGA‑based experimental TDC prototype rather than a production instrument.

---

## Applications

The architecture was developed with high‑resolution optical timing applications in mind, including:

- Quantum LiDAR
- Optical Time‑of‑Flight measurement
- Precision timing instrumentation
- Quantum communication experiments
- FPGA‑based detector readout
- Experimental Quantum Information

---

## Technology Stack

### Hardware
- Xilinx Zynq‑7020
- FPGA Programmable Logic
- ARM Cortex‑A9

### HDL / FPGA
- Verilog RTL
- Xilinx CARRY4 primitives
- Tapped‑Delay‑Line TDC
- Clock‑Domain Crossing
- Hardware histogramming

### Software
- ARM‑based control firmware
- UART communication
- Calibration tools
- Data acquisition and analysis

---

## Key Design Parameters

| Parameter               | Value                       |
|-------------------------|-----------------------------|
| FPGA SoC                | Xilinx Zynq‑7020            |
| Reference Clock         | 200 MHz                     |
| Coarse Clock Period     | 5 ns                        |
| TDL Channels            | 2                           |
| CARRY4 per Channel      | 64                          |
| Taps per Channel        | 256                         |
| Fine Timing Method      | Carry‑chain TDL             |
| Calibration             | Statistical Code Density    |
| Control Processor       | ARM Cortex‑A9               |
| HDL                     | Verilog                     |

---

## Project Status

Prototype implemented and experimentally validated on Zynq‑7020 hardware.

Current work focuses on:

- Timing characterization
- TDL calibration
- Nonlinearity analysis
- Optical Time‑of‑Flight integration
- Experimental performance evaluation

This repository documents the architecture and implementation of the experimental prototype.

---

## Research Context

This project was developed during a summer research internship at the **Center of Quantum Science and Technology (CQST)** as part of work on precision timing and quantum‑photonic instrumentation.

The broader motivation is to explore how inexpensive FPGA hardware can be pushed beyond conventional clock‑limited timing measurement and used as a practical platform for high‑resolution experimental instrumentation.

---

## Project Focus

Precision Timing · FPGA Instrumentation · Quantum LiDAR · Optical ToF · TDL TDC · Zynq‑7020 · Verilog · Quantum Information
