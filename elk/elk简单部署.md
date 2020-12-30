# ELK简单部署

**说明：由于公司对此设施的需求度不是很高，因此项目中没有涉及Logstash对日志信息的管道通信处理，因此此文档仅介绍E和K即Elasticsearch和Kibana**

## 简单介绍

ELK是elastic公司推出的一套分布式的日志分析系统，是elastic stack的核心组件，由三个子系统组成，即：elasticsearch（E）、logstash（L）以及kibana（K）。在此分别简单介绍一下它们的功能点：

* **elasticsearch**是一款分布式的搜索引擎，是elastic公司的核心组件，在elk系统中它仅仅是做日志处理和搜索的简单逻辑，其实elasticsearch功能强大，常常作为应用的搜索引擎，比如github的高亮代码检索以及购物网站的商品搜索等都离不开elasticsearch的强大集群支持。其特点就是可以给用户带来几乎实时的大数据搜索展示。
* **logstash**是依托于elasticsearch开发出的分布式日志分析系统，它的前身其实是elasticsearch的一个社区发起的第三方插件，之后被elastic公司收购，成为elastic stack中的一员，其主要功能是对收集到的日志进行管道处理，即：减缓大规模数据下elasticsearch的压力以及对日志的格式等做进一步的处理
* **kibana**是一个数据可视化分析平台，主要是对收集起来的数据比如日志信息进行可视化展示以及图形化构建，其本身是logstash的一个插件工具，之后同样被elastic公司收购，成为elastic stack成员

因此，从上述的介绍可以知道，ELK的主要功能都被elasticsearch占据，其余两个组件是依托于elasticsearch而存活的，我们可以通过下图来看一下elastic stack的相关内容：

