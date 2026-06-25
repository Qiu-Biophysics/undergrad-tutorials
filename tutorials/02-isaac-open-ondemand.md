# ISAAC & Open OnDemand Tutorial
## Beginner's Guide to ISAAC-NG
**University of Tennessee High Performance Computing Cluster**
 
---
 
## What Is ISAAC-NG?
 
ISAAC-NG (ISAAC Next Generation) is the University of Tennessee's high-performance computing (HPC) cluster, 
operated by the Office of Innovative Technologies (OIT). It has been in production since January 2022 and provides 
researchers, faculty, and students with access to powerful computational resources far beyond what a personal computer 
can offer.
 
Think of ISAAC-NG as a massive shared supercomputer made up of hundreds of individual machines (called **nodes**), all 
connected together and managed by a scheduling system that queues and runs your computational tasks. Instead of running 
a program on your laptop, you write a script describing what you want done, submit it to the scheduler, and the cluster 
runs it for you — often on many processors simultaneously.
 
This guide will walk you through everything you need to get started: what the system looks like, how to log in, how to 
find and load software, how to store your files, how to transfer data, and how to submit, run, and monitor your jobs.
 
---
 
## Part 1: System Overview
 
### Compute Nodes
 
ISAAC-NG has a diverse collection of compute nodes suited for different kinds of work. A **node** is essentially an 
individual server with its own processors, memory, and sometimes GPUs. The cluster currently contains over 18,000 CPU 
cores spread across 259+ nodes. Key node types include:
 
- **Standard CPU nodes** — for general-purpose computing. These use modern Intel (Cascade Lake, Ice Lake, Sapphire Rapids, 
Sky Lake) and AMD (Bergamo, Genoa, Milan, Rome) processors, with core counts ranging from 32 to 256 per node and memory 
from 128 GB to 2 TB.
- **GPU nodes** — for machine learning, AI, and accelerated computing. Available GPUs include NVIDIA V100S, T4, A40, A16, 
L40, and Hopper H100 (up to 80 GB memory per GPU).
- **BigMem nodes** — for analyses requiring very large amounts of RAM (up to 2 TB).
- **NVME scratch nodes** — GPU nodes with fast local temporary storage (up to 47 TB) for data-intensive workloads.
All compute nodes are connected via a high-speed **InfiniBand** network that enables fast communication between nodes for 
parallel jobs.
 
### The Login Node
 
When you first connect to ISAAC-NG, you land on a **login node**. This is your gateway to the cluster — a shared machine 
where you can prepare files, write scripts, and submit jobs. It is **not** a place to run computations.
 
> 🚨 **Critical Rule:** Never run heavy computational work on the login node. The login node is shared by all users, and 
running intensive programs there will slow down or crash it for everyone. Any production jobs running on login nodes are 
subject to termination.
 
The login node is for: editing files, compiling code, short tests, loading software modules, and submitting/monitoring jobs.
 
---
 
## Part 2: Getting an Account and Logging In
 
### Requesting an Account
 
