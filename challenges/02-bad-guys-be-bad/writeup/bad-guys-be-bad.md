# CTF Write-Up: Bad Guys Be Bad

## Challenge Overview

* **Category:** Network Forensics / C2 Traffic Analysis
* **Points:** 350
* **Objective:** Analyze the provided PCAP file (`deadface_pcap02.pcap`), identify Command and Control (C2) staging communications, track data exfiltration channels, and decode the final flag.
* **Tools Used:** Wireshark, CyberChef

##  Step 1: Initial Traffic Triage & Protocol Hierarchy
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/66258ac1-9b06-41ce-973d-ef53291b30e1" />

Upon opening `deadface_pcap02.pcap` in Wireshark, the packet capture reveals a substantial volume of traffic.

* To gain a high-level overview of the communication structure, navigate to **Statistics > Protocol Hierarchy Statistics**.
<img width="1535" height="852" alt="image" src="https://github.com/user-attachments/assets/f280370d-ecdf-4359-9c9e-c8e28e1a03c2" />


* The hierarchy highlights significant traffic distributed across UDP (DNS) and TCP (HTTP). Given that unencrypted protocols often hold clues or configuration files, the investigation begins by filtering for cleartext HTTP traffic.
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/4b0eec25-10b8-40f4-889d-2890201f6e0c" />

##  Step 2: Filtering HTTP stream of Suspicious Requests
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/4aac1324-a5ae-4202-9b2a-78bfadf04702" />
Filtering for HTTP traffic reveals multiple web requests and responses. Following the HTTP stream of suspicious requests I noticed that one packet has different length so I followed that stream.

<img width="1277" height="530" alt="image" src="https://github.com/user-attachments/assets/298918d8-de5e-40d9-b954-89e44b4f0721" />
Initially, the User-Agent header featured a suspicious Base64-encoded string:

```text
bmljZSB0cnkgLSB3cm9uZyB0aHJlYWQsIGtlZXAgZGlnZ2luZw==

```

Decoding this first might look like a promising flag candidate, but it turns out to be a decoy/distraction because it says:

```text
"nice try - wrong thread, keep digging"

```
Looking further down into the server's response payload uncovers the actual staging configuration:


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

<img width="1919" height="1078" alt="image" src="https://github.com/user-attachments/assets/e04dc792-5311-47af-842f-759052b7259c" />


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

<img width="1605" height="658" alt="image" src="https://github.com/user-attachments/assets/931f2042-a2cd-445a-a6f2-150cded68fc0" />


### Output / Flag:

```text
deadface{c2_traffic_be_bussing}

```
