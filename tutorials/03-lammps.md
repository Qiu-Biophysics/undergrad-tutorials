# Beginner's Guide to LAMMPS
## Large-scale Atomic/Molecular Massively Parallel Simulator

---

## What Is LAMMPS?

LAMMPS is a free, open-source **classical molecular dynamics (MD)** code developed and maintained by Sandia National 
Laboratories, with contributions from researchers worldwide. Originally written in the 1990s, it has grown into one of 
the most widely used MD codes in computational materials science, chemistry, and physics.

The key word in its name is *classical* — LAMMPS integrates Newton's equations of motion for atoms using 
**interatomic potentials (force fields)** rather than solving Schrödinger's equation like a DFT code. This makes it orders 
of magnitude faster than DFT for large systems or long timescales, at the cost of the quantum-mechanical accuracy that ab 
initio methods provide. For those coming from a DFT background, LAMMPS is where you take the physics you've learned at the 
quantum level and apply it at larger scales — sometimes using potentials trained directly on your DFT data.

LAMMPS can model systems from a few atoms up to billions, runs on a single laptop or on thousands of HPC cores, and supports
an enormous library of interatomic potentials, thermostats, barostats, and analysis tools. It is purely a command-line 
code — simulations are driven by a plain-text input script, and results are written to text output files. There is no 
built-in graphical interface for setting up or visualizing simulations, though a basic GUI launcher (LAMMPS-GUI) is now 
bundled with recent distributions, and OVITO is the standard tool for post-processing and visualizing LAMMPS output.

---

## Part 1: Installation

### Getting LAMMPS

