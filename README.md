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

![NetExec SMB Brute Force](attack-simulation/netexec-bruteforce.png)

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

Important fields observed:

```
Logon Type: 3
Account Name: Administrator
Failure Reason: Unknown user name or bad password
Source Network Address: 192.168.10.250
```

This confirms that authentication attempts originated from the Kali attacker machine.

---

# SIEM Analysis (Splunk)

Windows Security logs were ingested into Splunk using the Universal Forwarder.

Initial query:

```spl
index=endpoint EventCode=4625
```

This search reveals all failed authentication attempts.

---

# Detection Rule

To detect potential brute force activity, failed logins were aggregated by source IP address.

```spl
index=endpoint EventCode=4625
| stats count by Source_Network_Address
| where count > 10
```

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
