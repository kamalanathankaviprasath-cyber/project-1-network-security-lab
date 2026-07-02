<img width="2720" height="2320" alt="project1_network_diagram" src="https://github.com/user-attachments/assets/b538bb24-b104-495e-982a-458fefd99e3a" /># project-1-network-security-lab
Home lab network built with pfSense, Ubuntu Server and Windows 10 VMs — includes hardening, Nmap scanning and before/after evidence


# 🔐 Project 1 — Home Network Security Lab & Hardening

## Overview
Built a segmented virtual network simulating an enterprise 
environment using pfSense, Ubuntu Server, and Windows 10 VMs 
on VirtualBox. Conducted a full security audit using Nmap, 
identified real vulnerabilities in the default configuration, 
and applied structured hardening controls.

## Architecture & Documentation

This project includes an interactive network diagram built with 
SVG illustrating all five roles pfSense plays simultaneously:

- Gatekeeper (edge firewall)
- DHCP server (IP assignment)
- Traffic filter (allow/block rules)
- DNS resolver (name resolution)
- NAT router (internet sharing + privacy)

Full architecture walkthrough documented with before/after 
Nmap evidence showing measurable attack surface reduction.

## Network Architecture
![Network Diagram](screenshots/network_diagram.png)
