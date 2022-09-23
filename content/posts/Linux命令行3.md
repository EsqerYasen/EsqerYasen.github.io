---
title: "Linux命令行 3 - head,tail,wc"
date: 2020-09-15T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["o"]
author: "Esqer"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Linux命令____head,tail,wc"
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


head  从头开始输出
```
head -n file.log   显示某一日个日志文件的前n行 （n表示数字）
head     file.log   显示某一个文件的前十行
```

从最后开始输出
```
tail        catalina.out  显示后十行
tail -100 catalina.out  显示后100行
tail -100f catalina.out 从最后100行开始输出并实时输出更新日志
```

计算数量
```
[root@localhost logs]# wc -l catalina.out      ##计算行数
41 catalina.out
[root@localhost logs]# wc -w catalina.out     ##计算单词数量
422 catalina.out
[root@localhost logs]# wc -c catalina.out     ##计算字节数量
6547 catalina.out

```