# 🛡️ Week 2 – Footprinting & Network Scanning (W2-PM-FINAL)

![Kali Linux](https://img.shields.io/badge/Kali-Linux-557C94?logo=kalilinux&logoColor=white)
![Nmap](https://img.shields.io/badge/Nmap-7.993-blue)
![Phase](https://img.shields.io/badge/Phase-1%20%26%202-red)
![Purpose](https://img.shields.io/badge/Purpose-Education%20Only-green)

**Author:** Mavani Dharm  
**Program:** Cybersecurity program at Networkwalks | **Batch:** B084 | **Week:** 02  
**Date:** 09 November 2026  
**Modules:** W2-PM1 (Multiple Kali Tools) • W2-PM5 (Zenmap Scanning)

---

## 📑 Table of Contents

1. [Disclaimer](#-disclaimer)
2. [Overview](#-overview)
3. [Lab Setup](#-lab-setup)
4. [Tools Used](#-tools-used)
5. [Part 1 – Footprinting & Reconnaissance (Kali Linux)](#-part-1--footprinting--reconnaissance-kali-linux)
6. [Part 2 – Network Scanning (Zenmap on Windows)](#-part-2--network-scanning-zenmap-on-windows)
7. [Summary of Findings](#-summary-of-findings)
8. [Risk Analysis](#-risk-analysis)
9. [Recommendations](#-recommendations)
10. [Key Learnings](#-key-learnings)
11. [Repository Structure](#-repository-structure)
12. [How to Upload This Project to GitHub](#-how-to-upload-this-project-to-github)

---

## ⚠️ Disclaimer

I performed these activities only on systems where I had **written permission** (networkwalks.com, provided for this program) or on devices that **I own myself** (my own PC and virtual network).

This material is for **education and research only**. Do not use anything from here to break the law. Unauthorised access to computers is a crime in most countries, even when nothing is damaged. Every action you take is your own responsibility.

---

## 📖 Overview

This project shows how an attacker (or a penetration tester) moves from **gathering public information** to **mapping live hosts** on a network:

| Phase | Goal | Tools |
|-------|------|-------|
| 🔍 **Phase 1 – Reconnaissance & Footprinting** | Collect information about `networkwalks.com` without exploiting anything | whois, whatweb, nslookup, curl, wafw00f, dnsrecon |
| 📡 **Phase 2 – Scanning & Network Discovery** | Find live hosts on my own local network and draw a topology | ipconfig, Nmap / Zenmap |

> ℹ️ No exploitation or vulnerability validation was done. Everything below is **information gathering** only.

---

## 🧪 Lab Setup

| Item | Details |
|------|---------|
| 🐉 Footprinting machine | Kali Linux virtual machine in Oracle VirtualBox |
| 🪟 Scanning machine | Windows 11 PC with Nmap 7.993 + Zenmap |
| 🎯 Target 1 | `networkwalks.com` (permission secured) |
| 🎯 Target 2 | My own VirtualBox host-only network `192.168.56.0/24` |

### 📥 Installing the tools (if missing)

Most tools are already included in Kali Linux. If something is missing:

```bash
sudo apt update
sudo apt install -y whois whatweb dnsutils curl wafw00f dnsrecon
```

On Windows, download and install **Nmap** from <https://nmap.org/download.html>. The installer also installs **Zenmap** (the Nmap GUI).

---

## 🧰 Tools Used

| Tool | Purpose |
|------|---------|
| 🐉 Kali Linux & 🪟 Windows | Operating systems used for the activities |
| `whois` | Domain registration details (registrar, dates, name servers) |
| `whatweb` | Fingerprint web technologies (server, CMS, plugins, IP) |
| `nslookup` | Resolve the domain name to an IP address using DNS |
| `curl -I` | Read the HTTP response headers |
| `wafw00f` | Detect whether a Web Application Firewall (WAF) protects the site |
| `dnsrecon` | Enumerate DNS records (NS, MX, A, AAAA, TXT/SPF, SRV) |
| `ipconfig` | Find my local IP address and subnet on Windows |
| Zenmap / `nmap` | Discover live hosts and draw the network topology |

---

# 🔍 Part 1 – Footprinting & Reconnaissance (Kali Linux)

Open a terminal in Kali Linux and run the commands below one by one.

## 1️⃣ WHOIS – Domain Registration Information

```bash
whois networkwalks.com
```

**What it does:** asks the public WHOIS database who registered the domain, when, and which name servers it uses.

**What I found:**

| Field | Value |
|-------|-------|
| Registrar | GoDaddy.com, LLC |
| Creation date | 2019-11-06 |
| Expiry date | 2027-11-06 |
| Name servers | `NS6135.HOSTGATOR.COM`, `NS6136.HOSTGATOR.COM` |
| DNSSEC | unsigned |
| Status | clientDelete / Renew / Transfer / UpdateProhibited (domain locks) |

💡 **Why it matters:** the name servers show the website is hosted with HostGator. No personal registrant details were visible. The domain locks protect against accidental or unauthorised changes.

---

## 2️⃣ WhatWeb – Web Technology Fingerprinting

```bash
whatweb networkwalks.com
```

**What it does:** visits the website and identifies the technologies it uses (web server, CMS, plugins, scripts).

**What I found:**

- 🔁 HTTP redirected with **301 Moved Permanently** to HTTPS, which then answered **200 OK**
- 🌐 Web server: **Apache**
- 📝 CMS: **WordPress 7.1.4**
- 🔌 Plugin: **WordPress Download Manager 3.3.76**
- 📚 jQuery 3.7.1, HTML5, Google Tag Manager
- 📧 Exposed e-mail: `info@networkwalks.com`
- 🍪 Cookie: `__wpdm_client`
- 🖥️ Server IP: `192.232.216.135`

💡 **Why it matters:** knowing the CMS and plugin versions lets an attacker look for known issues, so this information should be reviewed and kept updated.

---

## 3️⃣ Nslookup – DNS Resolution

```bash
nslookup networkwalks.com
```

**What it does:** asks a DNS server for the IP address of the domain.

**What I found:**

- DNS server used: `10.49.34.151` (my network's resolver)
- Answer type: **Non-authoritative**
- IPv4: **`192.232.216.135`**
- A second address `64:ff9b::c0e8:d887` is in the `64:ff9b::/96` **NAT64** range. It only encodes the same IPv4 address, so it was most likely created by my network's DNS64 and is not a real IPv6 record of the site.

💡 **Why it matters:** the IP address tells an attacker where the web service lives on the internet.

---

## 4️⃣ Curl – HTTP Response Headers

```bash
curl -I https://networkwalks.com
```

**What it does:** `-I` fetches **only the headers** (no page body), which is quiet and quick.

**What I found:**

- `HTTP/2 200` and `server: Apache`
- `link:` header exposes the WordPress REST API: `/wp-json/` and `/wp-json/wp/v2/pages/53`
- `set-cookie: __wpdm_client=...; secure; HttpOnly`
- `x-nginx-cache: WordPress` and `x-endurance-cache-level: 0` (reveal the caching layer)
- `referrer-policy: no-referrer-when-downgrade`
- ❌ **Not seen in this output:** `Strict-Transport-Security`, `X-Frame-Options`, `X-Content-Type-Options`, `Content-Security-Policy`

💡 **Why it matters:** headers leak technology details, and missing security headers reduce browser-side protection.

---

## 5️⃣ Wafw00f – Web Application Firewall Detection

```bash
wafw00f networkwalks.com
```

**What it does:** sends a few normal and slightly unusual requests and checks the replies to guess which WAF is in front of the site.

**What I found:**

> The site `https://networkwalks.com` is behind **ModSecurity (SpiderLabs)** WAF. Number of requests: 2

💡 **Why it matters:** a WAF is a good defence. Knowing its type helps a tester understand the protection level.

---

## 6️⃣ DNSRecon – DNS Enumeration

```bash
dnsrecon -d networkwalks.com
```

**What it does:** `-d` sets the domain. The tool collects NS, MX, A, AAAA, TXT and SRV records.

**What I found:**

| Record | Result |
|--------|--------|
| ⚠️ Errors | No answer to the DNSSEC query, and a timeout resolving SOA through my resolver (enumeration continued) |
| NS | `ns6135.hostgator.com`, `ns6136.hostgator.com` – both disclosed **BIND 9.16.23-RH** |
| MX | `mail.networkwalks.com` → `192.232.216.135` |
| A / AAAA | `192.232.216.135` / `64:ff9b::c0e8:d887` |
| TXT | Google site-verification record |
| SPF | `v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all` |
| SRV | several `_autodiscover._tcp` records → `cpanelemaildiscovery.cpanel.net` |

💡 **Why it matters:** DNS records and software versions help an attacker build a profile of the whole infrastructure (name servers, mail server, e-mail service).

---

# 📡 Part 2 – Network Scanning (Zenmap on Windows)

## 7️⃣ Find My IP Address and Subnet

Open **Command Prompt** on Windows and run:

```cmd
ipconfig
```

**What I found:**

| Adapter | IPv4 | Subnet mask |
|---------|------|-------------|
| VirtualBox host-only Ethernet | `192.168.56.1` | `255.255.255.0` → subnet **`192.168.56.0/24`** |
| Wi-Fi | `10.49.34.210` | `255.255.255.0` (gateway `10.49.34.151`) |

## 8️⃣ Ping Scan with Zenmap

Steps:

1. 🖱️ Open **Zenmap**.
2. 🎯 In **Target**, enter `192.168.56.0/24`.
3. 📋 In **Profile**, select **Ping scan**.
4. ▶️ Click **Scan**. Zenmap runs this command for you:

```bash
nmap -sn 192.168.56.0/24
```

**What `-sn` means:** a *ping scan* only checks **which hosts are alive**. It does **not** scan ports or detect services.

**Result (Nmap 7.993, 09 Nov 2026, 13:25):**

| Item | Value |
|------|-------|
| Hosts scanned | 256 |
| Hosts up | **1** → `192.168.56.1` |
| Hosts down | 255 |
| Time taken | 11.56 seconds |

The only live host is **my own Windows PC** (its VirtualBox host-only adapter). Nmap does not list a MAC address for the machine it runs on, and no other device answered in this subnet.

## 9️⃣ Network Topology

1. Click the **Topology** tab in Zenmap.
2. Click **Legend** to show what the colours and icons mean.
3. *(Optional)* Click **Save Graphic** to export the topology.

The topology shows `192.168.56.1` connected to `localhost` as a **green** node. Green means "fewer than 3 open ports" in the legend; no port scan was run here.

---

# 📊 Summary of Findings

| # | Tool | Key result |
|---|------|------------|
| 1 | whois | GoDaddy registrar, HostGator name servers, DNSSEC unsigned |
| 2 | whatweb | Apache, WordPress 7.1.4, WP Download Manager 3.3.76 |
| 3 | nslookup | `192.232.216.135` |
| 4 | curl -I | `/wp-json/` exposed in headers, security headers not seen |
| 5 | wafw00f | ModSecurity (SpiderLabs) WAF |
| 6 | dnsrecon | BIND 9.16.23-RH disclosed, MX/SPF/SRV records |
| 7 | Zenmap | 1 live host (`192.168.56.1`) in `192.168.56.0/24` |

---

# ⚖️ Risk Analysis

These are **observations, not confirmed vulnerabilities**. Seeing a version number, IP address or DNS record does not mean the system is vulnerable; further authorised testing would be needed to confirm that.

| # | Finding | Evidence | Impact | Level |
|---|---------|----------|--------|-------|
| 1 | Web technology information exposed | WordPress 7.1.4, WP Download Manager 3.3.76 | Helps identify software that needs security review | 🟠 Medium |
| 2 | Server IP identifiable | `192.232.216.135` | Shows network location of the web service | 🟡 Low |
| 3 | HTTP technical information exposed | `/wp-json/`, cache headers, no HSTS/CSP/X-Frame-Options seen | Helps fingerprinting; fewer browser protections | 🟡 Low |
| 4 | WAF identifiable | ModSecurity (SpiderLabs) | Reveals part of the security architecture | 🟡 Low |
| 5 | DNS infrastructure exposed | BIND 9.16.23-RH, MX, SPF, SRV | Helps build an infrastructure profile | 🟠 Medium |
| 6 | Live host on local network | Only `192.168.56.1` (my own PC) | No unknown devices found | 🟡 Low |

**Key:** 🔴 Critical • 🟠 Medium • 🟡 Low

---

# ✅ Recommendations

1. 🔎 **Review publicly exposed technology information** (CMS, plugins, versions).
2. 🔄 **Keep software updated** (WordPress core, plugins, server software).
3. 🧾 **Review HTTP headers** and consider HSTS, X-Frame-Options, X-Content-Type-Options and Content-Security-Policy.
4. 🌐 **Review DNS records regularly** and do not disclose DNS software versions (such as the BIND version).
5. 🛡️ **Keep the WAF (ModSecurity) enabled and tuned.**
6. 📡 **Scan your own network regularly** to see which devices are active.
7. 🕵️ **Investigate any unknown device** found during scanning.
8. 🗺️ **Keep network documentation and topology up to date.**
9. 📝 **Test only with authorisation** and within the agreed scope.

---

# 🎓 Key Learnings

- Information gathering is a strong first step: a lot can be learned from **public** data before any attack.
- Each tool answers a different question (who owns it, what runs on it, where it is, what protects it, what DNS says).
- A ping scan is quick and quiet, but it only tells you which hosts are alive.
- Good reports explain **what was done, what was found, what it means, the risk, and how to fix it**.
- Reconnaissance and scanning must **always stay within an authorised scope**.

---

# 📁 Repository Structure

```text
.
├── README.md
├── W2-PM-FINAL_-_Mavani_Dharm_Report.docx
└── screenshots/
    ├── 1_whois.png
    ├── 2_whatweb.png
    ├── 3_nslookup.png
    ├── 4_curl.png
    ├── 5_wafw00f.png
    ├── 6_dnsrecon.png
    ├── 7_zenmap_ping_scan.png
    └── 8_zenmap_topology.png
```

---

# 🚀 How to Upload This Project to GitHub

```bash
# 1. Create a folder and put README.md, the report and the screenshots/ folder inside
cd W2-PM-FINAL

# 2. Start a Git repository
git init

# 3. Add and commit the files
git add .
git commit -m "Add Week 2 footprinting and network scanning report"

# 4. Connect to your GitHub repository (create an empty one on github.com first)
git branch -M main
git remote add origin https://github.com/<your-username>/<your-repo>.git

# 5. Push
git push -u origin main
```

> 💡 You can also upload everything from the browser: open your repository → **Add file** → **Upload files** → drag the folder contents.  
> Keep the screenshots in the `screenshots` folder as shown above; they are the evidence for each step.

---

## 👤 Author

**Mavani Dharm** – Cybersecurity Student, B084  
🔗 LinkedIn: <https://www.linkedin.com/in/dharm-mavani-075525344>  
📌 **Project Information:** Cybersecurity program at Networkwalks | Batch B084 | Week 02 | Repository: GitHub

⭐ *Made for learning – always hack ethically!*
