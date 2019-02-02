# 1.docker的安装

- win10家庭版安装总结

1. 下载docker toolbox镜像，并安装(一直next)

   <http://mirrors.aliyun.com/docker-toolbox/windows/docker-toolbox/>

2. 双击 Docker Quickstart Terminal 图标运行

3. 如果以前安装了git需要手动选择git/bin/bash.exe

   也可：在右击→属性→目标中设置如下
   "C:\Program Files\Git\bin\bash.exe" --login -i "C:\Program Files\Docker Toolbox\start.sh"

4. 下载boot2docker.iso插件

   <https://github.com/boot2docker/boot2docker/releases/download/v18.05.0-ce/boot2docker.iso>
   如果运行报错，需要手动下载boot2docker.iso放到默认路径中，再运行

注：
	cup虚拟化要打开
	任务管理器→性能→cup中查看



# 2.命令总结

## 1.帮助、辅助

```shell
$ docker version
$ docker info
$ docker --help
```



## 2.镜像

```shell
$ docker images
	-a  -q  --digests  --no-trunc
$ docker search image_name    #查找仓库中的镜像
	-s : $ docker search -s 30 tomcat
	--no-trunc
	--automated : 只列出automated build 类型镜像
$ docker pull image_name:tag	#从仓库获取镜像
$ docker push image_name:tag	#上传自己的镜像到仓库
#在上传的时候要先登录：$docker login  登录dockerhub仓库
$ docker rmi [image_name:tag ...]
	-f
	$ docker rmi -f $(docker images -qa)  #全部删除镜像
$ docker history imageId #列出镜像的变更历史,可以查看镜像的构建过程
$ docker tag image_name:tag new_name:tag	#为镜像重命名（包括tag）
```

### 2.1 创建镜像

- save

例如：存储本地的ubuntu:latest镜像为文件ubuntu_latest.tar 

```shell
#命令：docker save
#命令样例：
docker save -o docker_io_ubuntu_latest.tar docker.io/ubuntu:latest
#tar包存储位置：当前路径
```

- load
  - 将准备好的Docker镜像tar包上传至目标虚拟机的任意路径上。[/opt/images] 
  - 载入镜像。
  - 使用docker ps images命令确认是否载入成功。

```shell
#命令：docker load  用于从tar文件创建镜像;
docker load --input /opt/images/rhel7.3.tar
#-i、--input=“”:不使用标准输入，设置文件路径并创建镜像
docker load < **..tar    #如docker load < metricsservice\ (3).tar 
```

## 3.容器

```shell
$ docker run [options] image [command] [arg...]  #新建/启动容器
    [options]:
    --name=”new_name”  → 为容器指定一个名词
    -d  → 后台运行
    -i  → 以交互模式运行
    -t  → 为容器重新分配一个伪输入终端
    -P  → 随机端口映射
    -p  → 指定端口映射；有四种模式
        ip:hostPort:containerPort
        ip::containerPort
        hostPort:containerPort
        containerPort
e.g.
$ docker run -d centos echo hello docker
$ docker run -d -p 8080:80 nginx 

$ docker ps   #查看，发现容器已退出
#说明：docker容器后台运行，就必须有一个前台的进程，容器运行的命令如果不是那些一直挂起的命令（比如：top、tail），就会自动退出。因为docker觉得此容器已经没事可做了

$ docker ps [options]
    [options]
    -a  → 列出当前运行 + 历史运行过的容器
    -l  → 显示最近创建的容器
    -n  → 显示n个最近创建的容器
    -q  → 静默模式，只显示容器编号
    --no-trunc
```

- 退出容器

```shell
exit：停止并退出
Ctrl+P+Q：不停止退出
```

- 正在运行的容器

