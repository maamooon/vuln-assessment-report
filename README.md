# 🛡️ Vulnerability Assessment Report — Windows Server 2012 R2

![Classification](https://img.shields.io/badge/Classification-Confidential-red)
![Severity Critical](https://img.shields.io/badge/Critical-2-critical)
![Severity High](https://img.shields.io/badge/High-1-important)
![Severity Medium](https://img.shields.io/badge/Medium-10-yellow)
![Severity Low](https://img.shields.io/badge/Low-4-informational)
![Total Findings](https://img.shields.io/badge/Total%20Findings-17-blueviolet)
![Tools](https://img.shields.io/badge/Tools-Nessus%20%7C%20Nmap-blue)
![OS](https://img.shields.io/badge/Target-Windows%20Server%202012%20R2-0078D6?logo=windows)

> A comprehensive vulnerability assessment conducted on a **Windows Server 2012 R2** system deployed within a virtualized VMware lab environment. The target was intentionally misconfigured to simulate real-world legacy infrastructure weaknesses.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Assessment Details](#assessment-details)
- [Repository Structure](#repository-structure)
- [Tools Used](#tools-used)
- [Environment Setup](#environment-setup)
- [Findings Summary](#findings-summary)
- [Vulnerability Breakdown](#vulnerability-breakdown)
  - [Critical](#-critical-findings)
  - [High](#-high-findings)
  - [Medium](#-medium-findings)
  - [Low](#-low-findings)
- [Remediation Roadmap](#remediation-roadmap)
- [Key Takeaways](#key-takeaways)
- [Author](#author)

---

## Overview

This report documents a **vulnerability assessment** (not a penetration test) performed against a deliberately weakened Windows Server 2012 R2 instance. The goal was to identify exploitable misconfigurations and cryptographic weaknesses using industry-standard scanning tools, then map findings to CVSS scores and actionable remediation steps.

The assessment simulates a realistic **internal network attacker perspective** — scanning from a separate Kali Linux machine on the same virtual network segment, with no prior credentials.

---

## Assessment Details

| Field                 | Details                                              |
| --------------------- | ---------------------------------------------------- |
| **Target IP**         | `192.168.163.136`                                    |
| **Operating System**  | Microsoft Windows Server 2012 R2 Standard            |
| **Assessment Type**   | Vulnerability Assessment (Internal Network)          |
| **Network Segment**   | Internal virtual network (VMware Workstation)        |
| **Ports Assessed**    | All TCP/UDP (full port scan)                         |
| **Services in Scope** | FTP (21), SMB (445), RPC/DCE, IIS HTTPS, OS services |
| **Assessment Date**   | March 2026                                           |
| **Classification**    | Confidential                                         |

---

## Repository Structure

```
vulnerability-assessment-windows-server-2012/
│
├── README.md                         # This file
│
├── report/
│   ├── Vulnerability_Assessment_Report.docx    # Full formal report (Word)
│   └── Vulnerability_Assessment_Report.pdf     # PDF export of full report
│
├── evidence/
│   ├── environment-setup/
│   │   ├── 01_iis_ftp_installed.png            # IIS WebServer with FTP installed
│   │   ├── 02_firewall_disabled.png            # Windows Firewall disabled
│   │   ├── 03_smbv1_enabled.png                # SMBv1 enabled via server features
│   │   ├── 04_weak_ssl_cert_openssl.png        # Weak SHA-1 SSL cert created via OpenSSL
│   │   ├── 05_ssl_cert_installed_iis.png       # Weak SSL cert installed on IIS
│   │   └── 06_missing_kb4012598.png            # Missing EternalBlue patch (KB4012598)
│   │
│   ├── scanning/
│   │   └── 07_zenmap_nmap_scan_output.png      # Zenmap port scan results
│   │
│   └── nessus/
│       ├── critical_unsupported_os.png         # Plugin 108797 — Unsupported OS
│       ├── critical_sslv2v3_detection.png      # Plugin 20007 — SSL v2/v3 detected
│       ├── high_sweet32_cipher.png             # Plugin 42873 — SWEET32
│       ├── medium_smb_signing.png              # Plugin 57608 — SMB Signing
│       ├── medium_ms16_047.png                 # Plugin 90510 — SAM/LSAD
│       ├── medium_ssl_cert_issues.png          # Plugins 51192, 57582, 60108, 45411
│       ├── medium_tls_deprecated.png           # Plugins 104743, 157288
│       ├── low_icmp_timestamp.png              # Plugin 10114 — ICMP Timestamp
│       └── low_logjam.png                      # Plugin 83875 — DH Logjam

```

---

## Tools Used

| Tool                          | Version | Purpose                                |
| ----------------------------- | ------- | -------------------------------------- |
| **Tenable Nessus Essentials** | 10.x    | Vulnerability scanning & CVE detection |
| **Zenmap (Nmap)**             | 7.x     | Port scanning & service enumeration    |
| **VMware Workstation**        | Latest  | Virtualization platform                |
| **Kali Linux**                | 2025.3  | Attacker / scanning machine OS         |

---

## Environment Setup

The target was **intentionally misconfigured** to replicate legacy production environments commonly found in the wild:

- ✅ IIS Web Server with FTP service installed
- ✅ Windows Firewall **disabled**
- ✅ **SMBv1** enabled via server features
- ✅ Weak **512-bit RSA / SHA-1** SSL certificate generated via OpenSSL
- ✅ Weak SSL certificate installed on IIS
- ✅ Missing Windows Update **KB4012598** (EternalBlue patch)

### Assessment Phases

| Phase       | Activity                                                                 |
| ----------- | ------------------------------------------------------------------------ |
| **Phase 1** | Environment configuration — deliberate misconfiguration of target        |
| **Phase 2** | Network discovery — Zenmap (`-T4 -A -v`) for port/service enumeration    |
| **Phase 3** | Vulnerability scanning — Nessus credentialless scan against all services |
| **Phase 4** | Analysis & reporting — triage by severity with remediation guidance      |

---

## Findings Summary

| Service / Component   | Critical | High  | Medium |  Low  | Notes                     |
| --------------------- | :------: | :---: | :----: | :---: | ------------------------- |
| SSL/TLS (FTP Port 21) |    1     |   1   |   6    |   3   | Primary attack surface    |
| SMB / LSAD (Port 445) |    0     |   0   |   1    |   0   | Signing & elevation risks |
| Windows OS            |    1     |   0   |   0    |   0   | Unsupported OS lifecycle  |
| TLS Protocol Versions |    0     |   0   |   2    |   0   | TLS 1.0 & 1.1 deprecated  |
| ICMP Timestamp        |    0     |   0   |   0    |   1   | Information disclosure    |
| **TOTAL**             |  **2**   | **1** | **10** | **4** | **17 findings**           |

---

## Vulnerability Breakdown

### 🔴 Critical Findings

#### 1. Unsupported Windows Operating System

- **CVSS v3.0:** 10.0 | **Plugin:** 108797 | **Port:** N/A (OS-level)
- **Impact:** Windows Server 2012 R2 is end-of-life and receives no security updates. Any unpatched CVE can result in complete system compromise, data exfiltration, or ransomware deployment.
- **Fix:** Migrate to Windows Server 2022 or later. Apply ESU if migration is delayed and isolate from public networks.

#### 2. SSL Version 2.0 and 3.0 Protocol Detection

- **CVSS v3.0:** 9.8 | **Plugin:** 20007 | **Port:** 21/tcp, 443/tcp
- **Impact:** SSL 2.0/3.0 contain multiple cryptographic flaws including insecure CBC-mode padding and weak cipher suites, enabling POODLE and MITM attacks.
- **Fix:** Disable SSL 2.0 and SSL 3.0. Enable TLS 1.2 (minimum) or TLS 1.3.

---

### 🟠 High Findings

#### 1. SSL Medium Strength Cipher Suites Supported (SWEET32)

- **CVE:** CVE-2016-2183 | **CVSS v3.0:** ~7.5 | **Plugin:** 42873 | **Port:** 21/tcp
- **Impact:** 3DES cipher support enables the SWEET32 birthday attack, allowing plaintext recovery (credentials, session cookies) from encrypted sessions.
- **Fix:** Disable 3DES and all medium-strength ciphers. Enforce AES-128-GCM or AES-256-GCM with TLS 1.2+.

---

### 🟡 Medium Findings

| #   | Vulnerability                                               | CVE / Plugin                | CVSS | Port      |
| --- | ----------------------------------------------------------- | --------------------------- | ---- | --------- |
| 1   | SMB Signing Not Required                                    | Plugin 57608                | 5.3  | 445/tcp   |
| 2   | Security Update for SAM and LSAD (MS16-047)                 | CVE-2016-0128, Plugin 90510 | 6.8  | 49158/tcp |
| 3   | SSL Certificate Cannot Be Trusted                           | Plugin 51192                | 6.5  | 21/tcp    |
| 4   | SSL Self-Signed Certificate                                 | Plugin 57582                | 6.5  | 21/tcp    |
| 5   | SSL RC4 Cipher Suites Supported                             | Plugin 65821                | 5.3  | 21/tcp    |
| 6   | SSL Certificate with Wrong Hostname                         | Plugin 45411                | 5.3  | 21/tcp    |
| 7   | SSL Certificate Contains Weak RSA Keys                      | Plugin 60108                | 6.5  | 21/tcp    |
| 8   | TLS Version 1.0 Protocol Detection                          | Plugin 104743               | 6.5  | 21/tcp    |
| 9   | TLS Version 1.1 Deprecated Protocol                         | Plugin 157288               | 6.5  | 21/tcp    |
| 10  | SSL Certificate Signed Using Weak Hashing Algorithm (SHA-1) | Plugin 35291                | 5.3  | 21/tcp    |

**Key fix:** Replace the self-signed 512-bit/SHA-1 certificate with a CA-issued certificate (2048-bit+ RSA or ECDSA P-256, SHA-256+). Enable mandatory SMB signing via Group Policy. Apply MS16-047 (KB3148527). Disable TLS 1.0 and 1.1.

---

### 🔵 Low Findings

| #   | Vulnerability                                       | CVE           | Plugin | Port   |
| --- | --------------------------------------------------- | ------------- | ------ | ------ |
| 1   | ICMP Timestamp Request Remote Date Disclosure       | CVE-1999-0524 | 10114  | ICMP   |
| 2   | SSL/TLS Diffie-Hellman Modulus ≤ 1024 Bits (Logjam) | CVE-2015-4000 | 83875  | 21/tcp |
| 3   | SSLv3 Padding Oracle (POODLE)                       | CVE-2014-3566 | 73821  | 21/tcp |
| 4   | SSL Certificate Chain Contains RSA Keys < 2048 Bits | —             | 69551  | 21/tcp |

---

## Remediation Roadmap

| #   | Action                                         | Priority           | Effort | Expected Outcome                               |
| --- | ---------------------------------------------- | ------------------ | ------ | ---------------------------------------------- |
| 1   | Upgrade to Windows Server 2022                 | 🔴 **Immediate**   | High   | Eliminates all OS-level CVEs                   |
| 2   | Disable SSL 2.0, SSL 3.0, TLS 1.0, TLS 1.1     | 🔴 **Immediate**   | Low    | Eliminates POODLE & protocol downgrade attacks |
| 3   | Replace weak SSL certificate (512-bit / SHA-1) | 🔴 **Immediate**   | Low    | Removes MITM and certificate forgery risks     |
| 4   | Enforce strong cipher suites only (AES-GCM)    | 🟠 **Short-term**  | Low    | Mitigates SWEET32 and RC4 biasing              |
| 5   | Enable mandatory SMB signing                   | 🟠 **Short-term**  | Low    | Prevents NTLM relay attacks                    |
| 6   | Apply MS16-047 and all pending patches         | 🟠 **Short-term**  | Medium | Closes SAM/LSAD privilege escalation           |
| 7   | Block ICMP Timestamp via firewall              | 🟡 **Medium-term** | Low    | Removes time-based info leakage                |
| 8   | Generate 2048-bit+ DH parameters               | 🟡 **Medium-term** | Low    | Mitigates Logjam key-recovery                  |
| 9   | Establish patch management schedule            | 🔁 **Ongoing**     | Medium | Prevents future CVE exposure                   |
| 10  | Conduct quarterly vulnerability assessments    | 🔁 **Ongoing**     | Medium | Maintains continuous security visibility       |

---

## Key Takeaways

- **17 total findings** were identified across 4 severity tiers, demonstrating how legacy infrastructure compounds risk rapidly.
- The two **Critical findings** (EOL OS + SSL 2.0/3.0) each carry CVSS scores above 9.5 and allow **remote, unauthenticated exploitation**.
- The **FTP service** was the dominant attack surface, contributing 11 of 17 findings due to its weak SSL/TLS configuration.
- **Immediate actions** (OS upgrade, protocol hardening, certificate replacement) carry low implementation effort relative to their risk-reduction impact.
- This lab exercise demonstrates the value of **proactive vulnerability assessment** — detecting weaknesses before adversaries can exploit them is far less costly than breach response.

---

## Author

**Mamoon Ahmad**

[![Email](https://img.shields.io/badge/Email-mamoonahmad.dev%40gmail.com-D14836?logo=gmail&logoColor=white)](mailto:mamoonahmad.dev@gmail.com)
[![GitHub](https://img.shields.io/badge/GitHub-maamooon-181717?logo=github)](https://github.com/maamooon)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-maamooon-0A66C2?logo=linkedin)](https://linkedin.com/in/maamooon)

---

> **Disclaimer:** This assessment was conducted in a controlled, isolated lab environment for educational purposes only. All findings are intentional and no real production systems were targeted.
