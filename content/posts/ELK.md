---
title: "ELK+filebeat+redis安装部署文档"
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
description: "介绍用yum文件安装ELK"
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
ELK+filebeat+redis安装部署文档
1.首先安装jdk 1.8
>yum -y install java-1.8.0-openjdk.x86_64

查看java版本
>java -version

2.配置ELK yum源

1）
`rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch`

2）`vim /etc/yum.repos.d/elasticsearch.repo` #添加elk安装yum源
```
[elasticsearch-5.x]
name=Elasticsearch repository for 5.x packages
baseurl=https://artifacts.elastic.co/packages/5.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

3.安装elasticsearch

1)`yum -y install elasticsearch`
`Chkconfig elasticsearch on`

2)配置elasticsearch
```
cluster.name: htd 配置集群
node.name: htd-es-1 配置集群节点
path.data: /var/lib/elasticsearch 配置数据存储目录
path.logs:/var/log/elasticsearch 配置日志存储目录
network.host: 0.0.0.0 配置绑定IP
http.port: 9200 配置端口
discovery.zen.ping.unicast.hosts: ["10.8.28.146"] 配置集群寻址
```

3)启动
`Service elasticsearch start`
测试访问{+}http://10.8.28.146:9200/
4.安装logstash服务
1)yum install logstash
2)配置logstash
`vim logstash.yml`
>path.data: /var/lib/logstash 配置数据目录
>path.config: /etc/logstash/conf.d logstash配置目录
>http.host: "0.0.0.0" 配置服务IP
>http.port: 9600-9700 配置端口
>path.logs: /var/log/logstash logstash日志目录

3)生成logstash启动脚本
`/usr/share/logstash/bin/system-install /etc/logstash/startup.options sysv`
4)启动logstash
`Service logstash start`
`Chkconfig logstash on`
5)配置logstash服务文件：
配置文件目录：/etc/logstash/conf.d/redis.conf
>input {
>redis {
>host => "10.8.28.146"
>port => 6379
>data_type => "list"
>key => "filebeat"
>}
>}
>filter {
>grok {
>match => { "message" => "%{COMBINEDAPACHELOG} %{QS:x_forwarded_for}"}
>}
>date {
>match => [ "timestamp" , "dd/MMM/YYYY:HH:mm:ss Z" ]
>}
>geoip {
>source => "clientip"
>}
>}
>output {
>elasticsearch {
>hosts => "10.8.28.146:9200"
>index => "test-%{+YYYY.MM.dd}"
>}
>}

5.安装kibana
1)yum install kibana
2)配置kibana
Vim /etc/kibana/kibana.yml
>server.port: 5601 配置端口
>server.host: "0.0.0.0" 配置服务地址
>server.name: "HTD-Formal-Kibana" 配置kibana服务名
>elasticsearch.url: "http://10.8.28.146:9200" 配置连接elasticsearch参数
>3)启动kibana
>Chkconfig kibana on
>Service kibana start
>测试：http://10.8.28.146:5601/Agent端，Filebeat安装安装filebeat
>yum localinstall filebeat-6.2.1-x86_64.rpm -y
>2.编辑配置文件
>vim /etc/filebeat/filebeat.ymlinput_type: log
>paths:/var/log/nginx/access.log #路径为需要监控的日志文件路径
>tail_files: true
>output.redis:
>enabled: true
>hosts: ["10.8.28.146:6379"]
>datatype: list
>key: "filebeat"
>logging.level: debug

3.启动服务
>service filebeat start
>chkconfig filebeat on

访问web页面查看日志
访问地：elk.XX.com
索引为test-*