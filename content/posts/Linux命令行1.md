---
title: "Linux命令行 1 - 格式"
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
description: "Linux笔记____命令行"
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
***
2019/5/22

Linux命令格式

* command [options] [arguments]
* command：命令options：  --单词   或   -单字
* 如： ls --all
* 使用Ctrl+D组合键退出Shell会话
* 大都数命令可以通过 --help 获取帮助
```
cat --help
用法：cat [选项]... [文件]...
将[文件]或标准输入组合输出到标准输出。

  -A, --show-all           等于-vET
  -b, --number-nonblank    对非空输出行编号
  -e                           等于-vE
  -E, --show-ends          在每行结束处显示"$"
  -n, --number             对输出的所有行编号
  -s, --squeeze-blank      不输出多行空行
  -t                             与-vT 等价
  -T, --show-tabs          将跳格字符显示为^I
  -u                           (被忽略)
  -v, --show-nonprinting   使用^ 和M- 引用，除了LFD和 TAB 之外
      --help		显示此帮助信息并退出
      --version		显示版本信息并退出

如果没有指定文件，或者文件为"-"，则从标准输入读取。

示例：
  cat f - g  先输出f 的内容，然后输出标准输入的内容，最后输出g 的内容。
  cat        将标准输入的内容复制到标准输出。
```
* 多行命令通过 `:`  隔开

