# [HTB] Admirer - Penetration Testing Report

## Executive Summary
* **Target OS:** Linux
* **Difficulty:** Easy
* **Target IP:** 10.129.229.101
* **Vectors Used:** Adminer 4.7.8 LFI/SSR Vulnerability & Python Library Hijacking

---

## 1.  Reconnaissance

Port scan has been done via nmap against target.
```bash
$ nmap -sV -sC -p- 10.129.229.101 -oN
```
result:
