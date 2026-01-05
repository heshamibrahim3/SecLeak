# 🛡️ SecLeak: Advanced Web Exploitation

![Category](https://img.shields.io/badge/Category-Web%20Security-red)
![Vulnerabilities](https://img.shields.io/badge/Vulnerabilities-Stored%20XSS%20%7C%20Race%20Condition-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

## 📖 Project Overview
This repository contains the solution and methodology for the **SecLeak** security assessment. The target was an internal whistleblower platform (`secleak.secenv`) protected by a VPN. The objective was to chain client-side and server-side vulnerabilities to escalate privileges from a standard user to a premium subscriber, ultimately uncovering hidden communications from a target known as "The Hive."

## 🎯 Mission Objectives
1.  **Session Hijacking:** Compromise the account of a high-privilege user ("Mr. Short") to obtain their JSON Web Token (JWT).
2.  **Privilege Escalation:** Exploit a logic flaw in the credit system to bypass payment requirements.
3.  **Data Exfiltration:** Access the private, paid-only feed of "The Hive" to retrieve the secret flag.

## ⚙️ Technical Environment
* **Target:** Internal REST-API based application (`10.81.0.10`).
* **Authentication:** Stateless JWT (JSON Web Tokens) stored in LocalStorage.
* **Network:** Internal VPN access required.

## 💥 Vulnerability Analysis

### 1. Stored Cross-Site Scripting (XSS)
The application failed to properly sanitize user input in the "Bio" profile field.
* **Vector:** Stored XSS via the global feed.
* **Exploitation:** Malicious JavaScript was injected into a profile. When the bot "Mr. Short" viewed the feed, the script executed, accessing `localStorage`, and exfiltrating the active JWT to an external webhook.

### 2. Race Condition (TOCTOU)
The internal credit transfer system suffered from a **Time-of-Check to Time-of-Use** vulnerability.
* **Vector:** `POST /api/transfer` endpoint.
* **Exploitation:** By using a **Single-Packet Attack** (via Turbo Intruder/Python), concurrent transfer requests were sent faster than the database could update the balance. This allowed spending the same credits multiple times, inflating the account balance to purchase the 10,000 credit subscription.

## 📂 Repository Contents
| File | Description |
| :--- | :--- |
| [`secleak.md`](secleak.md) | **Technical Write-up**: Detailed walkthrough of the exploit chain, including the Python race condition script and XSS payloads. |
| [`flags.txt`](flags.txt) | **Artifacts**: The captured JWT of Mr. Short and the secret key from The Hive's private post. |

## 🛠️ Tools Used
* **Burp Suite Professional** (Traffic Interception)
* **Turbo Intruder** (Race Condition Exploitation)
* **Python 3** (Scripting)
* **cURL** (API Interaction)

---
### ⚠️ Disclaimer
*This project was performed in a controlled, authorized environment for educational purposes. The techniques demonstrated here should only be used on systems where you have explicit permission to test.*
