

# 一. centos7设置固定IP

1. 查看当前正在使用的网络情况

   ```shell
   [root@localhost ~]# nmcli dev status
   ```

   显示情况 :

   ```shell
   DEVICE  TYPE      STATE   CONNECTION 
   ens33   ethernet  连接的   ens33      
   lo      loopback  未托管   --   
   ```

2. 修改网络相关配置文件:

   **注意: 下面命令中ifcfg-ens33为自己当前使用的网络名称和上面显示的ens33对应**

   ```shell
   [root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-ens33
   ```

3. 修改内容如下:

   ```shell
   TYPE=Ethernet
   PROXY_METHOD=none
   BROWSER_ONLY=no
   BOOTPROTO=static		//如果是dhcp改为static
   DEFROUTE=yes
   IPV4_FAILURE_FATAL=no
   IPV6INIT=yes
   IPV6_AUTOCONF=yes
   IPV6_DEFROUTE=yes
   IPV6_FAILURE_FATAL=no
   IPV6_ADDR_GEN_MODE=stable-privacy
   NAME=ens33
   UUID=b8fd5718-51f5-48f8-979b-b9f1f7a5ebf2
   DEVICE=ens33
   ONBOOT=yes				 //如果是no, 改为yes
   IPADDR=192.168.200.128   //ip地址
   GATEWAY=192.168.200.2	//网关, 我的是2, 改成自己的
   NETMASK=255.255.255.0   //子网掩码
   NM_CONTROLLED=no 
   DNS1=8.8.8.8 			//dns1
   DNS2=8.8.4.4 			//dns2
   ```

4. 重启网络设置, 让配置生效:

   ```shell
   [root@localhost ~]# systemctl restart network.service 
   ```

5. 使用命令查看更改情况:

   ```shell
   [root@localhost ~]# ip addr
   ```

6. 连接linux虚拟机等待时间长解决方案:

   在使用SecureCRT连接Linux时，出现窗口显示已连接，但命令行迟迟不出现的情况。这个问题肯定让人十分恼火，尤其在时间紧迫的时候更是让人忍无可忍.

​		**问题原因:其实问题就出在DNS解析IP上**

​		**解决方法如下：**

 (1) 进入配置文件

```shell
vim /etc/ssh/sshd_config 
```

(2) 进入文件vim /etc/ssh/sshd_config中，添加 UseDNS no，之后保存退出。

```shell
#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
UseDNS no
HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_dsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key
```

(3) 使用命令重启ssh服务即可

```shell
[root@localhost ~]# service sshd restart   #centos6使用的命令
[root@localhost ~]# systemctl restart sshd.service  #centos7使用的命令
```



# 二. docker软件设置常用命令

**systemctl**命令是系统服务管理器指令

## 1.启动docker：

```
[root@localhost ~]# systemctl start docker
```

## 2.停止docker：

```
[root@localhost ~]# systemctl stop docker
```

## 3.重启docker：

```
[root@localhost ~]# systemctl restart docker
```

## 4.查看docker状态：

```
[root@localhost ~]# systemctl status docker
```

## 5.开机启动容器：

```
[root@localhost ~]# systemctl enable docker
```



# 三. docker镜像相关常用命令

## 1.查看镜像

```
docker images
```

REPOSITORY：镜像名称

TAG：镜像标签

IMAGE ID：镜像ID

CREATED：镜像的创建日期（不是获取该镜像的日期）

SIZE：镜像大小

这些镜像都是存储在Docker宿主机的/var/lib/docker目录下

## 2.搜索镜像

如果你需要从网络中查找需要的镜像，可以通过以下命令搜索

```
docker search 镜像名称
```

NAME：仓库名称

DESCRIPTION：镜像描述

STARS：用户评价，反应一个镜像的受欢迎程度

OFFICIAL：是否官方

AUTOMATED：自动构建，表示该镜像由Docker Hub自动构建流程创建的

## 3.拉取镜像

拉取镜像就是从中央仓库中下载镜像到本地

```
docker pull 镜像名称
```

