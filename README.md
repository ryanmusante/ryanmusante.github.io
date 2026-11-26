# Ryan Musante · Systems Engineering

<a id="systems-field-notes"></a>**Reproducible Linux workstation engineering:** one CachyOS host deployed by script, audited by an independent verifier, observed live, and diagnosed from its own logs, plus the firmware, network, input, and device references around it.

[GitHub profile](https://github.com/ryanmusante) · [Project site](https://ryanmusante.github.io/) · [All repositories](https://github.com/ryanmusante?tab=repositories)

The reference host is a Beelink GTR9 Pro (AMD Ryzen AI Max+ 395, Radeon 8060S graphics) running CachyOS. Around the four repositories that manage it sit installation media, firmware and tuning references, an isolated KVM guest, local LLM inference, the home router, keyboards and Bluetooth audio, RetroArch on Apple TV 4K, a game guide, a Chromebook bootstrap, and a car head-unit firmware guide.

**On this page:** [Start here](#start-here) · [CachyOS on the GTR9 Pro](#cachyos-on-the-gtr9-pro) · [Builds and references](#builds-and-reference-guides) · [Network](#network) · [Input and audio](#input-and-audio) · [Games and devices](#games-and-other-devices) · [Conventions](#engineering-conventions)

---

## Start here

| Goal | Start with |
| --- | --- |
| Reproduce the workstation profile | [ry-install], then [ry-verify] |
| Provision a new machine from install media | [cachyos-gtr9-iso] |
| Look up a GTR9 Pro firmware setting | [gtr9pro-bios-reference] |
| Turn post-boot logs into a diagnosis | [gtr9-postboot-log-analysis] |
| Audit an RT-BE92U router over SSH | [rt-be92u-ssh-audit] |
| Plan whole-home Wi-Fi coverage | [router-WiFi-expansion] |
| Use Keychron Launcher on Linux | [keychron-udev] |
| Keep a keyboard from acting as a game controller | [joystick-ignore-check] |
| Set up RetroArch on Apple TV 4K | [retroarch-appletv4k], then [retroarch-configs] |
| Play Civilization V with Vox Populi on Linux | [vox-populi-cachyos] |

## CachyOS on the GTR9 Pro

Four repositories manage one CachyOS workstation as a closed loop: deploy a profile, verify it after a reboot, watch it live, and diagnose problems from the logs. The installer and the verifier release as a pair under one version number. The scripts are written for this machine — ry-install exits on any CPU other than a Ryzen AI Max unless overridden — and each README lists its own requirements.

- [ry-install] — **Deploy.** A fish script that applies the profile in one unattended, idempotent run: it renders the managed configuration files, installs and removes packages, masks and enables systemd units, and rewrites the fstab.
- [ry-verify] — **Verify.** A separate, read-only fish script that regenerates every managed file in memory and compares it with the installed bytes, then checks the live kernel command line, modules, sysctls, units, fstab, and session; `--verify` reports every check and `--check` probes silently for drift.
- [ry-dashboard] — **Observe.** A read-only Bash terminal monitor whose six panels (CPU, GPU, network, storage, systemd, and thermal) color each live value against the value ry-install deploys. It needs no sudo; ry-verify remains the audit.
- [gtr9-postboot-log-analysis] — **Diagnose.** A Python report builder that turns a `cachyos-bugreport.sh` log and a ry-verify log into a PDF that leads with the findings, their evidence, and the fish commands they call for, plus a JSON summary.

## Builds and reference guides

- [cachyos-gtr9-iso] — **Provision a fresh machine.** A CachyOS ISO build kit whose Calamares installer applies the profile during setup and leaves ry-install and ry-verify on the target for maintenance.
- [gtr9pro-bios-reference] — **Find a firmware setting.** A PDF catalog of every BIOS Setup option on the GTR9 Pro v2.2 board, decoded from the firmware image and listed menu by menu, with each setting's type, allowed values, and compiled default.
- [cachyos-tuning-audit] — **Weigh a tuning change.** A research brief, pinned to a named release of the installer pair, that orders proposed changes by risk from observation-only checks to protected decisions and ranks gated candidates for gaming performance.
- [cachyos-kvm-configuration] — **Review software in containment.** A risk-ordered procedure for building, isolating, and recovering a CachyOS KVM guest on the same host; every change states its rollback before it runs.
- [strix-halo-guide] — **Run models locally.** A fork of [hogeheer499-commits/strix-halo-guide](https://github.com/hogeheer499-commits/strix-halo-guide) on local LLM inference with Radeon 8060S graphics: setup, inference backends, and benchmarks. The upstream repository is the maintained source.

## Network

Both projects are written for an ASUS RT-BE92U on stock Asuswrt.

- [rt-be92u-ssh-audit] — **Audit the router.** A printable PDF runbook of graded SSH checks, from backdoor indicators and WAN exposure to firmware currency, DNS over TLS, and Samba and FTP guest access, each paired with a pass rule and a fix.
- [router-WiFi-expansion] — **Extend Wi-Fi coverage.** A printable PDF decision report that scores mesh and router options against stated requirements, works out the backhaul, and recommends one deployment.

## Input and audio

- [keychron-udev] — **Use Keychron Launcher on Linux.** A fish script that installs a udev rule tagging Keychron and Lemokey boards and their DFU bootloaders `uaccess`, so the logged-in user can configure and flash them from the browser without group or permission changes.
- [joystick-ignore-check] — **Hide a keyboard from games.** A read-only Python check that finds keyboards and mice Linux reports as joysticks, checks the two Steam settings that hide them from a Proton game, prints the launch option, and proves the result from Proton's own log.
- [reconnect-soundbar] — **Reconnect a soundbar.** A fish script and systemd user unit that reconnects a trusted Yamaha SR-C20A over Bluetooth A2DP when the user session starts.

## Games and other devices

### RetroArch on Apple TV 4K

Two companion repositories for the Apple TV 4K (3rd generation), released in lockstep under one version number.

- [retroarch-appletv4k] — **Set up RetroArch.** The setup guide and a baseline `retroarch.cfg`: installation, file transfers, systems and BIOS, controllers, configuration, shaders, and WebDAV backups of the storage tvOS can purge.
- [retroarch-configs] — **Tune each core.** Overrides (`.cfg`) and core options (`.opt`) for individual libretro cores, from NES and Genesis to arcade and Nintendo 64, with per-game override and overclocking notes.

### Platform guides

- [vox-populi-cachyos] — **Play Civilization V with Vox Populi.** A guide to running the mod through Steam and Proton on CachyOS without a terminal, from installation and first launch through settings, other mods, multiplayer, updates, and removal, with notes for other distributions; the PDF carries the same text.
- [ry-crostini] — **Set up a Chromebook.** A Bash bootstrap for the Lenovo IdeaPad Duet 5 that turns a fresh Crostini Debian container into a configured desktop, gaming tools included, in one unattended run; every step is checkpointed, so a re-run resumes where it stopped.
- [kenwood-ddx57-firmware-update-guide] — **Update a car head unit.** A PDF guide to checking and updating the APP and MCU firmware of a Kenwood DDX57 from a USB drive, with drive preparation on Linux, macOS, or Windows.

## Engineering conventions

- **Independent verification.** Deployment and audit are separate programs: the verifier regenerates the expected files itself and compares bytes instead of trusting the installer's report.
- **Least privilege.** User-run scripts start as the normal user and call sudo only for the steps that need root; the dashboard and the joystick check never need it.
- **Documented behavior.** Every README explains what the project covers and how to use it, and every command-line tool documents its requirements and options.
- **Versioned releases.** Every original project carries a version number and a changelog; the installer pair and the RetroArch pair release in lockstep, and derived kits and briefs name the release they target.
- **Verifiable documents.** The BIOS catalog, router runbook, Wi-Fi report, and Kenwood guide publish SHA-256 checksums for their PDFs, and the post-boot report is byte-identical when rebuilt from the same inputs, fonts, and library versions.
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
