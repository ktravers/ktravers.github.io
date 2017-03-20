---
layout: post
title: Dev Ops Crash Course - Day One
---

## Internet 101

Videos: https://www.khanacademy.org/computing/computer-science/internet-intro/internet-works-intro/v/the-internet-wires-cables-and-wifi

Some vocab:
  - Bandwidth: capacity to receive information
  - Bitrate: speed of sending information (bits per second)
  - Latency: drag / delay when sending information

Internet is a design philosophy expressed through agreed upon protocols:
  - IPv4 - current protocol (32bit)
  - IPv6 - new protocol to provide more IP addresses (128bit)

IP addresses have A, B, and C classes.

## DNS

How DNS works: https://howdns.works/

In local network settings, DNS is set to [Google Public DNS](https://en.wikipedia.org/wiki/Google_Public_DNS).
  - better performance
  - better security

Anatomy of a URL: https://doepud.co.uk/blog/anatomy-of-a-url

|protocol|subdomain|domain     |
|--------|---------|-----------|
|https://|www.     |google.com |


## DNS records

Record types:
  1. **A record:** address, maps hostname to physical IP address
  2. **PTR record:** pointer, maps IP to name (helpful when scanning logs)
  3. **CNAME record:** alias, map domain name to another domain name

Can use multiple records for redundancy strategy. For example, you want multiple MX records for mail servers. Config priority on each record.

### DNS lookup

```
$ dig learn.co

; <<>> DiG 9.8.3-P1 <<>> learn.co
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 60914
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;learn.co.      IN  A

;; ANSWER SECTION:
learn.co.   212 IN  A 45.55.127.202

;; Query time: 29 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Mon Mar 20 12:06:16 2017
;; MSG SIZE  rcvd: 42
```

### Local spoofing:

1. Open localhost file: `/etc/hosts`

2. Add spoofed DNS entries (IP address, site+domain):

  ```
  36.36.36.36  learn.co
  ```

### DHCP

[DHCP](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol) "dynamically distributes network configuration parameters, such as IP addresses". Process of obtaining IP address on private network. It's obtained on a lease that will eventually expire.

Router gives out IP addresses. There's a limited number.

You can configure leases to expire after a certain period of time. Helps prevent running out.

## Packets and Traffic

Packets directed by routers. Routers owned by ISPs.

See the path of your request:

```
$ traceroute learn.co
traceroute to learn.co (45.55.127.202), 64 hops max, 52 byte packets
 1  lo0-100.nycmny-vfttp-407.verizon-gni.net (71.190.177.1)  4.954 ms  2.340 ms  2.594 ms
 2  b3407.nycmny-lcr-22.verizon-gni.net (100.41.0.152)  6.657 ms
    b3407.nycmny-lcr-21.verizon-gni.net (100.41.0.150)  7.809 ms  11.878 ms
 3  * * *
 4  * * *
 5  0.ae13.gw10.ewr6.alter.net (140.222.235.117)  29.671 ms
    0.ae1.gw10.ewr6.alter.net (140.222.230.151)  13.458 ms  6.881 ms
 6  157.130.91.86 (157.130.91.86)  14.051 ms  12.765 ms  10.800 ms
 7  nyk-bb4-link.telia.net (62.115.137.100)  5.831 ms
    nyk-bb4-link.telia.net (62.115.137.98)  7.330 ms
    nyk-bb1-link.telia.net (213.155.130.29)  6.450 ms
 8  nyk-b3-link.telia.net (80.91.247.21)  10.902 ms
    nyk-b3-link.telia.net (80.239.147.132)  6.161 ms
    nyk-b3-link.telia.net (62.115.123.49)  13.696 ms
 9  digitalocean-ic-306498-nyk-b3.c.telia.net (62.115.45.10)  9.293 ms  10.936 ms
    digitalocean-ic-306497-nyk-b3.c.telia.net (62.115.45.6)  18.379 ms
10  * * *
11  * * *
12  * * *
13  * * *
14  * * *
15  * * *
16  * * *
```

### TCP

TCP manages sending and receiving of packets (Transmission Control Protocol).

### Examining Network Traffic

Your computer has a hardware address that you can change. Look up with `ifconfig`.

Example: free wifi networks
  - allow you to log on for limited time, then they try to upsell you
  - they identify your device by hardware address
  - change hardware address will allow you to stay on for free (bc you look like a new device)

`tcpdump` command allows you to examine traffic on a server. [Wire Shark](https://www.wireshark.org/) is a GUI for `tcpdump`. Allows you to examine packets going across network. Can see handshake that's getting made (TCP).

Get capture file (`tcpdump -i eth0 -s 65536 -w capture.pcap`), then import into Wire Shark.
