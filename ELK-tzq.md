任务名称： ELK日志分析平台 



任务提交者：tangzhiqiang

## 介绍

ELK是日志分析平台，是一整套解决方案,是三个软件产品的首字母缩写，ELK分别代表：

Elasticsearch:负责日志检索和储存

Logstash:负责日志的收集和分析、处理

Kibana:负责日志的可视化

## 环境要求

* 安装包：elasticsearch  Kibana
* Elasticsearch插件：kopf、head、bigdesk
* 依赖组件：java-1.8.0-openjdk.x86_64
* 服务器配置：1cpu，1G内存，10G硬盘（两台）
* 其他：购买弹性公网IP

## 安装说明

准备好elk的安装包，购买两台云主机自定义yum源（查看），分别安装elasticsearch 和Kibana，购买公网IP，使用shell连接进行软件的安装

分别进行安装说明。

### CentOS安装elasticsearch

```shell
#本地安装elasticsearch和依赖软件
yum install -y java-1.8.0-openjdk elasticsearch
#本地安装3个插件
/usr/share/elasticsearch/bin/plugin install file:///root/elk/bigdesk-master.zip
/usr/share/elasticsearch/bin/plugin install file:///root/elk/elasticsearch-kopf-master.zip 
/usr/share/elasticsearch/bin/plugin install file:///root/elk/elasticsearch-head-master.zip 


```

## 配置说明

安装完成后，进行主要的配置和其他安全配置，则可以进行访问

### 基本配置

```shell
systemctl enable --now elasticsearch
firewall-cmd --set-default-zone=trusted  #设置防火墙
setenforce 0  #临时设置selinux
sed -i '/SELINUX/senforcing/permissve' /etc/selinux/config  #使用正则表达式永久设置selinux
vim /etc/elasticsearch/elasticsearch.yml  #修改配置文件
54：  network.host: 0.0.0.0   #修改54行（监听所有地址）
systemctl restartelasticsearch
curl http://本机地址或主机名:9200/
```



### 其他配置

* 如果项目需要做集群，则需要所有主机安装软件 ，地址和主机名解析，修改主配置文件

```shell
vim /etc/hosts
vim /etc/elasticsearch/elasticsearch.yml
17：  cluster.name: 集群名
23：  node.name: 本主机名
54：  network.host: 0.0.0.0
68：  discovery.zen.ping.unicast.hosts: ["集群中的主机名"，"...",...]

```

* 访问插件
```shell
http://弹性公网IP:9200/_plugin/插件名称  [bigdesk|head|kopf]
http://公网IP:9200/\_plugin/kopf
http://公网IP:9200/\_plugin/head
http://公网IP:9200/\_plugin/bigdesk
```
## 使用说明

### 基本操作

###### 查询_cat方法

```shell
# 查询支持的关键字
curl -XGET http://elk:9200/_cat/
# 查具体的信息
curl -XGET http://elk:9200/_cat/master
# 显示详细信息 ?v
curl -XGET http://elk:9200/_cat/master?v
# 显示帮助信息 ?help
curl -XGET http://elk:9200/_cat/master?help
```

###### 创建索引

指定索引的名称，指定分片数量，指定副本数量

创建索引使用 PUT 方法，创建完成以后通过 head 插件验证

```shell
curl -XPUT http://elk:9200/tedu -d \
'{
    "settings":{
       "index":{
          "number_of_shards": 5, 
          "number_of_replicas": 1
       }
    }
}'
```

###### 增加数据

```shell
 curl -XPUT http://elk:9200/tedu/teacher/1 -d \
'{
  "职业": "诗人",
  "名字": "李白",
  "称号": "诗仙",
  "年代": "唐"
}' 
```

###### 查询数据

```shell
curl -XGET http://elk:9200/tedu/teacher/1?pretty
```

###### 修改数据

```shell
 curl -XPOST http://elk:9200/tedu/teacher/1/_update -d '{ 
    "doc": {
        "年代": "公元701"
    }
}'
```

###### 删除数据

```shell
# 删除一条
[root@es-0001 ~]# curl -XDELETE http://elk:9200/tedu/teacher/1
# 删除索引
[root@es-0001 ~]# curl -XDELETE http://elk:9200/tedu
# 删除所有
[root@es-0001 ~]# curl -XDELETE http://elk:9200/*
```



### 路径

* 程序路径：/usr/share/elasticsearch/bin/elasticsearch

* 日志路径：/var/log/elasticsearch

* 配置文件路径：  vim /etc/elasticsearch/elasticsearch.yml

* 文件执行路径：/usr/share/elasticsearch/plugins

  ### kibana安装

  ```shell
  yum install -y kibana
  vim /opt/kibana/config/kibana.yml
  02  server.port: 5601
  05  server.host: "0.0.0.0"
  15  elasticsearch.url: "http://elk:9200"
  23  kibana.index: ".kibana"
  26  kibana.defaultAppId: "discover"
  systemctl enable --now kibana
  ```

  绑定弹性公网IP，通过 WEB 浏览器验证

  http://弹性公网IP:5601/status

  

#### 数据库密码

如果有数据库

* 数据库安装方式：包管理工具自带 or 自行安装
* 账号密码：

#### 后台账号

如果有后台账号

* 登录地址
* 账号密码
* 密码修改方案：最好是有命令行修改密码的方案


### 版本号

通过如下的命令获取主要组件的版本号: 

```
curl -XGET localhost:9200
{
  "name" : "Stiletto",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "2.3.4",
    "build_hash" : "e455fd0c13dceca8dbbdbb1665d068ae55dabe3f",
    "build_timestamp" : "2016-06-30T11:24:31Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.0"
  },
  "tagline" : "You Know, for Search"
}


```

### 端口号

本项目需要开启的端口号：

| item          | port |
| ------------- | ---- |
| Elasticsearch | 9200 |
| kibana        | 5601 |

## 常见问题

#### 有没有管理控制台？



#### 有没有CLI工具？



## 日志

* 2020-9-9 完成CentOS安装ELK
