:::info
# Fanatastic
## Difficulty : Easy
## Date : 11/11
:::

一開始先使用 nmap 去掃他看他有開啟哪些 port 發現有開啟以下的
![image](https://hackmd.io/_uploads/r1Wh7S1Mke.png)

去訪問他的 3000 port 的時候會進入到一個登入畫面 可以看到他的右下有版本號 v8.3.0
![image](https://hackmd.io/_uploads/r1ek4HyGJx.png)

同時去 google 一下了解 Grafana 是什麼，主要是一個開源的 ELK Stack 可視化工具

先去嘗試他的預設帳密 admin/admin 無法登入
![image](https://hackmd.io/_uploads/By_6LHJGyg.png)

那由於知道他的版本號 所以去查看看有沒有相關的漏洞 發現有一個 [CVE-2021-43798 目錄遍歷](https://blog.csdn.net/m0_52923241/article/details/129462736) 的漏洞

主要原理 : `pkg/api/plugins.go` 文件的 `getPluginAssets` 函數。先查找插件是否存在，如果插件不存在，則返回 "Plugin not found"，如果存在這個插件名，則會匹配 `/public/plugins/:pluginId/*` 中的 *，這個地方沒有做任何過濾。最後打開`pluginFilePath` 並輸出相關內容，就可以造成任意文件讀取。

因此 payload 會長這樣 : `public/plugins/插鍵名稱/想要訪問文件的相對路徑`

那由於他的插鍵非常多 你可以把他全部都導進去 burpsuite 做字典檔爆破 網路上也有別人寫好的腳本可以使用
![image](https://hackmd.io/_uploads/ByA23AlzJx.png)

使用 `searchsploit grafana` 可以找到相關漏洞腳本
![image](https://hackmd.io/_uploads/r1EmrJ-zkx.png)

把他下載下來 `searchsploit -m 50581.py`
![image](https://hackmd.io/_uploads/BkqLSJ-fJl.png)

先 python3 看一下如何使用
![image](https://hackmd.io/_uploads/BJ4jrJWfkg.png)

可以成功讀取 `/etc/passwd`
![image](https://hackmd.io/_uploads/SJEtgTpGJl.png)

但不知道要去看什麼 想說先去看另一個有開的 9090 port
他運行的是一套用於監控與警報的開源軟體，會將即時的 Metrics 資料儲存到時間序列資料庫，並透過 HTTP 接口讓使用者查詢以及設定預警規則，讓我們能輕鬆管理 Metrics 
![image](https://hackmd.io/_uploads/SJ59v0zfke.png)

先到處翻翻 在 status 有看到一個 user 想說能不能拿去登入 ssh 發現失敗
![image](https://hackmd.io/_uploads/HkvTuCfM1g.png)

也找不到可以利用的地方 最後是想說應該回去找看看 grafana 然後去讀取他的 db 或 config 檔 看裡面有沒有一些帳號密碼可以用 

根據這邊[文章](https://stackoverflow.com/questions/65860003/physical-location-of-grafana-dashboards)找到 db 位置 `/var/lib/grafana/gradana.db`

但他是 db 應該要載下來用 sql 開 所以最後還是要去爆他的 burpsuite
可以發現他幾乎每個插鍵都有
![image](https://hackmd.io/_uploads/H1g7f1mfJx.png)

於是便可以透過 curl 把 db 下載下來 `curl --path-as-is http://192.168.246.181:3000/public/plugins/alertlist/../../../../../../../../var/lib/grafana/grafana.db -o grafana.db`

把他丟進去 sqlite browser 打開
在 user table 有找到一組帳號密碼 但無法解開
```
63f576276a6db59bb750c34f126945c1e941f9e3b21ab2f5be74ae00cc8abfc1b9f7ee5840f9abdae46efc0ee5350bd65aa8
```
在 data_source 也有找到一組帳號密碼
這個好像就可以破譯
```
{"basicAuthPassword":"anBneWFNQ2z+IDGhz3a7wxaqjimuglSXTeMvhbvsveZwVzreNJSw+hsV4w=="}
```

需要利用這個[漏洞](https://github.com/jas502n/Grafana-CVE-2021-43798?source=post_page-----b14a6e535e1f--------------------------------)來進行破解(不看 writeup 根本找不到)

用[這個](https://goplay.tools/)來跑 go 程式 成功解出密碼為 SuperSecureP@ssw0rd

拿 prometheus 去 ssh 登入失敗 想到 `/etc/passwd` 還有另一個 user : sysadmin

成功登入
![image](https://hackmd.io/_uploads/S1Mrbp6zyg.png)

linpeas 發現 id 有多一個 disk
![image](https://hackmd.io/_uploads/S1yhBaTGkg.png)

可以參考[這一篇](https://vk9-sec.com/disk-group-privilege-escalation/?source=post_page-----b14a6e535e1f--------------------------------)

`dh -h` 看一下硬碟在哪 之後就可以用 debugfs 去讀取
```
debugfs /dev/sda2
```

因為硬碟有 root 權限 所以直接去讀他的 ssh key 登入
![image](https://hackmd.io/_uploads/Syleitaafyx.png)

記得要修改 ssh key 權限 成功登入拿到 flag
![image](https://hackmd.io/_uploads/r1Swq6aGJl.png)
