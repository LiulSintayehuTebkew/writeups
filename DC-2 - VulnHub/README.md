# DC-2 Machine Walkthrough

## Reconnaissance

I started with `net-discover` and found the IP address of the target machine. Then I did an Nmap scan on all ports to identify services running on those ports.

## Web Enumeration

After that, I went through the website, but it didn't work. So I assumed virtual hosting was in play. I mapped the IP address to `dc-2` in the `/etc/hosts` file. The website then showed me Flag 1 on top and gave me a hint for further exploitation.

## WordPress Attack

I used `wpscan` to get possible usernames and `cewl` to get possible passwords from the website. This gave me two usernames with their passwords: Jerry and Tom. I logged into the website's hidden login page provided for me and found Flag 2.

## SSH Access

After that, I used Tom's credentials to SSH into the machine, but I was unable to see the content of files. I checked the shell I was given. It was rbash (restricted bash). I listed the binaries available in that PATH.

## Shell Escape

By using `less`, I easily captured Flag 3. Using `vi`, I opened a file and broke out. When I ran the internal command again, I was given a more interactive shell.

## User Switch

Flag 3 gave me a hint to change to Jerry. I used `su` to switch to Jerry. As Jerry, I was able to see Flag 4.

## Privilege Escalation

Jerry can run a file called `git`. It is executable. When I did `sudo -l`, it showed that Jerry can run `git` with root privilege. I searched for a way - `git` generates or spawns a shell. Using `sudo` with `git`, it actually spawned a root shell. The root privilege was inherited. In the root folder, I found the final flag.

## Lessons Learned

In this machine, I understood a lot more about shells and how you can trick a computer into giving you more freedom. If you are restricted to a specific shell like rbash, you can use available binaries like `less` or `vi` to open a more interactive shell for you - not privilege escalation, but a more interactive shell.

You can also make the shell you have more powerful by adding additional folders to your PATH if they are writable. That's why I was not able to include other binaries.

I also learned how you can use privilege escalation through sudo misconfigurations. I was allowed to run `git` with root privilege. `git` uses `less` as its pager - not to show all contents at the same time, but when it opens, you can use the `!sh` technique to open a shell from that less executable.