例如，我要下载centos7镜像

```
docker pull centos:7
```

## 4.删除镜像

按镜像ID删除镜像

```
docker rmi 镜像ID
```

删除所有镜像

```
docker rmi `docker images -q`
```

## 

# 四. docker容器相关常用命令

## 1.容器查看命令

查看正在运行的容器

```
docker ps
```

查看所有容器

```
docker ps –a
```

## 2.容器的创建与启动

创建容器常用的参数说明：

创建容器命令：docker run

 -i：表示运行容器

 -t：表示容器启动后会进入其命令行。加入这两个参数后，容器创建就能登录进去。即分配一个伪终端。

 --name :为创建的容器命名。

 -v：表示目录映射关系（前者是宿主机目录，后者是映射到宿主机上的目录），可以使用多个－v做多个目录或文件映射。注意：最好做目录映射，在宿主机上做修改，然后共享到容器上。

 -d：在run后面加上-d参数,则会创建一个守护式容器在后台运行（这样创建容器后不会自动登录容器，如果只加-i -t两个参数，创建后就会自动进去容器）。

 -p：表示端口映射，前者是宿主机端口，后者是容器内的映射端口。可以使用多个-p做多个端口映射

### （1）交互式方式创建容器

```
docker run -it --name=容器名称 镜像名称:标签 /bin/bash
```

这时我们通过ps命令查看，发现可以看到启动的容器，状态为启动状态  

### （2）退出当前容器

```
exit
```

### （3）守护式方式创建容器：

```
docker run -di --name=容器名称 镜像名称:标签
```

### （4）登录守护式容器方式：

```
docker exec -it 容器名称 (或者容器ID)  /bin/bash
```



## 3. 停止与启动容器

停止容器：

```
docker stop 容器名称（或者容器ID）
```

启动容器：

```
docker start 容器名称（或者容器ID）
```

## 4. 文件拷贝

如果我们需要将文件拷贝到容器内可以使用cp命令

```
docker cp 需要拷贝的文件或目录 容器名称:容器目录
```

也可以将文件从容器内拷贝出来

```
docker cp 容器名称:容器目录 需要拷贝的文件或目录
```

## 5. 目录挂载

我们可以在创建容器的时候，将宿主机的目录与容器内的目录进行映射，这样我们就可以通过修改宿主机某个目录的文件从而去影响容器。
创建容器 添加-v参数 后边为   宿主机目录:容器目录，例如：

```
docker run -di -v /usr/local/myhtml:/usr/local/myhtml --name=mycentos3 centos:7
```

如果你共享的是多级的目录，可能会出现权限不足的提示。

这是因为CentOS7中的安全模块selinux把权限禁掉了，我们需要添加参数  --privileged=true  来解决挂载的目录没有权限的问题

## 6. 查看容器IP地址

我们可以通过以下命令查看容器运行的各种数据

```shell
docker inspect 容器名称（容器ID） 
```

也可以直接执行下面的命令直接输出IP地址

```shell
docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器名称（容器ID）
```

## 7. 删除容器 

删除指定的容器：

```
docker rm 容器名称（容器ID）
```



## 8. 容器中什么命令都没有怎么办

首先在容器中执行如下命令更新源:

```shell
apt-get update
```

然后执行安装各种命令的操作:

```shell
apt-get install 命令名称
```





# 五. docker中安装mysql

（1）拉取mysql镜像

```shell
docker pull mysql:5.6
```

（2）创建容器

```shell
docker run -di --name=changgou_mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=admin mysql:5.6
```

-p 代表端口映射，格式为  宿主机映射端口:容器运行端口

-e 代表添加环境变量  MYSQL_ROOT_PASSWORD  是root用户的登陆密码

（3）设置容器开机自动启动

```shell
docker update --restart=always 容器ID
```

（4）远程登录mysql

连接宿主机的IP  ,指定端口为3306



# 六. docker中安装Redis分布式缓存

（1）拉取镜像

```
docker pull redis
```

（2）创建容器

