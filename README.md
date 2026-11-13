# 🔎 Footprinting & Network Scanning

**Week 2 of the Network Walks Cybersecurity Internship — reconnaissance, technology fingerprinting and host discovery**

![Skill-Cybersecurity](https://img.shields.io/badge/Skill-Cybersecurity-404040?style=flat-square&labelColor=C00000)
![Kali](https://img.shields.io/badge/Kali%20Linux-v2026.x-E87500?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white)
![Nmap](https://img.shields.io/badge/Zenmap-Nmap%207.91-4682B4?style=flat-square&labelColor=000000&logo=nmap&logoColor=white)
![Windows](https://img.shields.io/badge/Windows%2010%2022H2-0078D6?style=flat-square&labelColor=000000&logo=windows&logoColor=white)
![Network](https://img.shields.io/badge/Network-10.0.0.0%2F24-238F89?style=flat-square&labelColor=000000)
![NetworkWalks](https://img.shields.io/badge/NetworkWalks-404040?style=flat-square&labelColor=C00000)

---

## 📌 Project Overview

This repository covers **Week 2 of the Network Walks Cybersecurity Internship**:
the two phases that come *before* any exploitation — **footprinting** (what can I
learn about a target from public sources?) and **scanning** (what is actually alive
on the network?).

Two modules were required:

| Module | Phase | Scope |
| --- | --- | --- |
| **W2-PM1** | Footprinting & Reconnaissance | `networkwalks.com` — 6 Kali Linux tools |
| **W2-PM5** | Scanning & Network Discovery | My own isolated lab LAN `10.0.0.0/24` — Zenmap |

Two more were completed as additional practice (**W2-PM2** GHDB, **W2-PM4**
theHarvester), and the full findings are written up in a 14-page
penetration-testing report.

---

## 🎯 Objectives

- Collect public registration, DNS and technology data about a target domain
- Fingerprint the web stack (server, CMS, plugins, hosting IP)
- Detect whether a WAF protects the site
- Install Zenmap on Windows and understand why it is no longer bundled with Nmap
- Identify the local IP address, subnet mask and default gateway
- Discover every live host in the local subnet
- Map IP addresses to MAC addresses and vendor (virtual NIC) identifiers
- Produce a network topology diagram and export it as PDF
- Document every finding with a screenshot, a command and a risk rating

---

## 🛡️ Scope & Authorization

| Target | Authorization |
| --- | --- |
| `networkwalks.com` | **Written permission** from Network Walks as part of the internship program |
| `10.0.0.0/24` | **My own** isolated VirtualBox NAT Network — no third-party devices |

⚠️ **Important:** No exploitation, credential testing or vulnerability validation
was performed. Both modules are information-gathering only. Every finding below is
an *observation*, not a confirmed vulnerability.

---

## 🏗️ Lab Architecture

```
              Host (Windows 10, i5-4200U, 7.9 GB RAM)
                          │
            ┌─────────────┴─────────────┐
            │   VirtualBox NAT Network  │
            │        10.0.0.0/24        │
            │   GW 10.0.0.1  DNS 8.8.8.8│
            └─┬──────────┬──────────┬───┘
              │          │          │
        10.0.0.2    10.0.0.3   10.0.0.10
          Kali        VM         Windows 10
       (footprinting)          (Zenmap scan source)
```

The Zenmap scan was launched **from** `10.0.0.10` (Windows 10) and discovered
its own gateway, the other lab VMs, and itself.

---

## ⚙️ Environment

| 🧩 Component | ⚙️ Configuration |
| --- | --- |
| 🖥️ Footprinting host | Kali Linux 2026.x (VM, `10.0.0.2`) |
| 🪟 Scanning host | Windows 10 22H2 (VM, `10.0.0.10/24`, 2048 MB RAM) |
| 🧰 Scanner | Zenmap / Nmap **7.91** (last official build bundling the GUI) |
| 🌐 Target subnet | `10.0.0.0/24` — 256 addresses |
| 🚪 Default gateway | `10.0.0.1` |
| 🌍 DNS server | `8.8.8.8` |
| 📸 Evidence | 23 screenshots + 2 raw-output folders + topology PDF |

---

## 🧰 Tools Used

| Tool | Purpose |
| --- | --- |
| `whois` | Domain registration, registrar, creation/expiry dates, name servers |
| `whatweb` | Web technology fingerprinting (server, CMS, plugins, IP) |
| `nslookup` | DNS resolution through a chosen resolver |
| `curl -I` | HTTP response headers and endpoint discovery |
| `wafw00f` | Web Application Firewall detection |
| `dnsrecon` | DNS record enumeration (NS, MX, SPF/TXT, SRV) |
| `Zenmap` (Nmap GUI) | Host discovery, port/service/OS detection, topology mapping |
| `ipconfig` | Local IP, subnet mask, gateway, MAC address |
| `theHarvester` | *(extra)* Public e-mail and sub-domain harvesting |
| Google Dorking (GHDB) | *(extra)* Public-index footprinting |

---

## 🪜 Activities Performed

### 1. Footprinting — W2-PM1

Six independent tools, six different angles on the same domain.

| # | Tool | Command | Key Result |
| --- | --- | --- | --- |
| 1 | `whois` | `whois networkwalks.com` | GoDaddy registrar · created **06 Nov 2019** · expires **06 Nov 2027** · NS `NS6135/NS6136.HOSTGATOR.COM` |
| 2 | `whatweb` | `whatweb networkwalks.com` | **WordPress 7.1.4** on **Apache** · plugin **WP Download Manager 3.3.76** · hosted on `192.232.216.135` |
| 3 | `nslookup` | `nslookup networkwalks.com 8.8.8.8` | Resolves to `192.232.216.135` — matches WhatWeb |
| 4 | `curl` | `curl -I https://networkwalks.com` | `HTTP/1.1 200 OK` · exposes the `/wp-json/` REST API |
| 5 | `wafw00f` | `wafw00f networkwalks.com` | **ModSecurity (SpiderLabs)** WAF in front of the site |
| 6 | `dnsrecon` | `dnsrecon -d networkwalks.com` | **8 records** — NS, MX `mail.networkwalks.com`, SPF/SRV TXT |

> **Attacker's-eye view:** before a single packet touches a port, we already know
> the CMS and its version, a plugin and its version, the hosting IP, the mail
> infrastructure and which WAF is watching.

### 2. Scanning — W2-PM5

**Task 1 — Install Zenmap.**
`nmap.org` no longer ships Zenmap with the current Windows installer (7.99x is
Nmap-only), and the old `zenmap-*-setup.exe` links now return 404. I installed
**`nmap-7.91-setup.exe`**, the last official build that still bundles the GUI
frontend together with Nmap core and Npcap.

**Task 2 — Local IP and subnet.**

```
IPv4 Address . . . . . . . . . . . : 10.0.0.10
Subnet Mask  . . . . . . . . . . . : 255.255.255.0
Default Gateway  . . . . . . . . . : 10.0.0.1
```

→ Local subnet is **`10.0.0.0/24`** (256 addresses, usable .1–.254).

**Tasks 3–6 — Live hosts, count, IPs, MACs.**

Target `10.0.0.0/24`, profile **Ping scan** (`nmap -sn 10.0.0.0/24`):

```
Nmap done: 256 IP addresses (4 hosts up) scanned in 3.06 seconds
```

| # | IP address | MAC address | Identified as |
| --- | --- | --- | --- |
| 1 | `10.0.0.1` | `52:54:00:12:35:00` | Default gateway — QEMU virtual NIC |
| 2 | `10.0.0.2` | `08:00:27:C5:FB:76` | Virtual machine — Oracle VirtualBox NIC |
| 3 | `10.0.0.3` | `08:00:27:54:C8:10` | Virtual machine — Oracle VirtualBox NIC |
| 4 | `10.0.0.10` | `08-00-27-D8-B4-DB` | **My own Windows PC** (confirmed with `ipconfig /all`) |

**Answer to Task 4: 4 hosts are live in the subnet.**

The OUI prefix is itself a finding — `52:54:00` and `08:00:27` are both
virtualisation MAC ranges, which confirms the whole subnet is lab infrastructure
and not a production network.

**Supporting check — Intense scan** (`nmap -T4 -A -v 10.0.0.0/24`) confirmed the
same four hosts and added service/OS detail:

- `10.0.0.1` → `53/tcp` open (domain)
- `10.0.0.10` → `135/tcp`, `139/tcp`, `445/tcp` open (MS RPC, NetBIOS, SMB),
  OS fingerprinted as **Windows 10 build 1909**

**Task 7 — Topology.** Opened the Topology tab, enabled the legend, and exported
the graph with *Save Graphic* as `topology.pdf`. Legend semantics: **green** = fewer
than 3 open ports, **yellow** = 3–6 open ports, **red** = more than 6.

### 3. Additional Modules

- **W2-PM2 — Google Hacking Database footprinting.** Documented publicly indexed
  information about a target. Camera and open-directory hits were recorded from
  search indexes only; **no device was opened, accessed or credential-tested**, and
  the table marks them as not tested.
- **W2-PM4 — theHarvester 4.11.1 against `microsoft.com`.**
  - Task 1 (`-b baidu -l 1000`) returned **nothing** — Baidu answers automated
    clients with an HTTP 302 redirect to `wappass.baidu.com/static/captcha/`, which
    blocks the scraper. The redirect is captured as evidence.
  - Task 2 (`-b all -l 50`) returned **9,501 unique sub-domains** and 3 public
    e-mail addresses.

---

## 📊 Findings & Risk Analysis

| # | Finding | Evidence | Impact | Level |
| --- | --- | --- | --- | --- |
| 1 | Web technology + versions exposed | WordPress 7.1.4, WP Download Manager 3.3.76, Apache | Exact versions can be matched against public advisories | ● Medium |
| 2 | Hosting server IP identifiable | `192.232.216.135` via nslookup | Reveals network location, enables further scanning | ● Low |
| 3 | WordPress REST API exposed | `curl -I` → `200 OK`, `/wp-json/` reachable | Endpoint can be enumerated for users, posts, plugins | ● Medium |
| 4 | WAF technology identifiable | wafw00f → ModSecurity (SpiderLabs) | Reveals defensive architecture, helps payload tuning | ● Low |
| 5 | DNS infrastructure exposed | 8 records incl. MX `mail.networkwalks.com`, SPF/TXT | Builds a wider infrastructure and phishing profile | ● Medium |
| 6 | Registration/hosting details public | GoDaddy registration, HostGator name servers | Useful for social engineering | ● Low |
| 7 | Multiple live hosts on local network | Zenmap: 4 hosts up in `10.0.0.0/24` | Unauthorised devices could hide in a subnet | ● Medium |
| 8 | Legacy SMB/NetBIOS reachable | `135/139/445` open on `10.0.0.10` | Common lateral-movement and ransomware surface | ● Medium |

**Risk level key:** Critical ● / High ● / Medium ● / Low ●

> These are footprinting and discovery observations. No exploitation was attempted,
> so none of them is a confirmed vulnerability on its own.

---

## ✅ Recommendations

1. Review what is publicly visible about the web stack — CMS, plugins, versions.
2. Keep WordPress core, themes and plugins patched; remove unused plugins entirely.
3. Disable or protect WordPress REST API endpoints that are not required; monitor `/wp-json/` access.
4. Trim unnecessary technical response headers and add security headers.
5. Publish only the DNS records that are needed; keep SPF, DKIM and DMARC accurate.
6. Keep the WAF enabled and tuned, and review its logs for attack attempts.
7. Scan your own networks periodically and keep a device inventory.
8. Investigate any unexpected host found during discovery.
9. Restrict SMB/NetBIOS (`135/139/445`) to hosts that genuinely need them.
10. Keep topology diagrams and asset documentation current.
11. Only ever reconnoitre systems you own or have written permission to test.

---

## 🐞 Problems Encountered & Solutions

### Problem 1 — Zenmap is no longer downloadable from nmap.org
Every `zenmap-*-setup.exe` URL on `nmap.org` returns **404**, and the current
`nmap-7.993-setup.exe` ships **Nmap only** — the GUI frontend was dropped from the
Windows build.

**Fix:** used the archived official **`nmap-7.91-setup.exe`**, the last release that
bundles Nmap core + Zenmap + Npcap in one installer. Verified afterwards with
`nmap --version` → `Nmap version 7.91`.

### Problem 2 — Headless Windows VM, no Guest Additions
`VBoxGuestAdditions` was never attached during the unattended install, so
`guestcontrol` only works in **session 0** and cannot drive the interactive desktop.
Elevation via `guestcontrol run` fails with `VERR_PROC_ELEVATION_REQUIRED`.

**Fix:** drive the guest through `controlvm keyboardputstring` / `keyboardputscancode`
and verify every step with `controlvm screenshotpng`. For GUI clicks that scancodes
cannot express, a small PowerShell helper (`mouse_event` + `SetCursorPos`) is
launched **inside the interactive session** through a scheduled task
(`schtasks /create ... /it`), which reports `session=1` and can actually reach the
desktop.

### Problem 3 — `keyboardputstring` drops and duplicates characters
Typing a long path or a target range straight into a field produced mangled text
(e.g. `10.0.0.0/24` arriving as `10.0.0.24`).

**Fix:** type, take a screenshot, *read it back*, correct, then press Enter. Never
blind-type followed by Enter.

### Problem 4 — Maltego 4.14 missing the `Domain` entity (module skipped)
Maltego 4.14 installed fine, but the `Domain` entity was absent from the palette, so
the required transform chain could not be built. The module was set aside rather
than faked; no screenshots from it are submitted.

### Problem 5 — Host RAM exhaustion
7.9 GB total with ~0.5 GB free; the Windows VM booted to a black framebuffer and
occasionally powered itself off. Freeing the saved-state Kali VM and orphaned MCP
`node.exe` processes, and running the VM headless, stabilised it.

---

## 💡 What I Learned

1. **Reconnaissance is cumulative.** WHOIS gave the name servers, which explained the
   hosting; WhatWeb gave the IP, which nslookup confirmed independently. Each tool
   validated the last one.
2. **Version numbers are the real finding.** "WordPress" is not a risk — "WordPress
   7.1.4 with a 3.3.76 plugin" is, because it can be matched against advisories.
3. **A ping scan is a `-sn` scan.** No ports, no services, just "who answers" — and
   on a local subnet it still yields MAC addresses, which is what makes the
   vendor/virtualisation identification possible.
4. **MAC prefixes are intelligence.** `08:00:27` and `52:54:00` immediately said
   "virtual machines", which told me the subnet was lab infrastructure, not a
   production LAN.
5. **Topology is a deliverable, not a screenshot.** Exporting a real PDF with a
   legend turns raw scan output into something a stakeholder can read.
6. **Tooling rots.** Zenmap being pulled from the official installer is a live lesson
   in why you verify download URLs and read the changelog instead of trusting a
   tutorial.
7. **Automation has a ceiling.** Keyboard injection into a headless VM works until
   it doesn't — knowing when to drop to a scheduled task inside the interactive
   session saved the exercise.
8. **Document as you go.** Commands, screenshots, raw output and a risk table were
   captured during the work, not reconstructed afterwards.

---

## 📁 Repository Contents

```
Week Two Submissions/
├── W2-PM-FINAL - Footprinting & Network Scanning - Mazen Ashraf.docx / .pdf
├── W2-PM1-Task1-whois.png
├── W2-PM1-Task2-whatweb.png
├── W2-PM1-Task3-nslookup.png
├── W2-PM1-Task4-curl-headers.png
├── W2-PM1-Task5-wafw00f.png
├── W2-PM1-Task6-dnsrecon.png
├── PM1-raw-output/                  # raw text output of all six tools
├── W2-PM5-Task1-install-license.png
├── W2-PM5-Task1-install-components-zenmap.png
├── W2-PM5-Task1-install-finished.png
├── W2-PM5-Task2-ipconfig.png
├── W2-PM5-Task3-4-5-6-ping-scan-4-hosts.png
├── W2-PM5-Task3-intense-scan-details.png
├── W2-PM5-Task6-ipconfig-all-mac.png
├── W2-PM5-Task7-topology-view.png
├── W2-PM5-Task7-topology-legend.png
├── W2-PM5-Task7-topology.pdf
├── W2-PM2 - GHDB Footprinting - Mazen Ashraf.docx / .pdf     # additional
└── W2-PM4-*.png + PM4-raw-output/                            # additional
```

---

## 🔐 Security & Ethical Use

All activities were performed against a domain for which **written permission** was
provided, or against **my own** isolated lab network. Nothing here is intended for
use against systems you do not own or have explicit authorisation to test.
Unauthorised access is a criminal offence in most jurisdictions, even when nothing
is modified or damaged.

---

## 🔗 Tools & Resources

- **Nmap / Zenmap:** https://nmap.org/download.html
- **Nmap 7.91 (last Zenmap-bundling build):** https://nmap.org/dist/
- **Kali Linux:** https://kali.org/get-kali
- **theHarvester:** https://github.com/laramies/theHarvester
- **Google Hacking Database:** https://www.exploit-db.com/google-hacking-database
- **OWASP WSTG — Information Gathering:** https://owasp.org/www-project-web-security-testing-guide/

---


## 📌 Project Information

**Program:** Network Walks Cybersecurity Internship · **Week:** 02 ·
**Modules:** W2-PM1 (Footprinting) + W2-PM5 (Zenmap Scanning) ·
**Deliverable:** W2-PM-FINAL Penetration Testing Report

