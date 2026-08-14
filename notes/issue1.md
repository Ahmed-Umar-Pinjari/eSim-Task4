# Issue 1 — Ubuntu 25.04 Compatibility

## Objective

Investigate why the original eSim 2.5 installer does not recognize Ubuntu 25.04 and determine a compatible solution.

---

## Environment

| Component | Details |
|---|---|
| Operating System | Ubuntu 25.04 |
| Ubuntu Codename | plucky |
| eSim Version | 2.5 |
| Shell | Bash |

---

## Problem

The original eSim installer did not support Ubuntu 25.04.

When the installation was started using:

```bash
./install-eSim.sh --install
```

the installer stopped with:

```text
Unsupported Ubuntu version: 25.04
```

---

## Initial Observation

Ubuntu 25.04 itself was correctly installed and reported by the operating system.

The version was checked using:

```bash
lsb_release -d
```

Expected output:

```text
Description:    Ubuntu 25.04
```

The version ID was also checked using:

```bash
grep '^VERSION_ID' /etc/os-release
```

Output:

```text
VERSION_ID="25.04"
```

Therefore, the problem was not with the Ubuntu installation or version reporting.

---

## Investigation

The main installer script was inspected to understand:

1. How the Ubuntu version was detected.
2. How the installation script was selected for each supported Ubuntu release.
3. Why Ubuntu 25.04 was rejected.

The installer contained version-selection logic based on a `case` statement.

Ubuntu 25.04 was not present in the supported version list.

The version-detection code also used:

```bash
FULL_VERSION=$(lsb_release -d | grep -oP '\d+.\d+.\d+')
```

This pattern expected a three-part version number.

Ubuntu 25.04 uses a two-part version format:

```text
25.04
```

As a result, the `FULL_VERSION` value could remain empty even though Ubuntu correctly reported `25.04`.

---

## Root Cause

Two compatibility problems were identified:

### 1. Missing Ubuntu 25.04 case

The original `install-eSim.sh` did not contain a dedicated entry for:

```text
25.04
```

Therefore, the installer treated Ubuntu 25.04 as unsupported.

### 2. Incompatible full-version pattern

The original `FULL_VERSION` extraction expected a three-component version number.

Ubuntu 25.04 provides:

```text
25.04
```

rather than:

```text
25.xx.xx
```

Therefore, the version information was not displayed correctly in the installer output.

---

## Fix

A dedicated Ubuntu 25.04 installation script was introduced:

```text
install-eSim-scripts/install-eSim-25.04.sh
```

The main installer was updated so that Ubuntu 25.04 is mapped to the new installation script.

The relevant logic was:

```bash
"25.04")
    SCRIPT="$SCRIPT_DIR/install-eSim-25.04.sh"
    ;;
```

The Ubuntu 25.04-specific installation script was then updated for the dependencies and tools required by the setup.

---

## KiCad Compatibility

During the Ubuntu 25.04 setup, KiCad installation through the existing PPA-based approach caused a dependency conflict.

The Ubuntu 25.04 installation script was therefore configured to use KiCad packages from the official Ubuntu repository.

The package installation used was:

```bash
sudo apt-get install -y --no-install-recommends \
    kicad \
    kicad-footprints \
    kicad-libraries \
    kicad-symbols \
    kicad-templates
```

The final installation log confirmed that the Ubuntu repository version of KiCad was available/installed.

---

## Verification

Ubuntu 25.04 was verified using:

```bash
lsb_release -rs
```

Output:

```text
25.04
```

The version ID was also verified using:

```bash
grep '^VERSION_ID' /etc/os-release
```

Output:

```text
VERSION_ID="25.04"
```

The main installer was then checked to confirm that Ubuntu 25.04 was mapped to:

```text
install-eSim-scripts/install-eSim-25.04.sh
```

---

## Evidence

The investigation was documented using:

[Screenshot](<screenshots/Screenshot 1 — Ubuntu version.png>)
[Screenshot](<screenshots/Screenshot 2 — VERSION_ID.png>)
[Screenshot](<screenshots/Screenshot 3 — Original eSim error.png>)
[Screenshot](<screenshots/Screenshot 4 — Original install-eSim.sh.png>)
[Screenshot](<screenshots/Screenshot 5 — Updated install-eSim.sh.png>)
[Screenshot](<screenshots/Screenshot 6 — 25.04 script present.png>)
[Screenshot](<screenshots/Screenshot 7 — KiCad successfully installed.png>)
[Screenshot](<screenshots/Screenshot 8 — KiCad GUI.png>)

[log](logsissue1-original.txt)
[log](logsissue1-fixed.txt)

