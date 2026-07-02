Internet (NAT)
│
pfSense VM (Firewall/Router)
WAN: 10.0.2.15 | LAN: 192.168.1.1
│
LabNet (192.168.1.0/24)
├── Ubuntu Server (192.168.1.100)
└── Windows 10 VM (192.168.1.x)



## Tools Used
| Tool | Version | Purpose |
|---|---|---|
| VirtualBox | 7.x | Hypervisor |
| pfSense | 2.6.0 | Firewall/Router VM |
| Nmap | 7.94 | Network scanning |
| Ubuntu Server | 22.04 | Internal server VM |
| Windows 10 | — | Internal client VM |

## Security Findings (Before Hardening)

| Finding | Severity | Details |
|---|---|---|
| Default credentials | 🔴 Critical | admin/pfsense publicly known |
| HTTP admin access (port 80) | 🟠 Medium | Credentials sent in plaintext |
| IPv6 exposed on WAN | 🟡 Low | Unnecessary attack surface |
| OS fingerprint confidence 97% | 🟡 Low | Easy OS identification |

## Hardening Steps Applied

1. ✅ Changed default admin credentials
2. ✅ Forced HTTPS-only — disabled plain HTTP
3. ✅ Added firewall rule blocking HTTP pass-through
4. ✅ Disabled IPv6 — removed unnecessary protocol
5. ✅ Confirmed SSH disabled on firewall

## Results (After Hardening)

| Finding | Before | After |
|---|---|---|
| Default credentials | 🔴 Exposed | ✅ Changed |
| Port 80 (HTTP) | 🟠 Serving plaintext | ✅ HTTPS redirect only |
| IPv6 on WAN | 🟡 Visible | ✅ Eliminated |
| OS fingerprint | 🟡 97% confidence | ✅ Reduced to 91% |

## Nmap Scan Evidence
See [/scans](./scans) folder for full before and after scan output files.

### Before Hardening
![Before Scan](screenshots/before_scan.png)

### After Hardening
![After Scan](screenshots/after_scan.png)

## Key Lessons Learned
See [lessons_learned.md](./lessons_learned.md) for all 10 
documented lessons covering sudo privileges, defense in depth, 
firewall rule logic, VM security, and more.

## Skills Demonstrated
- Network segmentation and VLAN concepts
- Firewall configuration (pfSense)
- Vulnerability identification via Nmap
- Security hardening using CIS benchmark principles
- Defense in depth strategy
- Linux command line (Ubuntu Server)
- Documentation and evidence collection

## What's Next
This is Project 1 of an 11-project cybersecurity portfolio.
Project 2: SIEM deployment and log analysis with Wazuh.
