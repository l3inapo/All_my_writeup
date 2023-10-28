# HTB-Sau WriteUp
- 登入後點擊htb 連線 隨便選一個連線，下載vpn檔
![](https://hackmd.io/_uploads/ryvfjDxxT.png)

- 在console開啟並連線，就可以連到了
![](https://hackmd.io/_uploads/B1nroPeg6.png)

- 用nmap掃一下發現80port被過濾，代表不能直接連上
![](https://hackmd.io/_uploads/ry-L2PleT.png)

- 嘗試去連555 port發現可以成功進去
![](https://hackmd.io/_uploads/SkNdhwgx6.png)

- 在右下角有版本號 去查看看有沒有漏洞
![](https://hackmd.io/_uploads/SypH8eNgT.png)

- 漏洞原理:request baskets服務存在api接口/api/baskets/{name}，/baskets/{name}，這里的name可以任意，這些接口會接收一個forward_url參數，而這個參數存在SSRF漏洞。

向/api/baskets/{name}發送forward_url參數後，需要訪問10.10.11.224:55555/{name}來觸發SSRF漏洞（請求給定的forward_url）。

利用方法：
之前我們發現靶機的80端口無法直接訪問，因此我們可以通過該漏洞訪問靶機的80端口

![](https://hackmd.io/_uploads/B1TRYlVeT.png)
![](https://hackmd.io/_uploads/SJhJ5eVea.png)
再訪問剛剛創建的url了
![](https://hackmd.io/_uploads/HyJV5eElp.png)


- 進來後一樣去查 maltrail(v0.53)有甚麼漏洞，發現有[https://github.com/spookier/Maltrail-v0.53-Exploit]

- 可以利用他給的py檔進行rce
![](https://hackmd.io/_uploads/Sk6p9lNg6.png)

- 先開啟一個監聽
![](https://hackmd.io/_uploads/Bkf-hl4gp.png)

- 再執行剛剛下載的py檔
 ![](https://hackmd.io/_uploads/HywQ3eVgp.png)

- 就成功監聽到的
![](https://hackmd.io/_uploads/BJE8nl4x6.png)

- 之後不斷的cd ls就可以找到第一個flag
![](https://hackmd.io/_uploads/BkFh3xVxp.png)

- 當發現沒有terminal外殼時，可以使用python3 -c 'import pty;pty.spawn("/bin/bash")' 生成一個虛擬終端，方便進行提權以及bash指令互動

![](https://hackmd.io/_uploads/ByqjbZEga.png)

- 輸入sudo -l發現沒有鎖，並且發現裡面有提示
![](https://hackmd.io/_uploads/Syvnz-4x6.png)

- 去[gtfobins](https://gtfobins.github.io/)查詢systemctl的漏洞
    -  GTFOBins，這個工具收集了 Linux 中可以被濫用的正常指令

- 發現可以用!sh進行提權
![](https://hackmd.io/_uploads/SkXx7ZEeT.png)

- 輸入後成功提權
![](https://hackmd.io/_uploads/SkbXXWNgp.png)

- 之後不斷cd ls便可找到flag
![](https://hackmd.io/_uploads/HJkP7WNep.png)

