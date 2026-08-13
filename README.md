````markdown
# eSim 2.5 — Ubuntu 25.04 Installation & Compatibility Fixes

## Overview

This repository documents the investigation, debugging, fixes, and final verification performed to install and run **eSim 2.5 on Ubuntu 25.04**.

The original eSim installation script was designed around older Ubuntu/toolchain assumptions. During installation on Ubuntu 25.04, multiple dependency, archive-path, compiler, simulator, and post-install path issues were encountered.

This repository maintains the complete troubleshooting trail, including:

- Original installation evidence
- Modified installation scripts
- Error logs
- Fix-by-fix verification logs
- Screenshots as evidence
- Final tool-version verification

---

## Environment

| Component | Version / Details |
|---|---|
| Operating System | Ubuntu 25.04 |
| eSim | eSim 2.5 |
| GHDL | 4.1.0 |
| LLVM | LLVM 18 |
| Verilator | 4.210 |
| Ngspice | ngspice-35 |
| Shell | Bash |
| Architecture | Linux |

---

# Problems Identified

During the eSim installation process, multiple issues were identified and investigated.

### Issue 1 — Ubuntu 25.04 compatibility

The original eSim installation process encountered compatibility problems on Ubuntu 25.04.

The original installation behaviour was captured before applying modifications.

Evidence:

- `screenshots/Screenshot 1 — Ubuntu version.png`
- `screenshots/Screenshot 3 — Original eSim error.png`
- `logs/issue1-original.txt`

---

### Issue 2 — libcanberra dependency

The installation encountered a missing `libcanberra` / related dependency issue.

The dependency was investigated, fixed, and verified on Ubuntu 25.04.

Evidence:

- `screenshots/Screenshot 9 - Issue 2 libcanberra error.png`
- `screenshots/Screenshot 10 - 25.04 canberra dependency before fix.png`
- `screenshots/Screenshot 12 - 25.04 canberra dependency fixed.png`
- `screenshots/Screenshot 15 - libcanberra package verification.png`
- `logs/issue2-canberra-fixed-installation.txt`

---

# Issue 3 — GHDL Installation

## Problem

The original installation script expected the GHDL archive in a location that did not match the actual archive location.

The investigation identified:

- GHDL archive availability
- Incorrect archive path
- GHDL installation function
- LLVM compatibility
- GHDL build and installation verification

### GHDL Archive Path Fix

The GHDL archive was located under the eSim source directory rather than the path expected by the original installation function.

The installation script was updated to use the correct source path.

Relevant fix:

