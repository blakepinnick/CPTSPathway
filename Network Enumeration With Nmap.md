#tip
#function
#technique 
#syntax   
# Introduction
## Enumeration
**Enumeration is the most critical part of all**

***IT'S NOT HARD TO GET ACCESS TO THE TARGET SYSTEM ONCE WE KNOW HOW TO DO IT***

#tip **The ways to get access can be narrowed down to two points:**
- Functions and/or resources that allows us to interact with the target and/or provide additional information
- Information that provides us with even more important information to access our target

Misconfigurations are either the result of ignorance or a wrong security mindset

#tip *Enumeration is the key*

#tip *Most of the time its not the tools we haven't tried, but rather the fact that we don't know how to interact with the service and what's relevant*

#tip *Investing a couple hours learning more about the service, how it works, and what it is meant for can save hours or even days from a goal with a system*

#tip Manual Enumeration is **critical**, scanning tools simplify and accelerate the process...**HOWEVER**, these cannot always bypass the security measures of the services

e.g. 
`Most scanning tools have a timeout set until they receive a response from the service. If this tool does not respond within a specific time, this service/port will be marked as closed, filtered, or unknown. In the last two cases, we will still be able to work with it. However, if a port is marked as closed and Nmap doesn't show it to us, we will be in a bad situation. This service/port may provide us with the opportunity to find a way to access the system. Therefore, this result can take much unnecessary time until we find it.`

## Introduction to Nmap

### Use Cases
- Audit network security
- Simulate penetration tests
- Verify firewall/IDS configurations
- Network mapping
- Response analysis
- Identify open ports
- Vulnerability assessment
### Nmap Architecture #function
- **Host Discovery**
- **Port Scanning**
- **Service enumeration/detection**
- **OS Detection**
- **Scriptable Interaction**
### Syntax 
nmap (scan types) (options) (target)

### Scan Techniques #function
```shell-session
SCAN TECHNIQUES:
  -sS/sT/sA/sW/sM: TCP SYN/Connect()/ACK/Window/Maimon scans
  -sU: UDP Scan
  -sN/sF/sX: TCP Null, FIN, and Xmas scans
  --scanflags <flags>: Customize TCP scan flags
  -sI <zombie host[:probeport]>: Idle scan
  -sY/sZ: SCTP INIT/COOKIE-ECHO scans
  -sO: IP protocol scan
  -b <FTP relay host>: FTP bounce scan
```

-sS is default
#tip
- If our target sends a `SYN-ACK` flagged packet back to us, Nmap detects that the port is `open`.
- If the target responds with an `RST` flagged packet, it is an indicator that the port is `closed`.
- If Nmap does not receive a packet back, it will display it as `filtered`. Depending on the firewall configuration, certain packets may be dropped or ignored by the firewall.
# Host Enumeration
## Host Discovery

*it's recommend to **store** every single scan for comparison, documentation, and reporting*

### Scan Network Range #technique #function #tip
```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5

10.129.2.4
10.129.2.10
10.129.2.11
10.129.2.18
10.129.2.19
10.129.2.20
10.129.2.28
```

|**Scanning Options**|**Description**|
|---|---|
|`10.129.2.0/24`|Target network range.|
|`-sn`|Disables port scanning.|
|`-oA tnet`|Stores the results in all formats starting with the name 'tnet'.|

^this scanning method only works if the firewalls of the hosts allow it^

### Scan IP List #technique

```shell-session
mricognito@htb[/htb]$ cat hosts.lst

10.129.2.4
10.129.2.10
10.129.2.11
10.129.2.18
10.129.2.19
10.129.2.20
10.129.2.28
```

```shell-session
mricognito@htb[/htb]$ sudo nmap -sn -oA tnet -iL hosts.lst | grep for | cut -d" " -f5

10.129.2.18
10.129.2.19
10.129.2.20
```

| **Scanning Options** | **Description**                                                      |
| -------------------- | -------------------------------------------------------------------- |
| `-sn`                | Disables port scanning.                                              |
| `-oA tnet`           | Stores the results in all formats starting with the name 'tnet'.     |
| `-iL`                | Performs defined scans against targets in provided 'hosts.lst' list. |

### Scan Multiple IPs #technique 
```shell-session
mricognito@htb[/htb]$ sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20| grep for | cut -d" " -f5

10.129.2.18
10.129.2.19
10.129.2.20
```

```shell-session
mricognito@htb[/htb]$ sudo nmap -sn -oA tnet 10.129.2.18-20| grep for | cut -d" " -f5

10.129.2.18
10.129.2.19
10.129.2.20
```

^.18-20 is scanning  the three IPs within that range^

### Scan Single IP #technique
```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.18 -sn -oA host 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-14 23:59 CEST
Nmap scan report for 10.129.2.18
Host is up (0.087s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.11 seconds
```

|**Scanning Options**|**Description**|
|---|---|
|`10.129.2.18`|Performs defined scans against the target.|
|`-sn`|Disables port scanning.|
|`-oA host`|Stores the results in all formats starting with the name 'host'.|
*If port scanning is disabled with -sn, Nmap automatically ping scans with ICMP Echo Requests (-PE)*

**--packet-trace** command #tip 
```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 00:08 CEST
SENT (0.0074s) ARP who-has 10.129.2.18 tell 10.10.14.2
RCVD (0.0309s) ARP reply 10.129.2.18 is-at DE:AD:00:00:BE:EF
Nmap scan report for 10.129.2.18
Host is up (0.023s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.05 seconds
```

**--reason** command #tip 
```shell-session
SENT (0.0074s) ARP who-has 10.129.2.18 tell 10.10.14.2
RCVD (0.0309s) ARP reply 10.129.2.18 is-at DE:AD:00:00:BE:EF
```

**--disable-arp-ping** command #tip 
```shell-session
SENT (0.0107s) ICMP [10.10.14.2 > 10.129.2.18 Echo request (type=8/code=0) id=13607 seq=0] IP [ttl=255 id=23541 iplen=28 ]
RCVD (0.0152s) ICMP [10.129.2.18 > 10.10.14.2 Echo reply (type=0/code=0) id=13607 seq=0] IP [ttl=128 id=40622 iplen=28 ]
```

## Host and Port Scanning

### 6 Different States for a Scanned Port #function 

|**State**|**Description**|
|---|---|
|`open`|This indicates that the connection to the scanned port has been established. These connections can be **TCP connections**, **UDP datagrams** as well as **SCTP associations**.|
|`closed`|When the port is shown as closed, the TCP protocol indicates that the packet we received back contains an `RST` flag. This scanning method can also be used to determine if our target is alive or not.|
|`filtered`|Nmap cannot correctly identify whether the scanned port is open or closed because either no response is returned from the target for the port or we get an error code from the target.|
|`unfiltered`|This state of a port only occurs during the **TCP-ACK** scan and means that the port is accessible, but it cannot be determined whether it is open or closed.|
|`open\|filtered`|If we do not get a response for a specific port, `Nmap` will set it to that state. This indicates that a firewall or packet filter may protect the port.|
|`closed\|filtered`|This state only occurs in the **IP ID idle** scans and indicates that it was impossible to determine if the scanned port is closed or filtered by a firewall.|
### Scanning Top 10 TCP Ports #technique
**--top-ports=10** command
```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.28 --top-ports=10 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:36 CEST
Nmap scan report for 10.129.2.28
Host is up (0.021s latency).

PORT     STATE    SERVICE
21/tcp   closed   ftp
22/tcp   open     ssh
23/tcp   closed   telnet
25/tcp   open     smtp
80/tcp   open     http
110/tcp  open     pop3
139/tcp  filtered netbios-ssn
443/tcp  closed   https
445/tcp  filtered microsoft-ds
3389/tcp closed   ms-wbt-server
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 1.44 seconds
```