```shell
#进入正在运行的容器
$ docker attach containerID 
	→ 直接进入容器启动命令的终端，不会启动新进程
$ docker exec -it containerID bashShell
	→ 在容器中打开新的终端，并可以启动新的进程
$ docker exec -it containerID ls -l /tmp
	→ 只输出 $ ls -l /tmp 的结果，不会进入容器交互：隔空操作容器内部
$ docker exec -it containerID /bin/bash
	→ 进入容器交互，打开新的终端
#启动、关闭、删除正在运行的容器
$ docker start containerID/containerName  → 启动容器
$ docker restart containerID  → 重启容器
$ docker stop containerID/containerName  → 停止容器
$ docker kill containerID/containerName  → 强制停止容器
$ docker rm containerID  → 删除已停止的容器
#一次删除所有容器：
$ docker rm -f $(docker ps -a -q)
$ docker ps -a -q | xargs docker rm
#查看日志
$ docker logs -f -t --tail n containerID  → 查看container日志
    -t  加入时间戳；
    -f  跟随最新的日志打印，追加；
    --tail  数字：显示最后多少条
$ docker top containerID   → 查看容器中的进程
$ docker inspect containerID   → 查看容器细节（内部）；可以检查一些容器的启动信息
#把容器中的文件copy到宿主机中
$ docker cp containerID:container路径 目的主机路径 
#把主机中的文件copy到镜像中
$ docker cp index.html containerId://containerAddress[usr/share/nginx/html]
#提交一个新的镜像
$ docker commit [options] containerID 镜像名:[tag]
	-m=”描述”  -a=”作者”
$ docker commit -a=”nyfblack” -m=”del docs” containerID nyf/tomcat:1.2 
	→ 把tomcat的docs删除，重新做一个镜像
#注意：docker在container中做出的改动都是暂时的，没有被保存下来，下次启动又会回到默认值；使用commit创建一个新的image，保存改动的image

$ docker port container_name		#查看端口映射
```



## 4.数据卷Volume

- 提供独立于容器之外的持久化存储，比如数据库操作
- 提供容器之间的共享数据
- 容器本身不能持久化文件，关闭容器后，数据销毁



- 命令

```shell
$ docker run -it -v /宿主机绝对路径目录:/容器目录 镜像名
#命令带权限：
$ docker run -it -v /myDataVolume:/dataVolume:ro centos
	→ 容器侧目录为只读：ro -> readonly 
e.g. 
#两个目录都指定
$ docker run -it -v /myDataVolume:/dataVolume centos
	→ 同时在Host上生成myDataVolume目录，在容器中生成dataVolume目录
#只指定一个宿主机目录
$ docker run -d --name nginx -v /usr/share/nginx/html nginx
	→ -v 挂载一个卷到容器中，/usr/share/nginx/html为容器nginx的资源访问路径
	→ 是把宿主机的此路径挂载到nginx[会自动生成路径]的路径中
```



- dockerfile 添加

```SHELL
dockerfile 的volume指令添加数据卷
#基于移植性的考虑，dockerfile中不支持host目录的设置，host的目录自动生成
构建：
$ docker build -f /mydocker/Dockerfile -t zy/centos

e.g.Dockerfile文件如下：
#volume test
FROM centos
VOLUME [“data1”,”data2”]
CMD /bin/bash

#这样初始化的数据卷目录只能是新创建的，不能是已经存在的目录；会自动在container和host中创建相互对应的数据卷目录，这样创建的数据卷不能共享数据
```

- 可能出现的问题

```shell
#如果docker挂载主机目录，docker访问出现以下错误:
#cannot open directory. : Permission denied
#这是权限问题；解决办法：在挂载目录后添加如下参数即可：
--privileged=true
```

- 数据卷容器

```shell
#创建一个只存储数据的container，可以把这个container的数据加载到其他容器中
$ docker create -v $PWD/data:/var/mydata --name data_container ubuntu
	→$PWD/data 表示当前目录下的data目录
	→/var/mydata 为容器中的目录；--name data_container 给新建的容器命名
	→ubuntu 为新建的容器给定的基础对象
$ docker run -it --volumes-from data_container ubuntu /bin/bash
	→ -it 以交互的方式运行，进入到ubuntu中
	→ --volumes-from data_container 把此容器加载到ubuntu中
# mount //可以在此信息中找到挂载的文件位置
# touch file → # exit //在本地挂载的文件中同步生成修改内容，挂载成功
# 删除了数据卷容器，挂载了此容器的container依然可以访问挂载的目录；数据卷容器只是配置信息的传递。docker规定，正在使用的数据卷会一直存在.
# 容器之间配置信息的传递，数据卷的生命周期一直持续到没有容器使用他为止
```

- 数据卷的备份和还原

```shell
#数据备份的方法：
$ docker run --volumes-from [container_name] -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar [container data volume]

$ docker run --volumes-from dvt5 -v ~/backup:/backup --name dvt10 ubuntu tar cvf /backup/dvt.tar /datavolume1

#数据还原的方法：
$ docker run --volumes-from [container name] -v $(pwd):/backup ubuntu tar xvf /backup/backup.tar [container data volume]
```



## 5.docker server命令

