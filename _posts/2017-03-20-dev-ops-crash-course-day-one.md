---
layout: post
title: Dev Ops Crash Course - Day One
---

Notes from [day one](http://blog.kate-travers.com/dev-ops-crash-course-day-one/), [day two](http://blog.kate-travers.com/dev-ops-crash-course-day-two/), [day three](http://blog.kate-travers.com/dev-ops-crash-course-day-three/), and [day four](http://blog.kate-travers.com/dev-ops-crash-course-day-four/).

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

```
$ whois 104.131.42.251
#
# Query terms are ambiguous.  The query is assumed to be:
#     "n 104.131.42.251"
#
# Use "?" to get help.
#

#
# The following results may also be obtained via:
# https://whois.arin.net/rest/nets;q=104.131.42.251?showDetails=true&showARIN=false&showNonArinTopLevelNet=false&ext=netref2
#

NetRange:       104.131.0.0 - 104.131.255.255
CIDR:           104.131.0.0/16
NetName:        DIGITALOCEAN-9
NetHandle:      NET-104-131-0-0-1
Parent:         NET104 (NET-104-0-0-0-0)
NetType:        Direct Allocation
OriginAS:       AS46652, AS14061, AS393406, AS62567
Organization:   Digital Ocean, Inc. (DO-13)
RegDate:        2014-06-02
Updated:        2014-06-02
Comment:        http://www.digitalocean.com
Comment:        Simple Cloud Hosting
Ref:            https://whois.arin.net/rest/net/NET-104-131-0-0-1

OrgName:        Digital Ocean, Inc.
OrgId:          DO-13
Address:        101 Ave of the Americas
Address:        10th Floor
City:           New York
StateProv:      NY
PostalCode:     10013
Country:        US
RegDate:        2012-05-14
Updated:        2017-01-28
Comment:        http://www.digitalocean.com
Comment:        Simple Cloud Hosting
Ref:            https://whois.arin.net/rest/org/DO-13

OrgNOCHandle: NOC32014-ARIN
OrgNOCName:   Network Operations Center
OrgNOCPhone:  +1-347-875-6044
OrgNOCEmail:  noc@digitalocean.com
OrgNOCRef:    https://whois.arin.net/rest/poc/NOC32014-ARIN

OrgTechHandle: NOC32014-ARIN
OrgTechName:   Network Operations Center
OrgTechPhone:  +1-347-875-6044
OrgTechEmail:  noc@digitalocean.com
OrgTechRef:    https://whois.arin.net/rest/poc/NOC32014-ARIN

OrgAbuseHandle: ABUSE5232-ARIN
OrgAbuseName:   Abuse, DigitalOcean
OrgAbusePhone:  +1-347-875-6044
OrgAbuseEmail:  abuse@digitalocean.com
OrgAbuseRef:    https://whois.arin.net/rest/poc/ABUSE5232-ARIN
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

HTTP and DNS manage the sending and receiving of web files

Made possible by TCP + IP routing


### Examining Network Traffic

Your computer has a hardware address that you can change. Look up with `ifconfig`.

Example: free wifi networks
  - allow you to log on for limited time, then they try to upsell you
  - they identify your device by hardware address
  - change hardware address will allow you to stay on for free (bc you look like a new device)

`tcpdump` command allows you to examine traffic on a server. [Wire Shark](https://www.wireshark.org/) is a GUI for `tcpdump`. Allows you to examine packets going across network. Can see handshake that's getting made (TCP).

Get capture file (`tcpdump -i eth0 -s 65536 -w capture.pcap`), then import into Wire Shark.

## Transport Layer Security

Data sent using HTTPS is secured via [Transport Layer Security protocol](https://en.wikipedia.org/wiki/Transport_Layer_Security).

When a company registers for a certificate, they provide info and Certificate Authority (trusted entity) verifies it. If valid, authority issues certificate. Certificate Authority signs for your authenticity. Process is not difficult. Relies on trust in the Certificate Authority.

You can be your own Certificate Authority with self-signed certificates, but that's not trusted.

View SSL Certificate on Chrome: open Developer Tools, then click on the Security Tab.

Often in the wild, people terminate SSL at the load balancer, so traffic between load balancer and hosts is unencrypted traveling over private network. Terminating at load balancer is a common pattern and makes things easier to scale. Otherwise you'd have to make sure certificate and keys are on all hosts.

## Encryption

Explaining public/private key encryption: https://www.youtube.com/watch?v=YEBfamv-_do

Understanding the SSH encryption and connection process: https://www.digitalocean.com/community/tutorials/understanding-the-ssh-encryption-and-connection-process

We disabled password identification on our servers (hosts). Instead, we identify by public keys, which is more secure. Developers' public keys have been distributed out to all servers. Then developers can connect from their local machines, where private keys are stored.

## DEMO

Setting up SSL on new Digital Ocean droplet

  1. Create new Droplet
    - Ubuntu
    - $5/mo
    - Datacenter: New York
    - Private networking, backups, IPv6, monitoring
    - Select SSH keys
    - Choose hostname (we used `enroll.flatironschool.com`)
  2. SSH into new droplet as `root` user
    - `ssh root@enroll.flatironschool.com` or `ssh enroll.flatironschool.com -l root`

    !!HOLD UP!!

    Our IP address was released into Digital Ocean pool. We had a server at 104.131.42.251, then deleted that server, so it went back into the pool. BUT we never updated our DNS record for enroll.flatironschool.com.

  3. Go to `manage.dynect.net`
  4. Update A record to new IPv4, add new AAAA record for IPv6
  5. Add PTR record to enroll.flatironschool.com

    Back to setting up SSL...

  6. Get certificate from ssls.com
  7. On server, run `apt-get update`, `apt-get upgrade`
  8. On server, run `openssl req -new -newkey rsa:4096 -nodes -keyout example.key -out example.csr` (generates certificate signing request and key)
  9. Install Apache on server: run `apt-get install apache2` on server
  10. Check if it's running `service apache2 status`
  11. Visit domain in browser, should say "It works!"
  12. On list of certificates, get the one for the domain, enter key and click activate
  13. Look for email when approved
  14. Save stuff into some file (I missed whatever that was)
  15. Now we're following [these instructions for Apache2](https://www.digitalocean.com/community/tutorials/how-to-create-a-ssl-certificate-on-apache-for-ubuntu-14-04) * NOTE: instructions for chain file are outdated
  16. `sudo a2enmod ssl`
  17. `sudo serve apache2 restart`
  18. `sudo vi /etc/apache2/sites-available/default-ssl.conf`
  19. `cat AddTrustExternalCARoot.crt COMODORSAddTrustCA.crt COMODORSADomain...crt enroll_flatironschool_com.crt > cert-ssl-enroll.pem`
  20. `cp example.key ssl-cert-enroll.key`
  21. `mv ssl-cert-enroll.key /etc/ssl/private/`
  22. `mv ssl-cert-enroll.pem /etc/ssl/certs/`
  23. `sudo a2ensite default-ssl`

