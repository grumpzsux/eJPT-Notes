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
nmap --script smb-vuln* -p 139,445 $ip
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
