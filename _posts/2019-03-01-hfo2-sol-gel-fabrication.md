---
layout: post # Do not change
category: [semiconductor] # Added materialscience for better discovery
title: "High-k Sol-Gel HfO₂ Gate Dielectric Layer Fabrication"
date:   2019-03-01 10:00:00 +0800 # Date of project start
author: LilyHua # Your author nick
nextPart: _posts/2021-09-01-gans-regression-paper.md # Links to your next project
# prevPart: _posts/2018-06-01-matlab-particle-visualization.md # Linking to the MATLAB post
og_description: "Developed low-cost, high-performance MOS capacitor (MOS-C) test structures using the Sol-Gel spin-coating process for HfO2 dielectrics. Used C-V and I-V measurements for electrical characterization."
---

This project was a hands-on investigation into developing alternative, low-cost fabrication methods for high-performance semiconductor components. The goal was to replace traditional silicon dioxide ($\text{SiO}_2$) with a **High-k dielectric layer ($\text{HfO}_2$)** for use in Metal-Oxide-Semiconductor Capacitors (MOS-C).

### Material Background: The Role of $\text{HfO}_2$

Hafnium Dioxide ($\text{HfO}_2$) is crucial in modern microelectronics as a **High-k material** used to increase gate capacitance and effectively suppress leakage current in scaled CMOS processes. More recently, it has been discovered that doped $\text{HfO}_2$ also exhibits **ferroelectric (Fe-) properties** and is fully compatible with existing advanced CMOS manufacturing technology, unlike traditional perovskite Fe-materials (e.g., PZT). This dual nature makes it a highly promising material for next-generation **FeRAM** (Ferroelectric RAM) and advanced logic gates.

### Core Fabrication Process: Sol-Gel Methodology

My role involved the complete fabrication and analysis workflow for developing a low-cost $\text{HfO}_2$ deposition process:

#### 1. Sol Preparation
The $\text{HfO}_2$ films were prepared using the **Sol-Gel method**, which involves dissolving a metal alkoxide precursor, such as Hafnium Oxychloride ($\text{HfOCl}_2 \cdot 8\text{H}_2\text{O}$), in a solvent like ethanol. The process involved:
* **Hydrolysis and Condensation:** Slowly adding water and a catalyst to the solution to initiate the chemical reaction, forming a stable suspension of nanoscale particles (the 'sol').
* **Aging:** Allowing the sol to age at room temperature to ensure proper viscosity and stability for uniform coating.

#### 2. Substrate Pre-treatment and Deposition
* **Substrate Cleaning:** Silicon ($\text{Si}$) wafers were used as substrates and required thorough cleaning.
* **Hydrophobic Treatment:** Chemical treatment (e.g., using HMDS) was applied to the substrate to reduce surface energy, ensuring the $\text{HfO}_2$ sol spread and coated the surface with high uniformity during deposition.
* **Spin Coating:** The $\text{HfO}_2$ sol was deposited onto the $\text{Si}$ substrate via the **Sol-Gel spin-coating process**.

#### 3. Post-Deposition and Electrical Characterization
* **Soft Baking:** The coated substrate was baked at a low temperature ($\text{150-200}^\circ\text{C}$) to remove residual solvents and moisture.
* **Electrical Analysis:** Developed complete MOS-C test structures to perform rigorous electrical analysis, including **C-V (Capacitance-Voltage) and I-V (Current-Voltage) measurements**. This allowed for direct evaluation of key performance metrics, such as the dielectric constant (k-value) and leakage current density, for comparison against traditional $\text{SiO}_2$ films.