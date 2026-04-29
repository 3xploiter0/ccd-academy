# 🛡️ DIT Cyber Club — Cybersecurity Learning Path

A structured 12-week curriculum organized into **Beginner**, **Intermediate**, and **Advanced** tiers. Each week includes theory + hands-on labs to build practical skills progressively.

---

## Table of Contents

- [Prerequisites & Promotion Criteria](#prerequisites--promotion-criteria)
- [Environment Setup Guide](#environment-setup-guide)
- [Beginner Tier](#beginner-tier)
- [Intermediate Tier](#intermediate-tier)
- [Advanced Tier](#advanced-tier)
- [Checkpoints & Assessments](#checkpoints--assessments)
- [Mentor System](#mentor-system)
- [Career Path & Certifications](#career-path--certifications)
- [Reporting & Soft Skills](#reporting--soft-skills)
- [Quick Reference Cards](#quick-reference-cards)
- [Progress Tracker Template](#progress-tracker-template)
- [FAQ & Troubleshooting](#faq--troubleshooting)

## Prerequisites & Promotion Criteria

Before moving to the next tier, you **must** pass the checkpoint assessment for your current tier.

| From | To | Requirements |
|------|----|-------------|
| Beginner | Intermediate | Complete all 12 Beginner weeks + pass Week 12 Capstone (score ≥70%) |
| Intermediate | Advanced | Complete all 12 Intermediate weeks + pass Week 12 CTF with at least 3 flags found |
| Advanced | Alumni/Mentor | Complete final project and present to club + recommend by a current mentor |

Lacking prerequisites? Stay at your current tier and re-attempt checkpoint labs. There's no shame in reinforcing fundamentals.

---

## Environment Setup Guide

Get your lab environment ready **before** Week 1.

### Required Tools (All Tiers)

| Tool | Install Link | Purpose |
|------|-------------|---------|
| VirtualBox | https://www.virtualbox.org/ | Run VMs for labs |
| Kali Linux VM | https://www.kali.org/get-kali/#kali-virtual-machines | Offensive security toolkit |
| Burp Suite Community | https://portswigger.net/burp/communitydownload | Web proxy |
| Wireshark | https://www.wireshark.org/download.html | Packet analysis |
| VS Code | https://code.visualstudio.com/ | Code editor |
| Python 3 | https://www.python.org/downloads/ | Scripting |
| Git | https://git-scm.com/downloads | Version control |
| Bitwarden (optional) | https://bitwarden.com/ | Password manager |

### Common Setup Issues

| Issue | Solution |
|-------|----------|
| VirtualBox won't start VMs | Enable virtualization in BIOS (Intel VT-x / AMD-V) |
| WSL not working on Windows | Run `wsl --install` as Administrator in PowerShell |
| Burp Suite CA cert not trusted | Export CA cert → browser settings → Trusted Root Authorities |
| Python command not found | Use `python3` instead of `python` on Linux/Mac |

---

# Beginner Tier

> **Goal**: Build foundational knowledge of cybersecurity concepts, Linux, networking, and web basics.

## Week 1 — Introduction to Cybersecurity

| Topic | Details |
|-------|---------|
| **Theory** | What is cybersecurity? CIA Triad (Confidentiality, Integrity, Availability). Types of threats (malware, phishing, DDoS). Risk vs Threat vs Vulnerability. |
| **Hands-on** | Create accounts on TryHackMe and OverTheWire. Complete TryHackMe "Intro to Cybersecurity" room. |
| **Resources** | [TryHackMe — Intro to Cybersecurity](https://tryhackme.com/path/outline/introtocyber) |

## Week 2 — Cyber Hygiene & Security Basics

| Topic | Details |
|-------|---------|
| **Theory** | Password security (managers, policies). Multi-Factor Authentication (MFA). Social engineering awareness. Safe browsing habits. |
| **Hands-on** | Set up a password manager (Bitwarden). Complete TryHackMe "Cyber Security 101" room. |
| **Resources** | [Have I Been Pwned](https://haveibeenpwned.com/), [Bitwarden](https://bitwarden.com/) |

## Week 3 — Linux Fundamentals

| Topic | Details |
|-------|---------|
| **Theory** | Linux file system hierarchy. File permissions (chmod, chown). Users, groups, sudo. Common commands (ls, cd, grep, find, ps, netstat). |
| **Hands-on** | Complete TryHackMe "Linux Fundamentals" series (Parts 1-3). Practice on a local Ubuntu VM or WSL. |
| **Resources** | [Linux Journey](https://linuxjourney.com/), [OverTheWire Bandit](https://overthewire.org/wargames/bandit/) (Levels 0-9) |

## Week 4 — Networking Basics (Part 1)

| Topic | Details |
|-------|---------|
| **Theory** | OSI & TCP/IP models. IP addressing, subnetting. Ports and common services (22 SSH, 80 HTTP, 443 HTTPS). |
| **Hands-on** | Use `ping`, `traceroute`, `nslookup`, `curl`. Analyze traffic with Wireshark basics. Complete TryHackMe "Intro to Networking". |
| **Resources** | [Professor Messer — Network+](https://www.professormesser.com/network-plus/n10-008/) |

## Week 5 — Networking Basics (Part 2)

| Topic | Details |
|-------|---------|
| **Theory** | TCP vs UDP (3-way handshake). DNS resolution process. HTTP/HTTPS in detail (methods, headers, status codes). |
| **Hands-on** | Capture TCP handshake in Wireshark. Inspect HTTP headers with browser DevTools. Complete TryHackMe "HTTP in Detail". |
| **Resources** | [How DNS Works](https://howdns.works/) |

## Week 6 — Web Technologies & How Websites Work

| Topic | Details |
|-------|---------|
| **Theory** | Client-server architecture. HTML, CSS, JavaScript basics. Cookies, sessions, same-origin policy. REST APIs. |
| **Hands-on** | Inspect and modify requests with Burp Suite (Community). Complete TryHackMe "Web Fundamentals" path. |
| **Resources** | [Burp Suite Tutorial](https://portswigger.net/burp/documentation/desktop/getting-started) |

## Week 7 — Introduction to Cryptography

| Topic | Details |
|-------|---------|
| **Theory** | Symmetric vs asymmetric encryption. Hashing (MD5, SHA). Base64 encoding vs encryption. TLS/SSL. Certificates. |
| **Hands-on** | Generate keys with OpenSSL. Hash files with `sha256sum`. Decode Base64. Complete TryHackMe "Cryptography 101". |
| **Resources** | [CryptoHack](https://cryptohack.org/) (Beginner challenges) |

## Week 8 — Python Basics for Security

| Topic | Details |
|-------|---------|
| **Theory** | Variables, loops, conditionals, functions. File I/O. String manipulation. Using `requests` library. |
| **Hands-on** | Write a simple port scanner. Write a script to fetch HTTP headers. Automate a hash checker. |
| **Resources** | [Python for Everybody](https://www.py4e.com/), [Automate the Boring Stuff](https://automatetheboringstuff.com/) |

## Week 9 — Information Gathering & Reconnaissance

| Topic | Details |
|-------|---------|
| **Theory** | Passive vs active recon. OSINT techniques. WHOIS, DNS enumeration. Subdomain discovery. Google dorking. |
| **Hands-on** | Use `whois`, `nslookup`, `theHarvester`, `sublist3r`. Perform Google dorks. Complete TryHackMe "Passive Reconnaissance". |
| **Resources** | [OSINT Framework](https://osintframework.com/) |

## Week 10 — Scanning & Enumeration

| Topic | Details |
|-------|---------|
| **Theory** | Nmap scan types (SYN, TCP connect, UDP). Service/version detection. NSE scripts. |
| **Hands-on** | Scan a target (Metasploitable/HTB) with various nmap flags. Identify services and versions. Complete TryHackMe "Nmap" room. |
| **Resources** | [Nmap Network Scanning](https://nmap.org/book/) |

## Week 11 — Vulnerability Assessment Basics

| Topic | Details |
|-------|---------|
| **Theory** | What is a CVE? CVSS scoring. Vulnerability vs exploit. Vulnerability scanners (Nessus, OpenVAS). |
| **Hands-on** | Run OpenVAS/Greenbone against a lab VM. Review and interpret scan results. |
| **Resources** | [CVE Details](https://www.cvedetails.com/), [NVD](https://nvd.nist.gov/) |

## Week 12 — Capstone Challenge

| Topic | Details |
|-------|---------|
| **Theory** | Review all topics covered. Discussion on ethics and cyber law. |
| **Hands-on** | Complete TryHackMe "Vulnversity" or "Basic Pentesting" room end-to-end (recon → scanning → exploitation → privilege escalation). Write a penetration testing report. |
| **Resources** | [Pentesting Report Template](https://github.com/jazzpio/PT-report-template) |

---

# Intermediate Tier

> **Goal**: Develop offensive (web, network) and defensive (monitoring, hardening) skills.

## Week 1 — OWASP Top 10 Overview

| Topic | Details |
|-------|---------|
| **Theory** | Introduction to OWASP Top 10 (2021). Broken Access Control, Cryptographic Failures, Injection. |
| **Hands-on** | Complete TryHackMe "OWASP Top 10" room. |
| **Resources** | [OWASP Top 10](https://owasp.org/www-project-top-ten/) |

## Week 2 — SQL Injection (SQLi)

| Topic | Details |
|-------|---------|
| **Theory** | SQLi types (in-band, blind, out-of-band). UNION-based, error-based, time-based. Bypassing filters. |
| **Hands-on** | Complete PortSwigger SQLi labs. TryHackMe "SQL Injection" room. |
| **Resources** | [PortSwigger SQLi](https://portswigger.net/web-security/sql-injection) |

## Week 3 — Cross-Site Scripting (XSS)

| Topic | Details |
|-------|---------|
| **Theory** | Reflected, stored, DOM-based XSS. XSS payloads. Cookie stealing, keylogging. CSP bypass. |
| **Hands-on** | Complete PortSwigger XSS labs. TryHackMe "XSS" room. |
| **Resources** | [PortSwigger XSS](https://portswigger.net/web-security/cross-site-scripting) |

## Week 4 — Other Web Attacks

| Topic | Details |
|-------|---------|
| **Theory** | CSRF, SSRF, IDOR, File Inclusion (LFI/RFI), Command Injection. |
| **Hands-on** | Complete PortSwigger labs for each. TryHackMe "OWASP Juice Shop". |
| **Resources** | [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/) |

## Week 5 — Network Security & Firewalls

| Topic | Details |
|-------|---------|
| **Theory** | Firewall types (packet filter, stateful, application). IDS/IPS (Snort, Suricata). DMZ, VLANs. |
| **Hands-on** | Configure iptables/nftables rules. Set up Snort and write custom rules. |
| **Resources** | [Snort Documentation](https://www.snort.org/documents) |

## Week 6 — Wireshark & Packet Analysis

| Topic | Details |
|-------|---------|
| **Theory** | Capture filters vs display filters. Follow TCP streams. Protocol hierarchy. Expert info. |
| **Hands-on** | Analyze pcap files (Malware Traffic Analysis exercises). Extract files from packets. |
| **Resources** | [Wireshark Labs](https://www.wireshark.org/docs/wsug_html/), [Malware Traffic Analysis](https://www.malware-traffic-analysis.net/) |

## Week 7 — Cryptography in Practice

| Topic | Details |
|-------|---------|
| **Theory** | Digital signatures. PKI infrastructure. Certificates (x.509). GPG. SSH key authentication. |
| **Hands-on** | Generate GPG keys, encrypt/decrypt files. Set up SSH key auth. Analyze TLS certificates. |
| **Resources** | [CryptoHack](https://cryptohack.org/) (Intermediate challenges) |

## Week 8 — Python for Offensive Security

| Topic | Details |
|-------|---------|
| **Theory** | Socket programming. Subprocess & os modules. Writing custom scanners and brute-forcers. |
| **Hands-on** | Write a TCP client/server. Build a port scanner. Create a simple SSH brute-forcer (educational only). |
| **Resources** | [Black Hat Python](https://nostarch.com/blackhatpython) |

## Week 9 — Privilege Escalation (Linux)

| Topic | Details |
|-------|---------|
| **Theory** | SUID binaries. Kernel exploits. Cron jobs. Sudo misconfigurations. Capabilities. |
| **Hands-on** | Enumerate and escalate on Linux boxes (TryHackMe "Linux PrivEsc", HTB). |
| **Resources** | [GTFOBins](https://gtfobins.github.io/), [LinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) |

## Week 10 — Privilege Escalation (Windows)

| Topic | Details |
|-------|---------|
| **Theory** | Windows tokens, services, unquoted paths, DLL hijacking, AlwaysInstallElevated, UAC bypass. |
| **Hands-on** | Enumerate and escalate on Windows boxes (TryHackMe "Windows PrivEsc", HTB). |
| **Resources** | [WinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS), [LOLBAS](https://lolbas-project.github.io/) |

## Week 11 — Password Cracking & Hash Analysis

| Topic | Details |
|-------|---------|
| **Theory** | Hash types identification. Dictionary vs brute-force vs rainbow tables. Rules/mangling. |
| **Hands-on** | Use Hashcat/John the Ripper with wordlists (rockyou). Practice with TryHackMe "Crack the Hash" rooms. |
| **Resources** | [Hashcat Wiki](https://hashcat.net/wiki/), [SecLists](https://github.com/danielmiessler/SecLists) |

## Week 12 — CTF Competition & Debrief

| Topic | Details |
|-------|---------|
| **Theory** | CTF strategy, time management, team roles. |
| **Hands-on** | Participate in a CTF (PicoCTF, HTB CTF, or club-hosted). Debrief and share write-ups. |
| **Resources** | [CTFtime](https://ctftime.org/), [Write-up templates](https://github.com/ctf-wiki/ctf-wiki) |

---

# Advanced Tier

> **Goal**: Specialize in binary exploitation, malware analysis, red teaming, or blue team operations.

## Week 1 — Binary Exploitation Basics

| Topic | Details |
|-------|---------|
| **Theory** | Memory layout (stack, heap, BSS, text). Registers. Assembly basics (x86/x64). Buffer overflows. |
| **Hands-on** | Complete TryHackMe "Buffer Overflow Prep". Write a simple stack-based overflow. |
| **Resources** | [LiveOverflow](https://www.youtube.com/playlist?list=PLhixgUqwRTjxglIswKp9mpkfPNfHkzyeN), [Pwn.College](https://pwn.college/) |

## Week 2 — Advanced Exploitation (ROP & Shellcode)

| Topic | Details |
|-------|---------|
| **Theory** | ASLR, DEP/NX bypass. Return-Oriented Programming (ROP). Writing custom shellcode. |
| **Hands-on** | Use pwntools to craft ROP chains. Exploit with stack pivoting. Complete ROP Emporium challenges. |
| **Resources** | [ROP Emporium](https://ropemporium.com/), [pwntools docs](https://docs.pwntools.com/) |

## Week 3 — Web Application Firewall (WAF) Bypass & Advanced Web

| Topic | Details |
|-------|---------|
| **Theory** | WAF detection & fingerprinting. Bypass techniques (encoding, parameter pollution, HTTP verb tampering). |
| **Hands-on** | Use SQLmap with WAF bypass scripts. Bypass ModSecurity rules. Complete PortSwigger WAF bypass labs. |
| **Resources** | [SQLmap WAF bypass](https://github.com/sqlmapproject/sqlmap/wiki/Usage#bypass), [ModSecurity](https://modsecurity.org/) |

## Week 4 — Malware Analysis (Static)

| Topic | Details |
|-------|---------|
| **Theory** | PE/ELF structure. Strings analysis. Import/export tables. VirusTotal. YARA rules. |
| **Hands-on** | Analyze malware samples (safe sources: MalwareBazaar). Write YARA rules. Use pestudio/Detect It Easy. |
| **Resources** | [MalwareBazaar](https://bazaar.abuse.ch/), [YARA](https://virustotal.github.io/yara/) |

## Week 5 — Malware Analysis (Dynamic)

| Topic | Details |
|-------|---------|
| **Theory** | Sandboxing (Cuckoo, CAPE). API monitoring (Process Monitor, API Monitor). Debugging (x64dbg). |
| **Hands-on** | Run malware in an isolated VM. Trace API calls. Dump indicators of compromise (IOCs). |
| **Resources** | [x64dbg](https://x64dbg.com/), [The Art of Memory Forensics](https://www.memoryforensics.com/) |

## Week 6 — Reverse Engineering with Ghidra

| Topic | Details |
|-------|---------|
| **Theory** | Decompiler concepts. Symbol recovery. Patching binaries. Writing Ghidra scripts. |
| **Hands-on** | Reverse a crackme. Patch a binary to bypass license checks. Create a Ghidra script. |
| **Resources** | [Ghidra docs](https://ghidra.re/), [Crackmes.one](https://crackmes.one/) |

## Week 7 — Active Directory Attacks

| Topic | Details |
|-------|---------|
| **Theory** | AD components (Domain, DC, OU, GPO). Kerberos authentication. NTLM hashing. |
| **Hands-on** | Set up a home lab (Windows Server + Win10 clients). Use BloodHound for enumeration. |
| **Resources** | [Active Directory Security Blog](https://adsecurity.org/), [TryHackMe "Active Directory"](https://tryhackme.com/room/activedirectorybasics) |

## Week 8 — Active Directory Exploitation

| Topic | Details |
|-------|---------|
| **Theory** | Kerberoasting, AS-REP roasting, DCSync, Pass-the-Hash, Golden/Silver tickets. |
| **Hands-on** | Execute AD attacks with Impacket. Mimikatz for credential dumping. Crack kerberos tickets. |
| **Resources** | [Impacket](https://github.com/fortra/impacket), [Mimikatz](https://github.com/gentilkiwi/mimikatz) |

## Week 9 — C2 Frameworks & Red Team Operations

| Topic | Details |
|-------|---------|
| **Theory** | C2 infrastructure. Beaconing. Payload generation. OPSEC. Evasion (AMSI, ETW, AppLocker). |
| **Hands-on** | Set up a Covenant or Sliver C2. Generate staged/stageless payloads. Bypass Defender. |
| **Resources** | [Sliver](https://github.com/BishopFox/sliver), [C2 Matrix](https://www.thec2matrix.com/) |

## Week 10 — SIEM & Threat Hunting (Blue Team)

| Topic | Details |
|-------|---------|
| **Theory** | SIEM architecture (Splunk, Wazuh, ELK). Log ingestion. Correlation rules. Alerting. |
| **Hands-on** | Install Wazuh. Ingest sysmon/Windows event logs. Write custom correlation rules. |
| **Resources** | [Splunk BOTS](https://www.splunk.com/en_us/blog/security/splunk-boss-of-the-soc.html), [Wazuh docs](https://documentation.wazuh.com/) |

## Week 11 — Digital Forensics & Incident Response (DFIR)

| Topic | Details |
|-------|---------|
| **Theory** | Forensic methodology (acquisition, analysis, reporting). Volatile vs non-volatile data. Timeline analysis. |
| **Hands-on** | Analyze a memory dump with Volatility. Perform disk forensics with Autopsy/Sleuth Kit. |
| **Resources** | [Volatility](https://www.volatilityfoundation.org/), [DFIR Resources](https://aboutdfir.com/) |

## Week 12 — Final Project & Presentation

| Topic | Details |
|-------|---------|
| **Theory** | Presentation skills. Documenting findings. Ethics disclosure. |
| **Hands-on** | Choose one: (a) Full red team engagement on a lab network, (b) Full malware analysis report, (c) Build a detection pipeline with SIEM + YARA. Present findings to the club. |
| **Resources** | [Red Team Report Template](https://github.com/six2dez/pentest-report-templates) |

---

## Checkpoints & Assessments

Every 4 weeks, complete a checkpoint to confirm understanding before moving forward.

### Beginner Checkpoints

| Checkpoint | Covers | Format |
|------------|--------|--------|
| **Week 4 — Mini-CTF** | Linux, Networking, Cyber Hygiene | 10 multiple-choice + 5 practical tasks |
| **Week 8 — Practical Quiz** | Web basics, Python, Crypto | Build a simple Python tool + Burp Suite exercise |
| **Week 12 — Capstone** | Everything (full pentest flow) | End-to-end box + written report |

### Intermediate Checkpoints

| Checkpoint | Covers | Format |
|------------|--------|--------|
| **Week 4 — Web Challenge** | SQLi, XSS, CSRF, SSRF | Capture-the-flag on a vulnerable web app (e.g., DVWA, Juice Shop) |
| **Week 8 — PrivEsc Sprint** | Linux + Windows PrivEsc | Timed challenge: escalate on 2 boxes within 90 min |
| **Week 12 — CTF** | All intermediate topics | External CTF or club-hosted mini-CTF with ≥5 categories |

### Advanced Checkpoints

| Checkpoint | Covers | Format |
|------------|--------|--------|
| **Week 4 — Reverse Engineering** | Binary exploitation, shellcode | Exploit a binary with custom ROP chain |
| **Week 8 — Red Team Engagement** | AD attacks, C2, evasion | Simulated engagement against a lab domain |
| **Week 12 — Final Project** | Specialization topic of choice | 30-min presentation + written report |

---

## Mentor System

Pair advanced members with beginners to accelerate learning.

### Structure

```
Mentor (Advanced tier, Week 8+)    →    1-3 Mentees (Beginner/Intermediate)
```

### Mentor Responsibilities

- Weekly 30-min check-in with mentees
- Review mentee lab reports and provide feedback
- Help unblock when mentees get stuck on labs
- Lead one lab walkthrough session per month

### Mentee Responsibilities

- Complete weekly tasks before the next session
- Ask questions early (don't wait until you're totally stuck)
- Write up at least one walkthrough per 4 weeks
- Respect the mentor's time — come prepared

### How to Join

1. **Mentors**: After completing Advanced Week 8, express interest to club lead
2. **Mentees**: After Beginner Week 4, fill out the pairing form

---

## Career Path & Certifications

After each tier, here are recommended certifications and job roles to target.

| After Tier | Recommended Certs | Entry-Level Roles |
|------------|------------------|-------------------|
| **Beginner** | CompTIA Security+, CompTIA Network+ | SOC Analyst Level 1, IT Support, Junior Auditor |
| **Intermediate** | eJPT, PNPT, CompTIA PenTest+ | Penetration Tester Jr., SOC Analyst Level 2, Security Engineer |
| **Advanced** | OSCP, OSWP, GPEN, GCIH | Senior Pen Tester, Red Team Operator, DFIR Analyst, Security Consultant |

### Study Resources for Certs

| Cert | Free Resources | Cost |
|------|---------------|------|
| Security+ | Professor Messer YouTube, TryHackMe Security+ path | ~$400 exam |
| eJPT | INE eJPT training (free with starter pass) | ~$200 exam |
| PNPT | TCM Security Academy | ~$300 bundle |
| OSCP | PWK/OSCP course + labs | ~$1,600 bundle |

---

## Reporting & Soft Skills

Security skills mean nothing if you can't communicate findings.

### Week 11 (All Tiers) — Report Writing Session

| Topic | Details |
|-------|---------|
| **Theory** | Executive summary vs technical findings. Risk ratings. Remediation recommendations. Screenshot best practices. |
| **Hands-on** | Write a sample pentest report from a completed lab. Peer review another member's report. |
| **Template** | Use the [Pentest Report Template](https://github.com/jazzpio/PT-report-template) or [Red Team Report Template](https://github.com/six2dez/pentest-report-templates) |

### Vulnerability Disclosure

- **Responsible Disclosure**: Notify vendor first, wait 90 days, then publish
- **Bug Bounty Programs**: HackerOne, Bugcrowd, Intigriti
- **Legal warning**: Never test without written authorization

---

## Progress Tracker Template

Copy this table into your own GitHub repo or a Google Sheet to track your progress.

### Member Progress — Beginner Tier

| Week | Topic | Completed? (Y/N) | Lab Notes / Walkthrough Link | Date |
|------|-------|-----------------|------------------------------|------|
| 1 | Intro to Cybersecurity | | | |
| 2 | Cyber Hygiene | | | |
| 3 | Linux Fundamentals | | | |
| 4 | Networking Part 1 | | | |
| 5 | Networking Part 2 | | | |
| 6 | Web Technologies | | | |
| 7 | Cryptography | | | |
| 8 | Python for Security | | | |
| 9 | Recon & OSINT | | | |
| 10 | Scanning & Enumeration | | | |
| 11 | Vuln Assessment | | | |
| 12 | Capstone | | | |

### Member Progress — Intermediate Tier

| Week | Topic | Completed? (Y/N) | Lab Notes / Walkthrough Link | Date |
|------|-------|-----------------|------------------------------|------|
| 1 | OWASP Top 10 | | | |
| 2 | SQL Injection | | | |
| 3 | XSS | | | |
| 4 | Other Web Attacks | | | |
| 5 | Network Security | | | |
| 6 | Wireshark Analysis | | | |
| 7 | Cryptography Practice | | | |
| 8 | Python Offensive | | | |
| 9 | Linux PrivEsc | | | |
| 10 | Windows PrivEsc | | | |
| 11 | Password Cracking | | | |
| 12 | CTF Competition | | | |

### Member Progress — Advanced Tier

| Week | Topic | Completed? (Y/N) | Lab Notes / Walkthrough Link | Date |
|------|-------|-----------------|------------------------------|------|
| 1 | Binary Exploitation | | | |
| 2 | ROP & Shellcode | | | |
| 3 | WAF Bypass | | | |
| 4 | Malware Static | | | |
| 5 | Malware Dynamic | | | |
| 6 | Reverse Engineering | | | |
| 7 | AD Attacks | | | |
| 8 | AD Exploitation | | | |
| 9 | C2 Frameworks | | | |
| 10 | SIEM & Threat Hunting | | | |
| 11 | DFIR | | | |
| 12 | Final Project | | | |

---

## FAQ & Troubleshooting

### General

**Q: I have zero experience. Am I ready for Beginner Week 1?**
Yes. The Beginner tier assumes nothing except basic computer literacy.

**Q: I'm a Computer Science student. Can I skip Beginner?**
Take the Week 4 checkpoint quiz. If you score ≥80%, you can jump to Intermediate. Otherwise, start at Beginner.

**Q: How many hours per week should I spend?**
Aim for 4-6 hours: 1-2 hrs theory/reading + 3-4 hrs hands-on labs.

**Q: Can I use Windows instead of Linux for labs?**
Some tools require Linux. Use WSL2 (Windows Subsystem for Linux) or a Kali VM.

### Lab & Tool Issues

**Q: Nmap scan is too slow / getting no results.**
Try `-T4` for timing, `-Pn` to skip host discovery, or check firewall rules on the target.

**Q: Burp Suite not intercepting traffic.**
Make sure: (1) proxy listener is on 127.0.0.1:8080, (2) browser is configured to use proxy, (3) CA certificate is installed and trusted.

**Q: Hashcat says "No devices found" / OpenCL error.**
Use `--force` flag, or fall back to `john` (John the Ripper) which doesn't require GPU.

**Q: VirtualBox VM has no internet.**
Change network adapter from NAT to Bridged, or check if host firewall is blocking.

**Q: My exploit crashes the target instead of working.**
Check: (1) offsets, (2) bad characters, (3) ASLR enabled? Try debugging with GDB.

---

# General Resources

| Resource | Link | Tier |
|----------|------|------|
| TryHackMe | https://tryhackme.com/ | Beginner → Intermediate |
| Hack The Box | https://www.hackthebox.com/ | Intermediate → Advanced |
| PicoCTF | https://picoctf.com/ | Beginner → Intermediate |
| OverTheWire | https://overthewire.org/wargames/ | Beginner |
| PortSwigger Web Security Academy | https://portswigger.net/web-security | Intermediate |
| Pwn.College | https://pwn.college/ | Advanced |
| CryptoHack | https://cryptohack.org/ | All |
| OWASP | https://owasp.org/ | All |
| CTFtime | https://ctftime.org/ | All |
| CyberChef | https://gchq.github.io/CyberChef/ | All |

---

## Quick Reference Cards

### Linux Command Cheatsheet (Beginner)

| Command | What it does |
|---------|-------------|
| `ls -la` | List all files with details |
| `cd /path` | Change directory |
| `pwd` | Print working directory |
| `cat file.txt` | Show file contents |
| `grep 'pattern' file` | Search for pattern in file |
| `find / -name 'file'` | Search for file recursively |
| `chmod 755 file` | Set file permissions |
| `chown user:group file` | Change file owner/group |
| `ps aux` | List running processes |
| `netstat -tulpn` | Show listening ports |
| `ssh user@host` | SSH into remote machine |
| `scp file user@host:/path` | Copy file over SSH |

### Nmap Quick Reference (Intermediate)

| Command | Purpose |
|---------|---------|
| `nmap -sS -sV -O target` | Stealth SYN scan + version + OS detection |
| `nmap -p- target` | Scan all 65535 ports |
| `nmap -sC target` | Run default NSE scripts |
| `nmap --script=vuln target` | Run vulnerability scanning scripts |
| `nmap -sU target` | UDP scan (slow) |

### Hashcat Quick Reference (Intermediate)

| Command | Purpose |
|---------|---------|
| `hashcat -m 0 -a 0 hash.txt rockyou.txt` | MD5 dictionary attack |
| `hashcat -m 1000 -a 0 hash.txt rockyou.txt` | NTLM dictionary attack |
| `hashcat -m 0 -a 3 hash.txt ?a?a?a?a?a?a?a?a` | MD5 brute force (8 chars) |
| `hashcat -m 0 -a 6 hash.txt rockyou.txt ?d?d` | MD5 dictionary + 2-digit suffix rule |

---

## Contribution

Clone this repo, suggest improvements, or submit write-ups via PR!

```bash
git clone https://github.com/your-org/dit-cyber-club
```

---

*Maintained by DIT Cyber Club — Learn, Practice, Defend.*
