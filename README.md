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
ip route add default via 192.168.3.110 dev eth0 table vpnroute
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
sudo systemctl start vpnroute.service
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

---

# Verification and Testing

To verify that your policy-based routing is working as expected, you can use a combination of `ip` commands and packet tracing tools. Here's a step-by-step checklist:

---

## ✅ Verify Routing Rules and Tables

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
default via 192.168.3.110 dev eth0
```

---

## 🧪 Test Routing Behavior

### 3. **Use `ip route get` to simulate routing**
```bash
ip route get 8.8.8.8 from 10.0.0.123
```
Replace `10.0.0.123` with a valid IP in your subnet. The output should show routing via `192.168.3.110` on `eth0`.

### 4. **Use `ping` or `curl` from a source IP**
If you have a host or container with an IP in `10.0.0.0/24`, try:
```bash
ping -I 10.0.0.123 8.8.8.8
```
or
```bash
curl --interface 10.0.0.123 https://ifconfig.me
```
This helps confirm that traffic is exiting via the expected gateway.