```
docker run -di --name=changgou_redis -p 6379:6379 redis
```

（3）设置容器开机自动启动

```shell
docker update --restart=always 容器ID
```



# 七. docker中安装FastDFS分布式文件系统

（1）拉取镜像

```shell
docker pull morunchang/fastdfs
```

（2）运行tracker

```shell
docker run -d --name tracker --net=host morunchang/fastdfs sh tracker.sh
```

（3）运行storage

```shell
docker run -d --name storage --net=host -e TRACKER_IP=<your tracker server address>:22122 -e GROUP_NAME=<group name> morunchang/fastdfs sh storage.sh
```

- 使用的网络模式是–net=host, <your tracker server address> 位置替换为你机器的Ip即可  

  我的是192.168.200.128

- <group name> 是组名，即storage的组, 例如 : 我设置的是group1  

- 如果想要增加新的storage服务器，再次运行该命令，注意更换 新组名

（4）修改nginx的配置  

进入storage的容器内部

```shell
docker exec -it storage  /bin/bash
```

进入后修改nginx.conf文件

```shell
vi /data/nginx/conf/nginx.conf
```

在文件的location / 节点结束后, 添加以下内容

```shell
location /group1/M00 {
   proxy_next_upstream http_502 http_504 error timeout invalid_header;
     proxy_cache http-cache;
     proxy_cache_valid  200 304 12h;
     proxy_cache_key $uri$is_args$args;
     proxy_pass http://fdfs_group1;
     expires 30d;
 }
```

（5）退出容器

```shell
exit
```

（6）重启storage容器

```shell
docker restart storage
```

   (7) 设置容器开机自动启动

```shell
docker update --restart=always 容器ID
```



# 八. docker中安装RabbitMq消息队列

- 下载镜像 

```shell
docker pull rabbitmq:management
```

- 创建容器 

```shell
docker run -di --name=changgou_rabbitmq -p 5671:5617 -p 5672:5672 -p4369:4369 -p 15671:15671 -p 15672:15672 -p 25672:25672 rabbitmq:management
```

```
解释如下：
	15672 (if management plugin is enabled.管理界面 )

	15671 management监听端口

	5672, 5671 (AMQP 0-9-1 without and with TLS 消息队列协议是一个消息协议)

	4369 (epmd) epmd 代表 Erlang 端口映射守护进程

	25672 (Erlang distribution)
```

- 访问后台 

浏览器中输入地址

```shell
http://192.168.200.128:15672/
```

- 设置容器开机自动启动

```shell
docker update --restart=always 容器ID
```



# 九. docker中安装ElasticSearch搜索

创建elasticsearch和kibana容器

(1) 此elasticsearch和kibana为6.5.4版本

```shell
docker run --name=changgou_es -d -p 9200:9200 -p 5601:5601 nshou/elasticsearch-kibana
```

容器创建完成后可以访问http://192.168.200.128:9200和http://192.168.200.128:5601查看es安装是否成功.

(2) 创建elasticsearch-head插件工具容器

```shell
docker run -d --name=changgou_es_head -p 9100:9100 mobz/elasticsearch-head:5
```

(3) 设置容器开机自动启动

```shell
docker update --restart=always 容器ID
```



### 1. 安装elasticsearch和kibana容器

下载elasticsearch-head

```shell
docker pull mobz/elasticsearch-head:5
```

- 下载elasticsearch版本为5.6.16

```shell
docker pull elasticsearch:5.6.16
```

- 下载kibana版本号为5.6.16

```shell
docker pull kibana:5.6.16
```

- 创建一个叫做somenetwork自定义网络

```shell
docker network create somenetwork
```

- 创建elasticsearch容器(开发模式)

```shell
docker run -d --name changgou_es --net somenetwork -p 9200:9200 -p 9300:9300 -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -e "discovery.type=single-node" elasticsearch:5.6.16
```

- 查看docker 下的网络  包括自带的及自定义

```shell
docker network ls 
```

- somenetwork 是自定义网络  查看此自定义网络下 elasticsearch 的ip

