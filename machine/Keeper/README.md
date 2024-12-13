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
  
![image](https://hackmd.io/_uploads/Bk40VgAXp.png)

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

- 最後用scp複製下來 (不用連著 ssh 要關閉他再 scp)
  
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

- 把 kdbx 轉成 hash 要記得打開把不是 hash 部分給刪掉
  
![image](https://hackmd.io/_uploads/BJtTlQt4yl.png)

密碼爆不出來 他有另外給一個 dump 檔 上網查了後有相關 poc 可以拿來做爆破利用 [這個](https://github.com/matro7sh/keepass-dump-masterkey)

這是掃出來的結果 貼上都不能用 原因應該是特殊符號無法轉譯
![image](https://hackmd.io/_uploads/B1XKFQKVke.png)

貼到 google 後得到 Rødgrød med Fløde, 改成全小寫就登入成功了
![image](https://hackmd.io/_uploads/ByOaF7FN1x.png)


有一個管理員帳號 notes 是puTTy, 一種開源的遠端程式
```
PuTTY-User-Key-File-3: ssh-rsa
Encryption: none
Comment: rsa-key-20230519
Public-Lines: 6
AAAAB3NzaC1yc2EAAAADAQABAAABAQCnVqse/hMswGBRQsPsC/EwyxJvc8Wpul/D
8riCZV30ZbfEF09z0PNUn4DisesKB4x1KtqH0l8vPtRRiEzsBbn+mCpBLHBQ+81T
EHTc3ChyRYxk899PKSSqKDxUTZeFJ4FBAXqIxoJdpLHIMvh7ZyJNAy34lfcFC+LM
Cj/c6tQa2IaFfqcVJ+2bnR6UrUVRB4thmJca29JAq2p9BkdDGsiH8F8eanIBA1Tu
FVbUt2CenSUPDUAw7wIL56qC28w6q/qhm2LGOxXup6+LOjxGNNtA2zJ38P1FTfZQ
LxFVTWUKT8u8junnLk0kfnM4+bJ8g7MXLqbrtsgr5ywF6Ccxs0Et
Private-Lines: 14
AAABAQCB0dgBvETt8/UFNdG/X2hnXTPZKSzQxxkicDw6VR+1ye/t/dOS2yjbnr6j
oDni1wZdo7hTpJ5ZjdmzwxVCChNIc45cb3hXK3IYHe07psTuGgyYCSZWSGn8ZCih
kmyZTZOV9eq1D6P1uB6AXSKuwc03h97zOoyf6p+xgcYXwkp44/otK4ScF2hEputY
f7n24kvL0WlBQThsiLkKcz3/Cz7BdCkn+Lvf8iyA6VF0p14cFTM9Lsd7t/plLJzT
VkCew1DZuYnYOGQxHYW6WQ4V6rCwpsMSMLD450XJ4zfGLN8aw5KO1/TccbTgWivz
UXjcCAviPpmSXB19UG8JlTpgORyhAAAAgQD2kfhSA+/ASrc04ZIVagCge1Qq8iWs
OxG8eoCMW8DhhbvL6YKAfEvj3xeahXexlVwUOcDXO7Ti0QSV2sUw7E71cvl/ExGz
in6qyp3R4yAaV7PiMtLTgBkqs4AA3rcJZpJb01AZB8TBK91QIZGOswi3/uYrIZ1r
SsGN1FbK/meH9QAAAIEArbz8aWansqPtE+6Ye8Nq3G2R1PYhp5yXpxiE89L87NIV
09ygQ7Aec+C24TOykiwyPaOBlmMe+Nyaxss/gc7o9TnHNPFJ5iRyiXagT4E2WEEa
xHhv1PDdSrE8tB9V8ox1kxBrxAvYIZgceHRFrwPrF823PeNWLC2BNwEId0G76VkA
AACAVWJoksugJOovtA27Bamd7NRPvIa4dsMaQeXckVh19/TF8oZMDuJoiGyq6faD
AF9Z7Oehlo1Qt7oqGr8cVLbOT8aLqqbcax9nSKE67n7I5zrfoGynLzYkd3cETnGy
NNkjMjrocfmxfkvuJ7smEFMg7ZywW7CBWKGozgz67tKz9Is=
Private-MAC: b0a0fd2edf4f0e557200121aa673732c9e76750739db05adc3ab65ec34c5

```

看起來就是 ssh key 查詢後這個是 putty-ssh 的格式，需要把他轉成 openssh 的格式才能作使用 查到可以利用 `putty-tools` 裡的 `puttygen` 來修改格式
![image](https://hackmd.io/_uploads/SydykLKVyg.png)


成功 ssh 進去並拿到 flag `4933b2896bb5517a6669d31a5b1dbe3f`
![image](https://hackmd.io/_uploads/BJCBbUYVye.png)