```shell
docker service ls      #查看创建出来的服务                       
docker create <选项><镜像名称，ID><命令><参数>  #使用指定的镜像创建容器 

docker service create --name XXX--network XXX--mode global --mount
type=bind,source=/etc/localtime,destination=/etc/localtime -p 8081:9092  xx 

docker service rm <选项><容器名称，ID>  	#删除容器，          
	#-f：强制停止容器后删除，-l：只删除连接，不删除容器，-v：删除连接到容器的数据卷。                 
docker service update --force ****	#更新服务，         
	#-publish-add参数指添加或者更新一个对外端口 image参数指更新镜像 hostname 更新或指定容器名称force 指强制更新，即使本次更新没有任何改变  
```



# 3.Dockerfile构建image

Dockerfile ----build---> image ----run----> container 

面向开发                         交付标准                涉及部署和运维

## 1.Dockerfile命令

| 指令          | 说明                   |
| ------------- | ---------------------- |
| FROM          | base image             |
| EXPOSE        | 暴露端口               |
| MAINTAINER    | 维护者                 |
| .dockerignore | 忽略；与.gitignore相似 |


RUN：镜像构建时执行该命令

CMD：容器运行时执行的默认命令，会被docker run指定的启动命令覆盖

ENTERYPOINT：不会被docker run命令中指定的启动命令覆盖；如果想覆盖，需要显式的在docker run中指定ENTERYPOINT指令

CMD/ENTERYPOINT：组合使用，CMD指定默认参数，ENTERYPOINT指定命令

---

ADD/COPY src desc → src为相对路径，desc为绝对路径 

ADD：tar解压/URL下载 + copy

COPY：如果只是copy文件，推荐使用此命令

VOLUME：添加卷

---

WORKDIR：从image创建container时设置工作目录，CMD/ENTERYPOINT都会在该目录下执行；在构建中为后续的指令指定工作目录；通常使用绝对路径，如果使用相对路径，结果会向下传递

ENV：设置环境变量；可以作用于构建中，运行中依然有效

```shell
ENV MY_PATH /usr/mytest      #定义
WORKDIR $MY_PATH             #使用
```

---

USER：指定该镜像以什么样的用户身份运行；也可以在此命令中使用用户组(uid，gid)，如果不指定，则默认使用root用户； 

```shell
USER user		USER user:group		USER user:gid
USER uid		USER uid:group		USER uid:gid
```

---

ONBUILD：为镜像添加触发器；当一个镜像被其他镜像作为基础镜像时执行；会在构建过程中插入指令； 

```shell
ONBUILD COPY index.html /usr/share/nginx/html/
```



## 2.Docker镜像的构建过程

- 从基础镜像运行一个容器
- 执行一条指令，对容器进行修改
- 执行类似docker commit的操作，提交一个新的镜像层
- 再基于刚提交的镜像层运行一个新容器
- 执行Dockerfile中的下一条指令，直至所有指令执行完毕



**在构建过程中，中间层的容器会被删除，但是中间层的镜像会保留下来，可以通过中间层的镜像来调试；** 

- docker构建缓存；再次构建时，直接使用缓存，使构建过程很高效 
- 不想使用构建缓存时：docker build --no-cache 可以不使用缓存



1. docker镜像分层：

   Docker的image是分层存储的

   Dockerfile中的每一行都产生一个新层

   ```
   Docker的image是只读文件，Docker run后的container会附件一个虚拟层，在此层可以读写，但是原层都是只读的。要想保存修改，只能重新commit一个新的镜像。Docker的操作思想：写时复制
   ```



## 3.tomcat的构建

```shell
FROM openjdk:8-jre   #openjdk:8-jre→基镜像

MAINTAINER

ENV JAVA_HOME /docker-java-home
ENV CATALINA_HOME /usr/local/tomcat
ENV PATH $CATALINA_HOME/bin:$PATH
ENV TIME_ZONE Asia/Shanghai
RUN mkdir -p "$CATALINA_HOME"
WORKDIR $CATALINA_HOME

RUN set -x \
\
# 下载Tomcat压缩文件
&& wget -O tomcat.tar.gz 'https://www.apache.org/dyn/closer.cgi?action=download&filename=tomcat/tomcat-8/v8.5.16/bin/apache-tomcat-8.5.16.tar.gz' \
# 解压
tar -xvf tomcat.tar.gz --strip-components=1 \
# 删除供Windows系统使用的.bat文件
rm bin/*.bat \
# 删除Tomcat压缩文件
&& rm tomcat.tar.gz* \
\
# 更改时区
&& echo "${TIME_ZONE}" > /etc/timezone \
&& ln -sf /usr/share/zoneinfo/${TIME_ZONE} /etc/localtime \
\
# 处理Tomcat启动慢问题（随机数产生器初始化过慢）
&& sed -i "s#securerandom.source=file:/dev/random#securerandom.source=file:/dev/./urandom#g" $JAVA_HOME/jre/lib/security/java.security

EXPOSE 8080
CMD ["catalina.sh", "run"]
```

