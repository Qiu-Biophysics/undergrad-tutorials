# Beginner's Guide to OVITO
## Open Visualization Tool for Molecular Dynamics and DFT Simulations

---

## What Is OVITO?

OVITO (Open Visualization Tool) is a scientific visualization and analysis software designed specifically for atomistic 
and particle-based simulation data. Developed since 2009 by OVITO GmbH in Germany, it has been cited in over 18,000 
research publications and is one of the most widely used post-processing tools in computational materials science.

Unlike general-purpose 3D software, OVITO is purpose-built for the kinds of data that come out of molecular dynamics 
(MD) and density functional theory (DFT) simulations — things like trajectories of thousands of atoms over time, per-atom 
energies and forces, charge densities, and crystal structure analysis. It handles everything from small ab initio cells 
with a handful of atoms to large-scale classical MD simulations with over 100 million particles.

This guide covers everything you need to get started using OVITO to visualize and analyze your simulation data, with a 
particular focus on workflows relevant to DFT and molecular dynamics.

---

## Part 1: Installation and Versions

### Two Versions of OVITO

**OVITO Basic (Free/Open Source):** Available at no cost for academic and personal use. Includes the core visualization 
and analysis tools, the full Python scripting interface, and the OpenGL renderer. This is sufficient for the vast majority 
of everyday analysis tasks.

**OVITO Pro (Licensed):** A commercial product aimed at academic and industrial research teams. Adds advanced features 
including ray-traced rendering (Tachyon, OSPRay, VisRTX), pipeline branching for side-by-side comparison of multiple 
simulations, the LAMMPS script data source, ASE trajectory support, and remote rendering on HPC clusters.

> For this tutorial, all features described are available in OVITO Basic unless explicitly marked as **[Pro]**.

### Installing OVITO

