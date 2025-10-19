# Week 4 – CMOS Circuit Design & SPICE Simulation (Sky130-PDK)

<details>
<summary>Day 1: Introduction to Circuit Design & SPICE Simulations</summary>
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
