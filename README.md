# Linux Kernel 7.x for Orange Pi 3B (RK3566)

A custom mainline-based kernel (7.1.9) for the Orange Pi 3B. Works with generic Linux rootfs — not tied to any single distribution.

> Warning: back up your data before flashing a new kernel, and make sure you have a recovery path (UART cable, or another Linux PC that can read your SD card / eMMC).

## Features

- SCX (sched_ext) — eBPF-based extensible CPU scheduler
- f2fs / erofs optimizations
- io_uring (`CONFIG_IO_URING=y`)
- eBPF JIT, enabled by default

## Compatibility

Only the Orange Pi 3B hardware revision **v2.1** has been verified. Revision v1.1 is not supported.

| Rootfs | Status |
| ------ | ------ |
| Armbian (Debian, minimal) | Tested |
| Armbian (Ubuntu, any desktop) | Not tested, expected to work |
| Other distributions | Unverified — reports are welcome |

## Known Issues

- Docker does not start in NAT/bridge mode because the legacy iptables NAT stack is not built into this kernel. Workaround: run containers with `--network=host`, or set `"iptables": false` in `/etc/docker/daemon.json`.
- The HDMI PHY (`phy-rockchip-inno-hdmi`) is built as a module. Make sure the matching modules are installed under `/lib/modules/$(uname -r)/`, otherwise HDMI output stays black after boot.

## Recovery (if the board won't boot)

1. Power-cycle the board.
2. Check the power supply and cables.
3. If you can drop into the initramfs (busybox) shell, use the rollback script to restore the previous kernel.
4. If you have a TTL-to-UART cable, connect to the serial console, capture the boot log, and open a GitHub issue with it attached.
5. Worst case: re-flash the system image.

Tip: with any other Linux PC you can mount the SD card / eMMC and replace the broken kernel with a known-good one.

## Preliminary Preparations

If you want to compile the kernel manually, install the dependencies first:

- Debian / Ubuntu: `sudo apt install build-essential flex bison bc libncurses-dev`
- Arch: `pacman -S base-devel flex bison bc ncurses`
- Red Hat / Fedora: `dnf install gcc make binutils flex bison bc ncurses-devel`

## Building

You can either use the provided kernel config from this repo, or compile directly from an existing kernel source tree.

1. Get the upstream Linux 7.1.9 source from kernel.org (or use an existing 7.1.9 source tree).
2. (Option A) Use the provided kernel config: copy `config-7.1.9-rk3566-opi3b` from the releases as your `.config`.
   (Option B) Or build directly from your existing 7.1.9 source tree, skipping the config step.
3. Build:

```bash
make olddefconfig
make -j$(nproc) Image dtbs
make -j$(nproc) modules
make modules_install INSTALL_MOD_PATH=/
```

## Docker Fix

The repo ships `./scripts/docker-host-net-fix.sh` to switch Docker into host networking mode (bypassing the missing iptables NAT stack).

Usage:

| Command                            | Function                                    |
| ---------------------------------- | ------------------------------------------- |
| `./scripts/docker-host-net-fix.sh apply`   | Fix Docker as host mode (write daemon.json) |
| `./scripts/docker-host-net-fix.sh restore` | Restore daemon.json from backup             |
| `./scripts/docker-host-net-fix.sh status`  | View Docker status                          |

## Join the Test

If you would like to help with testing, report your device results via GitHub Issues. Please include:

- A title prefixed with `[Test]`
- The system you are using (official, unofficial, other distributions, etc.)
- The output of `uname -a`
- Whether the test passed or failed (or which specific part failed)
- How the error was triggered, if applicable

## License

GPL-2.0 — this is a derivative work of the Linux kernel. See the `LICENSE` file.