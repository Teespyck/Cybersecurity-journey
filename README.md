# Lab 001: Setting Up a Linux Security Lab & Command-Line Fundamentals

> **Program:** Cybersecurity (ICDFA)
> **Registration:** 2026/FCDF/16972
> **Module/Week:** Week 1 — Linux Lab Setup & CLI Orientation
> **Date Completed:** 2026-06-27
> **Difficulty:** ⭐⭐☆☆☆
> **Tools Used:** `VirtualBox` `Kali Linux` `hostnamectl` `uname` `lsblk` `df` `man` `apropos`

---

## 🎯 Objective

This lab covered three foundational areas:

1. **1A** — Installing and validating a Kali Linux virtual machine (VM) using VirtualBox
2. **1B** — Orienting to the Linux command line: understanding who you are, where you are, and what the system is running
3. **1C / 2A–2C** — Using built-in help systems, navigating the filesystem, and discovering the Linux directory structure

The goal was to arrive at a functional, verified lab environment and develop command-line confidence from scratch.

---

## 🧠 Background / Why This Matters

Nearly every cybersecurity tool — vulnerability scanners, packet analysers, exploit frameworks, forensic utilities — runs on Linux. Kali Linux is the industry-standard penetration testing and security research distribution, used by red teams, CTF competitors, and SOC analysts worldwide. Setting it up inside a **virtual machine (VM)** is the standard practice because:

- A VM is **isolated** from your host machine — if something goes wrong (malware, a misconfigured firewall rule, a broken kernel module), you can simply delete and rebuild it without affecting your main OS
- VMs are **snapshottable** — you can take a snapshot of a clean state before running risky commands, and roll back if needed
- Running Kali on a VM means you can safely experiment with tools that would be dangerous to run on a real production system

**Why Kali Linux specifically?** Kali is a Debian-based distribution maintained by Offensive Security, pre-loaded with hundreds of security tools (Wireshark, Nmap, Metasploit, Burp Suite, and many more). Instead of spending hours installing and configuring tools, Kali gives you a ready-to-use security research environment from the moment it boots.

> **Compliance connection:** From a risk and controls perspective, isolated lab environments mirror the concept of **sandboxing** and **environment segregation** — core principles in frameworks like ISO 27001 (Annex A.12.1.4: Separation of development, testing, and operational environments) and NIST SP 800-53 (SC-7: Boundary Protection). What we do manually here with VirtualBox, enterprise teams do with network segmentation, VLANs, and cloud VPCs.

---

## 🛠️ Environment & Setup

| Item | Details |
|------|---------|
| **Hypervisor** | VirtualBox (Type 2 — runs on top of host OS) |
| **Guest OS** | Kali GNU/Linux Rolling (2026.02) |
| **Kernel** | `6.18.12+kali-amd64` |
| **VM Hostname** | `Teebabs` |
| **Architecture** | amd64 (64-bit) |
| **Disk Allocated** | 50GB total (20GB used, 28GB free at setup) |

---

## 📋 Step-by-Step Walkthrough

### Part 1A — Validating the Linux VM

After installing VirtualBox and the Kali Linux ISO, the first job was to verify the environment was running correctly. The commands below are what I used — and what any Linux administrator or security analyst would run when first accessing a new system.

---

#### `hostnamectl` — System Identity

```bash
hostnamectl
```

**Output (summarised):**
```
Static hostname: Teebabs
Operating System: Kali GNU/Linux Rolling
Kernel: Linux 6.18.12+kali-amd64
Architecture: x86-64
```

**What this does:** `hostnamectl` queries **systemd** (the Linux init system and service manager) for authoritative system information. It shows the hostname, OS name, kernel version, and hardware architecture in one clean output.

**Why it matters:** When you first land on a system — whether it's your own VM, a cloud instance, or a compromised host during incident response — this is often the first command you run. It tells you immediately: *What am I looking at? What OS and kernel version? What's this machine called?* In a security context, knowing the kernel version is critical because specific kernel versions have known CVEs (Common Vulnerabilities and Exposures) that an attacker — or a penetration tester — might exploit for privilege escalation.

---

#### `uname -r` — Kernel Version

```bash
uname -r
```

**Output:**
```
6.19.14+kali-amd64
```

