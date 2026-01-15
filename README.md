# MOS DVB Drivers

mos-dvb-drivers provides a **MOS plugin** for managing DVB drivers.

---

## Overview

This repository contains the **MOS plugin implementation**, optional helper
functions, and configuration files (such as `settings.json`) required to
integrate DVB driver management into MOS.

The plugin enables MOS to download and install DVB drivers on demand directly
onto the system.

Currently available driver packages include:

- **LibreELEC**
- **Digital Devices**

### Driver Sources

- LibreELEC: [https://github.com/LibreELEC/dvb-firmware](https://github.com/LibreELEC/dvb-firmware)
- Digital Devices: [https://github.com/DigitalDevices/dddvb](https://github.com/DigitalDevices/dddvb)

No driver binaries are included in this repository.

---

## Build & Automation

This repository includes a **GitHub Actions workflow** used to build and package
the plugin for MOS.

The build process is fully automated and produces artifacts that can be consumed
by the MOS Hub or MOS release tooling.

---

## Licensing

The contents of this repository (plugin code, scripts, and configuration)
are licensed under **GPL-3.0**.

Downloaded DVB drivers remain licensed under their respective upstream licenses
and are not part of this repository.
