- 先把pilgrimage加進host裡
 ![](https://hackmd.io/_uploads/BkRp52zM6.png)

  ![](https://hackmd.io/_uploads/Sy6Aqnzzp.png)
- 用nmap掃看看有開啟什麼port, 有開啟22跟80port 
![](https://hackmd.io/_uploads/S1FgeTGfp.png)

- 用dirsearch掃了發現有.git洩漏
![](https://hackmd.io/_uploads/S1-066mfp.png)


- 用githacker下載下來獲得源碼
```
#quick start
githacker --url http://127.0.0.1/.git/ --output-folder result
```
![](https://hackmd.io/_uploads/SygPaaXfa.png)

![](https://hackmd.io/_uploads/SJlKDRXf6.png)

- 在index.php裏頭找到他主要執行文件轉換的地方和洩漏資料庫的位置
![](https://hackmd.io/_uploads/HkUTvRXzp.png)

  ![](https://hackmd.io/_uploads/r1mlQ_SGT.png)


- 去找看看magick有沒有轉換圖片的漏洞，先看他的版本號
![](https://hackmd.io/_uploads/Bkaid07za.png)

- 用searchsploit 找尋magick漏洞
![](https://hackmd.io/_uploads/S1rTt0XfT.png)

- -m是把這個漏洞複製到當前的目錄下
![](https://hackmd.io/_uploads/ryGAs0mM6.png)
- 查看此漏洞
  ![](https://hackmd.io/_uploads/HkZ130Xza.png)
[CVE-2022-44268 ImageMagick任意文件读取漏洞](https://cloud.tencent.com/developer/article/2235689)

- 記得要找真的是png檔的，不能自己改檔名
![](https://hackmd.io/_uploads/HJXa1-4z6.png)

- 確認有把東西加在尾巴上
![](https://hackmd.io/_uploads/rk1Cy-4Gp.png)

- 然後上傳這個png，下載轉換後的png，用identify查看詳細資料中的profile
![](https://hackmd.io/_uploads/SyFvBZNG6.png)

- 拿去用cyberchef進行16進制的轉換可以發現有emily這個帳號
![](https://hackmd.io/_uploads/H1p07fEzT.png)

- 先前有發現db洩漏位置 用一樣的方法去讀取他的文件
![](https://hackmd.io/_uploads/Bkx-BdHGa.png)

- 保存為文件打開後在user的table看到了帳號密碼
![](https://hackmd.io/_uploads/BkQtudSM6.png)

- 透過帳號密碼成功連上ssh
![](https://hackmd.io/_uploads/Sy9XDPLz6.png)

- 得到user flag
![](https://hackmd.io/_uploads/S1LDvPIz6.png)

- 用ps auxwww看現在有進行甚麼process(ps 用來顯示當前進程狀態，aux顯示所有包含其他使用者的進程，www是把ps用寬表執行變得好閱讀)，發現有/usr/sbin/malwarescan.sh
![](https://hackmd.io/_uploads/rytj5vUMT.png)

- 在裡面發現binwalk 查看内容發現是一个bash腳本，功能是在"/var/www/pilgrimage.htb/shrunk/"目錄下監控新創建的腳本。每當有新文件被創建，他會使用"binwalk"命令檢查這個文件。如果文件包含在黑名單列表（"Executable script"或"Microsoft executable"）中的任何内容，这個文件就會被删除。
![](https://hackmd.io/_uploads/ByBtiDIGa.png)

- 去看看他的版本號
![](https://hackmd.io/_uploads/SkmIhPLG6.png)

- 在searchsploit找看看有沒有漏洞 發現有一個版本號也一樣[漏洞介紹](https://zhuanlan.zhihu.com/p/654020580)原理是會通過目錄遍歷覆蓋或者創建某個插件，而这个新寫入的會被binwalk執行
- ![](https://hackmd.io/_uploads/SyhAhPLfT.png)

- 使用這個腳本生成一個文件
![](https://hackmd.io/_uploads/HyNnW7cGp.png)

- 開啟一個監聽並透過httpsever把文件傳給emily
![](https://hackmd.io/_uploads/SkykmXqfT.png)

- 進入/var/www/pilgrimage.htb/shrunk 後把文件下載下來就提權成功了
![](https://hackmd.io/_uploads/H1d1SX5GT.png)

- flag 在root下
![](https://hackmd.io/_uploads/BJMEBmcGa.png)