**-F** command #technique
	top 100 ports
**-n** command
	disables DNS Resolution
### Connect Scan -sT #technique 
- is highly accurate because it completes the three-way TCP handshake
- is **NOT** the most stealthy, one of the least stealthy
- Useful when accuracy is a priority
- useful for bypassing the firewall if the firewall allows outgoing packets
- slower

### Filtered Ports
- Either **dropped** or **rejected**
- --max-retries is **set** to 10
- -Pn: deactivates the ICMP echo requests
- -n: deactivate DNS resolution
- --disable-arp-ping: deactivate ARP ping scan

### Discovering Open UDP Ports
- -sU: UDP scan, stateless, much longer than TCP
- -sS: TCP scan
- -F: Scans top 100 ports
```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.28 -F -sU

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:01 CEST
Nmap scan report for 10.129.2.28
Host is up (0.059s latency).
Not shown: 95 closed ports
PORT     STATE         SERVICE
68/udp   open|filtered dhcpc
137/udp  open          netbios-ns
138/udp  open|filtered netbios-dgm
631/udp  open|filtered ipp
5353/udp open          zeroconf
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 98.07 seconds
```
- If the port is open, we only get a response if the application is configured to do so

-  -sU: Performs a UDP scan
- -Pn: Disables ICMP Echo requests
-  -n: Disables DNS resolution
-  --disable-arp-ping: Disables ARP ping
-  --packet-trace: Shows all packets sent and received
-  -p 137: Scans only the specified port
-  --reason: Displays the reason a port is in a particular state

### Version Scan
```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.28 -Pn -n --disable-arp-ping --packet-trace -p 445 --reason  -sV

Starting Nmap 7.80 ( https://nmap.org ) at 2022-11-04 11:10 GMT
SENT (0.3426s) TCP 10.10.14.2:44641 > 10.129.2.28:445 S ttl=55 id=43401 iplen=44  seq=3589068008 win=1024 <mss 1460>
RCVD (0.3556s) TCP 10.129.2.28:445 > 10.10.14.2:44641 SA ttl=63 id=0 iplen=44  seq=2881527699 win=29200 <mss 1337>
NSOCK INFO [0.4980s] nsock_iod_new2(): nsock_iod_new (IOD #1)
NSOCK INFO [0.4980s] nsock_connect_tcp(): TCP connection requested to 10.129.2.28:445 (IOD #1) EID 8
NSOCK INFO [0.5130s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 8 [10.129.2.28:445]
Service scan sending probe NULL to 10.129.2.28:445 (tcp)
NSOCK INFO [0.5130s] nsock_read(): Read request from IOD #1 [10.129.2.28:445] (timeout: 6000ms) EID 18
NSOCK INFO [6.5190s] nsock_trace_handler_callback(): Callback: READ TIMEOUT for EID 18 [10.129.2.28:445]
Service scan sending probe SMBProgNeg to 10.129.2.28:445 (tcp)
NSOCK INFO [6.5190s] nsock_write(): Write request for 168 bytes to IOD #1 EID 27 [10.129.2.28:445]
NSOCK INFO [6.5190s] nsock_read(): Read request from IOD #1 [10.129.2.28:445] (timeout: 5000ms) EID 34
NSOCK INFO [6.5190s] nsock_trace_handler_callback(): Callback: WRITE SUCCESS for EID 27 [10.129.2.28:445]
NSOCK INFO [6.5320s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 34 [10.129.2.28:445] (135 bytes)
Service scan match (Probe SMBProgNeg matched with SMBProgNeg line 13836): 10.129.2.28:445 is netbios-ssn.  Version: |Samba smbd|3.X - 4.X|workgroup: WORKGROUP|
NSOCK INFO [6.5320s] nsock_iod_delete(): nsock_iod_delete (IOD #1)
Nmap scan report for 10.129.2.28
Host is up, received user-set (0.013s latency).

PORT    STATE SERVICE     REASON         VERSION
445/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
Service Info: Host: Ubuntu

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.55 seconds
```

- -Pn: Disables ICMP Echo requests
-  -n: Disables DNS resolution
-  --disable-arp-ping: Disables ARP ping
-  --packet-trace: Shows all packets sent and received
-  -p 445: Scans only the specified port
-  --reason: Displays the reason a port is in a particular state
-  -sV: Performs a service scan

### Activity #activity
 1. Find all TCP ports on your target. Submit the total number of found TCP ports as the answer.

==***7***==

 ```
 ┌──(kali㉿kali)-[~]
└─$ nmap -p- -sS 10.129.2.49 -T5 -v
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-07 23:37 EDT
Initiating Ping Scan at 23:37
Scanning 10.129.2.49 [4 ports]
Completed Ping Scan at 23:37, 0.08s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 23:37
Completed Parallel DNS resolution of 1 host. at 23:37, 0.03s elapsed
Initiating SYN Stealth Scan at 23:37
Scanning 10.129.2.49 [65535 ports]
Discovered open port 80/tcp on 10.129.2.49
Discovered open port 143/tcp on 10.129.2.49
Discovered open port 139/tcp on 10.129.2.49
Discovered open port 22/tcp on 10.129.2.49
Discovered open port 110/tcp on 10.129.2.49
Discovered open port 445/tcp on 10.129.2.49
Discovered open port 31337/tcp on 10.129.2.49
SYN Stealth Scan Timing: About 38.17% done; ETC: 23:39 (0:00:50 remaining)
Warning: 10.129.2.49 giving up on port because retransmission cap hit (2).
Completed SYN Stealth Scan at 23:39, 85.90s elapsed (65535 total ports)
Nmap scan report for 10.129.2.49
Host is up (0.049s latency).
Not shown: 64985 closed tcp ports (reset), 543 filtered tcp ports (no-response)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
110/tcp   open  pop3
139/tcp   open  netbios-ssn
143/tcp   open  imap
445/tcp   open  microsoft-ds
31337/tcp open  Elite

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 86.12 seconds
           Raw packets sent: 70404 (3.098MB) | Rcvd: 68610 (2.744MB)
```

2.  Enumerate the hostname of your target and submit it as the answer. (case-sensitive)

==***NIX-NMAP-DEFAULT***==

```
┌──(kali㉿kali)-[~]
└─$ nmap -p 22,80,110,139,143,445,31337 -sC 10.129.2.49
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-07 23:52 EDT
Nmap scan report for 10.129.2.49
Host is up (0.049s latency).

PORT      STATE SERVICE
22/tcp    open  ssh
| ssh-hostkey: 
|   2048 71:c1:89:90:7f:fd:4f:60:e0:54:f3:85:e6:35:6c:2b (RSA)
|   256 e1:8e:53:18:42:af:2a:de:c0:12:1e:2e:54:06:4f:70 (ECDSA)
|_  256 1a:cc:ac:d4:94:5c:d6:1d:71:e7:39:de:14:27:3c:3c (ED25519)
80/tcp    open  http
|_http-title: Apache2 Ubuntu Default Page: It works
110/tcp   open  pop3
|_pop3-capabilities: CAPA UIDL RESP-CODES TOP PIPELINING AUTH-RESP-CODE SASL
139/tcp   open  netbios-ssn
143/tcp   open  imap
|_imap-capabilities: have SASL-IR IDLE ENABLE more IMAP4rev1 LOGINDISABLEDA0001 post-login listed Pre-login LITERAL+ capabilities OK ID LOGIN-REFERRALS
445/tcp   open  microsoft-ds
31337/tcp open  Elite

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: nix-nmap-default


|   NetBIOS computer name: NIX-NMAP-DEFAULT\x00


|   Domain name: \x00
|   FQDN: nix-nmap-default
|_  System time: 2025-06-08T05:52:25+02:00
|_clock-skew: mean: -39m59s, deviation: 1h09m16s, median: 0s
|_nbstat: NetBIOS name: NIX-NMAP-DEFAUL, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-time: 
|   date: 2025-06-08T03:52:25
|_  start_date: N/A

Nmap done: 1 IP address (1 host up) scanned in 38.43 seconds

```

