# eSim 2.5 Installation Issues on Ubuntu 25.04

## Issue 1: Ubuntu 25.04 Unsupported

### Environment
- OS: Ubuntu 25.04
- Codename: plucky
- eSim Version: 2.5

### Problem
Running:

./install-eSim.sh --install

produces:

Unsupported Ubuntu version: 25.04

### Initial Investigation
The installer script supports:
- Ubuntu 22.04
- Ubuntu 23.04
- Ubuntu 24.04

Ubuntu 25.04 is not included in the case statement.

### Status
Issue reproduced successfully.
Further investigation required.
