---
layout: post
title: "Metalens Fabrication with Dry Etching: From Lam Research Process Tools to Diffraction-Limited Optics"
date: 2024-01-15
category: "Nanophotonics · Process Engineering · Lam Research"
summary: "A process engineer's perspective on fabricating dielectric metalenses using dry etching — the physics of metasurface phase control, etch chemistry that makes or breaks pillar profiles, and an open-source RCWA + Bosch simulation toolkit."
description: "Literature review and process engineering perspective on metalens fabrication via dry etching, drawing on hands-on experience with conductor etch tools at Lam Research. Covers geometric phase design, ICP-RIE recipe development, SEM profile analysis, and an open-source RCWA + roofline simulation toolkit."
tags: [Metalens, Dry Etching, ICP-RIE, RCWA, Nanophotonics, Metasurface, Lam Research, Silicon, Bosch DRIE, Phase Control]
---

This post grew out of two years on the process side at Lam Research, running conductor etch on production tools — and the persistent, nagging question of what happens when you point that hardware at a photonics problem instead of a logic device. Metalenses sit at an unusual intersection: the feature scales (200–500 nm pillars, 1–2 µm pitch) are well within what a modern ICP-RIE tool handles routinely, yet the optical performance demands on sidewall angle, aspect ratio, and dimension control are as tight as anything in leading-edge logic. This post is a literature synthesis of where the field stands on dry-etch metalens fabrication, why etch chemistry choices matter enormously, and what process tradeoffs to consider before committing wafer time.

---

## Background: What a Metalens Actually Is

Conventional refractive optics accumulate phase by propagating light through a bulk medium of varying thickness. A lens works because more glass means more phase delay — the thicker the center, the more the wavefront curves, and the more tightly the lens focuses. This is effective but fundamentally three-dimensional: a 50 mm camera lens is 50 mm long because it needs that propagation distance.

A metasurface decouples phase accumulation from propagation distance. Instead of a thick slab, it uses a two-dimensional array of subwavelength resonators — pillars, discs, fins — each of which imparts a locally controlled phase shift to the transmitted field. The phase is engineered by varying the geometry of each resonator. The whole structure is typically less than 2 µm tall. The resulting device, a **metalens**, is a flat, lithographically patterned optical element that can focus, collimate, or aberration-correct light in a single planar layer [1].

The enabling physics comes in two flavors, and the distinction matters for fabrication.

### Resonance-Based Phase (Mie / Propagation Phase)

Dielectric pillars support Mie-type electromagnetic resonances. By sweeping pillar diameter while holding height constant, you trace a phase response from 0 to 2π. Silicon nanopillars at NIR wavelengths (1310–1550 nm) are a canonical example: the refractive index contrast between Si (*n* ≈ 3.5) and air or SiO₂ is large enough to support strong resonances with feature sizes of 200–400 nm [2].

The transmitted phase at position **r** across the lens must follow the lens phase profile:

$$\phi(r) = \frac{2\pi}{\lambda} \left( \sqrt{r^2 + f^2} - f \right)$$

where *f* is the focal length and *λ* is the design wavelength. Every pillar is assigned the nearest-library geometry whose simulated phase matches the required φ(*r*).

### Geometric Phase (Pancharatnam–Berry Phase)

The geometric phase approach uses a single pillar geometry optimized to act as a half-wave plate, and varies its **rotation angle** α(*r*) = φ(*r*)/2. When circularly polarized light passes through a rotated half-wave plate, the cross-polarized output acquires a phase of 2α — entirely independent of the pillar's resonance wavelength [3].

For fabrication this is significant: dimensional errors primarily affect the **efficiency** of the lens rather than the **focal spot size** or **aberrations** (which are set by the rotation angle pattern). A lens that is 20 nm too wide in every pillar still focuses diffraction-limited; it just wastes more power into the unconverted polarization. This tolerance makes geometric-phase metalenses substantially more manufacturable [4].

---

## Why Dry Etching Is the Hard Part

Every dielectric metalens requires the same basic sequence: deposit the device layer, define the pillar pattern by lithography, transfer the pattern by etching, strip the mask. For most dielectric materials — Si, aSi, Si₃N₄, TiO₂, GaN — the etch step is the bottleneck. The requirements are severe:

