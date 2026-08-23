# eSim 2.5 Installation Issues and Compatibility Fixes on Ubuntu 25.04

## 1. Report Overview

This report documents the investigation, debugging, modification, and verification performed during the installation of **eSim 2.5 on Ubuntu 25.04**.

The purpose of this investigation was to identify compatibility and installation problems in the existing eSim installation process, determine their root causes, apply targeted fixes, and verify the corrected installation flow.

The investigation was performed systematically by:

1. Reproducing the original installation issue
2. Capturing the original error
3. Inspecting the installation scripts
4. Identifying the root cause
5. Creating backups before modifying scripts
6. Applying targeted fixes
7. Re-running the affected installation stage
8. Verifying the result
9. Preserving logs and screenshots as evidence

---

# 2. Environment

| Component | Details |
|---|---|
| Operating System | Ubuntu 25.04 |
| Ubuntu Codename | plucky |
| eSim Version | 2.5 |
| Shell | Bash |
| Platform | Linux |

---

# 3. Issue Summary

| Issue | Component | Problem | Status |
|---|---|---|---|
| Issue 1 | Ubuntu Compatibility | Ubuntu 25.04 not recognized by installer | FIXED |
| Issue 2 | libcanberra | Dependency/installation compatibility issue | FIXED |
| Issue 3A | GHDL | Incorrect source archive path | FIXED |
| Issue 3B | GHDL / LLVM | LLVM compatibility issue | FIXED |
| Issue 3C | Verilator | Archive/path problem | FIXED |
| Issue 3D | NGHDL | Source/archive path problem | FIXED |
| Issue 3E | NGHDL / Ngspice | Executable/path verification issue | VERIFIED |

---

# 4. Investigation Methodology

The troubleshooting process followed a controlled debugging approach:

```text
Original Installation
        ↓
Capture Error
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
Capture Log + Screenshot
        ↓
Proceed to Next Issue
        ↓
Final Tool Verification
```

For each issue, the following information was documented:

* Problem
* Observation
* Investigation
* Root Cause
* Fix
* Verification
* Evidence
* Final Status

---

# 5. Issue 1 — Ubuntu 25.04 Compatibility

## 5.1 Problem

The original eSim installer did not recognize Ubuntu 25.04 as a supported operating-system version.

The installation was started using:

```bash
./install-eSim.sh --install
```

The installer stopped with:

```text
Detected Ubuntu Version:
Unsupported Ubuntu version: 25.04 ()
```

The important observation was that Ubuntu 25.04 was detected, but the complete version information was not displayed correctly.

---

## 5.2 Environment Verification

Before modifying the installer, the operating-system version was independently verified.

The Ubuntu release information was checked using:

```bash
lsb_release -d
```

Output:

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

This confirmed that the operating system itself was correctly reporting Ubuntu 25.04.

Therefore, the problem was determined to be within the eSim installation script rather than the Ubuntu installation.

---

## 5.3 Investigation

The main installation script was inspected to understand:

1. How the Ubuntu version was detected.
2. How the version was converted into a supported installation script.
3. Why Ubuntu 25.04 was rejected.

The relevant version-detection logic contained:

```bash
VERSION_ID=$(grep "^VERSION_ID" /etc/os-release | cut -d '"' -f 2)
FULL_VERSION=$(lsb_release -d | grep -oP '\d+.\d+.\d+')
```

The `FULL_VERSION` pattern expected a three-part version number.

However, Ubuntu 25.04 reports its version as:

```text
25.04
```

which contains two version components.

---

## 5.4 Version Selection Investigation

The installer also contained version-selection logic based on a `case` statement.

The supported Ubuntu releases were checked to determine how the installation script was selected.

Ubuntu 25.04 was not present in the original version-selection logic.

As a result, the installer reached the unsupported-version condition even though the operating system was valid.

---

## 5.5 Root Cause Analysis

Two related compatibility problems were identified.

### Root Cause 1 — Missing Ubuntu 25.04 Support

The original `install-eSim.sh` did not contain a dedicated entry for:

```text
25.04
```

Therefore, Ubuntu 25.04 was treated as an unsupported release.

### Root Cause 2 — Version Pattern Mismatch

The original `FULL_VERSION` extraction used:

```bash
grep -oP '\d+.\d+.\d+'
```

This expected a three-component version.

Ubuntu 25.04 provides:

```text
25.04
```

Therefore, the full version string could remain empty.

This explains the installer output:

```text
Detected Ubuntu Version:
```

followed by the unsupported-version message.

---

# 6. Fix Applied

## 6.1 Ubuntu 25.04 Installation Script