---

## Result

The Ubuntu 25.04 compatibility issue was resolved.

The installer was modified to recognize Ubuntu 25.04 and select a dedicated Ubuntu 25.04 installation script.

The installation process was able to proceed beyond the original unsupported-version error.

---

## Status

**FIXED**

---

## Key Learning

This issue demonstrated the importance of checking both:

* Operating-system version detection
* Version-selection logic inside installation scripts

An installer may correctly detect an operating-system version but still reject it if that version is not explicitly handled by the script.

# eSim 2.5 Installation Issues and Compatibility Fixes on Ubuntu 25.04

## 1. Project Overview

This report documents the investigation, debugging, modification, and verification performed while installing **eSim 2.5 on Ubuntu 25.04**.

The objective was to identify installation and dependency issues, determine their root causes, apply appropriate fixes, and verify the affected components.

The troubleshooting process preserved:

- Original installation behaviour
- Error messages
- Installation logs
- Script modifications
- Before-fix and after-fix evidence
- Component-level verification
- Final tool verification

---

# 2. Task Objective

The objective of this task was to investigate the problems encountered while installing eSim 2.5 on Ubuntu 25.04 and fix installation issues.

The investigation focused on:

1. Ubuntu 25.04 compatibility
2. Dependency compatibility
3. GHDL installation
4. LLVM compatibility
5. Verilator installation
6. NGHDL installation
7. Ngspice/NGHDL executable paths
8. Final installation verification

---

# 3. Environment

| Component | Version / Details |
|---|---|
| Operating System | Ubuntu 25.04 |
| Codename | plucky |
| eSim | 2.5 |
| GHDL | 4.1.0 |
| LLVM | LLVM 18 |
| Verilator | 4.210 |
| Ngspice | ngspice-35 |
| NGHDL | Successfully installed |
| Shell | Bash |
| Platform | Linux |

---

# 4. Investigation Methodology

The troubleshooting process followed this sequence:

```text
Original Installation
        ↓
Capture Initial Error
        ↓
Identify Affected Component
        ↓
Inspect Installation Script
        ↓
Determine Root Cause
        ↓
Backup Original Script
        ↓
Apply Targeted Fix
        ↓
Run Installation Again
        ↓
Verify Component
        ↓
Save Log + Screenshot
        ↓
Continue to Next Issue
        ↓
Perform Final Verification
````

For every major issue, the following information was recorded:

* Problem
* Observation
* Root cause
* Fix
* Verification
* Evidence
* Final status

---

# 5. Issue Summary

| Issue    | Problem                        | Fix Status |
| -------- | ------------------------------ | ---------- |
| Issue 1  | Ubuntu 25.04 not recognized    | FIXED      |
| Issue 2  | libcanberra dependency problem | FIXED      |
| Issue 3A | GHDL archive path problem      | FIXED      |
| Issue 3B | LLVM compatibility problem     | FIXED      |
| Issue 3C | Verilator archive/path problem | FIXED      |
| Issue 3D | NGHDL source/path problem      | FIXED      |
| Issue 3E | Ngspice/NGHDL executable path  | FIXED      |

---

# 6. Issue 1 — Ubuntu 25.04 Compatibility

## Problem

The original eSim installer did not recognize Ubuntu 25.04.

The installation produced:

```text
Unsupported Ubuntu version: 25.04
```

## Investigation

The operating-system version was verified using:

```bash
lsb_release -d
```

and:

```bash
grep '^VERSION_ID' /etc/os-release
```

Ubuntu correctly reported:

```text
25.04
```

The installation script was then inspected.

Ubuntu 25.04 was missing from the version-selection `case` statement.

The original full-version extraction also expected a three-component version format, which did not match Ubuntu 25.04.

## Root Cause

Two compatibility problems were identified:

1. Ubuntu 25.04 was not included in the version-selection logic.
2. The full-version extraction pattern was not suitable for the `25.04` version format.

## Fix

A dedicated Ubuntu 25.04 installation script was introduced:

```text
install-eSim-scripts/install-eSim-25.04.sh
```

The main installer was updated to select it:

```bash
"25.04")
    SCRIPT="$SCRIPT_DIR/install-eSim-25.04.sh"
    ;;
