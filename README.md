# Turnkey Wireguard Server VPN Gateway Setup
This is the guide about how to use a Turnkey Linux Wireguard Server to route the traffic from all connected Wireguard clients to a secondary VPN gateway.

# Pre-requisites
Firstly, you have alreayd a Turneky Wireguard Server installed on a VM, let's say it's called Wireguard and its LAN IP address is 192.168.1.100 with its Wireguard server IP as 10.0.0.0, whch is the default config right after installation is complete for Turnkey Wireguard Server.

Secondly, you have a VPN gateway server that with IPv4 forward enabled and also tunnel mode enabled (let's say running Clash for Windows as an example) and its LAN IP address is 192.168.0.110 as it has to be in the same LAN with your Wireguard Server


# VPN Routing Setup with Policy-Based Routing

This guide demonstrates a textbook implementation of policy-based routing using `ip rule` and `ip route`. It assigns traffic from a specific subnet to a custom routing table and persists the configuration via a systemd service.

---

## 🛠️ Step 1: Create the Routing Script

Create the script file:

```bash
sudo nano /usr/local/bin/vpnroute.sh
```

Paste the following content:

```bash
#!/bin/bash
ip rule add from 10.0.0.0/24 table vpnroute
ip route add default via 192.168.3.XXX dev eth0 table vpnroute
```

---

## 🔐 Step 2: Make the Script Executable

```bash
sudo chmod +x /usr/local/bin/vpnroute.sh
```

---

## ⚙️ Step 3: Create a Systemd Service

Create the service file:

```bash
sudo nano /etc/systemd/system/vpnroute.service
```

Paste the following content:

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

## 🚀 Step 4: Enable and Start the Service

```bash
sudo systemctl daemon-reexec
sudo systemctl enable vpnroute.service
```

---

## 📁 Step 5: Register the Routing Table

Edit the routing tables file:

```bash
sudo nano /etc/iproute2/rt_tables
```

Add the following line at the bottom:

```
100 vpnroute
```

```bash
sudo systemctl start vpnroute.service
```


---

# Verification and Testing

To verify that your policy-based routing is working as expected, you can use a combination of `ip` commands and packet tracing tools. Here's a step-by-step checklist:

---

## 🧪 Verify Routing Rules and Tables

### 1. **Check the rule**
```bash
ip rule show
```
You should see a line like:
```
0:      from all lookup local
100:    from 10.0.0.0/24 lookup vpnroute
```

### 2. **Inspect the routing table**
```bash
ip route show table vpnroute
```
Expected output:
```
default via 192.168.3.XXX dev eth0
```

---


### 3. **Use `ip route get` to simulate routing**
```bash
ip route get 8.8.8.8 from 10.0.0.0
```
You can replace `10.0.0.0` with a valid IP in your subnet, provided this IP address has been assigned to a device. The output should show routing via `192.168.3.XXX` on `eth0`.

### 4. **Use `ping` or `curl` from a source IP**
If you have a host or container with an IP in `10.0.0.0/24`, try:
```bash
ping -I 10.0.0.0 8.8.8.8
```
or
```bash
curl --interface 10.0.0.0 https://ifconfig.me
```
This helps confirm that traffic is exiting via the expected gateway.


# Clash Verge on Win10 as Gateway - added on 2026-05-06

Above setting are still required, but when using an Clash Verge gateway (let's say 192.168.3.XXX:7897), you'll have to set up also the gloabl environment variables to enable global, system-wide proxy.


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