**What this does:** `uname` (short for "Unix name") prints system information. The `-r` flag shows just the **kernel release version**.

**Why it matters:** Software packages, kernel modules, and drivers are often version-specific. If you're installing a tool that requires kernel headers, or troubleshooting a driver issue, you need to know the exact kernel version. From a security perspective, this is also how you check whether a system is patched against kernel-level exploits — one of the first things a penetration tester checks.

> **Quick reference — `uname` flags:**
> - `uname -r` → kernel version only
> - `uname -a` → everything (kernel, hostname, architecture, build date)
> - `uname -m` → machine architecture only (e.g., `x86_64`)

---

#### `cat /etc/os-release` — Distribution Information

```bash
cat /etc/os-release
```

**Output (summarised):**
```
NAME="Kali GNU/Linux"
VERSION="2026.02"
ID=kali
ID_LIKE=debian
```

**What this does:** Reads and displays the `/etc/os-release` file — a standardised plain-text file that every modern Linux distribution provides, containing the distribution name, version, and family.

**Why it matters:** `hostnamectl` gives you the OS name, but `cat /etc/os-release` is the raw source of truth. This matters when writing scripts that need to behave differently on different distros — e.g., a setup script that runs `apt install` on Debian-based systems (like Kali, Ubuntu) but `yum install` on Red Hat-based systems (like CentOS, Fedora).

> **What `cat` actually does:** `cat` stands for "concatenate". It reads one or more files and prints their content to the terminal. For a single file, it's essentially "show me this file." It's one of the most-used commands in Linux because configuration, logs, and system information are almost always stored as plain text files.

---

#### `lsblk` — Block Storage Devices

```bash
lsblk
```

**Output (example):**
```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   50G  0 disk
├─sda1   8:1    0   49G  0 part /
└─sda2   8:2    0    1G  0 part [SWAP]
```

**What this does:** `lsblk` (list block devices) shows all storage devices connected to the system — hard drives, SSDs, USB drives, and their partitions — in a tree format.

**Why it matters:** In a security lab, you'll often work with USB drives (for booting live OS environments), disk images (for forensic analysis), or external storage. `lsblk` tells you what's attached, what it's called (device names like `sda`, `sdb`), and whether it's mounted. During incident response or digital forensics, this is one of the first commands run to identify storage media before any analysis begins.

---

#### `df -h /` — Disk Space

```bash
df -h /
```

