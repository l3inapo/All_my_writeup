nmap 結果
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 62:36:1a:5c:d3:e3:7b:e1:70:f8:a3:b3:1c:4c:24:38 (RSA)
|   256 ee:25:fc:23:66:05:c0:c1:ec:47:c6:bb:00:c7:4f:53 (ECDSA)
|_  256 83:5c:51:ac:32:e5:3a:21:7c:f6:c2:cd:93:68:58:d8 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: All topics | CODOLOGIC
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.46 seconds
```
連上後是一個類似 CMS 網站且使用 CODOLOGIC
![image](https://hackmd.io/_uploads/rk976Z-YJx.png)

掃他的路徑有發現 admin 入口
![image](https://hackmd.io/_uploads/Hker2bWKyl.png)

訪問後弱密碼 admin admin 可以直接成功登入 同時上面也可以很直接的看到版本號為 5.1.105
![image](https://hackmd.io/_uploads/Sk0DhZbYyl.png)

找到他有一個 File upload 的漏洞 CVE-2022-31854
運行腳本失敗 但他有告訴我們要如何自己去手動發現
![image](https://hackmd.io/_uploads/rJbb19bKJg.png)

上傳簡單的 webshell 後加上 reverseshell 成功進入

在 `/var/www/html/sites/default` 找到一個 config file 裡頭有密碼
![image](https://hackmd.io/_uploads/r1rVg9-Fkl.png)

想說可能會不會是密碼誤用的問題 所以把所有的 user 列出來一個一個試他的密碼

有這兩個 user 主要找 id > 1000
![image](https://hackmd.io/_uploads/ByfqlcZKkx.png)

su root 並輸入密碼成功登入
![image](https://hackmd.io/_uploads/HJXXbc-KJe.png)


echo "密碼" | su - 帳號
快速測帳號