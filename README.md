# 💾 CactBridge

<p align="center">
  <img src="https://img.shields.io/badge/license-GPLv3-blue.svg?style=for-the-badge" alt="License: GPLv3">
  <img src="https://img.shields.io/badge/role-ISO%20packager-purple.svg?style=for-the-badge" alt="Role: ISO packager">
  <img src="https://img.shields.io/badge/output-cact--gui.iso%20%2F%20cact--non--gui.iso-green.svg?style=for-the-badge" alt="Output: cact-gui.iso / cact-non-gui.iso">
  <img src="https://img.shields.io/badge/boot-Multiboot2-orange.svg?style=for-the-badge" alt="Boot: Multiboot2">
  <img src="https://img.shields.io/badge/version-3.0.0-yellow.svg?style=for-the-badge" alt="3.0.0">
</p>

<p align="center">
  ISO packager: copies a prebuilt <strong><code>kernel.bin</code></strong> and an optional <strong>Multiboot2 <code>module2</code></strong> (e.g. <strong><code>localbinrepo.img</code></strong>) into a staging tree and runs <strong><code>grub-mkrescue</code></strong>.<br>
  <strong>3.0.0:</strong> rewritten from Makefile to <strong><code>build.py</code></strong>. Supports <code>--gui-iso</code>, <code>--non-gui-iso</code>, <code>--no-deps</code> flags. Auto-detects sibling repos.
</p>

---

## 🔗 Ecosystem

| Piece | Role |
| --- | --- |
| **[CactOS-x86_64](https://github.com/QwaYer/CactOS-x86_64)** | Invokes **`build.py`** with `--non-gui-iso` or `--gui-iso` |
| **[CactKernel-x86_64](https://github.com/QwaYer/CactKernel-x86_64)** | Produces **`build/kernel.bin`** |
| **LocalRepoCactOS** | Produces **`localbinrepo.img`** and **`firmrepo.tar.zst`** |
| **[Network-for-Cact-x86_64](https://github.com/QwaYer/Network-for-Cact-x86_64)** | Produces **`netdrvrepo.tar.zst`** |

The ISO carries up to **three** Multiboot2 modules: **`localbinrepo`**
(the boot image), **`firmrepo`** (the firmware mirror, served as `/dev/firmrepo`)
and **`netdrvrepo`** (the signed network driver modules, served as
`/dev/netdrvrepo`). The last two are optional — `build.py` drops each module
line when its archive is absent, so the ISO still boots.

---

## 🔨 Building

**Recommended — full workspace**

One command installs missing dependencies and builds kernel + localbinrepo.img + ISO:

```sh
python3 build.py --non-gui-iso            # check deps, install missing, build non-GUI ISO
python3 build.py --gui-iso                # same, GUI ISO
python3 build.py --deps                   # install/detect dependencies only (packages + rustup nightly)
python3 build.py --non-gui-iso --no-deps  # skip dependency builds, repack only
python3 build.py --non-gui-iso --run      # build, then launch QEMU
python3 build.py --clean                  # clean CactBridge artefacts
```

Required packages (installed automatically via the system package manager): `base-devel`, `binutils`,
`clang`, `gcc`, `git`, `grub`, `meson`, `ninja`, `nasm`, `python`, `rustup`, `xorriso`, `mtools`.
`rustup` is then configured to **nightly** with the `rust-src` component, which the
kernel's Rust subsystems (`-Z build-std`) require. Building needs network access
for the ACPICA clone and the `crates.io` dependencies (`smoltcp`, `webpki-roots`).

**Manual path — the individual component build chain**

```sh
meson setup CactLibc-x86_64/build-meson --cross-file CactLibc-x86_64/cross/x86_64-cact-clang.ini
ninja -C CactLibc-x86_64/build-meson              # clibc.so / ld.so / start.o
ninja -C Cactsole-x86_64/build-meson              # shell
ninja -C Cgoct-x86_64/build-meson                 # init / supervisor
ninja -C CactUserBins-x86_64/build-meson stage    # fills LocalRepo lib/bin, lib/sbin
ninja -C LocalRepoCactOS-x86_64/build-meson stage # packs localbinrepo.img (modules + userland)
ninja -C Network-for-Cact-x86_64/build-meson netdrvrepo # netdrvrepo.tar.zst (network modules)
ninja -C CactKernel-x86_64/build-meson            # build-meson/kernel.bin
python3 build.py --non-gui-iso                    # assemble the ISO via grub-mkrescue
```

**Standalone `build.py`**

```sh
python3 build.py --help                  # show available flags
python3 build.py --non-gui-iso           # non-GUI ISO (auto-detects kernel.bin + localbinrepo.img)
python3 build.py --gui-iso               # GUI ISO
python3 build.py --non-gui-iso --no-deps # skip rebuild, repack only
```

`build.py` resolves sibling repositories by their current `-x86_64` names
(`LocalRepoCactOS-x86_64`, `CactLibc-x86_64`, `*-for-Cact-x86_64`) with the
legacy names as fallbacks, auto-**clones** any missing repo from
`https://github.com/QwaYer/<repo>`, and installs the toolchain (system packages
via pacman/apt/dnf + rust `nightly` + `rust-src`) when absent — so it works on a
fresh machine. Provision only the toolchain and clones with `--deps`.

Overrides via `python3 build.py --local-config FILE` (see `config/local.mk.example`).

---

## 📂 Layout

```
CactBridge/
├── build.py               # ISO builder — orchestrates meson/ninja builds in siblings + grub-mkrescue
├── drivers.list           # canonical out-of-tree module list (read by build.py and CactOS meson)
├── config/
│   └── local.mk.example   # example local overrides
├── grub/
│   ├── menu-with-mb2.cfg
│   ├── menu-kernel-only.cfg
│   └── menu-ramroot.cfg
├── makecore/              # workspace-root wrappers (Makefile, build.sh, README)
└── build/                 # staging + cact-gui.iso / cact-non-gui.iso (generated)
```