**Output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   20G   28G  42% /
```

**What this does:** `df` (disk free) reports filesystem disk space usage. The `-h` flag makes the output **human-readable** (GB instead of raw block counts). The `/` argument limits the output to just the root filesystem.

**Why it matters:** Security tools can generate enormous amounts of data — packet captures, log files, scan outputs. Running out of disk space mid-investigation is a real operational problem. Checking available space is a basic hygiene step. Also, in incident response, an unexpectedly full disk can itself be an indicator of compromise (an attacker exfiltrating data, or malware generating large log files to fill the disk and cause a denial of service).

---

### Part 1B — Command-Line Orientation

These commands establish **situational awareness** at the command line — the equivalent of asking "who am I, where am I, and what am I working with?"

| Command | What it tells you | Real-world use |
|---------|-------------------|----------------|
| `whoami` | Your current username | Confirm you're running as the right user before executing commands |
| `id` | Your UID, GID, and all group memberships | Understand what you're authorised to access; check if you're in the `sudo` group |
| `pwd` | Your current directory (full path) | Never get lost; confirm you're in the right place before running commands |
| `echo "$SHELL"` | Your active shell (e.g., `/bin/bash`, `/bin/zsh`) | Scripts written for one shell may not work in another |
| `date` | Current system date and time | Timestamps on log files and forensic evidence depend on accurate system time |
| `history \| tail -n 25` | Last 25 commands you ran | Audit your own session; scripts often use this for reproducibility |

> **Why `id` matters more than `whoami`:** `whoami` just tells you the username. `id` shows your full identity — including every group you belong to. On a Linux system, group membership controls access to files, devices, and capabilities. For example, being in the `sudo` group is what lets you run commands as root. Being in the `wireshark` group is what lets non-root users capture network packets. Group membership is something attackers look for during privilege escalation enumeration.

---

### Part 1C — Using Built-In Help Systems

Linux has multiple parallel help systems, each suited to different situations:

| Help system | How to use | Best for |
|-------------|-----------|---------|
| `man <command>` | `man pwd` | Full, authoritative documentation — use when you need to understand a command deeply |
| `<command> --help` | `ls --help \| head -n 20` | Quick reference — flags and options at a glance |
| `help <builtin>` | `help cd` | Shell built-in commands (they don't have `man` pages because they're part of the shell itself) |
| `apropos <keyword>` | `apropos archive` | Discover commands by topic when you don't know what to look for |
| `whatis <command>` | `whatis tar` | One-line description of what a command does |
| `command -V <cmd>` | `command -V ls` | Find out if something is an external binary or a shell built-in, and where it lives |

#### Why the distinction between built-ins and external commands matters

```bash
command -V ls     # Output: ls is /usr/bin/ls  (external binary)
command -V cd     # Output: cd is a shell builtin
```

`cd` is part of the shell itself — it doesn't exist as a file on disk. This is why `man cd` might not work in all shells (use `help cd` instead). External commands like `ls` are standalone programs that live in places like `/usr/bin/`. The shell finds them by searching through the directories listed in your `$PATH` environment variable.

> **The `apropos` trick:** If you need to do something but don't know what command to use, `apropos <keyword>` searches manual page descriptions. `apropos archive` returns `tar`, `zip`, `gzip`, and related tools. This is how experienced Linux users discover tools they didn't know existed.

---

### Part 2 — Filesystem Navigation & Directory Structure

#### The Linux Filesystem Hierarchy

Linux organises everything under a single root `/`. Understanding this structure is essential — tools, config files, logs, and user data all live in specific, standardised locations.

| Directory | Purpose | Security relevance |
|-----------|---------|-------------------|
| `/` | Root of the entire filesystem — everything lives under here | The top of the permission hierarchy |
| `/etc` | System-wide configuration files | **High value target** — passwords, SSH config, service configs all live here |
| `/home` | Personal directories for each user (e.g., `/home/babs`) | User data; attackers target `/home` for credentials, SSH keys, browser data |
| `/tmp` | Temporary files, world-writable, auto-cleared on reboot | **Common attacker staging area** — malware often writes to `/tmp` because any user can write here |
| `/usr` | User-installed applications and shared libraries | Most tools you install via `apt` land here |
| `/bin` | Essential system binaries (basic commands like `ls`, `cat`, `cp`) | Required for the system to function at all |
| `/sbin` | System administration binaries (used by root: `fdisk`, `ifconfig`) | Privileged tools — regular users generally can't run these |

> **Security note on `/tmp`:** The fact that `/tmp` is world-writable (any user can create files here) makes it a common target in privilege escalation attacks and malware staging. One of the hardening recommendations in the CIS Linux Benchmark is to mount `/tmp` with the `noexec` flag — preventing code placed there from being executed directly.

#### Exit Status — The Silent Signal

One of the most important concepts introduced in this lab was **exit status codes**:

```bash
ls /home          # Works — exit status: 0 (success)
ls /nonexistent   # Fails — exit status: 2 (error)
echo $?           # Prints the exit status of the PREVIOUS command
```

Every command returns an exit status when it finishes:
- `0` = success
- Any non-zero value = failure (the specific number often indicates what went wrong)

**Why this matters:** Exit status codes are the foundation of shell scripting, automation, and monitoring. A backup script that checks `if [ $? -ne 0 ]; then alert...` is using exit status to detect failures. In cybersecurity, this is how SIEM and monitoring tools detect when commands fail — a series of non-zero exit codes for privileged commands can indicate an attacker probing for access.

---

## 🔍 Key Findings / Results

| Task | Status | Notes |
|------|--------|-------|
| VirtualBox installed | ✅ Complete | Host: Windows |
| Kali Linux VM running | ✅ Complete | Hostname: `Teebabs`, Kernel: `6.18.12+kali-amd64` |
| System validation commands | ✅ Complete | `hostnamectl`, `uname -r`, `cat /etc/os-release`, `lsblk`, `df -h /` all executed successfully |
| CLI orientation | ✅ Complete | `whoami`, `id`, `pwd`, `echo "$SHELL"`, `date`, `history` understood and run |
| Help systems | ✅ Complete | `man`, `--help`, `help`, `apropos`, `whatis`, `command -V` all tested |
| Filesystem navigation | ✅ Complete | Linux directory hierarchy understood; `cd`, `pwd`, `ls` practiced |

---

## 🧩 Challenges & How I Solved Them

**Challenge 1:** Understanding the difference between `man` and `help`  
**What confused me:** Running `man cd` didn't work as expected  
**What worked:** `help cd` — because `cd` is a shell built-in, not an external binary, so it doesn't have a traditional `man` page  
**Lesson learned:** Built-ins live inside the shell. External commands live on disk. `command -V <name>` tells you which category any command belongs to.

**Challenge 2:** Reading `lsblk` output — the tree structure was new  
**What confused me:** Multiple `sda` entries (`sda`, `sda1`, `sda2`)  
**What worked:** Understanding that `sda` is the whole disk, while `sda1` and `sda2` are partitions on it — like a physical hard drive being divided into sections  
**Lesson learned:** In Linux, disks are named `sda`, `sdb`, `sdc` (first, second, third drive). Partitions on those disks are numbered: `sda1`, `sda2`. This naming matters when mounting, formatting, or analysing disks in forensic scenarios.

---

## 💡 Key Takeaways

1. **A VM is your safe sandbox** — isolated, snapshottable, and expendable. Always do risky security work in a VM.
2. **Exit status `0` = success; non-zero = failure** — this underpins scripting, automation, and monitoring across all of Linux.
3. **`id` > `whoami`** — group membership is the real picture of what a user is authorised to do on a system.
4. **`/tmp` is world-writable** — convenient for labs, but a known staging area for malware in the real world.
5. **Linux has three help systems: `man` (full docs), `--help` (quick flags), `help` (built-ins)** — knowing which to use saves time and frustration.
6. **`apropos` is the command-line Yellow Pages** — search by topic when you don't know the tool name.

---

## 🔗 Real-World Application

Every component of this lab maps to real professional scenarios:

- **VM setup** = how security teams build isolated testing environments before deploying tools to production
- **System validation commands** = the standard "orientation checklist" when an analyst first SSH's into an unknown host during incident response
- **`/etc` directory awareness** = knowing where credentials, SSH configs, and service configurations live is essential for both defenders (auditing) and attackers (exploitation)
- **Exit status understanding** = foundational for reading SIEM alerts and writing detection logic

From my compliance background: the Linux directory structure, file ownership, and permission model are the *implementation layer* beneath the access control policies I've reviewed in audits. Understanding how `/etc/passwd`, `/etc/sudoers`, and file permissions actually work means I can now verify that what a policy says is happening is what's actually configured on the system.

---

## 📚 Resources & Further Reading

- [The Linux Command Line (free online)](https://linuxcommand.org/tlcl.php) — William Shotts; the best free Linux reference for beginners
- [Kali Linux Official Documentation](https://www.kali.org/docs/) — setup guides and tool references
- [Linux Filesystem Hierarchy Standard](https://refspecs.linuxfoundation.org/fhs.shtml) — the official spec behind the directory structure
- [TryHackMe: Linux Fundamentals](https://tryhackme.com/module/linux-fundamentals) — three free hands-on rooms that build directly on what this lab covers
- [explainshell.com](https://explainshell.com) — paste any command to get a breakdown of every flag and argument

---

## ⏭️ What's Next

Next up: **Lab 002** — continuing with Linux, likely covering file permissions (`chmod`, `chown`), user and group management, and process control. I want to go deeper on how `sudo` is configured (`/etc/sudoers`) because understanding privilege escalation paths starts with understanding how privileges are granted in the first place.

---

## 🏷️ Tags

`cybersecurity` `linux` `kali-linux` `virtualbox` `command-line` `linux-fundamentals` `learning-in-public` `career-change` `bootcamp` `blue-team` `security-operations`

---

*Part of my [Cybersecurity Learning Journey](https://github.com/Teespyck/cybersecurity-journey) — documenting every lab as I transition from Risk & Compliance into Cybersecurity and Cloud Security.*  
*Cross-posted to [dev.to](https://dev.to/teebabs).*
