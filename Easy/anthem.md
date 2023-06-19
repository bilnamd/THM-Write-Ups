**THM** : Anthem<br>
**Room Link** : https://tryhackme.com/room/anthem<br>
**Difficulty** : Easy <br>
**Flags to capture** : User and Root<br>
**Key Topics, Techniques, and Tools** : <br>
 - Ports and Services scan : [nmap](https://nmap.org/)
 - Web content discovery
 - CMS : [Umbraco](https://umbraco.com/)
 - Remote Desktop Protocol
 - Windows privileges escalation

<p align="center">   <img src="https://i.imgur.com/olKE1Z3.png"> </p>  

# Summary

- [Adding IP to the hosts file](#adding-ip-to-the-hosts-file)
- [Nmap scan](#nmap-scan)
- [Web content discovery](#web-content-discovery)
- [User flag](#user-flag)
- [Privilege escalation](#privilege-escalation)

<p align="center">   <img src="https://i.imgur.com/Kmtrf12.png"> </p> 

## Adding IP to the hosts file 
I start by adding the IP of the machine to my hosts file:<br>
```
echo "10.10.206.250 anthem.thm"|sudo tee -a /etc/hosts
```   

## Nmap scan

I perform an `Nmap` scan on the target host and save the output to a file called `nmapResults.txt` <br>
The scan includes all ports `-p-` and uses aggressive scan options `-A` <br>

```
└─# nmap anthem.thm -A -p- -oN nmapResults.txt
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-12 19:51 CEST
Note: Host seems down. If it is really up, but blocking our ping probes, try -Pn
Nmap done: 1 IP address (0 hosts up) scanned in 3.49 seconds 
```
The host seems to block ping probes sent by Nmap so I add the "-Pn" option with the Nmap command to perform an analysis without sending ping probes :
```
└─# nmap anthem.thm -A -p- -oN nmapResults.txt -Pn

Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-12 19:53 CEST
Nmap scan report for anthem.thm (10.10.206.250)
Host is up (0.031s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=WIN-LU09299160F
| Not valid before: 2023-06-11T17:44:05
|_Not valid after:  2023-12-11T17:44:05
| rdp-ntlm-info: 
|   Target_Name: WIN-LU09299160F
|   NetBIOS_Domain_Name: WIN-LU09299160F
|   NetBIOS_Computer_Name: WIN-LU09299160F
|   DNS_Domain_Name: WIN-LU09299160F
|   DNS_Computer_Name: WIN-LU09299160F
|   Product_Version: 10.0.17763
|_  System_Time: 2023-06-12T17:55:29+00:00
|_ssl-date: 2023-06-12T17:56:31+00:00; 0s from scanner time.
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: specialized|general purpose
Running (JUST GUESSING): AVtech embedded (87%), Microsoft Windows XP (85%)
OS CPE: cpe:/o:microsoft:windows_xp::sp3
Aggressive OS guesses: AVtech Room Alert 26W environmental monitor (87%), Microsoft Windows XP SP3 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   31.91 ms 10.8.0.1
2   32.17 ms anthem.thm (10.10.206.250)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 190.20 seconds

```

There are 3 open ports :
- `HTTP` - `Microsoft HTTPAPI httpd 2.0` on port `80`
- `RDP` - `ms-wbt-server Microsoft Terminal Services` on port `3389`
<br>



## Web Content Discovery  
I  start by the website enumeration. 
The site is a blog with two posted articles :
<img src="https://i.imgur.com/K1X95jW.png">

There is a `robots.txt` file :

<img src="https://i.imgur.com/Bwh00Pg.png"> 

The `robots.txt` file contains references to `Umbraco`. After some research, I understand that it's a cms. I can't find any known exploit.  
The file also contains what looks like a password, which I keep in my notes.  
I decide to take a look at the various pages contained in the `robots.txt` file :

<img src="https://i.imgur.com/IXcgxzx.png">

The `/umbraco/` directory contains a login page to access the site admin.  
I try the password found earlier with the username `admin` as well as the default Umbraco credentials found on the internet, but none of them work.


I come back to the blog and look at the articles :

<img src="https://i.imgur.com/UdC54kw.png">

The first is a job offer with an e-mail address.  
This tells me the domain name (`anthem.com`) and how the e-mail addresses are created: they use the initials of the users.  
Example: for `Jane Doe` (the author of the article), it's `JD@anthem.com`.  
So I'm keeping all this information in my notes.


<br>

The second article is a message posted by `James Orchad Halliwell` which appears to be the admin's name. The message contains a poem : 
<img src="https://i.imgur.com/SagySHE.png">

After a search on the Internet, I found the name of the author of the poem `Solomon Grundy`, which I keep in my notes. 
<img src="https://i.imgur.com/STxKtE1.png">

I'm also looking for the 4 flags hidden on the source code and the pages of the site.  

Flag 1 :

<img src="https://i.imgur.com/WRDzi69.png">


Flag 2 :

<img src="https://i.imgur.com/M6ktRQp.png">

Flag 3 :   

<img src="https://i.imgur.com/LRbsNqO.png">

Flag 4 : 

<img src="https://i.imgur.com/tXUyHjE.png">

## User flag

After going around the website and presumably recovering the admin credentials, I decide to connect to RDP to test them : 
```
└─# rdesktop anthem.thm 
Autoselecting keyboard map 'fr' from locale

ATTENTION! The server uses and invalid security certificate which can not be trusted for
the following identified reasons(s);

 1. Certificate issuer is not trusted by this system.

     Issuer: CN=WIN-LU09299160F


Review the following certificate info before you trust it to be added as an exception.
If you do not trust the certificate the connection atempt will be aborted:

    Subject: CN=WIN-LU09299160F
     Issuer: CN=WIN-LU09299160F
 Valid From: Sun Jun 11 19:44:05 2023
         To: Mon Dec 11 18:44:05 2023

  Certificate fingerprints:

       sha1: fba9c04d2e6f1a0ff8d48514e2f43175a9fce7b2
     sha256: 29c7a58378d84b1a0314e6231370f53f35f425aedcf01241c93c2341b505d947


Do you trust this certificate (yes/no)? yes
Failed to initialize NLA, do you have correct Kerberos TGT initialized ?
Core(warning): Certificate received from server is NOT trusted by this system, an exception has been added by the user to trust this specific certificate.
Connection established using SSL.

```
<img src="https://i.imgur.com/jC02Gei.png">

I can now access the machine using credentials and I retrieve the `user.txt` flag from the desktop : 
<img src="https://i.imgur.com/xovbJd1.png">

## Privilege Escalation 


I start by looking for any hidden folders or files on the machine :  
<img src="https://i.imgur.com/cMDlekq.png">

I discovered an interesting folder called `backup` that had been hidden on `C:\` :
<img src="https://i.imgur.com/wkNj7IF.png">

The folder contains a `restore` file which I don't have access to : 
<img src="https://i.imgur.com/pZtO4zi.png">

I give full control to all users. :  
<img src="https://i.imgur.com/bGqJWsE.png">
<img src="https://i.imgur.com/gyX9EcA.png">

This allows me to access the file containing what appears to be a password : 
<img src="https://i.imgur.com/DqjyXcL.png">


So I test this password by trying to launch a command prompt as admin and I manage to be root of the machine :
<img src="https://i.imgur.com/Z0qYF6r.png">

So I can retrieve the `root.txt` flag :
<img src="https://i.imgur.com/6aSvl3C.png">