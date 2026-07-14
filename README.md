# MOS LSI Tools plugin

`mos-lsi-tools` provides a **headless MOS plugin** that integrates Broadcom/LSI command-line utilities into the MOS operating system environment. 

This allows users to easily manage, flash and monitor their LSI Host Bus Adapters (HBAs) directly from the MOS terminal.

---

## Overview

Because these tools are strictly command-line utilities, this plugin is completely **headless**, there should be no dashboard or menu in the MOS web UI. 

### Included Binaries
*   `storcli64` (v007.3703.0000.0000)


---

## Usage

After installing the plugin via the MOS Hub, you must execute the application with root privileges. You can access it globally from your terminal:

```bash
sudo storcli64 /c0 show
