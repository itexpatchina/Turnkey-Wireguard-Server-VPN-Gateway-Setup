# Turnkey WireGuard Server VPN Gateway Setup
> **Turnkey WireGuard 服务器 VPN 网关路由配置**

🔗 **Source Repository:** [https://github.com/itexpatchina/Turnkey-Wireguard-Server-VPN-Gateway-Setup](https://github.com/itexpatchina/Turnkey-Wireguard-Server-VPN-Gateway-Setup)  
> 🔗 **源仓库：** [https://github.com/itexpatchina/Turnkey-Wireguard-Server-VPN-Gateway-Setup](https://github.com/itexpatchina/Turnkey-Wireguard-Server-VPN-Gateway-Setup)

This is the guide about how to use a Turnkey Linux Wireguard Server to route the traffic from all connected Wireguard clients to a secondary VPN gateway.
> 本指南介绍如何使用 Turnkey Linux WireGuard 服务器，将所有连接的 WireGuard 客户端流量重定向路由至备用 VPN 网关。

---

## Pre-requisites
> **前置条件**

Firstly, you have alreayd a Turneky Wireguard Server installed on a VM, let's say it's called Wireguard and its LAN IP address is 192.168.1.100 with its Wireguard server IP as 10.0.0.0, whch is the default config right after installation is complete for Turnkey Wireguard Server.
> 首先，你已经在虚拟机上安装好了一个 Turnkey WireGuard 服务器。假设它命名为 Wireguard，其局域网 IP 地址为 192.168.1.100，WireGuard 服务端内网 IP 地址为 10.0.0.0（这是 Turnkey WireGuard 服务器完成安装后的默认配置）。

Secondly, you have a VPN gateway server that with IPv4 forward enabled and also tunnel mode enabled (let's say running Clash for Windows as an example) and its LAN IP address is 192.168.0.110 as it has to be in the same LAN with your Wireguard Server
> 其次，你需要一台启用了 IPv4 转发和 TUN 隧道模式的 VPN 网关服务器（例如运行 Clash for Windows），其局域网 IP 地址假设为 192.168.0.110（必须与你的 WireGuard 服务器处于同一局域网网段）。

---

## VPN Routing Setup with Policy-Based Routing
> **基于策略路由的 VPN 路由配置**

This guide demonstrates a textbook implementation of policy-based routing using ip rule and ip route . It assigns traffic from a specific subnet to a custom routing table and persists the configuration via a systemd service.
> 本指南展示了使用 `ip rule` 和 `ip route` 进行策略路由的标准实现。它将来自特定子网的流量分配给自定义路由表，并通过 systemd 服务进行持久化配置。

---

### 🛠 Step 1: Create the Routing Script
> **🛠 步骤 1：创建路由脚本**

Create the script file:
> 创建脚本文件：

```bash
sudo nano /usr/local/bin/vpnroute.sh
```

Paste the following content:
> 粘贴以下内容：

```bash
#!/bin/bash
ip rule add from 10.0.0.0/24 table vpnroute
ip route add default via 192.168.3.XXX dev eth0 table vpnroute
```

---

### 🔐 Step 2: Make the Script Executable
> **🔐 步骤 2：赋予脚本可执行权限**

```bash
sudo chmod +x /usr/local/bin/vpnroute.sh
```

---

### ⚙ Step 3: Create a Systemd Service
> **⚙ 步骤 3：创建 Systemd 服务**

Create the service file:
> 创建服务文件：

```bash
sudo nano /etc/systemd/system/vpnroute.service
```

Paste the following content:
> 粘贴以下内容：

```ini
[Unit]
Description=Apply vpnroute routing rules
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/vpnroute.sh
Type=oneshot
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

---

### 🚀 Step 4: Enable and Start the Service
> **🚀 步骤 4：启用并启动服务**

```bash
sudo systemctl daemon-reexec
sudo systemctl enable vpnroute.service
```

---

### 📁 Step 5: Register the Routing Table
> **📁 步骤 5：注册路由表**

Edit the routing tables file:
> 编辑路由表配置文件：

```bash
sudo nano /etc/iproute2/rt_tables
```

Add the following line at the bottom:
> 在文件底部添加以下行：

```text
100 vpnroute
```

```bash
sudo systemctl start vpnroute.service
```

---

## Verification and Testing
> **验证与测试**

To verify that your policy-based routing is working as expected, you can use a combination of ip commands and packet tracing tools. Here's a step-by-step checklist:
> 要验证基于策略的路由是否如预期工作，可以结合使用 `ip` 命令和数据包追踪工具。以下是逐步排查清单：

##### 1. Check the rule
> **1. 检查规则**

##### 2. Inspect the routing table
> **2. 查看路由表**

##### 3. Use ip route get to simulate routing
> **3. 使用 ip route get 模拟路由路径**

##### 4. Use ping or curl from a source IP
> **4. 从源 IP 使用 ping 或 curl 测试**

If you have a host or container with an IP in 10.0.0.0/24 , try:
> 如果你的主机或容器 IP 属于 10.0.0.0/24 网段，请尝试：

```bash
ping -I 10.0.0.0 8.8.8.8
```

or
> 或

```bash
curl --interface 10.0.0.0 https://ifconfig.me
```

This helps confirm that traffic is exiting via the expected gateway.
> 这有助于确认流量是否通过预期的网关出口。

---

## Clash Verge on Win10 as Gateway - added on 2026-05-06
> **使用 Win10 上的 Clash Verge 作为网关 - 2026-05-06 补充**

Above setting are still required, but when using an Clash Verge gateway (let's say 192.168.3.XXX:7897), you'll have to set up also the gloabl environment variables to enable global, system-wide proxy.
> 上述设置依然是必需的，但当使用 Clash Verge 作为网关时（假设为 192.168.3.XXX:7897），你还必须配置全局环境变量以启用系统级全局代理。

```bash
# edit global env
cat >> /etc/environment <<EOF
http_proxy=http://192.168.3.XXX:7897
https_proxy=http://192.168.3.XXX:7897
all_proxy=socks5://192.168.3.XXX:7897
HTTP_PROXY=http://192.168.3.XXX:7897
HTTPS_PROXY=http://192.168.3.XXX:7897
ALL_PROXY=socks5://192.168.3.XXX:7897
no_proxy=127.0.0.1,localhost,192.168.3.0/24
NO_PROXY=127.0.0.1,localhost,192.168.3.0/24
EOF

# apt proxy config
echo 'Acquire::http::Proxy "http://192.168.3.XXX:7897";' > /etc/apt/apt.conf.d/99proxy
echo 'Acquire::https::Proxy "http://192.168.3.XXX:7897";' >> /etc/apt/apt.conf.d/99proxy

source /etc/environment
```
