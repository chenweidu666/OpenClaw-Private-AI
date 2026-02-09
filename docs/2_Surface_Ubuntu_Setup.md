# Surface Pro 5 安装 Ubuntu 22.04 完整指南

> 本文是 [OpenClaw 实战部署](../README.md) 的配套文档，详细记录在 Surface Pro 5 上安装 Ubuntu 22.04 的完整过程。

## 为什么用 Ubuntu 而不是 Windows？

OpenClaw 在 Linux 下的 `exec` 工具能直接调用 bash 命令读取系统信息，而且 Ubuntu 资源占用远低于 Windows，非常适合当 7×24 小时服务器。

Surface Pro 5 跑 Windows 10 已经非常吃力——开几个 Chrome 标签页内存就飙到 90%，没有风扇的 Surface 直接歇菜。换 Ubuntu 后内存常驻只有 2GB 左右，跑 OpenClaw 绰绰有余。

## 准备工作

- 8GB+ U 盘
- [Ubuntu 22.04.3 LTS Desktop ISO](https://ubuntu.com/download/desktop)
- [Rufus](https://rufus.ie/) 或 [Ventoy](https://www.ventoy.net/) 制作启动盘

## Surface Pro BIOS (UEFI) 设置

Surface 的 BIOS 叫 **UEFI**，进入方式比较特殊：

1. 完全关机
2. **按住音量+键不放**，同时按一下电源键
3. 看到 Surface 标志后松开音量+键
4. 修改 **Boot Configuration** → 拖动 `USB Storage` 到第一位
5. **Secure Boot** → 设为 `Disabled`

> Surface Pro 的触摸屏在 UEFI 里可以用，直接拖拽排序。

## 安装分区方案

| 分区 | 大小 | 挂载点 | 类型 |
|------|------|--------|------|
| EFI | 300MB | /boot/efi | FAT32 |
| 系统 | 剩余全部 | / | ext4 |
| Swap | 等于内存 (8GB) | swap | swap |

## Surface 专用内核（推荐）

Ubuntu 默认内核对 Surface 的触摸屏、WiFi 支持不完美，安装 [linux-surface](https://github.com/linux-surface/linux-surface) 内核可以显著改善：

```bash
# 添加 linux-surface 仓库
wget -qO - https://raw.githubusercontent.com/linux-surface/linux-surface/master/pkg/keys/surface.asc \
  | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/linux-surface.gpg > /dev/null

echo "deb [arch=amd64] https://pkg.surfacelinux.com/debian release main" \
  | sudo tee /etc/apt/sources.list.d/linux-surface.list

sudo apt update
sudo apt install linux-image-surface linux-headers-surface libwacom-surface
sudo update-grub && sudo reboot
```

### 安装后验证

```bash
uname -r
# 应显示类似 6.x.x-surface 的内核版本
```

## 基础优化

```bash
# 系统更新
sudo apt update && sudo apt upgrade -y

# 常用工具
sudo apt install -y curl wget git vim htop lm-sensors net-tools

# 初始化温度传感器
sudo sensors-detect

# 设置时区
sudo timedatectl set-timezone Asia/Shanghai

# 验证温度读取
sensors
```

## 配置 SSH 远程管理

配好 SSH 之后，Surface Pro 就可以合上盖子放在角落，用其他电脑远程操作了：

```bash
# 安装 SSH
sudo apt install -y openssh-server
sudo systemctl enable ssh && sudo systemctl start ssh

# 主力电脑上配置密钥登录
ssh-keygen -t ed25519 -C "your_email@example.com"
ssh-copy-id user@192.168.1.100
```

> 建议在路由器上给 Surface Pro 绑定固定 IP，方便后续配置。

## 安装 Node.js 22

OpenClaw 强制要求 Node.js v22+：

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs
node -v  # 确认 v22.x
```

---

> 完成以上步骤后，回到 [OpenClaw 实战部署主文档](../README.md) 继续安装 OpenClaw。