- **Vertical sidewalls.** A sidewall angle of 85° instead of 90° changes the effective lateral dimension by ~50–100 nm over a 1.2 µm tall pillar — enough to detune the resonance entirely.
- **High aspect ratios.** NIR Si metalenses need pillars 1.0–1.5 µm tall with widths of 150–350 nm, giving gap aspect ratios of 5:1 to 10:1.
- **Smooth sidewalls.** Sub-10 nm RMS roughness is the target; roughness scatters light and raises background noise.
- **Dimensional fidelity.** The etch must reproduce the lithographic CD within ±20–30 nm to hit the design phase.

These requirements interact in ways that make a single "best" recipe impossible. Chemistry that gives great verticality tends to leave rougher sidewalls. Chemistry that gives smooth sidewalls can struggle at high aspect ratios. Bosch DRIE gives the best verticality and depth control but creates characteristic scalloped sidewalls. The field has converged on a rough taxonomy of etch approaches.

---

## Etch Approaches: A Process Engineer's Map

### Continuous ICP-RIE (F-based, for Si and aSi)

For silicon metalenses operating in NIR, fluorine-based ICP-RIE using SF₆/C₄F₈ or SF₆/C₄F₆ mixtures is the most common approach in research. The SF₆ provides the etch species (F radicals attack Si spontaneously), while C₄F₈ deposits a passivation polymer on sidewalls to suppress lateral etching. The ratio between the two controls the balance between etch rate and sidewall protection.

<figure>
  <img src="/assets/img/posts/metalens_aSi_etch_matrix.png" alt="SEM matrix of aSi metalens pillars under different SF6/C4F8 ratios">
  <figcaption>SEM images of aSi metalens pillars etched with varying SF₆/C₄F₈ ratios and chamber pressures on an Alcatel Speeder 100 ICP system. Moving from (a) low valve opening / low power to (f) optimized SF₆:C₄F₈ = 25:35 achieves ~10:1 aspect ratio with vertical sidewalls. Mask material matters: Cr gives better etch selectivity over Al₂O₃ for deep Si etches. Figure from Zhang et al. [5].</figcaption>
</figure>

The key process handles are SF₆ fraction (etch rate vs. isotropy), C₄F₈ fraction (passivation, risk of polymer buildup), pressure (anisotropy via mean free path), ICP power (plasma density), and bias RF (ion energy and directionality). The separation of plasma density and ion energy is the fundamental advantage of ICP over classical parallel-plate RIE — and why ICP tools are standard for metalens work.

HBr-based chemistries (HBr/Ar) offer an alternative with very high selectivity to oxide, enabling precise etch stops on buried SiO₂ layers. The wafer-scale aSi metalens results from the Penn State Ni group used HBr:Ar = 20:4 at 8 mTorr, 30W/1900W on an Oxford Cobra — producing the diffraction-colored wafers characteristic of a well-controlled uniform etch [5].

### Cl-Based ICP-RIE (for III-V and InGaAsP)

III-V materials (InGaAsP, InP, GaAs) used for active metasurfaces and OAM lasers cannot be etched with fluorine — the InF₃ etch products are involatile. Chlorine-based chemistry (BCl₃, Cl₂, BCl₃/N₂) is required. The volatility of chloride reaction products is strongly temperature-dependent: below ~190°C the etch products don't desorb efficiently; above ~280°C lateral selectivity collapses and RIE lag becomes severe [5].

<figure>
  <img src="/assets/img/posts/metalens_InGaAsP_etch_matrix.png" alt="SEM images of InGaAsP/InP microrings etched with BCl3 at different coil powers">
  <figcaption>InGaAsP/InP microring resonators etched with pure BCl₃ (30 sccm, 2 mTorr, 80°C chuck) at varying ICP coil power. (a) 250W — excessive sidewall angle and footing. (b) 400W — improved verticality. (c, d) BCl₃/N₂ mixtures with and without He backside cooling. The proper operating window for InP-based devices is 190–250°C substrate temperature. Scale bar: 1 µm. Figure from Zhang et al. [5].</figcaption>
</figure>

### Bosch DRIE (for Si, high aspect ratio)

