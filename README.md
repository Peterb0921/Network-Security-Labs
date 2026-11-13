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

---

### 🔹 Lab 03: Linux Infrastructure & Database Hardening (Ubuntu & MySQL)

* **Date:** November 3, 2026
* **Target:** `Localhost` (Ubuntu Virtual Machine)
* **Tools Used:** Oracle VirtualBox, Ubuntu Linux, Apache, UFW (Uncomplicated Firewall), MySQL

#### Steps

* I configured an Ubuntu virtual machine and installed the Apache web server, setting up `mod_userdir` to serve web content directly from local user directories.
* I applied host-level access control using Uncomplicated Firewall (UFW). I restricted inbound traffic strictly to essential web services (ports 80 and 443) and blocked all other unvetted connections.
* After deploying a MySQL database, I ran the `mysql_secure_installation` script to establish a secure baseline. This involved enforcing password complexity, disabling remote root logins, and removing anonymous test accounts.

#### Why It Matters

* **Attack Surface Reduction:** Default web and database configurations often have known vulnerabilities. Restricting firewall ports and securing database access are critical first steps in reducing a system's attack surface.
* **Service Isolation:** Running both a database and a web server requires strict privilege separation. If the web application is compromised, the backend database must remain isolated to prevent a full system takeover.

#### What I Learned & The Human Element

* Using UFW provided a much more accessible way to audit active firewall rules compared to configuring raw `iptables`.
* **The Human Side:** This lab highlighted a common behavioral issue in IT: administrators sometimes disable firewalls or leave databases open to all IP addresses (`0.0.0.0`) for the sake of convenience during testing. This prioritization of speed over security often creates shadow IT and leaves massive vulnerabilities in production environments.

---

### 🔹 Lab 04: Administrative Control Panels & Database Management (Webmin)

* **Date:** November 4, 2026
* **Target:** `Localhost:10000` (Webmin Interface)
* **Tools Used:** Webmin, MySQL, UFW, Ubuntu Terminal, curl

#### Steps

* I installed the Webmin administrative panel via the command line and updated my UFW firewall rules to allow inbound TCP traffic on port 10000.
* To enforce role-based access control (RBAC), I created dedicated user groups within Webmin and assigned them specific module permissions based on the principle of least privilege.
* I used Webmin's graphical interface to build a new MySQL database (`CIS2229`), structuring relational tables with defined primary keys and auto-incrementing values.

#### Why It Matters

* **Centralized Management Risks:** Web-based control panels streamline server management, but they represent high-value targets. If the panel is compromised, an attacker gains root-level access to the server directly from their browser.
* **Access Control:** Implementing strict user groups ensures that junior administrators only have access to the specific modules they need to do their jobs.

#### What I Learned & The Human Element

* Webmin significantly speeds up database management and service configurations by abstracting complex command-line tasks into a visual interface.
* **The Human Side:** Exposing administrative interfaces to the open web without additional protections like a VPN or Multi-Factor Authentication assumes the login page won't be targeted. It's a clear example of how seeking operational efficiency can inadvertently expand the attack surface if human convenience overrides secure architecture.

---

### 🔹 Lab 05: Network Traffic Analysis & Packet Sniffing (Wireshark)

* **Date:** November 5, 2026
* **Target:** Pre-captured Network Traffic Files (`.pcap`)
* **Tools Used:** Wireshark

#### Steps

* I analyzed several raw network packet captures (`.pcap`) in Wireshark to inspect HTTP, Telnet, SMTP, DNS, and DHCP traffic.
* By filtering the HTTP traffic, I reviewed client requests, server status codes, and browser metadata (user-agents).
* I utilized the "Follow TCP Stream" feature to reconstruct the full communication between a client and a Telnet server.
* I dissected SMTP email traffic to identify mail server hostnames, sender and receiver addresses, and email attachments.

#### Why It Matters

* **Protocol Dissection:** Understanding how data is encapsulated across different network layers is essential for isolating anomalies and understanding network architecture.
* **Incident Response:** Packet captures serve as the ground truth for network activity. If a breach occurs, analyzing the `.pcap` files allows security analysts to see exactly what data was exfiltrated.

#### What I Learned & The Human Element

* Inspecting legacy protocols like Telnet and basic SMTP revealed how dangerous unencrypted communication can be. Using the TCP Stream feature exposed usernames and passwords in cleartext immediately.
* **The Human Side:** Because people frequently reuse passwords across platforms, one intercepted plaintext login on an old, unencrypted system can easily lead to compromised accounts on highly secure platforms. Human memory limitations drive password reuse, making legacy protocols a massive security risk.
