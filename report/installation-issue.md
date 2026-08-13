````markdown
## Issue 1: Ubuntu 25.04 Not Supported by eSim Installer

### 1. Environment

- Operating System: Ubuntu 25.04
- Ubuntu Codename: plucky
- eSim Version: 2.5

### 2. Problem

While installing eSim 2.5 on Ubuntu 25.04, the installation process reported that Ubuntu 25.04 was not supported.

The installation was started using:

```bash
./install-eSim.sh --install
````

### 3. Observed Error

The installer displayed the following message:

```text
Detected Ubuntu Version:
Unsupported Ubuntu version: 25.04 ()
```

The Ubuntu version was correctly detected as `25.04`, but the full version string was empty.

### 4. Initial Investigation

The main eSim installation script was inspected to understand how the Ubuntu version was detected and how the appropriate installation script was selected.

The following command was used:

```bash
sed -n '1,70p' install-eSim.sh
```

The version detection function contained:

```bash
VERSION_ID=$(grep "^VERSION_ID" /etc/os-release | cut -d '"' -f 2)
FULL_VERSION=$(lsb_release -d | grep -oP '\d+.\d+.\d+')
```

The Ubuntu version was verified separately using:

```bash
lsb_release -d
```

Output:

```text
Description:    Ubuntu 25.04
```

The `VERSION_ID` was also verified using:

```bash
grep '^VERSION_ID' /etc/os-release
```

Output:

```text
VERSION_ID="25.04"
```

Therefore, Ubuntu itself was correctly reporting version `25.04`.

### 5. Version Selection Logic

The installer was also checked to determine which installation script was selected for each Ubuntu version.

The original version-selection logic supported specific Ubuntu releases through a `case` statement.

Ubuntu 25.04 was not handled in the original version-selection logic.

This caused the installer to report Ubuntu 25.04 as unsupported.

### 6. Root Cause

Two problems were identified in the version detection and selection logic:

1. Ubuntu 25.04 was not included in the original version-selection logic.
2. The `FULL_VERSION` detection command expected a three-part version number, while Ubuntu 25.04 reports a two-part version number.

Because of this, the installer displayed:

```text
Detected Ubuntu Version:
```

instead of displaying the complete version.

### 7. Solution

A dedicated Ubuntu 25.04 installation script was added:

```text
install-eSim-scripts/install-eSim-25.04.sh
```

The main `install-eSim.sh` script was updated to handle Ubuntu 25.04:

```bash
"25.04")
    SCRIPT="$SCRIPT_DIR/install-eSim-25.04.sh"
    ;;
```

The Ubuntu 25.04 installation script was also modified to recognize Ubuntu 25.04.

### 8. KiCad Handling for Ubuntu 25.04

The Ubuntu 25.04 installation script was configured to use KiCad from the official Ubuntu repository instead of the KiCad PPA.

The installation command used was:

```bash
sudo apt-get install -y --no-install-recommends kicad kicad-footprints kicad-libraries kicad-symbols kicad-templates
```

This approach was selected because the KiCad PPA caused a dependency conflict on Ubuntu 25.04.

### 9. Verification

The Ubuntu version was verified using:

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

The main installer was checked to confirm that Ubuntu 25.04 was mapped to:

```text
install-eSim-scripts/install-eSim-25.04.sh
```

### 10. Result

After modifying the installation scripts, Ubuntu 25.04 was recognized by the eSim installer and the Ubuntu 25.04-specific installation script could be selected.

The installation process was able to proceed beyond the initial Ubuntu-version compatibility issue.

### 11. Evidence

The following evidence was collected during the investigation:

* Screenshot of the original `Unsupported Ubuntu version: 25.04` error
* Screenshot of `/etc/os-release` showing `VERSION_ID="25.04"`
* Screenshot of `lsb_release -d`
* Screenshot of the original version-selection logic
* Screenshot of the updated `install-eSim.sh`
* Screenshot of `install-eSim-25.04.sh`
* Terminal output showing Ubuntu 25.04 detection

### 12. Status

**FIXED**

The Ubuntu 25.04 compatibility issue in the main eSim installer was resolved by adding Ubuntu 25.04 to the version-selection logic and creating a dedicated Ubuntu 25.04 installation script.

```