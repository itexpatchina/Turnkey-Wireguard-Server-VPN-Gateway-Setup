# Turnkey WireGuard Server & VPN Gateway Setup | Turnkey WireGuard 服务器与 VPN 网关配置

A comprehensive guide and configuration reference for deploying a **Turnkey Linux WireGuard Server** that routes traffic from connected WireGuard clients through a secondary upstream VPN gateway (such as Astrill, VLESS REALITY, or OpenVPN).

这是一个部署 **Turnkey Linux WireGuard 服务器** 的完整指南与配置参考。该方案可将所有连接的 WireGuard 客户端流量，通过二次上游 VPN 网关（如 Astrill、VLESS REALITY、OpenVPN 等）进行智能重定向与路由。

---

## 📐 Network Topology | 网络拓扑架构

```
[ Remote WireGuard Clients / 远程客户端 ]
                   │
                   ▼ (WireGuard Tunnel / 加密隧道)
    [ Turnkey WireGuard Server (192.168.3.109) ]
                   │
                   ▼ (Default Gateway Policy / 默认网关重定向)
     [ Secondary VPN Gateway (e.g., 192.168.3.140) ]
                   │
                   ▼ (Encrypted Egress / 境外加密出口)
           [ Internet / GFW Bypass ]
```

---

## ✨ Features | 核心功能

### English
* **Centralised Gateway Routing:** Force all connected WireGuard VPN clients to egress through a designated secondary VPN router or proxy host on the LAN.
* **Turnkey Linux Integration:** Lightweight, low-overhead Debian-based Turnkey Linux container/VM setup running on ESXi.
* **Automated NAT & Forwarding:** Configured with `iptables` rules and kernel IP forwarding (`net.ipv4.ip_forward=1`).
* **Cross-Border Optimization:** Bypasses local network restrictions (GFW) for all connected home and mobile devices.

### 中文说明
* **集中式网关路由：** 强制所有已连接的 WireGuard VPN 客户端通过局域网内指定的二次 VPN 路由器或代理主机出口。
* **Turnkey Linux 集成：** 基于 ESXi 上的轻量级 Debian Turnkey Linux 容器/虚拟机构建，内存与 CPU 占用极低。
* **自动 NAT 与转发：** 预配置 `iptables` MASQUERADE 规则与内核 IP 转发（`net.ipv4.ip_forward=1`）。
* **跨境网络优化：** 为所有连入的移动设备与远程终端提供无缝的 GFW 绕过与网络加速能力。

---

## 🛠️ Quick Setup Guide | 快速部署指南

### 1. Enable IP Forwarding | 开启 IP 转发

On your Turnkey WireGuard server, ensure IPv4 packet forwarding is enabled:

在 Turnkey WireGuard 服务器上开启内核 IPv4 数据包转发：

```bash
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

### 2. Configure WireGuard Interface (`/etc/wireguard/wg0.conf`) | 配置 WireGuard 接口

Edit `/etc/wireguard/wg0.conf` to set up PostUp / PostDown rules for NAT masquerading:

编辑 `/etc/wireguard/wg0.conf` 文件，设置 NAT 地址伪装与防火墙规则：

```ini
[Interface]
Address = 10.8.0.1/24
ListenPort = 51820
PrivateKey = <SERVER_PRIVATE_KEY>

# Add iptables rules for NAT masquerade
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

---

### 3. Change Default Gateway | 修改默认网关

To route all outbound client traffic through the secondary VPN gateway (e.g., `192.168.3.140`):

若需将所有客户端出口流量通过二次 VPN 网关（例如 `192.168.3.140`）转发：

```bash
# Remove current default gateway
sudo ip route del default

# Set the secondary VPN gateway as default
sudo ip route add default via 192.168.3.140 dev eth0
```

---

## 📄 License & Usage | 许可证与使用说明

Licensed under the **GNU General Public License v3.0**.  
本项目遵循 **GNU General Public License v3.0** 开源许可证。

*Disclaimer: This repository is intended for homelab research and network routing experiments.*  
*免责声明：本项目仅供 Homelab 网络研究与个人路由实验使用。*