=
## Saving the Results
### Different Formats #function
- Nmap can save the results in 3 **different** formats
	- - Normal output (`-oN`) with the `.nmap` file extension
	- Grepable output (`-oG`) with the `.gnmap` file extension
	- XML output (`-oX`) with the `.xml` file extension
	- All formats (-oA) 
```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.28 -p- -oA target

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-16 12:14 CEST
Nmap scan report for 10.129.2.28
Host is up (0.0091s latency).
Not shown: 65525 closed ports
PORT      STATE SERVICE
22/tcp    open  ssh
25/tcp    open  smtp
80/tcp    open  http
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 10.22 seconds
```

### Style Sheets #tip
With the XML output, we can easily create HTML reports that are easy to read, even for non technical people.

Very useful for documentation, presents information in a clear way.
	*To convert the stored results from XML format to HTML, we can use the tool **xsltproc***
#technique
```shell-session
mricognito@htb[/htb]$ xsltproc target.xml -o target.html
```

Opening the HTML file in our browser, we see a clear and structured presentation of our results:
![[Pasted image 20250608004211.png]]

### Activity #activity
Perform a full TCP port scan on your target and create an HTML report. Submit the number of the highest port as the answer.

```
nmap -sT -oX temp 10.129.2.49

Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-08 01:02 EDT
Nmap scan report for 10.129.2.49
Host is up (0.051s latency).
Not shown: 993 closed tcp ports (conn-refused)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
110/tcp   open  pop3
139/tcp   open  netbios-ssn
143/tcp   open  imap
445/tcp   open  microsoft-ds
31337/tcp open  Elite
```

```
xsltproc temp -o temp.html
```

![[Pasted image 20250608010831.png]]
## Service Enumeration
### Service Version Detection
- #tip To view the scan status, we can press the [Space bar] during the scan, which will have Nmap show the scan status
- #tip Define periods of time the status should be shown (s or m) `--stats-every=5s`
- (-v/-vv) increase the verbosity level, showing us the open ports directly when Nmap detects them

### Banner Grabbing
- -sV: is using signatures
- slows down Nmap
- Sometimes misses information

```bash
sudo nmap 10.129.2.28 -p- -sV
```

**Output:**

- **Host**: 10.129.2.28 (up, 0.013s latency)
    
- **Ports**:
    
    - 22/tcp: OpenSSH 7.6p1 Ubuntu 4ubuntu0.3
        
    - 25/tcp: Postfix smtpd
        
    - 80/tcp: Apache httpd 2.4.29 (Ubuntu)
        
    - 110/tcp: Dovecot pop3d
        
    - 139/tcp: Filtered (netbios-ssn)
        
    - 143/tcp: Dovecot imapd (Ubuntu)
        
    - 445/tcp: Filtered (microsoft-ds)
        
    - 993/tcp: Dovecot imapd (SSL, Ubuntu)
        
    - 995/tcp: Dovecot pop3d (SSL)
        
- **MAC**: DE:AD:00:00:BE:EF (Intel)
    
- **OS**: Linux
    
- **Host**: inlane
**Scanning Options**:

- 10.129.2.28: Target
    
- -p-: All ports
    
- -sV: Service version detection

**Advanced Scan**
```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.28 -p- -sV -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-16 20:10 CEST
<SNIP>
NSOCK INFO [0.4200s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 18 [10.129.2.28:25] (35 bytes): 220 inlane ESMTP Postfix (Ubuntu)..
Service scan match (Probe NULL matched with NULL line 3104): 10.129.2.28:25 is smtp.  Version: |Postfix smtpd|||
NSOCK INFO [0.4200s] nsock_iod_delete(): nsock_iod_delete (IOD #1)
Nmap scan report for 10.129.2.28
Host is up (0.076s latency).

PORT   STATE SERVICE VERSION
25/tcp open  smtp    Postfix smtpd
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Service Info: Host:  inlane

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.47 seconds
```


|**Scanning Options**|**Description**|
|---|---|
|`10.129.2.28`|Scans the specified target.|
|`-p-`|Scans all ports.|
|`-sV`|Performs service version detection on specified ports.|
|`-Pn`|Disables ICMP Echo requests.|
|`-n`|Disables DNS resolution.|
|`--disable-arp-ping`|Disables ARP ping.|
|`--packet-trace`|Shows all packets sent and received.|
#### Using manual banner grabbing to get more information

**tcpdump**
```
sudo tcpdump -i eth0 host 10.10.14.2 and 10.129.2.28
```

**bash**
```
nc -nv 10.129.2.28 25
```

**output:**
```shell-session
mricognito@htb[/htb]$  nc -nv 10.129.2.28 25

Connection to 10.129.2.28 port 25 [tcp/*] succeeded!
220 inlane ESMTP Postfix (Ubuntu)
```

### Activity #activity 
Target(s): 10.129.2.49 (ACADEMY-NMAP-DEFAULT)   
+ 1  Enumerate all ports and their services. One of the services contains the flag you have to submit as the answer.
```
nmap 10.129.2.49 -T5
```

```
┌──(kali㉿kali)-[~]
└─$ nmap 10.129.2.49 -T5
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-08 08:29 EDT
Note: Host seems down. If it is really up, but blocking our ping probes, try -Pn
Nmap done: 1 IP address (0 hosts up) scanned in 1.58 seconds
```

```
┌──(kali㉿kali)-[~]
└─$ nmap 10.129.2.49 -T5 -Pn
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-08 08:29 EDT
Stats: 0:00:25 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 49.50% done; ETC: 08:30 (0:00:27 remaining)
Stats: 0:00:29 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 57.00% done; ETC: 08:30 (0:00:23 remaining)
Stats: 0:00:38 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 99.99% done; ETC: 08:30 (0:00:00 remaining)
Nmap scan report for 10.129.2.49
Host is up (0.045s latency).
Not shown: 993 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
110/tcp   open  pop3
139/tcp   open  netbios-ssn
143/tcp   open  imap
445/tcp   open  microsoft-ds
31337/tcp open  Elite

Nmap done: 1 IP address (1 host up) scanned in 40.76 seconds
```

```
┌──(kali㉿kali)-[~]
└─$ nmap 10.129.2.49 -p 22,80,110,139,143,445,31337 -T5 -Pn -sV 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-08 08:31 EDT
Nmap scan report for 10.129.2.49
Host is up (0.044s latency).

PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http        Apache httpd 2.4.29 ((Ubuntu))
110/tcp   open  pop3        Dovecot pop3d
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp   open  imap        Dovecot imapd (Ubuntu)
445/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
31337/tcp open  ftp         ProFTPD
Service Info: Host: NIX-NMAP-DEFAULT; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 51.83 seconds
```

```
┌──(kali㉿kali)-[~]
└─$ nc -nv 10.129.2.49 31337
(UNKNOWN) [10.129.2.49] 31337 (?) open
220 HTB{pr0F7pDv3r510nb4nn3r}
```

`HTB{pr0F7pDv3r510nb4nn3r}` is my flag

## Nmap Scripting Engine #function

**Nmap Scripting Engine (NSE)**
- Provides us with the possibility to create scripts in Lua for interaction with certain services. 

*There are 14 categories into which these scripts can be divided:*

