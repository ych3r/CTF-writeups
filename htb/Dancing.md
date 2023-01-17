This box is about "SMB".

SMB provides shared access to files, printers, and serial ports between endpoints on a network.

App Layer: SMB
Transport Layer: NetBIOS TCP/IP (NBT)
SMB relies on lover-level protocols for transport, that's why they both exist.

### Scan
```
┌──(kali㉿kali)-[~/htb]
└─$ nmap -sC -sV 10.129.84.99  
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-12 20:17 EST
Nmap scan report for 10.129.84.99
Host is up (0.017s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT    STATE SERVICE       VERSION
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 4h00m00s
| smb2-time: 
|   date: 2022-03-13T05:17:45
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
```

### Use smbclient to connect to SMB shares
```
┌──(kali㉿kali)-[~/htb]
└─$ smbclient -L 10.129.84.99                                                                                 1 ⨯
Enter WORKGROUP\kali's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        WorkShares      Disk      
SMB1 disabled -- no workgroup available
```
ADMIN$ - Administrative shares are hidden network shares created by the Windows NT family of
operating systems that allow system administrators to have remote access to every disk volume on a
network-connected system. These shares may not be permanently deleted but may be disabled.

C$ - Administrative share for the C:\ disk volume. This is where the operating system is hosted.

IPC$ - The inter-process communication share. Used for inter-process communication via named
pipes and is not part of the file system.

WorkShares - Custom share.

```
┌──(kali㉿kali)-[~/htb]
└─$ smbclient \\\\10.129.84.99\\ADMIN$                                                                        1 ⨯
Enter WORKGROUP\kali's password: 
tree connect failed: NT_STATUS_ACCESS_DENIED
                                                                                                                  
┌──(kali㉿kali)-[~/htb]
└─$ smbclient \\\\10.129.84.99\\C$                                                                            1 ⨯
Enter WORKGROUP\kali's password: 
tree connect failed: NT_STATUS_ACCESS_DENIED

┌──(kali㉿kali)-[~/htb]
└─$ smbclient \\\\10.129.84.99\\WorkShares                                                                  130 ⨯
Enter WORKGROUP\kali's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Mar 29 04:22:01 2021
  ..                                  D        0  Mon Mar 29 04:22:01 2021
  Amy.J                               D        0  Mon Mar 29 05:08:24 2021
  James.P                             D        0  Thu Jun  3 04:38:03 2021

                5114111 blocks of size 4096. 1732421 blocks available
smb: \> ls Amy.J\
  .                                   D        0  Mon Mar 29 05:08:24 2021
  ..                                  D        0  Mon Mar 29 05:08:24 2021
  worknotes.txt                       A       94  Fri Mar 26 07:00:37 2021

                5114111 blocks of size 4096. 1731571 blocks available
smb: \> ls James.P\
  .                                   D        0  Thu Jun  3 04:38:03 2021
  ..                                  D        0  Thu Jun  3 04:38:03 2021
  flag.txt                            A       32  Mon Mar 29 05:26:57 2021

                5114111 blocks of size 4096. 1731571 blocks available
smb: \> get Amy.J\worknotes.txt 
getting file \Amy.J\worknotes.txt of size 94 as Amy.J\worknotes.txt (1.0 KiloBytes/sec) (average 1.0 KiloBytes/sec)
smb: \> get James.P\flag.txt 
getting file \James.P\flag.txt of size 32 as James.P\flag.txt (0.4 KiloBytes/sec) (average 0.7 KiloBytes/sec)
smb: \>

┌──(kali㉿kali)-[~/htb]
└─$ cat Amy.J\\worknotes.txt 
- start apache server on the linux machine
- secure the ftp server
- setup winrm on dancing                                                                                                                   
┌──(kali㉿kali)-[~/htb]
└─$ cat James.P\\flag.txt   
5f61c10dffbc77a704d76016a22f1664
```