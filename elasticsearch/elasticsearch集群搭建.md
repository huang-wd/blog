# ElasticSearch环境搭建 7.0的配置
## 节点角色
[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/modules-node.html) 

* master节点：master节点主要负责元数据的处理，比如索引的新建、删除、分片分配等，每当元数据有更新时，master节点负责同步到其他节点上。

* data节点：data节点上保存了数据分片。它负责数据相关操作，比如分片的增删查改以及搜索和整合操作。

* ingest节点：预处理节点，可以在实际的 indexing（索引）发生之前使用 ingest node 来预处理 documents（文档）。

* coordinating节点：协调节点：

Requests like search requests or bulk-indexing requests may involve data held on different data nodes. A search request, for example, is executed in two phases which are coordinated by the node which receives the client request — the coordinating node.

In the scatter phase, the coordinating node forwards the request to the data nodes which hold the data. Each data node executes the request locally and returns its results to the coordinating node. In the gather phase, the coordinating node reduces each data node’s results into a single global resultset.

Every node is implicitly a coordinating node. This means that a node that has all three node.master, node.data and node.ingest set to false will only act as a coordinating node, which cannot be disabled. As a result, such a node needs to have enough memory and CPU in order to deal with the gather phase.

## 机器分配
| 机器ip | 配置 | 角色配置 |
| --- | --- | --- |
| 172.20.240.239 | 2C 4G | node.master: true <br> node.data: false <br> node.ingest: false <br> cluster.remote.connect: false |
| 172.20.95.250 | 2C 4G | node.master: false <br> node.data: true <br> node.ingest: false <br> cluster.remote.connect: false |
| 172.20.95.251 | 2C 4G | node.master: false <br> node.data: true <br> node.ingest: false <br> cluster.remote.connect: false |

## 集群配置
配置参考：
[important-settings](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/important-settings.html)

[discovery-settings](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/discovery-settings.html)

* 172.20.240.239 2C 4G

```
cluster.name: skywalking-es-test-environment
node.name: master-239
network.host: 172.20.240.239
node.master: true 
node.data: false 
node.ingest: false 
cluster.remote.connect: false
discovery.seed_hosts: 
   - 172.20.240.239:9300 
cluster.initial_master_nodes: 
   - 172.20.240.239:9300
path:
  logs: /mnt/logs/elasticsearch
  data: /mnt/data/elasticsearch
```
* 172.20.95.250 2C 4G

```
cluster.name: skywalking-es-test-environment
node.name: data-250
network.host: 172.20.95.250
node.master: false 
node.data: true 
node.ingest: false 
cluster.remote.connect: false
discovery.seed_hosts: 
   - 172.20.240.239:9300 
cluster.initial_master_nodes: 
   - 172.20.240.239:9300
path:
  logs: /mnt/logs/elasticsearch
  data: /mnt/data/elasticsearch
```
* 172.20.95.251 2C 4G

```
cluster.name: skywalking-es-test-environment
node.name: data-251
network.host: 172.20.95.251
node.master: false 
node.data: true 
node.ingest: false 
cluster.remote.connect: false
discovery.seed_hosts: 
   - 172.20.240.239:9300 
cluster.initial_master_nodes: 
   - 172.20.240.239:9300
path:
  logs: /mnt/logs/elasticsearch
  data: /mnt/data/elasticsearch
```
# 6.7的配置：
```
cluster.name: skywalking-es-test-environment
node.name: data-251
network.host: 172.20.95.251
node.master: false 
node.data: true 
node.ingest: false 
cluster.remote.connect: false
discovery.zen.ping.unicast.hosts: 
   - 172.20.240.239:9300
   - 172.20.95.250:9300
   - 172.20.95.251:9300
discovery.zen.minimum_master_nodes: 1
path:
  logs: /mnt/logs/elasticsearch
  data: /mnt/data/elasticsearch
```


# SkyWalking环境搭建
[Backend and UI Setup 官方指南](https://github.com/apache/incubator-skywalking/blob/master/docs/en/setup/backend/backend-ui-setup.md)
![官方SkyWalking后端部署架构图](media/%E5%AE%98%E6%96%B9SkyWalking%E5%90%8E%E7%AB%AF%E9%83%A8%E7%BD%B2%E6%9E%B6%E6%9E%84%E5%9B%BE-1.png)

## 机器分配
| 服务 | 地址 | 备注 |
| --- | --- | --- |
| UI | 172.20.31.117 | Port: 8080 <br> 用户名: admin <br> 密码: admin123 |
| Backend | 172.20.31.117 | REST Port: 12800 <br> gRPC Port: 11800 |

SkyWalking目录结构

```
drwxrwxr-x 7 work work  4096 Jan 21 12:10 agent
drwxrwxr-x 2 work work  4096 Apr 18 11:41 bin
drwxrwxr-x 2 work work  4096 Apr 18 11:41 config
-rw-rw-r-- 1 work work   557 Jan 21 11:53 DISCLAIMER
-rw-rw-r-- 1 work work 34275 Jan 21 11:53 LICENSE
drwxrwxr-x 3 work work  4096 Apr 18 11:41 licenses
-rw-rw-r-- 1 work work 29638 Jan 21 11:53 NOTICE
drwxrwxr-x 2 work work 12288 Jan 21 12:00 oap-libs
-rw-rw-r-- 1 work work  2018 Jan 21 11:53 README.txt
drwxrwxr-x 2 work work  4096 Apr 18 11:41 webapp
```
SkyWalking backend distribution package includes following parts

1. bin/cmd scripts, in /bin folder. Include startup linux shell and Windows cmd scripts for Backend server and UI startup.

2. Backend config, in /config folder. Include setting files of backend, which are application.yml, log4j.xml and alarm-settings.yml. Most open settings are in these files.

3. Libraries of backend, in /oap-libs folder. All jar files of backend are in it.

4. Webapp env, in webapp folder. UI frontend jar file is in here and its webapp.yml setting file.

## 配置
[客户端设置参看：](https://github.com/apache/skywalking/blob/v6.0.0-GA/docs/en/setup/service-agent/java-agent/README.md)

[请求头格式](https://github.com/apache/skywalking/blob/v6.0.0-GA/docs/en/protocols/Skywalking-Cross-Process-Propagation-Headers-Protocol-v1.md)

[客户端覆盖配置变量](https://github.com/apache/skywalking/blob/v6.0.0-GA/docs/en/setup/service-agent/java-agent/Setting-override.md)

[指定配置文件路径](https://github.com/apache/skywalking/blob/v6.0.0-GA/docs/en/setup/service-agent/java-agent/Specified-agent-config.md)

[用空间变量来区隔离系统](https://github.com/apache/skywalking/blob/v6.0.0-GA/docs/en/setup/service-agent/java-agent/Namespace.md)

[从代码中获取traceId](https://github.com/apache/skywalking/blob/v6.0.0-GA/docs/en/setup/service-agent/java-agent/Application-toolkit-trace.md)

