- 先透過arp scan找尋靶機IP
![](https://hackmd.io/_uploads/S1HNRuvin.png)

---
- 確認靶機IP是否有連通
![](https://hackmd.io/_uploads/SJ68Ruvjh.png)

---
- 發現port 22 80 111是開放的
![](https://hackmd.io/_uploads/HylHbtPi3.png)

---
- 進行目錄爆破，裡面有三個網址
![](https://hackmd.io/_uploads/rkb1QYvin.png)

---
- 訪問http://192.168.253.132/dbadmin/，進去有地方可以登入，嘗試輸入admin就進去了
![](https://hackmd.io/_uploads/SkiKNtPo3.png)

---
- 在裏頭新增一個database並新增一個table，之後輸入一個sql指令<?php phpinfo();?>
![](https://hackmd.io/_uploads/SkYXdKDj3.png)

![](https://hackmd.io/_uploads/H1ZL_KDs3.png)

![](https://hackmd.io/_uploads/Sk5zFYwj2.png)

---
- 透過path traversal的方法可以成功訪問
![](https://hackmd.io/_uploads/r1UyY9Pin.png)

---
- 在weevely生成一個php檔並密碼設為pass
![](https://hackmd.io/_uploads/HkrtUsPih.png)

---
- 進去查看生成的php檔
![](https://hackmd.io/_uploads/Sy2MPjvsh.png)


---
- 把它複製起來輸入進database裡面
![](https://hackmd.io/_uploads/rJW6EjDi2.png)

![](https://hackmd.io/_uploads/B1dowjDi2.png)

---
- 進行連線
![](https://hackmd.io/_uploads/Hy7M_ovj2.png)

---
- 進去後在/home/zico/wordpress/wp-config.php發現有zico的帳號密碼
![](https://hackmd.io/_uploads/rJxl9iwsn.png)

---
- 連線進zico使用者並開始提權
![](https://hackmd.io/_uploads/SkzbjsPj3.png)

---
- 發現只有對tar 和 zip有權限
![](https://hackmd.io/_uploads/BkOAjiDo3.png)

---
- 透過zip進行提權 並確認身分
![](https://hackmd.io/_uploads/Byr_xnvih.png)

---
- 之後進去root並打開flag.txt就得到flag了
![](https://hackmd.io/_uploads/BkiVW2Djn.png)


