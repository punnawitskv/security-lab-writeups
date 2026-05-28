# OSCP Vulnhub Set 1 - HackLAB: Vulnix

Lab link: http://ccmtlab.ccmt.home.arpa:8888/user/missions/boxes?uuid=41934fc6-f438-4f9f-80e8-1968c506284b

Target IP: 10.101.85.20

---

## Scanning and Enumeration

### Nmap

แสกนหาพอร์ตที่เปิดอยู่

```
nmap -Pn 10.101.85.20
```

เจอหลายอันเลยที่เปิดอยู่ ที่น่าสนใจก็คงเป็น finger กับ shell และ nfs

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/10]
└─$ nmap -Pn 10.101.85.20
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-28 02:02 -0400
Nmap scan report for 10.101.85.20
Host is up (0.0030s latency).
Not shown: 988 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
25/tcp   open  smtp
79/tcp   open  finger
110/tcp  open  pop3
111/tcp  open  rpcbind
143/tcp  open  imap
512/tcp  open  exec
513/tcp  open  login
514/tcp  open  shell
993/tcp  open  imaps
995/tcp  open  pop3s
2049/tcp open  nfs
```

---

### Finger

ผมไปหาข้อมูลมา จาก doc นี้ fingeruser สามารถ enum ได้

```
https://github.com/pentestmonkey/finger-user-enum/blob/master/finger-user-enum.pl
```

โหลดเข้า kali

```
wget https://raw.githubusercontent.com/pentestmonkey/finger-user-enum/refs/heads/master/finger-user-enum.pl
```

เจอ username user กับ root

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/10]
└─$ perl finger-user-enum.pl -U /usr/share/seclists/Usernames/top-usernames-shortlist.txt -t 10.101.85.20
Starting finger-user-enum v1.0 ( http://pentestmonkey.net/tools/finger-user-enum )

 ----------------------------------------------------------
|                   Scan Information                       |
 ----------------------------------------------------------

Worker Processes ......... 5
Usernames file ........... /usr/share/seclists/Usernames/top-usernames-shortlist.txt
Target count ............. 1
Username count ........... 17
Target TCP port .......... 79
Query timeout ............ 5 secs
Relay Server ............. Not used

######## Scan started at Thu May 28 02:21:06 2026 #########
admin@10.101.85.20: finger: admin: no such user...
info@10.101.85.20: finger: info: no such user...
adm@10.101.85.20: finger: adm: no such user...
administrator@10.101.85.20: finger: administrator: no such user...
root@10.101.85.20: Login: root                                  Name: root..Directory: /root                      Shell: /bin/bash..Last login Wed Feb  5 07:31 2025 (GMT) on tty1..No mail...No Plan...
user@10.101.85.20: Login: user                                  Name: user..Directory: /home/user                 Shell: /bin/bash..Last login Mon May 18 08:09 (BST) on pts/0 from 10.101.55.217..No mail...No Plan.....Login: dovenull                              Name: Dovecot login user..Directory: /nonexistent               Shell: /bin/false..Never logged in...No mail...No Plan...
oracle@10.101.85.20: finger: oracle: no such user...
pi@10.101.85.20: finger: pi: no such user...
ftp@10.101.85.20: finger: ftp: no such user...
vagrant@10.101.85.20: finger: vagrant: no such user...
puppet@10.101.85.20: finger: puppet: no such user...
ec2-user@10.101.85.20: finger: ec2-user: no such user...
azureuser@10.101.85.20: finger: azureuser: no such user...
######## Scan completed at Thu May 28 02:21:06 2026 #########
13 results.

17 queries in 1 seconds (17.0 queries / sec)
```

---

### NFS

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/10]
└─$ nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.101.85.20
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-28 02:39 -0400
Nmap scan report for 10.101.85.20
Host is up (0.0050s latency).

PORT    STATE SERVICE
111/tcp open  rpcbind
| nfs-ls: Volume /root
|   access: Read Lookup Modify Extend Delete NoExecute
| PERMISSION  UID  GID  SIZE  TIME                 FILENAME
| drwx------  0    0    4096  2024-12-06T08:12:27  .
| drwxr-xr-x  0    0    4096  2024-12-06T08:02:13  ..
| -rw-------  0    0    356   2025-02-05T07:32:31  .bash_history
| -rw-r--r--  0    0    3106  2012-04-19T09:15:14  .bashrc
| drwx------  0    0    4096  2012-09-02T17:58:01  .cache
| -rw-r--r--  0    0    140   2012-04-19T09:15:14  .profile
| -rw-------  0    0    710   2012-09-02T17:47:19  .viminfo
| -r--------  0    0    39    2024-12-06T08:06:01  root.txt
| 
| 
| Volume /home/vulnix
|   access: Read Lookup NoModify NoExtend NoDelete NoExecute
| PERMISSION  UID   GID   SIZE  TIME                 FILENAME
| drwxr-xr-x  2008  2008  4096  2026-05-08T11:43:42  .
| drwxr-xr-x  0     0     4096  2025-02-05T06:17:24  ..
| -rw-------  2008  2008  1270  2026-05-18T07:43:28  .bash_history
| drwx------  2008  2008  4096  2025-02-05T06:57:02  .cache
| drwx------  2008  2008  4096  2026-05-11T03:56:00  .ssh
| lrwxrwxrwx  2008  2008  11    2026-05-08T09:30:40  exports
| lrwxrwxrwx  2008  2008  11    2026-05-08T08:50:49  mypasswd
|_
| nfs-showmount: 
|   /root *
|_  /home/vulnix *
| nfs-statfs: 
|   Filesystem    1K-blocks  Used      Available  Use%  Maxfilesize  Maxlink
|   /root         792040.0   737312.0  15000.0    99%   8.0T         32000
|_  /home/vulnix  792040.0   737316.0  14996.0    99%   8.0T         32000

Nmap done: 1 IP address (1 host up) scanned in 1.91 seconds
```