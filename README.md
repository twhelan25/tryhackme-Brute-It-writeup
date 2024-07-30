
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
Let's run a gobuster and nikto scans on the target:
```bash
gobuster dir -u $ip -w=/usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-words.txt -x php,txt,html -o buster.txt
```
```bash
nitko -h $ip | tee nikto.txt
```
The scans reveal an /admin directory, we have also gathered all the information to answer the reconn questions:

![nikto](https://github.com/user-attachments/assets/0f269903-1624-406a-b607-877f7e8a0097)

The /admin page brings us to a login page. Let's try some common default credentials like admin:admin, admin:password etc. It is also good to try some common user names such as james, and bob, as this can sometimes reveal error messages expose info about the login, like "invalid username & password" or "invalid password" in the same setting. Let's also check the source code. The source good reveals a user name in the html comments!

![source code](https://github.com/user-attachments/assets/e6d13c49-f350-4987-bf89-15518ddbc8c9)

Alright, now that we know the username, let's get a more indepth look at this login by capturing the login request on burp suite. I'm going to use the built in chrome browser. On the tryhackme Kali attack box, you can enable it by going to project settings, misc, and check the box at the very bottom for "Allow Burp's browser to run without a sandbox". Now, go to the proxy tab, open browser, and navigate to the login page. I put in john and Password12345 for the creds, turn on intercept on Burp, then sumbit your login request from chrome.

![post](https://github.com/user-attachments/assets/27a419f6-48f8-4ea0-9d93-f05499d65be4)

This capruted post request will provide us with everything we need to craft a dictionary attack. Here's how our command should look:
