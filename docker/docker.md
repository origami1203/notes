# Docker

### 安装与配置

1、安装docker

```shell
 yum install -y docker
```

2、查看docker是否安装成功

```shell
yum list installed | grep docker
```
3、启动docker服务(并设置开机自启)

```shell
systemctl start docker.service
systemctl enable docker.service
```

4、查看docker服务状态

```shell
systemctl status docker
```

5. 配置国内加速

```sh
vim  /etc/docker/daemon.json

# 镜像信息
{
    "registry-mirrors": ["https://registry.docker-cn.com"],
    "live-restore": true
}
```

### 镜像相关

1. 查看镜像

```
docker images
```

2. 搜索镜像

```
docker search 镜像名称
```

3. 拉取镜像

```
docker pull 镜像名称:版本号
```

4. 删除镜像

```
docker rmi 镜像名称:版本号
```

### 容器相关

1. 查看容器列表

```sh
# 查看正在运行的容器
docker ps
# 查看所有容器
docker ps -a
```

2. 创建容器

> 容器创建后,在容器列表中即可看到该容器(相当于安装好程序)
>
> 以后用后文的启、停、删就可以操作该容器了.

```shell
# eg: \是换行的意思
docker run -d -p 1337:1337 \
        --network kong-net \
        --name konga \
        -e "NODE_ENV=production"  \
        -e "DB_ADAPTER=postgres" \
        -e "DB_URI=postgresql://kong:kong@172.0.0.1:5432/konga" \
        pantsel/konga
```

**run参数**

- -rm

  在容器退出时自动清理容器内部的文件系统

  在Docker容器退出时,默认容器内部的文件系统仍然被保留,以方便调试并保留用户数据.

  但是,对于前台运行的容器,由于其只是在开发调试过程中短期运行,其用户数据并无保留的必要.

  启动时设置-rm选项，这样在容器退出时就能够自动清理容器内部的文件系统.

- -i

  以交 时使用；

- -t

  为容器重新分配一个伪输入终端，通常与 -i 同时使用；

- --privileged=true

  给予容器内最高权限

- --restart=always

  docker重启后自启

- -p

  指定端口映射，格式为：**主机(宿主)端口:容器端口**，P为随即映射

  - -e

  设置环境变量，如mysql的账号密码，MYSQL_ROOT_PASSWORD=123456

- -v

  文件夹挂在映射

- -d

  以守护进程模式运行容器，退出后容器不会停止

- -it

  创建一个交互式容器，退出后容器容器停止运行

- -id

  创建一个守护容器；退出后容器不停止运行

- –-name

  为创建的容器命名

3. 进入容器

```sh
docker exec -it 容器名称 /bin/bash
# eg:进入一个叫konga的容器
docker exec -it konga /bin/bash

# 在容器中执行了一个ping命令
ping 127.0.0.1
# 退出当前容器
exit
```

4. 启动容器

```sh
docker start 容器名称
```

5. 停止容器

```sh
docker stop 容器名称
```

6. 删除容器

```sh
docker rm 容器名称 

#删除所有容器
docker rm `docker ps -aq`
```

7. 查看容器详细信息

```
docker inspect 容器名称
```

**docker run 与docker start的区别**

> docker run 只在第一次运行时使用，将镜像放到容器中，以后再次启动这个容器时，只需要使用命令docker start
> 即可。docker run相当于执行了两步操作：将镜像放入容器中（docker
> create）,然后将容器启动，使之变成运行时容器（docker start）。而docker
> start的作用是，重新启动已存在的镜像。也就是说，如果使用这个命令，我们必须事先知道这个容器的ID，或者这个容器的名字，我们可以使用docker
> ps找到这个容器的信息。

镜像，容器、run和start

镜像就像安装包，run类似安装并启动了软件，此时软件运行在容器中，start类似启动了软件
