# Ryan Musante · Systems Engineering

<a id="systems-field-notes"></a>**Reproducible Linux workstation engineering:** one CachyOS host deployed by script, audited by an independent verifier, observed live, and diagnosed from its own logs, plus the firmware, network, input, and device references around it.

[GitHub profile](https://github.com/ryanmusante) · [Project site](https://ryanmusante.github.io/) · [All repositories](https://github.com/ryanmusante?tab=repositories)

The reference host is a Beelink GTR9 Pro: an AMD Ryzen AI Max+ 395 with Radeon 8060S graphics (`gfx1151`), running CachyOS. A fish installer deploys its profile, a separate verifier audits the result after reboot, a terminal dashboard compares live values with the profile, and a report builder turns post-boot logs into a PDF diagnosis. The surrounding repositories cover installation media, a decoded firmware catalog, a tuning audit, an isolated KVM guest, local LLM inference, an Asuswrt router, input devices, RetroArch on Apple TV 4K, and guides for a game, a Chromebook, and a car head unit.

**On this page:** [Start here](#start-here) · [CachyOS on the GTR9 Pro](#cachyos-on-the-gtr9-pro) · [Builds and references](#builds-and-reference-guides) · [Network](#network) · [Input and audio](#input-and-audio) · [Games and devices](#games-and-other-devices) · [Conventions](#engineering-conventions) · [Release index](#release-index)

---

## Start here

| Goal | Start with |
| --- | --- |
| Reproduce the workstation profile | [ry-install], then [ry-verify] after a reboot |
| Provision a new machine from install media | [cachyos-gtr9-iso] |
| Look up a firmware setting and its default | [gtr9pro-bios-reference] |
| Turn post-boot logs into a diagnosis | [gtr9-postboot-log-analysis] |
| Audit a stock Asuswrt router over SSH | [rt-be92u-ssh-audit] |
| Extend Wi-Fi coverage around an RT-BE92U | [router-WiFi-expansion] |
| Flash a Keychron board from Linux | [keychron-udev] |
| Keep a keyboard from acting as a game controller | [joystick-ignore-check] |
| Set up RetroArch on Apple TV 4K | [retroarch-appletv4k] |
| Play Vox Populi through Proton | [vox-populi-cachyos] |

## CachyOS on the GTR9 Pro

ry-install and ry-verify target CachyOS with systemd-boot (BLS entries), the EFI system partition at `/boot` without XBOOTLDR, fish 3.6 or newer, and a CPU matching `Ryzen AI Max`. The pair embeds the same values and shares one version; ry-dashboard names the release it tracks, and the report builder reads ry-verify's logs. Reboot between deploying and auditing so the verifier inspects the kernel and services that the new configuration started.

### Deploy, verify, observe, diagnose

| Project | What it does |
| --- | --- |
| [ry-install] | **Deploy.** One unattended, idempotent fish run renders 18 managed files, installs and removes pacman packages, masks and enables systemd units, and rewrites the fstab, then writes a JSONL log of the run. |
| [ry-verify] | **Verify.** Regenerates all 18 managed files in memory and compares them byte for byte with the installed copies, then reads the live kernel command line, modules, sysctls, units, mounts, and session. `--verify` reports every check; `--check` is a silent drift probe that exits `10` on drift. |
| [ry-dashboard] | **Observe.** A read-only Bash TUI whose six panels (CPU, GPU, network, storage, systemd, thermal) color each live value against the value ry-install deploys. It needs no `sudo` and writes nothing outside its own log. |
| [gtr9-postboot-log-analysis] | **Diagnose.** `build_report.py` combines a `cachyos-bugreport.log` with a ry-verify JSONL log into a PDF that leads with findings, evidence, and fish commands, then covers system health, the boot, dmesg, the journal, and the verifier's results. |

### Quick start

> **Before the first run,** read the [requirements](https://github.com/ryanmusante/ry-install#requirements) and [package lists](https://github.com/ryanmusante/ry-install#packages) of ry-install. The unattended run removes packages, and on a CPU that does not match `Ryzen AI Max` it stops with exit `3`.

```fish
# Clone the pair into your home directory
cd ~
git clone https://github.com/ryanmusante/ry-install.git
git clone https://github.com/ryanmusante/ry-verify.git
chmod +x ry-install/ry-install.fish ry-verify/ry-verify.fish

# Deploy as your normal user; never run the script itself with sudo
cd ~/ry-install
sudo -v
./ry-install.fish
systemctl reboot
```

```fish
# After the reboot: audit the running system
cd ~/ry-verify
sudo -v
./ry-verify.fish --verify

# Optional: watch live values against the profile, no sudo needed
cd ~
git clone https://github.com/ryanmusante/ry-dashboard.git
install -Dm755 ry-dashboard/ry-dashboard.bash ~/.local/bin/ry-dashboard
ry-dashboard
```

`sudo -v` caches the credential for the privileged steps, and ry-install re-arms it every 30 seconds during the run. Both scripts print results to stderr and write a JSONL log under `~/ry-install/logs/`. The report builder's [quick start](https://github.com/ryanmusante/gtr9-postboot-log-analysis#quick-start) lists its Python dependencies.

### Field notes

**CPU clocks changed after a kernel update.** Run `./ry-verify.fish --verify` and read the Runtime › Hardware state group: scaling driver, governor, EPP, `amd_pstate` status, prefcore, boost, and the GPU DPM level ([checks](https://github.com/ryanmusante/ry-verify#checks)). Repeat the workload under `ry-dashboard --panel 6` to compare `Tctl` with TjMax and `PPT` draw with the 85 W BIOS ceiling ([panels](https://github.com/ryanmusante/ry-dashboard#panels)).

**A managed file drifted.** `./ry-verify.fish --check` exits `0` when the host matches the profile and `10` on drift; a fresh install reads as drift until its first reboot. Repair a single file through the installer, for example `./ry-install.fish --install-file /etc/nftables.conf`, rather than by hand. A boot file re-runs the boot cascade, so do not reboot until that run succeeds.

**A report leaves the machine.** The PDF embeds both input logs unmasked, so build the copy you share with `--no-attach`. `--brief` keeps the cover and the first three sections, about six pages, for paper; `--json` and `--compare` track findings from one run to the next.

### Reference platform

| Component | Specification |
| --- | --- |
| System | Beelink GTR9 Pro mini-PC |
| Processor | AMD Ryzen AI Max+ 395 "Strix Halo": Zen 5, 16 cores, 32 threads |
| Graphics | Radeon 8060S: 40 RDNA 3.5 compute units, `gfx1151` |
| NPU | AMD XDNA 2 |
| Memory | 128 GB LPDDR5X-8000, unified; 32 GiB BIOS carveout |
| Storage | Dual M.2 NVMe |
| Network | Dual 10 GbE (Realtek RTL8127), Wi-Fi 7 (MediaTek MT7925), Bluetooth 5.4 |
| Power | Flat 85 W BIOS ceiling (SPL = fPPT = sPPT); stock firmware boosts to 140 W |
| Thermal | TjMax 90 °C, the firmware default |
| Display | BenQ PD3420Q: 3440x1440 at a fixed 60 Hz, no adaptive sync |
| Software | CachyOS (rolling), systemd-boot, fish 3.6 or newer |

The 85 W ceiling and the fixed 60 Hz panel frame every performance candidate in the [tuning audit][cachyos-tuning-audit].

## Builds and reference guides

| Project | When to use it |
| --- | --- |
| [CachyOS ISO build kit][cachyos-gtr9-iso] | **Provision a fresh machine.** `setup.fish` prepares the build tree for a CachyOS live ISO whose Calamares installer copies the profile payload into the target and applies it from the target chroot, leaving ry-install and ry-verify installed for maintenance. Every list is extracted from `ry-install.fish` at build time, and `setup.fish` refuses a pair at any other version unless `--force` is passed. |
| [GTR9 Pro UEFI setup reference][gtr9pro-bios-reference] | **Find a firmware setting.** A 46-page PDF catalogs 1,010 settings in 7 form-sets, from Aptio Setup and AMD CBS to AMD Overclocking and PMF, decoded from BIOS image `GTRPRPI1001C` for board revision v2.2. Check the revision first: v1.0 boards, with the Intel E610 NIC, run a different BIOS series. |
| [CachyOS tuning audit][cachyos-tuning-audit] | **Weigh a tuning change.** A deep-research brief pinned to one release of the pair ranks every candidate by blast radius, from observation only (T0) through the kernel command line and boot chain to closed decisions (T5), and gives each gaming and performance candidate its gates and measurement method. |
| [Isolated CachyOS KVM guest][cachyos-kvm-configuration] | **Review software in containment.** A risk-ordered procedure, phases P0 to P9, builds a virtio-only CachyOS guest on the same host: prerequisites, VM definition, guest integration, network containment, autostart, a snapshot baseline, and recovery. Every mutating phase states its rollback before it runs. It needs AMD-V, 70 GiB free under `/var/lib`, and 32 GiB of RAM for the guest. |
| [Local LLMs on Strix Halo][strix-halo-guide] | **Run models locally.** A fork of [hogeheer499-commits/strix-halo-guide](https://github.com/hogeheer499-commits/strix-halo-guide) covering BIOS and kernel setup, Ollama, `llama.cpp` on Vulkan/RADV and ROCm/HIP, benchmarks, and raw logs. The fork was last synced on 2026-07-29; the upstream repository carries the current edition. |

## Network

Both projects target an ASUS RT-BE92U on stock Asuswrt.

| Project | Scope |
| --- | --- |
| [Router security and health audit][rt-be92u-ssh-audit] | **A printable SSH runbook** for Asuswrt 3.0.0.6.102: 20 graded checks, from AyySSHush backdoor indicators and WAN exposure to firmware currency, DNS-over-TLS, Samba and FTP guest access, and USB SSD TRIM. Each check pairs a command with a pass rule and a fix, and four INFO items record a baseline. No item needs `nvram unset`, `nvram erase`, or a raw flash write. |
| [Whole-home Wi-Fi expansion][router-WiFi-expansion] | **A printable decision report** that scores 20 ways to extend coverage, from a matched AiMesh node to third-party mesh, UniFi, and GL.iNet routers, against six requirements for a network whose mini-PC rides a 6 GHz-only SSID. Verdict: a second RT-BE92U as an AiMesh node on Ethernet backhaul, MoCA where no Ethernet run exists, and wireless MLO backhaul only where neither does. |

## Input and audio

| Symptom | Project and approach |
| --- | --- |
| Keychron Launcher stops at "HID device connected", or a flash stalls at 0 %. | [keychron-udev] installs `70-keychron.rules`, which tags the board's hidraw nodes and its DFU bootloader `uaccess` so systemd-logind grants the active seat user access. It needs no `input` or `plugdev` group and no `MODE="0666"`, and it covers Keychron (`3434`) and Lemokey (`362d`) devices. |
| A game switches to controller mode and ignores the mouse. | [joystick-ignore-check] finds keyboards and mice that Linux enumerates as joysticks, prints the per-game launch option that hides them from the game under Proton, and proves the fix from a `PROTON_LOG=+hid` trace. It runs on Python 3.11 or newer with the standard library only. |
| A paired soundbar stays disconnected after power-on. | [reconnect-soundbar] runs once when the systemd user manager starts and has BlueZ reconnect A2DP to a trusted Yamaha SR-C20A, retrying within fixed bounds. |

## Games and other devices

### RetroArch on Apple TV 4K

The two repositories release in lockstep for Apple TV 4K (3rd generation) on tvOS 27 with RetroArch 1.22.x.

| Scope | Repository and contents |
| --- | --- |
| Global settings | The [Apple TV 4K setup guide][retroarch-appletv4k] ships a 74-key baseline `retroarch.cfg` and covers controllers, Manual Scan library imports, web and WebDAV transfers, and shaders. tvOS guarantees an app only 500 KB of persistent storage, so the guide backs up the ROMs, saves, and overrides that tvOS can purge. |
| Per-core settings | The [per-core configuration][retroarch-configs] pairs a `.cfg` override with an `.opt` core-options file for seven cores: Beetle PCE Fast, FinalBurn Neo, Genesis Plus GX, Mesen, mGBA, Mupen64Plus-Next, and Snes9x. Mupen64Plus-Next, for example, runs the Angrylion RDP plugin on the cached interpreter. |

### Platform guides

| Project | Scope |
| --- | --- |
| [Civilization V and Vox Populi][vox-populi-cachyos] | **A Steam and Proton guide** for *Civilization V: Brave New World* with Vox Populi, whose Windows game-core DLL rules out the native Linux build. Eight steps take CachyOS from forcing Proton to the first launch without a terminal, with a check after every step that writes to disk; later sections cover mods, multiplayer, updates, and removal. A PDF carries the same text. |
| [Crostini on the Lenovo Duet 5][ry-crostini] | **A Bash bootstrap** that turns a fresh Debian container on the IdeaPad Duet 5 Chromebook (Snapdragon 7c Gen 2) into a configured desktop in 13 checkpointed steps. Bookworm is the primary target and Trixie the secondary; an interrupted run resumes at the next unfinished step. |
| [Kenwood DDX57 firmware][kenwood-ddx57-firmware-update-guide] | **A 7-page PDF guide** for the DDX5707S and DDX57S head units: confirm the installed APP and MCU versions, prepare a FAT32 USB drive on Linux, macOS, or Windows, run the two-phase update from Kenwood's package, and verify the result. |

## Engineering conventions

- **Independent audit.** ry-verify regenerates every managed file in memory and compares bytes with the installed copy, so an audit never relies on the installer's own report.
- **Least privilege.** ry-install, ry-verify, and keychron-udev refuse to run as root and call `sudo` only for the steps that write system state; the dashboard and the joystick check run without it.
- **Documented interfaces.** The eight standalone tools document their exit codes, and the four workstation tools also document environment overrides and structured logs: JSONL, CSV, or a JSON summary.
- **Versioned releases.** Each of the 18 project repositories keeps a changelog. Derived repositories follow their source: the ISO kit mirrors the pair's version, the tuning audit pins to it, the dashboard names the release it tracks, and the RetroArch pair releases in lockstep.
- **Verifiable documents.** The firmware catalog, the router audit, the Wi-Fi study, and the Kenwood guide publish SHA-256 checksums for their PDFs, and the report builder produces a byte-identical PDF from identical inputs, fonts, and libraries.
- **Resumable automation.** ry-install converges in one idempotent run and records any restart it still owes; ry-crostini checkpoints each of its 13 steps.

## Release index

Versions as each project's README or changelog states them on 2026-11-25. A dash marks a repository without a license file.

| Repository | Version | License |
| --- | --- | --- |
| [ry-install] | 7.302.0 | MIT |
| [ry-verify] | 7.302.0 | MIT |
| [ry-dashboard] | 1.7.0 | MIT |
| [gtr9-postboot-log-analysis] | 7.32.0 | MIT |
| [cachyos-gtr9-iso] | 7.195.0 r1 | MIT |
| [gtr9pro-bios-reference] | 1.7.12 | — |
| [cachyos-tuning-audit] | 7.302.0 | — |
| [cachyos-kvm-configuration] | 6.5.0 | — |
| [strix-halo-guide] | fork | MIT |
| [rt-be92u-ssh-audit] | 1.8.0 | MIT |
| [router-WiFi-expansion] | 1.11.0 | MIT |
| [keychron-udev] | 1.6.2 | MIT |
| [joystick-ignore-check] | 1.9.0 | MIT |
| [reconnect-soundbar] | 1.5.8 | — |
| [retroarch-appletv4k] | 5.8 | MIT |
| [retroarch-configs] | 5.8 | MIT |
| [vox-populi-cachyos] | 11.8.1 | MIT |
| [ry-crostini] | 8.2.0 | MIT |
| [kenwood-ddx57-firmware-update-guide] | 1.0.3 | CC BY 4.0 |

---

[Browse all repositories](https://github.com/ryanmusante?tab=repositories) · [Source for this index](https://github.com/ryanmusante/ryanmusante.github.io) · Checked against all 20 repositories on 2026-11-25

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
