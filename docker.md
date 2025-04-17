# Docker

#### docker镜像命令

1. docker images 	//查看所有docker镜像
2. docker --help    //查看docker帮助命令
3. docker pull xxxx   //下载镜像
4. docker rmi -f  镜像id      //根据镜像id删除镜像 
5. docker rmi -f  $(docker images -aq)     //删除所有镜像



#### docker命令

1. docker run [可选]  容器镜像   //后台新建并启动容器  

   --name="name" //设置容器名字

   -d     //后台运行

   -it  //使用交互方式运行 进入容器

   -p  主机端口:容器端口   //端口映射

   -P    //随机指定端口  

   -v   /home/data（服务器目录）:/home/data（docker目录）      //把容器的目录挂载到服务器上

   -v   卷名:/home/data          //具名挂载

   -v   /home/data                  //匿名挂载

   ```
   docker run -itd --name mysql-test -p 3306:3306 -v /etc/mysql/:/etc/mysql/conf.d/ -v /data/mysql/data:/var/lib/mysql  -e MYSQL_ROOT_PASSWORD=123456 mysql	
   //后台运行mysql容器 指定映射端口3306  挂载目录
   ```

2. docker ps    //查看当前运行容器

3. docker ps -a   //查看所有运行过的容器

4. docker rm  容器id    //根据容器id删除容器   -f 强制删除

5. docker rm  -f  $(docker ps -aq)     //删除所有容器

6. docker start 容器id    //启动容器

7. docker stop 容器id    //停止容器

8. docker restart 容器id   //重启容器

9. docker kill 容器id   //强制停止当前容器

10. docker logs -f -t --tail 10 容器id     //查看容器最近10条操作日志

11. docker top  容器id    //查看容器的进程信息

12.  docker inspect 容器id    //查看容器的元数据信息

13. docker exec -it 容器id  /bin/bash    //进入容器

14. docker cp 容器id:/home/test  /home  //把容器内的home/test文件拷贝到服务器home目录下

15. docker commit -m="提交描述信息" -a="作者"  容器id  目标镜像名:tag      //提交容器为一个镜像

16. docker volume inspect  卷名  //查看容器文件目录

17. docker  run -it  --volumes-from 容器id  镜像id   //启动新容器并且挂载目录到另一个容器上

18. docker network ls        //查看docker 网络信息





# DockerFile

FROM				#基于什么镜像构建

MAINTAINER			#镜像作者信息  名字+邮箱

RUN				#构建时需要运行的命令

ADD				#添加需要的环境

WORKDIR			#镜像工作目录

VOLUME			#挂载的目录

ESXPOSE			#端口设置

CMD				#容器启动时运行命令，只有最后一个生生效，可被替代

ENTRYPOINT			#容器启动时运行命令，可追加命令

ONBUILD			#当构建一个被继承的dockerfile 会运行ONBUILD指令

COPY				#类似AAD,将我们文件拷贝到镜像中

ENV					#构建的时候设置环境变量



docker build -f  dockerfile文件名  -t  镜像名字	 .	//通过dockerfile 构建镜像



# Docker网络

### 创建自定义网络

```
docker  network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
```

通过自定义网络创建的容器可以直接互相ping通

```
docker run -itd  -P --name centos01 --net mynet centos:7
docker run -itd  -P --name centos02 --net mynet centos:7	//创建自定义网络的centos容器01，容器02

docker exec centos02 ping centos01
PING centos01 (192.168.0.3) 56(84) bytes of data.
64 bytes from centos01.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.068 ms
64 bytes from centos01.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.054 ms
64 bytes from centos01.mynet (192.168.0.3): icmp_seq=3 ttl=64 time=0.074 ms
64 bytes from centos01.mynet (192.168.0.3): icmp_seq=4 ttl=64 time=0.054 ms
```



### 网络联通

``` 
docker network connect  mynet  容器id		//把容器和某个网络打通 那么可以互相ping通
```





# Redis集群部署

for port in $(seq 1 6); \
do \
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
cat << EOF >/mydata/redis/node-${port}/conf/redis.conf
port 6379
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF

docker run -p 637${port}:6379 -p 1637${port}:16379 --name redis-${port} \
-v /mydata/redis/node-${port}/data:/data \
-v /mydata/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.1${port} redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf



# Docker-compose

使用yml文件运行

 ```
docker-compose up	//默认运行当前目录下的docker-compose.yml文件
docker-compose up -d	//后台运行
docker-compose down  //关闭docker-compose
 ```

