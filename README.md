##### 1.    服务器安装ElasticSearch
>   服务器版本：CentOS Linux release 7.6.1810 (Core) 
>   ElasticSearch版本：6.4.3
-   官网下载链接
[https://www.elastic.co/guide/en/elasticsearch/reference/6.4/install-elasticsearch.html#install-elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/install-elasticsearch.html#install-elasticsearch)

-   上传到服务器目录（本人习惯放在/usr/local/src目录下）
-   解压  
```
cd /usr/local/src/
tar -zxvf elasticsearch-oss-6.4.3.tar.gz
```
-   创建存放数据的目录
```
cd elasticsearch-6.4.3/
mkdir data
```
-   修改配置文件
```
cd config/
# 修改启动内存
vim jvm.options

# 修改为以下内容，否则内存不足报错
-Xms256m
-Xmx256m

# :wq保存退出

# 修改elasticsearch.yml
cluster.name: my-application
node.name: node-1
path.data: /usr/local/src/elasticsearch-6.4.3/data
path.logs: /usr/local/src/elasticsearch-6.4.3/logs
network.host: 0.0.0.0
http.port: 9200
discovery.zen.ping.unicast.hosts: ["0.0.0.0"]

# :wq保存退出
```

-   创建组和用户(ElasticSearch不能用root用户启动)
```
# 创建组
groupadd es
# 创建用户并加入组
useradd es -g es

# 给es用户赋予对ElasticSearch的执行权限
chown es:es /usr/local/src/elasticsearch-6.4.3/ -R

```
-   修改/etc/sysctl.conf 
```
vim /etc/sysctl.conf
vm.max_map_count=262144

# 保存退出后,使用sysctl -p 刷新生效

```

-   修改文件/etc/security/limits.conf
```
es soft nofile 65536
es hard nofile 131072
es soft nproc  4096
es hard nproc  4096
```

-   启动
```
# 进入到es用户下
su - es 

cd /usr/local/src/elasticsearch-6.4.3/bin

# 启动
./elasticsearch

# 后台启动
./elasticsearch -d
```

-   启动完成不报错即可，报错请自行百度，通过ip地址访问9200端口，会出现以下JSON串
```json
{
    "name": "node-1",
    "cluster_name": "my-application",
    "cluster_uuid": "RwRIXNCvRzK3kJr_R4BlmA",
    "version": {
        "number": "6.4.3",
        "build_flavor": "oss",
        "build_type": "tar",
        "build_hash": "fe40335",
        "build_date": "2018-10-30T23:17:19.084789Z",
        "build_snapshot": false,
        "lucene_version": "7.4.0",
        "minimum_wire_compatibility_version": "5.6.0",
        "minimum_index_compatibility_version": "5.0.0"
    },
    "tagline": "You Know, for Search"
}
```

##### 2.    整合springboot
-   导入pom
```xml
     <!-- https://mvnrepository.com/artifact/org.elasticsearch/elasticsearch -->
          <dependency>
              <groupId>org.elasticsearch</groupId>
              <artifactId>elasticsearch</artifactId>
              <version>6.4.3</version>
          </dependency>
  
          <!-- https://mvnrepository.com/artifact/org.elasticsearch.client/transport -->
          <dependency>
              <groupId>org.elasticsearch.client</groupId>
              <artifactId>transport</artifactId>
              <version>6.4.3</version>
              <exclusions>
                  <exclusion>
                      <groupId>org.elasticsearch</groupId>
                      <artifactId>elasticsearch</artifactId>
                  </exclusion>
              </exclusions>
          </dependency>
  
          <dependency>
              <groupId>log4j</groupId>
              <artifactId>log4j</artifactId>
              <version>1.2.17</version>
          </dependency>
         <!-- <dependency>
              <groupId>org.elasticsearch</groupId>
              <artifactId>elasticsearch</artifactId>
              <version>6.4.2</version>
          </dependency>-->
  
          <dependency>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok</artifactId>
              <version>1.16.20</version>
          </dependency>
  
          <dependency>
              <groupId>com.alibaba</groupId>
              <artifactId>fastjson</artifactId>
              <version>1.2.39</version>
          </dependency>
  
          <dependency>
              <groupId>org.apache.commons</groupId>
              <artifactId>commons-lang3</artifactId>
              <version>3.4</version>
          </dependency>
  
          <dependency>
              <groupId>commons-httpclient</groupId>
              <artifactId>commons-httpclient</artifactId>
              <version>3.1</version>
          </dependency>
```

