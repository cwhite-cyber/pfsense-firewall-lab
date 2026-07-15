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

### pfSense VM disk corruption after improper shutdown

**Date:** 2026-07-14/15
**Phase:** Post-configuration (after firewall rules were built and verified)

**Symptom:**
pfSense failed to boot with `ZFS I/O error - all block copies unavailable`, `elf64_loadimage: read failed`, `can't load 'kernel'`. VM was unrecoverable via snapshot (an empty/current-state-only snapshot existed and did not provide a working restore point).

**Cause:**
The VM window was closed during an active boot/write process rather than using a proper Halt (option 6) or ACPI shutdown, corrupting the virtual disk's filesystem.

**Fix:**
No repair was possible. Removed the corrupted VM and its files entirely (`VBoxManage unregistervm --delete`, plus manual cleanup of an orphaned VM folder that survived the initial removal), and rebuilt pfSense from the original ISO. Existing documentation (this repo) served as the rebuild guide, reducing rebuild time from several hours (original build) to under an hour.

**Verification:**
Fresh install completed, interfaces reassigned (WAN=em0, LAN=em1), LAN reconfigured to 192.168.56.2/24, firewall rules recreated (Kali -> theone SSH allow, default-allow rules disabled).

**Lesson:** Always use Halt system (or ACPI shutdown) before closing a VM window - never close a VM abruptly while it may be mid-write. Take snapshots only after a clean halt, not while a VM is running or mid-process.

---

### Kali retaining internet access despite correct default-deny rules

**Date:** 2026-07-15
**Phase:** Post-rebuild rule verification

**Symptom:**
After rebuilding pfSense's firewall rules (SSH-only allow for Kali -> theone, default-allow rules disabled and confirmed applied), Kali could still successfully ping 8.8.8.8 and resolve DNS - traffic that should have been blocked by the implicit deny rule.

**Cause:**
The Kali VM had a second network adapter (Adapter 2) enabled and attached to NAT, in addition to the correct host-only Adapter 1. This gave Kali a direct path to the internet that bypassed pfSense's LAN interface entirely, so no firewall rule on pfSense could affect that traffic.

**Fix:**
Disabled/detached Kali's Adapter 2 (NAT) in VirtualBox, leaving only Adapter 1 (Host-only, vboxnet0) active.

**Verification:**
`ip a` on Kali confirmed only one interface (eth0) present. `ip route` confirmed the default route pointed via 192.168.56.2 (pfSense LAN). Retested: `ping 8.8.8.8` failed (unreachable), `ping google.com` failed (DNS resolution failure - also correctly blocked), `ssh cdub@192.168.56.102` succeeded. Confirms the default-deny rule set was working correctly all along; the leak was a VM configuration issue, not a firewall rule issue.

---

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

\---

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

\---

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

\---

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

\---

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