```

## KiCad Handling

The Ubuntu 25.04-specific installation path was configured to use KiCad packages from the official Ubuntu repository instead of the problematic PPA-based path.

## Verification

Ubuntu 25.04 detection and installer mapping were verified.

## Evidence

[Screenshot](<screenshots/Screenshot 1 — Ubuntu version.png>)
[Screenshot](<screenshots/Screenshot 2 — VERSION_ID.png>)
[Screenshot](<screenshots/Screenshot 3 — Original eSim error.png>)
[Screenshot](<screenshots/Screenshot 4 — Original install-eSim.sh.png>)
[Screenshot](<screenshots/Screenshot 5 — Updated install-eSim.sh.png>)
[Screenshot](<screenshots/Screenshot 6 — 25.04 script present.png>)
[Screenshot](<screenshots/Screenshot 7 — KiCad successfully installed.png>)
[Screenshot](<screenshots/Screenshot 8 — KiCad GUI.png>)

[log](logsissue1-original.txt)
[log](logsissue1-fixed.txt)

## Status

**FIXED**

---

# 7. Issue 2 — libcanberra Dependency

## Problem

The installation encountered a missing `libcanberra` / related dependency issue.

## Investigation

The dependency failure was reproduced and the installation script and package requirements were examined.

The required dependency handling was modified for Ubuntu 25.04.

## Fix

The installation process was updated to correctly handle the dependency on Ubuntu 25.04.

The original script was backed up before modification.

## Verification

The corrected dependency installation was executed successfully and package availability was verified.

## Evidence

[Screenshot](<screenshots/Screenshot 9 - Issue 2 libcanberra error.png>)
[Screenshot](<screenshots/Screenshot 10 - 25.04 canberra dependency before fix.png>)
[Screenshot](<screenshots/Screenshot 11 - 25.04 script backup.png>)
[Screenshot](<screenshots/Screenshot 12 - 25.04 canberra dependency fixed.png>)
[Screenshot](<screenshots/Screenshot 13 - 25.04 after fix backup.png>)
[Screenshot](<screenshots/Screenshot 14 - Issue 2 fixed installation result.png>)
[Screenshot](<screenshots/Screenshot 15 - libcanberra package verification.png>)

[log](logsissue2-canberra-fixed-installation.txt)
[log](logsissue2-version-check.txt)
[log](logsissue2-fixed-installation.txt)

## Status

**FIXED**

---

# 8. Issue 3 — GHDL Installation

## 8.1 GHDL Archive Path Problem

### Problem

The original installation function expected the GHDL source archive at a path that did not match the actual archive location.

### Investigation

The archive was checked for availability and its actual location was identified.

The installation function was inspected before applying the modification.

### Root Cause

The installer used an incorrect source path for the GHDL archive.

### Fix

The extraction path was corrected to use the archive under the eSim source directory:

```bash
tar -xJf "$src_dir/ghdl/$ghdl-source.tar.xz" -C "$HOME"
```

### Evidence

[Screenshot](<screenshots/Screenshot 16 - Issue 2 fixed and GHDL tar error.png>)
[Screenshot](<screenshots/Screenshot 17 - GHDL archive check.png>)
[Screenshot](<screenshots/Screenshot 18 - GHDL archive missing.png>)
[Screenshot](<screenshots/Screenshot 19 - GHDL archive present but wrong path.png>)
[Screenshot](<screenshots/Screenshot 20 - GHDL install function before fix.png>)
[Screenshot](<screenshots/Screenshot 21 - GHDL before-fix backup.png>)
[Screenshot](<screenshots/Screenshot 22 - GHDL archive path fixed.png>)
[Screenshot](<screenshots/Screenshot 23 - GHDL after-fix backup.png>)

---

# 8.2 LLVM 18 Compatibility

## Problem

Ubuntu 25.04 provided a newer LLVM toolchain, while the GHDL build used in this setup required LLVM 18.

## Investigation

LLVM and Clang versions were checked.

LLVM 18 availability was verified and LLVM 18 was installed.

The GHDL source was then configured and built using LLVM 18.

## Verification

The GHDL build and installation completed successfully.

The installed GHDL version was verified as:

```text
GHDL 4.1.0
```

## Evidence

[Screenshot](<screenshots/Screenshot 24 - LLVM and Clang versions.png>)
[Screenshot](<screenshots/Screenshot 25 - GHDL configure source proof.png>)
[Screenshot](<screenshots/Screenshot 27 - LLVM 18 availability.png>)
[Screenshot](<screenshots/Screenshot 28 - LLVM 18 installation.png>)
[Screenshot](<screenshots/Screenshot 29 - LLVM 18 version check.png>)
[Screenshot](<screenshots/Screenshot 30 - LLVM 18 paths and versions.png>)
[Screenshot](<screenshots/Screenshot 31 - GHDL source location and LLVM18.png>)
[Screenshot](<screenshots/Screenshot 32 - GHDL configure with LLVM18 successful.png>)
[Screenshot](<screenshots/Screenshot 33 - GHDL LLVM18 build successful.png>)
[Screenshot](<screenshots/Screenshot 34 - GHDL LLVM18 installation verification.png>)
[Screenshot](<screenshots/Screenshot 35 - GHDL still using LLVM20 error.png>)
[Screenshot](<screenshots/Screenshot 36 - 25.04 script updated for LLVM18.png>)

Relevant logs:

[log](logsissue3-llvm18-availability.txt)
[log](logsissue3-llvm18-installed.txt)
[log](logsissue3-llvm-version-proof.txt)
[log](logsissue3-ghdl-llvm18-build.txt)
[log](logsissue3-ghdl-llvm18-install.txt)
[log](logsissue3-after-llvm18-script-fix.txt)

## Status

**FIXED**

---

# 8.3 Verilator Installation

## Problem

The original Verilator installation function expected the source archive at an unavailable or incorrect path.

## Investigation

The archive location and installation function were inspected.

The original installation script was backed up before modification.

## Fix

The Verilator archive/path handling was corrected.

## Verification

The installation was verified using:

```bash
verilator --version
```

Output:

```text
Verilator 4.210
```

## Evidence

[Screenshot](<screenshots/Screenshot 37 - Verilator archive missing error.png>)
[Screenshot](<screenshots/Screenshot 38 - Verilator archive check.png>)
[Screenshot](<screenshots/Screenshot 39 - Verilator install function.png>)
[Screenshot](<screenshots/Screenshot 40 - Verilator script backup.png>)
[Screenshot](<screenshots/Screenshot 41 - Verilator path fix.png>)
[Screenshot](<screenshots/Screenshot 42 - Verilator path and archive check.png>)
[Screenshot](<screenshots/Screenshot 43 - Verilator fixed installation NGHDL error.png>)
[Screenshot](<screenshots/Screenshot_53_Verilator_Installation_Success.png>)

Relevant logs:

[log](logsissue3-verilator-path-check.txt)
[log](logsissue3-verilator-fixed-installation.txt)

## Status

**FIXED**

---

# 8.4 NGHDL Installation

## Problem

After Verilator installation, NGHDL installation encountered a source/archive path problem.

The installation function could not locate:

```text
nghdl-simulator-source.tar.xz
```

## Investigation

The NGHDL source archive location was checked and the installation function was inspected.

## Fix

The NGHDL installation path was corrected to use the actual source archive location.

The NGHDL installation was then completed successfully.

## Verification

The installed NGHDL executable was verified at:

```text
/usr/local/bin/nghdl
```

## Evidence

[Screenshot](<screenshots/Screenshot 44 - NGHDL script backup.png>)
[Screenshot](<screenshots/Screenshot 45 - NGHDL install path function.png>)
[Screenshot](<screenshots/Screenshot 46 — NGHDL path fix.png>)
[Screenshot](<screenshots/Screenshot 47 — NGHDL installed, post-install path error.png>)
[Screenshot](<screenshots/Screenshot 50 — NGHDL Fixed Installation Success.png>)
[Screenshot](<screenshots/Screenshot_52_NGHDL_Installation_Success.png>)

Relevant logs:

[log](logsissue3-nghdl-path-fixed-installation.txt)
[log](logsissue3-nghdl-fixed-installation-v2.txt)

## Status

**FIXED**

---

# 8.5 Ngspice / NGHDL Path Verification

## Problem

After NGHDL installation, executable path and NGHDL/Ngspice soft-link behaviour required verification.

## Verification

The NGHDL executable was verified using:

```bash
which nghdl
```

Output:

```text
/usr/local/bin/nghdl
```

Ngspice was verified using:

```bash
which ngspice
```

Output:

```text
/usr/bin/ngspice
```

The final installation logs also recorded successful soft-link creation for Ngspice and NGHDL.

## Evidence

[Screenshot](<screenshots/Screenshot 48 — NGSpice Path Fix Verified.png>)
[Screenshot](<screenshots/Screenshot 49 — NGSpice Path Fix Applied.png>)
[Screenshot](<screenshots/Screenshot 51 — NGHDL Ngspice Verification.png>)

## Status

**FIXED / VERIFIED**

---

# 9. Final Verification

The final installation environment was checked using the installed tools.

## GHDL

```bash
ghdl --version
```

Verified:

```text
GHDL 4.1.0
```

## Verilator

```bash
verilator --version
```

Verified:

```text
Verilator 4.210
```

## Ngspice

```bash
ngspice -v
```

Verified:

```text
ngspice-35
```

## NGHDL

```bash
which nghdl
```

Verified:

```text
/usr/local/bin/nghdl
```

## Final Verification Command

```bash
echo "=== FINAL eSim TOOL VERIFICATION ==="

