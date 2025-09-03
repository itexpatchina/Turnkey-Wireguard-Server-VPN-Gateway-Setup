# Turnkey Wireguard Server VPN Gateway Setup
This is the guide about how to use a Turnkey Linux Wireguard Server to route the traffic from all connected Wireguard clients to a secondary VPN gateway.

# Pre-requisites
Firstly, you have alreayd a Turneky Wireguard Server installed on a VM, let's say it's called Wireguard and its LAN IP address is 192.168.1.100 with its Wireguard server IP as 10.0.0.0, whch is the default config right after installation is complete for Turnkey Wireguard Server.

Secondly, you have a VPN gateway server that with IPv4 forward enabled and also tunnel mode enabled (let's say running Clash for Windows as an example) and its LAN IP address is 192.168.0.110 as it has to be in the same LAN with your Wireguard Server

