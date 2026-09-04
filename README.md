# Zynq-7020 Sub-Nanosecond Time-to-Digital Converter

A **hybrid coarse--fine Time-to-Digital Converter (TDC)** implemented on a **Xilinx Zynq-7020 SoC** for high-resolution optical Time-of-Flight (ToF) measurement and quantum LiDAR applications.

The design combines a synchronous coarse counter with asynchronous fine-time measurement using FPGA carry-chain delay lines. The prototype was implemented and experimentally tested on physical Zynq-7020 hardware, with statistical code-density calibration used to characterize the delay-line bins.

## Overview

The TDC measures the time interval between asynchronous **START** and **STOP** events:

$$
T_{\mathrm{ToF}}
=
N_{\mathrm{coarse}}T_0
+
\tau_{\mathrm{start}}
-
\tau_{\mathrm{stop}}
$$

where the coarse clock period is

$$
T_0 = 5\,\mathrm{ns}
$$

using a **200 MHz reference clock**.

The coarse measurement provides the large-scale timing interval, while the fine measurement determines the position of each event within a single clock period.

## Architecture

The system uses a two-channel tapped-delay-line (TDL) architecture:

```text
                ┌──────────────────────────────┐
START ─────────►│  256-tap CARRY4 TDL          │
                │  Sampling + Thermometer Code │
                └──────────────┬───────────────┘
                               │
                               ▼
                        Fine Timestamp
                               │
                               │
                ┌──────────────┴───────────────┐
                │   Coarse 32-bit Counter      │
                │       200 MHz / 5 ns         │
                └──────────────┬───────────────┘
                               │
STOP ──────────► 256-tap CARRY4 TDL
                               │
                               ▼
                        Fine Timestamp
                               │
                               ▼
                    Coarse/Fine Reconstruction
                               │
                               ▼
                         Calibration LUT
                               │
                               ▼
                    AXI / ARM Readout Layer
```

Each timing channel contains:

* **64 × Xilinx CARRY4 primitives**
* **256 delay taps**
* asynchronous pulse capture
* synchronous sampling at 200 MHz
* thermometer-code decoding
* fine timestamp extraction

The system contains independent **START** and **STOP** delay lines.

## FPGA Implementation

The time-critical path is implemented in the programmable logic of the Zynq-7020.

### FPGA responsibilities

* CARRY4 tapped-delay lines
* asynchronous timing capture
* 200 MHz sampling
* thermometer-code decoding
* coarse timestamp generation
* acquisition state machine
* validity and error detection
* snapshot registers
* hardware histogramming
* calibration support
* hardware test generation

The START and STOP signals are intentionally captured through the delay lines rather than conventional synchronizers, preserving the sub-clock timing information required for fine measurement.

## ARM Processing System

The Zynq ARM Cortex-A9 acts as the control and instrumentation layer rather than performing the time-critical capture itself.

It handles:

* TDC configuration
* acquisition control
* register readout
* snapshot handshaking
* calibration control
* lookup-table management
* histogram collection
* UART communication
* data logging
* measurement analysis

This results in a clear separation between the **FPGA timing engine** and the **software control layer**.

## Calibration

FPGA carry-chain delays are not uniform, so the fine-time bins are calibrated statistically using a **code-density calibration** procedure.

For each delay bin:

$$
w_i =
\frac{N_i}{N_{\mathrm{total}}}T_0
$$

where:

* \(N_i\) is the number of hits in bin \(i\)
* \(N_{\mathrm{total}}\) is the total number of calibration hits
* \(T_0 = 5\,\mathrm{ns}\)

The calibrated bin widths are then accumulated to construct a delay map:

$$
\tau(k)=\sum_{i=0}^{k-1}w_i
$$

The resulting timing map can be stored as a lookup table and used during measurement reconstruction.

The calibration workflow also includes checks for:

* zero or inactive bins
* histogram saturation
* monotonicity
* DNL
* INL
* active-bin range
* calibration quality

## Clock-Domain Crossing

The design contains multiple clock domains because the asynchronous timing inputs, FPGA timing logic, AXI interface, and ARM software layer have different timing relationships.

Control signals crossing into the 200 MHz TDC domain are synchronized and edge-detected where required.

In contrast, the START and STOP inputs remain asynchronous and enter the CARRY4 TDL directly. Synchronizing these signals before the TDL would destroy the fine timing information being measured.

## Hardware Validation

The design was implemented and tested on an **actual Zynq-7020 FPGA platform**, moving beyond simulation to hardware-level validation.

The validation workflow includes:

1. FPGA timing-capture verification
2. coarse/fine timestamp reconstruction
3. asynchronous pulse acquisition
4. code-density calibration
5. histogram generation
6. ARM-based control and readout
7. optical ToF measurement experiments

The project currently represents an **FPGA-based experimental TDC prototype** rather than a production instrument.

## Applications

The architecture was developed with high-resolution optical timing applications in mind, including:

* Quantum LiDAR
* Optical Time-of-Flight measurement
* Precision timing instrumentation
* Quantum communication experiments
* FPGA-based detector readout
* Experimental photonics

## Technology Stack

**Hardware**

* Xilinx Zynq-7020
* FPGA programmable logic
* ARM Cortex-A9

**HDL / FPGA**

* Verilog RTL
* Xilinx CARRY4 primitives
* Tapped-delay-line TDC architecture
* Clock-domain crossing

**Software**

* ARM-based control firmware
* UART communication
* Calibration and data acquisition tools

## Project Status

**Prototype implemented and experimentally validated on Zynq-7020 hardware.**

Current work focuses on timing characterization, calibration, and integration with optical Time-of-Flight experiments.

## Research Context

Developed during a summer research project at the **Center of Quantum Science and Technology (CQST)** as part of work on precision timing and quantum-photonic instrumentation.
