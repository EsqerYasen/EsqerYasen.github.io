---
title: "Linux命令行 2 - type,passwd,date,file"
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
description: "Linux 命令行基础___type,passwd,date,file"
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

type (某一个命令)-可以查看这个命令是bash原生的命令还是第三方应用提供的命令
例如：

```
[root@localhost ~]# type passwd
passwd 是 /usr/bin/passwd
[root@localhost ~]# type date
date 是 /usr/bin/date
[root@localhost ~]# type cd
cd 是 shell 内嵌
[root@localhost ~]# 

```
   
passwd  密码操作相关的命令   
```
[root@localhost ~]# passwd --help
用法: passwd [选项...] <帐号名称>
  -k, --keep-tokens       保持身份验证令牌不过期
  -d, --delete               删除已命名帐号的密码(只有根用户才能进行此操作)
  -l, --lock                  锁定指名帐户的密码(仅限 root 用户)
  -u, --unlock              解锁指名账户的密码(仅限 root 用户)
  -e, --expire               终止指名帐户的密码(仅限 root 用户)
  -f, --force                 强制执行操作
  -x, --maximum=DAYS      密码的最长有效时限(只有根用户才能进行此操作)
  -n, --minimum=DAYS      密码的最短有效时限(只有根用户才能进行此操作)
  -w, --warning=DAYS       在密码过期前多少天开始提醒用户(只有根用户才能进行此操作)
  -i, --inactive=DAYS         当密码过期后经过多少天该帐号会被禁用(只有根用户才能进行此操作)
  -S, --status                   报告已命名帐号的密码状态(只有根用户才能进行此操作)
  --stdin                        从标准输入读取令牌(只有根用户才能进行此操作)

Help options:
  -?, --help               Show this help message
  --usage                 Display brief usage message
```

`passwd        `          修改当前用户       的密码
`passwd  user `   修改user的密码（只有root用户或者有root权限的用户才能执行）

锁定用户
```
[[root@localhost ~]# passwd  -l test
锁定用户 test 的密码 。
passwd: 操作成功
```
查看用户状态
```
[root@localhost ~]# passwd  -S test
test LK 2019-05-21 0 99999 7 -1 (密码已被锁定。)
```
解锁用户
```
[root@localhost ~]# passwd -u -f test
解锁用户 test 的密码。
passwd: 操作成功
```
清楚用户密码

