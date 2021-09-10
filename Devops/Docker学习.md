# Docker学习



## Docker安装

```powershell
# delete old env
sudo apt-get remove docker docker-engine docker.io containerd runc

# update apt-get
sudo apt-get update

# install
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
    
# Add Docker’s official GPG key     
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg    

# Use the following command to set up the stable repository. To add the nightly or test repository, add the word nightly or test (or both) after the word stable in the commands below. 
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
#install docker engine
sudo apt-get install docker-ce docker-ce-cli containerd.io

# for wsl (exception: Is the docker daemon running?)
sudo service docker start

# test
sudo docker run hello-world
```



## 帮助命令

```shell
docker version			# show docker version
docker info				# show all information
docker <COMMAND> --help	 # show help doc
```



## 镜像命令

-  **docker images**  # show all docker images

```shell
root@giannischen-virtual-machine:/home/giannischen# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d1165f221234   4 months ago   13.3kB

REPOSITORY	image仓库源
TAG		    image标签
IMAGE ID     image ID
CREATED      image创建时间
SIZE		image大小
```

- **docker search**  # search docker hub for an image

```shell
root@giannischen-virtual-machine:/home/giannischen# docker search mysql
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   11134     [OK]       
mariadb                           MariaDB Server is a high performing open sou…   4219      [OK]       
```

- **docker pull**  # pull container

```shell
docker pull container:tag
```

- **docker rmi**  # remove image(s)

```shell
root@giannischen-virtual-machine:/home/giannischen# docker search mysql
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   11134     [OK]       
mariadb                           MariaDB Server is a high performing open sou…   4219      [OK]       
```



## 容器命令

- **docker run [args] image-name**  # run image as container

```shell
docker run [] image

#参数说明
--name="Name"		# 容器名字，区分多个image派生的容器
-d				   # 后台运行
-p				   # 指定端口 8080:8089  -P随机指定端口 多端口映射(-p ... -p ...)
		-p 主机端口:容器端口
		-p ip:主机端口:容器端口
-it				   # 交互模式运行
```

- **docker ps**  # list all running container

```shell
docker ps
-a		# 当前运行+历史运行过的容器
-n=?	# 显示最近创建过的容器
-q		# 只显示ID
```

- **start and stop**

```shell
docker start <container-id>
docker stop <container-id>
docker restart <container-id>
docker kill <container-id>
```



## 其他命令

- **查看 log 记录**

```shell
docker logs
```

- **查看容器中的进程**

```shell
docker top
```

- **查看image的元数据**

```shell
docker inspect
```

- **进入当前正在运行的容器**

```
docker exec -it container-ID bashShell 

docker attach container-ID
```

- **从容器拷贝文件到主机上**

```shell
docker cp container-ID:dir host-dir
```



## 可视化

### portainer

```shell
docker run -d -p 8088:9000 \
--restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```

### Rancher

```shell
# to be conyinued...
```



## commit镜像

```shell
docker commit  # 提交容器成为一个新的副本

# same as git
docker commit -m="commit tags" -a="author" container-ID image-name:[TAG]
```



## 容器数据卷

- **目的（持久化）：**

  - 容器之间数据共享；
  - 数据本地化；

- **思路：**

  将容器的目录挂载到物理机的目录上；

### 使用数据卷

- 方式一：**直接使用命令来挂载**	-v

  ```shell
  docker run -it -v 主机目录 : 容器内目录
  
  # 查看挂载
  docker inspect container-ID		# 结果如下
  ```

  ![image-20210717165554131](C:\Users\giannischen\AppData\Roaming\Typora\typora-user-images\image-20210717165554131.png)



### 具名挂载和匿名挂载

-  具名

  ```shell
  docker run -it -P --name container-name -v name:dir image-name
  ```

- 匿名

  ```shell
  docker run -it -P --name container-name -v container-dir image-name
  ```

- 上述两种挂载方法默认路径：

  ```shell
  /var/lib/docker/volumes/xxx/_data
  ```

  通过具名挂载方便查找数据卷；

- 拓展

  ```shell
  # ro -- read only 只有宿主机可操作！
  docker run -it -P --name container-name -v name:dir:ro image-name
  # rw -- read and write
  docker run -it -P --name container-name -v name:dir:rw image-name
  ```

  

## DockerFile

- 是命令参数脚本，详见Dockerfile和官方文档；

- 通过脚本生成镜像；

- 每个指令都会创建并提交一个新的镜像层；

  ![image-20210719210845296](C:\Users\giannischen\AppData\Roaming\Typora\typora-user-images\image-20210719210845296.png)

1. 编写脚本：

```shell
# my own centos
FROM centos

VOLUME ["volume01","volume02"]

CMD echo "---------------end---------------"
CMD /bin/bash
```

2. 紧接着生成镜像（仍然是匿名挂载）：

```shell
docker build -f /shell-absolute-dir -t owner/image-name:tag-number .
```

