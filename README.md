# Ryan Musante · Systems Engineering

<a id="systems-field-notes"></a>**Reproducible Linux workstation engineering:** one CachyOS host deployed by script, audited by an independent verifier, observed live, and diagnosed from its own logs, plus the firmware, network, input, and device references around it.

[GitHub profile](https://github.com/ryanmusante) · [Project site](https://ryanmusante.github.io/) · [All repositories](https://github.com/ryanmusante?tab=repositories)

The reference host is a Beelink GTR9 Pro with an AMD Ryzen AI Max+ 395, running CachyOS. A fish installer deploys its profile, a separate verifier audits the result, a terminal dashboard compares live values with the profile, and a report builder turns post-boot logs into a PDF diagnosis. The surrounding repositories cover installation media, firmware and tuning references, an isolated KVM guest, local LLM inference, a home router, input devices, RetroArch on Apple TV 4K, and guides for a game, a Chromebook, and a car head unit.

**On this page:** [Start here](#start-here) · [CachyOS on the GTR9 Pro](#cachyos-on-the-gtr9-pro) · [Builds and references](#builds-and-reference-guides) · [Network](#network) · [Input and audio](#input-and-audio) · [Games and devices](#games-and-other-devices) · [Conventions](#engineering-conventions)

---

## Start here

| Goal | Start with |
| --- | --- |
| Reproduce the workstation profile | [ry-install], then [ry-verify] |
| Provision a new machine from install media | [cachyos-gtr9-iso] |
| Look up a firmware setting | [gtr9pro-bios-reference] |
| Turn post-boot logs into a diagnosis | [gtr9-postboot-log-analysis] |
| Audit an Asuswrt router over SSH | [rt-be92u-ssh-audit] |
| Plan whole-home Wi-Fi coverage | [router-WiFi-expansion] |
| Flash a Keychron board from Linux | [keychron-udev] |
| Keep a keyboard from acting as a game controller | [joystick-ignore-check] |
| Set up RetroArch on Apple TV 4K | [retroarch-appletv4k] |
| Play Vox Populi through Proton | [vox-populi-cachyos] |

## CachyOS on the GTR9 Pro

Four repositories manage one CachyOS workstation as a closed loop: deploy a profile, verify it after a reboot, watch it live, and diagnose problems from the logs. The installer and the verifier are released as a pair, and each repository's README lists its own requirements.

| Project | What it does |
| --- | --- |
| [ry-install] | **Deploy.** A fish script that applies the workstation profile in one idempotent run: configuration files, packages, systemd units, and mount options. |
| [ry-verify] | **Verify.** A read-only audit, separate from the installer, that checks the installed files and the running system against the profile and reports any drift. |
| [ry-dashboard] | **Observe.** A read-only terminal dashboard that shows CPU, GPU, network, storage, service, and thermal state against the deployed profile. |
| [gtr9-postboot-log-analysis] | **Diagnose.** A Python tool that turns a CachyOS bug report and a verifier log into a PDF of findings, evidence, and recommended actions. |

## Builds and reference guides

| Project | When to use it |
| --- | --- |
| [CachyOS ISO build kit][cachyos-gtr9-iso] | **Provision a fresh machine.** Builds CachyOS installation media whose installer applies the workstation profile during setup and leaves the installer and verifier in place for maintenance. |
| [GTR9 Pro UEFI setup reference][gtr9pro-bios-reference] | **Find a firmware setting.** A PDF catalog of the GTR9 Pro's BIOS Setup options, decoded from the firmware image, with each setting's menu, default, and allowed values. |
| [CachyOS tuning audit][cachyos-tuning-audit] | **Weigh a tuning change.** A research brief that evaluates proposed changes against the current profile and orders them by risk, from observation-only checks to settled decisions. |
| [Isolated CachyOS KVM guest][cachyos-kvm-configuration] | **Review software in containment.** A step-by-step procedure for building, isolating, and recovering a CachyOS virtual machine on the same host, with a rollback for every change. |
| [Local LLMs on Strix Halo][strix-halo-guide] | **Run models locally.** A fork of [hogeheer499-commits/strix-halo-guide](https://github.com/hogeheer499-commits/strix-halo-guide) on local LLM inference with Radeon 8060S graphics: setup, inference backends, and benchmarks. The upstream repository is the maintained source. |

## Network

Both projects are written for an ASUS RT-BE92U on stock Asuswrt.

| Project | Scope |
| --- | --- |
| [Router security and health audit][rt-be92u-ssh-audit] | **A printable SSH runbook** of graded checks for remote exposure, firmware status, DNS, and file sharing, each paired with a pass condition and a fix. |
| [Whole-home Wi-Fi expansion][router-WiFi-expansion] | **A printable decision report** that compares mesh systems and backhaul options for extending coverage, then recommends one deployment. |

## Input and audio

| Symptom | Project and approach |
| --- | --- |
| Keychron Launcher cannot reach the keyboard on Linux. | [keychron-udev] installs a udev rule that gives the logged-in user access to Keychron boards and their bootloader, without broad group or permission changes. |
| A game treats a keyboard as a controller. | [joystick-ignore-check] finds keyboards and mice that Linux reports as joysticks, gives the Steam settings that hide them from a Proton game, and confirms the fix from a Proton log. |
| A paired soundbar stays disconnected after power-on. | [reconnect-soundbar] reconnects a trusted Yamaha SR-C20A over Bluetooth when the user session starts. |

## Games and other devices

### RetroArch on Apple TV 4K

Two companion repositories, released together.

| Scope | Repository and contents |
| --- | --- |
| Global settings | The [Apple TV 4K setup guide][retroarch-appletv4k] provides a baseline `retroarch.cfg` and covers setup, controllers, file transfers, and backups. |
| Per-core settings | The [per-core configuration][retroarch-configs] adds overrides and core options for individual emulators. |

### Platform guides

| Project | Scope |
| --- | --- |
| [Civilization V and Vox Populi][vox-populi-cachyos] | **A setup guide** for running *Civilization V* with the Vox Populi mod through Steam and Proton on CachyOS, from installation to the first game. |
| [Crostini on the Lenovo Duet 5][ry-crostini] | **A bootstrap script** that turns a fresh Linux container on the Chromebook into a configured desktop and resumes where it left off if interrupted. |
| [Kenwood DDX57 firmware][kenwood-ddx57-firmware-update-guide] | **A PDF guide** to checking and updating the head unit's firmware from a USB drive. |

## Engineering conventions

- **Independent verification.** Deployment and audit are separate programs, and the verifier rebuilds the expected state itself instead of trusting the installer's report.
- **Least privilege.** Scripts run as the normal user and elevate only for the steps that change the system.
- **Documented behavior.** Every README explains what the project covers and how to use it; the command-line tools also document their requirements and exit codes.
- **Versioned releases.** Every project keeps a changelog, and companion repositories release together.
- **Verifiable documents.** The reference PDFs publish checksums, and generated reports are reproducible from the same inputs.
- **Safe to re-run.** The installers are idempotent or resume where they stopped.

---

[Browse all repositories](https://github.com/ryanmusante?tab=repositories) · [Source for this index](https://github.com/ryanmusante/ryanmusante.github.io)

[ry-install]: https://github.com/ryanmusante/ry-install
[ry-verify]: https://github.com/ryanmusante/ry-verify
[ry-dashboard]: https://github.com/ryanmusante/ry-dashboard
[gtr9-postboot-log-analysis]: https://github.com/ryanmusante/gtr9-postboot-log-analysis
[cachyos-gtr9-iso]: https://github.com/ryanmusante/cachyos-gtr9-iso
[gtr9pro-bios-reference]: https://github.com/ryanmusante/gtr9pro-bios-reference
[cachyos-tuning-audit]: https://github.com/ryanmusante/cachyos-tuning-audit
[cachyos-kvm-configuration]: https://github.com/ryanmusante/cachyos-kvm-configuration
[strix-halo-guide]: https://github.com/ryanmusante/strix-halo-guide
[rt-be92u-ssh-audit]: https://github.com/ryanmusante/rt-be92u-ssh-audit
[router-WiFi-expansion]: https://github.com/ryanmusante/router-WiFi-expansion
[keychron-udev]: https://github.com/ryanmusante/keychron-udev
[joystick-ignore-check]: https://github.com/ryanmusante/joystick-ignore-check
[reconnect-soundbar]: https://github.com/ryanmusante/reconnect-soundbar
[retroarch-appletv4k]: https://github.com/ryanmusante/retroarch-appletv4k
[retroarch-configs]: https://github.com/ryanmusante/retroarch-configs
[vox-populi-cachyos]: https://github.com/ryanmusante/vox-populi-cachyos
[ry-crostini]: https://github.com/ryanmusante/ry-crostini
[kenwood-ddx57-firmware-update-guide]: https://github.com/ryanmusante/kenwood-ddx57-firmware-update-guide
