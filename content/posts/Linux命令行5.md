---
title: "Linux命令行 3  history 命令"
date: 2020-09-15T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["Linux","运维","云原生"]
author: "Esqer"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "history 命令用于查看历史命令和历史操作"
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
history 命令用于查看历史命令和历史操作


history命令用于显示指定数目的指令命令，读取历史命令文件中的目录到历史命令缓冲区和将历史命令缓冲区中的目录写入命令文件。该命令单独使用时，仅显示历史命令，在命令行中，可以使用符号!执行指定序号的历史命令。
例如，要执行第2个历史命令，则输入!2。
历史命令是被保存在内存中的，当退出或者登录shell时，会自动保存或读取。
在内存中，历史命令仅能够存储1000条历史命令，该数量是由环境变量HISTSIZE进行控制。
语法history(选项)(参数)选项
```
-c：清空当前历史命令；
-a：将历史命令缓冲区中命令写入历史命令文件中；
-r：将历史命令文件中的命令读入当前历史命令缓冲区；
-w：将当前历史命令缓冲区命令写入历史命令文件中。
```
参数n：打印最近的n条历史命令。
实例使用history命令显示最近使用的10条历史命令，输入如下命令：

`[root@localhost ~]# history 10`

可以利用上下箭头按钮切换到最近使用的命令
!序号 执行序号对应的命令