![elastic_stack](https://raw.githubusercontent.com/lzl82891314/Notes/main/elk/resource/elastic_stack.png)

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

解压结束后进入目录可以看到如下的几个文件夹，在此做一下简单的说明：

![elasticsearch_files](https://raw.githubusercontent.com/lzl82891314/Notes/main/elk/resource/elasticsearch_files.png)

* bin: 存放elasticsearch的脚本文件
* config：是集群的配置文件，核心的配置在`elasticsearch.yml`中，这个文件需要做大量的配置，会在后面的章节说明
* jdk：存放Java的运行环境，因为elasticsearch是用java开发的，因此需要java运行时，并且在6之前的版本都需要单独安装运行时，在7之后的版本，运行时默认在安装包内一起存在
* data：存放数据文件，可以通过`elasticsearch.yml`文件中的`path.data`指定
* lib：是elasticsearch运行时需要的Java依赖包
* logs：存放日志文件，可以通过`elasticsearch.yml`文件中的`path.log`指定
* modules：包含所有ES模块
* plugins：存放所有已安装插件，elasticsearch有完善的插件机制，并且很多功能都需要依赖插件

之后就可以通过以下命令执行启动elasticsearch了：

``` shell
# 需要在安装目录下
bin/elasticsearch
```

如果存在权限问题，最好将目录的所有者配置给当前的用户，比如我可以执行以下命令将elasticsearch的所有权限配置给自己：

``` shell
sudo chown -R jeffery:jeffery /mnt/work/elk/elasticsearch
```

之后就可以看到elasticsearch已经开始运行了：

![elasticsearch_starting](https://raw.githubusercontent.com/lzl82891314/Notes/main/elk/resource/elasticsearch_starting.png)

如果想了解运行状态，可以通过9200端口查看，比如：

``` shell
curl localhost:9200
```

可以看到返回如下信息，说明elasticsearch已经成功运行：

![elasticsearch_running](https://raw.githubusercontent.com/lzl82891314/Notes/main/elk/resource/elasticsearch_running.png)

此外，如果想为elasticsearch安装插件，可以执行以下命令进行安装：

``` shell
# 查看当前已经安装的插件
bin/elasticsearch-plugin list

# 安装分词插件
bin/elasticsearch-plugin install analysis-icu
```

**注意：安装完插件后，elasticsearch需要重启，目前尚不支持热加载**

### 安装kibana

和上述elasticsearch的安装方式类似，我们同样可以在我[官网](https://www.elastic.co/cn/downloads/kibana)寻找适合自己设备的[kibana镜像](https://artifacts.elastic.co/downloads/kibana/kibana-7.10.1-linux-x86_64.tar.gz)或者[华为云镜像](https://mirrors.huaweicloud.com/kibana/7.10.1/kibana-7.10.1-linux-x86_64.tar.gz)进行下载，命令如下：

``` shell
sudo curl -O https://mirrors.huaweicloud.com/kibana/7.10.1/kibana-7.10.1-linux-x86_64.tar.gz
```

完成后同样解压：

``` shell
sudo tar -xzvf kibana-7.10.1-linux-x86_64.tar.gz
```

完成后进入kibana的目录，同样可以通过以下命令执行kibana，但是需要说明一点，因为kibana是依赖于elasticsearch执行的，所以需要先开启elasticsearch之后再运行kibana：

``` shell
# 需要在安装目录下
bin/kibana
```

在默认情况下，kibana就会寻找本地9200端口下的elasticsearch，因此在默认情况下可以不用配置任何信息即可运行elasticsearch和kibana，运行成功后，可以访问本机的5601端口访问kibana，如下图所示：

![kibana_home](https://raw.githubusercontent.com/lzl82891314/Notes/main/elk/resource/kibana_home.png)

可以看到kibana已经成功运行了，并且提示你可以为kibana添加一些默认默认的样例数据，如果添加了，这些数据就会被写入elasticsearch中，并且在kibana的`Discover`菜单下就可以对其进行操作。

至此，一个简单的系统就已经被搭建成功了，由于我们的项目中没有使用logstash，而是直接将数据写入elasticsearch中的，因此logstash的安装就不介绍了。如果你想安装logstash，其实和上述两个方式完全大同小异，只要找到对应的镜像就可以下载安装使用了。

### 以Docker的形式安装（项目使用）

**注：这种方式在写这篇文档的时候应该已经确定是我们的项目使用ELK系统的方式了，因此如果只是需要复制一个ELK监控系统，那完全通过以下的方式即可**

通过上述的方式我们可以成功安装E和K，但是由于我们的项目简单，不需要对ELK搭建集群处理数据管道，直接将一些简单的监控日志以及数据写入elasticsearch中在kibana展示，因此我们完全可以将elasticsearch和kibana通过docker的形式安装，这样简单易用还很容易移植。

因此，我们可以通过一个`docker-compose`来创建我们所需要的所有监控信息的基础设施，这个docker-compose中包括：

* 一个elasticsearch服务，作为搜索引擎和数据库
* kibana服务，数据展示
* cerebro服务，作为elasticsearch管理和索引信息可视化管理，并且可以监控elasticsearch集群
* filebeat采集器，收集文本日志，直接将数据写入elasticsearch中
* metricbeat采集器，收集监控数据日志，直接将数据写入elasticsearch中

具体的docker-compose命令如下，我会对其中关键的配置节点做一下备注说明：

``` yaml
version: "2.1"
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.10.0
    container_name: elasticsearch
    environment:
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m" # 是JavaGC的配置，这事默认执行栈和存储栈都是512M，可以修改其大小，但是我们的数据量来说足够了
    mem_limit: "2621440000" # elasticsearch所需的内存限制，是字节值，需要大于6M，我这里设置了2.5G
    ulimits: # 之下是对elasticsearch本身做的限制，默认可以不写
      memlock:
        soft: -1
        hard: -1
    volumes:
      - /mnt/work/elk/elasticsearch/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml # 将配置信息挂载到容器外部，这样便于管理，这里一定要注意挂载路径
      - elasticsearch_data:/mnt/work/elk/elasticsearch/data # 运行所需挂载
    ports:
      - 9200:9200 # 开放9200端口访问
      - 9300:9300 # 开放9300端口用于集群内通知，因为我们的例子中不需要集群，因此这个9300端口可以不暴露
    networks:
      - elk_network
  cerebro:
    image: lmenezes/cerebro:0.9.2
    container_name: cerebro
    ports:
      - 9000:9000
    networks:
      - elk_network
    command:
      - -Dhosts.0.host=http://elasticsearch:9200 # 默认将9200端口作为elasticsearch的监控端口
    depends_on: ['elasticsearch']
  kibana:
    image: docker.elastic.co/kibana/kibana:7.10.0
    container_name: kibana
    environment:
      - I18N_LOCALE=zh-CN # 本地化配置，修改为中文
    volumes:
      - /mnt/work/elk/kibana/kibana.yml:/usr/share/kibana/config/kibana.yml # 和上述elasticsearch相同，将配置文件挂载到程序外方便管理
    ports:
      - 5601:5601
    networks:
      - elk_network
    depends_on: ['elasticsearch']
  filebeat:
    image: elastic/filebeat:7.10.0
    container_name: filebeat
    volumes:
      - /mnt/work/elk/filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml # 同上配置文件挂载
      - filebeat_data:/mnt/work/elk/filebeat/data
    networks
      - elk_network
    restart: on-failure
    depends_on: ['elasticsearch', 'kibana']
  metricbeat:
    image: elastic/metricbeat:7.10.0
    container_name: metricbeat
    volumes:
      - /mnt/work/elk/metricbeat/metricbeat.yml:/usr/share/metricbeat/metricbeat.yml # 同上配置文件挂载
      - /var/run/docker.sock:/var/run/docker.sock
    restart: on-failure
    depends_on: ['elasticsearch', 'kibana']

volumes:
  elasticsearch_data:
    driver: local
  filebeat_data:
    driver: local

networks:
  elk_network:
    driver: bridge
```

通过上述的docker-compose文件，我们就可以通过`docker-compose up`直接启动我们的站点了，但是如上述配置节点所示，我们需要对一些配置信息做处理，因此我们需要以下4个不同的配置文件：

``` shell
# 1. elasticsearch.yml
cluster.name: forguncy-cloud # 集群的名称，如果需要扩展集群，则需要通过这个名称加入
node.name: master # 节点名称，因为我们的系统是单节点的，因此就承担master的职责
network.host: 0.0.0.0 # 网络开放的IP，如果配置0.0.0.0就是所有IP都可以访问，日后管理可以改为特定IP增加安全性
discovery.seed_hosts: 192.168.238.118 # 集群主机IP集合，我们只有一个单节点，因此这里直接填部署宿主机的IP即可
cluster.initial_master_nodes: 192.168.238.118 # 集群的默认master节点IP，由于单机同样宿主机IP即可
bootstrap.memory_lock: true # 这个配置官方推荐加入，我一直不知道这个是干啥的，所以就加上吧

# 2. kibana.yml
server.name: kibana
server.host: "0"
elasticsearch.hosts: [ "http://elasticsearch:9200" ] # 主要配置elasticsearch的端口

# 3. filebeat.yml
filebeat.config:
  modules:
    path: ${path.config}/modules.d/*.yml
    reload.enabled: false

filebeat.inputs: # 文本日志的配置路径
- type: log
  enabled: true
  paths:
    - /var/log/*.log
    - /var/log/*/*.log # 递归搜索文件

output.elasticsearch:
  hosts: ["elasticsearch:9200"]

# 4. metricbeat.yml
metricbeat.config:
  modules:
    path: ${path.config}/modules.d/*.yml
    reload.enabled: false

metricbeat.modules: # 配置监控开关，比如此配置就是开启了system监控，关闭了docker监控，默认不加入就是关闭
- module: system 
  metricsets:
    - "cpu"
    - "memory"
    - "network"
  period: 10s
  enabled: true
- module: docker
  metricsets:
    - "container"
    - "cpu"
    - "diskio"
    - "healthcheck"
    - "info"
    #- "image"
    #- "memory"
    #- "network"
  hosts: ["unix:///var/run/docker.sock"]
  period: 10s
  enabled: false

output.elasticsearch:
  hosts: ['elasticsearch:9200']
```

上述四个配置文件，最主要的是elasticsearch的，这是整个系统能否启动的关键，其余三个默认配置就是这样，为了后期维护的便利性将其挂载出来，如果想省事可以不创建其余三个，当然对应的docker-compose中的挂载配置需要删除。

有了上述的配置文件和docker-compose文件，我们就可以直接启动一个集群了（文件的相对路径一定要写准确，需要和配置挂载项中的路径完全一致），命令如下：

``` shell
# 需要在docker-compose.yaml文件的路径下执行
docker-compose up

# 卸载docker-compose
docker-compose down
```

一切配置运行成功之后，我们就可以看到一个完整的监控项目了，下图是当前系统的elasticsearch信息，通过9000端口访问cerebro即可：

![cerebro](https://raw.githubusercontent.com/lzl82891314/Notes/main/elk/resource/cerebro.png)

可以看到，我们的集群中有一个elasticsearch节点，有8个索引，分布在9个分片上，已经写入了7428个文档，占用了13.53M。其中，写入的这些文件，就是通过上述的metricbeat采集到的监控数据，数据信息是所在宿主机的系统数据，我们可以在kibana中看到它们的完整数据：

![kibana_metricbeat_data](https://raw.githubusercontent.com/lzl82891314/Notes/main/elk/resource/kibana_metricbeat_data.png)

此外，如果首次运行elasticsearch是可能会报错：`max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]`，此时是系统中的最大虚拟内存设置过低，elasticsearch运行至少需要262144，因此可以执行以下命令解决：

``` shell
sysctl -w vm.max_map_count=262144
```

至此，整个监控系统的简单部署就结束了，之后的任务就是配置集群或者采集点了，这些工作其实都是操作对应的采集器的配置文件即可完成，由于写文档的时候还没有具体的服务实体，因此无法给出具体的配置，但是，只需要操作挂载出来的配置文件即可实现。