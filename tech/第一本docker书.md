## chapter 1

- docker推荐单个容器只运行一个应用程序或进程，这样就形成了一个分布式的应用程序模型。
- docker组件：docker客户端和服务器（即Docker引擎）、docker镜像、Registry、Docker容器

- 容器是基于镜像启动起来的，容器中可以运行一个或多个进程。镜像是docker生命周期中的构建或打包阶段，而容器则是启动和运行阶段。

## chapter 2



## chapter 3

- 除了创建上述交互式的容器，也可以创建长期运行的容器。守护式容器没有交互式会话，非常适合运行应用程序和服务。大多数时候我们都需要以守护式来运行我们的容器

## chapter 4

- 并不推荐使用docker commit 的方法构建镜像。相反，推荐使用被称为Dockerfile的定义文件和docker build命令来构建镜像。

- CMD命令

  >  Dockerfile中RUN命令是镜像创建时要执行的命令，而CMD是容器启动时要运行的命令。且docker run命令可以覆盖CMD命令
  >
  > 在Dockerfile中，只能指定一条CMD命令。如果指定了多条CMD命令，也只有最后一条CMD命令会被使用

- WORKDIR指令

  > WORKDIR指令用来在从镜像创建一个想新容器时，在容器内部设置一个工作目录，ENTRYPOINT和CMD指定的程序会在这个目录下执行。
  >
  > 我们可以使用该指令为Dockerfile中后续的一系列指令设置工作目录，也可以为最终的容器设置工作目录。

- ENV指令

  > ENV指令用来在镜像构建过程中设置环境变量

- VOLUME指令

  > VOLUME指令用来向基于镜像创建的容器添加卷。一个卷是可以存在于一个或多个容器内的特定的目录。
  >
  > - 卷可以在容器间共享和重用
  > - 对卷的修改不会对更新镜像产生影响

- ADD指令

  > ADD指令用来将构建环境下的文件或目录复制到镜像中
  >
  > 不能对构建目录或上下文之外的文件进行ADD操作
  >
  > 如果将一个归档文件（gzip、bzip2、xz）指定为源文件，Docker会自动将归档文件解开（unpack）。如下：
  >
  > ```shell
  > ADD latest.tar.gz /var/www/wordpress/
  > ```
  >
  > ADD指令会使得构建缓存变得无效

- COPY指令

  > 非常类似ADD，只是不会去做文件解压操作

- LABEL

  > LABEL指令用于为Docker镜像添加元数据。元数据以键值对的形式展现。
  >
  > 推荐将所有的元数据都放到一条LABEL指令中（以空格分隔），以防止不同的元数据指令创建过多的镜像层

## chapter 5

- docker networking可以将容器连接到不同宿主机上的容器，且不必事先创建容器再去连接它。同样，也不必关心容器的运行顺序，可以在网络内部通过容器名解析和发现

## 常用命令

```shell
######## 容器运行
# 查看正在运行的docker容器信息
sudo docker ps
# 查看所有docker容器信息
sudo docker ps -a
# 获取更详细的容器信息
sudo docker inspect frank_test
# 交互式方式创建一个ubuntu的docker容器，指定名称为frank_test
sudo docker run --name frank_test -i -t ubuntu /bin/bash
# 自动重启容器
# --restart=always不管什么原因退出，都会重启
# --restart=failure:5，当容器退出代码为非0时才重启，重试5次
sudo docker run --restart=always --name frank_test -i -t ubuntu /bin/bash
# 启动已经停止的容器
sudo docker start frank_test
# 附着到正在运行的容器上
sudo docker attach frank_test
# 将容器放到后台运行,-d参数指定
sudo docker run --name frank_test -d ubuntu /bin/bash

###### 日志
# 获取容器内部日志,也可以加-f参数，类似于tail -f，持续监控,-t为每条日志加上时间戳
sudo docker logs -ft frank_test
# 获取日志的最后10行数据
sudo docker logs --tail 10 frank_test

###### 查看容器运行状态
# 查看容器内运行的进程
sudo docker top frank_test
# 查看容器的统计信息，如CPU、磁盘占用情况，可以查看多个
sudo docker stats frank_test frank_test2 frank_test3

# 在容器中运行后台任务,新建/etc/new_config_file文件
sudo docker exec -d frank_test touch /etc/new_config_file
# 删除容器
sudo docker rm 容器id
# 删除所有容器,-a表示列出所有容器，-q标志则表示只返回容器id
# 注意： 后面的不是单引号，而是键盘左上角的 `
sudo docker rm `sudo docker ps -a -q`
# 删除docker镜像
sudo docker rmi image_id 

# Dockerfile构建,--no-cache无缓存，-t指定仓坤和镜像名。仓库名：dudet,镜像名：static_web，tag：v0.1,默认为latest
sudo docker build ./ --no-cache -t="dudet/static_web:v0.1"

# 查看docker容器服务端口（这里是80）对应的宿主机的端口
sudo docker port container_id 80

# 创建docker网络
sudo docker network create app
# 查看docker网络
sudo docker network inspect app
# 在docker网络中创建docker容器
sudo docker run -d --net=app --name redis dudet/redis
# 将已经运行的容器连接到app网络,redis容器名
sudo docker network connect app redis
# 从网络中断开一个容器
sudo docker network disconnect app redis
```

## 