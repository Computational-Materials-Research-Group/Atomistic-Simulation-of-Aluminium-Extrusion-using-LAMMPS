# Atomistic Simulations using LAMMPS


<img width="1600" height="1200" alt="extrusion" src="https://github.com/user-attachments/assets/3b38f264-1ef2-4745-ac17-378295ecb0d3" />

**Author:** Akshansh Mishra
**GitHub:** [@akshansh11](https://github.com/akshansh11)

---

Molecular dynamics simulations built and validated in LAMMPS (7 Feb 2024). The repository covers four physically distinct problems — soft robotics, tribology, soft matter fracture, and metal forming — each modelled from geometry construction through potential assignment to trajectory analysis. Every script is a working input file, not a template.

All metal simulations use the Al99.eam.alloy EAM potential. Soft matter simulations use LJ/cut pair interactions and FENE bond potentials in reduced LJ units. Trajectories are written in LAMMPS custom dump format for direct import into OVITO.

---

## Simulations

### Soft Robotic Actuator

A coarse-grained pneumatic finger modelled as a 10x3x3 bead-spring lattice in LJ units. Three atom layers represent the fixed base, free body, and pressurised chamber wall. FENE bonds connect nearest neighbours. Pressure is applied as a per-atom force that ramps from zero over 50,000 steps, driving the actuator from flat to bent and back.

```
units        lj
bond_style   fene
fix          addforce  with variable pressure v_p_act
```

The simulation runs in three phases: inflation, hold, and deflation. Tip displacement is tracked throughout using a COM compute on the free end group.

---

### Rolling Sphere on Aluminium Slab

A cold aluminium sphere rolling across a hot Al slab under EAM forces. The sphere is treated as a rigid indenter using `fix rigid/nve` combined with a prescribed rolling velocity set from the no-slip condition at the contact point.

```
Rolling velocity :  vx_cm  = 3.0  Ang/ps
Angular velocity :  omega  = 0.15 rad/ps   ( = vx / R )
No-slip check    :  vx at bottom atom = 3.0 - 0.15*20 = 0  (correct)
```

The slab is thermostated at 700 K throughout. The sphere carries no thermostat and no thermal motion — it is a prescribed rigid body. Per-atom von Mises stress and hydrostatic pressure are computed at every dump step. The Hertzian contact zone and the stress wave propagating ahead of the sphere are both visible in OVITO.

Output files:

| File | Contents |
|------|----------|
| `stress_rolling.lammpstrj` | Per-atom position + full stress tensor + von Mises |
| `temp_rolling.lammpstrj` | Per-atom kinetic energy for temperature mapping |
| `sphere_com.txt` | Sphere centre of mass vs time |
| `slab_stress_vs_time.txt` | Global slab pressure in GPa vs timestep |

---

### Spider Web Impact

A coarse-grained spider web generated in Python and simulated under sphere impact in LAMMPS. The web consists of 12 radial threads and 8 spiral rings built as a bead-spring network. Outer boundary beads are pinned as a fixed atom type, removing any dependency on region-based grouping.

Bond parameters are set to approximate the real mechanical contrast between dragline and capture silk:

| Thread type | Bond coeff K | Max extension R0 | Physical basis |
|-------------|-------------|------------------|----------------|
| Radial (type 1) | 30.0 | 1.5 | Stiff dragline silk, breaks at ~50% extension |
| Spiral (type 2) | 10.0 | 2.0 | Soft capture silk, breaks at ~100% extension |

The stiffness ratio of 3:1 matches experimental measurements of real spider webs. The sphere is a single heavy atom (mass 50) falling under gravity. Under sufficient impact the central bonds fail first and fracture propagates radially outward — the same sequence observed in physical web damage experiments.

To generate the web geometry and run:

```bash
python generate_web.py
lmp -in in.spiderweb
```

---

### Direct Extrusion of Aluminium

Forward extrusion of an FCC aluminium billet through a rigid die at 2:1 reduction ratio. The punch advances at 0.5 Ang/ps using `fix move linear`. The die is fully frozen. The billet is thermostated at 300 K with NVT. Per-atom von Mises stress is computed throughout and the die reaction force is logged every 200 steps.

Geometry (lattice units, a = 4.05 Ang):

```
Punch     :   3 units thick
Billet    :  30 x 20 x 8 units
Die       :   5 units thick, opening = 10 units  (ratio 2:1)
Collection:  30 units downstream
```

Three deformation zones appear in the trajectory:

**Dead zone.** The corner where the billet face meets the die wall. Atoms stagnate here with near-zero velocity and low von Mises stress. In industrial extrusion this region can detach and emerge as a surface defect. The simulation shows its formation and growth in real time.

**Shear band.** A 45-degree band of intense plastic flow running from the punch face to the die opening. Von Mises stress peaks here. The original FCC crystal order is destroyed within this band as atoms are sheared into new configurations.

**Extrudate.** Material that has passed through the die opening and is now flowing freely downstream. Cross-section is half the billet height. Crystal structure is gone.

The file `extrusion_force.txt` contains punch position and die reaction force at every logged step. Plotting force against position gives the load-displacement curve with a clear peak at extrusion onset followed by a drop to steady-state flow.

---

## Requirements

- LAMMPS 7 Feb 2024 or later
- `Al99.eam.alloy` potential file — available from the [NIST Interatomic Potentials Repository](https://www.ctcms.nist.gov/potentials/)
- Python 3.x with NumPy (spider web geometry generation only)
- OVITO 3.x for trajectory visualization

Place `Al99.eam.alloy` in the same directory as the input script before running any simulation in metal units.

---

## OVITO Visualization

All trajectories are in LAMMPS custom dump format and open directly in OVITO via File > Load File.

**Von Mises stress:**
Add a Color Coding modifier. Set property to `vm_stress`, colormap to Hot, range 0 to 20000. Disable auto-adjust range so the scale stays consistent across frames.

**Temperature (kinetic energy):**
Load `temp_rolling.lammpstrj`. Add Color Coding on `c_atom_ke`, colormap Rainbow, range 0 to 0.15 eV.

**Bond fracture in spider web:**
Add a Create Bonds modifier set to show only bonds shorter than 1.8 sigma. Broken bonds disappear as atoms separate beyond R0. Fracture sequence is visible frame by frame.

---

## Repository Structure

```
.
├── soft_robot/
│   └── in.soft_robot
├── rolling_sphere/
│   └── in.rolling
├── spider_web/
│   ├── generate_web.py
│   └── in.spiderweb
├── extrusion/
│   └── in.extrusion
└── README.md
```

---

## License

[![CC BY-NC 4.0](https://licensebuttons.net/l/by-nc/4.0/88x31.png)](https://creativecommons.org/licenses/by-nc/4.0/)

This work is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License](https://creativecommons.org/licenses/by-nc/4.0/).

You are free to share and adapt the material for any non-commercial purpose, provided that appropriate credit is given to **Akshansh Mishra**, a link to the license is included, and any changes are indicated. Commercial use of any kind is not permitted without prior written permission from the author.

---

## Contact

**Akshansh Mishra**
GitHub: [akshansh11](https://github.com/akshansh11)
