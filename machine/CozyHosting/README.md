- 先sudo vi /etc/hosts把ip加進去
![](https://hackmd.io/_uploads/HkrFvlnfT.png)

- nmap掃描後發現有開啟80,4444,5555 port
![](https://hackmd.io/_uploads/SJPDdx3M6.png)

- 也用dirbsearch掃看看
![](https://hackmd.io/_uploads/Hkigix2Ga.png)

- 發現有個/actuator (Actuator是Spring Boot提供的對應用系統的自省和監控的集成功能，可以查看應用配置的詳細信息，例如自動化配置信息、創建的Spring beans以及一些環境屬性等。) [actuator介紹](https://blog.csdn.net/JINXFOREVER/article/details/104945214?spm=wolai.workspace.0.0.6da2767bDhFTDY)
1. mappings顯示所有@RequestMapping路徑的整理列表
2. health獲取應用程式健康狀態（運行狀況信息）
3. env獲取所有環境變量
4. sessions允許Spring Session支持的會話存取中檢索和刪除用户會話。需要使用Spring Session的基於Servlet的Web應用程序
5. beans獲取應用中所有的Spring Beans 的完整關係列表
![](https://hackmd.io/_uploads/HkSvoehzp.png)

- 去看看env有放什麼，沒有甚麼可以用的,health mappings beans也一樣
![](https://hackmd.io/_uploads/ry3ckW2MT.png)

- 而最有可能有用的是sessions，發現裡面有kanderson的session
![](https://hackmd.io/_uploads/SJBMnc3za.png)

- 利用kanderson的session進入admin，先用burp攔截login後把session改掉並把login改成admin 就可以成功進入 dashboard
![](https://hackmd.io/_uploads/HkeT05nMT.png)

- 在下面有看到一個可以建立ssh連線的地方
![](https://hackmd.io/_uploads/BkXXBi2fa.png)

- 先抓個包試試，隨便輸入會跳出connect timeout
![](https://hackmd.io/_uploads/SyCZKi3zT.png)

- 把host設為空的話會跳出invalid host name，但是把username設為空的話會跳出ssh usage，感覺可以用命令注入，因為感覺是透過command來實現這功能
![](https://hackmd.io/_uploads/ryoHqshz6.png)

- 輸入ls 後發現可以進行命令注入
![](https://hackmd.io/_uploads/HkMDyn3zT.png)

- 發現禁止有空格鍵
![](https://hackmd.io/_uploads/Sk8Bb2hzp.png)

- 在網路上有看到一個payload，${IFS}是一個 (Internal Field Separator)，而在默認的情況下通常當像是空格、逗號或是換行的字符串，因此可以繞過空格問題
```
 ;echo${IFS}"[ PAYLOAD ]"|base64${IFS}-d|bash;
```
- 先開啟一個監聽
![](https://hackmd.io/_uploads/HyE5fn3fa.png)

- 嘗試注入reverse shell並看有沒有監聽成功
![](https://hackmd.io/_uploads/HJrrMhhza.png)

- 成功get app的shell 並在裏頭發現一個 jar檔
![](https://hackmd.io/_uploads/SJ27Q3hGa.png)

- 在那邊架一個server並在本地使用wget 把jar檔下載下來
![](https://hackmd.io/_uploads/S18c6a3fT.png)

- 在jar檔裏頭找看看設定檔，有發現application.properties，裡面有postgresql的帳號密碼
![](https://hackmd.io/_uploads/Sk7bG1TMT.png)

- 先在app端加上pty介面
![](https://hackmd.io/_uploads/SJpBMkTG6.png)

- 再進行連線 psql -h 127.0.0.1 -p 5432 -U postgres ( -U是指username
![](https://hackmd.io/_uploads/rk-UIJpGa.png)

- psql操作方法 \list是列出所有資料庫，\c是切換到指定資料庫 \dt是列出所有table
- 先看有什麼database 並切換到cozyhosting 列出他所有table
![](https://hackmd.io/_uploads/rydSsJ6G6.png)

![](https://hackmd.io/_uploads/SJV_oy6fa.png)
- 而我們想要的資料應該會放在user裡 ，查看select裡面有放甚麼
![](https://hackmd.io/_uploads/S1RN31pfT.png)
- 裡面有hash 拿去用john爆破看看 有一組密碼manchesterunited
![](https://hackmd.io/_uploads/S1X_RJ6Mp.png)

- 查看/etc/passwd在裡面有發現josh用戶
![](https://hackmd.io/_uploads/BJD2lg6fT.png)

![](https://hackmd.io/_uploads/SkR6geaGp.png)

- 用josh跟那組密碼去連線試看看 可以成功連線
![](https://hackmd.io/_uploads/Bkgr9lTzT.png)

- 拿到user flag
![](https://hackmd.io/_uploads/ByXDqx6fT.png)

- sudo -l看看有什麼可以利用的，發現它可以用ssh提權
![](https://hackmd.io/_uploads/HkHhox6MT.png)

- 去GTFOBins看關於ssh提權有什麼可以用的
![](https://hackmd.io/_uploads/HyG7hxTfp.png)

- 成功提權
![](https://hackmd.io/_uploads/BysUnxpMa.png)



關於這個提權的解釋

- `-o ProxyCommand=';sh 0<&2 1>&2'`: specifies a custom ProxyCommand configuration using the `-o` option. ProxyCommand is used to specify a command that should be executed to establish the SSH connection through a proxy or intermediate server.
- `sh`: This is the shell (usually `/bin/sh` or `/bin/bash`) being invoked as a command.
- `0<&2`: redirects file descriptor 0 (stdin) to file descriptor 2 (stderr).
- `1>&2`: redirects file descriptor 1 (stdout) to file descriptor 2 (stderr).
- When combined, `sh 0<&2 1>&2` essentially redirects both standard input and standard output to standard error. This redirection is a common technique used in reverse shell payloads.
- `x`: This is the target hostname or IP address to which the SSH connection is being established.

> ssh中的參數選項ProxyCommand表示用一个代理服務器連接到遠程主機，是在進行ssh操作之前就執行的命令，此處正是利用這個機制進行提權。-o表示option，引出ProxyCommand，';bash 0<&2 1>&2'是核心的提權邏輯，啟動bash后對输入输出進行了重定向，但如果僅僅這樣寫（不加分號的話），引號中的內容會作為ssh的通道，完成代理服務，此時在ssh進行密鑰交換得时候會無法與主機通信導致錯誤。分號就是起了這樣的作用，分號的前面隐含了一個空語句，空語句一定能執行成功，只有分號前面的語句執行成功时後面的提權命令才會執行，这是bash的邏輯。最後的x就表示要連接的主機，随便寫個啥都行，只要參數不空即可。
————————————————
原文連結：[https://blog.csdn.net/Bossfrank/article/details/132662689](https://blog.csdn.net/Bossfrank/article/details/132662689)