|**Category**|**Description**|
|---|---|
|`auth`|Determination of authentication credentials.|
|`broadcast`|Scripts, which are used for host discovery by broadcasting and the discovered hosts, can be automatically added to the remaining scans.|
|`brute`|Executes scripts that try to log in to the respective service by brute-forcing with credentials.|
|`default`|Default scripts executed by using the `-sC` option.|
|`discovery`|Evaluation of accessible services.|
|`dos`|These scripts are used to check services for denial of service vulnerabilities and are used less as it harms the services.|
|`exploit`|This category of scripts tries to exploit known vulnerabilities for the scanned port.|
|`external`|Scripts that use external services for further processing.|
|`fuzzer`|This uses scripts to identify vulnerabilities and unexpected packet handling by sending different fields, which can take much time.|
|`intrusive`|Intrusive scripts that could negatively affect the target system.|
|`malware`|Checks if some malware infects the target system.|
|`safe`|Defensive scripts that do not perform intrusive and destructive access.|
|`version`|Extension for service detection.|
|`vuln`|Identification of specific vulnerabilities.|
### Default Scripts
```shell-session
mricognito@htb[/htb]$ sudo nmap <target> -sC
```

### Specific Scripts Category #technique #tip
```shell-session
mricognito@htb[/htb]$ sudo nmap <target> --script <category>
```

### Defined Scripts
```shell-session
mricognito@htb[/htb]$ sudo nmap <target> --script <script-name>,<script-name>,...
```

### e.g. Nmap - Specifying Scripting #important
 
```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.28 -p 25 --script banner,smtp-commands

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-16 23:21 CEST
Nmap scan report for 10.129.2.28
Host is up (0.050s latency).

PORT   STATE SERVICE
25/tcp open  smtp
|_banner: 220 inlane ESMTP Postfix (Ubuntu)
|_smtp-commands: inlane, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8,
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
```

- The banner script gives us the Ubunutu distribution of Linux
- The smtp-commands script shows us which commands we can use by interacting with the target SMTP server

### Nmap - Aggressive Scan
```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.28 -p 80 -A
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-17 01:38 CEST
Nmap scan report for 10.129.2.28
Host is up (0.012s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.3.4
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: blog.inlanefreight.com
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.32 (96%), Linux 3.2 - 4.9 (96%), Linux 2.6.32 - 3.10 (96%), Linux 3.4 - 3.10 (95%), Linux 3.1 (95%), Linux 3.2 (95%), 
AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Synology DiskStation Manager 5.2-5644 (94%), Netgear RAIDiator 4.2.28 (94%), 
Linux 2.6.32 - 2.6.35 (94%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop

TRACEROUTE
HOP RTT      ADDRESS
1   11.91 ms 10.129.2.28

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.36 seconds
```

- This scans the target with multiple options as service detection (`-sV`), OS detection (`-O`), traceroute (`--traceroute`), and with the default NSE scripts (`-sC`).
*With the help of the used scan option (`-A`), we found out what kind of web server (`Apache 2.4.29`) is running on the system, which web application (`WordPress 5.3.4`) is used, and the title (`blog.inlanefreight.com`) of the web page. Also, `Nmap` shows that it is likely to be `Linux` (`96%`) operating system.*

### Vulnerability Assessment #important #function #tip

**Nmap - Vuln Category**
```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.28 -p 80 -sV --script vuln 

Nmap scan report for 10.129.2.28
Host is up (0.036s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-enum:
|   /wp-login.php: Possible admin folder
|   /readme.html: Wordpress version: 2
|   /: WordPress version: 5.3.4
|   /wp-includes/images/rss.png: Wordpress version 2.2 found.
|   /wp-includes/js/jquery/suggest.js: Wordpress version 2.5 found.
|   /wp-includes/images/blank.gif: Wordpress version 2.6 found.
|   /wp-includes/js/comment-reply.js: Wordpress version 2.7 found.
|   /wp-login.php: Wordpress login page.
|   /wp-admin/upgrade.php: Wordpress login page.
|_  /readme.html: Interesting, a readme.
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-wordpress-users:
| Username found: admin
|_Search stopped at ID #25. Increase the upper limit if necessary with 'http-wordpress-users.limit'
| vulners:
|   cpe:/a:apache:http_server:2.4.29:
|     	CVE-2019-0211	7.2	https://vulners.com/cve/CVE-2019-0211
|     	CVE-2018-1312	6.8	https://vulners.com/cve/CVE-2018-1312
|     	CVE-2017-15715	6.8	https://vulners.com/cve/CVE-2017-15715
<SNIP>
```

|**Scanning Options**|**Description**|
|---|---|
|`10.129.2.28`|Scans the specified target.|
|`-p 80`|Scans only the specified port.|
|`-sV`|Performs service version detection on specified ports.|
|`--script vuln`|Uses all related scripts from specified category.|

### Activity #activity 
Target(s): 10.129.2.49 (ACADEMY-NMAP-DEFAULT)
 Use NSE and its scripts to find the flag that one of the services contain and submit it as the answer.

```
nmap 10.129.2.49 -Pn
```

```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-08 08:54 EDT
Nmap scan report for 10.129.2.49
Host is up (0.052s latency).
Not shown: 993 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
110/tcp   open  pop3
139/tcp   open  netbios-ssn
143/tcp   open  imap
445/tcp   open  microsoft-ds
31337/tcp open  Elite
Nmap done: 1 IP address (1 host up) scanned in 0.91 seconds      
```

```
┌──(kali㉿kali)-[~]│
└─$ nmap 10.129.2.49 -p 563,617,666,1031,1091,1234,2605,3809,5800,6129 --script vuln -Pn│
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-08 08:53 EDT│
Nmap scan report for 10.129.2.49│
Host is up (0.047s latency).│
│
PORT     STATE  SERVICE│
563/tcp  closed snews│
617/tcp  closed sco-dtmgr│
666/tcp  closed doom│
1031/tcp closed iad2│
1091/tcp closed ff-sm│
1234/tcp closed hotline│
2605/tcp closed bgpd│
3809/tcp closed apocd│
5800/tcp closed vnc-http│
6129/tcp closed unknown│

Nmap done: 1 IP address (1 host up) scanned in 10.33 seconds
```
- Ports are now showing closed against the script

I ran it again and now the ports show filtered
```
┌──(kali㉿kali)-[~]                                                                                                                     │
└─$ nmap 10.129.2.49 -p 563,617,666,1031,1091,1234,2605,3809,5800,6129 --script vuln -Pn -v                                             │
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-08 08:56 EDT│
NSE: Loaded 105 scripts for scanning.│
NSE: Script Pre-scanning.│
Initiating NSE at 08:56│
Completed NSE at 08:56, 10.00s elapsed│
Initiating NSE at 08:56│
Completed NSE at 08:56, 0.00s elapsed│
Initiating Parallel DNS resolution of 1 host. at 08:56│
Completed Parallel DNS resolution of 1 host. at 08:56, 0.03s elapsed│
Initiating SYN Stealth Scan at 08:56│
Scanning 10.129.2.49 [10 ports]│
Completed SYN Stealth Scan at 08:56, 3.03s elapsed (10 total ports)│
NSE: Script scanning 10.129.2.49.│
Initiating NSE at 08:56│
Completed NSE at 08:56, 1.00s elapsed│
Initiating NSE at 08:56│
Completed NSE at 08:56, 0.00s elapsed│
Nmap scan report for 10.129.2.49│
Host is up.│
│
PORT     STATE    SERVICE│
563/tcp  filtered snews│
617/tcp  filtered sco-dtmgr│
666/tcp  filtered doom│
1031/tcp filtered iad2│
1091/tcp filtered ff-sm│
1234/tcp filtered hotline│
2605/tcp filtered bgpd│
3809/tcp filtered apocd│
5800/tcp filtered vnc-http│
6129/tcp filtered unknown│
│
NSE: Script Post-scanning.│
Initiating NSE at 08:56│
Completed NSE at 08:56, 0.00s elapsed│
Initiating NSE at 08:56│
Completed NSE at 08:56, 0.00s elapsed│
Read data files from: /usr/share/nmap│
Nmap done: 1 IP address (1 host up) scanned in 14.21 seconds│
           Raw packets sent: 20 (880B) | Rcvd: 0 (0B)    
```

