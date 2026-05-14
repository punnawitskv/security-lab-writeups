# OSCP Vulnhub Set 1 - Kioptrix: Level 1.1 (#2)

lab link: http://ccmtlab.ccmt.home.arpa:8888/user/missions/boxes?uuid=310958ee-e03d-4545-bcf0-f5de26a76405

target ip: `10.101.85.12`

---

## Scanning and Enumeration

find the service that running on that target ip.

```
nmap -A -sC -sV 10.101.85.12
```

- -A : Aggressive Scan
- -sC : Default Script Scan
- -sV : Version Detection

found ssh, http and etc. that running on terget ip.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/02]
└─$ nmap -A -sC -sV 10.101.85.12
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-14 05:22 -0400
Nmap scan report for 10.101.85.12
Host is up (0.0034s latency).
Not shown: 994 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 3.9p1 (protocol 1.99)
|_sshv1: Server supports SSHv1
| ssh-hostkey: 
|   1024 8f:3e:8b:1e:58:63:fe:cf:27:a3:18:09:3b:52:cf:72 (RSA1)
|   1024 34:6b:45:3d:ba:ce:ca:b2:53:55:ef:1e:43:70:38:36 (DSA)
|_  1024 68:4d:8c:bb:b6:5a:bd:79:71:b8:71:47:ea:00:42:61 (RSA)
80/tcp   open  http     Apache httpd 2.0.52 ((CentOS))
|_http-server-header: Apache/2.0.52 (CentOS)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
111/tcp  open  rpcbind  2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            901/udp   status
|_  100024  1            904/tcp   status
443/tcp  open  ssl/http Apache httpd 2.0.52 ((CentOS))
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_DES_64_CBC_WITH_MD5
|     SSL2_RC4_64_WITH_MD5
|_    SSL2_RC4_128_WITH_MD5
|_ssl-date: 2026-05-14T13:23:18+00:00; +4h00m06s from scanner time.
|_http-server-header: Apache/2.0.52 (CentOS)
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2009-10-08T00:10:47
|_Not valid after:  2010-10-08T00:10:47
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
631/tcp  open  ipp      CUPS 1.1
|_http-title: 403 Forbidden
| http-methods: 
|_  Potentially risky methods: PUT
|_http-server-header: CUPS/1.1
3306/tcp open  mysql    MySQL (unauthorized)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.27
Network Distance: 2 hops

Host script results:
|_clock-skew: 4h00m05s

TRACEROUTE (using port 21/tcp)
HOP RTT     ADDRESS
1   5.62 ms 10.101.55.1
2   6.63 ms 10.101.85.12

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.39 seconds
```