```bash
tar -xJf "$src_dir/ghdl/$ghdl-source.tar.xz" -C "$HOME"
````

Evidence:

* `screenshots/Screenshot 18 - GHDL archive missing.png`
* `screenshots/Screenshot 19 - GHDL archive present but wrong path.png`
* `screenshots/Screenshot 20 - GHDL install function before fix.png`
* `screenshots/Screenshot 22 - GHDL archive path fixed.png`

---

# LLVM 18 Compatibility

Ubuntu 25.04 contains a newer LLVM toolchain, while the required GHDL build process in this setup required LLVM 18.

LLVM 18 availability and installation were verified before rebuilding GHDL.

The GHDL source was configured and built using LLVM 18.

Evidence:

* `screenshots/Screenshot 27 - LLVM 18 availability.png`
* `screenshots/Screenshot 28 - LLVM 18 installation.png`
* `screenshots/Screenshot 29 - LLVM 18 version check.png`
* `screenshots/Screenshot 30 - LLVM 18 paths and versions.png`
* `screenshots/Screenshot 31 - GHDL source location and LLVM18.png`
* `screenshots/Screenshot 32 - GHDL configure with LLVM18 successful.png`
* `screenshots/Screenshot 33 - GHDL LLVM18 build successful.png`
* `screenshots/Screenshot 34 - GHDL LLVM18 installation verification.png`

---

# Verilator Installation

## Problem

The Verilator installation function initially failed because the expected Verilator archive was not available at the path used by the script.

The archive location was investigated and the installation path was corrected.

The final installation was verified using:

```bash
verilator --version
```

Verified version:

```text
Verilator 4.210
```

Evidence:

* `screenshots/Screenshot 37 - Verilator archive missing error.png`
* `screenshots/Screenshot 38 - Verilator archive check.png`
* `screenshots/Screenshot 39 - Verilator install function.png`
* `screenshots/Screenshot 41 - Verilator path fix.png`
* `screenshots/Screenshot 42 - Verilator path and archive check.png`
* `screenshots/Screenshot 53_Verilator_Installation_Success.png`

---

# NGHDL Installation

## Problem

After Verilator installation, NGHDL installation initially failed because the script could not locate:

```text
nghdl-simulator-source.tar.xz
```

The NGHDL source archive location was investigated and the installation function was updated to use the correct source directory.

The NGHDL installation process was then completed successfully.

The final executable was verified at:

```text
/usr/local/bin/nghdl
```

Evidence:

* `screenshots/Screenshot 43 - Verilator fixed installation NGHDL error.png`
* `screenshots/Screenshot 44 - NGHDL script backup.png`
* `screenshots/Screenshot 45 - NGHDL install path function.png`
* `screenshots/Screenshot 46 — NGHDL path fix.png`
* `screenshots/Screenshot 50 — NGHDL Fixed Installation Success.png`
* `screenshots/Screenshot_52_NGHDL_Installation_Success.png`

---

# Ngspice Path Fix

The installation process also required verification of the Ngspice executable path and NGHDL/Ngspice soft links.

The final verification confirmed:

```bash
which nghdl
```

Output:

```text
/usr/local/bin/nghdl
```

and:

```bash
which ngspice
```

Output:

```text
/usr/bin/ngspice
```

Evidence:

* `screenshots/Screenshot 48 — NGSpice Path Fix Verified.png`
* `screenshots/Screenshot 49 — NGSpice Path Fix Applied.png`
* `screenshots/Screenshot 51 — NGHDL Ngspice Verification.png`

---

# Final Verification

The final environment was verified using the installed tools.

### GHDL

```text
GHDL 4.1.0
```

### Verilator

```text
Verilator 4.210
```

### Ngspice

```text
ngspice-35
```

### NGHDL

```text
/usr/local/bin/nghdl
```

The final installation log records:

```text
GHDL installed successfully
Verilator installed successfully
Added softlink for Ngspice.....
Added softlink for NGHDL.....
```

Evidence:

* `screenshots/Screenshot_54_Final_Verification.png`
* `screenshots/Screenshot_55_Final_eSim_Tool_Verification.png`
* `logs/issue3-nghdl-fixed-installation-v2.txt`

---

# Repository Structure

```text
eSim-Task4/
│
├── fixes/
│   ├── Original installation scripts
│   ├── Backup versions
│   └── Corrected installation scripts
│
├── logs/
│   ├── Original error logs
│   ├── Intermediate debugging logs
│   └── Final installation and verification logs
│
├── notes/
│   └── Issue-specific notes
│
├── report/
│   └── installation-issue.md
│
├── screenshots/
│   ├── Original errors
│   ├── Fix implementation
│   ├── Installation verification
│   └── Final tool verification
│
└── README.md
```

---

# Debugging Approach

The troubleshooting process followed a structured approach:

```text
Original eSim Installation
          ↓
Identify Installation Error
          ↓
Capture Error Evidence
          ↓
Inspect Original Script
          ↓
Locate Missing / Incorrect Path
          ↓
Create Backup of Script
          ↓
Apply Targeted Fix
          ↓
Run Installation Again
          ↓
Verify Individual Component
          ↓
Record Logs + Screenshot
          ↓
Proceed to Next Dependency
          ↓
Final Tool Verification
```

---

# Key Technical Learnings

This task provided practical experience with:

* Bash scripting and shell debugging
* Linux package management
* Ubuntu version compatibility
* Source archive/path management
* Building software from source
* GHDL and LLVM toolchain compatibility
* Verilator installation
* NGHDL simulator setup
* Ngspice integration
* Symbolic links and executable paths
* Installation log analysis
* Reproducible troubleshooting documentation
* Git version control and evidence management

---

# Evidence & Reproducibility

This repository intentionally preserves both **before-fix and after-fix evidence**.

The `fixes/` directory contains script versions used during the investigation.

The `logs/` directory contains installation and verification outputs.

The `screenshots/` directory provides visual evidence for the major debugging stages.

The `report/` directory contains the consolidated installation issue documentation.

This structure makes it possible to follow the troubleshooting process from the original failure to the final verified installation.

---

# Final Status

## Installation Status: SUCCESSFUL

The required eSim-related tools were successfully installed and verified in the final setup:

* ✅ GHDL 4.1.0
* ✅ Verilator 4.210
* ✅ Ngspice-35
* ✅ NGHDL
* ✅ NGHDL/Ngspice soft links
* ✅ Final eSim tool verification

---

## Author

**Ahmed Umar Pinjari**


```markdown
This repository was created as a technical troubleshooting and documentation record for eSim 2.5 installation on Ubuntu 25.04.