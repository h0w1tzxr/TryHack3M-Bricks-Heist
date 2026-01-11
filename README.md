# TryHack3M: Bricks Heist

<div align="center">

![TryHackMe](https://img.shields.io/badge/TryHackMe-212C42?style=for-the-badge&logo=tryhackme&logoColor=white)
![CVE](https://img.shields.io/badge/CVE--2024--25600-Critical-DC143C?style=for-the-badge&logo=security&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.8+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![License](https://img.shields.io/badge/License-Educational-yellow?style=for-the-badge)

<br />

**Exploit • Writeup • CTF Challenge**

*A comprehensive repository containing a custom exploit and detailed writeup for the TryHack3M: Bricks Heist challenge*

<br />

[Quick Start](#quick-start) •
[Repository Structure](#repository-structure) •
[The Exploit](#the-exploit) •
[The Writeup](#the-writeup) •
[Disclaimer](#disclaimer)

---

<img src="https://img.shields.io/badge/WordPress-21759B?style=flat&logo=wordpress&logoColor=white" alt="WordPress" />
<img src="https://img.shields.io/badge/Bricks_Builder-E4405F?style=flat&logo=bricks&logoColor=white" alt="Bricks" />
<img src="https://img.shields.io/badge/CVSS-9.8_Critical-red?style=flat" alt="CVSS" />

</div>

---

## Overview

This repository documents the **TryHack3M: Bricks Heist** challenge — a medium-difficulty CTF room involving:

| Phase | Description |
|:-----:|-------------|
| **Reconnaissance** | Discovering a vulnerable WordPress site with Bricks Builder |
| **Exploitation** | Leveraging CVE-2024-25600 for unauthenticated RCE |
| **Investigation** | Uncovering a hidden crypto miner linked to **LockBit** ransomware |
| **Attribution** | Tracing Bitcoin transactions to sanctioned threat actors |

---

## Repository Structure

```
TryHack3M-Bricks-Heist/
│
├── README.md              ← You are here
│
├── Exploit/
│   ├── README.md          ← Detailed exploit documentation
│   ├── brickbreaker.py    ← The exploit tool
│   └── requirements.txt   ← Python dependencies
│
└── WriteUp/
    └── WRITEUP.md         ← Full challenge walkthrough
```

<div align="center">

| Directory | Contents | Link |
|:---------:|----------|:----:|
| **Exploit/** | BrickBreaker v2.2 — Custom CVE-2024-25600 exploit with interactive shell | [Read More](Exploit/README.md) |
| **WriteUp/** | Complete step-by-step walkthrough with screenshots and answers | [Read More](WriteUp/WRITEUP.md) |

</div>

---

## Quick Start

### Get a Shell in 30 Seconds

```bash
# Clone the repository
git clone https://github.com/h0w1tzxr/TryHack3M-Bricks-Heist.git
cd TryHack3M-Bricks-Heist/Exploit

# Set up environment
python3 -m venv brickbreaker && source brickbreaker/bin/activate
pip install -r requirements.txt

# Launch exploit
python3 brickbreaker.py https://target.com
```

### Read the Full Writeup

Jump straight to the [Complete Writeup](WriteUp/WRITEUP.md) to see how the challenge was solved from start to finish.

---

## The Exploit

<div align="center">

### BrickBreaker v2.2

*Unauthenticated Remote Code Execution for WordPress Bricks Builder*

</div>

| Feature | Description |
|:-------:|-------------|
| **Auto-exploitation** | Extracts nonce and exploits automatically |
| **Interactive Shell** | Full PTY-like experience with history |
| **File Transfer** | Upload/download files via base64 |
| **Reverse Shells** | 5 payload types (bash, python, nc, php, perl) |
| **Batch Scanner** | Mass vulnerability scanning |
| **Beautiful TUI** | Rich terminal interface |

```
 ██████╗ ██████╗ ██╗ ██████╗██╗  ██╗    ██████╗ ██████╗ ███████╗ █████╗ ██╗  ██╗███████╗██████╗ 
 ██╔══██╗██╔══██╗██║██╔════╝██║ ██╔╝    ██╔══██╗██╔══██╗██╔════╝██╔══██╗██║ ██╔╝██╔════╝██╔══██╗
 ██████╔╝██████╔╝██║██║     █████╔╝     ██████╔╝██████╔╝█████╗  ███████║█████╔╝ █████╗  ██████╔╝
 ██╔══██╗██╔══██╗██║██║     ██╔═██╗     ██╔══██╗██╔══██╗██╔══╝  ██╔══██║██╔═██╗ ██╔══╝  ██╔══██╗
 ██████╔╝██║  ██║██║╚██████╗██║  ██╗    ██████╔╝██║  ██║███████╗██║  ██║██║  ██╗███████╗██║  ██║
 ╚═════╝ ╚═╝  ╚═╝╚═╝ ╚═════╝╚═╝  ╚═╝    ╚═════╝ ╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝
```

<div align="center">

**[Full Exploit Documentation →](Exploit/README.md)**

</div>

---

## The Writeup

<div align="center">

### TryHack3M: Bricks Heist — Complete Walkthrough

*From exploitation to threat actor attribution*

</div>

The writeup covers the entire attack chain and investigation:

| Phase | What You'll Learn |
|:-----:|-------------------|
| **Reconnaissance** | Port scanning, WordPress enumeration, identifying Bricks Builder |
| **Exploitation** | Using BrickBreaker to gain initial access via CVE-2024-25600 |
| **Forensics** | Discovering hidden files, malicious services, and crypto miners |
| **Blockchain** | Decoding obfuscated wallet addresses and tracing transactions |
| **Attribution** | Linking activity to the **LockBit** ransomware group via OFAC |

### Challenge Answers Preview

| # | Question | Hint |
|:-:|----------|------|
| 1 | Hidden .txt file content | `THM{...}` |
| 2 | Suspicious process name | `nm-****-******` |
| 3 | Service name | `******.service` |
| 4 | Miner log file | `****.****` |
| 5 | Wallet address | `bc1q...` |
| 6 | Threat group | 7 characters |

<div align="center">

**[Read Full Writeup →](WriteUp/WRITEUP.md)**

</div>

---

## Tools & Technologies

<div align="center">

| Tool | Purpose |
|:----:|---------|
| ![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white) | Exploit development |
| ![RustScan](https://img.shields.io/badge/RustScan-000000?style=flat&logo=rust&logoColor=white) | Port scanning |
| ![CyberChef](https://img.shields.io/badge/CyberChef-4285F4?style=flat&logo=google&logoColor=white) | Data decoding |
| ![Blockchain](https://img.shields.io/badge/WalletExplorer-F7931A?style=flat&logo=bitcoin&logoColor=white) | Transaction tracing |

</div>

---

## Disclaimer

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                                  WARNING                                     ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║  This repository is for EDUCATIONAL and AUTHORIZED SECURITY TESTING only.    ║
║                                                                              ║
║  • Only use on systems you OWN or have EXPLICIT WRITTEN PERMISSION to test   ║
║  • Unauthorized access to computer systems is ILLEGAL                        ║
║  • The authors are NOT responsible for any misuse or damage caused           ║
║  • Always follow RESPONSIBLE DISCLOSURE practices                            ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

---

## References

| Resource | Link |
|----------|------|
| CVE-2024-25600 (NVD) | [nvd.nist.gov](https://nvd.nist.gov/vuln/detail/CVE-2024-25600) |
| Original Disclosure | [Snicco Security](https://snicco.io/vulnerability-disclosure/bricks/unauthenticated-rce-in-bricks-1-9-6) |
| OFAC Sanctions | [Treasury.gov](https://ofac.treasury.gov/recent-actions/20240220) |
| LockBit Advisory | [CISA](https://www.cisa.gov/news-events/cybersecurity-advisories/aa23-165a) |
| TryHackMe Room | [TryHackMe](https://tryhackme.com/room/introtosecurityarchitecture) |

---

## Credits

<div align="center">

| Role | Credit |
|:----:|--------|
| **Author** | [constantines](https://github.com/h0w1tzxr) |
| **Vulnerability Discovery** | [Snicco Security](https://snicco.io) |
| **Original PoC** | [Chocapikk](https://github.com/Chocapikk/CVE-2024-25600) |
| **Challenge** | [TryHackMe](https://tryhackme.com) |

</div>

---

<div align="center">

### Star this repository if you found it useful

<br />

![Stars](https://img.shields.io/github/stars/h0w1tzxr/TryHack3M-Bricks-Heist?style=social)
![Forks](https://img.shields.io/github/forks/h0w1tzxr/TryHack3M-Bricks-Heist?style=social)
![Watchers](https://img.shields.io/github/watchers/h0w1tzxr/TryHack3M-Bricks-Heist?style=social)

<br />

**Created by constantines**

<sub>*Happy Hacking!*</sub>

</div>
