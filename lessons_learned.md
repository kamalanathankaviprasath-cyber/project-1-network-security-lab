# Lessons Learned — Project 1: Home Network Security Lab & Hardening

## Overview
This document captures all key lessons learned during the 
build, audit, and hardening of a virtual enterprise-style 
network using pfSense, Ubuntu Server, and Windows 10 VMs.
These lessons reflect real mistakes made, problems solved,
and concepts discovered during hands-on work — not theory.

---

## Lesson 1: Security Tools Often Require Elevated Privileges

**What happened:**
Running `nmap -sV -O -p- 192.168.1.1` failed with:
"TCP/IP fingerprinting requires root privileges. QUITTING."

**Root cause:**
OS detection (-O flag) requires raw packet access, which 
Linux restricts to root-level users for security reasons.

**Fix applied:**
Added `sudo` prefix: `sudo nmap -sV -O -p- 192.168.1.1`

**Key takeaway:**
Always check privilege requirements before running security 
tools. Understanding WHY a tool needs root (raw socket 
access for packet crafting) is more valuable than just 
memorising to add sudo. This applies to Wireshark, 
tcpdump, and many other security tools.

---

## Lesson 2: Default Credentials Are a Critical Vulnerability

**What happened:**
pfSense shipped with default credentials admin/pfsense — 
publicly documented and the first thing any attacker tries.

**Risk:**
Any device on LabNet could access the firewall admin panel 
using publicly known credentials. This is the most common 
cause of firewall compromise in real organisations.

**Fix applied:**
Changed admin password immediately during hardening step 1,
before any other configuration changes.

**Key takeaway:**
Changing default credentials is always Priority 1 on any 
network device. This is explicitly required by CIS 
Benchmarks and is a finding in virtually every real-world 
penetration test. Default credentials are not just a 
beginner mistake — they cause real enterprise breaches.

---

## Lesson 3: HTTP vs HTTPS on Admin Interfaces

**What happened:**
Nmap scan revealed port 80 (HTTP) open on pfSense by 
default, serving the admin dashboard over unencrypted 
connection.

**Risk:**
Any device on LabNet could use Wireshark or a similar 
tool to intercept admin login credentials in plain text 
— a classic man-in-the-middle attack.

**Fix applied:**
Changed webConfigurator protocol from HTTP to HTTPS in
System → Advanced → Admin Access. Port 443 now serves
the dashboard with full TLS encryption.

**Key takeaway:**
Administrative interfaces must always use encrypted 
connections. Plain HTTP transmits credentials in 
cleartext — visible to anyone on the same network 
segment. In a real bank environment, this finding would 
be rated CRITICAL severity.

---

## Lesson 4: Console Access Is Your Recovery Lifeline

**What happened:**
After changing the pfSense admin password, login failed.
The new password was not working on the web dashboard.
Locked out completely.

**Recovery method:**
Used pfSense console menu option 3 (Reset webConfigurator 
password) and option 8 (Shell → passwd admin command) to 
reset the password directly from the VM console without 
losing configuration.

**Key takeaway:**
Physical/console access to a network device is the last 
resort recovery method — and intentionally so. The fact 
that password reset requires console access (not remote 
access) is a security feature, not a limitation. In a 
real company, only physically present authorised staff 
can recover a locked firewall. Remote attackers cannot 
reset credentials this way. Always document credentials 
securely when first configuring any device.

---

## Lesson 5: Reading and Interpreting Nmap Output

**What happened:**
Nmap warned "OSScan results may be unreliable" despite 
correctly identifying pfSense as FreeBSD with 91-97% 
confidence.

**Root cause:**
OS fingerprinting works by comparing responses from open 
AND closed ports. pfSense had 65,532 filtered ports 
(no response) and only 3 open ports — insufficient 
variety for high-confidence OS detection.

**Key takeaway:**
Understanding scan limitations is as important as 
understanding scan results. A lower confidence percentage 
does not mean the scan failed — it means the tool is 
being honest about uncertainty. Real security engineers 
correlate multiple data sources rather than trusting 
any single scan result completely.

---

## Lesson 6: Every Open Port Exists for a Reason

**What happened:**
Before hardening, Nmap showed ports 53, 80, and 443 open.
Initial reaction was "these should all be closed."

**Analysis:**
- Port 53 (DNS/Unbound): Required for VMs to resolve 
  domain names. Closing it breaks all internet access.
- Port 80 (HTTP/nginx): Legacy convenience for dashboard 
  access. Creates credential interception risk.
- Port 443 (HTTPS/nginx): Secure dashboard access. 
  Must remain open.

**Key takeaway:**
Security engineering means understanding the business 
and technical reason for each service before deciding 
to close it. Blindly closing ports breaks functionality. 
The correct approach: identify what each port does, 
assess the risk, and either accept, mitigate, or 
remediate — never just close without understanding.

---

## Lesson 7: Port Behaviour vs Port Closure Are Different

**What happened:**
After disabling HTTP in pfSense settings, port 80 still 
appeared open in Nmap. Expected it to disappear 
completely.