构建：

```shell
$ docker build -f /mydocker/Dockerfile -t tomcat:8.5.15-jre8
#如果dockerfile的命名为Dockerfile 则-f参数指定文件可省略；docker会默认使用名为Dockerfile的文件创建镜像
#进入Dockerfile所在路径，执行以下命令构造镜像
$ docker build -t tomcat:8.5.15-jre8 .   # . 表示此路径下的所有文件

#运行容器
$ docker run -d --name tomcat-test -p 8888:8080 tomcat:8.5.15-jre8
```



- 有加速器的Dockerfile示例：

```shell
FROM ubuntu
MAINTAINER xbf
##使用国内的镜像，用来加速##
RUN sed -i ‘s/archive.ubuntu.com/mirrors.ustc.edu.cn/g’ /etc/apt/sources.list
RUN apt-get update
RUN apt-get install -y nginx
COPY index.html /var/www/html
ENTRYPOINT[“/usr/sbin/nginx”,”-g”,”daemon off;”]
EXPOSE 80 
```



# 4.多容器APP(docker-compose)

## 1.docker-compose的安装

- Mac、windows自带
- linux安装：

```shell
$ curl -L https://github.com/docker/compose/releases/download/1.9.0/docker-compose-$(uname -s)-$(uname -m) > /usr/local/bin/docker-compose
#-$(uname -s)-$(uname -m) 表示把 uname -s 和 uname -m 的输出写到此路径中；再把结果通过 > 写入到文件中
$ chmod a+x /usr/local/bin/docker-compose #把此文件设置为所有用户可执行文件
$ docker-compose --version #查看版本，以确认是否安装成功
```



## 2.使用示例

user → nginx → ghost app → mysql 

1. 创建以下目录结构：

   ghost

   ​	|ghose 

   ​		|Dockerfile[f]

   ​		|config.js

   ​	|nginx

   ​		|Dockerfile[f]

   ​		|nginx.conf

   ​	|data

   ​	|docker-compose.yml

2. 编写文件

ghost/Dockerfile

```shell
FROM ghost
COPY ./config.js /var/lib/ghost/config.js
EXPOSE 2368
CMD [“npm”,”start”,”--production”]
```

ghost/config.js

```js
var path = require(‘path’),
config;

config = {
    production: {
        url: ‘http://mytestblog.com’,
        mail: {},
        database: {
            client : ‘mysql’,
            connection : {
               host : ’db’,
               user : ’ghost’,
               password : ’ghost’,
               database : ’ghost’,
               port : ’3306’,
               charset : ‘utf8’
            }
            debug : false
        },
        paths: {
            contentPath: path.join(pocess.env.GHOST_CONTENT, ‘/’)
    	},
        server: {
            host: ’0.0.0.0’,
            port: ‘2368’
        }
     }
};
module.exports = config;
```

nginx/Dockerfile

```shell
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
```

nginx/nginx.conf

```shell
worker_processes 4;
events {worker_connections 1024;}
http {
   server {
      listen 80;
      location / {
         proxy_pass http://ghost-app:2368;
      }
   }
}
```

docker-compose.yml

```shell
version:’2’
networks:
  ghost:
services:
  ghost-app:
    build: ghost
    networks:
      - ghost
    depends_on:
      - db
    ports:
      - “2368:2368”
  nginx:
    build: nginx
    networks:
      - ghost
    depends_on:
      - ghost-app
    ports:
      - “80:80”
  db:
    image: “mysql:5.7.15”
    networks:
      - ghost
    environment:
      MYSQL_ROOT_PASSWORD: mysqlroot
      MYSQL_USER: ghost
      MYSQL_PASSWORD: ghost
    volumes:
      - $PWD/data:/var/lib/mysql
    ports:
      - “3306:3306”
```

3. 构建容器

```shell
$ docker-compose up -d  #构建容器
# 如果第一次构建错误，需要如下操作
$ docker-compose stop   #停止已运行的compose-app
$ docker-compose rm     #删除已停止的compose-app
$ docker-compose build  #重新构建
$ docker-compose up -d  #重新构建容器

#项目访问路径：localhost/ghost/setup/
```



