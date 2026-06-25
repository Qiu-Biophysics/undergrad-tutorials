# Beginner's Guide to DP-GEN (Hands-On Tutorial)
Using the methane example from DeepModeling

**Link to data:** https://dp-public.oss-cnbeijing.aliyuncs.com/community/dpgen_example.tar.xz

---

## Part 1: What You Are Doing

DP-GEN is a tool that automatically builds machine-learned interatomic potentials using an iterative workflow of:

- Training
- Exploration
- First-principles labeling

This loop improves the model step-by-step.

Each iteration includes:

- `00.train` → train models
- `01.model_devi` → test model uncertainty
- `02.fp` → run DFT calculations

---

## Part 2: Folder Setup (Sample Data)

### Step 1: Navigate to Example Directory

After running these commands:

```bash
wget https://dp-public.oss-cn-beijing.aliyuncs.com/community/dpgen_example.tar.xz
tar xvf dpgen_example.tar.xz
```

to download and unzip the example's data, run this in the terminal:

```bash
cd dpgen_example/run
ls
```

You should see files like:

- `param.json`
- `machine.json`
- `INCAR_methane`
- `POTCAR_*`

These define:

- Simulation parameters (`param.json`)
- Machine resources (`machine.json`)
- DFT inputs (VASP files)

```
init
├── CH4.POSCAR
└── CH4.POSCAR.01x01x01
    ├── 00.place_ele
    ├── 01.scale_pert
    ├── 02.md
    └── param.json
├── INCAR_methane.md
├── INCAR_methane.rlx
└── param.json

5 directories, 5 files
```

> Screenshot of contents of `dpgen_example`

---

## Part 3: Understanding Input Files

### Step 2: Inspect `param.json`

```bash
cat param.json
```

Key parameters:

- `type_map` → atom types (e.g., H, C)
- `mass_map` → atomic masses
- `init_data_sys` → initial training data
- `sys_configs` → structures for exploration

```json
{
    "type_map": ["H", "C"],
    "mass_map": [1, 12],
    "init_data_prefix": "../",
    "init_data_sys": ["init/CH4.POSCAR.01x01x01/02.md/sys-0004-0001/deepmd"],
    "sys_configs_prefix": "../",
    "sys_configs": [
        ["init/CH4.POSCAR.01x01x01/01.scale_pert/sys-0004-0001/scale-1.000/00000*/POSCAR"],
        ["init/CH4.POSCAR.01x01x01/01.scale_pert/sys-0004-0001/scale-1.000/00001*/POSCAR"]
    ],
    "_comment": " that's all ",
    "numb_models": 4,
    "default_training_param": {
        "model": {
            "type_map": ["H", "C"],
            "descriptor": {
                "type": "se_a",
                "sel": [16, 4],
                "rcut_smth": 0.5,
                "rcut": 5.0,
                "neuron": [120, 120, 120],
                "resnet_dt": true,
                "axis_neuron": 12,
                "seed": 1
            },
            "fitting_net": {
                "neuron": [25, 50, 100],
                "resnet_dt": false,
                "seed": 1
            }
        },
        "learning_rate": {
            "type": "exp",
            "start_lr": 0.001,
            "decay_steps": 5000
        },
        "loss": {
            "start_pref_e": 0.02,
            "limit_pref_e": 2,
            "start_pref_f": 1000,
            "limit_pref_f": 1
        }
    }
}
```

> Screenshot of `param.json` open in the editor

---

### Step 3: Inspect `machine.json`

```bash
cat machine.json
```

This file defines:

- Where jobs run (local or cluster)
- CPU/GPU usage
- Execution commands

Example:

- `"command": "dp"` → runs DeePMD training
- `"cpu_per_node": 4, "gpu_per_node": 1`

```json
{
    "api_version": "1.0",
    "deepmd_version": "2.0.1",
    "train": [
        {
            "command": "dp",
            "machine": {
                "batch_type": "Shell",
                "context_type": "local",
                "local_root": "./",
                "remote_root": "/home/ecust/lws"
            },
            "resources": {
                "number_node": 1,
                "cpu_per_node": 4,
                "gpu_per_node": 1,
                "group_size": 1
            }
        }
    ],
    "model_devi": [
        {
            "command": "mpirun -n 4 lmpdplws -i input.lammps",
            "machine": {
                "batch_type": "Shell",
                "context_type": "local",
                "local_root": "./",
                "remote_root": "/home/ecust/lws"
            },
            "resources": {
                "number_node": 1,
                "cpu_per_node": 4,
                "gpu_per_node": 1,
                "group_size": 5
            }
        }
    ],
    "fp": [
        {
            "command": "mpirun -n 4 vasplws",
            "machine": {
                "batch_type": "Shell",
                "context_type": "local",
                "local_root": "./",
                "remote_root": "/home/ecust/lws"
            },
            "resources": {
                "number_node": 1,
                "cpu_per_node": 4,
                "gpu_per_node": 1,
                "group_size": 125
            }
        }
    ]
}
```

> Screenshot of `machine.json`

---

## Part 4: Running DP-GEN

### Step 4: Start the Run Process

