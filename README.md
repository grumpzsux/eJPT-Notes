# eJPT Notes 2022 *(eLearnSecurity Junior Penetration Tester)*
#### Twitter: @GRuMPzSux
### Collection of notes to prepare for the eLearnSecurity eJPT certification exam.

### Before you continue reading, please do the following:
###### 1. Please sign up for a **FREE** https://ine.com account, and complete the Penetration Testing Student course.  It is ABSOLUTE FREE! To be honest, everything you need to pass the eJPT is in that free course.
###### 2. Do NOT treat this exam as a CTF box.  This is NOT like a CTF box. Host Discovery is important. 😉



# Host Discovery / Enumeration
#### Let's find out what we are working with.  We are completely blind.
![image](https://user-images.githubusercontent.com/80599694/147906631-44437a80-1098-4275-bc2f-22b8827da8bb.png)

### Ping Sweep, who can we find on the network?


#### fping:
````bash
fping -a -g {IP RANGE} 2>/dev/null
````
#### fping example:
````bash
fping -a -g 10.10.10.0/8 2>/dev/null
`````
#### Nmap Ping Sweep:
````bash
 nmap -sn 10.10.10.0/8 | grep -oP '(?<=Nmap scan report for )[^ ]*'
````

# Enumerate Hosts Found on Network
#### Once you have found alive hosts on a network, its time to knock on the doors.

#### Nmap TCP Quick Scan (step 1)
````bash
nmap -sC -sV 10.10.10.10
````
#### Nmap TCP Full Scan (Step 2)
````bash
nmap -sC -sV -p- 10.10.10.10
````
#### Nmap UDP Quick Scan
````bash
nmap -sU -sV 10.10.10.10
````
#### Always save your scans, you never know when you need to pull them up.
````bash
nmap -sn 10.10.10.0/24 -oN hosts.nmap
````

# Find Common Vulnerabilities
#### After you have done all of your scans, and identified open ports on your target, it's time to see if any services are vulnerable.
![image](https://user-images.githubusercontent.com/80599694/147908174-3d6663e0-4049-4471-969c-f43db2c9c74e.png)

#### Common Ports to Look at:
| Port | Protocol |
| ------------- |:-------------:|
| 21 | FTP |
| 22 | SSH |
| 23 | TELNET |
| 25 | SMTP |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |
| 110 | POP3 |
| 115 | SFTP |
| 143 | IMAP |
| 135 | MSRPC |
| 137 | NETBIOS |
| 138 | NETBIOS |
| 139 | NETBIOS |
| 445 | SMB |
| 3306 | MYSQL | 
| 1433 | MYSQL | 
| 3389 | RDP |

### Use Nmap as a Lightweight Vulnerability Scanner
````bash
nmap -sV --script=vulners -v 10.10.10.1
````
##### if you do not have vulners installed, please install here: https://github.com/vulnersCom/nmap-vulners
````bash
nmap --script vuln --script-args=unsafe=1 -iL hosts.nmap
````

## Port 21 - FTP Enumeration
##### Sometimes clues are put here. :wink: Old versions of FTP maybe vulnerable. Always check the version. Search for the exploit using Google / Searchsploit / Rapid7. If you find some credential, try it on SSH / Login page / database.

#### Enumerate FTP Service with Nmap:
````bash
nmap --script=ftp-anon,ftp-bounce,ftp-libopie,ftp-proftpd-backdoor,ftp-vsftpd-backdoor,ftp-vuln-cve2010-4221,tftp-enum -p 21 $ip
````
#### Check for FTP Vulnerabilities with Nmap:
````bash
nmap --script=ftp-* -p 21 10.10.10.1
````
#### Connect to FTP Service:
````bash
ftp 10.10.10.1
````
````bash
ncftp 10.10.10.1
````
##### Many ftp-servers allow anonymous users. anonymous:anonymous 👈️

#### Bruteforce FTP with a Known Username You Found:
````bash
hydra -l $user -P /usr/share/john/password.lst ftp://10.10.10.1:21
````
````bash
hydra -l $user -P /usr/share/wordlistsnmap.lst -f 10.10.10.1 ftp -V
````
````bash
medusa -h 10.10.10.1 -u $user -P passwords.txt -M ftp
````

#### Enumerate Users on FTP Service:
````bash
ftp-user-enum.pl -U users.txt -t 10.10.10.1
````
````bash
ftp-user-enum.pl -M iu -U users.txt -t $ip
````
##### If you do not have ftp-user-enum.pl, you can download it here: https://pentestmonkey.net/tools/ftp-user-enum/ftp-user-enum-1.0.tar.gz

#### Useful Commands for FTP Service (cmd line):
````bash
• send # Send single file
• put # Send one file.
• mput # Send multiple files.
• mget # Get multiple files.
• get # Get file from the remote computer.
• ls # list 
• mget * # Download everything
• binary = Switches to binary transfer mode.
• ascii = Switch to ASCII transfer mode
````
#### Always Check for FTP Configuration Files:
````bash
• ftpusers
• ftp.conf
• proftpd.conf
````
##### Vulnerable FTP Versions:
````bash
• ProFTPD-1.3.3c Backdoor
• ProFTPD 1.3.5 Mod_Copy Command Execution
• VSFTPD v2.3.4 Backdoor Command Execution
````
#### FTP Exploitation Methodology:
````bash
1. Gather version numbers
2. Check Searchsploit
3. Check for Default Creds
4. Use Creds previously gathered
5. Download the software
````
## Port 445 - SMB Enumeration
#### Always check for SMB.  You might get lucky and find a vulnerable machine running SMB that has remote code execution.  Remember to use searchsploit, or google to check all service versions for publicly available exploits.

#### Scan for NETBIOS/SMB Service with Nmap:
````bash
nmap -p 139,445 --open -oG smb.txt 192.168.1.0/24
````
#### Scan for NETBIOS/SMB Service with nbtscan:
````bash
nbtscan -r 192.168.1.0/24
````

#### Enumerate the Hostname:
````bash
nmblookup -A 10.10.10.1
````
#### Check for Null Sessions:
````bash
smbmap -H 10.10.10.1
````
````bash
rpcclient -U "" -N 10.10.10.1
````
````bash
smbclient \\\\$ip\\ShareName
`````
##### if getting error "protocol negotiation failed: NT_STATUS_CONNECTION_DISCONNECTED"
````bash
smbclient -L //10.10.10.3/ --option='client min protocol=NT1'
````

#### List Shares:
````bash
smbmap -H 10.10.1.1
````
````bash
echo exit | smbclient -L \\\\10.10.10.10
````
````bash
nmap --script smb-enum-shares -p 139,445 10.10.10.10
````
#### Check for SMB Vulnerabilities with Nmap:
````bash
nmap --script smb-vuln* -p 139,445 10.10.10.10
````
#### Vulnerable Versions:
````bash
• Windows NT, 2000, and XP (most SMB1) - VULNERABLE: Null Sessions can be created by default
• Windows 2003, and XP SP2 onwards - NOT VULNERABLE: Null Sessions can't be created default
• Most Samba (Unix) servers
````
#### List of SMB versions and corresponding Windows versions:
````bash
• SMB1 – Windows 2000, XP and Windows 2003.
• SMB2 – Windows Vista SP1 and Windows 2008
• SMB2.1 – Windows 7 and Windows 2008 R2
• SMB3 – Windows 8 and Windows 2012.
````

# Web Application Enumeration / Exploitation - Port 80,443,8080
#### Make sure that you enumerate, and enumerate some more. :wink:
![image](https://user-images.githubusercontent.com/80599694/147911298-00e672ff-9bb8-475c-ac03-a00ab3cb170c.png)
## Web Application Enumeration Checklist:
````bash
1. Checkout the entire webpage and what it is displaying.
2. Read every page, look for emails, names, user info, etc.
3. Directory Discovery (time to dir bust!)
4. Enumerate the interface, what is the CMS & Version? Server installation page?
5. Check for potential Local File Inclusion, Remote File Inclusion, SQL Injection, XXE, and Upload vulnerabilities
6. Check for a default server page, identify the server version
7. View Source Code:
      a. Check for hidden values
      b. Check for comments/developer remarks
      c. Check for Extraneous Code
      d. Check for passwords
8. Check for robots.txt file
9. Web Scanning
````
### Directory Discovery/Dir Busting:
````bash
gobuster dir -u 10.10.10.181 -w /usr/share/seclists/Discovery/Web-Content/common.txt
````
#### Gobuster Quick Directory Discovery
````bash
gobuster -u $ip -w /usr/share/seclists/Discovery/Web_Content/common.txt -t 80 -a Linux
`````
#### Gobuster Directory Busting:
````bash
wget https://raw.githubusercontent.com/danielmiessler/SecLists/master/Discovery/Web-Content/Top1000-RobotsDisallowed.txt; gobuster -u http://10.10.10.10. -w Top1000-RobotsDisallowed.txt
````
````bash
gobuster dir -u http://$ip -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php -o gobuster-root -t 50
````
#### Gobuster comprehensive directory busting:
````bash
gobuster -s 200,204,301,302,307,403 -u 10.10.10.10 -w /usr/share/seclists/Discovery/Web_Content/big.txt -t 80 -a 'Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0'
````
#### Gobuster search with file extension:
````bash
gobuster -u 10.10.10.10 -w /usr/share/seclists/Discovery/Web_Content/common.txt -t 80 -a Linux -x .txt,.php
````
#### wfuzz search with files:
````bash
wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --sc 200 http://10.10.10.10/FUZZ
````
#### Erodir by PinkP4nther
````bash
./erodir -u http://10.10.10.10 -e /usr/share/wordlists/dirb/common.txt -t 20
````
#### dirsearch.py
````bash
cd /root/dirsearch; python3 dirsearch.py  -u http://10.10.10.10/ -e .php
````
#### If you are really stuck, run this:
````bash
for file in $(ls /usr/share/seclists/Discovery/Web-Content); do gobuster -u http://$ip/ -w /usr/share/seclists/Discovery/Web-Content/$file -e -k -l -s "200,204,301,302,307" -t 20 ; done
````
#### Check different extensions:
````bash
sh,txt,php,html,htm,asp,aspx,js,xml,log,json,jpg,jpeg,png,gif,doc,pdf,mpg,mp3,zip,tar.gz,tar
````

## SQL Injection Testing (automated!)
#### if you follow the above check list, you should have a list of parameters to test for SQL injection. Automate it with SQLMAP!

#### SQLmap Commands:
````bash
sqlmap -u http://10.10.10.10 -p parameter
sqlmap -u http://10.10.10.10  --data POSTstring -p parameter
sqlmap -u http://10.10.10.10 --os-shell
sqlmap -u http://10.10.10.10 --dump
````
# Password Cracking
#### I highly suggest you learn how to use John The Ripper, Hydra, and how to unshadow passwd files. :wink:
![image](https://user-images.githubusercontent.com/80599694/147914303-6417c637-a380-41d7-a716-76cbc6b4ff9e.png)

### Unshadow
#### This will prepare the file for John The Ripper, you need a Passwd & Shadow File.
````bash
unshadow passwd shadow > unshadow
````
### Hash Cracking - John The Ripper
````bash
john -wordlist /path/to/wordlist -users=users.txt hashfile
````

# Networking - Routing
#### I highly recommend that you get comfortable with general networking and routing concepts, including be able to read and understand .PCAP files.
![image](https://user-images.githubusercontent.com/80599694/147913167-35155f9d-f7f5-473e-90f9-302f0b5d7bb2.png)

### Set up IP Routing and Routing Tables
````bash
ip route - prints the routing table for the host you are on
ip route add ROUTETO via ROUTEFROM - add a route to a new network if on a switched network and you need to pivot
````

### ARP Spoofing
````bash
echo 1 > /proc/sys/net/ipv4/ip_forward
arpspoof -i tap0 -t 10.10.10.10 -r 10.10.10.11
````
### SSH Tunneling / Port Forwarding
````bash
# local port forwarding
# the target host 192.168.0.100 is running a service on port 8888
# and you want that service available on the localhost port 7777

ssh -L 7777:localhost:8888 user@192.168.0.100

# remote port forwarding
# you are running a service on localhost port 9999 
# and you want that service available on the target host 192.168.0.100 port 12340

ssh -R 12340:localhost:9999 user@192.168.0.100

# Local proxy through remote host
# You want to route network traffic through a remote host target.host
# so you create a local socks proxy on port 12001 and configure the SOCKS5 settings to localhost:12001

ssh -C2qTnN -D 12001 user@target.host
````
# Using Metasploit
#### I highly recommend getting comfortable with metasploit, and meterpreter just incase you find Remote Code Execution, and spawn a shell.
![image](https://user-images.githubusercontent.com/80599694/147913813-b8f42c43-3377-420f-ad8a-7524f50192ed.png)

### Basic Metasploit Commands
````bash
search x
use x
info
show options, show advanced options
SET X (e.g. set RHOST 10.10.10.10, set payload x)
````

### Useful Meterpreter Commands (reverse shell)
````bash
background
sessions -l
sessions -i 1
sysinfo, ifconfig, route, getuid
getsystem (privesc)
bypassuac
download x /root/
upload x C:\\Windows
shell
use post/windows/gather/hashdump
````
