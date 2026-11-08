# 🌵 CactOS — workspace root

Monolithic OS for **x86_64**: low-level in **C**, critical subsystems in **Rust**.

## 🚀 Quick Build

```sh
python3 CactBridge-x86_64/build.py --non-gui-iso         # full ISO (+ empty ext4 disk on first --run)
python3 CactBridge-x86_64/build.py --non-gui-iso --run   # build + launch QEMU
```

## 📦 Components

| Directory | Role | Standalone build |
|---|---|---|
| `CactKernel-x86_64` | Kernel | `ninja -C CactKernel-x86_64/build-meson` |
| `CactLibc-x86_64` | libc | `ninja -C CactLibc-x86_64/build-meson` |
| `Cactsole-x86_64` | Shell | `ninja -C Cactsole-x86_64/build-meson` |
| `Cgoct-x86_64` | Init (PID 1) | `ninja -C Cgoct-x86_64/build-meson` |
| `CactUserBins-x86_64` | User utilities | `ninja -C CactUserBins-x86_64/build-meson stage` |
| `LocalRepoCactOS-x86_64` | cctkfs.img packer | `ninja -C LocalRepoCactOS-x86_64/build-meson stage` |
| `CactBridge-x86` | ISO packer (GRUB) | `python3 CactBridge-x86_64/build.py --non-gui-iso` |
| `AHCI-for-Cact-x86_64` | AHCI driver | `ninja -C AHCI-for-Cact-x86_64/build-meson stage` |
| `NVMe-for-Cact-x86_64` | NVMe driver | `ninja -C NVMe-for-Cact-x86_64/build-meson stage` |
| `Virtio-net-for-Cact-x86_64` | virtio-net driver | `ninja -C Virtio-net-for-Cact-x86_64/build-meson stage` |
| `Virtio-gpu-for-Cact-x86_64` | virtio-gpu driver | `ninja -C Virtio-gpu-for-Cact-x86_64/build-meson stage` |
| `Yukon-for-Cact-x86_64` | Yukon NIC driver | `ninja -C Yukon-for-Cact-x86_64/build-meson stage` |
| `Intel-HDA-for-Cact-x86_64` | HD Audio driver | `ninja -C Intel-HDA-for-Cact-x86_64/build-meson stage` |
| `EXT4-for-Cact-x86_64` | ext4 filesystem module | `ninja -C EXT4-for-Cact-x86_64/build-meson stage` |
| `FAT32-for-Cact-x86_64` | FAT32 filesystem module | `ninja -C FAT32-for-Cact-x86_64/build-meson stage` |
| `RT2800USB-for-Cact-x86_64` | RT2800 USB Wi-Fi driver | `ninja -C RT2800USB-for-Cact-x86_64/build-meson stage` |
| `CactOS-x86_64` | Workspace integrator | `ninja -C CactOS-x86_64/build-meson stage` |

Every component is its own Meson project and **auto-detects sibling directories** — its `build-meson/` just needs one `meson setup build-meson --cross-file cross/x86_64-cact-clang.ini` (the integrator targets do that automatically on first use). Override paths with `-D` options, e.g. `meson configure build-meson -Dcactlib=/custom/path`.

## 🔧 Full build step by step

```sh
ninja -C CactOS-x86_64/build-meson stage    # libc → shell → userbins → drivers → cctkfs.img
ninja -C CactOS-x86_64/build-meson kernel   # kernel only
ninja -C CactOS-x86_64/build-meson iso      # full ISO (non-GUI)
ninja -C CactOS-x86_64/build-meson disk     # empty ext4 nvme.img
ninja -C CactOS-x86_64/build-meson drivers  # out-of-tree modules only
```

## ▶️ Run in QEMU

```sh
CACT_ISO=CactBridge-x86_64/build/cact-non-gui.iso CactKernel-x86_64/run_qemu.sh
```

Requires: `qemu-system-x86_64`, `grub-mkrescue`, `gcc` (x86_64), `nasm`, `cargo +nightly`.

## 🏗️ Build chain

```
libc → cactsole / cgoct → CactUserBins → drivers → localrepo (cctkfs.img) → kernel → CactBridge (cact-*.iso)
```

See [CactOS-x86_64/README.md](../../CactOS-x86_64/README.md) for full tech specs and architecture.