```
┌──(kali㉿kali)-[~]
└─$ cat temp1
# Nmap 7.95 scan initiated Sun Jun  8 09:04:47 2025 as: /usr/lib/nmap/nmap --privileged -p 563,617,666,1031,1091,1234,2605,3809,5800,6129 --script discovery -Pn -oN temp1 10.129.2.49                                                                                             
Pre-scan script results:
| targets-asn:
|_  targets-asn.asn is a mandatory parameter
| broadcast-ping:
|   IP: 192.168.40.2  MAC: 00:50:56:f1:00:69
|_  Use --script-args=newtargets to add the results as targets
| broadcast-igmp-discovery:
|   192.168.40.1
|     Interface: eth0
|     Version: 2
|     Group: 224.0.0.251
|     Description: mDNS (rfc6762)
|   192.168.40.1
|     Interface: eth0
|     Version: 2
|     Group: 224.0.0.252
|     Description: Link-local Multicast Name Resolution (rfc4795)
|   192.168.40.1
|     Interface: eth0
|     Version: 2
|     Group: 239.255.255.250
|     Description: Organization-Local Scope (rfc2365)
|_  Use the newtargets script-arg to add the results as targets
| targets-ipv6-multicast-slaac: 
|   IP: fe80::f55e:e2bb:682e:c9f9  MAC: 00:50:56:c0:00:08  IFACE: eth0
|   IP: fe80::6cd9:a9a4:151f:8793  MAC: 00:50:56:c0:00:08  IFACE: eth0
|_  Use --script-args=newtargets to add the results as targets
| ipv6-multicast-mld-list: 
|   fe80::ee5f:3c07:2ed2:54e4: 
|     device: eth0
|     mac: 00:50:56:c0:00:08
|     multicast_ips: 
|       ff02::c                   (SSDP)
|       ff02::1:3                 (Link-local Multicast Name Resolution)
|       ff02::1:3                 (Link-local Multicast Name Resolution)
|       ff02::1:3                 (Link-local Multicast Name Resolution)
|       ff02::1:3                 (Link-local Multicast Name Resolution)
|       ff02::1:3                 (Link-local Multicast Name Resolution)
|       ff02::1:3                 (Link-local Multicast Name Resolution)
|       ff02::1:ff2e:c9f9         (Solicited-Node Address)
|       ff02::1:ff1f:8793         (Solicited-Node Address)
|       ff02::fb                  (mDNSv6)
|       ff02::1:ffd2:54e4         (NDP Solicited-node)
|_      ff02::1:3                 (Link-local Multicast Name Resolution)
|_hostmap-robtex: *TEMPORARILY DISABLED* due to changes in Robtex's API. See https://www.robtex.com/api/
|_multicast-profinet-discovery: 0
|_http-robtex-shared-ns: *TEMPORARILY DISABLED* due to changes in Robtex's API. See https://www.robtex.com/api/
| targets-ipv6-multicast-mld: 
|   IP: fe80::ee5f:3c07:2ed2:54e4  MAC: 00:50:56:c0:00:08  IFACE: eth0
| 

```

After this I started just going down this list:

| **Category** | **Description**                                                                                                                         |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
| `auth`       | Determination of authentication credentials.                                                                                            |
| `broadcast`  | Scripts, which are used for host discovery by broadcasting and the discovered hosts, can be automatically added to the remaining scans. |
| `brute`      | Executes scripts that try to log in to the respective service by brute-forcing with credentials.                                        |
| `default`    | Default scripts executed by using the `-sC` option.                                                                                     |
| `discovery`  | Evaluation of accessible services.                                                                                                      |
| `dos`        | These scripts are used to check services for denial of service vulnerabilities and are used less as it harms the services.              |
| `exploit`    | This category of scripts tries to exploit known vulnerabilities for the scanned port.                                                   |
| `external`   | Scripts that use external services for further processing.                                                                              |
| `fuzzer`     | This uses scripts to identify vulnerabilities and unexpected packet handling by sending different fields, which can take much time.     |
| `intrusive`  | Intrusive scripts that could negatively affect the target system.                                                                       |
| `malware`    | Checks if some malware infects the target system.                                                                                       |
| `safe`       | Defensive scripts that do not perform intrusive and destructive access.                                                                 |
| `version`    | Extension for service detection.                                                                                                        |
| `vuln`       | Identification of specific vulnerabilities.                                                                                             |
There is a web server on 5800, i'm going to start targeting that one.

Nmap doesn't seem to like when you load multiple scripts to go against multiple ports

I kept getting filtered as a state until I used this command
`sudo nmap 10.129.2.49 -p 5800 -sA`

I can't get any scripts to work with -sA

I went back and look, I don't know how I messed this up but I was looking at the wrong port list. I'm now going to run everything against the web server on port 80.

I ran a script against the port and now the port is filtered, it was open

```
┌──(kali㉿kali)-[~]│
└─$ sudo nmap 10.129.2.49 -p 80 -Pn --script vuln│
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-08 10:16 EDT│
Nmap scan report for 10.129.2.49│
Host is up.│
│
PORT   STATE    SERVICE│
80/tcp filtered http│
│
Nmap done: 1 IP address (1 host up) scanned in 13.20 seconds  
```
I ran this command and it finally worked
```
┌──(kali㉿kali)-[~]│
└─$ sudo nmap 10.129.142.6 -p 80 -Pn -sV --script vuln 
```

	it seems like -sV is causing the scripts to work

- going to run all the scripts against port 80
After scanning a bunch with different scripts I decided to check out the /robots.txt that I've seen multiple times

```
┌──(kali㉿kali)-[~]
└─$ curl http://10.129.142.6/robots.txt                                             
User-agent: *

Allow: /

HTB{873nniuc71bu6usbs1i96as6dsv26}
```

Unfortunately I thought the flag was going to be contained within the results of the Nmap scans, I should I have just checked this the first time I saw it.

## Performance #function #tip 
- How fast (-T 0-5)
- Which frequency (--min-parallelism *number*)
- Which timeouts (--max-rtt-timeout *time*) the test packets should have
- How many packets should be sent simultaneously (--min-rate *number*)
- Number of retries (--max-retries **number**)
### Timeouts
When Nmap sends a packet it takes time to receive a response from the scanned port
- **Round-Trip-Time - RTT**
- Generally --min-RTT-timeout is 100ms

#### Default Scan

```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.0/24 -F

<SNIP>
Nmap done: 256 IP addresses (10 hosts up) scanned in 39.44 seconds
```

#### Optimized RTT #function

```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.0/24 -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms

<SNIP>
Nmap done: 256 IP addresses (8 hosts up) scanned in 12.29 seconds
```

| **Scanning Options**         | **Description**                                       |
| ---------------------------- | ----------------------------------------------------- |
| `10.129.2.0/24`              | Scans the specified target network.                   |
| `-F`                         | Scans top 100 ports.                                  |
| `--initial-rtt-timeout 50ms` | Sets the specified time value as initial RTT timeout. |
| `--max-rtt-timeout 100ms`    | Sets the specified time value as maximum RTT timeout. |

- setting the initial RTT timeout too short a time period may cause us to overlook hosts #tip
### Max Retries #function

Another way to increase scan speed is by specifying the retry rate of sent packets (`--max-retries`). The default value is `10`, but we can reduce it to `0`. This means if Nmap does not receive a response for a port, it won't send any more packets to that port and will skip it.

#### Default Scan

```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.0/24 -F | grep "/tcp" | wc -l

23
```

#### Reduced Retries

