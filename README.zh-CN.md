# Linux Kernel 7.x for Orange Pi 3B（RK3566）

一个为 Orange Pi 3B 定制的、基于主线内核（7.1.9）的精简内核。可配合通用 Linux rootfs 使用——不绑定任何特定发行版。

> 警告：刷写新内核前请备份数据，并确保有可用的恢复途径（UART 串口线，或另一台能读取你 SD 卡 / eMMC 的 Linux 电脑）。

## 特性

- SCX（sched_ext）—— 基于 eBPF 的可扩展 CPU 调度器
- f2fs / erofs 优化
- io_uring（`CONFIG_IO_URING=y`）
- eBPF JIT，默认启用

## 兼容性

仅验证过 Orange Pi 3B 硬件版本 **v2.1**。v1.1 版本不支持。

| Rootfs | 状态 |
| ------ | ------ |
| Armbian（Debian，minimal） | 已测试 |
| Armbian（Ubuntu，任意桌面） | 未测试，预期可用 |
| 其他发行版 | 未验证 —— 欢迎反馈测试结果 |

## 已知问题

- Docker 在 NAT / bridge 模式下无法启动，因为本内核未内置 legacy iptables NAT 栈。解决办法：容器用 `--network=host` 运行，或在 `/etc/docker/daemon.json` 中设置 `"iptables": false`。
- HDMI PHY（`phy-rockchip-inno-hdmi`）是作为**模块**编译的。请确保匹配的模块已安装到 `/lib/modules/$(uname -r)/` 下，否则开机后 HDMI 输出会保持黑屏。

## 恢复方法（如果板子无法启动）

1. 板子断电重启。
2. 检查电源和线缆。
3. 如果能进入 initramfs（busybox）shell，使用回滚脚本恢复之前的内核。
4. 如果有 TTL 转 UART 线，连接到串口控制台，抓取启动日志，并附到 GitHub issue 中。
5. 最坏情况：重新刷写系统镜像。

提示：借助任意其他 Linux 电脑，你可以挂载 SD 卡 / eMMC，用已知可用的内核替换损坏的内核。

## 前期准备

如果你想手动编译内核，请先安装依赖：

- Debian / Ubuntu：`sudo apt install build-essential flex bison bc libncurses-dev`
- Arch：`pacman -S base-devel flex bison bc ncurses`
- Red Hat / Fedora：`dnf install gcc make binutils flex bison bc ncurses-devel`

## 编译

你可以使用本仓库提供的内核配置，也可以直接用现成的内核源码树编译。

1. 从 kernel.org 获取上游 Linux 7.1.9 源码（或直接使用现有的 7.1.9 源码树）。
2. （方式 A）使用仓库配置：把 releases 中的 `config-7.1.9-rk3566-opi3b` 复制为你的 `.config`。
   （方式 B）或直接用现成的 7.1.9 源码树编译，跳过配置步骤。
3. 编译：

```bash
make olddefconfig
make -j$(nproc) Image dtbs
make -j$(nproc) modules
make modules_install INSTALL_MOD_PATH=/
```

## Docker 修复

仓库提供了 `./scripts/docker-host-net-fix.sh` 脚本，用于将 Docker 切换到 host 网络模式（绕过缺失的 iptables NAT 栈）。

用法：

| 命令 | 功能 |
| ---------------------------------- | ------------------------------------------- |
| `./scripts/docker-host-net-fix.sh apply`   | 将 Docker 修复为 host 模式（写入 daemon.json） |
| `./scripts/docker-host-net-fix.sh restore` | 从备份恢复 daemon.json             |
| `./scripts/docker-host-net-fix.sh status`  | 查看 Docker 状态                          |

## 参与测试

如果你想帮忙测试，请通过 GitHub Issues 报告你的设备测试结果。请包含：

- 标题以 `[Test]` 开头
- 你使用的系统（官方、非官方、其他发行版等）
- `uname -a` 的输出
- 测试是通过还是失败（或具体哪部分失败）
- 错误是如何触发的（如适用）

## 许可证

GPL-2.0 —— 本内核是 Linux 内核的衍生作品。参见 `LICENSE` 文件。