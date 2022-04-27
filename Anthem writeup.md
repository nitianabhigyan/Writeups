# Anthem
This writeup is more geared towards guiding a reader on how to approach this problem ( or at least the way i did), rather than just telling you the flags!
## IP address
10.10.36.112<br>
<b>NOTE</b>: The challenge description clearly states: 
> "In this room, you don't need to brute force any login page. Just your preferred browser and Remote Desktop."
## Scans
### NMAP 
#### Command used
<code>$ nmap -sC -sV -Pn 10.10.36.112</code>
The HOST IS NOT RESPONDING ping requests!!!
#### Results 
<pre>
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-26 14:24 EDT
Nmap scan report for 10.10.36.112
Host is up (0.15s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| http-robots.txt: 4 disallowed entries 
|_/bin/ /config/ /umbraco/ /umbraco_client/
|_http-title: Anthem.com - Welcome to our blog
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2022-04-26T18:24:59+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: WIN-LU09299160F
|   NetBIOS_Domain_Name: WIN-LU09299160F
|   NetBIOS_Computer_Name: WIN-LU09299160F
|   DNS_Domain_Name: WIN-LU09299160F
|   DNS_Computer_Name: WIN-LU09299160F
|   Product_Version: 10.0.17763
|_  System_Time: 2022-04-26T18:24:55+00:00
| ssl-cert: Subject: commonName=WIN-LU09299160F
| Not valid before: 2022-04-25T18:13:06
|_Not valid after:  2022-10-25T18:13:06
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.04 seconds
</pre>

### GoBuster
#### Command used:
<code>$ gobuster dir -u http://10.10.36.112/ -w /usr/share/wordlists/dirb/common.txt </code>
#### Results 
<pre>
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.36.112/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/04/26 14:26:31 Starting gobuster in directory enumeration mode
===============================================================
/archive              (Status: 301) [Size: 118] [--> /]
/Archive              (Status: 301) [Size: 118] [--> /]
/authors              (Status: 200) [Size: 4070]       
/blog                 (Status: 200) [Size: 5394]       
/Blog                 (Status: 200) [Size: 5394]       
/categories           (Status: 200) [Size: 3541]  
============================snipped============================
/install              (Status: 302) [Size: 126] [--> /umbraco/]
/robots.txt           (Status: 200) [Size: 192]                
/rss                  (Status: 200) [Size: 1873]               
/RSS                  (Status: 200) [Size: 1873]               
/search               (Status: 200) [Size: 3468]               
/Search               (Status: 200) [Size: 3468]               
/SiteMap              (Status: 200) [Size: 1041]               
/sitemap              (Status: 200) [Size: 1041]               
/tags                 (Status: 200) [Size: 3594]               
/umbraco              (Status: 200) [Size: 4078]               
                                                               
===============================================================
2022/04/26 14:40:22 Finished
===============================================================
                                   
</pre>

### Random but useful observations

While browsing the pages (indicators: nmap scan and gobuster scan) we observe /http-robots.txt which contains intresting values: 
<pre>
<b>UmbracoIsTheBest!</b>

# Use for all search robots
User-agent: *
# Define the directories not to crawl
Disallow: /bin/
Disallow: /config/
Disallow: /umbraco/
Disallow: /umbraco_client/
</pre>

This is followed by a nice login page at : http://10.10.36.112/umbraco/#/login
which asks for 2 things:
1. Username with a placeholder saying: > "Your Username is usually your email"
2. An obvious password field!

Googling umbraco tells us that it is a **CMS**.

Furthermore, <u>THOROUGHLY</u> inspecting the pages reveals  juicy details such as:
> ""placeholder="Search... 								THM{---SNIPPED---}" 

Reading and inspecting the blogs closely reveals a couple of things:
1. there are at least 2 authors:
	a. Jane Doe
	b. James Orchard Halliwell 

2. The "We are hiring" blog reveals the email id pattern "JD@anthem.com" for the "company".
3. it reveals a hiiden link in headers : http://10.10.36.112/rsd/1073 which opens a big can of worms of it's own!!!
4. The poem is important! It is a nursery rhyme called "Solomon Grundy" :-) do what you want with this.
<br>
So we did manage to find 3/4 flags by just plain enumeration and inspection of the page.	
For the last one, it's hard but you the power of meta !

## Exploitation and RDP

### RDP 
Fairly staright forward.

While logging in, remmeber you bases, short names are good and a strong enumeration goes long way!!!

Other than that be "aware" of playing with file oermissions and maybe read up on your icacls.
	
	
	
	
	
