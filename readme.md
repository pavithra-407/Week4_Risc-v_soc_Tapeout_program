# Week 4 – CMOS Circuit Design & SPICE Simulation (Sky130-PDK)

Day 1: Introduction to Circuit Design & SPICE Simulations</summary>
<br>

## Circuit Design and SPICE Simulation

**Circuit Design** is the process of creating the **functionality of an electronic system** — it defines what the circuit should do and how it will accomplish this.

![Circuit Design](https://github.com/user-attachments/assets/bd30bae7-41e9-4ea9-8848-864318cbe27f)

The type of transistors used (**NMOS** and **PMOS**), their sizes (**W/L ratio**), and the way they’re connected all depend on the **required functionality**.

**SPICE simulation** is used to study how a circuit performs **before it is physically built**.  
It allows designers to see the **circuit’s output characteristics—such as voltage, current, and delay**—by displaying them as **waveforms on a graph**.

![SPICE Waveforms](https://github.com/user-attachments/assets/65ca0258-5a49-43a4-9387-8eb8c0878a16)

### Why We Need SPICE

**SPICE (Simulation Program with Integrated Circuit Emphasis)** is essential because it allows us to find the **real, accurate delay values** of circuits *before manufacturing*.

![SPICE Example](https://github.com/user-attachments/assets/9108b8a9-dbb9-4412-a726-ab7fa346176a)

Consider a circuit with several **buffers**.  
Each buffer is built using different **NMOS** and **PMOS** transistors and drives a different **load**.  
Because of this, each buffer type (e.g., **Type 1** and **Type 2**) has **different delay values** — these depend on:
- Circuit design  
- Transistor sizes  
- Loading conditions  

SPICE simulations are used to measure these delays by analyzing how the circuit responds under various:
- **Input slews** (signal transition times)  
- **Output loads** (capacitive loads)  

The resulting **delay values** are then stored in **lookup tables**.

| Lookup Table 1 | Lookup Table 2 |
|-----------|-----------|
| ![LUT1](https://github.com/user-attachments/assets/d52f1ff9-a7cd-4242-a99b-b9ae91f48406) | ![LUT2](https://github.com/user-attachments/assets/12e13c45-cab0-4f9d-8b6a-f12a9266de76) |

Below is a simplified example of a delay lookup table for an inverter (`INVX1`):

```liberty
cell (INVX1) {
    pin (A) {
        direction : input;
        capacitance : 0.005;
    }

    pin (Y) {
        direction : output;
        function : "(!A)";
        max_capacitance : 0.1;

        timing() {
            related_pin : "A";
            timing_sense : negative_unate;

            cell_rise (delay_template_2d) {
                index_1 ("10, 30, 50, 70, 90");         // Input Slew (ps)
                index_2 ("10, 30, 50, 70, 90, 110");     // Output Load (fF)
                values (
                    "0.05, 0.07, 0.09, 0.11, 0.13, 0.15",
                    "0.06, 0.08, 0.10, 0.12, 0.14, 0.16",
                    "0.07, 0.09, 0.11, 0.13, 0.15, 0.17",
                    "0.08, 0.10, 0.12, 0.14, 0.16, 0.18",
                    "0.09, 0.11, 0.13, 0.15, 0.17, 0.19"
                );
            }
        }
    }
}



Delay Values - CBUF1
Delay Values - CBUF2



Input Slew = 40 psOutput Load = 60 fFDelay Value = However, 60 fF is not explicitly listed in the LUT  The closest columns are:  x₉ - 50 fF  x₁₀ - 70 fF To estimate the delay at 60 fF, we perform linear interpolation between x₉ and x₁₀:  Delay_60fF = x9 + [(60fF - 50fF) / (70fF - 50fF)] * (x10 - x9)
Input Slew = 60 psOutput Load = 50 fF Fortunately, 50 fF is explicitly listed in the LUT So we can directly pick the delay value from the table 60 ps(slew) - 50 fF(load) - y₁₅(delay)  Delay_CBUF2 = y_15
```

Interpolation and Extrapolation in Delay Calculation
In real-world Static Timing Analysis (STA), it’s uncommon for the required input slew and output load to match the exact entries in the cell delay lookup tables (LUTs). To estimate delay values accurately, interpolation or extrapolation techniques are applied.
Interpolation

Used when: The required values fall between two known points in the LUT.  
Purpose: To estimate delay or transition values within the characterized range.  
Method: STA tools perform linear or bilinear interpolation between neighboring entries based on:
Input slew  
Output load


Accuracy: High — since it stays within the characterized (measured) data range.

$$Delay_{est} = D_1 + \frac{(X - X_1)}{(X_2 - X_1)} \times (D_2 - D_1)$$
Where:
```
( D_1, D_2 ): Known delay values from the LUT  
( X_1, X_2 ): Corresponding known input slew or output load points  
( X ): The required (intermediate) value for which delay is being estimated
```
Extrapolation

Used when: The required slew or load falls outside the bounds of the LUT (e.g., >110 fF or <10 fF).  
Purpose: To estimate delay values beyond the characterized range.  
Method: Extends the slope of the nearest known data points to estimate new values.  
Accuracy: Lower — results can deviate from SPICE simulation or real silicon.

$$Delay_{ext} = D_1 + \frac{(X - X_1)}{(X_2 - X_1)} \times (D_2 - D_1)$$
Where:
```
( D_1, D_2 ): Delay values from the nearest two known LUT entries  
( X_1, X_2 ): Corresponding input slew or output load points  
( X ): The out-of-range value for which the delay is being estimated
```

Note:Extrapolated delay values should be used only when necessary.Designers should aim to keep analysis within the characterized LUT range for more reliable and consistent timing results.

Key Questions
By completing this week’s tasks, you will address:

Where does the delay of a cell actually come from?  
Delays stem from transistor characteristics (Id, Vt), load capacitance, and input slew.


Are the delay models accurate?  
SPICE provides accurate data; STA models are reliable if derived from well-characterized SPICE simulations.


How do we verify that what STA is reporting is correct?  
Compare STA delays with SPICE-simulated delays under identical conditions.



NMOS and PMOS Transistors



NMOS
PMOS





NMOS stands for N-channel Metal Oxide Semiconductor:

A 4-terminal device: Source (S), Drain (D), Gate (G), Substrate/Body (B).
Consists of P-substrate, gate oxide (SiO2), n+ diffusion regions, and polysilicon gate.
In p-type substrates, majority carriers are holes, and minority carriers are electrons.

Case I: VGS = 0

Source, drain, and body are at ground.
Substrate-to-source and substrate-to-drain form reverse-biased PN junction diodes.
Resistance is effectively infinite (no current flows, no conductive channel).
Summary: NMOS is OFF, acting as an open circuit.

Case II: Small Positive VGS

Positive gate voltage creates a capacitor with the oxide layer as dielectric.
Repels holes, leaving immobile negative ions and forming a depletion region.
Electrons are attracted but insufficient to form a conduction channel.

Installation of NGSPICE
Download the tarball from SourceForge:
```
tar -zxvf ngspice-44.tar.gz
cd ngspice-44
mkdir release
cd release
../configure --with-x --with-readline=yes --disable-debug
make
sudo make install
ngspice
```

Workshop Collaterals
```
Clone the Sky130 Circuit Design Workshop repository:
git clone https://github.com/kunalg123/sky130CircuitDesignWorkshop.git
cd sky130CircuitDesignWorkshop
ls
```

CMOS Design Flow: From Transistor Behavior to STA
Mind-Map Structure

Level 1: Transistor Behavior (Device Physics)↳ Foundation: Individual NMOS/PMOS operation  

I–V Characteristics (Id vs. Vds / Vgs)
Linear & saturation regions
Threshold voltage (Vt) extraction
Velocity saturation in short-channel devices


Device Parameters
W/L ratio → Drive strength
Tox, doping, channel length → Vt variation
Supply voltage (VDD) → Switching speed



Level 2: CMOS Inverter Behavior (Circuit Level)↳ Combination of NMOS + PMOS devices  

VTC Curve (Vout vs. Vin)
Defines switching threshold (Vm)
Basis for noise margins (NMH, NML)


Transient Response
Rise/fall delay
Load capacitance and drive current impact


Noise Margin Analysis
Determines logical robustness



Level 3: Timing Behavior (Digital Logic Level)↳ Behavior of gates in real timing conditions  

Propagation Delays (tPHL, tPLH)
Used in timing libraries (.lib)


Variation Effects
Delay, threshold, and noise margins under PVT corners


Power Supply Variation Impact
Dynamic delay & power trade-off



Level 4: Static Timing Analysis (System Level)↳ How EDA tools use transistor data to verify chip timing  

From SPICE → .lib Models
Extracted delay tables
Slew, load, and drive strength dependencies


STA Concepts
Arrival time, required time, slack
Setup/hold margins derived from transistor behavior


Goal: Accurate timing closure under variation and process constraints

Transistor Physics
      ↓
Device I–V Characteristics
      ↓
CMOS Inverter (VTC + Transient)
      ↓
Delay, Noise Margin, Variation
      ↓
Timing Models (.lib)
      ↓
Static Timing Analysis (STA)

Connection to Static Timing Analysis (STA)
STA tools rely on SPICE-generated lookup tables (.lib) to estimate delays without using simple formulas, ensuring timing accuracy. This week’s experiments bridge transistor-level behavior to STA by:

Generating delay data through SPICE simulations.
Understanding how Id, Vt, and load capacitance affect delays.
Verifying STA models against SPICE results.

Importance of Correct SPICE Setup
Incorrect SPICE setups (e.g., wrong transistor models, unrealistic loads, or inaccurate slew rates) lead to inaccurate STA results, deviating from real silicon behavior. Proper setup ensures reliable timing analysis.
References

NGSPICE Download
Sky130 Circuit Design Workshop
Sky130 PDK Documentation (available in the workshop repository)



Week 4 Task – CMOS Circuit Design (Sky130-PDK)
Why This Task Matters
This task connects transistor-level properties (device physics, sizing, variation) to the timing behavior analyzed by STA. By working through CMOS design and SPICE simulations using the Sky130 PDK, you will:

Gain hands-on experience with transistor and circuit characterization.
Understand how SPICE data populates .lib files for STA.
Develop intuition for delays, noise margins, and variation impacts.

Task Components
The task involves a series of SPICE simulation activities to explore CMOS circuit behavior, mirroring the Sky130 Circuit Design Workshop. The components are:

MOSFET Behavior & Id vs. Vds Characteristics

Simulate an NMOS transistor, sweeping Vds for different Vgs to observe linear and saturation regions.
Plot Id vs. Vds curves.


Threshold Voltage Extraction & Velocity Saturation

Sweep Vgs vs. Id and extract Vt (e.g., by linear extrapolation).
Observe velocity saturation effects in short-channel devices.


CMOS Inverter: Voltage Transfer Characteristic (VTC)

Build a CMOS inverter (NMOS + PMOS).
Sweep input (Vin) and plot Vout vs. Vin.
Identify the switching threshold (Vm, where Vin = Vout).


Transient Behavior: Rise/Fall Delays

Apply a pulse input to the inverter.
Extract rise and fall propagation delays (times at Vdd/2 crossing).


Noise Margin & Robustness Analysis

From the VTC plot, determine VIL, VIH, VOL, VOH.
Compute NML = VIL - VOL and NMH = VOH - VIH.


Power-Supply and Device Variation Studies

Vary supply voltage (Vdd) and re-plot VTCs to observe Vm shifts.
Modify transistor sizing (e.g., W/L of PMOS or NMOS) and analyze effects on VTC, noise margins, and delays.



Deliverables
Document your work in a markdown report (or Jupyter notebook) following the style of the Sky130 workshop repository. Include:

Introduction/Background

Explain the purpose of each experiment (e.g., why study Id vs. Vds, VTC, delays, variations).


SPICE Netlists/Code

Provide complete SPICE netlists for each experiment (e.g., Id vs. Vds, VTC, transient, variations).
Use Sky130 PDK models (e.g., sky130_fd_pr__nfet_01v8, sky130_fd_pr__pfet_01v8).


Plots & Figures

Include annotated graphs:
Id vs. Vds (mark linear/saturation regions).
Id vs. Vgs (mark Vt).
VTC (mark Vm, VIL, VIH, VOL, VOH).
Transient waveforms (mark rise/fall delays).
VTC under variation (highlight shifts).


Use NGSPICE or Python (e.g., Matplotlib) for plotting.


Tabulated Results

Summarize:
Extracted Vt.
Rise (tPLH) and fall (tPHL) delays.
Noise margins (NML, NMH).
Effects of Vdd or W/L variations on Vm, delays, and noise margins.





Example Table



Parameter
Value (Nominal)
Value (Vdd=1.6V)
Value (W/L Doubled)



Threshold Voltage (Vt)
0.45V
N/A
0.44V


Rise Delay (tPLH)
50ps
55ps
45ps


Fall Delay (tPHL)
48ps
53ps
43ps


Noise Margin Low (NML)
0.7V
0.65V
0.72V


Noise Margin High (NMH)
0.68V
0.62V
0.70V


Getting Started

Install NGSPICE (see Installation section).
```
Clone the Sky130 workshop repository and access PDK models.
Create and simulate SPICE netlists for each task component.
Generate plots, analyze results, and document findings in a markdown report.
Compare results with STA concepts (e.g., .lib delay tables).
```
Happy simulating!


Week 4 – CMOS Circuit Design & SPICE Simulation (Sky130-PDK)
Introduction to Circuit Design & SPICE Simulations
Circuit Design and SPICE Simulation
Circuit Design involves defining the functionality of an electronic system, specifying what the circuit does and how it achieves its purpose. The choice of transistors (NMOS and PMOS), their sizes (W/L ratio), and their interconnections are determined by the required functionality.
SPICE Simulation (Simulation Program with Integrated Circuit Emphasis) is a critical tool for analyzing circuit performance before physical fabrication. It provides insights into circuit characteristics—such as voltage, current, and delay—displayed as waveforms, enabling designers to verify and optimize designs.
Why SPICE is Essential
SPICE simulations allow designers to measure accurate delay values before manufacturing, crucial for timing-critical circuits. For example, in a circuit with multiple buffers, each using different NMOS and PMOS transistors and driving varying loads, SPICE evaluates delays based on:

Circuit design: Topology and component choices.
Transistor sizes: W/L ratios affecting drive strength.
Loading conditions: Capacitive loads impacting timing.

These delays are stored in lookup tables (LUTs) for use in Static Timing Analysis (STA). Below is an example of a delay LUT for an inverter (INVX1):
cell (INVX1) {
    pin (A) {
        direction : input;
        capacitance : 0.005;
    }
    pin (Y) {
        direction : output;
        function : "(!A)";
        max_capacitance : 0.1;
        timing() {
            related_pin : "A";
            timing_sense : negative_unate;
            cell_rise (delay_template_2d) {
                index_1 ("10, 30, 50, 70, 90"); // Input Slew (ps)
                index_2 ("10, 30, 50, 70, 90, 110"); // Output Load (fF)
                values (
                    "0.05, 0.07, 0.09, 0.11, 0.13, 0.15",
                    "0.06, 0.08, 0.10, 0.12, 0.14, 0.16",
                    "0.07, 0.09, 0.11, 0.13, 0.15, 0.17",
                    "0.08, 0.10, 0.12, 0.14, 0.16, 0.18",
                    "0.09, 0.11, 0.13, 0.15, 0.17, 0.19"
                );
            }
        }
    }
}

Interpolation and Extrapolation in Delay Calculation
In STA, input slew and output load values often do not match LUT entries exactly, requiring interpolation or extrapolation:

Interpolation:

Used when values fall between LUT points.
STA tools perform linear or bilinear interpolation for high accuracy.
Formula: Delay_est = D1 + [(X - X1)/(X2 - X1)] * (D2 - D1)
Example: For input slew = 40 ps, output load = 60 fF (between 50 fF and 70 fF), interpolate between LUT values.


Extrapolation:

Used when values fall outside LUT bounds (e.g., >110 fF).
Extends the slope of nearest points but is less accurate.
Should be minimized for reliable timing results.



Key Questions Addressed

Where does cell delay come from? Transistor characteristics, load capacitance, and input slew.
Are delay models accurate? SPICE provides ground truth; STA models are accurate if derived from well-characterized SPICE data.
How to verify STA results? Compare with SPICE-simulated delays under identical conditions.

NMOS Transistor
NMOS Overview
An NMOS (N-channel Metal Oxide Semiconductor) transistor is a 4-terminal device:

Source (S), Drain (D), Gate (G), Substrate/Body (B)
Built on a p-type substrate with n+ diffusion regions, gate oxide (SiO₂), and a polysilicon gate.
Majority carriers: holes; minority carriers: electrons.

Case I: V_GS = 0

Source, drain, and body are grounded, forming reverse-biased PN junctions (substrate-to-source and substrate-to-drain).
Resistance (R = V/I) is infinite as V = 0, I = 0.
Result: High resistance, no current flow, no conductive channel (MOSFET OFF).

Case II: Small Positive V_GS

Positive gate voltage repels holes, creating a depletion region with immobile negative ions.
Electrons are attracted but insufficient for conduction.
The p-type substrate near the surface resembles a reverse-biased PN junction.
Result: No channel forms; MOSFET remains OFF.

Case III: V_GS > V_T (Threshold Voltage)

Increased V_GS widens the depletion region, inducing an n-type channel (strong inversion) at the Si–SiO₂ interface.
Electrons from the n+ source form a conductive channel, allowing current (I_D) to flow.
Channel conductivity increases with higher V_GS.
Result: MOSFET ON, current flows from source to drain.

Body Effect
The body effect increases the threshold voltage (V_T) when the source-to-body voltage (V_SB) is non-zero:

V_T = V_T0 + γ [√(|−2ϕ_F + V_SB|) − √(|−2ϕ_F|)]
V_T0: Threshold voltage when V_SB = 0
γ: Body-effect coefficient, γ = √(2qε_Si N_A) / C_ox
ϕ_F: Fermi potential, ϕ_F = −(kT/q) ln(n_i / N_A)
q: Electron charge (1.6 × 10⁻¹⁹ C)
ε_Si: Permittivity of silicon (11.7 × ε_0)
N_A: Substrate doping concentration
C_ox: Oxide capacitance, C_ox = ε_ox / t_ox


Example: Positive V_SB increases V_T, requiring higher V_GS to turn ON the transistor.

Linear (Resistive) Region

Condition: V_GS > V_T, small V_DS
Channel charge: Q_i(x) = −C_ox * [(V_GS − V(x)) − V_T]
Drift current dominates: I_D = −v_n(x) * Q_i(x) * W
v_n(x) = μ_n * E = μ_n * (dv/dx) (electron velocity)
μ_n: Electron mobility
W: Channel width


Integrating across the channel (x = 0 to L, v = 0 to V_DS):
I_D * L = μ_n C_ox W [ (V_GS − V_T)V_DS − (V_DS² / 2) ]


Define k_n = μ_n C_ox (W/L) (gain factor):
I_D = k_n [ (V_GS − V_T)V_DS − (V_DS² / 2) ]


Behavior: MOSFET acts as a voltage-controlled resistor.

Numerical Example
For V_T = 0.454 V, V_DS = 0.05 V, V_GS = 1.0 V:

Check: V_GS − V_T = 1.0 − 0.454 = 0.546 V > V_DS (linear region).
I_D = k_n [ (0.546)(0.05) − (0.05² / 2) ] = k_n * 0.02605




V_GS (V)
ΔV = V_GS − V_T (V)
Region
I_D (in terms of k_n)



0.5
0.046
Linear
0.00125 * k_n


1.0
0.546
Linear
0.02605 * k_n


1.5
1.046
Linear
0.05125 * k_n


2.0
1.546
Linear
0.07625 * k_n


2.5
2.046
Linear
0.10125 * k_n


Saturation Region and Pinch-Off

Pinch-off condition: V_DS ≥ V_GS − V_T
At pinch-off, the channel disappears at the drain, and current saturates:
I_D = (k_n / 2) * (V_GS − V_T)² * (1 + λ * V_DS)
λ: Channel-length modulation parameter (increases with shorter channels).


Example: For V_GS = 1.0 V, V_DS = 0.55 V, V_T = 0.454 V:
V_GS − V_DS = 1.0 − 0.55 = 0.45 V ≈ V_T → Pinch-off occurs.
Current saturates as the channel ends at the drain.



Velocity Saturation (Short-Channel Devices)

At high electric fields (E > E_c), carrier velocity saturates:
v(E) = (μ * E) / (1 + E / E_c)
E_c = μ / v_sat, v_sat: Maximum carrier velocity (~10⁵ m/s).


Drain current: I_D = W * Q_i(x) * v(x)
Impacts short-channel devices, reducing I_D and flattening I_D-V_GS curves.

Installation of NGSPICE and Workshop Collaterals
NGSPICE Installation
Download and install NGSPICE (version 44):
tar -zxvf ngspice-44.tar.gz
cd ngspice-44
mkdir release
cd release
../configure --with-x --with-readline=yes --disable-debug
make
sudo make install
ngspice

Workshop Collaterals
Clone the Sky130 workshop repository:
git clone https://github.com/kunalg123/sky130CircuitDesignWorkshop.git
cd sky130CircuitDesignWorkshop
ls

This includes SPICE models, netlists, and scripts for the Sky130 PDK.
SPICE Overview
SPICE, developed at UC Berkeley, solves nonlinear equations for components like transistors, resistors, and capacitors. Variants include:

Free: Ngspice, LTSpice
Commercial: HSPICE, PSPICE

A SPICE deck includes:

Netlist: Component interconnections.
Device Models: Transistor behavior parameters.
Initial Conditions: Starting voltages/states.
Inputs: Voltage/current sources.
Simulation Commands: Analysis types (e.g., .op, .dc, .tran).

CMOS NMOS I–V Characteristics and Velocity Saturation
Experiment 1: MOSFET Behavior (I_D vs. V_DS)
Simulate NMOS transistors to study I_D vs. V_DS for different V_GS values:

Long-channel: W = 1.8 μm, L = 1.2 μm, W/L = 1.5
Short-channel: W = 0.375 μm, L = 0.25 μm, W/L = 1.5

SPICE Netlist:
* SKY130 NMOS Circuit Simulation
.param temp=27
.lib "sky130_fd_pr/models/sky130.lib.spice" tt
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=5 l=2
R1 n1 in 55
Vdd vdd 0 1.8V
Vin in 0 1.8V
.op
.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2
.control
run
plot -vdd#branch
.endc
.end

Observations:

Long-channel:
Quadratic I_D-V_GS dependence.
Linear I_D-V_DS in the linear region.
Saturation current independent of V_DS.
Peak I_D: 410 μA.


Short-channel:
Quadratic I_D-V_GS at low V_GS, linear at high V_GS due to velocity saturation.
Peak I_D: 210 μA (lower due to velocity saturation).
Additional mode: Velocity saturation between linear and saturation regions.



Experiment 2: Threshold Voltage Extraction

Sweep V_GS to plot I_D vs. V_GS.
Extract V_T using linear extrapolation in the linear region.
Short-channel devices exhibit velocity saturation, reducing I_D and flattening the I_D-V_GS curve.

CMOS Inverter Analysis
CMOS Inverter Overview
A CMOS inverter consists of a PMOS (pull-up) and NMOS (pull-down) transistor:

V_in = V_DD: NMOS ON, PMOS OFF → V_out = 0.
V_in = 0: PMOS ON, NMOS OFF → V_out = V_DD.
Load capacitance (C_L) affects transient response.

Experiment 3: Voltage Transfer Characteristic (VTC)
Simulate VTC by sweeping V_in (0 to 1.8V):
* CMOS Inverter VTC Simulation
.param temp=27
.lib "sky130_fd_pr/models/sky130.lib.spice" tt
M1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.9375u l=0.25u
M2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.375u l=0.25u
Cload out 0 10f
Vdd vdd 0 1.8
Vin in 0 1.8
.op
.dc Vin 0 1.8 0.05
.control
run
plot out vs in
.endc
.end

Results:

Switching threshold (V_m): V_in = V_out, typically near V_DD/2.
Regions of operation:
PMOS linear, NMOS OFF
PMOS linear, NMOS saturation
PMOS saturation, NMOS saturation (V_m occurs here)
PMOS saturation, NMOS linear
PMOS OFF, NMOS linear


VTC slope = −1 at V_IL (max input for LOW) and V_IH (min input for HIGH).

Experiment 4: Transient Response
Apply a pulse input to measure rise (t_PLH) and fall (t_PHL) delays at 50% of V_DD:
* CMOS Inverter Transient Simulation
.param temp=27
.lib "sky130_fd_pr/models/sky130.lib.spice" tt
M1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.9375u l=0.25u
M2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.375u l=0.25u
Cload out 0 10f
Vdd vdd 0 1.8
Vin in 0 PULSE(0 1.8 0 1n 1n 50n 100n)
.tran 0.1n 200n
.control
run
plot out vs time in
.endc
.end

Results:

Rise time (t_r): Time for V_out to transition from LOW to 50% of HIGH.
Fall time (t_f): Time for V_out to transition from HIGH to 50% of LOW.
Delays depend on transistor sizing (W/L) and load capacitance (C_L).
Balanced sizing (W_p/W_n ≈ 2) yields symmetric delays (~80 ps).

Experiment 5: Noise Margin Analysis
From VTC, extract:

V_IL: Maximum input voltage for logic LOW.
V_IH: Minimum input voltage for logic HIGH.
V_OL: Output LOW voltage.
V_OH: Output HIGH voltage.
Noise margins:
NML = V_IL − V_OL
NMH = V_OH − V_IH


SPICE Command:ngspice day4_inv_noisemargin_wp1_wn036.spice
plot out vs in



Results:

Balanced sizing ensures symmetric noise margins.
Noise margins define tolerance to input voltage fluctuations.
Undefined region (V_IL to V_IH): Input noise here causes unstable outputs.

Experiment 6: Power Supply and Device Variation
Simulate VTC and delays for:

Power supply variations: V_DD = 1.6V, 1.8V, 2.0V.
Device variations: Double/halve W/L of PMOS or NMOS.
SPICE Command:ngspice day5_inv_supplyvariation_Wp1_Wn036.spice
plot out vs in



Results:<img width="1366" height="768" alt="Screenshot from 2025-10-17 19-20-36" src="https://github.com/user-attachments/assets/69777a65-372c-403e-9073-5ebc2bc135ab" />
<img width="1366" height="768" alt="Screenshot from 2025-10-17 19-00-58" src="https://github.com/user-attachments/assets/2a5d7a39-0f1d-45c4-96d5-d67ca732ba51" />



Power Supply:
Lower V_DD (e.g., 0.5V): Reduces noise margins and delays, saves ~90% energy, but slows switching.
Higher V_DD: Improves noise margins, increases power consumption.


Device Variations:
Etching variations: Affect W/L, shifting V_m and delays.
Oxide thickness (T_ox): Impacts C_ox, altering drive current and performance.
Stronger PMOS (higher W_p): Increases V_m (e.g., 1.2V vs. 0.98V for equal sizing).
Weaker transistors: Higher resistance, slower transitions.



CMOS Design Flow: From Transistor to STA
Transistor Physics
      ↓
Device I–V Characteristics
      ↓
CMOS Inverter (VTC + Transient)
      ↓
Delay, Noise Margin, Variation
      ↓
Timing Models (.lib)
      ↓
Static Timing Analysis (STA)


Level 1: Transistor Behavior:
I_D-V_DS/V_GS, V_T, velocity saturation.


Level 2: CMOS Inverter:
VTC, transient response, noise margins.


Level 3: Timing Behavior:
Delays (t_PHL, t_PLH), PVT effects, power trade-offs.


Level 4: STA:
.lib models for delay, slew, and load.
Ensures timing closure via arrival time, slack, and setup/hold margins.



Deliverables
Document in a markdown report:

Introduction: Purpose and objectives of each experiment.
SPICE Netlists: Complete netlists for all experiments.
Plots:
I_D vs. V_DS/V_GS (mark linear, saturation, velocity saturation regions).
VTC (mark V_m, V_IL, V_IH, V_OL, V_OH).
Transient waveforms (mark t_r, t_f, t_PHL, t_PLH).
Effects of supply and device variations.


Tabulated Results:
V_T, t_PHL, t_PLH, NML, NMH, variation effects.


Analysis:
Compare long- vs. short-channel behavior.
Discuss impact of sizing, supply voltage, and variations on performance.



References

NGSPICE
Sky130 Circuit Design Workshop
UC Berkeley SPICE
LTSpice
HSPICE


