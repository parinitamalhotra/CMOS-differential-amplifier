# CMOS-differential-amplifier
## Summary
This project presents the design, theoretical derivation, and SPICE simulation of a **CMOS Differential Amplifier with an Active Current-Mirror Load and Tail Bias**. 

## 1. Resistive Load Differential Amplifier

### A. Circuit Topology & Architecture
The baseline design uses a symmetric NMOS differential pair ($M_1, M_2$) biased by an ideal tail current source ($I_1$) with passive pull-up resistors ($R_1, R_2$).

* **Supply Voltage ($V_{DD}$):** $5.0\text{ V}$
* **Tail Current ($I_1$):** $100\ \mu\text{A}$ ($I_{D1} = I_{D2} = 50\ \mu\text{A}$)
* **Load Resistors ($R_1, R_2$):** $10\text{ k}\Omega$
* **Transistor Sizing ($M_1, M_2$):** $W/L = 10\mu\text{m} / 1\mu\text{m}$
<div align="center">
<img width="1573" height="861" alt="image" src="https://github.com/user-attachments/assets/b53101ba-f8bc-4f15-ba41-d4eba2625b18" />
<p><b>Figure 1:</b> Circuit diagram</p>
</div>

### B. Theoretical Analysis
For a differential input $v_{id} = v_{in1} - v_{in2}$, the differential transconductance gain is given by:

$$A_d = -g_m \cdot R_D$$

Given $\mu_n C_{ox} = 200\ \mu\text{A/V}^2$, $W/L = 10$, and $I_D = 50\ \mu\text{A}$:
* $g_{m1,2} = \sqrt{2 \mu_n C_{ox} (W/L) I_D} = \sqrt{2 \cdot 200\mu \cdot 10 \cdot 50\mu} = 0.447\text{ mS}$
* **Calculated Differential Gain:** $A_d = -0.447\text{ mS} \times 10\text{ k}\Omega \approx \mathbf{-4.47\text{ V/V}}\ \mathbf{(13.0\text{ dB})}$

### C. Simulation Results & Large-Signal Behavior
1. **DC Operating Point (`.op`):**
   * $V(n003) = V(n004) = 4.5\text{ V}$ (Confirming $0.5\text{ V}$ drop across $10\text{ k}\Omega$ resistors at $50\ \mu\text{A}$).
   * Both transistors remain firmly in the active saturation region ($V_{DS} > V_{GS} - V_{TH}$).
2. **Current-Steering Large-Signal DC Sweep (`.dc`):**
   * Sweeping $V_{in1}$ around $V_{ICM} = 0.924\text{ V}$ demonstrates classic current steering: as $V_{in1}$ rises, $I_1$ is steered into $M_1$ ($I_D \rightarrow 100\ \mu\text{A}$), turning $M_2$ off ($I_D \rightarrow 0\ \mu\text{A}$).
<div align="center">
<img width="1600" height="226" alt="image" src="https://github.com/user-attachments/assets/0d79540d-2d55-4595-8485-e69f79a5c82c" />
<p><b>Figure 2:</b> Differential sweep results</p>
</div>
---

## 2. So, Why Active Current-Mirror Load?

While the passive resistive-load differential pair offers good linearity, it suffers from severe limitations in integrated circuit (IC) design:

1. **Integrated Area:** High-value resistors consume massive physical silicon area on-chip compared to compact MOSFETs.
2. **Strict Gain vs. Headroom Trade-Off:** To increase differential gain ($A_d = g_m R_D$), $R_D$ must be increased. However, large $R_D$ causes a large DC drop ($I_D R_D$), pulling $V_{DS}$ down and driving the input transistors out of saturation.
3. **Differential-to-Single-Ended Loss:** With a resistively loaded differential pair, taking the output from only one side gives approximately half the differential output signal for a given differential input. The current-mirror load combines the two branch-current variations at a single output node.

**The Active Current-Mirror Solution:**
Replacing passive resistors with an active PMOS current mirror ($M_1, M_2$) addresses these three limitations:
* It provides a high small-signal output resistance while requiring less DC voltage headroom than a large passive resistor would require for comparable resistance.
* It automatically converts differential currents into a single-ended output voltage without sacrificing gain.

---

## 3. CMOS Differential Amplifier with Active Load

### A. Circuit Topology & Architecture
The primary active-loaded topology consists of an NMOS input pair ($M_3, M_4$), a PMOS active current mirror ($M_1, M_2$), and an NMOS tail current source ($M_5$).

* **Supply Voltage ($V_{DD}$):** $1.8\text{ V}$
* **Differential Input Pair ($M_3, M_4$):** NMOS, $W/L = 10\mu\text{m} / 0.18\mu\text{m}$
* **Active Mirror Load ($M_1, M_2$):** PMOS, $W/L = 20\mu\text{m} / 0.18\mu\text{m}$ (4-terminal `pmos4` with bulk tied to $V_{DD}$)
* **Tail Current Bias ($M_5$):** NMOS, $W/L = 20\mu\text{m} / 0.18\mu\text{m}$ ($V_{bias} = 0.8\text{ V}$)
* **Load Capacitance ($C_L$):** $0.1\text{ pF}$
<div align="center">
<img width="1600" height="931" alt="image" src="https://github.com/user-attachments/assets/03f8fc2f-7217-4f60-b924-fbb0e30d1c11" />
<p><b>Figure 3:</b> Circuit diagram</p>
</div>

