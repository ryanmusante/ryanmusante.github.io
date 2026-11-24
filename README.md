# Ryan Musante · Systems Engineering

<a id="systems-field-notes"></a>

[GitHub profile](https://github.com/ryanmusante) · [Project site](https://ryanmusante.github.io/)

The **CachyOS workstation profile** defines boot parameters, systemd services, and power policy for the Beelink GTR9 Pro. An independent verifier checks both the files on disk and the state reported by the running host. When a change needs investigation, the post-boot report brings audit findings, kernel messages, and journal entries into one diagnostic record.

The reference workstation pairs an AMD Ryzen AI Max+ 395 with Radeon 8060S graphics. Beyond the host, the portfolio includes an SSH audit for stock Asuswrt, input diagnostics for Proton, and companion RetroArch configurations for Apple TV 4K.

[Workstation tools](#cachyos-on-the-gtr9-pro) · [Builds and references](#builds-and-reference-guides) · [Input and audio](#input-and-audio) · [Network](#network) · [Games and devices](#games-and-other-devices)

## CachyOS on the GTR9 Pro

The deployment and verification tools target CachyOS installations that use systemd-boot and mount the EFI system partition at `/boot`. After applying the profile to an existing installation, reboot before auditing so the verifier can inspect the kernel and services started with that configuration.

| Project | Role and output |
| --- | --- |
| [**ry-install**](https://github.com/ryanmusante/ry-install) | **Deploy the profile.** An idempotent fish script generates managed configuration files, installs and removes pacman packages, configures systemd units, and writes persistent mount options. |
| [**ry-verify**](https://github.com/ryanmusante/ry-verify) | **Verify after reboot.** The read-only audit compares installed files byte for byte with their expected contents and checks active kernel parameters, modules, services, and mounts. Use `--verify` for the findings or `--check` for a silent drift check. |
| [**ry-dashboard**](https://github.com/ryanmusante/ry-dashboard) | **Observe the running host.** Six terminal panels cover CPU, GPU, network, storage, systemd, and thermals. The dashboard reads `/proc`, sysfs, and systemd to show how live values relate to the profile. |
| [**Post-boot report builder**](https://github.com/ryanmusante/gtr9-postboot-log-analysis) | **Investigate a finding.** A Python tool combines `cachyos-bugreport.log` and a `ry-verify` JSONL log into a PDF. Each report brings findings, supporting evidence, and follow-up checks together. |

**Tracing a change in CPU behavior.** If CPU clocks behave differently after a kernel update, look up the active scaling driver, governor, and energy-performance preference in the [verification checks](https://github.com/ryanmusante/ry-verify#checks). Run the workload again and compare CPU temperature and socket power with their configured limits in the [dashboard panels](https://github.com/ryanmusante/ry-dashboard#panels).

### Builds and reference guides

| Project | When to use it |
| --- | --- |
| [CachyOS ISO build kit](https://github.com/ryanmusante/cachyos-gtr9-iso) | To provision a fresh machine, build installation media that carries the profile into the Calamares installer. A target-side step applies the configuration and leaves the deployment and verification scripts available for maintenance. |
| [GTR9 Pro UEFI setup reference](https://github.com/ryanmusante/gtr9pro-bios-reference) | Identify a firmware setting by its setup menu, default, and allowed values. The PDF catalog was decoded from the **v2.2 board firmware** and identifies the hardware revision and firmware family it covers. |
| [CachyOS tuning audit](https://github.com/ryanmusante/cachyos-tuning-audit) | Evaluate a proposed tuning change against a specific installer and verifier revision. The research brief records the expected effect on frame times, the rationale, and the measurement that would justify adopting the change. |
| [Isolated CachyOS KVM guest](https://github.com/ryanmusante/cachyos-kvm-configuration) | Build a separate environment for reviewing software on the GTR9 Pro. The guide covers the guest definition, virtio devices, network isolation, snapshots, and recovery. |
| [Local inference on Strix Halo](https://github.com/ryanmusante/strix-halo-guide) | A fork of the [upstream guide by hogeheer499-commits](https://github.com/hogeheer499-commits/strix-halo-guide). It documents local inference on the Radeon 8060S with Ollama and `llama.cpp`, including Vulkan and ROCm configurations, benchmark results, and raw logs. |

## Input and audio

| Symptom | Project and diagnostic approach |
| --- | --- |
| Keychron Launcher stalls at device access. | [keychron-udev](https://github.com/ryanmusante/keychron-udev) uses a udev rule and systemd-logind to grant the active local user access. Launcher communicates with the keyboard over WebHID and with its DFU bootloader over WebUSB. |
| A keyboard triggers controller mode in a game. | [joystick-ignore-check](https://github.com/ryanmusante/joystick-ignore-check) identifies keyboards and mice that Linux exposes as joysticks. It checks per-game Steam settings and reads Proton’s HID trace to verify which devices the game sees. |
| A paired soundbar stays disconnected after power-on. | [reconnect-soundbar](https://github.com/ryanmusante/reconnect-soundbar) runs a fish script when the systemd user manager starts. The one-shot service asks BlueZ to establish a Bluetooth A2DP connection to the trusted Yamaha SR-C20A. |

## Network

The **ASUS RT-BE92U** running stock Asuswrt is the reference router for the security audit and coverage study.

- **[Router security and health audit](https://github.com/ryanmusante/rt-be92u-ssh-audit)** is a printable SSH runbook for checking WAN exposure, firmware status, DNS-over-TLS, and file-sharing services. Each graded check pairs a command with a pass condition and a corrective action.
- **[Whole-home Wi-Fi expansion](https://github.com/ryanmusante/router-WiFi-expansion)** compares AiMesh and third-party systems for a network with a 6 GHz client. It evaluates Ethernet, MoCA over coax, and wireless backhaul against the available cabling and link speeds.

## Games and other devices

### RetroArch on Apple TV 4K

The two RetroArch projects share a configuration baseline for the third-generation Apple TV 4K.

| Configuration scope | Repository and contents |
| --- | --- |
| Global settings | The [Apple TV 4K setup guide](https://github.com/ryanmusante/retroarch-appletv4k) supplies the baseline `retroarch.cfg` and covers controller pairing, library imports, and WebDAV transfers. Its backup guidance identifies the content tvOS can evict under storage pressure. |
| Emulator-specific settings | The [per-core configuration](https://github.com/ryanmusante/retroarch-configs) uses `.cfg` files to override RetroArch settings for an individual emulator and `.opt` files for the core’s own options. The Mupen64Plus-Next options, for example, select the Nintendo 64 rendering plugin. |

### Platform guides

| Project | Scope |
| --- | --- |
| [Civilization V and Vox Populi](https://github.com/ryanmusante/vox-populi-cachyos) | Vox Populi replaces a Windows game-core DLL. The guide follows the Windows edition of *Civilization V: Brave New World* through Steam and Proton on CachyOS, from dependency setup and mod placement to the first launch. |
| [Crostini on the Lenovo Duet 5](https://github.com/ryanmusante/ry-crostini) | A post-install bootstrap provisions the Chromebook’s Debian container. It configures the desktop in stages and records a checkpoint after each one so an interrupted run can resume with the next unfinished step. |
| [Kenwood DDX57 firmware](https://github.com/ryanmusante/kenwood-ddx57-firmware-update-guide) | A guide for the DDX5707S and DDX57S head units. It covers checking the installed APP and MCU versions, determining which updates apply, preparing USB media, and verifying the versions after installation. |

---

[Browse all repositories](https://github.com/ryanmusante?tab=repositories) · [Source for this index](https://github.com/ryanmusante/ryanmusante.github.io)