```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.0/24 -F --max-retries 0 | grep "/tcp" | wc -l

21
```

|**Scanning Options**|**Description**|
|---|---|
|`10.129.2.0/24`|Scans the specified target network.|
|`-F`|Scans top 100 ports.|
|`--max-retries 0`|Sets the number of retries that will be performed during the scan.|
- once again, accelerating can also have a negative effect on our results, which means we can overlook important information

### Rates
we can set the minimum rate with: --min-rate (number)

#### Default Scan


```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.0/24 -F -oN tnet.default

<SNIP>
Nmap done: 256 IP addresses (10 hosts up) scanned in 29.83 seconds
```

#### Optimized Scan


```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.0/24 -F -oN tnet.minrate300 --min-rate 300

<SNIP>
Nmap done: 256 IP addresses (10 hosts up) scanned in 8.67 seconds
```

|**Scanning Options**|**Description**|
|---|---|
|`10.129.2.0/24`|Scans the specified target network.|
|`-F`|Scans top 100 ports.|
|`-oN tnet.minrate300`|Saves the results in normal formats, starting the specified file name.|
|`--min-rate 300`|Sets the minimum number of packets to be sent per second.|

#### Default Scan - Found Open Ports


```shell-session
mricognito@htb[/htb]$ cat tnet.default | grep "/tcp" | wc -l

23
```

#### Optimized Scan - Found Open Ports


```shell-session
mricognito@htb[/htb]$ cat tnet.minrate300 | grep "/tcp" | wc -l

23
```

- there doesn't appear to be a drop off of accuracy between these results
### Timing

Because such settings cannot always be optimized manually, as in a black-box penetration test, `Nmap` offers six different timing templates (`-T <0-5>`) for us to use. These values (`0-5`) determine the aggressiveness of our scans. This can also have negative effects if the scan is too aggressive, and security systems may block us due to the produced network traffic. The default timing template used when we have defined nothing else is the normal (`-T 3`).

- `-T 0` / `-T paranoid`
- `-T 1` / `-T sneaky`
- `-T 2` / `-T polite`
- `-T 3` / `-T normal`
- `-T 4` / `-T aggressive`
- `-T 5` / `-T insane`

The link for what the specific settings for each level are here:
https://nmap.org/book/performance-timing-templates.html

#### Default Scan


```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.0/24 -F -oN tnet.default 

<SNIP>
Nmap done: 256 IP addresses (10 hosts up) scanned in 32.44 seconds
```

#### Insane Scan


```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.0/24 -F -oN tnet.T5 -T 5

<SNIP>
Nmap done: 256 IP addresses (10 hosts up) scanned in 18.07 seconds
```

|**Scanning Options**|**Description**|
|---|---|
|`10.129.2.0/24`|Scans the specified target network.|
|`-F`|Scans top 100 ports.|
|`-oN tnet.T5`|Saves the results in normal formats, starting the specified file name.|
|`-T 5`|Specifies the insane timing template.|
- the results here were the same, but you can set off a firewall or miss information if the timing is set too high
# Bypass Security Measures
## Firewall and IDS/IPS Evasion

**Ways to Bypass**
- Fragmentation of packets
- The use of decoys
- MTU Adjustment
- Source IP Spoofing
- Source Port Selection
- ACK Scan
- Custom DNS Servers
- Interface Selection

### Determine Firewall and Their Rules 
 The packets can either be `dropped`, or `rejected`. The `dropped` packets are ignored, and no response is returned from the host.

This is different for `rejected` packets that are returned with an `RST` flag. These packets contain different types of ICMP error codes or contain nothing at all.

Such errors can be:

- Net Unreachable
- Net Prohibited
- Host Unreachable
- Host Prohibited
- Port Unreachable
- Port Unreachable

###   Nmap TCP ACK Scan

- **Command**: nmap -sA (target)
- **Purpose**: Bypasses firewalls/IDS-IPS.
- **Mechanism**: Sends TCP packet with ACK flag only.
- **Response**: Open/closed ports return RST flag.
- **Advantage**: Firewalls often pass ACK packets, unable to distinguish internal/external connection origin, unlike SYN packets which are typically blocked.
### Comparing SYN-Scan to ACK-Scan #technique

**SYN-Scan**
```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.28 -p 21,22,25 -sS -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 14:56 CEST
SENT (0.0278s) TCP 10.10.14.2:57347 > 10.129.2.28:22 S ttl=53 id=22412 iplen=44  seq=4092255222 win=1024 <mss 1460>
SENT (0.0278s) TCP 10.10.14.2:57347 > 10.129.2.28:25 S ttl=50 id=62291 iplen=44  seq=4092255222 win=1024 <mss 1460>
SENT (0.0278s) TCP 10.10.14.2:57347 > 10.129.2.28:21 S ttl=58 id=38696 iplen=44  seq=4092255222 win=1024 <mss 1460>
RCVD (0.0329s) ICMP [10.129.2.28 > 10.10.14.2 Port 21 unreachable (type=3/code=3) ] IP [ttl=64 id=40884 iplen=72 ]
RCVD (0.0341s) TCP 10.129.2.28:22 > 10.10.14.2:57347 SA ttl=64 id=0 iplen=44  seq=1153454414 win=64240 <mss 1460>
RCVD (1.0386s) TCP 10.129.2.28:22 > 10.10.14.2:57347 SA ttl=64 id=0 iplen=44  seq=1153454414 win=64240 <mss 1460>
SENT (1.1366s) TCP 10.10.14.2:57348 > 10.129.2.28:25 S ttl=44 id=6796 iplen=44  seq=4092320759 win=1024 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up (0.0053s latency).

PORT   STATE    SERVICE
21/tcp filtered ftp


22/tcp open     ssh


25/tcp filtered smtp
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.07 seconds
```

**ACK-Scan**
```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.28 -p 21,22,25 -sA -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 14:57 CEST
SENT (0.0422s) TCP 10.10.14.2:49343 > 10.129.2.28:21 A ttl=49 id=12381 iplen=40  seq=0 win=1024
SENT (0.0423s) TCP 10.10.14.2:49343 > 10.129.2.28:22 A ttl=41 id=5146 iplen=40  seq=0 win=1024
SENT (0.0423s) TCP 10.10.14.2:49343 > 10.129.2.28:25 A ttl=49 id=5800 iplen=40  seq=0 win=1024
RCVD (0.1252s) ICMP [10.129.2.28 > 10.10.14.2 Port 21 unreachable (type=3/code=3) ] IP [ttl=64 id=55628 iplen=68 ]
RCVD (0.1268s) TCP 10.129.2.28:22 > 10.10.14.2:49343 R ttl=64 id=0 iplen=40  seq=1660784500 win=0
SENT (1.3837s) TCP 10.10.14.2:49344 > 10.129.2.28:25 A ttl=59 id=21915 iplen=40  seq=0 win=1024
Nmap scan report for 10.129.2.28
Host is up (0.083s latency).

PORT   STATE      SERVICE
21/tcp filtered   ftp


22/tcp unfiltered ssh


25/tcp filtered   smtp
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.15 seconds
```
-   SYN scan (-sS) sends SYN packets, receiving SYN-ACK for open ports, but firewalls often block it. ACK scan (-sA) sends ACK packets, getting RST for open/closed ports, bypassing firewalls as they can't distinguish connection origins, vital for penetration testing.
- the key to looking for the presence of a firewall #tip

#technique

|**Scanning Options**|**Description**|
|---|---|
|`10.129.2.28`|Scans the specified target.|
|`-p 21,22,25`|Scans only the specified ports.|
|`-sS`|Performs SYN scan on specified ports.|
|`-sA`|Performs ACK scan on specified ports.|
|`-Pn`|Disables ICMP Echo requests.|
|`-n`|Disables DNS resolution.|
|`--disable-arp-ping`|Disables ARP ping.|
|`--packet-trace`|Shows all packets sent and received.|

