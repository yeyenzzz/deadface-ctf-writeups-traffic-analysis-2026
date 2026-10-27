# CTF Write-Up: DNS Zone Transfer Analysis

##  Challenge Overview

* **Category:** Network Forensics / DNS Enumeration
* **Points:** 50
* **Tools Used:** Wireshark / Network Packet Analyzer


##  Step 1: Initial Traffic Triage & Protocol Analysis

Upon opening the packet capture in Wireshark, filtering for DNS traffic reveals a series of standard and administrative queries targeting the `deadface-cc.net` domain zone.

I think the attacker is scanning for the available dns

##  Step 2: Identifying the Zone Transfer (AXFR) Leak

In this capture, examining the DNS payload response reveals a complete enumeration of the domain's infrastructure architecture, exposing internal and external subdomains such as:

* `ns1.deadface-cc.net` & `ns2.deadface-cc.net`
* `www.deadface-cc.net` & `mail.deadface-cc.net`
* `admin.deadface-cc.net` & `intranet.deadface-cc.net`
* `dev.deadface-cc.net`, `staging.deadface-cc.net`, & `test.deadface-cc.net`
* `db.deadface-cc.net`, `secret.deadface-cc.net`, & `vpn.deadface-cc.net`


##  Step 3: Extracting the Flag

Following the stream of the zone transfer uncovers a dedicated entry under the zone:

```text
_flag.deadface-cc.net -> deadface{dns_zone_transfer}

```

### Output / Flag:

```text
deadface{dns_zone_transfer}

```