Download OVITO from [https://www.ovito.org/download/](https://www.ovito.org/download/).

Packages are available for **Windows**, **macOS**, and **Linux**. Simply download the installer for your operating system 
and follow the on-screen instructions. No additional dependencies or configuration are required — OVITO is a self-contained
application.

On Linux, the download is typically a compressed archive. Extract it and run the `ovito` executable directly:

```bash
tar -xzf ovito-basic-*.tar.gz
cd ovito-basic-*/
./ovito
```

### The OVITO Python Package

OVITO also provides a standalone Python package (`ovito`) that can be installed via pip, allowing you to run OVITO 
pipelines as Python scripts without the graphical interface — ideal for batch processing large numbers of simulation
frames on an HPC cluster:

```bash
pip install ovito
```

Python scripting is covered in Part 9 of this guide.

---

## Part 2: The OVITO Interface

When you first open OVITO, you are presented with a clean interface divided into several key areas.

### The Viewport

The large central window is the **3D viewport** — the interactive display where your simulation is visualized. By default,
OVITO shows a 2×2 grid of viewports (Top, Front, Left, and Perspective), but you can maximize any single viewport by 
clicking the maximize button in its corner. Most users work primarily in the **Perspective viewport**.

**Navigating the viewport:**

| Action | Method |
|--------|--------|
| Rotate | Hold right mouse button + drag, or Ctrl + left mouse button + drag |
| Pan | Hold middle mouse button + drag, or Shift + right mouse button |
| Zoom | Scroll mouse wheel, or hold right mouse button + drag up/down |
| Reset view to fit all atoms | Press `F` or click the zoom-to-fit button |
| Orbit around a specific atom | Right-click on an atom → "Focus camera on this particle" |

The small coordinate axis indicator in the lower-left of each viewport shows your current orientation. A thin yellow 
border around a viewport indicates it is the **active viewport**, which is the one used for rendering.

### The Pipeline Editor (Right Panel)

The **pipeline editor** is the heart of OVITO's workflow. It lives in the right-side command panel and displays three 
sections stacked vertically:

1. **Visual Elements (top)** — controls how your data is drawn (particle appearance, bond style, simulation cell visibility,
etc.)
2. **Modifications (middle)** — the list of modifiers you've applied, processed in bottom-up order
3. **Data source (bottom)** — the imported simulation file

When you first load a file, only the Data source and Visual Elements sections are visible. The Modifications section 
appears as soon as you add your first modifier.

> **Tip:** Modifiers are listed in the Modifications section and processed bottom-up — the modifier at the bottom of 
the list acts first on the data. Each item has a checkbox to temporarily enable or disable it.

### The Command Panel Tabs

The right panel has several tabs at the top:

- **Pipeline** — the main pipeline editor (described above)
- **Render** — rendering settings for producing high-quality images and animations
- **Viewport Layers** — add 2D overlay elements like color legends, scale bars, and coordinate axes

### The Data Inspector

At the bottom of the screen is the **Data Inspector**, accessible by clicking or dragging the tab bar upward. It displays
a table of per-particle properties (position, type, energy, etc.) for the output of your current pipeline — useful for 
verifying that modifiers are working correctly or for reading off specific values.

### The Timeline

When a simulation trajectory with multiple frames is loaded, a timeline and time slider appear at the bottom of the screen.
Use it to scrub through your simulation in real time. The play button runs the animation, and the playback speed can be 
adjusted in the **Animation Settings**.

---

## Part 3: Supported File Formats and DFT Data

OVITO supports a wide range of input file formats used across atomistic simulation codes. For those working with DFT or 
ab initio MD, the most relevant formats are:

| Format | Code | Notes |
|--------|------|-------|
| POSCAR / CONTCAR | VASP | Atomic positions and lattice |
| XDATCAR | VASP | MD trajectory (multiple ionic steps) |
| CHGCAR | VASP | Charge density as voxel grid |
| XSF | XCrySDen / SIESTA | Atoms + optional volumetric data |
| Cube (`.cube`) | Gaussian, GPAW, CP2K | Atoms + electronic density / potential |
| Quantum Espresso input | QE | Atomic positions and cell |
| FHI-aims `geometry.in` / `aims.out` | FHI-aims | Atomic positions and log output |
| CASTEP `.cell` / `.md` / `.geom` | CASTEP | Cell + trajectory |
| XYZ / Extended XYZ | ASE, CP2K, many others | Universal; extended format carries per-atom properties |
| LAMMPS dump | LAMMPS | Classical MD trajectories |
| LAMMPS data | LAMMPS | Topology and initial positions |
| CIF | Crystallography | Crystal structures |
| PDB | Biomolecular codes | Protein/molecule structures |

> **Tip for VASP users:** OVITO directly reads POSCAR, CONTCAR, and XDATCAR files. For an ab initio MD run, load the 
XDATCAR file to visualize the full ionic trajectory. Load CHGCAR to visualize the electron charge density as an isosurface.

> **Tip for CP2K / ASE users:** Export your trajectory in Extended XYZ format (`.extxyz`). This format can carry arbitrary
per-atom properties (forces, energies, Mulliken charges, etc.) as extra columns, which OVITO reads and makes available for 
color-coding and analysis.

### Importing a File

1. Go to **File → Load File** (or click the folder icon in the toolbar).
2. Navigate to your simulation file and select it. OVITO auto-detects the format.
3. For compressed files (`.gz`, `.zst`), OVITO decompresses and reads them automatically.
4. The structure appears in the 3D viewport immediately.

You can also drag and drop files directly onto the OVITO window.

### Loading Trajectories

**Multi-frame files** (XDATCAR, XYZ with multiple frames, LAMMPS dump): OVITO detects multiple frames automatically. 
After loading, the timeline appears at the bottom. Check the **Contains multiple timesteps** box in the file source panel
if OVITO doesn't detect them automatically.

**Numbered file sequences** (e.g., `frame_0.xyz`, `frame_1000.xyz`, …): Load any one file in the sequence. OVITO 
automatically generates a wildcard pattern (e.g., `frame_*.xyz`) and finds all matching files in the same directory, 
assembling them into a trajectory.

**Topology + trajectory pairs** (e.g., LAMMPS data + dump): Select both files simultaneously in the file dialog. 
OVITO detects which is the topology file and which is the trajectory, and automatically inserts a **Load Trajectory** 
modifier to merge them.

---

## Part 4: The Pipeline Concept — Non-Destructive Analysis

The most important idea in OVITO is the **data pipeline**. Rather than modifying your data in place, OVITO applies a 
sequence of operations called **modifiers** to your imported data, passing the result from one modifier to the next.

This design has key advantages:

- **Non-destructive:** Your original simulation data is never modified. You can always undo or adjust any step.
- **Real-time updates:** When you change a modifier's parameters, the visualization updates instantly.
- **Reusable pipelines:** Set up a complex analysis pipeline once, then apply it to a different simulation file by simply
swapping the data source.
- **Order matters:** Modifiers are applied in bottom-up order in the pipeline editor. The modifier at the bottom processes
the raw data first; results flow upward through subsequent modifiers.

### Adding a Modifier

Click the **"Add modification…"** button at the top of the Modifications section. A categorized list of all available 
modifiers appears. Select one to insert it above the currently selected pipeline item. You can also search by name.

Modifiers can be temporarily disabled with their checkbox or permanently removed with the trash button. Drag them up or
down to reorder.

---

## Part 5: Visualizing Atoms — Appearance and Display

### Particle Appearance

When you load a file, OVITO automatically assigns colors and radii to atoms based on their chemical element. These 
defaults come from a built-in element database (using standard CPK coloring and van der Waals or covalent radii).

To customize atom appearance:

1. In the **Visual Elements** section at the top of the pipeline editor, click on **Particles**.
2. The parameter panel opens, where you can adjust the default particle radius and shape (sphere, box, ellipsoid, etc.).
3. To change colors per element type: click the particle types entry under the data source panel. A list of all species 
appears, with individual color and radius controls for each.

### Bonds

OVITO does not create bonds automatically for most formats — you must add them explicitly using the **Create bonds**
modifier. Set a cutoff distance, and OVITO draws bonds between all pairs of atoms within that distance.

For formats that include bond topology (LAMMPS data, PDB, GSD), bonds are imported directly.

Bond appearance is controlled through the **Bonds** visual element in the pipeline editor.

### The Simulation Cell

The simulation cell (unit cell or supercell) is shown as a wireframe box by default. Toggle its visibility using the 
**Simulation cell** visual element. You can change the line width and color in its parameter panel.

For periodic systems, OVITO automatically wraps atoms that have been displaced outside the cell using the **Wrap at 
periodic boundaries** modifier. This is often needed for trajectory visualization where atoms drift across periodic 
boundaries.

### Coloring Atoms by a Property

One of the most powerful visualization techniques is coloring each atom by a computed or loaded per-atom property — 
for example, potential energy, temperature, coordination number, or Mulliken charge. This is done with the **Color coding**
modifier.

1. Add the **Color coding** modifier from the Modifications menu.
2. Under **Input property**, select the property you want to visualize (e.g., `c_pe` for per-atom potential energy from a
LAMMPS dump, or `forces` from an extended XYZ).
3. Choose a **color map** (e.g., Viridis, Jet, Cool-to-Warm).
4. Set the **Start value** and **End value** to define the range. Click **Adjust range** to auto-fit to the data.
5. Add a **Color legend layer** (in the Viewport Layers tab) to display a color bar in your rendered images.

---

## Part 6: Key Analysis Modifiers for MD and DFT

OVITO includes dozens of analysis tools. Here are the most important ones for molecular dynamics and DFT workflows.

### Common Neighbor Analysis (CNA)

The **Common Neighbor Analysis** modifier identifies the local crystal structure around each atom and classifies it into 
categories: FCC, HCP, BCC, Icosahedral, or Other. This is extremely useful for tracking phase transformations, identifying
defects, or isolating grain boundaries.

The modifier colors atoms automatically:

| Color | Structure |
|-------|-----------|
| 🟢 Green | FCC |
| 🔴 Red | HCP |
| 🔵 Blue | BCC |
| 🟡 Yellow | Icosahedral |
| ⚪ White | Unknown / disordered (defects, surfaces, grain boundaries) |

There are four CNA modes:

- **Fixed cutoff** — you specify a cutoff distance manually (must be chosen to lie between the 1st and 2nd nearest-neighbor
shells)
- **Adaptive cutoff** — automatically chooses the cutoff per atom; more robust for distorted structures
- **Interval CNA (i-CNA)** — tests all possible cutoffs; best accuracy at elevated temperatures, but slower
- **Bond-based** — uses the existing bond network rather than a distance cutoff

### Polyhedral Template Matching (PTM)

The **Polyhedral Template Matching** modifier is a more modern and robust alternative to CNA, particularly at elevated 
temperatures and in the presence of strain. It identifies FCC, HCP, BCC, simple cubic, diamond cubic/hexagonal, and 
graphene structures.

Beyond structure identification, PTM additionally computes:

- Per-atom crystal orientation (as a quaternion) — useful for grain orientation mapping
- Local elastic deformation and strain tensor
- Chemical ordering (for alloys)

PTM is generally preferred over CNA for most applications due to its greater reliability.

### Displacement Vectors

The **Displacement vectors** modifier computes the displacement of each atom relative to a reference frame (typically 
the first frame of the trajectory or a separate reference structure). It visualizes the displacement as arrows and writes
the magnitude to a `Displacement magnitude` property that you can use for color-coding.

This is useful for identifying which atoms have moved the most during a simulation — for example, tracking diffusion events
or deformation hotspots.

### Voronoi Analysis

The **Voronoi analysis** modifier computes the Voronoi tessellation of the particle configuration — essentially partitioning
space into regions, one per atom, where each region contains all points closer to that atom than to any other. This provides
per-atom information including:

- **Coordination number** (number of Voronoi faces = number of nearest neighbors)
- **Voronoi cell volume** (related to local atomic density)
- **Voronoi index** (the tuple of face counts, which encodes local structural topology)

For DFT and AIMD, Voronoi analysis is useful for computing per-atom volumes and for analyzing the local environment of 
specific defect sites.

### Wigner-Seitz Defect Analysis

The **Wigner-Seitz defect analysis** modifier identifies point defects (vacancies and interstitials) by comparing the 
current configuration to a perfect reference crystal. It places each atom in the Wigner-Seitz cell of the nearest reference
site. Sites with no atom become **vacancies**; sites with more than one atom indicate **interstitials**.

This is ideal for studying radiation damage cascades, diffusion, or defect formation from DFT-based simulations.

### Identify Diamond Structure

For carbon and silicon systems, the **Identify diamond structure** modifier classifies atoms into cubic diamond, hexagonal 
diamond, or graphitic environments — useful for analyzing phase stability or graphitization in carbon AIMD runs.

### Coordination Analysis

The **Coordination analysis** modifier computes the radial distribution function (RDF) and the coordination number for each
atom, given a specified cutoff radius. The RDF is one of the most fundamental structural analysis tools in condensed matter
physics, and OVITO can compute it and export it to a text file or display it as a chart.

### Expression Select / Compute Property

Two general-purpose modifiers give you the power to work with arbitrary per-atom data from your DFT output:

- **Expression select:** Select atoms using a mathematical expression involving any per-atom property. For example: 
`forces.x > 0.5` to select atoms with large x-forces, or `Potential_Energy < -3.5` to select low-energy sites.
- **Compute property:** Define a new per-atom property using a mathematical formula. For example, compute the magnitude
of the force vector: `sqrt(forces.x^2 + forces.y^2 + forces.z^2)` and save it as a new property called `force_magnitude`,
which you can then color-code.

### Slice

The **Slice** modifier cuts your simulation with an infinite plane, either hiding everything on one side or inserting a 
planar cross-section. This is useful for looking inside 3D structures, or for creating visually clear cross-sectional 
images for publications.

### Construct Surface Mesh

The **Construct surface mesh** modifier identifies the outer surface of a particle system and builds a polyhedral mesh 
representing it. This is useful for visualizing nanoparticle shapes, voids, and grain boundary surfaces.

### Dislocation Analysis (DXA)

For metals and crystals subjected to deformation in classical MD, the **Dislocation analysis (DXA)** modifier extracts 
and visualizes dislocation lines from the atomistic data, computing Burgers vectors and dislocation types automatically. 
Note that this modifier is most relevant for classical MD; DFT cells are typically too small to contain extended dislocations.

---

## Part 7: Rendering High-Quality Images and Animations

### The Rendering Tab

Once you have set up your pipeline, switch to the **Render** tab in the command panel to produce publication-quality 
images or animation videos.

Key settings:

- **Renderer:** Choose the rendering engine (see below)
- **Resolution:** Set image width and height in pixels (e.g., 1920×1080 for HD)
- **Background:** White, transparent, or custom color
- **Output file:** Set a filename and format (PNG, JPEG, EIF, etc.) before rendering

Click **Render active viewport** to produce a still image of whatever the current frame and camera angle shows.

### Rendering Engines

| Engine | Quality | Speed | Notes |
|--------|---------|-------|-------|
| OpenGL | Good | Very fast | Default. Matches the interactive viewport exactly |
| Tachyon **[Pro]** | Excellent | Medium | Ray-traced; supports shadows and ambient occlusion |
| OSPRay **[Pro]** | Excellent | Medium | Ray-traced; excellent for surfaces and volumetric data |
| VisRTX **[Pro]** | Excellent | Fast (GPU) | Hardware-accelerated ray tracing on NVIDIA GPUs |

For OVITO Basic users, the OpenGL renderer produces clean, high-resolution images suitable for publication — especially
when you set a high pixel resolution.

### Viewport Layers — Adding Annotations to Images

Before rendering, you can add 2D overlay elements to your images via the **Viewport Layers** tab:

- **Color legend** — a gradient bar showing the color mapping scale, labeled with the property name and range
- **Scale bar** — displays a distance scale bar (e.g., "2 nm") in the corner
- **Coordinate tripod** — shows XYZ axis orientation
- **Text overlay** — adds custom text (simulation name, timestep, temperature, etc.)
- **Python script layer [Pro]** — draws custom graphics using Python

### Rendering Animations

To render a full trajectory as a video:

1. In the Render settings, change the mode from **Active frame** to **Complete animation**.
2. Set the output filename with a video extension (`.avi`, `.mp4`).
3. Choose the frame rate (fps) in the **Animation Settings** dialog.
4. Click **Render active viewport** to start rendering all frames.

Alternatively, render all frames as individual PNG files and combine them with ffmpeg:

```bash
ffmpeg -framerate 24 -i frame_%04d.png -c:v libx264 -pix_fmt yuv420p output.mp4
```

---

## Part 8: Working with DFT-Specific Data

### Visualizing Charge Density (VASP CHGCAR / Gaussian Cube)

OVITO can load volumetric data from CHGCAR (VASP) and `.cube` (Gaussian, GPAW, CP2K) files. The volumetric data is loaded
as a voxel grid object.

To visualize charge density:

1. Load the CHGCAR or `.cube` file — OVITO imports both the atoms and the volumetric field simultaneously.
2. In the **Visual Elements** section, find the **Volume vis** element (or **Voxel grid**).
3. Enable **isosurface rendering** and set the isovalue (threshold density) to define the isosurface level.
4. Adjust color and transparency to create the desired appearance.

For difference charge densities or electrostatic potential maps, the same workflow applies — just load the appropriate 
file.

### Working with Extended XYZ from CP2K, ASE, or GPAW

Many DFT codes can export structures and trajectories in the Extended XYZ format, where additional per-atom columns carry
DFT-computed quantities like forces, partial charges, or Hirshfeld volumes. OVITO reads these extra columns as particle 
properties, making them immediately available for color-coding and analysis.

A typical Extended XYZ file looks like this:

```
64
Lattice="10.5 0.0 0.0 0.0 10.5 0.0 0.0 0.0 10.5" Properties=species:S:1:pos:R:3:forces:R:3:energy:R:1
Si  2.1  0.0  3.3  -0.02  0.01  0.00  -3.521
Si  0.0  2.1  3.3   0.03 -0.01  0.00  -3.519
...
```

After loading, OVITO automatically parses `forces` and `energy` as per-atom properties. Use **Color coding** with the 
`energy` property to immediately visualize which sites are high-energy, or use **Compute property** to compute 
`|F| = sqrt(forces.x^2 + forces.y^2 + forces.z^2)` and map it to color.

### Visualizing AIMD Trajectories from VASP

For ab initio MD runs using VASP:

1. Load the **XDATCAR** file — OVITO reads all ionic steps as separate trajectory frames.
2. Use the timeline slider to scrub through the trajectory.
3. Add **Common Neighbor Analysis** or **PTM** to track structural changes through the run.
4. Use **Color coding** on displacement magnitude to see which atoms moved the most over the run.

To also visualize the per-step energy or temperature (from the OUTCAR file), you can parse these with a Python script 
(see Part 9) and add them as global attributes that appear in the timeline display.

### Analyzing DFT-Optimized Structures (CIF, POSCAR)

For a single optimized structure from a DFT geometry relaxation:

1. Load the POSCAR/CONTCAR or CIF file.
2. Add the **Create bonds** modifier with an appropriate cutoff for your material.
3. Adjust species colors and radii for a publication-quality figure.
4. Render a high-resolution PNG.

For supercell structures, use the **Replicate** modifier to expand the unit cell in any direction (nx × ny × nz repetitions)
to show a larger section of the material.

---

## Part 9: Python Scripting

OVITO's Python API (`ovito` package) allows you to build and run complete analysis pipelines from a Python script — no GUI 
required. This is invaluable for batch-processing hundreds of frames on an HPC cluster, or for integrating OVITO analysis 
into a larger data pipeline.

### Basic Script Structure

```python
from ovito.io import import_file
from ovito.modifiers import CommonNeighborAnalysisModifier
from ovito.io import export_file

# 1. Import the simulation file
pipeline = import_file("XDATCAR")

# 2. Add a modifier
pipeline.modifiers.append(CommonNeighborAnalysisModifier())

# 3. Compute the pipeline output for each frame
for frame in range(pipeline.source.num_frames):
    data = pipeline.compute(frame)
    
    # Access per-atom structure types
    structure = data.particles['Structure Type']
    
    # Count FCC atoms (type 1)
    n_fcc = (structure == 1).sum()
    print(f"Frame {frame}: {n_fcc} FCC atoms")
```

### Exporting Data to Text Files

```python
from ovito.io import export_file

# Export the number of FCC/HCP atoms vs. timestep
export_file(
    pipeline,
    "structure_vs_time.txt",
    "txt/attr",
    columns=["Timestep",
             "CommonNeighborAnalysis.counts.FCC",
             "CommonNeighborAnalysis.counts.HCP",
             "CommonNeighborAnalysis.counts.Other"],
    multiple_frames=True
)
```

### Rendering Images from a Script

```python
from ovito.vis import Viewport, RenderSettings

vp = pipeline.scene.viewports[0]    # Select the perspective viewport
vp.type = Viewport.Type.Perspective
vp.camera_dir = (1, 2, -1)

vp.render_image(
    size=(1920, 1080),
    filename="output.png",
    background=(1, 1, 1),    # White background
    frame=0
)
```

### Computing the Radial Distribution Function

```python
from ovito.modifiers import CoordinationAnalysisModifier
import numpy as np

pipeline.modifiers.append(
    CoordinationAnalysisModifier(cutoff=6.0, number_of_bins=200)
)

data = pipeline.compute(0)

# Get the RDF table
rdf = data.tables['coordination-rdf'].xy()
np.savetxt("rdf.txt", rdf, header="r g(r)")
```

### Running Batch Analysis on ISAAC-NG or HPC Systems

To run OVITO Python scripts without a display on an HPC cluster, use the `ovitos` script interpreter that ships with OVITO,
or use the `ovito` Python package installed via pip. No display server is needed for non-GUI use:

```bash
# Using the bundled ovitos interpreter
/path/to/ovito/bin/ovitos my_analysis.py

# Or using the pip-installed package
python my_analysis.py
```

---

## Part 10: Practical Workflows

### Workflow 1: Visualizing a DFT-Relaxed Structure for Publication

1. Load your CONTCAR or POSCAR file (**File → Load File**).
2. Add the **Replicate** modifier if you need a larger supercell view.
3. Add **Create bonds** with a cutoff appropriate to your material.
4. Adjust species colors (click species names in the pipeline editor).
5. Switch to the **Render** tab. Set resolution to 2400×2400 or higher.
6. Choose a camera angle — orbit in the viewport until satisfied.
7. Add a **Coordinate tripod** viewport layer to indicate orientation.
8. Click **Render active viewport** and save the PNG.

### Workflow 2: Tracking Structural Changes in an AIMD Trajectory

1. Load the **XDATCAR** file.
2. Add **Polyhedral Template Matching (PTM)** with FCC, HCP, and BCC enabled.
3. Observe the structure type distribution as you scrub through the timeline.
4. Add **Delete selected** after a **Select type** modifier (selecting "Other") to hide disordered atoms and view only 
crystalline regions.
5. Export quantitative data using Python or **File → Export File → txt/attr** format.

### Workflow 3: Analyzing Forces from a DFT Calculation

1. Load an extended XYZ file containing per-atom forces.
2. Add **Compute property:** define `force_mag = sqrt(forces.x^2 + forces.y^2 + forces.z^2)`.
3. Add **Color coding → Input property: force_mag** → choose a diverging colormap.
4. Adjust the range to highlight atoms under high force.
5. Optionally add **Expression select → force_mag > 0.5** → then add an **Assign color** modifier in red to highlight 
those atoms distinctly.
6. Render the image.

### Workflow 4: Identifying Defects in a Radiation-Damaged Structure

1. Load the defective structure as the main file, and the pristine reference as a second file.
2. Add **Wigner-Seitz defect analysis** and point to the reference structure.
3. Vacancies and interstitials appear as special particles with modified appearance.
4. Add **Color coding** or **Select type** to isolate and highlight defect types.
5. Use the **Construct surface mesh** modifier on the defect particles only to visualize defect cluster shapes.

---

## Part 11: Tips, Best Practices, and Common Mistakes

**Always check your file import.** After loading a file, click on the Data source in the pipeline editor and verify that 
the number of atoms, simulation cell, and detected format are correct. OVITO's auto-detection is reliable, but it's worth 
a sanity check.

**Use the Data Inspector.** When something looks wrong with your visualization, open the Data Inspector at the bottom to 
see the raw per-atom properties. This quickly reveals whether a modifier produced the expected output.

**Be mindful of pipeline order.** Modifiers act bottom-up. If you want to color by coordination number, the Coordination 
analysis modifier must appear below (i.e., run before) the Color coding modifier. Getting the order wrong is the most common
beginner mistake.

**Avoid creating bonds with too large a cutoff.** If your cutoff radius is too large, bonds will appear between non-bonded 
atoms, cluttering the visualization. For a new system, start with 10–20% above the known nearest-neighbor distance.

**Use Adaptive CNA or PTM for high-temperature MD.** Fixed-cutoff CNA can fail at elevated temperatures because thermal 
vibrations move atoms across the cutoff boundary. PTM and adaptive CNA are more thermally robust.

**Name your output files before rendering animations.** Set the output filename in the Render tab before starting a long 
animation render — otherwise OVITO renders to a temporary location.

**Save your session.** OVITO can save the full session (loaded files, pipeline setup, camera angle, render settings) as 
a `.ovito` file via **File → Save Session**. This lets you pick up exactly where you left off.

**Citation.** When publishing work that used OVITO, cite:
> A. Stukowski, *Visualization and analysis of atomistic simulation data with OVITO — the Open Visualization Tool*, 
Modelling Simul. Mater. Sci. Eng. 18, 015012 (2010).

---

## Quick Reference

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `F` | Zoom to fit all atoms in view |
| `Ctrl + Z` | Undo |
| `Ctrl + R` | Render active viewport |
| `Ctrl + S` | Save session |
| `Space` | Play / pause animation |
| `Left / Right arrow` | Step backward / forward one frame |
| `X`, `Y`, `Z` | Align view along X, Y, or Z axis |

### Common Modifiers at a Glance

| Modifier | Use Case |
|---------|---------|
| Common Neighbor Analysis | Identify crystal phases and defects (FCC/HCP/BCC) |
| Polyhedral Template Matching | More robust crystal structure identification |
| Displacement Vectors | Track atom movement relative to reference |
| Voronoi Analysis | Per-atom volume, coordination number, topology |
| Wigner-Seitz Defect Analysis | Identify vacancies and interstitials |
| Coordination Analysis | Compute RDF and coordination numbers |
| Color coding | Color atoms by any per-atom property |
| Compute property | Define new properties via math expressions |
| Expression select | Select atoms by mathematical criteria |
| Create bonds | Generate bonds from a distance cutoff |
| Slice | Cut through the structure with a plane |
| Replicate | Tile the simulation cell |
| Wrap at periodic boundaries | Re-wrap atoms into the unit cell |
| Delete selected | Remove atoms from view |
| Assign color | Set a fixed color for selected atoms |
| Affine transformation | Rotate, scale, or translate atoms |
| Freeze property | Preserve a property value from a specific frame |

### Supported DFT/AIMD File Formats

| Code | Readable Formats |
|------|----------------|
| VASP | POSCAR, CONTCAR, XDATCAR, CHGCAR |
| Quantum Espresso | Input files (`.in`) |
| FHI-aims | `geometry.in`, `aims.out` |
| CASTEP | `.cell`, `.md`, `.geom` |
| Gaussian / GPAW / CP2K | `.cube` |
| CP2K / ASE / many others | `.xyz`, `.extxyz` |
| SIESTA | `.xsf` |
| General crystallography | `.cif` |

### Where to Get Help

- **Official documentation:** [https://www.ovito.org/manual/](https://www.ovito.org/manual/)
- **Python API reference:** [https://www.ovito.org/manual/python/](https://www.ovito.org/manual/python/)
- **Community forum:** [https://www.ovito.org/forum/](https://www.ovito.org/forum/)
- **Tutorials:** [https://www.ovito.org/manual/tutorials/](https://www.ovito.org/manual/tutorials/)
- **OVITO Extensions Directory:** Community-contributed custom modifiers and scripts available at 
[https://www.ovito.org/extensions/](https://www.ovito.org/extensions/)