```shell
docker inspect somenetwork
```

显示如下:   会看到IP地址为172.18.0.2/16

```json
[
    {
        "Name": "somenetwork",
        "Id": "3ac29fe43eae9287dcfaed2096948a29c3486b871feda29c71487cd300ef44ba",
        "Created": "2019-05-31T13:09:58.07434932+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "3fb4988562c80a06cc9db2940383d2596289232855c105278ab5d4b76d466ff8": {
                "Name": "changgou_es",
                "EndpointID": "c417394e4b2db8180469f3dfd2853598ff593d458f402642998873a0988b4af8",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

- 注意：在此示例中，Kibana使用默认配置，并期望在http://localhost:9200连接到正在运行的Elasticsearch实例
- 下面的ip是通过上面查看到的elasticsearch的ip 为了让kibana 连接上es

```shell
docker run -d --name kibana --net somenetwork -e ELASTICSEARCH_URL=http://172.18.0.2:9200 -p 5601:5601 kibana:5.6.16
```

- 浏览器可以通过http://192.168.200.128:5601访问



### 2. 集成中文分词器Ik分词器

- 进入elasticsearch容器中

```shell
docker exec -it changgou_es /bin/bash
```

- 下载ik分词器

```shell
wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v5.6.16/elasticsearch-analysis-ik-5.6.16.zip
```

- 进入elasticsearch的插件目录

```shell
cd /usr/share/elasticsearch/plugins/
```

- 创建一个叫做ik的目录

```shell
mkdir ik
```

- 回到ik分词器下载的所在目录解压

```shell
unzip elasticsearch-analysis-ik-5.6.16.zip -d ./
```

- 拷贝解压后里面的内容到刚才创建的ik目录中

```shell
 cp elasticsearch/* plugins/ik/
```

- 进入到ik所在目录

```shell
cd /usr/share/elasticsearch/plugins/ik/
```

- 退出容器

```shell
exit
```

- 重启elasticsearch容器

```shell
 docker restart changgou_es
```



### 3. 容器中安装vim编辑器

- 进入容器中, changgou_es为容器名称

```shell
docker exec -it changgou_es /bin/bash
```

- 更新apt

```shell
apt-get update
```

- 安装vi命令

```shell apt-get install vi
 apt-get install vim
```



# 十. centos7中安装JDK8

- JDK版本 jdk-8u181-linux-x64.tar.gz

1. 在root权限下, 到/usr目录中新建java文件夹

   ```shell
   [root@localhost usr]# mkdir java
   ```

   

2. 将 jdk-8u181-linux-x64.tar.gz压缩包拷贝到新建的java文件夹中

   ```shell
   [root@localhost java]# cp /root/jdk-8u181-linux-x64.tar.gz  /usr/java
   ```

   

3. 进入到/usr/java文件夹中

   ```shell
   [root@localhost java]# cd /usr/java
   ```

   

4. 解压 jdk-8u181-linux-x64.tar.gz压缩包

   ```shell
   [root@localhost java]# tar -zxvf jdk-8u181-linux-x64.tar.gz -C ./
   ```

   

5. 编辑Linux系统中环境变量所在文件

   ```shell
   [root@localhost java]# vi /etc/profile
   ```

   

6. 在文件最后加入环境变量设置, 加入后保存文件

   ``` shell
   export JAVA_HOME=/usr/java/jdk1.8.0_181      #这是自己的jdk所在位置
   export CLASSPATH=.:$JAVA_HOME/jre/lib/dt.jar:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/jre/lib/tools.jar
   export PATH=$PATH:$JAVA_HOME/bin
   ```

   

7. 让环境变量生效

   ```shell
   [root@localhost java]# source /etc/profile
   ```

   

8. 测试JDK是否已经配置好

   ```shell
   #执行命令
   [root@localhost java]# java -version
   #显示如下
   java version "1.8.0_181"
   Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
   Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
   #执行命令
   [root@localhost java]# javac -version
   #显示如下
   javac 1.8.0_181
   ```

   

