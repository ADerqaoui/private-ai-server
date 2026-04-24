# Ubuntu to Debian: a switch driven by ACPI

*A post-mortem on a failed Ubuntu Server 24.04 installation on AM5 hardware, and why Debian 13 became the chosen OS for this project.*

---

## Context

The target platform for this AI server was an AMD Ryzen 9 7900 on an ASUS B650E MAX Gaming WiFi motherboard — a current-generation AM5 consumer configuration. Ubuntu Server 24.04 LTS was the initial choice of operating system: a long-term support release, familiar tooling, broad community support, and the de-facto default for Linux server workloads.

What was expected to be a straightforward 20-minute installation turned into a multi-session debugging process spread over several evenings. This document records what was tried, what failed, and why the decision was made to move to Debian 13.

---

## The failure sequence

### 1. USB not detected in BIOS

The first obstacle appeared before the installer even loaded. A USB stick flashed with Balena Etcher was not detected in the BIOS boot menu at all. Rufus was tried next with the recommended GPT / UEFI (non-CSM) settings — same result. In both cases the USB stick itself was readable on other machines, ruling out a hardware failure.

**Fix:** switched to Ventoy. Ventoy installs a boot loader directly on the USB and allows ISO files to be dropped in as plain files. The stick was immediately detected by the ASUS boot menu.

### 2. Wrong ISO on the stick

With Ventoy working, the installer booted — but into Ubuntu Server 20.04, not 24.04. The 20.04 ISO had been left on the stick from an earlier test. Downloading and copying the correct 24.04.2 LTS ISO solved this in a few minutes.

### 3. ACPI crash during mirror check

With the correct ISO and the HWE kernel selected, the installer booted, connected to a phone hotspot over USB tethering, and proceeded through the initial configuration. During the mirror check step it crashed with an on-screen error and a long hexadecimal ACPI table dump — no readable error message, no stack trace in plain English.

A second attempt produced the same crash at the same step. This was the first indication that the problem was not simply a bad flash or a misconfiguration, but something deeper about how the installer interacted with the hardware.

### 4. Offline installation hangs at "connecting..."

To bypass the suspected network-phase crash, the next attempt was an offline installation — no hotspot, no network configured at all. The installer was expected to skip the mirror check entirely and proceed to disk partitioning.

Instead it hung at a `connecting…` screen for over an hour without advancing. Not a fast-looking hang — the animated spinner continued, the system was responsive, but the installer simply never moved to the next stage.

### 5. GRUB boot parameters — `cloud-init=disabled`

A known workaround for similar Ubuntu 24.04 installer hangs is to disable cloud-init via GRUB boot parameters. The `linux` line in the GRUB menu was edited to append `cloud-init=disabled` before booting.

Result: the installer still hung at the same stage.

### 6. GRUB boot parameters — `network-config=disabled`

The next escalation was to add a second parameter, `network-config=disabled`, on top of the first. This targets the installer's network configuration phase directly rather than relying on cloud-init to skip itself.

Result: the installer still hung.

At this point the combined boot parameter line read approximately:

```
linux /casper/vmlinuz ... quiet splash cloud-init=disabled network-config=disabled ---
```

And the installer still would not progress past `connecting…`.

---

## Root cause

The Ubuntu 24.04 Server installer uses cloud-init for early-boot configuration. On this specific hardware — an AM5 consumer platform with a MediaTek WiFi chipset, installed via USB from a phone hotspot — cloud-init waits for network initialisation to complete before handing control back to the installer, even when the user has explicitly selected an offline install and even when GRUB parameters instruct it not to.

On server-class hardware with a wired NIC and no exotic peripherals, cloud-init typically resolves quickly and this behaviour is invisible. On AM5 consumer boards with only wireless connectivity available during install, the initialisation does not complete, and the installer waits indefinitely.

This is not a single bug that can be patched in the ISO. It is a behavioural mismatch between Ubuntu's server installation assumptions and the hardware profile of current consumer desktop platforms.

---

## Why Debian 13 was chosen

Rather than continuing to debug Ubuntu's installer, the decision was made to evaluate Debian 13 (Trixie) as an alternative. The reasoning:

- **Same Linux kernel family.** Anything that would run on Ubuntu Server 24.04 runs on Debian 13. The AI stack (Ollama, Docker, NVIDIA drivers, CUDA) is identical on both.
- **No cloud-init dependency in the default installer path.** Debian's installer does not wait for cloud-init to complete before advancing.
- **Simpler installer.** Fewer moving parts means fewer opportunities for the installer to deadlock on hardware edge cases.
- **Longer release cycle and more conservative package selection.** Appropriate for a server that should stay running without frequent maintenance churn.

Debian 13 installed cleanly from USB in under 20 minutes on the same hardware, with the same USB stick, on the same phone hotspot connection that had defeated Ubuntu. The only manual step required after installation was enabling the `non-free` and `non-free-firmware` repositories to install the NVIDIA proprietary driver — a one-line edit to `/etc/apt/sources.list`.

---

## Lessons

**Hardware compatibility matrices for Linux server distributions are not uniform.** Ubuntu Server is optimised for cloud and data-center hardware profiles; Debian's installer is more tolerant of unusual desktop-class configurations. Neither is objectively better — they target different deployment realities.

**Boot-time log capture on consumer boards is limited.** The ACPI hex dump that triggered the initial investigation was effectively unreadable without serial console capture or external logging — hardware most consumer builders do not have. This places a practical ceiling on how deeply a single engineer can debug installer-level failures on this class of machine.

**Switching tools is a valid engineering decision, not a failure.** Persisting with Ubuntu after four documented failure modes would have cost further time without clear evidence of an imminent solution. Debian was tested and verified in a single evening. The time cost of the switch was significantly lower than the expected cost of continued debugging.

---

## Outcome

The server has now been running on Debian 13 for several weeks with no OS-level issues. The AI stack (Ollama, Docker, Open WebUI, n8n) was installed and operational within a single session after the Debian base system was in place. The NVIDIA driver, CUDA toolkit, and container GPU runtime all work as expected with the RTX 3080.

For anyone building an AI home-lab or a similar self-hosted stack on AM5 consumer hardware in 2026, Debian 13 is recommended as the starting point. The Ubuntu 24.04 installer is likely to be fixed eventually; until it is, there is no practical reason to fight it when a working alternative exists.

---

*Last updated: April 2026*
