- 加到/etc/hosts
  
![](https://hackmd.io/_uploads/ByZvq_sMa.png)

- 先用nmap掃一下 有開啟22和80 port
  
![](https://hackmd.io/_uploads/r1Wr4ujza.png)

- dirb沒有掃到甚麼可以用的東西
  
![](https://hackmd.io/_uploads/SJCN2ujfT.png)

- 發現一個子域名 把她也加進host
  
![](https://hackmd.io/_uploads/HJgbA_jMa.png)

![](https://hackmd.io/_uploads/BJ_ia_iGp.png)
  
- 網站打開後是一個可以幫你生成數學公式的網站
  
![](https://hackmd.io/_uploads/SkpmW9ofp.png)

- 查看關於latex有沒有特別的漏洞[LaTex Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LaTeX%20Injection)

- 發現有讀取文件的漏洞 把每條拿去嘗試
  
![](https://hackmd.io/_uploads/BylnGqsGa.png)

- 其他條都會被擋掉，只有\lstinputlisting{/etc/passwd}可以，但要前後都加上$，這樣可以用數學模式運行這個latex命令。
  
![](https://hackmd.io/_uploads/Hkb9X9jG6.png)
![](https://hackmd.io/_uploads/ByIim9jza.png)

- 可以成功運行，這時嘗試去找htpasswd，在 apache中，如果用http而不是https訪問某個網站目錄時，出现了401錯誤，表示需要驗證，驗證的密碼通常會以hash方式存在.htpasswd文件中 (要如何找到這文件的路徑?)

- 因此通過$\lstinputlisting{/var/www/dev/.htpasswd}$得到一段hash (vadaisley:$apr1$1ONUB/S2$58eeNVirnRDB5zAIbIxTY0)

![](https://hackmd.io/_uploads/SJ8uE5jG6.png)

- 用john來爆破密碼 (calculus20    (vadaisley) )
  
![](https://hackmd.io/_uploads/HysD95iMa.png)

- 拿來進行ssh連線
  
![](https://hackmd.io/_uploads/rkOXj5ofa.png)

- 得到user flag
  
![](https://hackmd.io/_uploads/B1YYsqiMa.png)

- sudo -l沒有結果
  
![image.png](https://hackmd.io/_uploads/SJOFUpJ76.png)

- 查看定時任務也沒有可以使用的
  
![image.png](https://hackmd.io/_uploads/BJz6LTJXT.png)

- 給pspy64權限 檢查現在有甚麼proces在運行 (為甚麼用ps auxwww不行)
  
![](https://hackmd.io/_uploads/rJMg9pjGa.png)

- 在opt路徑下發現gnuplot (找opt路徑可能的原因:通常都放第三方程式以及一些可執行的文件，因此可能有漏洞)

- 查看版本號並去找看看有沒有漏洞 可是找不到
  
![](https://hackmd.io/_uploads/ry2lHAifT.png)

- 發現有find /opt/gunplot -name *.plt -exec gunplot {} 這段的意思在/opt/gunplot这个目錄下搜索 *.plt的文件然後作为gunplot的參數執行
  
![](https://hackmd.io/_uploads/rJBVERjzp.png)

- 1.創建someb0dy1.plt的文件並把其中的内容設置為執行cp /bin/bash /tmp/someb0dy;chmod u+s /tmp/someb0dy的命令把someb0dy1.plt 文件複製到 /opt/gnuplot/目錄 3.執行bash -p
- 目的是複製 /bin/bash 到 /tmp/someb0dy並設置SUID權限來獲得root
  
![](https://hackmd.io/_uploads/rJw3Wynzp.png)

- 成功提權並拿到flag
  
![](https://hackmd.io/_uploads/Bk_BWy2GT.png)