### B. Theoretical Small-Signal & High-Frequency Derivations

At $V_{in1} = V_{in2}$:

$$I_{M3} = I_{M4} \approx \frac{I_{tail}}{2}$$

$M_1$ is diode-connected, so its current establishes a $V_{SG1}$. $M_2$ receives the same gate-source voltage, mirroring the $M_1$ current. At the output node:
- $M_4$ pulls current downward.
- $M_2$ supplies current upward.
- The small-signal changes in these currents add constructively at the output node.

1. **Differential Voltage Gain ($A_d$):**
   * $A_d = g_{m3,4} \cdot (r_{o2} \parallel r_{o4})$
   
   * $I_{D5} = \frac{1}{2}\mu C_{ox}\left(\frac{W}{L}\right)_5(V_{bias}-V_{TH})^2$
     
   * $r_{o2} = r_{o4} = \frac{1}{\lambda I_D} = \frac{1}{0.02 \times 55.67\mu\text{A}} \approx 898\text{ k}\Omega \implies R_{out} \approx 449\text{ k}\Omega$
   * **Theoretical Gain:** $A_d \approx 1.11\text{ mS} \times 449\text{ k}\Omega = \mathbf{498\text{ V/V}}\ \mathbf{(53.9\text{ dB})}$

2. **Bandwidth ($f_{-3\text{dB}}$) & Gain-Bandwidth Product (GBW):**

   $$f_{-3\text{dB}} = \frac{1}{2\pi R_{out} C_L} \approx \frac{1}{2\pi (449\text{ k}\Omega) (0.1\text{ pF})} \approx \mathbf{3.54\text{ MHz}}$$

   $$\text{GBW} = A_d \times f_{-3\text{dB}} = \frac{g_{m3,4}}{2\pi C_L} \approx \mathbf{1.77\text{ GHz}}$$

### C. Simulation Directives & Verification
1. **DC Operating Point (`.op`):**
   * Tail current $I_d(M_5) = 111.33\ \mu\text{A}$, splitting approximately equally into $I_{d1-4} = 55.67\ \mu\text{A}$.
   * The PMOS bodies are explicitly tied to \(V_{DD}\) using 4-terminal pmos4 devices; the resulting bulk currents are approximately 0.8 pA.
   * Output DC voltage settled at $V(vout) = 1.00\text{ V}$.
2. **AC Frequency Response Directives (`.ac`):**
   ```spice
   .meas AC max_gain MAX mag(V(vout))
   .meas AC bw TRIG mag(V(vout))=max_gain/sqrt(2) FALL=1
   .meas AC gbw TRIG mag(V(vout))=1 FALL=1
   ```
<div align="center">
<img width="1600" height="801" alt="image" src="https://github.com/user-attachments/assets/6db055e5-94ba-4853-85ae-91187bb3f596" />
<p><b>Figure 4:</b> AC frequency response</p>
</div>

## 4. Results & Performance Summary

The table below compares first-order analytical predictions with LTspice simulation results for the CMOS differential amplifier with an active current-mirror load:

| Performance Metric | Theoretical Formula | Analytical Calculation | LTspice Measurement (`.meas`) | Error / Deviation |
| :--- | :--- | :---: | :---: | :---: |
| **Tail Bias Current ($I_{SS}$)** | $I_{D5} = \frac{1}{2}\mu_n C_{ox}\left(\frac{W}{L}\right)_5(V_{bias}-V_{TH})^2$ | $111.11\ \mu\text{A}$ | **$111.33\ \mu\text{A}$** | $+0.2 percent\%$ |
| **Branch Currents ($I_{D1-4}$)** | $I_{SS} / 2$ | $55.66\ \mu\text{A}$ | **$55.67\ \mu\text{A}$** | $+0.2 percent\%$ |
| **Peak Gain ($A_{d0}$)** | $g_{m3,4} \cdot (r_{o2} \parallel r_{o4})$ | $53.90\text{ dB}$ ($498\text{ V/V}$) | **$54.19\text{ dB}$ ($512.3\text{ V/V}$)** | **$+0.29\text{ dB}$** ($+2.8 percent\%$) |
| **$-3\text{dB}$ Bandwidth ($f_{-3\text{dB}}$)** | $\frac{1}{2\pi (r_{o2} \parallel r_{o4}) C_L}$ | $3.54\text{ MHz}$ | **$3.48\text{ MHz}$** | $-1.6 percent\%$ |
| **Gain-Bandwidth Product (GBW)** | $\frac{g_{m3,4}}{2\pi C_L}$ | $1.77\text{ GHz}$ ($C_L = 0.1\text{ pF}$) | **$1.78\text{ GHz}$** | $+0.5 percent\%$ |

## 5. How to run these simulations
Follow these steps to replicate the simulation results in LTspice:
1. Download and install LTspice.
2. Clone this repository: git clone https://github.com/parinitamalhotra/CMOS-differential-amplifier.git
3. Open schematic/CMOS_differential_amplifier.asc in LTspice.
4. Click Run (F5) to simulate the AC frequency response.
5. Press Ctrl + L to open the SPICE Error Log and inspect the extracted .meas values (max_gain, bw, gbw).
6. Open schematic/Resistive_differential_amplifier.asc to run and observe the baseline resistive load current steering and operating point.   
