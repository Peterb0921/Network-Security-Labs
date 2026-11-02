# 🛡️ Network Security & Systems Defense Labs

Hands-on technical labs documenting network traffic inspection, attack surface reconnaissance, and defensive systems auditing.

---

### 🔹 Lab 01: Inspecting Plaintext HTTP Traffic (info.cern.ch)
* **Date:** November 1, 2026
* **Target:** `http://info.cern.ch/` (HTTP over Port 80)
* **Tools Used:** Wireshark, macOS Terminal, Firefox

#### Steps
* Initialized a live packet capture on active network interface using Wireshark.
* Navigated to `http://info.cern.ch/` (runs over unencrypted HTTP on port 80).
* Applied an `http` display filter to isolate packet `147` containing the `GET / HTTP/1.1` request.
* Used **Follow -> TCP Stream** to reconstruct the entire communication stream between client and CERN web server.

#### Wireshark Capture
![Wireshark TCP stream showing plaintext HTTP traffic from CERN](./wireshark%20cern%20traffic.png)

#### Why It Matters
* Demonstrates what unencrypted web traffic looks like to anyone on the same network segment running a passive packet sniffer.
* No private key or certificate needed to read page headers, server type (`Apache`), and raw HTML (`<h1>http://info.cern.ch - home of the first website</h1>`). Everything traveled over the physical wire in cleartext.

#### What I Learned & The Human Element
* In Wireshark's TCP stream window, **red text** displays client requests, and **blue text** displays server responses.
* The "Follow TCP Stream" feature strips away lower-layer transport overhead to expose the exact application payload crossing the wire.
* If a site doesn't enforce HTTPS, sensitive data (credentials, session tokens, form inputs) is fully exposed to network sniffers.
* **The Human Side:** Users rarely inspect URLs or transport security before clicking. Security architecture cannot rely on human caution alone; networks and browsers must enforce automated redirects and HSTS (HTTP Strict Transport Security) to protect users proactively.

---

### 🔹 Lab 02: Network Reconnaissance & Service Fingerprinting (Nmap)
* **Date:** November 2, 2026
* **Target:** `scanme.nmap.org` (`45.33.32.156`) — Authorized testing host
* **Tools Used:** Nmap v7.9x, Windows Terminal / PowerShell

#### Steps
* Executed a fast discovery scan (`nmap -F scanme.nmap.org`) across top 100 ports, discovering open ports `22/tcp` (SSH) and `80/tcp` (HTTP).
* Executed an aggressive scan (`nmap -A -p 22,80 scanme.nmap.org`) targeting open ports to trigger OS stack detection, banner grabbing, and Nmap Scripting Engine (NSE) audits.
* Extracted active public host keys from OpenSSH and audited HTTP response headers without launching a browser.

#### Scan Breakdown
![Nmap aggressive scan showing open ports, SSH keys, and Apache version](./nmap%20scanme.png)

#### Key Findings & Defense Context
* **Service Exposure & Version Fingerprinting:**
  * Port 22 exposes `OpenSSH 6.6.1p1` and Port 80 exposes `Apache httpd 2.4.7` on Ubuntu Linux.
  * Verbose service banners allow adversaries to cross-reference software releases with known public CVEs (e.g., via NIST NVD).
* **Cryptographic Weakness Audit:**
  * Discovered an active `1024-bit DSA` host key.
  * Deprecated under NIST SP 800-131A and DISA STIG guidelines due to low bit length and collision susceptibility; requires migration to modern algorithms like Ed25519 or ECDSA.
* **Operating System Fingerprinting:**
  * TCP/IP stack behavior analysis allowed Nmap to accurately guess the target OS family (Linux kernel 4.x–6.x at ~95% confidence).

#### What I Learned & The Human Element
* **Attack Surface Minimization:** Every open port is an entry point. Enforcing strict firewall boundaries and restricting administrative access (like SSH) exclusively behind VPNs drastically reduces an organization's attack surface.
* **Banner Suppression:** Production web servers should disable verbose headers (e.g., configuring `ServerTokens Prod` and `ServerSignature Off` in Apache) to deny automated scanners free reconnaissance data.
* **The Human Side:** Deprecated ciphers, weak keys, and verbose defaults often persist because "they work" and maintain backward compatibility. Robust security requires proactive configuration audits rather than assuming default settings are secure.
