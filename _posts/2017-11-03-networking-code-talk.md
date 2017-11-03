---
layout: post
title: Code Talk - Networking
---

Low level talk about how information moves across the internet.

Why? Because we’re moving to AWS and [Scot](https://github.com/awesomescot) wants to try teaching this.


## Types of addresses:

1. IP address
  - Dynamic
  - Change every time you reboot
  - Somewhat related to geolocation (network location associated with physical location)
  - Higher level than Mac address (IP layer, layer 3)
  - 32bit, example: `10.10.2.2`

2. Mac address
  - Static
  - Hardwired to computer's network card
  - Lower level (ethernet, layer 2)
  - 48bit, usually represented in hex, example: `F1:4F:AF:FF:FF:FF`


`ifconfig` - shows IP address, netmask, _and more_

```bash
$ ifconfig
lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> mtu 16384
  options=1203<RXCSUM,TXCSUM,TXSTATUS,SW_TIMESTAMP>
  inet 127.0.0.1 netmask 0xff000000
  inet6 ::1 prefixlen 128
  inet6 fe80::1%lo0 prefixlen 64 scopeid 0x1
  nd6 options=201<PERFORMNUD,DAD>
gif0: flags=8010<POINTOPOINT,MULTICAST> mtu 1280
stf0: flags=0<> mtu 1280
en0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
  ether a4:5e:60:d0:47:85
  inet6 fe80::4b4:2f9:9d8c:bc76%en0 prefixlen 64 secured scopeid 0x4
  inet 10.9.97.112 netmask 0xfffff800 broadcast 10.9.103.255
  nd6 options=201<PERFORMNUD,DAD>
  media: autoselect
  status: active
```

`ipcalc <ip address> <netmask>` - tells you stuff about ip address and subnet

```bash
$ ipcalc 10.9.97.112 0xfffff800
Address:   10.9.97.112          00001010.00001001.01100 001.01110000
Netmask:   255.255.248.0 = 21   11111111.11111111.11111 000.00000000
Wildcard:  0.0.7.255            00000000.00000000.00000 111.11111111
=>
Network:   10.9.96.0/21         00001010.00001001.01100 000.00000000
HostMin:   10.9.96.1            00001010.00001001.01100 000.00000001
HostMax:   10.9.103.254         00001010.00001001.01100 111.11111110
Broadcast: 10.9.103.255         00001010.00001001.01100 111.11111111
Hosts/Net: 2046                  Class A, Private Internet
```

Note: `brew install ipcalc` if not already installed

Subnet is a neighborhood. Individual connections have IP addresses, grouped by subnet.

## Local network communication

Local communication uses the Mac address.

Packet: data we're sending across the internet, has many layers

Packet layers (inner to outer):

1. JSON from application (for example)
2. HTTP request
3. TCP
4. IP address (both source and destination)
5. Mac address (both source and destination)

Getting closer to hardware as we go outwards.

**Scenario:** on our local network, we want to send a packet to everyone in subnet

When sending packets in local network, we go from IP address to Mac address using ARP request. ARP request asks "who has this IP address?", then whichever host responds "yes". Then source host remembers destination host Mac address.

Get all address in local network: `arp -a`

```bash
$ arp -a
? (10.9.96.1) at 0:0:5e:0:1:14 on en0 ifscope [ethernet]
? (10.9.97.117) at e0:5f:45:8d:ea:b0 on en0 ifscope [ethernet]
? (10.9.97.127) at 64:b0:a6:e1:8e:f8 on en0 ifscope [ethernet]
? (10.9.97.201) at 60:f4:45:8c:56:af on en0 ifscope [ethernet]
? (10.9.97.209) at d4:61:9d:0:7c:c6 on en0 ifscope [ethernet]
```

## Default gateway and routes

How to communicate outside our local network?

Our router is our default gateway. Check it out in System Preferences > Network > TCP/IP.

In our packet, Mac address is local router, and IP address is for example, Google's IP address.

Route tables resolve urls to IP addresses.

Check out routes using `netstat -nr`

```bash
$ netstat -nr
Routing tables

Internet:
Destination        Gateway            Flags        Refs      Use   Netif Expire
default            10.9.96.1          UGSc           39        0     en0
10.9.96/21         link#4             UCS            37        0     en0
10.9.96.1/32       link#4             UCS             1        0     en0
10.9.96.1          0:0:5e:0:1:14      UHLWIir        39       90     en0   1085
10.9.96.2          b0:c6:9a:77:c6:10  UHLWI           0        1     en0   1143
10.9.97.112/32     link#4             UCS             0        0     en0
10.9.97.127        64:b0:a6:e1:8e:f8  UHLWI           0        0     en0    545
10.9.97.201        60:f4:45:8c:56:af  UHLWI           0        0     en0   1025
10.9.97.209        d4:61:9d:0:7c:c6   UHLWI           0        0     en0    763
10.9.97.225        e0:ac:cb:8c:d6:60  UHLWI           0        0     en0    752
10.9.97.227        0:24:36:b8:14:1a   UHLWI           0        0     en0    830
```