**Root cause:**
pfSense keeps port 80 open to serve an HTTPS redirect 
(HTTP 301 Moved Permanently → https://192.168.1.1). 
The behaviour behind port 80 changed fundamentally 
even though the port remained open.

**Verification:**
curl -v http://192.168.1.1:80 confirmed:
"HTTP/1.1 301 Moved Permanently → Location: 
https://192.168.1.1/"

**Key takeaway:**
Closing a port and securing a port are different 
security controls. A port serving an HTTPS redirect 
is meaningfully safer than one serving plain HTTP, 
even if both show as "open" in a scan. Context and 
behaviour matter more than port state alone.

---

## Lesson 8: Firewall Rules vs Local Service Rules

**What happened:**
Added a LAN firewall rule to block port 80. Port 80 
still responded from pfSense itself.

**Root cause:**
Firewall rules in pfSense control traffic PASSING 
THROUGH the firewall. Traffic destined FOR pfSense 
itself (192.168.1.1) is handled by local service 
configuration, not the firewall rule engine.

**Fix applied:**
Two-layer approach:
Layer 1 — webConfigurator set to HTTPS only (controls 
local service behaviour)
Layer 2 — Firewall rule added blocking HTTP pass-through 
(controls traffic through the firewall)

**Key takeaway:**
This concept applies to all network devices, not just 
pfSense. Firewall rules and service configuration are 
separate security controls that protect different 
traffic paths. Defense in depth means applying both.

---

## Lesson 9: VM Features Create Security Channels

**What happened:**
Needed to enable shared clipboard between Ubuntu VM and 
Windows host to copy terminal output. Questioned whether 
this was safe.

**Analysis:**
Shared clipboard, shared folders, and network bridging 
all create communication channels between the VM and 
host machine.

**Risk assessment:**
- Clean lab VMs (our situation): Low risk, acceptable
- Malware analysis VMs: Never enable — malware can 
  read clipboard data including passwords
- Exploit testing VMs: Disable all sharing features

**Key takeaway:**
Always evaluate the risk of VM features based on what 
is running inside the VM — not just convenience. In 
a malware sandbox, shared clipboard is a serious 
attack vector. In a clean lab it is a minor 
convenience feature. The threat model determines 
the answer, not a fixed rule.

---

## Lesson 10: Security Hardening Is About Layers, Not Perfection

**What happened:**
After hardening, port 80 still showed open in Nmap. 
A beginner might conclude "hardening failed."

**Reality:**
Multiple meaningful security improvements were achieved:
- Default credentials eliminated
- HTTP admin access changed to HTTPS redirect
- Firewall rule added for HTTP pass-through blocking
- IPv6 attack surface eliminated
- OS fingerprint confidence reduced 97% → 91%
- SSH confirmed disabled

**Key takeaway:**
No system is perfectly secure. The goal of security 
hardening is to make attacks harder, slower, and more 
detectable — not to achieve zero findings. Every 
control added increases the cost and effort for an 
attacker. This is the core principle behind defense 
in depth and is how real security teams measure 
success.

---

## Lesson 11: Always Sanitize Config Files Before Publishing

**What happened:**
pfSense configuration XML exported for GitHub contained:
- sha512-hash (password hash — crackable offline)
- SSL certificate (server identity data)
- Private key (most critical — decrypts HTTPS traffic)

**Fix applied:**
All three replaced with REDACTED placeholder before 
uploading. Sanitized version uploaded to GitHub.

**Key takeaway:**
Config files contain secrets hidden in plain sight. 
A private key is the most dangerous — it can decrypt 
all HTTPS traffic to your firewall. Real security 
engineers use tools like git-secrets, GitHub secret 
scanning, and .gitignore to prevent accidental 
exposure. The rule: when in doubt, redact it.

---

## Lesson 12: Evidence Collection Is Part of Security Work

**What happened:**
Documented before/after Nmap scans, firewall rule 
screenshots, and configuration changes throughout 
the project.

**Why this matters:**
In a real company this becomes your audit trail — 
proof that controls were implemented, tested, and 
verified. Regulators (PCI-DSS, ISO 27001, SOC 2) 
require documented evidence of security controls, 
not just the controls themselves.

**Key takeaway:**
Security without documentation is just activity. 
Security with documentation is a defensible control. 
Get into the habit of capturing before/after states, 
tool outputs, and reasoning for every security 
decision — this habit alone separates junior 
engineers from senior ones.

---

## Summary Table

| # | Lesson | Category |
|---|---|---|
| 1 | Tools need root privileges | Linux fundamentals |
| 2 | Default credentials = critical risk | Hardening |
| 3 | HTTP exposes credentials in plaintext | Encryption |
| 4 | Console access is recovery lifeline | Operations |
| 5 | Read Nmap output with context | Reconnaissance |
| 6 | Every port exists for a reason | Risk analysis |
| 7 | Port behaviour vs port closure differ | Firewall concepts |
| 8 | Firewall rules vs local service rules | Network security |
| 9 | VM features create security channels | Lab security |
| 10 | Hardening is layers, not perfection | Defense in depth |
| 11 | Sanitize configs before publishing | Secure practices |
| 12 | Evidence collection is essential | Documentation |

---

*Project 1 of 11 — Cybersecurity Portfolio*
*Tools: pfSense 2.6.0 | Nmap 7.94 | VirtualBox | 
Ubuntu Server 22.04 | Windows 10*
*Framework reference: CIS Benchmarks, NIST 800-53*
