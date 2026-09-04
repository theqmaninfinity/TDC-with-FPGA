# ⚡ Zynq-7020 Sub-Nanosecond Time-to-Digital Converter

A **hybrid coarse–fine Time-to-Digital Converter (TDC)** implemented on a **Xilinx Zynq-7020 SoC** for high-resolution optical **Time-of-Flight (ToF)** measurement and quantum LiDAR applications.

The design combines a synchronous coarse counter with asynchronous fine-time measurement using FPGA carry-chain delay lines. The prototype was implemented and experimentally tested on physical Zynq-7020 hardware, with statistical **code-density calibration** used to characterize the delay-line bins.

---

## ⚡ Overview

The TDC measures the time interval between two asynchronous events — a **START** pulse and a **STOP** pulse.

In other words, it tries to answer the very simple question:

> *"How much time passed?"*

...with a rather unreasonable amount of FPGA hardware. 😄

The measured time is reconstructed as:

```text
T_ToF = N_coarse × T₀ + τ_start − τ_stop
where the coarse clock period is:
T₀ = 5 ns
corresponding to a 200 MHz reference clock.
The coarse counter keeps track of the large-scale timing interval, while the fine TDL determines where each asynchronous event occurred inside that 5 ns clock period.

🧠 The basic idea
The clock tells us which 5 ns window we're in.
The CARRY4 delay line tells us where inside that window the event happened. 🎯

                         START
                           │
                           ▼
                 ┌───────────────────┐
                 │   256-tap CARRY4  │
                 │       TDL         │
                 └─────────┬─────────┘
                           │
                           ▼
                     Fine Timestamp
                           │
                           │
                           ▼
                 ┌───────────────────┐
                 │   Coarse Counter  │
                 │    200 MHz / 5 ns │
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
                 │   256-tap CARRY4  │
                 │       TDL         │
                 └─────────┬─────────┘
                           ▲
                           │
                          STOP

So the architecture is essentially:
coarse timing for the big picture + carry-chain timing for the tiny details.
That's the whole trick.
Everything else is making the FPGA cooperate. 😅

🏗️ Architecture
The system uses a two-channel tapped-delay-line (TDL) architecture with independent START and STOP timing paths.
Each timing channel contains:
64 × Xilinx CARRY4 primitives
256 delay taps
asynchronous pulse capture
synchronous sampling at 200 MHz
thermometer-code decoding
fine timestamp extraction
The overall reconstruction flow is:

START / STOP
     │
     ▼
┌──────────────────────┐
│  CARRY4 Delay Line   │
│      256 taps        │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Thermometer Decoder  │
└──────────┬───────────┘
           │
           ▼
     Fine Timestamp
           │
           ├──────────────┐
           │              │
           ▼              ▼
   Coarse Counter    Calibration LUT
           │              │
           └──────┬───────┘
                  ▼
           Timestamp Reconstruction
                  │
                  ▼
                T_ToF

The coarse stage provides the clock-cycle count, while the TDL provides the sub-clock timing information.

🧩 FPGA Implementation
The time-critical measurement path is implemented in the programmable logic (PL) of the Zynq-7020.
FPGA responsibilities
CARRY4 tapped-delay lines
asynchronous timing capture
200 MHz sampling
thermometer-code decoding
coarse timestamp generation
acquisition state machine
validity and error detection
snapshot registers
hardware histogramming
calibration support
hardware test generation
The START and STOP inputs are intentionally captured through the CARRY4 delay lines rather than conventional synchronizers.
This is important: a normal synchronizer would make the signal safely synchronous...
...and completely ruin the reason we built the TDL in the first place. 😄

🧠 ARM Processing System
The Zynq ARM Cortex-A9 acts as the control and instrumentation layer rather than performing the time-critical capture itself.
It handles:
TDC configuration
acquisition control
register readout
snapshot handshaking
calibration control
lookup-table management
histogram collection
UART communication
data logging
measurement analysis
This creates a clean separation:
FPGA  →  High-speed timing engine
ARM   →  Control, calibration & data handling

🧪 Calibration
FPGA carry-chain delays are fast, but they are not perfectly uniform.
Physics is allowed to be messy; our timing bins are not. 😄
The fine-time bins are therefore characterized using statistical code-density calibration.
For each TDL bin:
w_i = (N_i / N_total) × T₀
where:
N_i = number of calibration hits in bin i
N_total = total number of calibration hits
T₀ = 5 ns reference clock period
The calibrated bin widths are accumulated to construct a delay map:
τ(k) = Σ w_i
       i=0...k-1
The resulting timing map can then be stored as a lookup table (LUT) and used during timestamp reconstruction.
Calibration checks
The calibration workflow includes checks for:
🔹 inactive / zero-count bins
🔹 histogram saturation
🔹 monotonicity
🔹 Differential Nonlinearity (DNL)
🔹 Integral Nonlinearity (INL)
🔹 active TDL range
🔹 calibration quality

🔄 Clock-Domain Crossing
The system contains multiple timing domains because the asynchronous inputs, TDC logic, AXI interface, and ARM processing system do not share identical timing relationships.
Control signals crossing into the 200 MHz TDC domain are synchronized and edge-detected where required.
The START and STOP timing inputs are treated differently.
They remain asynchronous and enter the CARRY4 TDL directly so that the delay chain can preserve their sub-clock timing information.
In short:
Control signals  → synchronize
Timing signals   → measure
Mixing the two would defeat the purpose of the TDC.

🧰 Hardware Validation
The design was implemented and tested on an actual Zynq-7020 FPGA platform, moving beyond simulation to hardware-level validation.
The validation workflow includes:
FPGA timing-capture verification
coarse/fine timestamp reconstruction
asynchronous pulse acquisition
code-density calibration
histogram generation
ARM-based control and readout
optical Time-of-Flight experiments

The project currently represents an FPGA-based experimental TDC prototype rather than a production instrument.

🔬 Applications
The architecture was developed with high-resolution optical timing applications in mind, including:
🚀 Quantum LiDAR
📡 Optical Time-of-Flight measurement
⏱️ Precision timing instrumentation
🔗 Quantum communication experiments
🧩 FPGA-based detector readout
💡 Experimental Quantum Information

🛠️ Technology Stack
Hardware
Xilinx Zynq-7020
FPGA Programmable Logic
ARM Cortex-A9
HDL / FPGA
Verilog RTL
Xilinx CARRY4 primitives
Tapped-Delay-Line TDC
Clock-Domain Crossing
Hardware histogramming
Software
ARM-based control firmware
UART communication
Calibration tools
Data acquisition and analysis

📐 Key Design Parameters
Parameter	Value
FPGA SoC	Xilinx Zynq-7020
Reference Clock	200 MHz
Coarse Clock Period	5 ns
TDL Channels	2
CARRY4 per Channel	64
Taps per Channel	256
Fine Timing Method	Carry-chain TDL
Calibration	Statistical Code Density
Control Processor	ARM Cortex-A9
HDL	Verilog

📊 Project Status
Prototype implemented and experimentally validated on Zynq-7020 hardware.
Current work focuses on:
timing characterization
TDL calibration
nonlinearity analysis
optical Time-of-Flight integration
experimental performance evaluation

This repository documents the architecture and implementation of the experimental prototype.

🎯 Research Context
This project was developed during a summer research internship at the Center of Quantum Science and Technology (CQST) as part of work on precision timing and quantum-photonic instrumentation.
The broader motivation is to explore how inexpensive FPGA hardware can be pushed beyond conventional clock-limited timing measurement and used as a practical platform for high-resolution experimental instrumentation.

👨‍🔬 Project Focus
Precision Timing · FPGA Instrumentation · Quantum LiDAR · Optical ToF · TDL TDC · Zynq-7020 · Verilog · Quantum Information
