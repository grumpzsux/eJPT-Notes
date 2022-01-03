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
