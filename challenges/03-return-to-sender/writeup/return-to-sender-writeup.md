
# CTF Write-Up: Return to Sender

##  Challenge Overview

* **Category:** Network Forensics / C2 Traffic Analysis
* **Points:** 25
* **Tools Used:** Wireshark, Linux Terminal (`base64`)


##  Step 1: Traffic Inspection & Anomaly Identification

Upon reviewing the packet capture, the majority of the traffic consists of legitimate, routine background HTTP interactions (such as `GET` requests for resources like `/style.css`, `/app.js`, and `/img/logo.png`, alongside generic `POST /submit` actions across benign hosts like `[www.youtube.com](https://www.youtube.com)`, `[www.nytimes.com](https://www.nytimes.com)`, and `www.wikipedia.org`).

However, filtering for HTTP `POST` requests uncovers a distinct pattern of suspicious outbound exfiltration traffic targeting an anomalous Command and Control (C2) endpoint: **`telemetry.deadface-cc.net`**.

<img width="1917" height="1079" alt="image" src="https://github.com/user-attachments/assets/1f0ffc4c-e443-4d7d-b452-61b0e048c618" />


### Key Observations from Anomalous Streams:

* **Host Target:** `telemetry.deadface-cc.net`
* **URI Path:** `/api/upload`
* **Header Signature:** Includes a persistent session header:
```http
X-Session-Id: YnVzaW5lc3N9

```


*(Decoding this `X-Session-Id` via Base64 yields `business}`, further confirming malicious C2 tagging).*
* **Transmission Scheme:** Payload data is split into sequentially numbered upload chunks (`?seq=1` through `?seq=5`).


##  Step 2: Extracting the Sequential Payloads

Inspecting the POST body across the five sequential HTTP upload requests reveals raw Base64-encoded string chunks:

| Sequence (`seq`) | URI Target | Base64 Chunk Payload |
| --- | --- | --- |
| **`seq=1`** | `/api/upload?seq=1` | `ZGVhZGZh` |
| **`seq=2`** | `/api/upload?seq=2` | `Y2V7aHR0` |
| **`seq=3`** | `/api/upload?seq=3` | `cF9leGZp` |
| **`seq=4`** | `/api/upload?seq=4` | `bF9zbmVh` |
| **`seq=5`** | `/api/upload?seq=5` | `a3lf` |

---

##  Step 3: Reconstructing & Decoding the Flag

By concatenating the extracted Base64 fragments in sequential order (`seq=1` $\rightarrow$ `seq=5`), we obtain the complete encoded string:

```text
ZGVhZGZhY2V7aHR0cF9leGZpbF9zbmVha3lf

```

Piping the reconstructed string into Base64 decoding reveals the underlying plaintext flag:

```bash
echo "ZGVhZGZhY2V7aHR0cF9leGZpbF9zbmVha3lf" | base64 -d

```

### Output / Flag:

```text
deadface{http_exfil_sneaky_

```

*(Note: Adding the decoded `X-Session-Id` session suffix `business}` yields the completed flag format: `deadface{http_exfil_sneaky_business}`)*.
