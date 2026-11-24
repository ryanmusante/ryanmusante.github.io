# Systems field notes

**Ryan Musante** · [GitHub profile](https://github.com/ryanmusante) · [Project site](https://ryanmusante.github.io/)

I run CachyOS on a **Beelink GTR9 Pro** (Ryzen AI Max+ 395, Radeon 8060S). When I change `/etc/fstab`, I use these tools to check which mount options took effect after reboot. The guides range from Wi-Fi backhaul to RetroArch core overrides.

[Workstation](#cachyos-on-the-gtr9-pro) · [Input and audio](#input-and-audio) · [Network](#network) · [Games and devices](#games-and-other-devices)

## CachyOS on the GTR9 Pro

1. [**ry-install**](https://github.com/ryanmusante/ry-install) deploys the profile with fish, managing configuration files, pacman packages, systemd units, and `/etc/fstab`.
2. [**ry-verify**](https://github.com/ryanmusante/ry-verify) runs a read-only audit after reboot. It compares managed files byte for byte and checks live kernel parameters and mounts.
3. [**ry-dashboard**](https://github.com/ryanmusante/ry-dashboard) tracks CPU policy, GPU power state, and temperatures against the deployed profile in the terminal.

| Workstation project | Use it to |
| --- | --- |
| [CachyOS ISO build kit](https://github.com/ryanmusante/cachyos-gtr9-iso) | Build an installer that applies the bundled profile through Calamares. |
| [UEFI setup reference](https://github.com/ryanmusante/gtr9pro-bios-reference) | Look up settings and defaults decoded from the GTR9 Pro v2.2 firmware. |
| [Post-boot report builder](https://github.com/ryanmusante/gtr9-postboot-log-analysis) | Turn a CachyOS bug report and a `ry-verify` JSONL log into a diagnostic PDF. |
| [Tuning audit](https://github.com/ryanmusante/cachyos-tuning-audit) | Review proposed profile changes and the measurements that would justify them. |
| [Isolated KVM guest](https://github.com/ryanmusante/cachyos-kvm-configuration) | Configure a CachyOS VM with network isolation, snapshots, and recovery. |
| [Local LLMs on Strix Halo](https://github.com/ryanmusante/strix-halo-guide) | Run and benchmark local models with Ollama and `llama.cpp`. Fork of [hogeheer499-commits’ guide](https://github.com/hogeheer499-commits/strix-halo-guide). |

## Input and audio

| Project | Use it to |
| --- | --- |
| [Keychron Launcher access](https://github.com/ryanmusante/keychron-udev) | Grant the active user keyboard and bootloader access with a udev rule. |
| [Proton joystick check](https://github.com/ryanmusante/joystick-ignore-check) | Find keyboards and mice Linux reports as joysticks, then verify the game ignores them. |
| [Yamaha soundbar reconnect](https://github.com/ryanmusante/reconnect-soundbar) | Reconnect a paired SR-C20A over Bluetooth A2DP with a systemd user service. |

## Network

| Project | Use it to |
| --- | --- |
| [RT-BE92U security audit](https://github.com/ryanmusante/rt-be92u-ssh-audit) | Check WAN exposure, DNS, and services over SSH on stock Asuswrt. |
| [Whole-home Wi-Fi](https://github.com/ryanmusante/router-WiFi-expansion) | Compare expansion options for the ASUS RT-BE92U, including Ethernet, MoCA, and wireless backhaul. |

## Games and other devices

| Project | Use it to |
| --- | --- |
| [Vox Populi on CachyOS](https://github.com/ryanmusante/vox-populi-cachyos) | Run the Vox Populi overhaul for *Civilization V* through Steam and Proton. |
| [RetroArch on Apple TV 4K](https://github.com/ryanmusante/retroarch-appletv4k) | Set up controllers and the global `retroarch.cfg` on the third-generation model. |
| [RetroArch core settings](https://github.com/ryanmusante/retroarch-configs) | Tune individual Apple TV emulator cores with `.cfg` overrides and `.opt` files. |
| [Crostini on the Duet 5](https://github.com/ryanmusante/ry-crostini) | Provision the Debian container on a Lenovo IdeaPad Duet 5 Chromebook. |
| [Kenwood DDX57 firmware](https://github.com/ryanmusante/kenwood-ddx57-firmware-update-guide) | Check update eligibility, prepare the USB drive, and update APP and MCU firmware. |

[Browse all repositories](https://github.com/ryanmusante?tab=repositories) · [Source for this index](https://github.com/ryanmusante/ryanmusante.github.io)
