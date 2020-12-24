# ELK简单部署

**说明：由于公司对此设施的需求度不是很高，因此项目中没有涉及Logstash对日志信息的管道通信处理，因此此文档仅介绍E和K即Elasticsearch和Kibana**

## 简单介绍

ELK是elastic公司推出的一套分布式的日志分析系统，是elastic stack的核心组件，由三个子系统组成，即：elasticsearch（E）、logstash（L）以及kibana（K）。在此分别简单介绍一下它们的功能点：

* **elasticsearch**是一款分布式的搜索引擎，是elastic公司的核心组件，在elk系统中它仅仅是做日志处理和搜索的简单逻辑，其实elasticsearch功能强大，常常作为应用的搜索引擎，比如github的高亮代码检索以及购物网站的商品搜索等都离不开elasticsearch的强大集群支持。其特点就是可以给用户带来几乎实时的大数据搜索展示。
* **logstash**是依托于elasticsearch开发出的分布式日志分析系统，它的前身其实是elasticsearch的一个社区发起的第三方插件，之后被elastic公司收购，成为elastic stack中的一员，其主要功能是对收集到的日志进行管道处理，即：减缓大规模数据下elasticsearch的压力以及对日志的格式等做进一步的处理
* **kibana**是一个数据可视化分析平台，主要是对收集起来的数据比如日志信息进行可视化展示以及图形化构建，其本身是logstash的一个插件工具，之后同样被elastic公司收购，成为elastic stack成员

因此，从上述的介绍可以知道，ELK的主要功能都被elasticsearch占据，其余两个组件是依托于elasticsearch而存活的。

## 安装流程

elastic公司的商业化做的很好，因此官网有完善的资源包和安装流程，在此，我仅对Linux安装的一些步骤进行简单梳理，让你可以不用看官方文档也可以在本机部署一个简单的ELK系统。

**注1：所有安装步骤均在Ubuntu(v20.04LTS)下执行，其他发行版需要去官方找其他安装命令**

**注2：elasticsearch的几乎每次大版本升级都会对之前版本的config文件进行breaking change，因此最好将ELK版本统一并且使用最新稳定版安装，本例使用7.10.1**

**注3：由于GFW原因，如果没有外网节点的网络可能下载会很慢，所以这里推荐国内可以在[华为云](https://mirrors.huaweicloud.com/)直接下载相应的镜像，或者直接使用迅雷开会员加速。我在第一次做的时候两种方法都用过了，因为华为云只对elasticsearch镜像实时更新，其他的镜像比如kibana等都没有最新版本，因此其余的镜像我是通过迅雷开会员加速下载的国外资源。**

### 安装elasticsearch

首先需要安装elasticsearch，可以在[官方站点](https://www.elastic.co/cn/downloads/elasticsearch)找到对应版本的资源的下载包，Ubuntu版本可以使用二进制包安装或者apt-get安装，本例[直接下载二进制压缩包](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.1-linux-x86_64.tar.gz)或[华为云镜像](https://mirrors.huaweicloud.com/elasticsearch/7.10.1/elasticsearch-7.10.1-linux-x86_64.tar.gz)安装，可以直接执行以下命令下载：

``` shell
# 源资源
sudo curl -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.1-linux-x86_64.tar.gz

# 华为云镜像
sudo curl -O https://mirrors.huaweicloud.com/elasticsearch/7.10.1/elasticsearch-7.10.1-linux-x86_64.tar.gz
```

下载后是一个压缩包，因此还需要解压缩，对默认文件路径有强迫症的同学应该在相应的路径下解压缩，命令如下：

``` shell
sudo tar -xzvf elasticsearch-7.10.1-linux-x86_64.tar.gz
```

解压结束后进入目录可以看到如下的几个文件夹：

![elasticsearch_files]()