```bash
dpgen run param.json machine.json
```

This launches the **main iterative workflow**:

- Training → MD exploration → DFT labeling

```
DeepModeling
------------
Version: 0.xx.x

INFO : check_all_finished: False
INFO : start iteration 000000
INFO : make_train
INFO : run_train
INFO : post_train
INFO : make_model_devi
INFO : run_model_devi
INFO : make_fp
INFO : run_fp
```

> This is approximately what it should look like when `dpgen run` starts. It's a bad sign if it starts immediately 
crashing, there are permission errors, or if there are missing iteration folders. It may take some troubleshooting, 
but it should work.

---

## Part 5: Monitoring Progress

### Step 5: Check Output Files

After starting, run:

```bash
ls
```

You should see:

- `dpgen.log`
- `record.dpgen`
- `iter.000000/`

```
run/
├── dpgen.log
├── record.dpgen
├── param.json
├── machine.json
└── iter.000000/
```

> After running dpgen, your directory should look approximately like this.

Then, within `iter.000000/`, it should look something like this:

```
iter.000000/
├── 00.train/
├── 01.model_devi/
└── 02.fp/
```

---

### Step 6: Check Iteration Progress

```bash
cat record.dpgen
```

Each line: `iteration_index  stage_index`

Stages include:

- `0` → training
- `4` → model deviation
- `7` → DFT calculations

```
0 0
0 1
0 2
0 3
0 4
0 5
0 6
0 7
0 8
```

> This is what `record.dpgen` output should look like. These numbers are stages in each iteration. For example, 
`00` means start training, `01` means the training is running, `04` means the MD (model deviation) is running, 
and `07` means the FP (first principles) is running.

---

## Part 6: Exploring Results

### Step 7: Look Inside Iteration Folder

```bash
tree iter.000000 -L 1
```

You should see:

```
iter.000000
├── 00.train
├── 01.model_devi
└── 02.fp
```

> The iteration folder structure, `iter.000000`

---

## Part 7: Training Results

### Step 8: Inspect Training Outputs

```bash
tree iter.000000/00.train -L 1
```

Contents:

- `000`, `001`, `002`, `003` → different trained models
- `graph.000.pb` → trained model files

These are neural network potentials trained with different seeds.

```
00.train/
├── 000/
├── 001/
├── 002/
├── 003/
├── lcurve.out
├── train.log
└── graph.000.pb
```

> What the `00.train` directory should look like

---

## Part 8: Model Deviation (Exploration)

### Step 9: Understand Model Deviation

DP-GEN runs MD simulations and compares models:

- **Low deviation** → model is accurate
- **Medium** → candidate for DFT
- **High** → unreliable configuration

```
01.model_devi/
├── confs/
├── task.000.000000/
├── task.000.000001/
├── task.000.000002/
├── model_devi.out
└── log.lammps
```

> What `01.model_devi/` folder structure should look like

Example of contents of `model_devi.out`:

```
frame  max_devi_f  min_devi_f  avg_devi_f
0      0.12        0.05        0.08
1      0.20        0.07        0.13
```

---

## Part 9: First-Principles Calculations

### Step 10: Inspect DFT Tasks

```bash
tree iter.000000/02.fp -L 1
```

This contains:

- DFT jobs (e.g., VASP inputs)
- Selected configurations for labeling

```
02.fp/
├── candidate.shuffle.000.out
├── rest_accurate.shuffle.000.out
├── rest_failed.shuffle.000.out
├── data.000/
├── task.000.000000/
├── task.000.000001/
```

> `candidate` is the selected structures for DFT, `task.*` is the actual Quantum Espresso / VASP runs, `data.000` is 
the collected labeled dataset.

---

### Important Note (VASP vs QE)

- This tutorial uses **VASP input files** (`INCAR`, `POTCAR`)
- If using **Quantum ESPRESSO**, you must:
  - Replace VASP inputs with QE `.in` files
  - Adjust `param.json` accordingly

DP-GEN supports both as labeling engines.

---

## Part 10: Understanding the Iterative Loop

Each iteration does:

1. Train models (`00.train`)
2. Run MD and detect uncertainty (`01.model_devi`)
3. Run DFT on selected structures (`02.fp`)

Then:

- New data is added
- Next iteration begins automatically

```
run/
├── iter.000000/
├── iter.000001/
├── iter.000002/
├── iter.000003/
├── dpgen.log
└── record.dpgen
```

> Multiple iteration folders — what "finished runs" should look like.

Each iteration repeats: `00.train` → `01.model_devi` → `02.fp`

---

## Part 11: Logs and Debugging

### Step 11: Check Log File

```bash
cat dpgen.log
```

This shows:

- Iteration progress
- Errors
- Runtime info

Example `dpgen.log` when full run is finished:

```
iteration 0 finished
iteration 1 finished
iteration 2 finished
iteration 3 finished
training converged
job completed successfully
```

---

## Final Summary

You just demonstrated you can:

- Run DP-GEN
- Monitor iterations
- Understand how training, MD, and DFT connect

DP-GEN automates:

- Data generation
- Model training
- Accuracy improvement

All with minimal manual intervention.
