# Navigating the Linux Filesystem

If you've never used a Linux terminal before, the filesystem can feel foreign. There are no icons to click and everything 
looks like a wall of text. This guide will walk you through how Linux organizes its files and teach you the essential 
commands to find your way around.

---

## Part 1: How the Linux Filesystem Is Structured

### Everything Starts at Root

In Linux, the entire filesystem begins at a single point called the **root directory**, written as a single forward 
slash: `/`

It's like the trunk of a tree. Every file and folder on the system, no matter what hard drive or device it lives on, 
branches out from this one starting point.

This is different from Windows, where each drive gets its own letter (`C:\`, `D:\`, etc.). In Linux, there is only one 
tree, and everything hangs off it.

### The Home Directory

When you log into a Linux system, you start in your **home directory**. This is your personal space where your files, 
settings, and documents live. It's written as:

```
/home/yourusername
```

For example, if your username is `ada`, your home directory is `/home/ada`. Linux also gives you a shorthand for this: 
the tilde symbol `~`. Whenever you see `~`, it means "my home directory."

### Key Directories and What They're For

The Linux filesystem has a set of standard top-level directories. It helps to know what they're for when you encounter them.

| Directory | What it contains |
|-----------|-----------------|
| `/` | The root — the top of everything |
| `/home` | Personal directories for each user |
| `/root` | The home directory for the administrator (root user) |
| `/etc` | System-wide configuration files |
| `/bin` | Essential programs and commands (like `ls`, `cp`) |
| `/usr` | Installed software and user programs |
| `/var` | Files that change often, like logs |
| `/tmp` | Temporary files, cleared on reboot |
| `/dev` | Representations of hardware devices |
| `/mnt` | A place to access external drives or mounted filesystems |

You'll spend most of your time inside `/home/yourusername`. The other directories belong to the system, and you generally 
don't need to modify them.

### Absolute vs. Relative Paths

A **path** is simply the address of a file or folder. There are two kinds:

**Absolute paths** start from root (`/`) and describe the full location of something:
```
/home/ada/documents/report.txt
```

**Relative paths** describe a location starting from wherever you currently are. If you're already inside `/home/ada`, you 
could refer to the same file as:
```
documents/report.txt
```

Two special shortcuts are used in relative paths:

- `.` (a single dot) means "the current directory"
- `..` (two dots) means "the directory one level up (the parent)"

So if you're in `/home/ada/documents`, then `..` refers to `/home/ada`.

---

## Part 2: Essential Navigation Commands

Now that you understand the structure, here are the commands you need to move around in it. Open a terminal and follow along.

### `pwd` — Print Working Directory

**What it does:** Tells you where you are right now.

```bash
pwd
```

Example output:
```
/home/ada
```

Use this whenever you're lost. It's the equivalent of asking "where am I?"

---

### `ls` — List Files and Directories

**What it does:** Shows you what's inside the current directory.

```bash
ls
```

Example output:
```
Desktop  Documents  Downloads  Music  Pictures
```

**Useful options:**

`ls -l` — Long format. Shows extra details like permissions, owner, size, and modification date.
```bash
ls -l
```
```
drwxr-xr-x 2 ada ada 4096 Apr 10 09:21 Documents
-rw-r--r-- 1 ada ada 2048 Apr 18 14:05 notes.txt
```

`ls -a` — All files. Shows hidden files too. In Linux, any file whose name starts with a dot (`.`) is hidden by default.
```bash
ls -a
```
```
.  ..  .bashrc  .profile  Desktop  Documents
```

`ls -la` — Combines both options, showing all files in long format. This is very commonly used.
```bash
ls -la
```

---

### `cd` — Change Directory

**What it does:** Moves you into a different directory.

```bash
cd Documents
```

After running this, you're now inside the `Documents` folder.

**Important `cd` shortcuts:**

Go home:
```bash
cd ~
```
or simply:
```bash
cd
```
Both take you back to your home directory from anywhere.

Go up one level (to the parent directory):
```bash
cd ..
```

Go up two levels:
```bash
cd ../..
```

Go to an absolute path:
```bash
cd /etc
```

Go back to the previous directory you were in:
```bash
cd -
```
This is handy when switching back and forth between two locations.

---

### `mkdir` — Make Directory

**What it does:** Creates a new folder.

```bash
mkdir projects
```

This creates a folder called `projects` in your current location.

Create nested folders all at once using the `-p` flag:
```bash
mkdir -p projects/linux/notes
```
This creates `projects`, then `linux` inside it, then `notes` inside that — even if none of them existed before.

---

### `touch` — Create an Empty File

**What it does:** Creates a new, empty file.

```bash
touch myfile.txt
```

If the file already exists, `touch` just updates its "last modified" timestamp.

---

### `cp` — Copy Files or Directories

**What it does:** Copies a file from one location to another.

```bash
cp notes.txt backup_notes.txt
```

This creates a copy of `notes.txt` called `backup_notes.txt` in the same directory.

Copy to a different directory:
```bash
cp notes.txt Documents/notes.txt
```

Copy an entire directory (use the `-r` flag, which stands for "recursive"):
```bash
cp -r projects projects_backup
```

---

### `mv` — Move or Rename Files

**What it does:** Moves a file to a new location, or renames it. (Unlike `cp`, the original is gone after moving.)

Rename a file:
```bash
mv oldname.txt newname.txt
```

Move a file to a different directory:
```bash
mv notes.txt Documents/
```

Move and rename at the same time:
```bash
mv notes.txt Documents/my_notes.txt
```

---

### `rm` — Remove Files

**What it does:** Deletes a file permanently. There is no Recycle Bin in the Linux terminal — once it's gone, it's gone.

```bash
rm oldfile.txt
```

Delete an entire directory and everything inside it (use `-r` for recursive, and `-f` to skip confirmation prompts):
```bash
rm -rf old_folder
```

> ⚠️ **Be careful with `rm -rf`.** It will delete everything in the specified folder without asking. Double-check what 
you're deleting before running this command.

---

### `cat` — Display File Contents

**What it does:** Prints the contents of a file to your terminal.

```bash
cat notes.txt
```

This is great for quickly reading short files. For longer files, use `less` instead (see below).

---

### `less` — Read Files Page by Page

**What it does:** Opens a file so you can scroll through it at your own pace.

```bash
less long_document.txt
```

- Use the arrow keys or Page Up / Page Down to scroll.
- Press `q` to quit and return to the terminal.
- Press `/` followed by a word to search within the file.

---

### `find` — Search for Files

**What it does:** Searches for files and directories by name, type, or other criteria.

Find a file by name:
```bash
find ~ -name "notes.txt"
```
This searches your entire home directory for a file named `notes.txt`.

Find all `.txt` files:
```bash
find ~ -name "*.txt"
```
The `*` is a wildcard that matches anything.

---

## Part 3: Putting It All Together

Here's a short practice session to get comfortable with everything you've learned.

```bash
# Step 1: Find out where you are
pwd

# Step 2: Look at what's in your home directory
ls -la

# Step 3: Create a practice folder
mkdir linux_practice

# Step 4: Move into it
cd linux_practice

# Step 5: Create a couple of files
touch file1.txt file2.txt

# Step 6: Confirm they're there
ls

# Step 7: Create a subfolder and copy a file into it
mkdir backup
cp file1.txt backup/file1_copy.txt

# Step 8: Look inside the backup folder
ls backup

# Step 9: Go up one level, back to home
cd ..

# Step 10: Confirm you're back home
pwd
```

Work through these steps one at a time, and don't worry about making mistakes — the practice folder is yours to 
experiment with.

---

## Quick Reference Cheat Sheet

| Command | What it does |
|---------|-------------|
| `pwd` | Show current directory |
| `ls` | List files in current directory |
| `ls -la` | List all files with details |
| `cd foldername` | Enter a directory |
| `cd ..` | Go up one level |
| `cd ~` | Go to your home directory |
| `cd -` | Go back to previous directory |
| `mkdir foldername` | Create a new directory |
| `touch filename` | Create an empty file |
| `cp source dest` | Copy a file |
| `cp -r source dest` | Copy a directory |
| `mv source dest` | Move or rename a file |
| `rm filename` | Delete a file |
| `rm -rf foldername` | Delete a directory (careful!) |
| `cat filename` | Print file contents |
| `less filename` | Read file with scrolling |
| `find ~ -name "file"` | Search for a file |

---

## What to Learn Next

Once you're comfortable navigating the filesystem, these topics make great next steps:

- **File permissions** — understanding what `rwxr-xr-x` means and how to change who can read, write, or run a file using `chmod`
- **Piping and redirection** — chaining commands together with `|` and saving output to files with `>` and `>>`
- **`grep`** — a powerful command for searching inside files for specific text
- **Package management** — installing software from the terminal using tools like `apt` or `dnf`
