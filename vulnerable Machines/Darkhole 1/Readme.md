# DarkHole: 1 — CTF Write-up

## Target
- **Victim hostname:** `darkhole.lan`
- **Victim IP:** `192.168.31.113`
- **Scope:** Authorized local CTF lab only

> **Redaction:** Other discovered hostnames/IP addresses and all passwords, session tokens, and credential values have been omitted. Do not reuse lab credentials on real systems.

## Summary
The supplied screenshots show a path from web enumeration on TCP/80 to a shell as `www-data`, then to `john` through PATH-based command resolution, and finally to root through a sudo rule that permits running a modifiable Python script as root.

## Port scan
```bash
nmap -sC -sV 192.168.31.113
```
Observed services:
- **22/tcp — SSH:** OpenSSH 8.2p1 Ubuntu 4ubuntu0.2
- **80/tcp — HTTP:** Apache httpd 2.4.41 (Ubuntu), with a PHP web application

No other target ports are confirmed by the supplied screenshots.

## Web enumeration
```bash
gobuster dir -u http://192.168.31.113 -w /usr/share/wordlists/dirb/big.txt -x php

