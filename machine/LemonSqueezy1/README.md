![](https://hackmd.io/_uploads/rkqMxei5n.png)

透過nmap掃描靶機IP有哪個port有開放 發現其中port 80是開放的

![](https://hackmd.io/_uploads/ryuE-gj5n.png)

進去網頁和原始碼查看後發現沒有什麼有用的資訊後選擇使用dirb進行探測
![](https://hackmd.io/_uploads/HJWtMni9n.png)

裡面有phpmyadmin和wordpress，先對wordpress進行wpscan探測
![](https://hackmd.io/_uploads/ry__7x25h.png)
![](https://hackmd.io/_uploads/HyX5Xl25n.png)

找到裡面有lemon跟orange兩個user，嘗試爆破他們倆的密碼
![](https://hackmd.io/_uploads/SJxUFgncn.png)
![](https://hackmd.io/_uploads/Sy0wtxn5n.png)

找到orange的密碼為ginger 而lemon的暫時沒找到先放棄

![](https://hackmd.io/_uploads/HJ4e2Zhqn.png)

![](https://hackmd.io/_uploads/ry5b2Zn5h.png)

把lemonsqueezy加入到host
![](https://hackmd.io/_uploads/SyfB3Wn9n.png)

再來登入wordpress 通常路徑為wp-admin
![](https://hackmd.io/_uploads/SyP9h-25h.png)
在裡面找到一個可能為phpmyadmin的密碼
![](https://hackmd.io/_uploads/HJW-pb392.png)

登入phpmyadmin
![](https://hackmd.io/_uploads/Hk1spiPsn.png)

在裡面建立一個簡易的後門
![](https://hackmd.io/_uploads/H1JNhBfi3.png)
確認有成功建立後門
![](https://hackmd.io/_uploads/HkcshSfj2.png)

在web使用連結連到主機

![](https://hackmd.io/_uploads/ryjbMGlj3.png)

確認有連結成功上


![](https://hackmd.io/_uploads/rJp6LFms3.png)

用python -c 'import pty;pty.spawn("/bin/bash")'取得shell

![](https://hackmd.io/_uploads/SyHox3Xjh.png)

下載LinEnum解壓縮後放到網路上
![](https://hackmd.io/_uploads/HykKZnQs3.png)
之後在LemonSqueezy端把他下載下來
![](https://hackmd.io/_uploads/S1Nyo2Qsh.png)

給他加上權限並執行

![](https://hackmd.io/_uploads/Skbbo6mj3.png)

發現裏頭有/etc/logrotate.d/logrotate自動排程腳本

![](https://hackmd.io/_uploads/B11QS0Qj2.png)

對這個腳本進行修改
![](https://hackmd.io/_uploads/B121uyVjn.png)

把bash權限提升並確認有沒有成功寫入
![](https://hackmd.io/_uploads/Bkqr_y4j2.png)

輸入/bin/bash -p取得root權限
![](https://hackmd.io/_uploads/Byn9u1Ejh.png)

確認提升成功
![](https://hackmd.io/_uploads/BkCf3JNsn.png)

進入bash

![](https://hackmd.io/_uploads/r1UUhJEs2.png)

確認身分並移動到root 取得flag