LAMMPS source code and pre-compiled binaries are available at [https://www.lammps.org/download.html](https://www.lammps.org/download.html).

**Option 1 — Pre-compiled binary (easiest for beginners):** Download the binary for your operating system. On Linux, the 
binary is named something like `lmp_linux`. On Windows, an installer package is provided. Recent LAMMPS releases also bundle
LAMMPS-GUI, a lightweight graphical launcher.

**Option 2 — Package manager (Linux):**

On Ubuntu/Debian:
```bash
sudo apt install lammps
```

On Fedora/RHEL:
```bash
sudo dnf install lammps
```

> Note that package manager versions may lag behind the latest LAMMPS release.

**Option 3 — Build from source (recommended for HPC and full feature set):** Building from source gives you control over 
which optional packages are compiled in (e.g., `MANYBODY` for EAM/Tersoff potentials, `REAXFF`, `ML-SNAP`, etc.). LAMMPS 
uses CMake:

```bash
git clone https://github.com/lammps/lammps.git
cd lammps
mkdir build && cd build
cmake ../cmake -DPKG_MANYBODY=yes -DPKG_REAXFF=yes -DPKG_ML-SNAP=yes
make -j 4
```

For MPI-parallel execution (required for running on HPC clusters), ensure MPI libraries are installed and compile with:
```bash
cmake ../cmake -DBUILD_MPI=yes -DPKG_MANYBODY=yes
make -j 4
```

### Running LAMMPS

Once installed, LAMMPS is run from the terminal, pointing it at an input script:

```bash
# Serial (single processor)
lmp -in in.script

# Parallel with MPI (e.g., 8 processors)
mpirun -np 8 lmp -in in.script

# On SLURM-based HPC clusters (see ISAAC-NG tutorial)
srun lmp -in in.script
```

The `-in` flag specifies the input script. All output goes to the terminal and to a `log.lammps` file automatically.

### LAMMPS-GUI

Recent LAMMPS distributions include **LAMMPS-GUI**, a graphical front-end that lets you open and edit input scripts, run 
simulations, and monitor output in a basic visualization window. The left panel is the input script editor; the right panel
shows a live visualization of the simulation that updates as it runs.

LAMMPS-GUI is a convenient way to get started, but for serious research workflows you will use the command line directly 
and visualize output in OVITO.

---

## Part 2: How LAMMPS Works — The Input Script

LAMMPS is entirely controlled by a **plain-text input script**. There is no graphical setup — you write commands in a text 
file, and LAMMPS executes them one line at a time, top to bottom. This makes LAMMPS simulations reproducible, scriptable, 
and easy to adapt.

A LAMMPS input script has four logical sections:

1. **Initialization** — global settings that must be defined before atoms are created
2. **System definition** — create or read in the atomic structure
3. **Simulation settings** — force fields, thermostats, output commands
4. **Run** — actually run the simulation or perform a minimization

Here is a complete, minimal example for an NVT molecular dynamics simulation of copper using an EAM potential:

```lammps
# ========================================================
# Minimal NVT MD simulation of copper (EAM potential)
# ========================================================

# --- SECTION 1: INITIALIZATION ---
units metal            # Å, eV, ps, K, bar
atom_style atomic      # Simple atoms (no bonds or charges)
boundary p p p         # Periodic in all three directions

# --- SECTION 2: SYSTEM DEFINITION ---
lattice fcc 3.615                  # FCC lattice, a = 3.615 Å (Cu)
region box block 0 5 0 5 0 5      # 5×5×5 unit cells
create_box 1 box                   # 1 atom type
create_atoms 1 box                 # Fill with type-1 atoms
mass 1 63.546                      # Copper atomic mass

# --- SECTION 3: SIMULATION SETTINGS ---
pair_style eam                     # Embedded Atom Method potential
pair_coeff * * Cu_u3.eam           # Load EAM potential file for Cu

velocity all create 300.0 12345 mom yes rot yes    # Initialize velocities
fix 1 all nvt temp 300.0 300.0 0.1                 # NVT thermostat

thermo 100                         # Print thermodynamics every 100 steps
thermo_style custom step temp pe ke etotal press vol

dump 1 all atom 500 dump.cu.lammpstrj              # Output trajectory

# --- SECTION 4: RUN ---
timestep 0.001         # 1 fs timestep (metal units: ps)
run 10000              # 10,000 steps = 10 ps
```

The sections below explain each part of this script in detail.

---

## Part 3: Initialization — Units, Boundaries, and Atom Style

### Units

The `units` command is the first thing you should set in every LAMMPS script. It determines what physical units all 
quantities are expressed in. **Choosing the wrong units system for your potential is a common beginner error** that 
produces completely wrong results without any error message.

```lammps
units metal     # Most common for materials science
units real      # Common for organic chemistry / biomolecules
units si        # SI units (rarely used in practice)
units lj        # Lennard-Jones reduced units
units electron  # For DFT-compatible electron-volt based units
```

The two most important systems for materials science work:

| Parameter | `metal` units | `real` units |
|-----------|--------------|-------------|
| Distance | Ångström (Å) | Ångström (Å) |
| Time | Picoseconds (ps) | Femtoseconds (fs) |
| Energy | Electron-volts (eV) | kcal/mol |
| Temperature | Kelvin | Kelvin |
| Pressure | Bars | Atmospheres |
| Force | eV/Å | kcal/(mol·Å) |
| Timestep typical value | 0.001 (1 fs) | 1.0 (1 fs) |

Use `metal` units for metals, ceramics, and most solid-state systems. Use `real` units for molecular systems with 
CHARMM/AMBER force fields. Most machine-learning potentials trained on DFT data (which uses eV) are used with `metal` 
units.

### Boundary Conditions

```lammps
boundary p p p    # Periodic in x, y, z (most common for bulk)
boundary p p f    # Periodic in x,y; fixed walls in z (for slabs/surfaces)
boundary p p s    # Periodic in x,y; shrink-wrapped in z
```

`p` = periodic, `f` = fixed walls, `s` = shrink-wrapped (box adjusts to fit atoms).

Most bulk crystal simulations use `p p p`. Slab geometry (surface calculations) typically uses `p p f` or `p p s`.

### Atom Style

The `atom_style` command sets what information is stored per atom. Choose the simplest style that your simulation needs:

```lammps
atom_style atomic    # Position, velocity, force only. Use for metals.
atom_style charge    # Adds per-atom charge. Use for ionic systems with Coulombics.
atom_style full      # Adds charge, bonds, angles, dihedrals. Use for molecules.
```

---

## Part 4: System Definition — Creating or Reading Structures

### Method 1: Create a Lattice Directly in LAMMPS

LAMMPS can place atoms on a crystal lattice without needing an input file:

```lammps
lattice fcc 3.52                     # FCC lattice, lattice constant 3.52 Å (Ni)
region simbox block 0 10 0 10 0 10   # 10×10×10 unit cells
create_box 1 simbox                  # 1 atom type
create_atoms 1 box                   # Fill the region with lattice atoms
mass 1 58.693                        # Nickel mass
```

Supported lattice types include `fcc`, `bcc`, `hcp`, `sc`, `diamond`, `sq`, `sq2`, `hex`.

For multi-component systems, specify multiple atom types:

```lammps
lattice bcc 2.87
create_box 2 simbox
create_atoms 1 box                            # Fill with Fe (type 1)
set region simbox type/ratio 2 0.1 12345      # Randomly change 10% to Cr (type 2)
mass 1 55.845    # Fe
mass 2 51.996    # Cr
```

### Method 2: Read a Data File

For structures that can't be created on a simple lattice — interface geometries, amorphous systems, molecules, or 
structures from DFT output — you provide a LAMMPS data file with the `read_data` command:

```lammps
read_data my_structure.lammps
```

A LAMMPS data file is a plain-text file with a specific format. Here is a minimal example for a 2-atom BCC iron unit cell:

```
# BCC Fe unit cell (LAMMPS data file)

2 atoms
1 atom types

0.0 2.87 xlo xhi
0.0 2.87 ylo yhi
0.0 2.87 zlo zhi

Masses

1 55.845

Atoms # atomic

1 1 0.000 0.000 0.000
2 1 1.435 1.435 1.435
```

Lines starting with `#` are comments. OVITO can export LAMMPS data files from any supported input format, which is the 
easiest way to convert DFT output (POSCAR, XYZ, etc.) into LAMMPS format.

### Method 3: Read a Restart File

To continue a previously interrupted simulation:

```lammps
read_restart restart.10000
```

LAMMPS periodically writes `.restart` binary files that capture the complete simulation state.

### Replicating the Simulation Cell

To expand a small unit cell into a larger supercell:

```lammps
replicate 5 5 5    # Create a 5×5×5 supercell
```

---

## Part 5: Interatomic Potentials (Force Fields)

The interatomic potential is the most scientifically important choice in any classical MD simulation. It determines 
how atoms interact — their equilibrium structure, vibrational properties, defect energetics, melting point, and everything
else. Choosing the right potential for your material and property of interest requires careful consideration.

### Lennard-Jones (LJ)

The simplest pair potential. Only appropriate for noble gases and as a model system. Not suitable for real materials:

```lammps
pair_style lj/cut 10.0              # LJ with 10 Å cutoff
pair_coeff 1 1 0.0104 3.40         # ε (eV), σ (Å) for Ar-Ar
```

### Embedded Atom Method (EAM)

The standard potential for metals (Cu, Ni, Fe, Al, Au, etc.). EAM captures metallic bonding through an embedding energy 
that depends on the local electron density. It is fast, well-validated, and many parameterizations exist for pure metals 
and alloys.

```lammps
pair_style eam/alloy
pair_coeff * * FeNiCr.eam.alloy Fe Ni Cr
```

EAM potential files (`.eam`, `.alloy`, `.fs`) are available from the 
[NIST Interatomic Potentials Repository](https://www.ctcms.nist.gov/potentials/).

### Modified EAM (MEAM)

An extension of EAM that adds angular-dependent terms, making it suitable for covalent elements (Si, C, B) and 
intermetallic compounds:

```lammps
pair_style meam
pair_coeff * * library.meam Si C SiC.meam Si C
```

### Stillinger-Weber (SW) and Tersoff

Bond-order potentials suitable for covalently bonded systems like silicon, germanium, and carbon:

```lammps
# Stillinger-Weber for Si
pair_style sw
pair_coeff * * Si.sw Si

# Tersoff for SiC
pair_style tersoff
pair_coeff * * SiC.tersoff Si C
```

### ReaxFF

A reactive force field that allows bond formation and breaking during the simulation. It can model chemical reactions, 
oxidation, combustion, and failure of materials. Computationally more expensive than EAM/Tersoff:

```lammps
pair_style reaxff NULL
pair_coeff * * ffield.reax C H O N
fix 1 all qeq/reaxff 1 0.0 10.0 1.0e-6 reaxff    # Charge equilibration
```

### Machine Learning Potentials

A rapidly growing category. These potentials are trained on DFT energy/force data and can achieve near-DFT accuracy at 
a fraction of the cost. LAMMPS supports several ML potential frameworks natively:

```lammps
# SNAP (Spectral Neighbor Analysis Potential)
pair_style snap
pair_coeff * * W.snapcoeff W.snapparam W

# Behler-Parrinello neural network potentials (via N2P2)
pair_style hdnnp 6.0 dir /path/to/n2p2 showew no showewsum 0 resetew no maxew 200
pair_coeff * * W H

# DeePMD (deep learning potential)
pair_style deepmd frozen_model.pb
pair_coeff * *
```

> For those coming from DFT: ML potentials are typically trained by running a series of DFT single-point calculations on 
a diverse set of configurations, then fitting a model to reproduce those energies, forces, and stresses. Tools like VASP, 
CP2K, or Quantum Espresso generate the training data; tools like PACEMAKER, N2P2, FitSNAP, or DeePMD-kit train the model; 
and LAMMPS runs the classical MD with that model.

### Choosing the Right Potential

| System type | Recommended potential |
|------------|----------------------|
| Pure metals (Cu, Ni, Fe, Al) | EAM |
| Metal alloys (FeNi, CuZn) | EAM/alloy or MEAM |
| Covalent semiconductors (Si, Ge) | SW or Tersoff |
| Carbon systems (graphene, diamond) | AIREBO or Tersoff |
| Ionic systems (NaCl, MgO, Al₂O₃) | Buckingham + Coulomb |
| Reactive systems (combustion, oxidation) | ReaxFF |
| Near-DFT accuracy on a specific material | ML potential (SNAP, NNP, ACE, DeePMD) |
| Learning/testing/prototyping | Lennard-Jones |

---

## Part 6: Simulation Settings — Fixes, Computes, and Output

### Groups

Before setting up fixes and computes, you often need to define **groups** — subsets of atoms that a command acts on. The
predefined group `all` includes every atom in the system:

```lammps
group mobile region mobile_region     # Atoms in a spatial region
group frozen subtract all mobile      # Everything else
group Fe type 1                       # All type-1 atoms
```

### Fixes — The Heart of LAMMPS Settings

In LAMMPS, nearly everything that happens during a simulation is implemented as a **fix**. This includes time integration
(advancing atom positions), thermostats, barostats, walls, constraints, and output. Multiple fixes can be active 
simultaneously.

The syntax is:
```lammps
fix fix-ID group-name fix-style fix-options
```

### Time Integration Fixes

Every MD run requires exactly one time integration fix acting on your atoms:

```lammps
fix 1 all nve                                    # Microcanonical (NVE): constant energy
fix 1 all nvt temp 300 300 0.1                   # Canonical (NVT): constant temperature
fix 1 all npt temp 300 300 0.1 iso 0.0 0.0 1.0  # Isothermal-isobaric (NPT)
```

### Statistical Ensembles

Understanding which ensemble to use is fundamental to molecular dynamics:

**NVE (microcanonical):** Number of atoms (N), Volume (V), and Energy (E) are all fixed. The system evolves under pure 
Newtonian mechanics with no temperature or pressure control. Use for: production runs after equilibration, checking energy
conservation (debugging), dynamics where coupling to a thermostat would be unphysical.

```lammps
fix 1 all nve
```

**NVT (canonical):** Number of atoms (N) and Volume (V) are fixed; Temperature (T) is held constant by a thermostat. The 
system exchanges heat with a virtual heat bath. Use for: most equilibration runs, computing temperature-dependent properties 
at a fixed volume.

```lammps
# Nosé-Hoover thermostat: temp start end Tdamp
fix 1 all nvt temp 300.0 300.0 0.1
```

The three numbers are: starting temperature, ending temperature (allowing temperature ramps), and the thermostat damping
time (Tdamp). A good value for Tdamp is ~100 timesteps worth of time. For metal units with a 1 fs timestep (0.001 ps), 
Tdamp = 0.1 ps (100 timesteps) is usually good.

**NPT (isothermal-isobaric):** Temperature and Pressure are both controlled; the simulation box size fluctuates. Use for: 
computing equilibrium lattice parameters, pressure-dependent properties, fully relaxing the cell at a given T and P.

```lammps
# Nosé-Hoover thermostat + barostat
# Isotropic pressure control (iso)
fix 1 all npt temp 300.0 300.0 0.1 iso 0.0 0.0 1.0

# Anisotropic pressure control (each axis independent)
fix 1 all npt temp 300.0 300.0 0.1 aniso 0.0 0.0 1.0

# Triclinic (full stress tensor control, including shear)
fix 1 all npt temp 300.0 300.0 0.1 tri 0.0 0.0 1.0
```

The pressure control parameters are: starting pressure, ending pressure, and damping time (Pdamp). A good value is ~1000 
timesteps worth of time. For metal units: Pdamp = 1.0 ps.

### Initializing Velocities

Before running NVT or NVE, you must give atoms initial velocities. This creates a Maxwell-Boltzmann distribution at the 
specified temperature:

```lammps
velocity all create 300.0 12345 mom yes rot yes dist gaussian
```

Arguments: group, `create`, target temperature, random seed (any integer), `mom yes` removes net momentum 
(prevents center-of-mass drift), `rot yes` removes angular momentum, `dist gaussian` uses a Gaussian distribution.

### Thermodynamic Output

The `thermo` and `thermo_style` commands control what is printed to the screen and log file:

```lammps
thermo 100    # Print every 100 timesteps
thermo_style custom step time temp pe ke etotal press vol lx ly lz
```

**Common `thermo_style` keywords:**

| Keyword | Meaning |
|---------|---------|
| `step` | Timestep number |
| `time` | Simulation time (time units) |
| `temp` | Temperature (K) |
| `pe` | Potential energy |
| `ke` | Kinetic energy |
| `etotal` | Total energy (pe + ke) |
| `press` | Pressure (isotropic) |
| `vol` | Volume of simulation box |
| `lx, ly, lz` | Box dimensions |
| `pxx, pyy, pzz` | Diagonal stress tensor components |
| `pxy, pxz, pyz` | Off-diagonal stress tensor components |
| `density` | Mass density |

### Dump Files — Trajectory Output

The `dump` command writes atomic positions (and other per-atom quantities) to a file at regular intervals. This is the 
trajectory file you will load into OVITO for visualization:

```lammps
# Standard LAMMPS dump format
dump 1 all atom 1000 dump.lammpstrj

# More informative: include velocities and forces
dump 2 all custom 1000 dump.custom.lammpstrj &
    id type x y z vx vy vz fx fy fz pe

# XYZ format (readable by many tools)
dump 3 all xyz 1000 dump.xyz
```

**Per-atom output keywords for the `custom` style:**

| Keyword | Output |
|---------|--------|
| `id` | Atom ID number |
| `type` | Atom type number |
| `element` | Element name (requires `dump_modify element` mapping) |
| `x y z` | Wrapped atomic positions |
| `xu yu zu` | Unwrapped positions (for diffusion analysis) |
| `vx vy vz` | Velocities |
| `fx fy fz` | Forces |
| `pe` | Per-atom potential energy |
| `stress` | Per-atom stress tensor |
| `c_mycompute` | Any user-defined compute output |

To label dump output with element names instead of type numbers (important for OVITO):

```lammps
dump_modify 1 element Cu Ni    # Map type 1 → Cu, type 2 → Ni
```

### Restart Files

Restart files capture the complete simulation state (atom positions, velocities, fix state) and allow you to resume a 
crashed or extended simulation:

```lammps
restart 10000 restart.sim    # Write restart every 10,000 steps
```

This creates alternating files `restart.sim.10000`, `restart.sim.20000`, etc. To resume:

```lammps
read_restart restart.sim.10000
```

### Computes — Calculating Properties During a Run

The `compute` command defines a calculation that LAMMPS performs every timestep (or on demand):

```lammps
# Per-atom kinetic energy
compute myke all ke/atom

# Per-atom potential energy
compute mype all pe/atom

# Mean squared displacement (for diffusion)
compute mymsd all msd

# Radial distribution function (g(r))
compute myrdf all rdf 50 1 1 1 2 2 2    # 50 bins, pairs 1-1, 1-2, 2-2

# Common neighbor analysis
compute mycna all cna/atom 3.1          # Cutoff 3.1 Å

# Stress per atom
compute mystress all stress/atom NULL   # NULL = uses all pair interactions
```

To output a compute via `thermo`:
```lammps
thermo_style custom step temp c_mymsd[4]    # MSD component [4] = total
```

To output per-atom computes via `dump`:
```lammps
dump 1 all custom 1000 dump.out id type x y z c_mype c_myke
```

---

## Part 7: Running Simulations

### Energy Minimization (Geometry Optimization)

Before running any MD, always minimize the structure to remove any forces from the initial configuration — especially 
if atoms were placed on a lattice or converted from a DFT file with slightly different lattice parameters than what your 
potential predicts.

```lammps
minimize 1.0e-6 1.0e-8 10000 100000
```

Arguments: energy convergence tolerance, force convergence tolerance (eV/Å in metal units), maximum number of iterations, 
maximum number of force evaluations.

The default minimization algorithm is conjugate gradient (`min_style cg`). For systems near a saddle point, `min_style hftn`
(Hessian-free truncated Newton) can be more robust.

### Running MD

```lammps
timestep 0.001     # 1 fs (in metal units)
run 100000         # 100,000 steps = 100 ps
```

**Choosing the timestep:** The timestep must be small enough that the equations of motion are integrated accurately. A rule 
of thumb is that the timestep should be about 1/20th to 1/100th of the shortest vibrational period in your system. For most 
metals with EAM: 1–2 fs is typical. For covalent systems with light atoms (H, C): 0.5–1 fs. For ReaxFF: 0.1–0.5 fs.

> Using too large a timestep causes energy drift and simulation instability. Always check that your total energy is stable 
(for NVE) or that temperature converges properly (for NVT) at the start of a simulation.

### Running Multiple Stages in One Script

LAMMPS scripts can contain multiple `run` commands, changing settings between them. This is how you chain equilibration and 
production stages:

```lammps
# Stage 1: NVT equilibration at 300 K
fix 1 all nvt temp 300.0 300.0 0.1
run 50000

# Stage 2: Remove thermostat, run NVE production
unfix 1
fix 2 all nve
run 100000
```

### The minimize → NVT → NVE Workflow

The standard workflow for a production MD run is:

1. **Minimize** the initial structure
2. **NVT equilibration** — run with a thermostat until temperature, energy, and pressure are stable (typically 10–100 ps 
depending on the system)
3. **NVE production** — run without a thermostat to collect statistics in the microcanonical ensemble

---

## Part 8: A Complete Annotated Example

The following script performs a full equilibration and production MD run for a nickel FCC crystal:

```lammps
# ============================================================
# LAMMPS input script: NVT equilibration + NVE production
# System: FCC Nickel, EAM potential
# Units: metal (eV, Å, ps, K, bar)
# ============================================================

# === INITIALIZATION ===
units metal
atom_style atomic
boundary p p p    # Fully periodic (bulk simulation)

# === SYSTEM DEFINITION ===
# Build 8×8×8 FCC supercell of Ni (a = 3.52 Å)
lattice fcc 3.52
region simbox block 0 8 0 8 0 8    # 8x8x8 unit cells = 2048 atoms
create_box 1 simbox
create_atoms 1 box
mass 1 58.693    # Ni mass in g/mol

# === FORCE FIELD ===
pair_style eam
pair_coeff * * Ni_u3.eam

# Verify neighbor list settings (optional but good practice)
neighbor 2.0 bin                           # Skin distance 2 Å
neigh_modify every 1 delay 0 check yes     # Rebuild every step if needed

# === ENERGY MINIMIZATION ===
# Always minimize before MD to relieve initial lattice stress
thermo 100
thermo_style custom step pe fmax
minimize 1.0e-8 1.0e-10 10000 100000

# Reset the timestep counter after minimization
reset_timestep 0

# === VELOCITY INITIALIZATION ===
# Assign velocities at 300 K using a random seed
velocity all create 300.0 4928459 mom yes rot yes dist gaussian

# === STAGE 1: NVT EQUILIBRATION (50 ps) ===
fix 1 all nvt temp 300.0 300.0 0.1    # Tdamp = 0.1 ps
thermo 500
thermo_style custom step time temp pe ke etotal press vol
dump 1 all custom 1000 dump.equil.lammpstrj &
    id type x y z pe
dump_modify 1 element Ni
timestep 0.001    # 1 fs
run 50000         # 50 ps

# === STAGE 2: NVE PRODUCTION (100 ps) ===
unfix 1           # Remove thermostat
fix 2 all nve     # Microcanonical ensemble

# Update dump for production trajectory
undump 1
dump 2 all custom 1000 dump.prod.lammpstrj &
    id type x y z vx vy vz pe

# Compute mean squared displacement for diffusion analysis
compute mymsd all msd com yes

# Compute RDF for structural analysis
compute myrdf all rdf 100 1 1
fix rdfavg all ave/time 10 100 1000 c_myrdf[*] &
    file rdf.dat mode vector

thermo 500
thermo_style custom step time temp pe ke etotal press c_mymsd[4]
run 100000    # 100 ps production

# === SAVE FINAL STATE ===
write_data ni_final.lammps      # Save final positions for restart/analysis
write_restart final.restart
```

---

## Part 9: Reading and Understanding Log File Output

LAMMPS writes all thermodynamic output to `log.lammps` (a copy of what it prints to the screen). Learning to read this 
file is essential for monitoring simulations.

A typical log file section looks like:

```
Step  Temp      PotEng       KinEng     TotEng      Press
0     0.000000  -3590.1234   0.000000   -3590.1234  -25431.23
500   298.847145 -3564.8921  192.2834   -3372.6087  -231.45
1000  300.134822 -3563.4512  193.1012   -3370.3500  142.89
...
```

**What to look for:**

- **Temperature** should stabilize around the target for NVT. Wild fluctuations indicate a timestep that is too large or 
a thermostat damping that is too aggressive.
- **Total energy (`etotal`)** should be conserved in NVE. Steady drift upward (energy increase) indicates numerical 
instability from too-large a timestep.
- **Potential energy (`pe`)** should decrease and stabilize during minimization. A large jump during the first few MD 
steps after minimization indicates the atoms were not fully relaxed.
- **Pressure** will fluctuate strongly in any MD simulation. Only the time-average is meaningful. Large persistent 
non-zero pressure in NPT means the box has not yet equilibrated.

---

## Part 10: Output Files and Post-Processing

### Dump File Format

LAMMPS dump files (`.lammpstrj`) are plain-text files with a specific header format that OVITO reads natively:

```
ITEM: TIMESTEP
0
ITEM: NUMBER OF ATOMS
2048
ITEM: BOX BOUNDS pp pp pp
0.0000000000000000e+00 2.8160000000000001e+01
0.0000000000000000e+00 2.8160000000000001e+01
0.0000000000000000e+00 2.8160000000000001e+01
ITEM: ATOMS id type x y z pe
1 1 0.000000 0.000000 0.000000 -3.516234
2 1 1.760000 1.760000 0.000000 -3.519847
...
```

Load this in OVITO using **File → Load File** and selecting the `.lammpstrj` file. OVITO detects the format automatically.

### Analyzing the Log File with Python

The thermodynamic data from `log.lammps` can be read and plotted easily with Python:

```python
import numpy as np
import matplotlib.pyplot as plt

# Parse the log file
steps, temps, energies = [], [], []
reading = False
with open('log.lammps', 'r') as f:
    for line in f:
        if 'Step' in line and 'Temp' in line:
            reading = True
            continue
        if 'Loop time' in line:
            reading = False
        if reading:
            try:
                vals = line.split()
                steps.append(int(vals[0]))
                temps.append(float(vals[1]))
                energies.append(float(vals[2]))  # Adjust index to match your thermo_style
            except:
                pass

# Plot temperature vs time
timestep_fs = 1.0    # 1 fs per step
time_ps = np.array(steps) * timestep_fs / 1000.0

plt.figure(figsize=(10,4))
plt.subplot(1, 2, 1)
plt.plot(time_ps, temps)
plt.xlabel('Time (ps)')
plt.ylabel('Temperature (K)')
plt.title('Temperature vs Time')

plt.subplot(1, 2, 2)
plt.plot(time_ps, energies)
plt.xlabel('Time (ps)')
plt.ylabel('Potential Energy (eV)')
plt.title('Energy vs Time')

plt.tight_layout()
plt.savefig('thermo.png', dpi=150)
```

---

## Part 11: Running LAMMPS in Parallel on HPC Systems (ISAAC-NG)

For large simulations or to speed up smaller ones, LAMMPS parallelizes efficiently across multiple CPU cores using 
MPI. See the ISAAC-NG tutorial for how to access the cluster. Here is an example SLURM job script for running LAMMPS 
on ISAAC-NG:

```bash
#!/bin/bash
#SBATCH -J lammps_md              # Job name
#SBATCH -A ACF-UTK0011            # Project account
#SBATCH --nodes=2                 # 2 nodes
#SBATCH --ntasks-per-node=48      # 48 cores per node = 96 total MPI ranks
#SBATCH --partition=campus        # Use the campus partition
#SBATCH --qos=campus
#SBATCH --time=0-12:00:00         # 12 hour wall time
#SBATCH --output=lammps.o%j
#SBATCH --error=lammps.e%j

# Load LAMMPS (check what modules are available with 'module avail lammps')
module load LAMMPS/2023

# Run LAMMPS in parallel
srun lmp -in in.my_simulation -log log.lammps

# Or with explicit MPI launch
mpirun -np 96 lmp -in in.my_simulation
```

**LAMMPS scaling:** LAMMPS scales well up to the point where the communication overhead between processors dominates 
over the computation. A rough guide: plan for at least ~500–1000 atoms per CPU core. Very small systems will not benefit
from many processors.

**Speeding up LAMMPS with GPU acceleration:** If your potential supports it (EAM, LJ, Tersoff, ReaxFF all have GPU 
implementations), you can use LAMMPS's GPU package:

```bash
# Request a GPU node and use GPU acceleration
#SBATCH --partition=campus-gpu
#SBATCH --gpus=1

srun lmp -sf gpu -pk gpu 1 -in in.my_simulation
```

---

## Part 12: Practical Workflows

### Workflow 1: Lattice Parameter and Elastic Constants

Determine the equilibrium lattice parameter of a material using NPT MD:

```lammps
units metal
atom_style atomic
boundary p p p

lattice fcc 3.52    # Initial guess for Ni
region box block 0 6 0 6 0 6
create_box 1 box
create_atoms 1 box
mass 1 58.693

pair_style eam
pair_coeff * * Ni_u3.eam

minimize 1e-8 1e-10 10000 100000
velocity all create 300.0 12345 mom yes rot yes

# NPT to find equilibrium volume at 300K, 0 bar
fix 1 all npt temp 300 300 0.1 iso 0.0 0.0 1.0
thermo 100
thermo_style custom step temp press vol lx ly lz
run 50000

# Report the final box dimensions (convert to lattice constant)
# lx/6 gives the equilibrium lattice constant for a 6×6×6 cell
```

### Workflow 2: Melting Point Determination

Heat a crystal slowly and watch for the latent heat jump:

```lammps
# Ramp temperature from 300 K to 2000 K over 1 ns
fix 1 all npt temp 300.0 2000.0 0.1 iso 0.0 0.0 1.0
run 1000000

# Look for discontinuity in energy/volume vs temperature in log file
```

### Workflow 3: Computing the Radial Distribution Function (RDF)

```lammps
# Run equilibrated NVT, then compute RDF
fix 1 all nvt temp 300 300 0.1
compute myrdf all rdf 200 1 1    # 200 bins, type-1 to type-1

# Average RDF over 1000 samples taken every 10 steps
fix rdffix all ave/time 10 100 1000 c_myrdf[*] &
    file rdf_output.dat mode vector
run 10000
```

The output file `rdf_output.dat` contains columns `r`, `g(r)` which you can plot to compare with experimental X-ray 
or neutron diffraction data.

### Workflow 4: Using a DFT-Trained ML Potential

This workflow bridges DFT output with classical MD (LAMMPS) using a machine-learned potential:

```lammps
units metal
atom_style atomic
boundary p p p

# Read in a structure from DFT output (converted to LAMMPS format)
read_data my_dft_structure.lammps

# Load a SNAP potential trained on DFT data
pair_style snap
pair_coeff * * W.snapcoeff W.snapparam W

# Minimize with the ML potential
minimize 1e-6 1e-8 1000 10000

# NVT production at 1000 K
velocity all create 1000.0 9872 mom yes rot yes
fix 1 all nvt temp 1000 1000 0.1
dump 1 all custom 500 dump.W.lammpstrj id type x y z pe
dump_modify 1 element W
run 200000    # 200 ps
```

Post-process the resulting `dump.W.lammpstrj` in OVITO using the Common Neighbor Analysis modifier to track structural 
changes.

---

## Part 13: Tips, Best Practices, and Common Mistakes

**Always minimize before running MD.** No matter how carefully a structure was built, atoms on a perfect lattice will 
not exactly match the equilibrium configuration for your potential (especially if you're using a potential fitted to DFT
data with slightly different lattice parameters than experiment). A minimization step removes this stress and prevents 
large force spikes at the start of the run.

**Match units to your potential.** Using `units real` with an EAM potential (which expects `metal` units) will give 
completely wrong results with no error. Check your potential documentation and set units accordingly.

**Check energy conservation in NVE.** Run a short NVE test after equilibration. If total energy drifts systematically
upward, your timestep is too large. Halve it and try again.

**The thermostat damping parameter matters.** Tdamp should be around 100 timesteps. In metal units with a 1 fs timestep: 
use Tdamp = 0.1 (100 × 0.001 ps). Too small → erratic temperature oscillations. Too large → very slow temperature equilibration.

**Use unwrapped coordinates for diffusion.** For computing mean squared displacement or diffusion coefficients, use `xu yu zu` 
in your dump output instead of `x y z`. Wrapped coordinates reset when atoms cross periodic boundaries, breaking the MSD calculation.

**Don't run production in the same run as equilibration.** Use separate run blocks (or separate scripts with restart files) 
so you have clear records of when equilibration ended and production began.

**New directories for each simulation.** LAMMPS overwrites output files without warning. Create a new directory for each new
run, or give dump files unique names with variables.

**Check your structure visually before running.** Always load your initial data file or a dump from the first few steps into
OVITO to confirm atoms are positioned correctly, the box looks right, and no atoms are unreasonably close together.

**Citation.** When publishing work that used LAMMPS, cite:
> A.P. Thompson et al., *LAMMPS - A flexible simulation tool for particle-based materials modeling at the atomic, meso, and
continuum scales*, Comp. Phys. Comm. 271, 108171 (2022).

---

## Quick Reference

### Key Commands at a Glance

| Command | Purpose |
|---------|---------|
| `units metal` | Set unit system |
| `boundary p p p` | Set boundary conditions |
| `atom_style atomic` | Set atom data format |
| `lattice fcc 3.52` | Define a crystal lattice |
| `read_data file.lammps` | Read atom positions from file |
| `pair_style eam` | Choose potential type |
| `pair_coeff * * file.eam` | Load potential file |
| `mass 1 58.693` | Set atomic mass for type 1 |
| `group name type 1` | Define an atom group |
| `velocity all create 300 12345` | Initialize velocities |
| `fix 1 all nve` | Microcanonical time integration |
| `fix 1 all nvt temp T T Tdamp` | Canonical (NVT) thermostat |
| `fix 1 all npt temp T T Td iso P P Pd` | Isothermal-isobaric (NPT) |
| `timestep 0.001` | Set timestep (metal: ps) |
| `thermo 100` | Print thermo every 100 steps |
| `thermo_style custom step temp pe` | Choose output columns |
| `dump 1 all custom N file.trj id type x y z` | Write trajectory |
| `dump_modify 1 element Cu` | Add element labels |
| `minimize 1e-6 1e-8 10000 100000` | Energy minimization |
| `run 100000` | Run MD for N steps |
| `write_data final.lammps` | Save final structure |
| `write_restart final.restart` | Save restart file |
| `read_restart file.restart` | Resume from restart |
| `compute id all pe/atom` | Define per-atom compute |
| `fix id all ave/time N Nfreq Nevery file.dat` | Time-average a compute |

### Statistical Ensembles

| Ensemble | Fixed | Controlled | Use for |
|---------|-------|-----------|---------|
| NVE | N, V, E | nothing | Production MD, energy conservation check |
| NVT | N, V | T | Equilibration, temperature-dependent properties |
| NPT | N | T, P | Finding equilibrium volume, lattice parameters |
| NPH | N, H | P | Constant enthalpy (rarely needed) |

### Thermostat/Barostat Damping Guidelines (metal units, 1 fs timestep)

| Parameter | Recommended Value | Notes |
|-----------|-------------------|-------|
| Tdamp | 0.1 ps | 100 × timestep |
| Pdamp | 1.0 ps | 1000 × timestep |

### Where to Get Interatomic Potentials

- **NIST Interatomic Potentials Repository:** [https://www.ctcms.nist.gov/potentials/](https://www.ctcms.nist.gov/potentials/) 
— the standard source for EAM, MEAM, and Tersoff potentials for metals and alloys
- **OpenKIM:** [https://openkim.org](https://openkim.org) — curated database of verified potentials, directly callable from
LAMMPS via the `kim_init` command
- **Molecular Dynamics Potential Repository:** embedded in LAMMPS at `lammps/potentials/`
- **NIST JARVIS:** [https://jarvis.nist.gov](https://jarvis.nist.gov) — ML potentials trained on DFT data

### Where to Get Help

- **Official documentation:** [https://docs.lammps.org/](https://docs.lammps.org/)
- **LAMMPS mailing list:** [https://lammps.org/mail.html](https://lammps.org/mail.html)
- **LAMMPS forum (GitHub Discussions):** [https://github.com/lammps/lammps/discussions](https://github.com/lammps/lammps/discussions)
- **Example scripts:** Included with every LAMMPS installation at `lammps/examples/`
- **LAMMPS tutorials site:** [https://lammpstutorials.github.io](https://lammpstutorials.github.io)