3. 这里 有个人小坑，设置的挂在路径为相对路径似乎变得不太好使了，需要将相对路径改为绝对路径才能成功启动自定义的 image；看起来像这样：

```shell
# my own centos
FROM centos

VOLUME ["/volume01","/volume02"]

CMD echo "---------------end---------------"
CMD /bin/bash
```

​	报错内容为：

```shell
docker: Error response from daemon: OCI runtime create failed: invalid mount {Destination:volume01 Type:bind Source:/var/lib/docker/volumes/1c0b7544460e5916ba609bb66b033e8e81589e1f306356759815dc9a925b7400/_data Options:[rbind]}: mount destination volume01 not absolute: unknown.
ERRO[0000] error waiting for container: context canceled 
```

### 数据卷容器

- 目标：多个容器共享数据；

```shell
# 通过--volumes-from 完成
docker run -it --name centos02 --volumes-from centos01 gc/centos:0.1
```

​	拷贝思想，而非数据共享访问，停止 `centos01` 不会影响 `centos02` ；

### DockerFile指令

![DockerFileCommand](D:\Photo\DockerFileCommand.png)

<img src="D:\Photo\images.png" alt="images" style="zoom:150%;" />

```dockerfile
FROM 				# 基础镜像 e.g. centos
MAINTAINER 		 	 # 镜像的编写者，姓名 + 邮箱
RUN 				# 镜像构建的时候需要运行的命令
ADD					# 步骤：例如田间tomcat镜像
WORKDIR				# 镜像工作目录，e.g. /bin/bash
VOLUME				# 挂载目录
EXPOSE				# 需要暴露的端口
CMD					# 指定容器启动时候要执行的命令，只有最新的一个CMD会执行
ENTRYPOINT			 # 指定容器启动时候要执行的命令，可追加命令
ONBUILD				 # 当构建一个被继承 DockerFile，触发指令
COPY				# 类似ADD，将文件拷贝到镜像中
ENV					# 构建时设置环境变量
```

### 实战：CentOS

- 构建一个自己的 `centos` 

```dockerfile
FROM centos
MAINTAINER giannischen<1065619715@qq.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "----end----"
CMD /bin/bash
```

### 实战：Tomcat镜像

1. 准备镜像文件：

![image-20210728104229398](C:\Users\giannischen\AppData\Roaming\Typora\typora-user-images\image-20210728104229398.png)

2. 编写`dockerfile`文件

```dockerfile
FROM centos
MAINTAINER GiannisChen<1065619715@qq.com>

COPY readme.txt /usr/local/readme.txt

ADD /home/download/jdk-8u301-linux-x64.tar.gz /usr/local/
ADD /home/download/apache-tomcat-9.0.50.tar.gz /usr/local/

RUN yum -y install vim

ENV MYPATH /usr/local/

WORKDIR $MYPATH

ENV JAVA_HOME /usr/local/jdk1.8.0_301
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.50
ENV CATALINA_BASH /usr/local/apache-tomcat-9.0.50
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

EXPOSE 8080

CMD /usr/local/apache-tomcat-9.0.50/bin/startup.sh && tail -F/usr/local/apache-tomcat-9.0.50/bin/logs/catalina.out
```

3. `build`

```shell
docker build -t mytomcat .
```

4. 启动并挂载数据卷

```shell
docker run -d -p 9091:8080 --name testtomcat -v /home/myTomcat/test:/usr/local/apache-tomcat-9.0.50/webapps/test -v /home/myTomcat/logs/:/usr/local/apache-tomcat-9.0.50/logs mytomcat
```

5. 访问测试（经典tomcat）
6. 发布项目（本地 -> docker 数据卷）

###### web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.4" 
    xmlns="http://java.sun.com/xml/ns/j2ee" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee 
        http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
</web-app>
```

###### index.html

```html
<html>
	<head>
		<title>first page</title>
	</head>
	<body>
		<h1>hello world!</h1>
		content
		<p>ppppp</p>
		<div>
			<p>pp</p>
		</div>
		<ul>
			<li>hh1</li>
			<li>hh2</li>
		</ul>
	</body>
</html>
```

###### 效果

![image-20210728192347058](C:\Users\giannischen\AppData\Roaming\Typora\typora-user-images\image-20210728192347058.png)

### 发布自己的Docker镜像

1. 登录Docker账号

```shell
docker login --help

Usage:  docker login [OPTIONS] [SERVER]

Log in to a Docker registry.
If no server is specified, the default is defined by the daemon.

Options:
  -p, --password string   Password
      --password-stdin    Take the password from stdin
  -u, --username string   Username
```

2. push

```shell
docker push giannischen/image:tags
```

- 注意：`push`需要带`namespace`，不然会报错！
- 解决方法：修改`tag`

```shell
docker tag old-image-name new-image-name
```

## Docker网络



## Dockers Compose  



## Docker Swarm



## CI/CD jenkins 流水线



## TODO：阿里云镜像加速和存储

- todo：P7、P32

