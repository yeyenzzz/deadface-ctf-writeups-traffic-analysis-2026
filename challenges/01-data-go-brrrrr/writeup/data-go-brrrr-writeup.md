# CTF Write-Up: Data Go Brrrrr

## Challenge Overview

* **Category:** Network Forensics / Traffic Analysis
* **Points:** 25
* **Tools Used:** Wireshark, CyberChef

## Step 1: Initial PCAP Inspection

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/8d7c2a5e-90de-43f8-801b-349721ee42ab" />

Upon opening PCAP01.pcap in Wireshark, the packet stream displays a mix of standard network traffic (local ARP/ICMP, DNS queries, and HTTP requests). However, a closer look at the packet list reveals an anomaly within the ICMP Echo (ping) traffic between host 192.168.1.50 and 10.10.14.7:

 * Several ICMP Echo requests are explicitly flagged by Wireshark with (no response found!).

 * Unlike legitimate ping requests that receive matching Echo replies, these specific packets lack corresponding responses, strongly suggesting a covert channel or data exfiltration attempt over ICMP.

## Step 2: Data Extraction & Reconstruction

<img width="1661" height="1079" alt="image" src="https://github.com/user-attachments/assets/57519f41-4074-43b9-beb8-fcad0b521892" />

Inspecting the packet details for the flagged ICMP Echo requests reveals custom data payloads embedded within the Data field rather than standard OS ping payload patterns.

By systematically inspecting the data payloads of all packets marked with (no response found!) and concatenating them sequentially, the following Base64-encoded payload was reconstructed:

```text
ZGVhZGZhY2V7aWNtcF8zeGZpbF93aDNuX3RoM3lfbGVhc3Rfc3VzcDNjdH0=

```

## Step 3: Decoding the Flag

The presence of trailing = padding confirms that the extracted string is Base64 encoded. Decoding it using the Linux terminal yields the plain-text flag:

```bash
echo "ZGVhZGZhY2V7aWNtcF8zeGZpbF93aDNuX3RoM3lfbGVhc3Rfc3VzcDNjdH0=" | base64 -d

```

Captured Flag

deadface{icmp_3xfil_wh3n_th3y_least_susp3ct}
