# Incident Report — Simulated Brute Force Attack

**Date:** April 29, 2026  
**Analyst:** Shubham Singh Darmwal 
**Severity:** Medium  
**Status:** Resolved (Simulated)

---

## Summary

During a controlled home lab exercise, a brute force attack was 
simulated against a Windows 10 endpoint monitored by Wazuh SIEM. 
The attack was successfully detected and alerted within seconds.

---

## Environment

| Component | Details |
|---|---|
| SIEM | Wazuh 4.7.5 |
| Endpoint | Windows 10 (Wazuh Agent v4.7.5) |
| Attacker | Host machine via PowerShell |
| Network | VirtualBox Host-Only (192.168.56.0/24) |

---

## Timeline

| Time | Event |
|---|---|
| 10:59:15 | First failed logon attempt detected |
| 10:59:25 | Multiple logon failures alert triggered (Rule 60204) |
| 10:59:38 | Brute force technique T1110 flagged — Level 10 |
| 10:59:49 | Account lockout triggered (Rule 60115) — Level 9 |
| 11:18:14 | Windows audit policy change detected (Rule 60112) |

---

## Alerts Generated

| Rule ID | Description | Level | MITRE |
|---|---|---|---|
| 60204 | Multiple Windows logon failures | 10 | T1110 |
| 60115 | User account locked out | 9 | T1110, T1531 |
| 60122 | Logon failure — bad password | 5 | T1078, T1531 |
| 60112 | Windows audit policy changed | 8 | — |
| 60106 | Windows logon success | 3 | T1078 |

---

## MITRE ATT&CK Mapping

| Technique | Name | Tactic |
|---|---|---|
| T1110 | Brute Force | Credential Access |
| T1078 | Valid Accounts | Defense Evasion, Persistence |
| T1531 | Account Access Removal | Impact |

---

## Analysis

The brute force simulation generated 10 consecutive failed logon 
attempts within 34 seconds. Wazuh correctly:

- Detected individual failures (Rule 60122, Level 5)
- Correlated them into a brute force pattern (Rule 60204, Level 10)
- Triggered an account lockout alert (Rule 60115, Level 9)
- Mapped findings to MITRE ATT&CK T1110

This demonstrates Wazuh's correlation capability — not just logging 
individual events but connecting them into meaningful attack patterns.

---

## Response Actions (Simulated)

1. Identified source of failed logons — local machine
2. Confirmed account lockout triggered automatically
3. Would escalate to L2 if source was external/unknown IP
4. Recommend reviewing logon success (Rule 60106) after failures
   to check if any attempt succeeded before lockout

---

## Lessons Learned

- Windows audit policy must be explicitly enabled for process 
  creation logging (Event ID 4688)
- Wazuh correlates multiple low-severity events into high-severity 
  alerts automatically
- Account lockout is a strong indicator of brute force — always 
  check for successful logon following failed attempts

---

## Evidence

Screenshots available in `/screenshots` folder of this repository.