## 3.docker-compose常用命令

- docker-compose.yml常用指令

| 命令       | 说明         |
| ---------- | ------------ |
| build      | 本地创建镜像 |
| command    | 覆盖缺省命令 |
| depends_on | 连接容器     |
| ports      | 暴露端口     |
| volumes    | 卷           |
| image      | pull镜像     |

- docker-compose常用命令

| 命令       | 说明         |
| ---------- | ------------ |
| up   | 启动服务             |
| stop | 停止服务             |
| rm   | 删除服务中的各个容器 |
| logs | 观察各个容器的日志   |
| ps   | 列出服务相关的容器   |



# 5.docker容器网络实现

## 1.docker容器网络基础

- docker0：相当于一个虚拟网桥

  Linux虚拟网桥的特点：

  ​	可以设置IP地址；

  ​	相当于拥有一个隐藏的虚拟网卡；

  运行docker image时，相当于创建了网络连接的两端，即container中的eth0和host中的veth

  HOST 			docker0[veth   veth     veth]

  ​						   |          |          |

  containers			       eth0     eth0     etho

```
docker为每个container都分配了一个Mac地址
```

- ubuntu的网桥管理工具

```shell
apt-get install bridge-utils
brctl show		#查看网桥设备
```

- 自定义docker0

```shell
#修改docker0地址：
$ sudo ifconfig docker0 192.168.200.1 netmask 255.255.255.0
```

- 自定义虚拟网桥

```shell
#添加虚拟网桥：
$ sudo brctl addbr br0
$ sudo ifconfig br0 192.168.100.1 netmask 255.255.255.0
#更改docker守护进程的启动配置：
$ vi /etc/default/docker #添加DOCKER_OPS值 [-b=br0]
```



## 2.docker容器的互联

- 允许所有容器互联

```shell
--icc=true		#默认
#启动的所有容器是加到同一虚拟网桥上的，所有可以互联
```

容器的ip地址是不可靠的连接，会随着容器的重新启动而改变；

如果在一个容器使用ip来连接了另一个容器，当容器重启的时候，该连接失效；

```shell
#解决办法： --link
$ docker run --link=[container_name]:[alias] [image] [commond]
e.g.
$ docker run -it --name cct3 --link cct1:webtest image_name
#docker会在容器启动的时候自动把cct1的ip映射到webtest上，在cct3中要连接cct1时，直接使用webtest即可
#在/etc/hosts中可以查看到cct1的ip到webtest的映射
```

- 拒绝容器间的互联

  修改docker配置文件，添加`--icc=false` 

```shell
$ sudo vi /etc/default/docker
#添加：DOCKER_OPS值 [--icc=false]
```

​	考虑到安全性，阻断容器之间的连接；

​	修改配置之后需要重启docker服务才能生效；

- 允许特定容器间的连接

```shell
Docker守护进程的启动项配置：
--icc=false 
--iptables=true
启动时设置：
--link
说明：
阻断所有容器之间的访问，只使用link设置的容器访问

#Linux命令：
$ sudo iptables -L -n	#查看设置
$ sudo iptables -f 		#清空当前内容，重新加载设置
```



## 3.docker容器与外部网络的连接

- ip_forward

  linux中的配置项，此项设置是否允许流量转发

  docker中也有ip-forward配置项：

```shell
--ip-forward=true	//默认
在docker启动的时候会自动设置linux中的ip_forward为1，即允许流量转发

查看linux中的ip_forward设置：
$ sysctl net.ipv4.conf.all.forwarding
```

- iptables

  Iptables是Linux内核集成的包过滤防火墙系统。

  Iptables包含：

```shell
表（table）	同一类操作
链（chain）	数据处理中的不同环节（阶段）
规则（rule）	每个链下的操作
	ACCEPT/REJECT/DROP
#filter表中包含的链：
INPUT		FORWARD		OUTPUT
#查看filter表：
$ sudo iptables -t filter -L -n
    -t：指定查看的表明，默认查看filter表
    -L：执行的操作
    -n：列表显示
$ sudo iptables -L -n		//默认就是filter表
```

​	限制ip访问容器

```shell
#可以限制特定的端口访问特定的容器：
$ sudo iptables -I DOCKER -s 10.211.55.3 -d 172.17.0.7 -p TCP --dport 80 -j DROP
    -s：源路径ip：被访问的ip
    -d：目标ip：被禁止的ip
    -p：协议
    --dport：端口
    -j：访问规则
```