```ate --help
用法：date [选项]... [+格式]
　或：date [-u|--utc|--universal] [MMDDhhmm[[CC]YY][.ss]]
Display the current time in the given FORMAT, or set the system date.

Mandatory arguments to long options are mandatory for short options too.
  -d, --date=STRING         display time described by STRING, not 'now'
  -f, --file=DATEFILE       like --date once for each line of DATEFILE
  -I[TIMESPEC], --iso-8601[=TIMESPEC]  output date/time in ISO 8601 format.
                            TIMESPEC='date' for date only (the default),
                            'hours', 'minutes', 'seconds', or 'ns' for date
                            and time to the indicated precision.
  -r, --reference=文件		显示文件指定文件的最后修改时间
  -R, --rfc-2822		以RFC 2822格式输出日期和时间
				例如：2006年8月7日，星期一 12:34:56 -0600
      --rfc-3339=TIMESPEC   output date and time in RFC 3339 format.
                            TIMESPEC='date', 'seconds', or 'ns' for
                            date and time to the indicated precision.
                            Date and time components are separated by
                            a single space: 2006-08-07 12:34:56-06:00
  -s, --set=STRING          set time described by STRING
  -u, --utc, --universal    print or set Coordinated Universal Time (UTC)
      --help		显示此帮助信息并退出
      --version		显示版本信息并退出

给定的格式FORMAT 控制着输出，解释序列如下：

  %%	一个文字的 %
  %a	当前locale 的星期名缩写(例如： 日，代表星期日)
  %A	当前locale 的星期名全称 (如：星期日)
  %b	当前locale 的月名缩写 (如：一，代表一月)
  %B	当前locale 的月名全称 (如：一月)
  %c	当前locale 的日期和时间 (如：2005年3月3日 星期四 23:05:25)
  %C	世纪；比如 %Y，通常为省略当前年份的后两位数字(例如：20)
  %d	按月计的日期(例如：01)
  %D	按月计的日期；等于%m/%d/%y
  %e	按月计的日期，添加空格，等于%_d
  %F	完整日期格式，等价于 %Y-%m-%d
  %g	ISO-8601 格式年份的最后两位 (参见%G)
  %G	ISO-8601 格式年份 (参见%V)，一般只和 %V 结合使用
  %h	等于%b
  %H	小时(00-23)
  %I	小时(00-12)
  %j	按年计的日期(001-366)
  %k   hour, space padded ( 0..23); same as %_H
  %l   hour, space padded ( 1..12); same as %_I
  %m   month (01..12)
  %M   minute (00..59)
  %n	换行
  %N	纳秒(000000000-999999999)
  %p	当前locale 下的"上午"或者"下午"，未知时输出为空
  %P	与%p 类似，但是输出小写字母
  %r	当前locale 下的 12 小时时钟时间 (如：11:11:04 下午)
  %R	24 小时时间的时和分，等价于 %H:%M
  %s	自UTC 时间 1970-01-01 00:00:00 以来所经过的秒数
  %S	秒(00-60)
  %t	输出制表符 Tab
  %T	时间，等于%H:%M:%S
  %u	星期，1 代表星期一
  %U	一年中的第几周，以周日为每星期第一天(00-53)
  %V	ISO-8601 格式规范下的一年中第几周，以周一为每星期第一天(01-53)
  %w	一星期中的第几日(0-6)，0 代表周一
  %W	一年中的第几周，以周一为每星期第一天(00-53)
  %x	当前locale 下的日期描述 (如：12/31/99)
  %X	当前locale 下的时间描述 (如：23:13:48)
  %y	年份最后两位数位 (00-99)
  %Y	年份
  %z +hhmm		数字时区(例如，-0400)
  %:z +hh:mm		数字时区(例如，-04:00)
  %::z +hh:mm:ss	数字时区(例如，-04:00:00)
  %:::z			数字时区带有必要的精度 (例如，-04，+05:30)
  %Z			按字母表排序的时区缩写 (例如，EDT)

默认情况下，日期的数字区域以0 填充。
The following optional flags may follow '%':

  -  (hyphen) do not pad the field
  _  (underscore) pad with spaces
  0  (zero) pad with zeros
  ^  use upper case if possible
  #  use opposite case if possible

在任何标记之后还允许一个可选的域宽度指定，它是一个十进制数字。
作为一个可选的修饰声明，它可以是E，在可能的情况下使用本地环境关联的
表示方式；或者是O，在可能的情况下使用本地环境关联的数字符号。

Examples:
Convert seconds since the epoch (1970-01-01 UTC) to a date
  $ date --date='@2147483647'

Show the time on the west coast of the US (use tzselect(1) to find TZ)
  $ TZ='America/Los_Angeles' date

Show the local time for 9AM next Friday on the west coast of the US
  $ date --date='TZ="America/Los_Angeles" 09:00 next Fri'

GNU coreutils online help: <http://www.gnu.org/software/coreutils/>
请向<http://translationproject.org/team/zh_CN.html> 报告date 的翻译错误
要获取完整文档，请运行：info coreutils 'date invocation'
[root@localhost ~]# passwd -d test
清除用户的密码 test。
passwd: 操作成功
```

date 命令

```
date --help    ##查看date命令相关参数
```

例如 （不要忘记date后面有一个空格 然后+开始以指定的格式输出）

```
[root@localhost ~]# date +%F          ##年-月-日  格式输出
2019-05-25
[root@localhost ~]# date +%Y%m%d ##年月日格式输出
20190525
[root@localhost ~]# date +%j          ##一年的第几天
145
[root@localhost ~]# date +%w        ##周几？
6
[root@localhost ~]# date +%W       ##一年的第几周？
20
[root@localhost ~]# date -s 20190526
2019年 05月 26日 星期日 00:00:00 CST    ##设置日期

```
file 命令
用途： 查看文件类型


语法file(选项)(参数)选项-b：列出辨识结果时，不显示文件名称；
-c：详细显示指令执行过程，便于排错或分析程序执行的情形；
-f<名称文件>：指定名称文件，其内容有一个或多个文件名称时，让file依序辨识这些文件，格式为每列一个文件名称；
-L：直接显示符号连接所指向的文件类别；
-m<魔法数字文件>：指定魔法数字文件；
-v：显示版本信息；
-z：尝试去解读压缩文件的内容。参数文件：要确定类型的文件列表，多个文件之间使用空格分开，可以使用shell通配符匹配多个文件。实例显示文件类型

```
[root@localhost ~]# file install.log
install.log: UTF-8 Unicode text

[root@localhost ~]# file -b install.log      <== 不显示文件名称
UTF-8 Unicode text

[root@localhost ~]# file -i install.log      <== 显示MIME类别。
install.log: text/plain; charset=utf-8

[root@localhost ~]# file -b -i install.log
text/plain; charset=utf-8
显示符号链接的文件类型[root@localhost ~]# ls -l /var/mail
lrwxrwxrwx 1 root root 10 08-13 00:11 /var/mail -> spool/mail

[root@localhost ~]# file /var/mail
/var/mail: symbolic link to `spool/mail'

[root@localhost ~]# file -L /var/mail
/var/mail: directory
```