A dedicated Ubuntu 25.04 installation script was introduced:

```text
install-eSim-scripts/install-eSim-25.04.sh
```

The main installer was updated to select this script when Ubuntu 25.04 was detected.

The relevant version-selection logic was:

```bash
"25.04")
    SCRIPT="$SCRIPT_DIR/install-eSim-25.04.sh"
    ;;
```

This allowed the main installer to route Ubuntu 25.04 installations to the dedicated installation script.

---

## 6.2 Ubuntu 25.04 Installation Script

The Ubuntu 25.04-specific installation script was also updated to handle the dependencies and tools required for the installation environment.

---

# 7. KiCad Compatibility

During the Ubuntu 25.04 installation, the existing KiCad PPA-based approach resulted in a dependency conflict.

To avoid the PPA-related dependency issue, the Ubuntu 25.04 installation script was configured to use KiCad packages from the official Ubuntu repository.

The installation command used was:

```bash
sudo apt-get install -y --no-install-recommends \
    kicad \
    kicad-footprints \
    kicad-libraries \
    kicad-symbols \
    kicad-templates
```

KiCad installation and the KiCad GUI were subsequently verified.

---

# 8. Verification

After the modification, Ubuntu 25.04 was verified using:

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

The main installer was then inspected to confirm that Ubuntu 25.04 was mapped to:

```text
install-eSim-scripts/install-eSim-25.04.sh
```

The installation process was able to proceed beyond the original unsupported Ubuntu version error.

---

# 9. Before-Fix vs After-Fix

| Area                   | Before Fix                     | After Fix                             |
| ---------------------- | ------------------------------ | ------------------------------------- |
| Ubuntu 25.04 detection | Version format mismatch        | Ubuntu version correctly recognized   |
| Ubuntu 25.04 selection | Not present                    | Dedicated case added                  |
| Installation script    | Ubuntu 25.04 rejected          | Ubuntu 25.04-specific script selected |
| KiCad installation     | PPA dependency conflict        | Ubuntu repository packages used       |
| Installation flow      | Stopped at compatibility check | Proceeded beyond original issue       |

---

# 10. Evidence

The investigation was supported by screenshots and logs stored in the repository.

## Screenshots

* [Screenshot](<screenshots/Screenshot 1 — Ubuntu version.png>)
* [Screenshot](<screenshots/Screenshot 2 — VERSION_ID.png>)
* [Screenshot](<screenshots/Screenshot 3 — Original eSim error.png>)
* [Screenshot](<screenshots/Screenshot 4 — Original install-eSim.sh.png>)
* [Screenshot](<screenshots/Screenshot 5 — Updated install-eSim.sh.png>)
* [Screenshot](<screenshots/Screenshot 6 — 25.04 script present.png>)
* [Screenshot](<screenshots/Screenshot 7 — KiCad successfully installed.png>)
* [Screenshot](<screenshots/Screenshot 8 — KiCad GUI.png>)


---

# 11. Result

The Ubuntu 25.04 compatibility problem in the original eSim installation flow was resolved.

The installer was modified to:

* Recognize Ubuntu 25.04
* Select a dedicated Ubuntu 25.04 installation script
* Handle the Ubuntu 25.04-specific installation path
* Use an appropriate KiCad installation method

The installation process was able to proceed beyond the original unsupported-version error.

---

# 12. Status

**FIXED**

---

# 13. Key Learning

This issue demonstrated that installation compatibility depends on more than simply detecting the operating-system version.

Both of the following must be handled correctly:

1. Operating-system version parsing
2. Version-specific installation-script selection

A valid operating-system release can still be rejected when the installation script does not explicitly support its version format or does not contain a corresponding installation path.

This investigation also demonstrated the importance of:

* Reproducing the original failure
* Inspecting the actual installation logic
* Identifying the root cause before modifying code
* Keeping backups of original scripts
* Verifying each modification independently
* Maintaining logs and screenshots as evidence

---

# 14. Repository Evidence Structure

The related evidence is organized as follows:

```text
eSim-Task4/
│
├── fixes/
│   └── Installation script backups and fixes
│
├── logs/
│   ├── issue1-original.txt
│   └── issue1-fixed.txt
│
├── notes/
│   └── issue1.md
│
├── report/
│   └── installation-issue.md
│
└── screenshots/
    ├── Original installation evidence
    ├── Investigation evidence
    ├── Fix evidence
    └── Verification evidence
```

---

# 15. Issue Status

**Issue 1 — Ubuntu 25.04 Compatibility: FIXED**

The issue was reproduced, investigated, root-caused, fixed, and verified with supporting logs and screenshots.


