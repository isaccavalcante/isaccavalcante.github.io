+++
title = "Setting up a Virtual Lab with Virtualbox"
description = "Setting up a Virtual Lab with Virtualbox"
date = "2019-02-28"
author = "Isac Cavalcante"
cover = "images/topology.png"
+++
# Setting up a Virtual Lab with Virtualbox

This post shows how to configure an virtual environment for pentesting, researching and experimentation using [VirtualBox](https://www.virtualbox.org), a virtualization software that allow us to simulate a real life scenario. We will have the topology described above.

Configure the VMs with the following specifications by downloading the ISO files.

* Gateway

| Configuration      | Value   |
|--------------------|---------|
| OS                 | [Ubuntu 18.04](http://ubuntu.c3sl.ufpr.br/releases/18.04.3/ubuntu-18.04.3-live-server-amd64.iso)   |
| CPU                | 1                   |
| RAM                | 1GB                 |
| Network Interfaces | 1 Host-Only, 1 NAT  |
| IP Address         | 10.0.0.1/24         |
| User               | user                |
| Password           | password            |

* Victim

| Configuration      | Value   |
|--------------------|---------|
| OS                 | [Windows 10](https://az792536.vo.msecnd.net/vms/VMBuild_20190311/VirtualBox/MSEdge/MSEdge.Win10.VirtualBox.zip)   |
| CPU                | 1            |
| RAM                | 2GB          |
| Network Interfaces | 1 Host-Only  |
| IP Address         | 10.0.0.2/24  |
| User               | IE User      |
| Password           | Passw0rd!    |

* Attacker

| Configuration      | Value   |
|--------------------|---------|
| OS                 | [Kali 2.0](https://cdimage.kali.org/kali-2019.3/kali-linux-2019.3-amd64.iso)   |
| CPU                | 2            |
| RAM                | 4GB          |
| Network Interfaces | 1 Host-Only  |
| IP Address         | 10.0.0.3/24  |
| User               | root         |
| Password           | toor         |

You will need to Create a Host Only network interface to be used by all VMs. Go to the main menu, select `File > Host Network Manager`, then you should see an empty white box with "Host-Only Networks". Click the create button. A new Host-only network will be created and added to the list, you can now change the network adapter in you VMs to use the one you just created. For the Gateway VM, you'll also need to add an NAT network interface.


## Configuring the Attacker

For this Kali VM we are considering the following network interface:

* `eth0`: Host-Only (Local/internal) -> 10.0.0.3

In your attacker machine, go to the upper right corner in the network icon, `Wired Connected > Wired Settings > Network > Wired > Engine icon > IPv4` and fill in with the following values:

![kali-network](/images/kali-network.png)


## Configuring the Victim

For this Windows VM we are considering the following network interface:

* `Ethernet`: Host-Only (Local/internal) -> 10.0.0.2

Click on the network icon in right lower corner, click on `Network & Internet Settings > Change your adapter options`, then right click in the network interface, click on `Properties > Internet Protocol Version 4 (TCP/IPv4)` and fill in with the following values:

![windows-network](/images/windows-network.png)

## Configuring the Gateway


Check all the network interface connected to your VM:
```sh
ifconfig -a

```

Install the `ifupdown` tool to configure the adition network interface:
```sh
apt update
apt install ifupdown
```

Edit /etc/network/interfaces with the following content:
```
iface inet enp0s8 inet static
	address 10.0.0.1
	netmask 255.255.255.0
```
> Change the `enp0s8` accordingly to the output from the `ifconfig -a` command.


Bring the interface up and running with the following command:
```sh
ifup enp0s8
```

For this VM we are considering the following interfaces

* `enp0s3`: NAT (Internet/external) -> 10.0.2.15*
* `enp0s8`: Host-Only (Local/internal) 10.0.0.1

> \*This will change accordingly to VirtuaBox

Enable packet forwarding:
```sh
echo 1 > /proc/sys/net/ipv4/ip_forward
```
Allow every traffic that comes through local interface to go out to the internet interface
```sh
iptables -A FORWARD -i enp0s8 -o enp0s3 -j ACCEPT 
```

Allow every traffic that comes through internet interface to come in to the local interface
```sh
iptables -A FORWARD -i enp0s3 -o enp0s8 -j ACCEPT -m state --state ESTABLISHED,RELATED 
```

Change the source IP address for the outgoing packets
```sh
iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE 
```
> This will make the hosts address be the gateway address to the internet


Ping the other hosts (Required by VirtualBox)
```sh
ping 10.0.0.2 -c 1
ping 10.0.0.3 -c 1
```



This script configures all the previous commands. 
```bash
#!/bin/bash
# configure_gateway.sh
# 

LOCAL_IFACE=enp0s8
EXTERNAL_IFACE=enp0s3
GATEWAY_ADDRESS=10.0.0.1
HOST_ADDRESSES='10.0.0.2 10.0.0.3'

apt update && apt install -y ifupdown 

cat <<EOF > /etc/network/interfaces
iface $LOCAL_IFACE inet static
	address $GATEWAY_ADDRESS
	netmask 255.255.255.0
EOF

ifup $LOCAL_IFACE

echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -A FORWARD -i $LOCAL_IFACE -o $EXTERNAL_IFACE -j ACCEPT 
iptables -A FORWARD -i $EXTERNAL_IFACE -o $LOCAL_IFACE -j ACCEPT -m state --state ESTABLISHED,RELATED 
iptables -t nat -A POSTROUTING -o $EXTERNAL_IFACE -j MASQUERADE 


for ADDRESS in $HOST_ADDRESSES; do
  ping $ADDRESS -c 1
done
```