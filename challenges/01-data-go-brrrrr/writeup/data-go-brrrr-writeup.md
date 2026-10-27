# CTF Write-Up: Data Go Brrrrr

### Challenge Overview

Category: Network Forensics / Traffic Analysis

Artifact: PCAP01.pcap

Objective: Analyze the network capture, identify anomalous traffic, extract hidden data, and decode the flag.

Tools Used: Wireshark, Linux Terminal (base64)

Step 1: Initial PCAP Inspection

Upon opening PCAP01.pcap in Wireshark, the packet stream displays a mix of standard network traffic (local ARP/ICMP, DNS queries, and HTTP requests).

However, a closer look at the packet list reveals an anomaly within the ICMP Echo (ping) traffic between host 192.168.1.50 and 10.10.14.7:

Several ICMP Echo requests are explicitly flagged by Wireshark with (no response found!).

Unlike legitimate ping requests that receive matching Echo replies, these specific packets lack corresponding responses, strongly suggesting a covert channel or data exfiltration attempt over ICMP.

Step 2: Data Extraction & Reconstruction

Inspecting the packet details for the flagged ICMP Echo requests reveals custom data payloads embedded within the Data field rather than standard OS ping payload patterns.

By systematically inspecting the data payloads of all packets marked with (no response found!) and concatenating them sequentially, the following Base64-encoded payload was reconstructed:

ZGVhZGZhY2V7aWNtcF8zeGZpbF93aDNuX3RoM3lfbGVhc3Rfc3VzcDNjdH0=

Step 3: Decoding the Flag

The presence of trailing = padding confirms that the extracted string is Base64 encoded. Decoding it using the Linux terminal yields the plain-text flag:

echo "ZGVhZGZhY2V7aWNtcF8zeGZpbF93aDNuX3RoM3lfbGVhc3Rfc3VzcDNjdH0=" | base64 -d

Captured Flag

deadface{icmp_3xfil_wh3n_th3y_least_susp3ct}

Key Takeaways

Covert Channels: Attackers frequently leverage standard protocols like ICMP or DNS to bypass egress filtering and exfiltrate sensitive data.

Detection: Unmatched ICMP Echo requests with varying or non-standard payload data are a primary Indicator of Compromise (IoC) for ICMP tunneling.