### Detect IDS/IPS
- more difficult to detect because these are passive until they find something
- worst case is the administrator is notified
- IPS serves as a compliment to IDS
- **Several virtual private servers (VPS) with different IP Addresses are recommended to determine whether such systems are on the target network** #tip 
- We can trigger certain security measures from an administrator, for example, by aggressively scanning a single port and its service. To see if an **IDS** may be present #technique
- A method to determine whether an **IPS** system is present is to scan from a single host (**VPS**). If at any time this host is blocked and has no access to the target network, we know that the administrator has taken some security measures. Accordingly, we can continue our penetration test with another **VPS** #technique
### Decoys
- When admins block specific subnets from different regions
- When IPS should block us
- -D: flag
- Nmap generates various random IP addresses inserted into the IP header to disguise the origin of the packet sent. 
- Generate Random (RND) of a number (for example: 5) of IP addresses separated by a colon.
#### Scan by Using Decoys

  Firewall and IDS/IPS Evasion

```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 16:14 CEST
SENT (0.0378s) TCP 102.52.161.59:59289 > 10.129.2.28:80 S ttl=42 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0378s) TCP 10.10.14.2:59289 > 10.129.2.28:80 S ttl=59 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 210.120.38.29:59289 > 10.129.2.28:80 S ttl=37 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 191.6.64.171:59289 > 10.129.2.28:80 S ttl=38 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 184.178.194.209:59289 > 10.129.2.28:80 S ttl=39 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 43.21.121.33:59289 > 10.129.2.28:80 S ttl=55 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
RCVD (0.1370s) TCP 10.129.2.28:80 > 10.10.14.2:59289 SA ttl=64 id=0 iplen=44  seq=4056111701 win=64240 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up (0.099s latency).

PORT   STATE SERVICE
80/tcp open  http
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.15 seconds
```
- the second ip is the real one (10.10.14.2)

|**Scanning Options**|**Description**|
|---|---|
|`10.129.2.28`|Scans the specified target.|
|`-p 80`|Scans only the specified ports.|
|`-sS`|Performs SYN scan on specified ports.|
|`-Pn`|Disables ICMP Echo requests.|
|`-n`|Disables DNS resolution.|
|`--disable-arp-ping`|Disables ARP ping.|
|`--packet-trace`|Shows all packets sent and received.|
|`-D RND:5`|Generates five random IP addresses that indicates the source IP the connection comes from.|
- the spoofed packets are usually filtered out by ISPs and routers
- we can specify our VPS servers' IP addresses and use them in combination with `ID ID` manipulation in the IP headers to scan the target

Another scenario would be that only individual subnets would not have access to the server's specific services. So we can also manually specify the source IP address (`-S`) to test if we get better results with this one. Decoys can be used for SYN, ACK, ICMP scans, and OS detection scans. So let us look at such an example and determine which operating system it is most likely to be.

#### Testing Firewall Rule #technique #tip

  Firewall and IDS/IPS Evasion

```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.28 -n -Pn -p445 -O

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-22 01:23 CEST
Nmap scan report for 10.129.2.28
Host is up (0.032s latency).

PORT    STATE    SERVICE
445/tcp filtered microsoft-ds
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Too many fingerprints match this host to give specific OS details
Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3.14 seconds
```

#### Scan by Using Different Source IP

  Firewall and IDS/IPS Evasion

```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-22 01:16 CEST
Nmap scan report for 10.129.2.28
Host is up (0.010s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.32 (96%), Linux 3.2 - 4.9 (96%), Linux 2.6.32 - 3.10 (96%), Linux 3.4 - 3.10 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Synology DiskStation Manager 5.2-5644 (94%), Linux 2.6.32 - 2.6.35 (94%), Linux 2.6.32 - 3.5 (94%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 4.11 seconds
```

|**Scanning Options**|**Description**|
|---|---|
|`10.129.2.28`|Scans the specified target.|
|`-n`|Disables DNS resolution.|
|`-Pn`|Disables ICMP Echo requests.|
|`-p 445`|Scans only the specified ports.|
|`-O`|Performs operation system detection scan.|
|`-S`|Scans the target by using different source IP address.|
|`10.129.2.200`|Specifies the source IP address.|
|`-e tun0`|Sends all requests through the specified interface.|
### DNS Proxying #technique
- **Default**: Nmap uses reverse DNS (UDP port 53) to gather target info.
- **TCP Port 53**: Increasingly used due to IPv6/DNSSEC.
- **Custom DNS**: nmap --dns-servers (ns),(ns) - Use trusted internal DNS in DMZ.
- **Source Port**: nmap --source-port 53 - Bypass firewalls if port 53 is less filtered.
#### SYN-Scan of a Filtered Port

  Firewall and IDS/IPS Evasion

```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 22:50 CEST
SENT (0.0417s) TCP 10.10.14.2:33436 > 10.129.2.28:50000 S ttl=41 id=21939 iplen=44  seq=736533153 win=1024 <mss 1460>
SENT (1.0481s) TCP 10.10.14.2:33437 > 10.129.2.28:50000 S ttl=46 id=6446 iplen=44  seq=736598688 win=1024 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up.

PORT      STATE    SERVICE
50000/tcp filtered ibm-db2

Nmap done: 1 IP address (1 host up) scanned in 2.06 seconds
```

#### SYN-Scan From DNS Port

  Firewall and IDS/IPS Evasion

```shell-session
mricognito@htb[/htb]$ sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53

SENT (0.0482s) TCP 10.10.14.2:53 > 10.129.2.28:50000 S ttl=58 id=27470 iplen=44  seq=4003923435 win=1024 <mss 1460>
RCVD (0.0608s) TCP 10.129.2.28:50000 > 10.10.14.2:53 SA ttl=64 id=0 iplen=44  seq=540635485 win=64240 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up (0.013s latency).

PORT      STATE SERVICE
50000/tcp open  ibm-db2
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.08 seconds
```

|**Scanning Options**|**Description**|
|---|---|
|`10.129.2.28`|Scans the specified target.|
|`-p 50000`|Scans only the specified ports.|
|`-sS`|Performs SYN scan on specified ports.|
|`-Pn`|Disables ICMP Echo requests.|
|`-n`|Disables DNS resolution.|
|`--disable-arp-ping`|Disables ARP ping.|
|`--packet-trace`|Shows all packets sent and received.|
|`--source-port 53`|Performs the scans from specified source port.|
- now that we have found out that the firewall accepts TCP port 53, it is very likely that IDS/IPS filters might also be configured much weaker than others. We can test this by trying to connect to this port by using Netcat #tip 
#### Connect To The Filtered Port

  Firewall and IDS/IPS Evasion

```shell-session
mricognito@htb[/htb]$ ncat -nv --source-port 53 10.129.2.28 50000

Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Connected to 10.129.2.28:50000.
220 ProFTPd
```

## Firewall and IDS/IPS Evasion Labs

In the next three sections, we get different scenarios to practice where we have to scan our target. Firewall rules and IDS/IPS protect the systems, so we need to use the techniques shown to bypass the firewall rules and do this as quiet as possible. Otherwise, we will be blocked by IPS.
## Easy Lab
+ 1  Our client wants to know if we can identify which operating system their provided machine is running on. Submit the OS name as the answer.
+  10.129.174.149

```
┌──(kali㉿kali)-[~]│
└─$ nmap 10.129.174.149 -F -sS -Pn -O -n -D RND:5│
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-08 14:11 EDT│
Nmap scan report for 10.129.174.149│
Host is up (0.046s latency).│
Not shown: 60 closed tcp ports (reset), 38 filtered tcp ports (no-response)              │
PORT   STATE SERVICE│
22/tcp open  ssh│
80/tcp open  http│
Device type: general purpose|router│
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X│
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:│
7 cpe:/o:linux:linux_kernel:5.6.3│
OS details: Linux 4.15 - 5.19, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)                 │
Network Distance: 2 hops│
│
OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .│
Nmap done: 1 IP address (1 host up) scanned in 2.91 seconds 
```

