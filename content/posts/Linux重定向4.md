---
title: "nohup命令，重定向和用途"
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

nohup
用途：不挂断地运行命令。
语法：nohup Command [ Arg ... ] [　& ]
描述：nohup 命令运行由 Command 参数和任何相关的 Arg 参数指定的命令，忽略所有挂断（SIGHUP）信号。在注销后使用 nohup 命令运行后台中的程序。要运行后台中的 nohup 命令，添加 & （ 表示“and”的符号）到命令的尾部。操作系统中有三个常用的流：
　　0：标准输入流 stdin
　　1：标准输出流 stdout
　　2：标准错误流 stderr      
  一般当我们用 

  >  \>console.txt，实际是 1>console.txt的省略用法；
  < console.txt ，实际是 0 < console.txt的省略用法。

  示例用法1：`nohup java -jar xx.jar >output 2>&1 &`
  &解释：
    1.带&的命令行，即使terminal（终端）关闭，或者电脑死机程序依然运行（前提是你把程序递交到服务器上)； 
    2.  2>&1的意思　　这个意思是把标准错误（2）重定向到标准输出中（1），而标准输出又导入文件output里面，所以结果是标准错误和标准输出都导入文件output里面了。 至于为什么需要将标准错误重定向到标准输出的原因，那就归结为标准错误没有缓冲区，而stdout有。这就会导致 >output 2>output 文件output被两次打开，而stdout和stderr将会竞争覆盖，这肯定不是我门想要的. 这就是为什么有人会写成： 
    `nohup ./command.sh >output 2>output` (这是错的 应该写 2>&1)
    出错的原因了 。      
    0,1,2可以用来指定需要重定向的标准输入或输出。在一般使用时，默认的是标准输出，既1。当我们需要特殊用途时，可以使用其他标号。
    例如，将某个程序的错误信息输出到log文件中：`./program 2>log`，这样标准输出还是在屏幕上，但是错误信息会输出到log文件中。另外，也可以实现0，1，2之间的重定向。2>&1：将错误信息重定向到标准输出。
      Linux下还有一个特殊的文件/dev/null，它就像一个无底洞，所有重定向到它的信息都会消失得无影无踪，任何东西都可以定向到这里，但是却无法打开。这一点非常有用，一般很大的stdou和stderr当你不关心的时候或者当我们由于其他原因不需要回显程序的所有信息时，就可以将输出重定向到/dev/null。
      例如：# 
      `ls 1>/dev/null 2>/dev/null`
      还有一种做法是将错误重定向到标准输出，然后再重定向到 /dev/null，
      例如：# ls >/dev/null 2>&1
      注意：此处的顺序不能更改，否则达不到想要的效果，此时先将标准输出重定向到 /dev/null，然后将标准错误重定向到标准输出，由于标准输出已经重定向到了/dev/null，因此标准错误也会重定向到/dev/null。           由于使用nohup时，会自动将输出写入nohup.out文件中，如果文件很大的话，nohup.out就会不停的增大，这是我们不希望看到的，因此，可以利用/dev/null来解决这个问题。
      `# nohup ./program >/dev/null 2>log &`  
    如果错误信息也不想要的话：
    `# nohup ./program >/dev/null 2>&1 &`