Before you can use ISAAC-NG, you need an account. Account requests are submitted through the ISAAC User Portal at 
[portal.acf.utk.edu/accounts/request](https://portal.acf.utk.edu/accounts/request). Once your account is created, 
you can also manage your projects and view your storage quotas through the portal at 
[portal.acf.tennessee.edu](https://portal.acf.tennessee.edu).
 
### What You Need to Log In
 
You will need three things to connect:
 
1. **An SSH client** — a program that creates a secure connection to the cluster.
   - On Mac and Linux, SSH is built into the terminal — no installation needed.
   - On Windows 10 and later, SSH is available in Command Prompt and PowerShell.
   - On older Windows (7 or 8), download **PuTTY** or **MobaXterm**.
2. **Your UT NetID and password.**
3. **The Duo Mobile app** for two-factor authentication (2FA). Download it from the App Store (iOS) or Google Play (Android).
### Logging In via SSH (Mac, Linux, or Windows 10+)
 
Open your terminal and type:
 
```bash
ssh your-netid@login.isaac.tennessee.edu
```
 
Replace `your-netid` with your actual UT NetID. When prompted, enter your NetID password. You'll then be asked to 
authenticate with Duo — type `1` and press Enter to receive a push notification on your phone, or type `2` to receive 
an SMS code. After successful authentication, you're in.
 
### Logging In via PuTTY (Older Windows)
 
Open PuTTY, enter `login.isaac.tennessee.edu` as the hostname, ensure the port is `22` and SSH is selected, then click 
**Open**. You'll be prompted for your NetID, password, and Duo code.
 
### Logging In via Open OnDemand (Web Browser)
 
If you prefer a graphical interface instead of a terminal, you can access ISAAC-NG through your web browser. 
Navigate to [login.isaac.utk.edu](https://login.isaac.utk.edu) or [login.isaac.tennessee.edu](https://login.isaac.tennessee.edu). 
Log in with your NetID and Duo 2FA, and you'll see the Open OnDemand dashboard. This is a great option for beginners and is 
covered in detail in Part 7.
 
---
 
## Part 3: File Systems
 
ISAAC-NG provides three distinct storage areas, each designed for a different purpose. Understanding which one to use for
what is essential.
 
### Home Directory (`/nfs/home/<username>`)
 
| Property | Value |
|----------|-------|
| Quota | 50 GB, 1 million files |
| Backed up | Not currently backed up |
| Purged | Never |
 
This is where you land when you log in. It's intended for small, important files: job scripts, configuration files, 
virtual environments, and similar items. **Do not use it for large datasets or job output** — it has a small quota and 
will fill up quickly.
 
You can refer to your home directory with `$HOME` or the shorthand `~`.
 
### Scratch Directory (`/lustre/isaac24/scratch/<username>`)
 
| Property | Value |
|----------|-------|
| Quota | 10 TB, 5 million files |
| Backed up | Never |
| Purged | Files not accessed or modified in 180+ days may be deleted on approximately the 3rd Monday of each month |
 
This is your **primary workspace** for running jobs. It sits on a high-performance Lustre parallel filesystem with 3.6 
petabytes of total capacity, designed for fast reading and writing of large datasets across many compute nodes simultaneously.
 
Use your scratch directory for data you're actively computing on. **Do not store permanent data here** — it is not backed up 
and will be purged periodically. You can refer to your scratch directory using the environment variable `$SCRATCHDIR`.
 
To find which files in your scratch space are old enough to be purged:
```bash
lfs find $SCRATCHDIR -mtime +180 -type f
```
 
To check how much Lustre space you are using:
```bash
lfs quota -u <your-netid> /lustre/isaac24
```
 
To check all your quotas at once:
```bash
checkquotas
```
 
### Project Directory (`/lustre/isaac24/proj/<project>`)
 
| Property | Value |
|----------|-------|
| Quota | 1 TB default (additional space can be requested) |
| Backed up | No |
| Purged | Never |
 
Project directories are shared spaces for research groups and are not subject to purging. To get one, a UT faculty 
member (as PI) must submit a project request through the User Portal or via an HPSC service request. Graduate students 
and post-docs must have their faculty advisor submit the request.
 
> ⚠️ **Important:** You are responsible for backing up your own data. ISAAC does not have archival storage built 
in — plan to transfer important results off-cluster to a safe location.
 
---
 
## Part 4: Available Software and Modules
 
ISAAC-NG provides a large library of scientific software. Rather than installing programs permanently in your environment, 
the cluster uses a **module system (Lmod)** that lets you load and unload software on demand.
 
### Default Available Software
 
Without loading any modules, some commonly used tools are immediately available, including Git, Julia, Jupyter Notebooks,
R, Python, MATLAB, SAS, machine learning frameworks (TensorFlow, PyTorch, Keras, OpenCV), and Conda environments.
 
### Listing and Searching for Modules
 
To see all available software:
```bash
module avail
```
 
To search for a specific package (e.g., Python):
```bash
module avail Python
```
 
To get more details about a module:
```bash
module show Python
```
 
### Loading and Unloading Software
 
To load a module and make it available in your environment:
```bash
module load Python/3.8.6-GCCcore-10.2.0
```
 
> **Note:** Module names are case-sensitive. Always use the full version name (e.g., `Python/3.8.6-GCCcore-10.2.0` not 
just `Python`) to ensure reproducibility — default module versions can change over time as new versions are installed.
 
To load multiple compatible modules at once:
```bash
module load Python/3.8.6-GCCcore-10.2.0 BLAST+/2.11.0-GCCcore-10.2.0
```
 
To unload a specific module:
```bash
module unload R
```
 
To unload all modules:
```bash
module purge
```
 
> **Note:** `module purge` will warn you about a "sticky" module called `StdEnv` that should remain loaded. Avoid 
removing it unless specifically instructed.
 
To see what modules you currently have loaded:
```bash
module list
```
 
### Using Modules in Job Scripts
 
For batch jobs, the recommended approach is to include your `module load` commands directly inside the job script 
itself. This makes your work reproducible and self-contained. For example:
 
```bash
module load Python/3.8.6-GCCcore-10.2.0
python my_script.py
```
 
To request software not currently installed, submit an HPSC service request through the sidebar on the HPSC website.
 
---
 
## Part 5: Transferring Data
 
### Data Transfer Nodes (DTNs)
 
Do not use the login node to transfer large files. ISAAC-NG provides dedicated **Data Transfer Nodes (DTNs)** for 
this purpose:
 
| Node | Hostname | Globus Endpoint |
|------|----------|----------------|
| dtn1 | dtn1.isaac.utk.edu | UTK ISAAC-NG DTN1 |
| dtn2 | dtn2.isaac.utk.edu | UTK ISAAC-NG DTN2 / UTK Google Drive |
 
You can connect to these nodes the same way as the login node — just replace the hostname.
 
### Method 1: SCP (Small Transfers from the Command Line)
 
SCP (Secure Copy) is built into Mac, Linux, and modern Windows terminals. It is best for quick, small transfers.
 
Upload a file to ISAAC-NG:
```bash
scp ~/myfile.txt yournetid@dtn1.isaac.utk.edu:~/Documents
```
 
Download a file from ISAAC-NG:
```bash
scp yournetid@dtn1.isaac.utk.edu:~/Documents/results.txt ~/Desktop
```
 
### Method 2: SFTP (Interactive File Transfer)
 
SFTP opens an interactive session for navigating and transferring files:
```bash
sftp yournetid@dtn1.isaac.utk.edu
```
 
Once connected, use these commands:
 
| Command | Action |
|---------|--------|
| `put filename` | Upload a file from your local machine |
| `get filename` | Download a file to your local machine |
| `cd directory` | Change remote directory |
| `lcd directory` | Change local directory |
| `bye` or `exit` | Quit SFTP |
 
### Method 3: Globus (Large, Fast Transfers — Recommended)
 
Globus is the fastest and most reliable method for large data transfers. It handles transfers in the background and 
can resume if interrupted.
 
1. Go to [app.globus.org](https://app.globus.org) and log in using "University of Tennessee" as your organization.
2. Authenticate with your NetID and Duo.
3. In the File Manager, search for the ISAAC-NG endpoint: **UTK ISAAC-NG DTN1** (or DTN2).
4. Set up your local machine as a Globus endpoint by installing **Globus Connect Personal** and logging in.
5. With both endpoints selected in the dual-panel File Manager, select files and click **Start** to transfer.
You can also transfer between ISAAC-NG and your UTK Google Drive using the **UTK Google Drive** endpoint on dtn2.
 
### Method 4: FileZilla or WinSCP (GUI Tools for Windows)
 
Both are free graphical tools for Windows. Connect using these settings:
 
- **Host:** `dtn1.isaac.utk.edu` or `dtn2.isaac.utk.edu`
- **Protocol:** SFTP
- **Logon Type:** Interactive
- **Username:** your UT NetID
- **Port:** 22
When prompted for authentication, enter your password then type `1` to receive a Duo push.
 
### Preparing Data for Transfer
 
Before transferring large amounts of files, compress and archive them first to reduce transfer time and simplify 
organization:
 
```bash
# Create a compressed archive
tar -czf results.tar.gz results_directory/
 
# Extract an archive
tar -xzf results.tar.gz
```
 
---
 
## Part 6: Writing and Running Jobs
 
### Understanding the Scheduler
 
ISAAC-NG uses **SLURM** (Simple Linux Utility for Resource Management) to manage and schedule computational jobs. 
When you submit a job, SLURM places it in a queue and runs it when the requested resources become available. You never 
run computations directly — you always submit them through SLURM.
 
### Partitions and QOS
 
SLURM organizes the cluster into logical groups of nodes called **partitions**. Each partition has a set of rules 
(**QOS — Quality of Service**) that govern how long a job can run and how many resources it can use. The most common 
ones for new users:
 
| Partition | QOS | Max Run Time | Notes |
|-----------|-----|-------------|-------|
| campus | campus | 24 hours | Default. Good for most jobs |
| campus-gpu | campus-gpu | 24 hours | GPU jobs |
| campus-bigmem | campus-bigmem | 24 hours | Large memory jobs |
| short | short | 3 hours | Quick jobs, fast queue |
| long | long | 6 days | Long-running jobs |
| long-gpu | long-gpu | 6 days | Long GPU jobs |
 
### Project Accounts
 
Every job must be associated with a **project account**. For most UTK users, the default is `ACF-UTK0011`. You can see 
your available accounts by logging into the User Portal. Your SLURM script must specify this account.
 
### Writing a Batch Job Script
 
A batch job script is a text file (`.sh`) that tells SLURM what resources you need and what commands to run. Here is a 
template:
 
```bash
#!/bin/bash
# This script requests resources from SLURM and runs a job
 
#SBATCH -J my_job_name           # Name of the job
#SBATCH -A ACF-UTK0011           # Project account
#SBATCH --nodes=1                # Number of nodes
#SBATCH --ntasks-per-node=4      # CPU cores per node
#SBATCH --partition=campus       # Partition to use
#SBATCH --qos=campus             # Quality of service (must match partition)
#SBATCH --time=0-02:00:00        # Max wall time: days-hours:minutes:seconds
#SBATCH --output=job.o%j         # File for standard output
#SBATCH --error=job.e%j          # File for error messages
 
# Load any software you need
module load Python/3.8.6-GCCcore-10.2.0
 
# Run your program
srun python my_analysis.py
```
 
**Key sections explained:**
 
- **Line 1 (`#!/bin/bash`)** — Required. Tells the system to use the bash shell.
- **`#SBATCH` lines** — These are SLURM directives. They are not comments — SLURM reads them to allocate resources.
- **`module load`** — Load software before your commands.
- **`srun`** — Runs your program using the resources allocated. Use it for parallel or MPI programs. Serial programs 
can be run with or without `srun`.
> 💡 **Tip:** Submit jobs from your Lustre scratch directory (`$SCRATCHDIR`), not from your home directory. SLURM uses
the submission directory as the working directory, and Lustre has much more space.
 
### Submitting a Job
 
Once your script is written (e.g., `myjob.sh`), submit it:
 
```bash
sbatch myjob.sh
```
 
You'll receive a job ID, like: `Submitted batch job 1234`. Keep this number — you'll use it to monitor and manage your job.
 
### Interactive Jobs (for Testing and Debugging)
 
If you want to run commands interactively on a compute node (instead of submitting a script), use `salloc`:
 
```bash
salloc -A ACF-UTK0011 --nodes=1 --ntasks=1 --partition=campus --qos=campus --time=01:00:00
```
 
Once resources are allocated, you'll get a prompt on the compute node. From there, you can load modules, run tests, and 
debug your code. When done, type `exit`.
 
### Checking Available Resources Before Submitting
 
Before submitting a job, it's good practice to see what resources are currently free, to avoid long queue waits:
 
```bash
# View all partitions with available resources
isaac-sinfo | egrep 'idle|mixed'
 
# View only GPU partitions
isaac-sinfo | grep gpu | egrep 'idle|mixed'
```
 
Nodes labeled `idle` are entirely free; `mixed` means some resources are in use but more are available.
 
### GPU Jobs
 
To request GPU resources, use the `campus-gpu` partition and specify GPUs:
 
```bash
#SBATCH --partition=campus-gpu
#SBATCH --qos=campus-gpu
#SBATCH --gpus=1
```
 
> ⚠️ **GPU Policy:** Jobs that request GPUs but don't actually use them are subject to cancellation. After 60 minutes 
of an idle GPU, you'll receive a warning email; after 120 minutes, the job will be automatically cancelled. Only request 
GPUs when your code is actually configured to use them.
 
### Job Arrays (Running Many Similar Jobs at Once)
 
If you need to run the same script many times with different inputs, use a job array instead of submitting individual jobs:
 
```bash
#SBATCH --array=1-30    # Runs jobs indexed 1 through 30
```
 
Within your script, `$SLURM_ARRAY_TASK_ID` will be set to the current job's index (1, 2, 3, … 30), letting you point each job 
at a different input file or parameter.
 
---
 
## Part 7: Monitoring Jobs
 
### Checking the Status of Your Jobs
 
```bash
squeue -u your-netid
```
 
Sample output:
```
JOBID   PARTITION   NAME     USER  ST   TIME  NODES  NODELIST(REASON)
1202    campus      my_job   abc   PD   0:00  2      (Resources)
1201    campus      my_job   abc   R    0:05  2      node[001-002]
```
 
**Job status codes in the `ST` column:**
 
| Code | Meaning |
|------|---------|
| R | Running |
| PD | Pending (waiting in queue) |
| CG | Completing |
| S | Suspended |
| NF | Node failure |
 
### Getting Details About a Specific Job
 
```bash
scontrol show job 1234
```
 
This shows all details about the job including its state, node assignment, requested resources, and output file locations.
 
### Checking What Script a Job Is Running
 
```bash
sacct -B -j 1234
```
 
### Checking System-Wide Partition Status
 
```bash
isaac-sinfo
showpartitions
```
 
### Cancelling a Job
 
```bash
scancel 1234
```
 
### Modifying a Pending Job
 
You can change some attributes of a job that hasn't started yet:
 
```bash
# Change wall time
scontrol update JobID=1234 TimeLimit=2-00:00:00
 
# Change job name
scontrol update JobID=1234 JobName=new_name
 
# Hold or release a job
scontrol hold 1234
scontrol release 1234
```
 
---
 
## Part 8: Open OnDemand (Web Browser Interface)
 
Open OnDemand is a browser-based portal for accessing ISAAC-NG without needing a terminal. It is especially useful for 
beginners and for running graphical applications.
 
### Logging In
 
Navigate to [login.isaac.utk.edu](https://login.isaac.utk.edu) or [login.isaac.tennessee.edu](https://login.isaac.tennessee.edu) 
in your browser. Log in with your NetID and Duo 2FA. You'll arrive at the Open OnDemand dashboard.
 
### File Manager
 
Under the **Files** menu, you can browse, upload, download, create, edit, rename, copy, and delete files — all from within your 
browser. The interface shows your home directory by default. To navigate to your scratch space, click **Change Directory** and 
type `/lustre/isaac24/scratch/yournetid`.
 
> For large file transfers, use Globus instead of the File Manager's upload/download feature.
 
### Job Management
 
Under the **Jobs** menu:
 
- **Active Jobs** — shows all your queued and running jobs with details and output file locations.
- **Job Composer** — a graphical interface to create, configure, and submit new batch jobs using templates. Useful for
learning how jobs are structured without writing scripts from scratch.
### Interactive Apps (ISAAC Desktop and Jupyter Notebook)
 
Under **Interactive Apps**, you can launch graphical sessions directly in your browser:
 
**ISAAC Desktop (noVNC):** Provides a full Linux desktop environment (XFCE) in your browser. Useful when you need to 
run applications with graphical interfaces. Specify your resource requirements, click **Launch**, and then click 
**Launch ISAAC Desktop** when the session is ready.
 
**Jupyter Notebook:** Provides a browser-based Python development environment. Specify resources, partition, and 
Anaconda version, then click **Launch**. When ready, click **Connect to Jupyter** to open a notebook. You can run 
Linux commands within notebook cells by prefixing them with `!` (e.g., `!ls $SCRATCHDIR`).
 
> Before launching an interactive session, run `isaac-sinfo | grep campus | grep -E 'idle|mixed'` from the login node 
to check what resources are available, so you don't wait unnecessarily in the queue.
 
---
 
## Part 9: Getting Help
 
ISAAC-NG has active support channels:
 
- **Office Hours:** Join the ISAAC team on Zoom every **Tuesday, Thursday, and Friday from 11:00 AM – 12:00 PM Eastern**. 
The Zoom link is available on the HPSC website (authenticated users only).
- **Help Tickets:** Submit an HPSC Service Request at the top of any HPSC webpage, or through the OIT Service Portal at 
[help.utk.edu](https://help.utk.edu).
- **Example Job Scripts:** A collection of ready-to-use example scripts is available on the cluster at `
/lustre/isaac/examples/jobs`. Copy these to your scratch directory and adapt them to your needs.
---
 
## Quick Reference
 
### Key Paths
 
| Purpose | Path |
|---------|------|
| Home directory | `/nfs/home/<your-netid>` or `~` or `$HOME` |
| Scratch directory | `/lustre/isaac24/scratch/<your-netid>` or `$SCRATCHDIR` |
| Project directory | `/lustre/isaac24/proj/<project-name>` |
| Example job scripts | `/lustre/isaac/examples/jobs` |
 
### Essential SLURM Commands
 
| Command | What It Does |
|---------|-------------|
| `sbatch myjob.sh` | Submit a job script |
| `squeue -u netid` | View your jobs |
| `scontrol show job ID` | View job details |
| `scancel ID` | Cancel a job |
| `salloc …` | Request an interactive session |
| `srun executable` | Run a program on allocated nodes |
| `isaac-sinfo` | View partition and node status |
| `showpartitions` | View a summary of partition availability |
 
### Essential Module Commands
 
| Command | What It Does |
|---------|-------------|
| `module avail` | List all available software |
| `module avail Python` | Search for Python modules |
| `module load Module/version` | Load a software module |
| `module unload Module` | Unload a module |
| `module purge` | Unload all modules |
| `module list` | Show currently loaded modules |
 
### Storage Quotas at a Glance
 
| Storage | Path | Default Quota | Purged? |
|---------|------|--------------|---------|
| Home | `/nfs/home/<user>` | 50 GB | No |
| Scratch | `/lustre/isaac24/scratch/<user>` | 10 TB | Yes (180-day inactivity) |
| Project | `/lustre/isaac24/proj/<project>` | 1 TB | No |
 
---
 
## More on Open OnDemand
 
Open OnDemand is an easy way to visualize the filesystems you are working with when running these computation jobs. 
What you can do is create a folder for the project you are working on and title it as the name of the project. Within 
the folder, you can easily import the files you need, look at their contents, move files around, and more. You can also 
open the terminal in which you can access the login node.
