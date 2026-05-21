# OSCP Vulnhub Set 1 - FristiLeaks 1.3

Lab link: http://ccmtlab.ccmt.home.arpa:8888/user/missions/boxes?uuid=19eeead8-0be9-42cf-a338-12aaf55c1533

Target IP: 10.101.85.16

---

## Scanning and Enumeration

### Nmap

Scan the available ports.

```
nmap -Pn 10.101.85.16
```

Only HTTP port is open.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/06]
└─$ nmap -Pn 10.101.85.16
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-21 02:05 -0400
Nmap scan report for 10.101.85.16
Host is up (0.0029s latency).
Not shown: 988 filtered tcp ports (no-response), 11 filtered tcp ports (host-prohibited)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 5.72 seconds
```

---

### Nikto

Scan for web server vulnerabilities.

```
nikto -h 10.101.85.16
```

The scan results reveal that the server runs Apache/2.2.15 and PHP/5.3.3. Additionally, a robots.txt file was discovered containing 3 hidden entries worth investigating.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/06]
└─$ nikto -h 10.101.85.16                      
- Nikto v2.6.0
---------------------------------------------------------------------------
+ Target IP:          10.101.85.16
+ Target Hostname:    10.101.85.16
+ Target Port:        80
+ Platform:           Linux/Unix
+ Start Time:         2026-05-21 01:42:11 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.2.15 (CentOS) DAV/2 PHP/5.3.3
+ ERROR: Failed to check for updates: 403
+ [999984] /: Server may leak inodes via ETags, header found with file /, inode: 12722, size: 703, mtime: Tue Nov 17 13:45:47 2015. See: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2003-1418
+ No CGI Directories found (use '-C all' to force check all possible dirs). CGI tests skipped.
+ [999996] /robots.txt: contains 3 entries which should be manually viewed. See: https://developer.mozilla.org/en-US/docs/Glossary/Robots.txt
+ [600625] PHP/5.3.3 appears to be outdated (current is at least 8.5.1).
+ [600050] Apache/2.2.15 appears to be outdated (current is at least 2.4.66).
+ [013587] /: Suggested security header missing: strict-transport-security. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
+ [013587] /: Suggested security header missing: content-security-policy. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP
+ [013587] /: Suggested security header missing: permissions-policy. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy
+ [013587] /: Suggested security header missing: referrer-policy. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy
+ [013587] /: Suggested security header missing: x-content-type-options. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options
+ [800262] /: PHP/5.3 - PHP 3/4/5 and 7.0 are End of Life products without support.
+ [999990] OPTIONS: Allowed HTTP Methods: GET, HEAD, POST, OPTIONS, TRACE .
+ [000434] /: HTTP TRACE method is active and replies which suggests the host is vulnerable to XST. See: https://owasp.org/www-community/attacks/Cross_Site_Tracing
+ [750500] /icons/: Directory indexing found.
+ [750500] /images/: Directory indexing found.
+ [003584] /icons/README: Apache default file found. See: https://www.vntweb.co.uk/apache-restricting-access-to-iconsreadme/
+ [007342] /: X-Frame-Options header is deprecated and was replaced with the Content-Security-Policy HTTP header with the frame-ancestors directive. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/X-Frame-Options
+ [007352] /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ 8117 requests: 16 errors and 17 items reported on the remote host
+ End Time:           2026-05-21 01:48:20 (GMT-4) (369 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

---

### Web Application Enumeration

Open the website and try to find anything which may leads us to CVE or something else.

```
http://10.101.85.16/
```