echo "--- GHDL ---"
ghdl --version | head -n 3

echo "--- Verilator ---"
verilator --version

echo "--- Ngspice ---"
ngspice -v | head -n 8

echo "--- NGHDL ---"
which nghdl
```

## Final Evidence

[Screenshot](<screenshots/Screenshot_54_Final_Verification.png>)
[Screenshot](<screenshots/Screenshot_55_Final_eSim_Tool_Verification.png>)

[log](logsissue3-nghdl-fixed-installation-v2.txt)

---

# 10. Final Installation Status

The final setup successfully verified the required eSim-related toolchain:

| Component                  | Final Status |
| -------------------------- | ------------ |
| Ubuntu 25.04 compatibility | ✅ Fixed      |
| libcanberra dependency     | ✅ Fixed      |
| GHDL installation          | ✅ Verified   |
| LLVM 18 compatibility      | ✅ Fixed      |
| Verilator                  | ✅ Verified   |
| NGHDL                      | ✅ Verified   |
| Ngspice                    | ✅ Verified   |
| NGHDL/Ngspice paths        | ✅ Verified   |
| Final tool verification    | ✅ Successful |

---

# 11. Before-Fix vs After-Fix

| Component      | Before Fix               | After Fix              |
| -------------- | ------------------------ | ---------------------- |
| Ubuntu 25.04   | Unsupported              | Recognized             |
| libcanberra    | Dependency failure       | Dependency handled     |
| GHDL archive   | Incorrect path           | Correct path           |
| GHDL toolchain | LLVM compatibility issue | Built with LLVM 18     |
| Verilator      | Archive/path failure     | Installed and verified |
| NGHDL          | Source/path failure      | Installed and verified |
| Ngspice        | Path/soft-link issue     | Verified               |
| Final tools    | Installation incomplete  | Toolchain verified     |

---

# 12. Evidence and Reproducibility

This repository intentionally preserves both **before-fix and after-fix evidence**.

### fixes/

Contains:

* Original installation scripts
* Backup versions
* Modified installation scripts
* Corrected installation scripts

### logs/

Contains:

* Original installation errors
* Intermediate debugging output
* Dependency installation logs
* Build logs
* Final verification logs

### screenshots/

Contains:

* Original errors
* Version checks
* Script inspection
* Fix implementation
* Installation results
* Final verification

### notes/

Contains issue-specific investigation notes.

### report/

Contains the consolidated technical report.

---

# 13. Repository Structure

```text
eSim-Task4/
│
├── fixes/
│   ├── Original scripts
│   ├── Backup scripts
│   └── Corrected scripts
│
├── logs/
│   ├── Original errors
│   ├── Debugging logs
│   ├── Installation logs
│   └── Verification logs
│
├── notes/
│   └── issue1.md
│
├── report/
│   └── installation-issue.md
│
├── screenshots/
│   ├── Before-fix evidence
│   ├── Fix evidence
│   └── Final verification
│
└── README.md
```

---

# 14. Key Technical Learnings

This investigation provided practical experience with:

* Bash scripting
* Linux shell debugging
* Ubuntu version compatibility
* Package management using APT
* Installation-script debugging
* Source archive management
* Path and file-location debugging
* LLVM/GHDL toolchain compatibility
* Building software from source
* Verilator installation
* NGHDL installation
* Ngspice integration
* Symbolic links
* Executable path verification
* Installation log analysis
* Screenshot-based technical documentation
* Git and GitHub version control
* Reproducible troubleshooting

---

# 15. Conclusion

The investigation identified multiple compatibility, dependency, archive-path, compiler-toolchain, simulator, and post-installation path issues while installing eSim 2.5 on Ubuntu 25.04.

The issues were investigated individually using installation logs, source-script inspection, controlled modifications, and repeated verification.

Multiple installation issues were successfully resolved, including Ubuntu 25.04 compatibility, dependency handling, GHDL/LLVM compatibility, Verilator installation, NGHDL installation, and executable-path verification.

The final environment successfully verified:

```text
GHDL 4.1.0
Verilator 4.210
Ngspice-35
NGHDL
```

The repository preserves the complete troubleshooting trail so that the investigation can be reviewed and reproduced.

---

# 16. Author

**Ahmed Umar Pinjari**

Technical troubleshooting and documentation record for:

**eSim 2.5 — Ubuntu 25.04 Installation and Compatibility Fixes**