# OSCP Vulnhub Set 1 - Kioptrix: Level 1.2 (#3)

lab link: http://ccmtlab.ccmt.home.arpa:8888/user/missions/boxes?uuid=55216f27-3ed8-47fe-a730-2f61fd896124

target ip: `10.101.85.13`

---

## Network Scanning

### nmap

scan all port.

```
nmap -p- 10.101.85.13
```

only 22 and 80 are open.
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

vulnerbality scan.

```
nmap --script vuln -p22,80 10.101.85.13
```



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

next, nikto.

---

### nikto

use this.

```
nikto -h http://10.101.85.13 
```

the `/phpmyadmin` are interesting.

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
+ ERROR: Failed to check for updates: 403
+ [999986] /: Retrieved x-powered-by header: PHP/5.2.4-2ubuntu5.6.
+ [95] /: Cookie PHPSESSID created without the httponly flag. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies
+ [750500] /icons/: Directory indexing found.
+ No CGI Directories found (use '-C all' to force check all possible dirs). CGI tests skipped.
+ [600050] Apache/2.2.8 appears to be outdated (current is at least 2.4.66).
+ [600625] PHP/5.2.4-2ubuntu5.6 appears to be outdated (current is at least 8.5.1).
+ [999984] /favicon.ico: Server may leak inodes via ETags, header found with file /favicon.ico, inode: 631780, size: 23126, mtime: Fri Jun  5 15:22:00 2009. See: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2003-1418
+ [013587] /: Suggested security header missing: content-security-policy. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP
+ [013587] /: Suggested security header missing: strict-transport-security. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
+ [013587] /: Suggested security header missing: x-content-type-options. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options
+ [013587] /: Suggested security header missing: permissions-policy. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy
+ [013587] /: Suggested security header missing: referrer-policy. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy
+ [800262] /: PHP/5.2 - PHP 3/4/5 and 7.0 are End of Life products without support.
+ [999967] /: Web Server returns a valid response with junk HTTP methods which may cause false positives.
+ [000434] /: HTTP TRACE method is active and replies which suggests the host is vulnerable to XST. See: https://owasp.org/www-community/attacks/Cross_Site_Tracing
+ [001384] /?=PHPB8B5F2A0-3C92-11d3-A3A9-4C7B08C10000: PHP Easter Eggs reveals potentially sensitive information via HTTP requests that contain specific QUERY strings. See: https://labs.detectify.com/writeups/do-you-dare-to-show-your-php-easter-egg/
+ [001385] /?=PHPE9568F36-D428-11d2-A769-00AA001ACF42: PHP Easter Egg reveals potentially sensitive information via HTTP requests that contain specific QUERY strings. See: https://labs.detectify.com/writeups/do-you-dare-to-show-your-php-easter-egg/
+ [001386] /?=PHPE9568F34-D428-11d2-A769-00AA001ACF42: PHP Easter Egg reveals potentially sensitive information via HTTP requests that contain specific QUERY strings. See: https://labs.detectify.com/writeups/do-you-dare-to-show-your-php-easter-egg/
+ [001387] /?=PHPE9568F35-D428-11d2-A769-00AA001ACF42: PHP Easter Egg reveals potentially sensitive information via HTTP requests that contain specific QUERY strings. See: https://labs.detectify.com/writeups/do-you-dare-to-show-your-php-easter-egg/
+ [001795] /phpmyadmin/changelog.php: phpMyAdmin is for managing MySQL databases, and should be protected or limited to authorized hosts.
+ [003584] /icons/README: Apache default file found. See: https://www.vntweb.co.uk/apache-restricting-access-to-iconsreadme/
^C    
```

i'll leave it for now, let's explore the site

---

### http explore

open the target ip site.

```
http://10.101.85.13
```

home page.

![](./images/01.png)

blog page.

![](./images/02.png)

login page, it's powered by `lotuscms`.

![](./images/03.png)

