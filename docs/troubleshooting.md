# Troubleshooting Log

Real, reproducible errors encountered during this build only. Format for each entry below. Copy the template for each new issue.

\---

## Template

### \[Short error title]

**Date:** YYYY-MM-DD
**Phase:** (e.g. VM install / interface config / rule policy / Graylog forwarding)

**Symptom:**
What you saw (exact error text if possible).

**Cause:**
What was actually wrong.

**Fix:**
Exact steps taken to resolve it.

**Verification:**
How you confirmed it was actually fixed (not just "seemed to work").

\---
### VM booted with "Other/Unknown" OS type instead of FreeBSD 64-bit

**Date:** 2026-07-10
**Phase:** VM creation

**Symptom:**
`CPU doesn't support long mode` on boot, VM refused to load kernel in 64-bit mode.

**Cause:**
VM was created with OS type set to "Other/Unknown" instead of BSD/FreeBSD (64-bit). This affects how VirtualBox exposes CPU features to the guest.

**Fix:**
Settings → General → Basic → set OS to BSD family, FreeBSD (64-bit).

**Verification:**
VM progressed past kernel load to the next boot stage.

---

### panic: running without device atpic requires a local APIC

**Date:** 2026-07-10
**Phase:** VM creation

**Symptom:**
Kernel panic immediately after boot, referencing atpic/local APIC.

**Cause:**
I/O APIC was disabled in VM settings — likely a side effect of the VM originally being created under "Other/Unknown" OS type, which doesn't auto-enable it.

**Fix:**
Settings → System → Motherboard → enabled "Enable I/O APIC."

**Verification:**
VM booted past this panic into the installer.

---

### Unable to connect to installer daemon

**Date:** 2026-07-10
**Phase:** pfSense install

**Symptom:**
Installer failed immediately with "unable to connect to installer daemon," retry loop.

**Cause:**
Default PCnet-FAST III virtual NIC type has driver compatibility issues with FreeBSD in this installer version, which needs network access to Netgate's servers during install.

**Fix:**
Changed both network adapters (Adapter 1 and Adapter 2) from PCnet-FAST III to Intel PRO/1000 MT Desktop (82540EM).

**Verification:**
Installer proceeded past this point to disk partitioning.

---

### Install loop after completing installation

**Date:** 2026-07-10
**Phase:** pfSense install

**Symptom:**
After a successful install and reboot, the VM booted straight back into the installer instead of the installed system.

**Cause:**
Installer ISO was still attached to the virtual optical drive, and boot order tried the optical drive first.

**Fix:**
Settings → Storage → removed the ISO attachment, then restarted the VM.

**Verification:**
VM booted into the actual pfSense console (interface assignment menu) instead of the installer.

---

### LAN web GUI unreachable — IP conflict with host

**Date:** 2026-07-10
**Phase:** GUI access

**Symptom:**
Browser showed "refused to connect" / site can't be reached at http://192.168.56.1, despite successful ping to that address.

**Cause:**
Windows host's own VirtualBox host-only adapter (vboxnet0) was already using 192.168.56.1, conflicting with the address assigned to pfSense's LAN interface.

**Fix:**
Reassigned pfSense LAN interface to 192.168.56.2/24 via the console (option 2, Set interface IP address).

**Verification:**
https://192.168.56.2 loaded the pfSense login page successfully.


<!-- Add entries above this line as you hit real issues during the build -->

