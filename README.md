
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
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt $ip http-post-form "/admin/:user=^USER^&pass=^PASS^:Username or password invalid"
```
And this cracks the login password! After logging in we are see the flag as well as a message to john and a link to his rsa key. We need to save this key.
![msg to john](https://github.com/user-attachments/assets/bf154e4f-0d71-45b3-b085-6f78a1558a37)

Now, we need to change it's permissions and attempt to log in with it:
![ssh rsa](https://github.com/user-attachments/assets/47efd633-5b22-4075-9b0d-8ccfede64645)
But it prompts for a password. So we need to use ssh2john to create a hash that john can crack:
```bash
ssh2john id_rsa > hash
```
Then crack this hash with john:
```bash
/sbin/john --wordlist=/usr/share/wordlists/rockyou.txt hash
```
Now that we have the password let's use it to ssh onto the target:
```bash
ssh -i id_rsa john@$ip
```
And we're in! Cat user.txt in john's home directory.

# Privilege Escalation
Now, let's see what john can do:

![sudo -l](https://github.com/user-attachments/assets/3086283e-c108-4795-b07d-4c8db86db830)

This looks promising, let's look it up on gtfobins.github.io:

![gtfo](https://github.com/user-attachments/assets/fe26013d-dd64-44f3-a2da-d70147dbbc12)

Let's try this on the target. Since we can assign any file to be "LFILE" let's make it the /etc/shadow so that we can get the hash of root's password:

![lfile](https://github.com/user-attachments/assets/c8f83217-3b0d-4df9-8856-cf1f20ba7cf6)

Now, we just have to save the line for root as a file on our machine and crack it using john.
![root](https://github.com/user-attachments/assets/c2e39c12-7b86-4bd2-b406-efde5288af16)
And we have root's password! Now, just switch users to root and grab the root flag:
```bash
su root
```
I hope you enjoyed this room and learned some new techniques!