# 5.docker容器的跨主机连接

1. 使用网桥实现跨主机容器连接

```SHELL
#网络设置：修改/etc/network/interfaces文件
auto br0
iface br0 inet static
address 10.211.55.3
netmask 255.255.255.0
gateway 10.211.55.1
bridge_ports eth0
```

​	Docker修改

```shell
修改/etc/default/docker文件
    -b指定使用自定义网桥
        -b=br0
    --fixed-cidr限制ip地址分配范围
        IP地址划分：
        Host1:10.211.55.64/26
            地址范围：10.211.55.65~10.211.55.126
        Host2:10.211.55.128/26
            地址范围：10.211.55.129~10.211.55.190
```

**总结：**

优点：

​	配置简单，不依赖第三方软件

缺点：

​	与主机在同网段，需要小心划分IP地址

​	需要有网段控制权，在生产环境中不易实现

​	不容易管理

​	兼容性不佳

1. 使用Open vSwitch实现跨主机容器连接

   Open vSwitch是一个高质量的、多层虚拟交换机；

操作步骤：

- 建立ovs网桥
- 添加gre连接
- 配置docker容器虚拟网桥
- 为虚拟网桥添加ovs接口
- 添加不同docker容器网段路由

```shell
#建立ovs网桥，添加gre连接
$ apt-get install openvswitch-switch		#安装
$ sudo ovs-vsctl show		#查看ovs的版本
$ sudo ovs-vsctl add-br obr0
$ sudo ovs-vsctl add-port obr0 gre0
$ sudo ovs-vsctl set interface gre0 type=gre options:remote_ip=192.168.59.104
$ sudo ovs-vsctl show

#设置虚拟网桥并连接到gre上
$ sudo brctl addbr br0
$ sudo ifconfig br0 192.168.1.1 netmask 255.255.255.0
$ sudo brctl addif br0 obr0
$ suod brctl show

$ route		#查看路由表
$ sudo ip route add 192.168.2.0/24 via 192.168.59.104 dev eth0		#添加路由表
```



3. 使用weave实现跨主机容器连接

   建立一个虚拟的网络，用于将运行在不同主机的Docker容器连接起来

   <http://weave.works>

   <https://github.com/weaveworks/weave#readme>

操作步骤：

- 安装Weave
- 启动Weave    `$weave launch`
- 连接不同主机
- 通过weave启动容器

DockerHost1：

```shell
#下载、安装weave：
$ sudo wget -o /usr/bin/weave https://raw.githubusercontent.com/zettio/weave/master/weave
$ sudo chmod a+x /usr/bin/weave
#启动weave：
$ weave launch
#通过weave启动容器：
$ weave run 192.168.1.10/24 -it --name wc1 ubuntu /bin/bash
$ docker attach wc1
$ ping 另一台主机的container_addr
```

DockerHost2：

```shell
#下载安装同host1；
#启动weave：
$ weave launch 192.168.59.103
#启动容器：
$ c2=$(weave run 192.168.1.2/24 -it ubuntu /bin/bash)
$ echo $c2	#查看c2的值
$ docker attch $c2
```



# 附录1：docker相关网站

| 标签             | 网址                                   |
| ---------------- | -------------------------------------- |
| docker官网-中国  | http://docker-cn.com/                  |
| docker官网       | https://www.docker.com                 |
| docker下载地址   | get.daocloud.io                        |
| docker官方模拟器 | https://www.docker.com/tryit/          |
| docker官网资料   | https://docs.docker.com                |
| docker中国社区   | http://dockboard.org/                  |
| docker@github    | http://github.com/docker/docker/issues |


docker的镜像仓库：

- daocloud: http://hub.daocloud.io/

- aliyun:https://opsx.alibaba.com/mirror

    https://dev.aliyun.com/search.html                  

- 网易云：c.163.com；<http://mirrors.163.com/>1111

- docker默认仓库：https://hub.docker.com



docker客户端与守护进程：

[Docker官方的Remote Api Reference](https://docs.docker.com/reference/api/docker_remote_api/)



一些博文：

[docker tools外部不能访问内部端口](https://blog.csdn.net/xbinworld/article/details/78945879)



# 附录2：术语

| host      | 宿主机   |
| --------- | -------- |
| image     | 镜像     |
| container | 容器     |
| registry  | 仓库     |
| daemon    | 守护进程 |
| client    | 客户端   |







