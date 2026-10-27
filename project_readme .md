# Thermomechanical Simulation of Power Module Solder Joints (COMSOL)

This repository contains a COMSOL Multiphysics (`.mph`) model simulating the thermomechanical stress and viscoplastic creep in the solder layer of a Single-Sided Cooling (SSC) power module. 

To keep the repository lightweight, the provided `.mph` file includes the full geometry, physics setup, and mesh, but the solution data has been cleared. Instructions for running the model and viewing the results are provided below.

## Reference
This model replicates the failure mechanism and stress analysis methodology detailed in the following paper:
> **FEA-Dominant Reliability and Lifetime Model of Double-Sided Cooling SiC Power Module**  
> *Peng Sun, Fuli Niu, Zheng Zeng, Kaiyan Li, and Kaihong Ou*  
> IEEE Transactions on Device and Materials Reliability, Vol. 23, No. 2, June 2023.  
> [DOI: 10.1109/TDMR.2023.3245302](https://doi.org/10.1109/TDMR.2023.3245302)

## Model Overview

Mismatch in the Coefficients of Thermal Expansion (CTE) between packaging materials causes severe internal stress during power cycling. Over time, this stress leads to viscoplastic creep and fatigue failure in the solder layer.

This model is a 2D half-symmetry representation of a standard SSC power module (based on the commercial Semikron SKM50GB123D). 

### Physics & Materials
* **Coupled Physics:** Thermal-Structure Interaction (Heat Transfer in Solids + Solid Mechanics).
* **Materials:** Silicon (Chip), Copper (DBC tracks), Al2O3 (Ceramic substrate), and 60Sn40Pb (Solder).
* **Viscoplasticity:** The solder layer utilizes the **Anand Viscoplasticity Model** to accurately capture strain rate and temperature-dependent deformation (parameters sourced from the reference paper).
* **Thermal Loading:** A periodic square-wave heat source is applied to the chip to induce a junction temperature swing ($\Delta T_j$) of approximately $120^\circ\text{C}$.

### Mesh Strategy
The mesh is pre-generated in the file. Because stress and plastic strain concentrate heavily at the outer corners of the solder layer, a custom meshing sequence is used:
* The global assembly uses a *Fine* mesh.
* The critical Solder domain uses a targeted *Extremely fine* mesh to properly capture the stress concentrations.

## How to Run the Model

Since the solution data has been cleared to save space, follow these steps to generate the results in COMSOL:

### 1. Compute
* Open the file and simply run **Study 1 > Compute**. 
* The study is set up to run three 4-second heating/cooling cycles `range(0, 0.05, 12)` to allow the viscoplastic hysteresis loops to stabilize.

### 2. Solver Settings Note
Because of the rapid thermal shock and extreme nonlinearity of the Anand model, the default solver may fail. The model is pre-configured to avoid this, but if you make changes, ensure:
* **Segregated Solver:** Heat Transfer and Solid Mechanics must be solved in segregated steps.
* **Strict Time Stepping:** In the Time-Dependent Solver settings, Time Stepping must be set to *Strict* (Max step: 0.05 s) to prevent the solver from overshooting the thermal transients.

### 3. Evaluating Results
* **Von Mises Stress:** Plot `solid.mises` (in MPa). At peak heating dwells ($t = 2, 6, 10\text{ s}$), the stress concentrates at the outer upper corner of the solder layer, reaching $150\text{–}160\text{ MPa}$ (matching Figure 14 in the reference paper).
* **Creep Dissipation:** Plot `solid.Wvp` (in $\text{J/m}^3$). This variable tracks the accumulated plastic dissipation energy. Use the energy accumulated over the final cycle ($\Delta W_{sc}$) in conjunction with the Morrow fatigue model to predict the cycle lifetime ($N_{sf}$) of the power module.