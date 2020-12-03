### 1、安装步骤，创建以下目录
```
## 放置es的配置文件
mkdir -p /usr/local/es/config
touch /usr/local/es/config/es1.yml
touch /usr/local/es/config/es2.yml
touch /usr/local/es/config/es3.yml

## 存放es插件
mkdir -p /usr/local/es/plugins1
mkdir -p /usr/local/es/plugins2
mkdir -p /usr/local/es/plugins3

# 存放es数据目录
mkdir -p /data/es/es1
mkdir -p /data/es/es2
mkdir -p /data/es/es3
```

### 2-1、配置文件 /usr/local/es/config/es1.yml
```
cluster.name: elasticsearch-cluster
node.name: es-node1
network.bind_host: 0.0.0.0
network.publish_host: 192.168.110.8
http.port: 9200
transport.tcp.port: 9300
http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: true 
node.data: true  
discovery.zen.ping.unicast.hosts: ["192.168.110.8:9300","192.168.110.8:9301","192.168.110.8:9302"]
discovery.zen.minimum_master_nodes: 2
```

### 2-2、配置文件 /usr/local/es/config/es2.yml
```
cluster.name: elasticsearch-cluster
node.name: es-node2
network.bind_host: 0.0.0.0
network.publish_host: 192.168.110.8
http.port: 9201
transport.tcp.port: 9301
http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: true 
node.data: true  
discovery.zen.ping.unicast.hosts: ["192.168.110.8:9300","192.168.110.8:9301","192.168.110.8:9302"]
discovery.zen.minimum_master_nodes: 2
```

### 2-3、配置文件 /usr/local/es/config/es3.yml
```
cluster.name: elasticsearch-cluster
node.name: es-node3
network.bind_host: 0.0.0.0
network.publish_host: 192.168.110.8
http.port: 9202
transport.tcp.port: 9302
http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: true 
node.data: true  
discovery.zen.ping.unicast.hosts: ["192.168.110.8:9300","192.168.110.8:9301","192.168.110.8:9302"]
discovery.zen.minimum_master_nodes: 2
```

### 2-4、配置文件解析
```
#集群名
cluster.name: crm
 
#节点名
node.name: node-111

# 默认的配置是把索引分为5个分片
#index.number_of_shards: 2

# 设置每个index的默认的冗余备份的分片数，默认是1
#index.number_of_replicas: 1

#设置绑定的ip地址，可以是ipv4或ipv6的，默认为0.0.0.0，指绑定这台机器的任何一个ip
network.bind_host: 0.0.0.0
 
#设置其它节点和该节点交互的ip地址，如果不设置它会自动判断，值必须是个真实的ip地址 
network.publish_host: 192.168.1.111
 
#设置对外服务的http端口，默认为9200
http.port: 9200
 
#设置节点之间交互的tcp端口，默认是9300
transport.tcp.port: 9300
 
#是否允许跨域REST请求
http.cors.enabled: true
 
#允许 REST 请求来自何处
http.cors.allow-origin: "*"
 
#节点角色设置
node.master: true
node.data: true
 
#有成为主节点资格的节点列表
discovery.zen.ping.unicast.hosts: ["192.168.1.111:9300","192.168.1.112:9300","192.168.1.113:9300"]
 
#集群中一直正常运行的，有成为master节点资格的最少节点数（默认为1） (totalnumber of master-eligible nodes / 2 + 1)
discovery.zen.minimum_master_nodes: 2

# 当JVM做分页切换（swapping）时，ElasticSearch执行的效率会降低，推荐把ES_MIN_MEM和ES_MAX_MEM两个环境变量设置成同一个值，并且保证机器有足够的物理内存分配给ES，同时允许ElasticSearch进程锁住内存
#bootstrap.memory_lock: true
```

### 3、启动容器
```
docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d  -p 9200:9200 -p 9300:9300 -v /usr/local/es/config/es1.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /usr/local/es/plugins1:/usr/share/elasticsearch/plugins -v /data/es/es1:/usr/share/elasticsearch/data --name ES01 elasticsearch:5.6.4

docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d  -p 9201:9201 -p 9301:9301 -v /usr/local/es/config/es2.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /usr/local/es/plugins2:/usr/share/elasticsearch/plugins -v /data/es/es2:/usr/share/elasticsearch/data --name ES02 elasticsearch:5.6.4

docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d  -p 9202:9202 -p 9302:9302 -v /usr/local/es/config/es3.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /usr/local/es/plugins3:/usr/share/elasticsearch/plugins -v /data/es/es3:/usr/share/elasticsearch/data --name ES03 elasticsearch:5.6.4
```

### 4、验证是否搭建成功
```
http://192.168.110.8:9200/_cat/nodes?pretty
```
![image.png](https://github.com/xinput123/about-me/blob/main/Docker/image/docker01.png)

- 有*号的节点名称表示为主节点

### 5、使用 elasticsearch-head 前端框架
```
## 拉取
docker pull mobz/elasticsearch-head:5

## 启动
docker run -d -p 9100:9100 --name es-manager  mobz/elasticsearch-head:5

## 浏览器访问
http://192.168.110.8:9100
```