```
kali㉿kali)-[~]│
└─$ nmap 10.129.174.149 -p 80 -sV -sS -Pn -O -n -D RND:5│
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-08 14:15 EDT│
Nmap scan report for 10.129.174.149│
Host is up (0.046s latency).│
│
PORT   STATE SERVICE VERSION│
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))│
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1│
 closed port│
Device type: general purpose|router│
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X│
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:│
7 cpe:/o:linux:linux_kernel:5.6.3│
OS details: Linux 4.15 - 5.19, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)                 │
Network Distance: 2 hops│
│
OS and Service detection performed. Please report any incorrect results at https://nmap.o│
rg/submit/ .│
Nmap done: 1 IP address (1 host up) scanned in 8.16 seconds
```

Answer: Ubuntu
## Medium Lab
Using a SYN scan with port 53 (DNS) port
```
┌──(kali㉿kali)-[~]│
└─$ sudo nmap 10.129.141.93 --source-port 53 -n -Pn -sS -F -D RND:5                                                          │
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-08 14:38 EDT│
Nmap scan report for 10.129.141.93│
Host is up (0.053s latency).│
Not shown: 92 closed tcp ports (reset)│
PORT    STATE    SERVICE│
21/tcp  open     ftp│
22/tcp  open     ssh│
53/tcp  filtered domain│
80/tcp  open     http│
110/tcp open     pop3│
139/tcp open     netbios-ssn│
143/tcp open     imap│
445/tcp filtered microsoft-ds
```

```
┌──(kali㉿kali)-[~]│
└─$ sudo nmap 10.129.141.93 -p 53 --source-port 53 -n -Pn -sA -sU --script version│
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-08 14:42 EDT│
Nmap scan report for 10.129.141.93│
Host is up (0.046s latency).│
│
PORT   STATE    SERVICE│
53/tcp filtered domain│
53/udp open     domain│
│
Nmap done: 1 IP address (1 host up) scanned in 3.32 seconds
```

```
┌──(kali㉿kali)-[~]│
└─$ sudo nmap 10.129.141.93 -p 53 --source-port 53 -Pn -sA -sU -O --reason│
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-08 14:46 EDT│
Nmap scan report for 10.129.141.93│
Host is up, received user-set (0.043s latency).│
│
PORT   STATE    SERVICE REASON│
53/tcp filtered domain  no-response│
53/udp open     domain  udp-response ttl 63│
Too many fingerprints match this host to give specific OS details│
Network Distance: 2 hops│
│
OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .                                    │
Nmap done: 1 IP address (1 host up) scanned in 3.88 seconds     
```

I know UDP works and there is a webserver so the firewall may allow traffic from port 80 to their DNS server. Here is the command that will do that and run the `--script discovery` through this:
`sudo nmap 10.129.141.93 -p 53 -Pn -sA -sU -sV --disable-arp-ping --packet-trace --reason --source-port 80 --script discovery`

```
PORT   STATE      SERVICE REASON              VERSION
53/tcp unfiltered domain  reset ttl 63

53/udp open       domain  udp-response ttl 63 (unknown banner: HTB{GoTtgUnyze9Psw4vGjcuMpHRp})

|_dns-cache-snoop: 0 of 100 tested domains are cached.
| dns-nsid:

|_  bind.version: HTB{GoTtgUnyze9Psw4vGjcuMpHRp}

|_dns-nsec3-enum: Can't determine domain for host 10.129.141.93; use dns-nsec3-enum.domains script arg.
|_dns-nsec-enum: Can't determine domain for host 10.129.141.93; use dns-nsec-enum.domains script arg.
| fingerprint-strings:
|   DNS-SD:
|     _services
|     _dns-sd
|     _udp
|     local
|     ROOT-SERVERS
|   DNSVersionBindReq:
|     version
|     bind

|     HTB{GoTtgUnyze9Psw4vGjcuMpHRp}

|   NBTStat:
|     CKAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
|_    ROOT-SERVERS 
```

Answer: ` HTB{GoTtgUnyze9Psw4vGjcuMpHRp}`

## Hard Lab
Target(s): 10.129.255.177
```
┌──(kali㉿kali)-[~]
└─$ sudo nmap 10.129.141.93 -p 53 -Pn -sA -sU -sV --disable-arp-ping --packet-trace --reason --source-port 80 --script discovery
```

```
PORT     STATE         SERVICE     REASON              VERSION
49/udp   open|filtered tacacs      no-response
68/udp   open|filtered dhcpc       no-response
69/udp   open|filtered tftp        no-response
111/udp  open|filtered rpcbind     no-response
120/udp  open|filtered cfdptkt     no-response
135/udp  open|filtered msrpc       no-response
137/udp  open          netbios-ns  udp-response ttl 63 Samba nmbd netbios-ns (workgroup: WORKGROUP)
138/udp  open|filtered netbios-dgm no-response
500/udp  open|filtered isakmp      no-response
515/udp  open|filtered printer     no-response
1030/udp open|filtered iad1        no-response
1701/udp open|filtered L2TP        no-response
2000/udp open|filtered cisco-sccp  no-response
5353/udp open|filtered zeroconf    no-response
Service Info: Host: NIX-NMAP-HARD

Host script results:
|_dns-brute: Can't guess domain of "10.129.255.177"; use dns-brute.domain script argument.
|_fcrdns: FAIL (No PTR record)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 462.79 seconds
```

- got  a udp response for netBIOS on port 137,  going to try and use that to get the version of the  services running

```
kali㉿kali)-[~]
└─$ nc -nv -u -p53 10.129.255.177 137
(UNKNOWN) [10.129.255.177] 137 (netbios-ns) open
```

port 445 for smb gives us a udp-response:
```
┌──(kali㉿kali)-[~]
└─$ nmap 10.129.255.177 -p137 -sU -Pn -sA --source-port 445 --script version --reason
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-08 15:54 EDT
Nmap scan report for 10.129.255.177
Host is up, received user-set (0.045s latency).

PORT    STATE    SERVICE    REASON
137/tcp filtered netbios-ns no-response
137/udp open     netbios-ns udp-response ttl 63

Nmap done: 1 IP address (1 host up) scanned in 3.37 seconds
```

running a scan with no dns resolution, no ICMP, and disabling arp-ping
```
┌──(kali㉿kali)-[~]
└─$ nmap 10.129.255.177 -v -p- -Pn -n --disable-arp-ping --source-port 53 
```

```
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-08 16:37 EDT
Initiating SYN Stealth Scan at 16:37
Scanning 10.129.255.177 [65535 ports]
Discovered open port 80/tcp on 10.129.255.177
Discovered open port 22/tcp on 10.129.255.177
Discovered open port 50000/tcp on 10.129.255.177
Completed SYN Stealth Scan at 16:38, 18.25s elapsed (65535 total ports)
Nmap scan report for 10.129.255.177
Host is up (0.048s latency).
Not shown: 64562 closed tcp ports (reset), 970 filtered tcp ports (no-response)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
50000/tcp open  ibm-db2

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 18.33 seconds
           Raw packets sent: 66505 (2.926MB) | Rcvd: 64565 (2.583MB)
```


turns out the netcat command is not always instantaneous, you may need to wait a moment for your version/flag
```
┌──(kali㉿kali)-[~]
└─$ nc -nv -p53 10.129.255.177 50000
(UNKNOWN) [10.129.255.177] 50000 (?) open
220 HTB{kjnsdf2n982n1827eh76238s98di1w6}
```