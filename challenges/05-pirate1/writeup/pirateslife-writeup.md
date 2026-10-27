# CTF Write-Up: Pirates Life (DEADFACE CTF)

##  Challenge Overview

* **Category:** Network Forensics / HTTP Header Analysis
* **Points:** 25
* **Tools Used:** Wireshark

## Step 1: Filtering the Traffic
Upon opening the packet capture, I noticed there were tons of packets. To narrow down the investigation, I decided to filtered out the protocols that have plain text or I can have clues. I started with HTTP.
<img width="1908" height="1079" alt="image" src="https://github.com/user-attachments/assets/da1c04b1-7661-4b2b-ab08-218f66ddcb85" />


## Step 2: Discovering Custom Headers:
While analyzing the filtered HTTP streams, I found suspicious custom tracking headers (`X-1-Trace-Part`, `X-2-Mark`, and `X-3-Spot`) embedded in the request headers:


* `X-1-Trace-Part`: `deadface{X-M4rk5_`


* `X-2-Mark`: `th3_sp0t_`


* `X-3-Spot`: `I_gu355}`

## Step 3: Reconstructing the Flag:
By combining the values from these custom headers in order, I successfully assembled the complete flag:
```text
deadface{X-M4rk5_th3_sp0t_I_gu355}

```
