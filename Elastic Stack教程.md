# Elastic Stack教程

## Elasticsearch配置详解

#### 常用术语

文档 Document

		- 用户存储在es中的数据文档

类型：

​		字符串: text,keyword

​		数值型: long,integer,short,byte,double,float,half_scaled_float

​		布尔: boolean

​		日期: date

​		二进制: binary

​		范围类型: integer_range,float_range,long_range,double_range,date_range

每个文档有唯一的id标识

​	自行指定

​	es自动生成

元数据：

- _index  文档所在的索引名
- _type　文档所在的类型名
- _id　文档唯一id
- _uid  组合id，由_type和_id组成（６.x_type不在起作用，同_id一样）
- _source  文档的原始Json数据，可以从这里获取每个字段的内容
- _all  整合所有字段内容到该字段，默认禁用

索引 Index

​		- 由具有相同字段的文档列表组成

索引中存储具有相同结构的文档(Document)

​		每个索引都由自己的mapping定义，用于定义字段名和类型

一个集群可以由多个索引，比如：

​		nginx日志存储的时候可以按照日期每天生成一个索引来存储

​				nginx-log-2019-01-01

​				nginx-log-2019-01-02

​				nginx-log-2019-01-03

创建索引与写入数据

​	Elasticsearch集群对外提供RESTful  API



节点  Node

​		- 一个Elasticsearch的运行实例,是集群的构成单元

集群  Cluster

​		- 由一个或多个节点组成，对外提供服务







#### Elasticsearch配置说明

配置文件位于config目录中：

- ​		elasticearch.yml		es的相关配置
- ​		jvm.options 		jvm的相关参数
- ​		log4j2.propertises		日志相关配置

elasticsearch.yml关键配置说明：

- cluster.name	集群名称，以此作为是否同一集群的判断条件
- node.name      节点名称,以此作为集群中不同节点的区分条件
- network.host/http.port   网络地址和端口,用于http和transport服务使用
- path.data    数据存储地址
- path.log　日志存储地址

Development与Production模式说明

- 以transport的地址是否绑定在localhost为判断标准network.host
- Development模式下在启动时会以warnging的方式提示配置检查异常
- Production模式下在启动时会以error的方式提示配置检查异常并退出

参数修改的第二种方式

​	bin/elasticearch -Ehttp.port=19200

1. #### Elasticearch本地情动集群的方式

2. bin/elasticsearch

3. bin/elasticsearch -Ehttp.port=8200 -Epath.data=node2

4. bin/elasticsearch -Ehttp.port=7200 -Epath.data=node3

## Kibana配置说明

- 配置位于config文件夹中
- kibana.yml关键配置说明
  - server.host/server.port    访问kibana用的地址和端口
  - elasticsearch.url待访问elasticsearch的地址

## Kibana常用功能说明

- Discover		数据搜索查看
- Visualize		图表制作
- Dashboard		仪表盘制作
- Timelion      时序数据的高级可视化分析
- DevTools      开发工具
- Management       配置

## Elasticsearch常用术语

- Docunment    文档数据
- Index　索引
- Type     索引中的数据类型
- Field   字段，文档的属性
- Query DSL     查询语法

## Elasticsearch　CRUD

create    创建文档

```
POST /accounts/person/1
{
  "name":"Joh",
  "lastname":"Doe",
  "job_description":"Systems administrator and Linux specialit"
}
```

read　　读取文档

```
GET /accounts/person/
```

update    　更新文档

```
post /accounts/person/1/_update
{
  "doc":{
    "name":"tom"
  }
}
```

delete　　删除文档

```
DELETE /accounts/person/1
```

## Elasticsearch Query

- Query String

```
GET /accounts/person/_search?q=tim
```

- Query DSL

```
GET /accounts/person/_search
{
  "query": {
    "match": {
      "name": "tom"
    }
  }
}
```

## Beats入门

简介

- lightweight Data Shipper
  - Filebeat      日志文件
  - Metricbeat    度量数据
  - Packagebeat   网络数据
  - Winlogbeat   Windows数据
  - Hearbeat   健康检查

#### Filebeat简介

- 处理流程
  - 输入input
  - 处理filter
  - 输出output

#### Filebeat Input配置简介

- yaml语法
- input_type
  - log
  - stdin

```yaml
filebeat.prospectors:

​	-input_type:log

​	paths:

​		- /var/log/apache/httpd-*log

​	-input_type:log

​	paths:

​		- /var/log/messages

​		- /var/log/*.log


```

#### Filebeat Output配置简介

- console
- Elasticesearch
- Logstash
- Kafka
- Redis
- File

```yaml
output.elasticsearch:
	hosts:["http://localhost:9200"]
	username:"admin"
	passowrd:"passowd"
```

```yaml
output.console:
	pretty:true
```

#### Filebeat Filter简介

- Input 时处理
  - Include_lines
  - exclude_lines
  - exclude_files

- Output 前处理   -Processor
  - drop_event
  - drop_fields
  - Decode_json_fields
  - Include_fields

#### Filebeat Filter配置简介

```yaml
processors:
	-drop_event:
		when:
			regexp:
				message:"^DBG:"
```

当一个字段以dbg开头时丢弃掉这条数据

#### Filebeat+Elasticsearch Ingest Node

- 新增的node类型
- 在数据写入es前对数据进行处理装换
- pipeline api

#### Packetbeat简介

- 实时抓取网络包
- 自动解析应用层协议
  - ICMP(v4 and v6)
  - DNS
  - HTTP
  - Mysql
  - Redis
  - ......

packetbeat解析http协议

​	解析elasticesearch http请求

```yaml
packetbeat.interfaces.device: any
packetbeat.protocols.http: ports: [9200]
send_request: true
include_body_for: ["application/json","x-www-form-urlencoded"]
output.console:
	pretty: true
```

运行命令：

sudo ./packetbeat -e -c es.yml -strict.perms=false

## Logstash入门

- Input
  - file
  - redis
  - beats
  - kafka

- filter
  - grok
  - mutate
  - drop
  - date

- Output
  - reids
  - kafka
  - elasticsearch

#### 处理流程　--Input和Output配置

```shell
input {file {path=> "/tmp/abc.log"}}
```

```shell
output {stout{codec => rubydebug}}
```

#### 处理流程 -- Filter 配置

- Grok
  - 基于正则表达式提供了丰富可重用的模式(pattern)
  - 基于此可以将非结构化数据作结构化处理

- Date
  - 将字符串类型的时间字段转换为时间戳类型,方便后续数据处理

- Mutate
  - 进行增加、修改、删除、替换等字段相关的处理

- ...

