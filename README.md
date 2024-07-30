
![intro](https://github.com/user-attachments/assets/1642a9ac-b7f2-44e3-b042-453b11eed19d)

# tryhackme-Brute-It-writeup
This is a walkthrough for the tryhackme CTF Brute It. I will not provide any flags or passwords as this is intended to be used as a guide.

## Scanning/Reconnaissance

First off, let's store the target IP as a variable for easy access.

Command: export ip=xx.xx.xx.xx

Next, let's run an nmap scan on the target IP:
```bash
nmap -A -v $ip -D RND:10 -oN nmap.txt
```

Command break down:

-A: This flag enables aggressive scanning. It combines various scan types (like OS detection, version detection, script scanning, and traceroute) into a single scan.

-v: increases verbosity, providing more detailed output during the scan.

—$ip: provides the target IP we stored as the variable $ip.

-D RND:10 Nmap can send additional packets to confuse network intrusion detection systems (IDS) or hide the true source of the scan by randomly selecting up to 10 decoy IP addresses.

-oN nmap.txt: This option specifies normal output that should be saved to a file named “nmap.txt.

This scan reveals two open ports:

![nmap](https://github.com/user-attachments/assets/9be2630f-edec-4198-81f5-28b9301f18e5)

First, let's investigate the websever on port 80. This brings us to the Apache2 Ubuntue Default Page. 
Let's run a gobuster scan on the target:
gobuster dir -u $ip -w=/usr/share/wordlists/
