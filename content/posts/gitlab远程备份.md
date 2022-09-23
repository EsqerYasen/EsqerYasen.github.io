---
title: "Gitlab 远程备份，增量备份文档"
date: 2022-09-15T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["运维","云原生"]
author: "Esqer"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "简单介绍gitlab备份机制和几种简单方法"
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
    Text: "Edit This Page" # edit text
    appendFilePath: true # to append file path to Edit link
---


文件定时&&增量备份
配置两台机器ssh密钥
两台机器都要安装rsync
yum -y install rsync
rsync 是 remote sync的缩写，用于linux系统下的数据镜像备份工具，可以实现本地目录之间和远程服务器之间的文件拷贝。一.本地同步：
rsync -azp /root/要备份的目录 /备份目录
注意：注意以下两个命令的区别
>rsync -azp /root/A /root/B
>rsync -azp /root/A/ /root/B(或/root/B/)

第一个命令把A目录同步到B目录里面，第二个命令把A目录下的全部文件同步到B目录下。二.远程同步
两个机器ssh密钥认证完成以后彼此之间登陆和传输文件不需要输入密码。
创建一个脚本文件
>touch ***/gitlab_auto_backup.sh
>vim ***/gitlab_auto_backup.sh

```
#!/bin/bash
LocalBackDir=/var/opt/gitlab/backups  ##定义本地gitlab备份目录，这个目录跟gitlab.rb配置的备份目录一样
RemoteBackDir=/root/gitlab_backup    ##远程备份机器存放备份文件的目录
RemoteUser=root               ##远程机访问用户身份
RemoteIP=192.168.234.146         ##远程机器IP
DATE=`date +"%Y-%m-%d"`          ##输出日期，日期格式年月日>2018-08-10
LogFile=$LocalBackDir/$DATE.log     ##备份日存放目录，格式日期.log>2018-08-10.log
BACKUPFILE_SEND_TO_REMOTE=$(find $LocalBackDir -type f -mmin -60 -name '.tar')      ##查找本地备份目录下面60分钟内生成的.tar文件
touch $LogFile                                                                                                                                            
echo "Gitlab auto backup to remote server, start at $(date +"%Y-%m-%d %H:%M:%S")" >> $LogFile  ##把开始日期和开始备份输入到日志文件
echo "-------------------" >> $LogFile               ##空格，为了区分每次备份
echo "-----The file to rsync to remote server is: $BACKUPFILE_SEND_TO_REMOTE----" >> $LogFile  ##开始发送
rsync -avp $BACKUPFILE_SEND_TO_REMOTE $RemoteUser@$RemoteIP:$RemoteBackDir  ##同步，从本机目录同步到远程机指定目录
echo "------------------------------------------------" >> $LogFile  ##分割 三.定时备份到远程机器
```
配置crontab定时任务实现定时同步到远程机。

>1 1 * * * /opt/gitlab/bin/gitlab-rake gitlab:backup:create ##每天1点1分进行gitlab备份任务
>30 1 * * * sh /root/auto_backup_to_remote.sh ##每天1点30分运行上面代码，同步本地备份目录和远程机目录