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

* Provisioned a virtualized Ubuntu Linux environment and installed the Apache web server, configuring `mod_userdir` to serve content from local user directories (`public_html`).
* Configured host-level access control using Uncomplicated Firewall (`ufw`), explicitly allowing inbound traffic solely for "Apache Full" (ports 80/443) and blocking unvetted services.
* Deployed a MySQL database server and executed the `mysql_secure_installation` script to establish a secure baseline.
* Hardened the database by enforcing password validation, disabling remote root logins, removing anonymous users, and dropping default testing databases.

#### Firewall & MySQL Configuration

#### Why It Matters

* **Attack Surface Reduction:** Default installations of web and database servers are notoriously insecure. Hardening steps like restricting firewall ports and stripping anonymous database access are mandatory first lines of defense against automated exploitation.
* **Service Isolation:** Running databases and web servers requires strict privilege separation so that a compromise in the web application does not automatically grant root access to the backend database.

#### What I Learned & The Human Element

* Using `ufw` provides a much simpler abstraction layer over `iptables`, making it easier to audit open ports and active rules (`sudo ufw status`).
* The `mysql_secure_installation` tool highlights how many glaring vulnerabilities exist in a default SQL deployment if not actively managed.
* **The Human Side:** Developers and system administrators often disable firewalls entirely or leave databases open to `0.0.0.0` (all IP addresses) just to "make it work" quickly during testing. This prioritization of convenience over configuration creates shadow IT and massive network vulnerabilities when those systems inevitably end up in production.

---

### 🔹 Lab 04: Administrative Control Panels & Database Management (Webmin)

* **Date:** November 4, 2026
* **Target:** `Localhost:10000` (Webmin Interface)
* **Tools Used:** Webmin, MySQL, UFW, Ubuntu Terminal, curl

#### Steps

* Downloaded and installed the Webmin repository setup script via `curl` and integrated it into the Ubuntu package manager (`apt`).
* Updated `ufw` firewall rules to allow inbound TCP traffic on port 10000 to expose the Webmin administrative GUI.
* Created dedicated Webmin user groups and assigned specific module permissions to enforce role-based access control (RBAC) for new administrative users.
* Leveraged the Webmin GUI to architect a new MySQL database (`CIS2229`) and structured relational tables with defined primary keys and auto-increment schemas.

#### Webmin Database Management

#### Why It Matters

* **Centralized Management Risks:** Web-based control panels like Webmin are incredibly powerful but represent high-value targets for threat actors. If compromised, the attacker gains full root-level control over the server infrastructure through a browser.
* **Access Control:** Implementing strict user groups ensures that junior administrators only have access to the modules they need, adhering to the principle of least privilege.

#### What I Learned & The Human Element

* Webmin abstracts complex CLI configurations into a graphical interface, making database management, user creation, and service restarting significantly faster.
* Running commands with static outputs (like `ping -c 10 www.yahoo.com`) via a web-based command shell demonstrates how web panels execute backend system commands.
* **The Human Side:** IT professionals love administrative panels because they reduce friction and cognitive load. However, exposing administrative interfaces on open ports (like 10000) without VPN restrictions or Multi-Factor Authentication relies on the dangerous assumption that the login page won't be brute-forced or exploited by a zero-day vulnerability.

---

### 🔹 Lab 05: Network Traffic Analysis & Packet Sniffing (Wireshark)

* **Date:** November 5, 2026
* **Target:** Pre-captured Network Traffic Files (`.pcap`)
* **Tools Used:** Wireshark

#### Steps

* Loaded raw `.pcap` files into Wireshark to analyze diverse network traffic, including HTTP, Telnet, SMTP, DNS, and DHCP.
* Filtered HTTP traffic to dissect client requests, identify requested endpoints, review server status codes, and extract user-agent metadata (browser and OS footprints).
* Utilized the "Follow TCP Stream" feature on a Telnet packet capture to fully reconstruct the communication between client and server.
* Dissected SMTP email traffic streams to identify the hostname of the mail server, the sender/receiver email addresses, and the names of attached files.

#### Wireshark Protocol Dissection

#### Key Findings & Defense Context

* **Protocol Dissection:** Understanding encapsulation—how a TCP segment sits inside an IP packet, which sits inside an Ethernet frame—is essential for isolating where a network anomaly is occurring.
* **Incident Response:** Packet captures are the ground truth in cybersecurity. If a system is compromised, analyzing the `.pcap` allows a security analyst to see exactly what data was exfiltrated and what commands were executed.

#### What I Learned & The Human Element

* Legacy protocols like Telnet and basic SMTP offer absolutely no encryption. Using the "Follow TCP Stream" feature instantly exposes the user's username and password in perfect cleartext.
* Wireshark acts as an X-ray for network traffic, stripping away protocol overhead to reveal the exact application payload being transmitted over the wire.
* **The Human Side:** People are habitual and frequently reuse passwords across multiple services. If a user logs into a legacy Telnet server or an unencrypted SMTP relay, a passive network sniffer immediately captures those credentials. The human tendency toward password reuse means that one exposed plaintext login can compromise their entire digital identity across highly secure platforms as well.
