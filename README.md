# SMB Brute Force Detection Lab (SOC Simulation)

## Overview

This project simulates a brute force authentication attack against a Windows endpoint and demonstrates how the activity can be detected using Splunk SIEM.

The objective of this lab is to replicate a common attack technique used by threat actors attempting to gain unauthorized access through password guessing.

The detection workflow follows the full SOC investigation lifecycle:

Attacker → Windows Authentication Logs → SIEM Detection

---

# Lab Environment

| System | Role | IP Address |
|------|------|------|
| Kali Linux | Attacker | 192.168.10.250 |
| Windows 10 | Target Endpoint | 192.168.10.100 |
| Splunk Enterprise | SIEM | 192.168.10.x |

Tools Used:

- NetExec (successor to CrackMapExec)
- Windows Event Viewer
- Splunk Enterprise
- VirtualBox Lab Environment

---

# Attack Simulation

A brute force attack was performed against the SMB service on the Windows target.

### Attack Command

```
netexec smb 192.168.10.100 -u Administrator -p test.txt
```
### Attack Execution

<img width="700" height="834" alt="Image" src="https://github.com/user-attachments/assets/b7185486-0bc7-4cf0-9783-16d9f147757f" />

The tool attempts multiple password guesses against the Administrator account.

Example output:

```
STATUS_LOGON_FAILURE
```

Each failed login generates a Windows Security Event.

---

# Windows Log Evidence

The Windows target logs each authentication failure as:

```
Event ID: 4625
```
<img width="742" height="600" alt="Image" src="https://github.com/user-attachments/assets/f7fa8e65-b8e8-4f83-898a-c31b3dcb6172" />
Important fields observed:

```
Logon Type: 3
Account Name: Administrator
Failure Reason: Unknown user name or bad password
Source Network Address: 192.168.10.250
```
<img width="626" height="441" alt="Image" src="https://github.com/user-attachments/assets/34416be5-a785-4045-a205-71b7f22576f7" />

This confirms that authentication attempts originated from the Kali attacker machine.

<img width="628" height="441" alt="Image" src="https://github.com/user-attachments/assets/6d2652c4-5994-42ac-920d-fe976d8b9233" />

---

# SIEM Analysis (Splunk)

Windows Security logs were ingested into Splunk using the Universal Forwarder.

Initial query:

```spl
index=endpoint EventCode=4625
```
<img width="1903" height="806" alt="Image" src="https://github.com/user-attachments/assets/7e33cd14-7853-4286-875a-3b9fd169db12" />

This search reveals all failed authentication attempts.

---

# Detection Rule

To detect potential brute force activity, failed logins were aggregated by source IP address.

```spl
index=endpoint EventCode=4625
| stats count by Source_Network_Address
| where count > 10
```
<img width="1904" height="838" alt="Image" src="https://github.com/user-attachments/assets/d42d58a4-c812-47fa-8b77-90ac0b448863" />

Detection Result:

```
192.168.10.250 → 51 failed login attempts
```

This pattern is indicative of a brute force attack.

---

# MITRE ATT&CK Mapping

Technique:

```
T1110 – Brute Force
```

Description:

Adversaries attempt to gain access by systematically guessing passwords for user accounts.

---

# Key Takeaways

- Brute force authentication attempts generate Windows Event ID 4625
- Multiple failed logins from a single source IP are a strong indicator of brute force activity
- SIEM tools such as Splunk can detect abnormal authentication patterns
- Aggregating failed login attempts helps identify malicious login behavior

---

# Skills Demonstrated

- Security log analysis
- Windows authentication investigation
- Attack simulation
- SIEM detection engineering
- MITRE ATT&CK mapping
- Incident investigation workflow