The Bosch process — alternating SF₆ isotropic etch pulses and C₄F₈ passivation pulses — is the workhorse for high-aspect-ratio silicon structures. It achieves near-vertical sidewalls at gap ARs well above 10:1, but leaves characteristic **scalloped sidewalls**: each etch/passivate cycle leaves a half-elliptical indent of depth *r_d* and height 2*r_h*.

The key insight from Dirdal et al. is that scallops do not disqualify Bosch from metalens use — they can be **compensated by design**. The scalloped pillar has a reduced effective cross-section; increasing the lateral dimensions in the mask by:

$$w' = 1.038 \left( w + \frac{\pi R}{2} \right), \quad l' = 1.038 \left( l + \frac{\pi R}{2} \right)$$

(for scallop radius *R*) recovers the target cross-polarization efficiency spectrum [4]. The 3.8% factor above pure volume-conservation accounts for the fact that scallops shift the Mie resonance — a correction that must be verified by simulation for each design.

<figure>
  <img src="/assets/img/posts/metalens_bosch_scallops.png" alt="SEM and FDTD simulation of Bosch scalloped Si pillars and compensation strategy">
  <figcaption>(a) FDTD-simulated cross-polarized transmission for smooth Si pillars (solid black), scallopy pillars with compensated dimensions (dashed red), and two reference curves. The compensated scallopy pillar recovers near-identical response to the smooth target. (b) SEM of actual Bosch-etched Si pillar (center) alongside smooth-wall FDTD model (left) and FDTD refractive-index cross-section of the scalloped structure (right, R = 50 nm). Figure from Dirdal et al. [4].</figcaption>
</figure>

<figure>
  <img src="/assets/img/posts/metalens_bosch_scallop_depths.png" alt="SEM comparison of three scallop depth regimes">
  <figcaption>Three scallop depth regimes on the same tool. (i) Optimized: ~14 nm — minimal compensation needed. (ii) Aggressive passivation removal: ~76.5 nm — used to trim oversized pillars from resist foot broadening. (iii) Moderate scallops (~44 nm) followed by thermal oxidation and oxide strip: final ~29 nm and dimensions near target. Figure from Dirdal et al. [4].</figcaption>
</figure>

### Si₃N₄: CHF₃/CF₄ Chemistry

Silicon nitride is favored for visible-wavelength metalenses (transparent to ~400 nm, *n* ≈ 1.9–2.0). The etch chemistry is CHF₃/CF₄-based ICP-RIE. The CHF₃/CF₄ ratio controls the F/C balance: higher CHF₃ improves sidewall angle but risks polymer buildup at feature bases. Achievable aspect ratios (~5:1) are lower than for Si, motivating multi-layer stacked designs like the SiNₓ double-layer achromatic metalens from the Penn State Ni group [5].

---

## From Lam Tools to Metalens Profiles

At Lam Research, conductor etch runs on TCP (Transformer-Coupled Plasma) or Kiyo tools optimized for tungsten, polysilicon, and metal gate stacks. Several process characteristics transfer directly; others require deliberate adjustment.

**What transfers well:** The fundamental plasma physics is identical. TCP source power controls plasma density; bias RF controls ion bombardment energy. The independent control that makes conductor etch capable of etching high-AR contacts also enables the anisotropy needed for metalens pillars. Endpoint detection infrastructure (OES, interferometry) is mature. Chamber conditioning discipline is if anything more critical for metalens work.

**What requires adjustment:** Etch loading is the biggest surprise. A metalens die is a tiny patterned area (1–5 mm²) on a mostly bare wafer — loading approaches 100%. This drives microloading effects where the etch rate inside the metalens can differ substantially from a blanket-film calibration. Standard conductor etch tools compensate loading for large patterned areas; for a small optical die on a carrier wafer, the regime is unusual and must be characterized explicitly. Gas chemistry also differs: SF₆/C₄F₈ for Si metalenses is not a standard conductor etch recipe. And the mask stack — thin Cr or Al₂O₃ over e-beam resist — has selectivity of only 5–15:1 against Si, tight for a 1.2 µm etch.

The practical conclusion: a Lam TCP tool can produce metalens-quality profiles in silicon, but it requires a process development effort distinct from the standard recipe library. The dimensional control infrastructure (in-situ OES, post-etch CD-SEM, scatterometry) is a genuine advantage over university MEMS tools.

---

