# OSCP Vulnhub Set 1 - Zico 2

Lab link: http://ccmtlab.ccmt.home.arpa:8888/user/missions/boxes?uuid=2bd816c6-a344-4397-9e5e-027546fb11c5

Target IP: 10.101.85.29

---

## Scanning and Enumeration

### Nmap

สแกน port ที่เปิดอยู่

```
nmap -Pn 10.101.85.29
```

เจอ ssh, http, แลพ rpcbind

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

### Web Enum

เปิด ip ใน browser

```
http://10.101.85.29/
```

ยังไม่เจออะไรสำคัญ

![](./images/01.png)

หลังจากอ่าน page source พบว่ามีการใช้ view.php?page= ซึ่งมีโอกาสสูงที่โค้ด view.php จะใช้คำสั่งประเภท include() หรือ require() เพื่อดึงไฟล์ tools.html มาแสดงบนหน้าเว็บ และอาจเป็นช่องโหว่ LFI หากโค้ดไม่รัดกุม

```
[...snip...]

<a href="/view.php?page=tools.html" class="btn btn-default btn-xl sr-button">Check them out!</a>

[...snip...]
```

---

### Burp Suit

เปิด burp suit เพื่อดักจับ request ของลิ้งนี้แล้วส่งไปที่ repeater

```
http://10.101.85.29/view.php?page=tools.html
```

![](./images/02.png)
