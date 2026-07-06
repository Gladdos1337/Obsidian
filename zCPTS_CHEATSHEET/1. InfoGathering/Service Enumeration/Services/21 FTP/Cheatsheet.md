
# Cheatsheet

| **Command**                                      | **Description**                                                                 |
| ------------------------------------------------ | ------------------------------------------------------------------------------- |
| `sudo nmap -sC -sV -p21 -v <target-ip>`          | Footprint the FTP server with default scripts and version detection.            |
| `ftp <target-ip>`                                | Connect to the target FTP server via terminal.                                  |
| `wget -r ftp://anonymous:anonymous@<target-ip>/` | Recursively download all files from the FTP server using anonymous credentials. |
|                                                  |                                                                                 |
# FTP Core Practical Points

## 1. Anonymous Login

• Goal: Check for misconfigurations allowing unauthenticated access.
• Credentials: Try logging in with `anonymous:anonymous` or `ftp:ftp`.
• Action: Check read permissions to find sensitive files, and write permissions (`put`) to see if you can upload a web shell.
## 2. Active vs. Passive Mode

• Symptom: Connection establishes successfully, but commands like `ls`, `dir`, or `get` hang or freeze.
• Cause: Strict firewall rules blocking the data channel.
• Fix: Toggle between modes. In the interactive FTP client, type `passive` to switch to passive mode (or vice versa) to bypass the block.
## 3. Software Versions & Public Exploits

• Goal: Identify the specific FTP banner and software version to find known vulnerabilities.
• Examples: Look out for classic vulnerabilities like the `vsftpd 2.3.4` backdoor or specific `ProFTPD` remote code execution exploits.
• Action: Run targeted nmap scripts (`--script ftp-*`) to gather more detail or search exploit databases immediately once a version is found.