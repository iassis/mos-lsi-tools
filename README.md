# MOS LSI Tools plugin

`mos-lsi-tools` provides a **headless MOS plugin** that integrates Broadcom/LSI command-line utilities into the MOS operating system environment. 

This allows users to easily manage, flash and monitor their LSI Host Bus Adapters (HBAs) directly from the MOS terminal.

---

## Overview

Note: This is a CLI-only plugin. It runs entirely via the terminal, so no web GUI components or dashboard pages are installed.

### Included Binaries
* `storcli64` (v007.3703.0000.0000)
* `sas3flash` (vP15)
* `megacli64` (v8.07)

---

## Usage

After installing the plugin via the MOS Hub, you must execute the application with root privileges. You can access it globally from your terminal:

### storcli
```bash
# Show summary of all controllers
sudo storcli64 show

# Show detailed information for Controller 0
sudo storcli64 /c0 show
```

### sas3flash
```bash
# List all Broadcom/LSI adapters detected in the system
sudo sas3flash -listall

# Show detailed information for the primary adapter (including SAS address and firmware version)
sudo sas3flash -list
```

### Megacli
```bash
# View detailed information about all RAID adapters
sudo megacli64 -AdpAllInfo -aALL

# List all physical drives connected to the controllers
sudo megacli64 -PDList -aALL
```
