# OSCP Vulnhub Set 1 - Tr0ll 1

Lab link: http://ccmtlab.ccmt.home.arpa:8888/user/missions/boxes?uuid=a0d13c68-a02b-4472-8323-c6d260f5046f

Target IP: 10.101.85.31

---

## Scanning and Enumeration

### Nmap

Scan all popular ports with OS, version, and script detection.

```
nmap -Pn -A 10.101.85.31
```

เจอ ftp, ssh และ http

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/12]
└─$ nmap -Pn -A 10.101.85.31
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-28 22:15 -0400
Nmap scan report for 10.101.85.31
Host is up (0.0037s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rwxrwxrwx    1 1000     0            8068 Aug 10  2014 lol.pcap [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.101.55.75
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 600
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 d6:18:d9:ef:75:d3:1c:29:be:14:b5:2b:18:54:a9:c0 (DSA)
|   2048 ee:8c:64:87:44:39:53:8c:24:fe:9d:39:a9:ad:ea:db (RSA)
|   256 0e:66:e6:50:cf:56:3b:9c:67:8b:5f:56:ca:ae:6b:f4 (ECDSA)
|_  256 b2:8b:e2:46:5c:ef:fd:dc:72:f7:10:7e:04:5f:25:85 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| http-robots.txt: 1 disallowed entry 
|_/secret
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.11 - 4.9
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 256/tcp)
HOP RTT     ADDRESS
1   2.28 ms 10.101.55.1
2   2.28 ms 10.101.85.31

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.04 seconds
```

---

### FTP

ลอง access ftp ด้วย annonymous ดู

```
ftp 10.101.85.31
```
- Name: `anonymous`
- Password: [No Password]

ดูไฟล์หรือข้อมูลที่เราเข้าถึงได้

```
ls
```

เจอ lol.pcap แค่ไฟล์เดียว

```
ftp> ls
229 Entering Extended Passive Mode (|||11694|).
150 Here comes the directory listing.
-rwxrwxrwx    1 1000     0            8068 Aug 10  2014 lol.pcap
226 Directory send OK.
```

โหลดเข้า kali และออกมา

```
get lol.pcap
exit
```

เปิด wireshark

- คลิกขวา
- follow
- tcp stream

text

```
220 (vsFTPd 3.0.2)

USER anonymous

331 Please specify the password.

PASS password

230 Login successful.

SYST

215 UNIX Type: L8

PORT 10,0,0,12,173,198

200 PORT command successful. Consider using PASV.

LIST

150 Here comes the directory listing.
226 Directory send OK.

TYPE I

200 Switching to Binary mode.

PORT 10,0,0,12,202,172

200 PORT command successful. Consider using PASV.

RETR secret_stuff.txt

150 Opening BINARY mode data connection for secret_stuff.txt (147 bytes).
226 Transfer complete.

TYPE A

200 Switching to ASCII mode.

PORT 10,0,0,12,172,74

200 PORT command successful. Consider using PASV.

LIST

150 Here comes the directory listing.
226 Directory send OK.

QUIT

221 Goodbye.
```