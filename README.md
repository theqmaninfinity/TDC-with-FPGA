## ⚡ Overview

The TDC measures the time interval between two asynchronous events — a **START** pulse and a **STOP** pulse.

In other words: it tries to answer the very simple question _"how much time passed?"_ with a rather unreasonable amount of FPGA hardware. 😄

The measured time is reconstructed as:

$$
T_{\mathrm{ToF}}
=
N_{\mathrm{coarse}}T_0
+
\tau_{\mathrm{start}}
-
\tau_{\mathrm{stop}}
$$

where the coarse clock period is:

$$
T_0 = 5\,\mathrm{ns}
$$

corresponding to a **200 MHz reference clock**.

The **coarse counter** keeps track of the large-scale time interval, while the **fine TDL stage** resolves where each asynchronous event occurred inside a single 5 ns clock period.

So, conceptually:

```text
            "When did START happen?"
                       │
                       ▼
              ┌────────────────┐
              │   Fine TDL     │
              │ 256 CARRY taps │
              └───────┬────────┘
                      │
                      ▼
                Fine timestamp
                      │
                      +
                      │
              ┌───────▼────────┐
              │ Coarse Counter │
              │    200 MHz     │
              └───────┬────────┘
                      │
                      ▼
               Coarse timestamp
                      │
                      │
                      │
            "When did STOP happen?"
                      │
                      ▼
              ┌────────────────┐
              │   Fine TDL     │
              │ 256 CARRY taps │
              └───────┬────────┘
                      │
                      ▼
                Fine timestamp
                      │
                      ▼
             Δt / ToF reconstruction
