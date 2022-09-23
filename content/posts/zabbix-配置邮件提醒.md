---
title: zabbix 配置邮件提醒
date: 2020-09-15T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["Linux","运维","云原生"]
author: "Esqer"
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: false
description: "工作中频繁用到的git命令"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/EsqerYasen/EsqerYasen.github.io/content"
    Text: "test page" # edit text
    appendFilePath: true # to append file path to Edit link
---
配置邮件

* * *
环境 
系统：centos 7.6
zabbix 版本：4.2
* * *
QQ个人邮箱: smtp..qq.com（`QQ企业邮箱: smtp.exmail.qq.com`）
>#POP3/SMTP协议  
>接收邮件服务器：pop.exmail.qq.com ，使用SSL，端口号995  
>发送邮件服务器：smtp.exmail.qq.com ，使用SSL，端口号465  
>#海外用户可使用以下服务器  
>接收邮件服务器：hwpop.exmail.qq.com ，使用SSL，端口号995  
>发送邮件服务器：hwsmtp.exmail.qq.com ，使用SSL，端口号465 


常用命令

```
查看邮件日志：cat /var/log/maillog  
配置邮件：     vim /etc/mail.rc  
编辑主机地址：vim /etc/hosts 
```
 配置邮件
 企业邮箱配置实例
 
 
> vim /etc/mail.rc
>#使用SSL的方式发送邮件 增加如下关于SSL的配置
>set nss-config-dir=/etc/pki/nssdb/
>set smtp-user-starttls
>set ssl-verify=ignore
 set from=aisikeer.yasen@企业名称.com
set smtp=smtps://smtp.exmail.qq.com:465
set smtp-auth-user=aisikeer.yasen@企业名称.com
set smtp-auth-password=XXXXXXX
set smtp-auth=login


执行shel命令生成认证
>
>echo -n | openssl s_client -connect smtp.exmail.qq.com:465 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > /etc/pki/nssdb/qq.crt 
>certutil -A -n "GeoTrust SSL CA" -t "C,," -d /etc/pki/nssdb/ -i /etc/pki/nssdb/qq.crt  
>certutil -A -n "GeoTrust Global CA" -t "C,," -d /etc/pki/nssdb/ -i /etc/pki/nssdb/qq.crt  
>certutil -L -d /etc/pki/nssdb/ 
>certutil -A -n "GeoTrust SSL CA - G3" -t "Pu,Pu,Pu"  -d ./ -i qq.crt  #认证
 
 
发送邮件测试：
命令行: mail -v -s "主题" aisikeer.yasen@企业名称.com ,回车后输入内容按Ctrl+D发送邮件.
管道符: 
echo "mail main content" | mail -v -s "title" addressee 
echo "mail content" | mail -s "title" addressee
文件内容作为邮件内容: mail -v -s "title" addressee < /test.txt 

```
[root@wiki-poc ~]# echo "this is test mail" | mail -v -s "test" aisikeer.yasen@企业名称com
Resolving host smtp.exmail.qq.com . . . done.
Connecting to 58.251.82.205:465 . . . connected.
Comparing DNS name: "*.exmail.qq.com"
SSL parameters: cipher=AES-128, keysize=128, secretkeysize=128,
issuer=CN=DigiCert SHA2 Secure Server CA,O=DigiCert Inc,C=US
subject=CN=*.exmail.qq.com,OU=R&D,O=Tencent Technology (Shenzhen) Company Limited,L=Shenzhen,ST=Guangdong,C=CN
220 smtp.qq.com Esmtp QQ Mail Server
>>> EHLO wiki-poc
250-smtp.qq.com
250-PIPELINING
250-SIZE 73400320
250-AUTH LOGIN PLAIN
250-AUTH=LOGIN
250-MAILCOMPRESS
250 8BITMIME
>>> AUTH LOGIN
334 VXNlcm5hbWU6
>>> YWlzaWtlZXIueWFzZW5AOTUzMDMuY29t
334 UGFzc3dvcmQ6
>>> OTQwMzE2QWE=
235 Authentication successful
>>> MAIL FROM:<aisikeer.yasen@企业名称.com>
250 Ok
>>> RCPT TO:<aisikeer.yasen@企业名称.com>
250 Ok
>>> DATA
354 End data with <CR><LF>.<CR><LF>
>>> .
250 Ok: queued as 
>>> QUIT
221 Bye

```
配置和配置验证完成


* * *

zabbix服务器端编写邮件发送脚本

1.修改zabbix_server.conf配置文件，指定zabbix
```
vim /usr/local/zabbix/etc/zabbix_server.conf  
#修改alertscripts为以下路径
AlertScriptsPath=/usr/local/zabbix/share/zabbix/alertscripts  ##存放脚本目录
```
2.创建邮件发送脚本 
(1)在zabbix 2.x版本中，当有报警通知时，默认会传3个参数给脚本，它分别为是:

>$1（发送给谁）
>$2（发送标题）
>$3（发送内容）。

例如发送邮件给test@qq.com,编辑如下脚本：

```
mkdir /usr/local/zabbix/alertscripts #创建脚本目录
vim /usr/local/zabbix/alertscripts/sendmail.sh  #编辑脚本，以下为脚本内容
```
 
```
#!/bin/bash
messages=`echo $3 | tr '\r\n' '\n'`
subject=`echo $2 | tr '\r\n' '\n'`
echo "${messages}" | mail -s "${subject}" $1 >>/tmp/mailx.log 2>&1
```

（2）更改属主及赋予可执行权限

```
chown -R zabbix.zabbix  /tmp/mailx.log
chmod +x /usr/local/zabbix/alertscripts/mailx.sh
chown -R zabbix.zabbix /usr/local/zabbix/
```
（3）测试邮件发送脚本

```
/usr/local/zabbix/alertscripts/sendmail.sh test@qq.com "测试邮件标题" "测试邮件内容"
```
但从zabbix 3.0之后，可以自定义参数了，所以不写参数，它是不会传参数给脚本的，需要注意：

vim /etc/zabbix/zabbix_server.conf
#修改alertscripts为以下路径
AlertScriptsPath=/usr/local/zabbix/alertscripts
 
 
 
#重启zabbix服务
service zabbix_server restart

zabbix 前端页面测试邮件测试是否正确。


确认主题和内容参数顺序是否正确。