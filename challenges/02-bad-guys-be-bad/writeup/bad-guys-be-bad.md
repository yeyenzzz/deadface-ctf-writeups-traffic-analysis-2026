# CTF Write-Up: Bad Guys Be Bad

## Challenge Overview

* **Category:** Network Forensics / C2 Traffic Analysis
* **Points:** 350
* **Objective:** Analyze the provided PCAP file (`deadface_pcap02.pcap`), identify Command and Control (C2) staging communications, track data exfiltration channels, and decode the final flag.
* **Tools Used:** Wireshark, CyberChef

##  Step 1: Initial Traffic Triage & Protocol Hierarchy

Upon opening `deadface_pcap02.pcap` in Wireshark, the packet capture reveals a substantial volume of traffic.

* To gain a high-level overview of the communication structure, navigate to **Statistics > Protocol Hierarchy Statistics**.


* The hierarchy highlights significant traffic distributed across UDP (DNS) and TCP (HTTP). Given that unencrypted protocols often hold clues or configuration files, the investigation begins by filtering for cleartext HTTP traffic.

##  Step 2: Discovering the Staging Configuration via HTTP

Filtering for HTTP traffic reveals multiple web requests and responses. Following the HTTP stream of suspicious requests leads to a file download from `45.13.37.7:8080` for `/staging.bin`.

Examining the response payload uncovers a critical staging configuration file:

```text
# staging config -- do not commit
mode=exfil
channel=dns
xor_key=Sp00ky
target=exfil.deadface-c2.net

```

This configuration provides clue for the next phase of the investigation:

* **Exfiltration Channel:** DNS


* **XOR Key:** `Sp00ky`


* **C2 Target Domain:** `exfil.deadface-c2.net`

## Step 3: Tracking DNS Exfiltration Traffic

Armed with the C2 domain from the staging config, a Wireshark display filter is applied to isolate relevant DNS queries:

```wireshark
dns.qry.name contains "exfil.deadface-c2.net"

```

* Inspecting the filtered packets reveals anomalous DNS queries where data chunks are encoded directly into subdomains (e.g., `s00-G4KVCVANDAYB...`, `s01-...`).


* Rather than functioning as standard domain name lookups, these queries are being used as a covert channel to smuggle data out of the network.

---

##  Step 4: Reassembling and Decoding the Payload

Extracting and concatenating the fragmented data strings found across the DNS query subdomains yields a continuous encoded block:

```text
G4KVCVANDAYBKS2TLETCOASRKYGRAMBPKJKTIGZGANBVSBI6FY

```

Using **CyberChef**, the payload is processed through a two-stage recipe based on the discoveries from the HTTP configuration file:

1. **From Base32** (to decode the raw byte stream)
2. **XOR** (using the key `Sp00ky` identified in the staging config)



### Output / Flag:

```text
deadface{c2_traffic_be_bussing}

```
