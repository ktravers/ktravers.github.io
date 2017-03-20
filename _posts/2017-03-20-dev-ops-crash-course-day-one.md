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


Record types:
  1. **A record:** address, maps hostname to physical IP address
  2. **PTR record:** pointer, maps IP to name (helpful when scanning logs)
  3. **CNAME record:** alias, map domain name to another domain name


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