## Optical Performance: What the Profiles Produce

<figure>
  <img src="/assets/img/posts/metalens_focal_spots.png" alt="Focal spot profiles of two metalenses vs. aspherical reference">
  <figcaption>Measured focal spot intensity profiles for metalens MLII (large scallops, 307 × 460 nm pillars), MLIII (oxidation-trimmed, 210 × 320 nm pillars), and an AR-coated aspherical reference — all at NA = 0.045, measured at 1310 nm (a) and 1550 nm (b). Both metalenses achieve diffraction-limited spot sizes. Measured efficiencies: 25.5% (MLII) and 29.2% (MLIII) at 1310 nm, below the theoretical ~60–70% limit primarily due to dimensional mismatch from resist foot broadening. The large chromatic focal shift (~2 mm from 1550→1310 nm) is a fundamental limitation of single-layer geometric-phase metalenses. Figure from Dirdal et al. [4].</figcaption>
</figure>

The efficiency gap between 25–30% demonstrated and ~60–70% theoretical traces directly to mask profile fidelity going into the etch, not to the etch itself. Resist foot broadening in NIL (or proximity effect in EBL) is the dominant source of dimensional error. The message for process development: **characterize your mask profile before optimizing the etch recipe.**

---

---

## Summary

Metalens fabrication with dry etching is a solvable process engineering problem. The field has demonstrated diffraction-limited focusing in Si at NIR using processes compatible with wafer-scale throughput. The efficiency gap between current demonstrations (25–50%) and theoretical limits (60–80%) traces almost entirely to mask profile fidelity and dimensional control, not to fundamental physics.

Three takeaways from a process engineering perspective: geometric phase is the most fabrication-tolerant design because dimensional errors degrade efficiency but not focal quality; Bosch scallops are compensable by design up to ~50 nm depth; and resist foot broadening at the mask base is usually the dominant dimensional error source, not the etch itself.

---

## References

[1] N. Yu and F. Capasso, "Flat optics with designer metasurfaces," *Nature Materials*, vol. 13, pp. 139–150, 2014. https://doi.org/10.1038/nmat3839

[2] M. Khorasaninejad, W. T. Chen, R. C. Devlin, J. Oh, A. Y. Zhu, and F. Capasso, "Metalenses at visible wavelengths: Diffraction-limited focusing and subwavelength resolution imaging," *Science*, vol. 352, no. 6290, pp. 1190–1194, 2016. https://doi.org/10.1126/science.aaf6644

[3] X. Ni, A. V. Kildishev, and V. M. Shalaev, "Metasurface holograms for visible light," *Nature Communications*, vol. 4, p. 2807, 2013. https://doi.org/10.1038/ncomms3807

[4] C. A. Dirdal, G. U. Jensen, H. Angelskår, P. C. V. Thrane, J. Gjessing, and D. A. Ordnung, "Towards high-throughput large-area metalens fabrication using UV-nanoimprint lithography and Bosch deep reactive ion etching," *Optics Express*, vol. 28, no. 10, pp. 15542–15561, 2020. https://doi.org/10.1364/OE.393328

[5] L. Zhang, "Fabrication of Si, Si₃N₄ & InGaAsP Optical Metasurfaces with Dry Etching," Penn State University, presentation, advisor Prof. Xingjie Ni, 2021.

[6] J.-S. Park, S. Zhang, A. She, W. T. Chen, P. Lin, K. M. Yousef, J.-X. Cheng, and F. Capasso, "All-glass, large metalens at visible wavelength using deep-ultraviolet projection lithography," *Nano Letters*, vol. 19, no. 12, pp. 8673–8682, 2019. https://doi.org/10.1021/acs.nanolett.9b03333

[7] W. T. Chen, A. Y. Zhu, V. Sanjeev, M. Khorasaninejad, Z. Shi, E. Lee, and F. Capasso, "A broadband achromatic metalens for focusing and imaging in the visible," *Nature Nanotechnology*, vol. 13, pp. 220–226, 2018. https://doi.org/10.1038/s41565-017-0034-6

[8] C. F. Carlström et al., "Temperature window for BCl₃-based dry etching of InP with low RIE lag," *Journal of Vacuum Science & Technology B*, vol. 26, no. 5, pp. 1675–1683, 2008.
