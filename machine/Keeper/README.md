- 先把keeper 加入ect/host
![image.png](https://hackmd.io/_uploads/HkJ9tKDQ6.png)

- 成功進入首頁
![image.png](https://hackmd.io/_uploads/BkdlRFPmT.png)

- 用nmap去掃看看 有開啟22 80 port
![image.png](https://hackmd.io/_uploads/rkducYDXT.png)

- 用dirb掃發現東西很多掃不完 所以應該不是從這裡下手
![image.png](https://hackmd.io/_uploads/By6B6KPX6.png)

- 在右下角有看到版本號 去查看看有沒有漏洞可以使用
![image.png](https://hackmd.io/_uploads/B1AXAYvm6.png)

- 送到 intruder並把要爆破的地方加上$
![image](https://hackmd.io/_uploads/ry_B7v0Xp.png)

- 下載一個字典檔
![image](https://hackmd.io/_uploads/rJpMkPAm6.png)

- 匯入username 的wordlist
![image](https://hackmd.io/_uploads/rJtw1vAQa.png)

- 匯入 password的wordlist
![image](https://hackmd.io/_uploads/rJgmxDRQ6.png)

- 進行爆破 然後成功找到帳號密碼
![image](https://hackmd.io/_uploads/SkDENDRQp.png)

- 看了很多可是都不是我們現在可以使用的 最後是發現可以去嘗試透過他的預設帳號密碼 可以成功登入
![image.png](https://hackmd.io/_uploads/rJW6k5vXa.png)

  ![image.png](https://hackmd.io/_uploads/Byh6k5w7T.png)

- 在裏頭翻了好久找到了一個可以上傳文件的地方
![image](https://hackmd.io/_uploads/Bk40VgAXp.png)``

![image.png](https://hackmd.io/_uploads/SJLtv5vm6.png)


- 創建一個php檔
![image.png](https://hackmd.io/_uploads/Sk6ZqqDQ6.png)

- 然後把檔案傳上去
![image.png](https://hackmd.io/_uploads/BkI7ccvXa.png)

- 他有阻擋所以傳不上去
![image](https://hackmd.io/_uploads/Hkx6nEgCm6.png)

- 在admin users select裡有發現一個Inorgaard帳號
![image](https://hackmd.io/_uploads/rJYwalCX6.png)

- 裡面有他的預設密碼
![image](https://hackmd.io/_uploads/BkIYTeC7T.png)

- 拿去ssh連線後有成功連上
![image](https://hackmd.io/_uploads/SJL6aeRQ6.png)

- 成功在裡面找到user flag
![image](https://hackmd.io/_uploads/B1WlAlAm6.png)

- 先嘗試sudo -l 沒有權限
![image](https://hackmd.io/_uploads/H1uXRe07p.png)

- 用wget想說把檔案下載下來可是失敗
![image](https://hackmd.io/_uploads/HysDMbR7T.png)
![image](https://hackmd.io/_uploads/Hk8cMWA7p.png)

- 最後用scp複製下來
```
scp [帳號@來源主機]:來源檔案 [帳號@目的主機]:目的檔案
```
![image](https://hackmd.io/_uploads/ByMkAZC76.png)

- 解壓縮完後得到了KeePassDumpFull.dmp  passcodes.kdbx 兩個檔案
![image](https://hackmd.io/_uploads/S1CmJz0X6.png)

- passcodes是keepass password的資料庫 keepassdump是當程式crush時的轉儲文件
![image](https://hackmd.io/_uploads/Syv-DvRXa.png)

- 用keepass2去打開passcodes.kdbx需要他的主密碼
![image](https://hackmd.io/_uploads/SycY8P0mp.png)


