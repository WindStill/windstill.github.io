---
layout: post
title: "SSL证书申请与在Nginx下的配置"
date: 2014-11-05 14:23:45 +0800
comments: true
categories: [SSL, Nginx]
---

http协议默认情况下是不加密的，各种信息都是明文传送。SSL是Secure Socket Layer的简称，具体的作用就是在部署了SSL证书的网站跟用户浏览器之间建立一个安全的会话。以防止信息在互联网任何中间节点上被盗用。

####购买SSL
如果购买就不赘述了，我是从 [SSLs.com](https://www.ssls.com/)购买的**Positive SSL**，$4.99/年。可以比较下其他提供商，按需购买。

####申请证书
一般购买SSL后，激活的时候会要求提供一下 CSR 的内容。CSR 就是 Certificate Signing Request 简称，需要去部署的服务器上生成一个。步骤如下：

```
sudo mkdir /etc/nginx/ssl
cd /etc/nginx/ssl  

#生成private key
sudo openssl genrsa -des3 -out server.key 2048
这里问你输入一个passphrase，选择一个容易记得，后面会用到

#生成 CSR
sudo openssl req -new -key server.key -out server.csr

Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:                  
Email Address []:
A challenge password []:
An optional company name []:
```
输入内容说明：

| 输入内容| 说明 |
|:----|:----|
|Country Name (2 letter code) | 使用国际标准组织(ISO)国码格式，填写2个字母的国家代号。中国请填写CN|
|State or Province Name (full name)| 省份，如 Beijing|
|Locality Name (eg, city)| 城市，如 Beijing|
|Organization Name (eg, company)|公司名称|
|Organizational Unit Name|部门名称|
|Common Name (eg, your websites domain name)|网站域名，注意区分abc.com和www.abc.com是不同的|
|Email Address|邮件地址，可不填|
|A challenge password|可不填|
|An optional company name|可不填|

<!-- more -->

然后将CSR文件内容复制到激活SSL时需要填写CSR的地方，过程中还会要求选择你的管理员邮箱，一定要选择可靠、存在的邮箱，因为证书是通过邮箱发送给你的。

完成之后，邮箱会收到一个Approval的邮件，点击连接Approve。下一个邮件就会收到证书了。收到证书后，再/etc/nginx/ssl文件夹下面新建一个server.crt的文件，把证书内容粘贴进去。

####配置Nginx

```
server {
        listen 443;
        server_name example.com;

        root /usr/share/nginx/www;
        index index.html index.htm;

        ssl on;
        ssl_certificate /etc/nginx/ssl/server.crt;
        ssl_certificate_key /etc/nginx/ssl/server.key; 
}
```
重启 Nginx，需要输入之前生成 private key 时填写的 passphrase。可以去掉private key的 passphrase 让 Nginx 自由启动，不再输入。

```
sudo cp server.key server.key.org
sudo openssl rsa -in server.key.org -out server.key
```