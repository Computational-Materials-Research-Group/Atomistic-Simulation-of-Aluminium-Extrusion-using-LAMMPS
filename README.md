# Aluminum Metal Extrusion — LAMMPS Molecular Dynamics Simulation


<img width="1600" height="1200" alt="extrusion" src="https://github.com/user-attachments/assets/5f2a8761-2d75-41e1-a03a-411b3122ff7e" />

**Author:** Akshansh Mishra  
**GitHub:** [akshansh11](https://github.com/akshansh11)

---

## Overview

This repository contains a LAMMPS input script for simulating the direct extrusion of aluminum at the atomic scale using molecular dynamics. The simulation models a three-component setup — a punch, a billet, and a die — constructed from FCC aluminum atoms and driven by an embedded atom method (EAM) potential.

The script has been corrected to eliminate NaN energy values caused by atom overlaps at region interfaces. A gap variable ensures no two adjacent regions share boundary atoms.

---

## Simulation Setup

### Components

| Component | Atom Type | Role |
|-----------|-----------|------|
| Billet    | 1         | Deformable aluminum workpiece |
| Die       | 2         | Fixed constraint with extrusion opening |
| Punch     | 3         | Rigid indenter driving the billet |

### Geometry

The simulation box is constructed in lattice units based on the FCC aluminum lattice parameter `a = 4.05 Angstrom`. The key dimensions are:

- Punch length: 3 lattice units
- Billet length: 30 lattice units
- Die thickness: 5 lattice units
- Die opening (extrusion channel): 10 lattice units
- Billet height: 20 lattice units
- Box depth (z): 8 lattice units

A `gap = 0.6` lattice units is inserted at every component interface to prevent atom overlap during initialization.

### Boundary Conditions

- x: periodic
- y: shrink-wrapped (non-periodic, free surface)
- z: periodic

---

## Simulation Phases

### Phase 0 — Minimization

Before any dynamics, a two-stage energy minimization is performed:

1. `quickmin` — robust for systems with severe overlaps or near-infinite forces
2. `cg` (conjugate gradient) — fine refinement to convergence tolerances

Both the die and punch are held fixed during this phase. The `thermo_modify lost warn` flag is set to prevent abrupt crashes if atoms drift outside the box during early minimization.

### Phase 1 — Thermal Equilibration (10 ps)

The billet is brought to 300 K using an NVT thermostat (Nose-Hoover, damping constant 0.1 ps). Top and bottom billet rows are held fixed to simulate die contact boundary conditions. The die and punch remain stationary.

### Phase 2 — Extrusion (50 ps)

The punch is displaced along the +x direction at a constant velocity of `0.5` lattice units per picosecond using `fix move linear`. Per-atom von Mises stress is computed and dumped every 200 steps. The die reaction force along x is tracked throughout.

### Phase 3 — Hold (10 ps)

The punch is stopped and the system is allowed to relax. This captures springback and residual stress redistribution after extrusion.

---

## Output Files

| File | Contents |
|------|----------|
| `extrusion.lammpstrj` | Per-atom trajectory: id, type, x, y, z, von Mises stress |
| `extrusion_force.txt` | Time-averaged punch position and die reaction force |
| `extrusion_final.restart` | Binary restart file at end of simulation |

---

## Von Mises Stress

Per-atom von Mises stress is computed from the stress tensor components output by `compute stress/atom`:

```
vm = sqrt( 0.5 * [ (sxx-syy)^2 + (syy-szz)^2 + (szz-sxx)^2
                   + 6*(sxy^2 + sxz^2 + syz^2) ] )
```

This quantity is written per atom to the trajectory file and can be visualized directly in OVITO.

---

## Requirements

- LAMMPS (any recent stable release with EAM support)
- EAM potential file: `Al99.eam.alloy`  
  Available from the [NIST Interatomic Potentials Repository](https://www.ctcms.nist.gov/potentials/)

---

## Usage

```bash
mpirun -np 4 lmp -in al_extrusion.lammps
```

Or for serial execution:

```bash
lmp -in al_extrusion.lammps
```

Visualization of the output trajectory is best done with [OVITO](https://www.ovito.org/), using the von Mises stress field for color mapping.

---

## Known Issues and Fixes

### NaN Potential Energy During Minimization

**Root cause:** Adjacent regions (punch/billet, billet/die) shared exact boundary coordinates in lattice units, causing atoms from neighboring regions to be placed on top of each other. This produced infinite pair forces and NaN energies.

**Fix:** A `gap = 0.6` lattice unit offset was added at every interface. Each region's boundaries were pre-computed with this gap factored in so no two regions can share boundary atoms.

**Additional fix:** `quickmin` is used as the first minimization pass. Unlike conjugate gradient, quickmin uses a velocity damping scheme that can recover from near-infinite forces without immediately diverging.

---

## License

[![License: CC BY-NC 4.0](https://licensebuttons.net/l/by-nc/4.0/88x31.png)](https://creativecommons.org/licenses/by-nc/4.0/)

This work is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License](https://creativecommons.org/licenses/by-nc/4.0/).

You are free to share and adapt this material for non-commercial purposes, provided appropriate credit is given to **Akshansh Mishra** and a link to the license is included. Commercial use is not permitted.

---

## Contact

For questions or contributions, open an issue or reach out via GitHub: [github.com/akshansh11](https://github.com/akshansh11)
