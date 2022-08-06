# 一、Docker简介

**Docker的架构：**

* **镜像(image)**：Docker镜像(lmage)就是一个只读的模板。镜像可以用来创建Docker容器，一个镜像可以创建很多容器

  ![](https://gitee.com/jobim/blogimage/raw/master/img/20211224173753.png)

* **容器(container)**：Docker利用容器(Container) 独立运行的一个或一组应用。**容器是用镜像创建的运行实例。**它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。

* **仓库(repository)**：仓库是**集中存放镜像**文件的场所。仓库分为公开仓库(**Public**) 和私有仓库(**Private**) 两种形式。最大的公开仓库是Docker Hub(https://hub.docker.com/)存放了数量庞大的镜像供用户下载。国内的公开仓库包括阿里云、网易云等

**Docker的架构图：**

![image-20220412212631487](https://blog.zhaobincode.cn/blogimages/202204122126574.png)



> docker官网： https://www.docker.com/
>
> docker中文网站: https://www.docker-cn.com/
>
> 官方文档：https://docs.docker.com/
>
> Docker Hub官网：https://hub.docker.com/



# 二、Docker安装

> 操作系统：Centos7.0
>
> 官方教程：https://docs.docker.com/engine/install/centos/

## 2.1 安装步骤

**1、卸载旧版本**

```
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

**2、安装所需的软件包**。yum-utils 提供了 yum-config-manager ，并且 device mapper 存储驱动程序需要 device-mapper-persistent-data 和 lvm2。

```
yum install -y yum-utils device-mapper-persistent-data lvm2
```

![image-20211224091336970](https://gitee.com/jobim/blogimage/raw/master/img/20211224091344.png)



**3、设置镜像仓库**（可以使用阿里云镜像安装）

```
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

**4、更新yum软件包索引**

```
yum makecache fast
```

![image-20211224091500011](https://gitee.com/jobim/blogimage/raw/master/img/20211224091500.png)

**5、安装Docker CE**

Docker有两个分支版本：Docker CE和Docker EE，即社区版和企业版，因为企业版需要官方授权，所以我们一般用社区版

```
yum -y install docker-ce
```

![image-20211224091617720](https://gitee.com/jobim/blogimage/raw/master/img/20211224091617.png)

**6、启动Docker**

```
systemctl start docker
```

![image-20211224091709402](https://gitee.com/jobim/blogimage/raw/master/img/20211224091709.png)

7、通过运行 hello-world 映像来验证是否正确安装了 Docker Engine-Community 

```
docker run hello-world
```

> run干了什么：
>
> ![image-20211228154411300](https://gitee.com/jobim/blogimage/raw/master/img/20211228154411.png)



## 2.2 卸载Docker

**1、停止Docker**

```
systemctl stop docker 
```

**2、删除安装包**

```
yum -y remove docker-ce
```

**2、删除镜像、容器、配置文件等内容**

```
rm -rf /var/lib/docker
```



## 2.3 配置阿里云镜像加速

**1、登陆阿里云**

> 阿里云镜像获取地址：https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors

![image-20211228153528298](https://gitee.com/jobim/blogimage/raw/master/img/20211228153528.png)

**2、配置镜像加速器**

![image-20211224111730064](https://gitee.com/jobim/blogimage/raw/master/img/20211224111730.png)



**3、检查加速器是否生效**

![image-20211228154034870](https://gitee.com/jobim/blogimage/raw/master/img/20211228154035.png)



# 三、Docker常用命令

## 3.1 帮助命令

```
docker Version

docker info

docker --help
```



## 3.2 镜像命令

**1、查看所有本地主机上的镜像**

```
docker images
                -a 列出本地所有的镜像(含中间映射层)
                -q 只显示镜像ID
                --digests 显示镜像的摘要信息
                --no-trunc 显示完整的镜像信息
```

![image-20211228155936007](https://gitee.com/jobim/blogimage/raw/master/img/20211228155936.png)

![image-20211228155814614](https://gitee.com/jobim/blogimage/raw/master/img/20211228155814.png)



**2、搜索镜像**

```
docker search [OPTIONS] 镜像名字

OPTIONS 说明:
    --filter,-f：基于给定条件过滤输出
    --format：使用模板格式化显示输出    
    --limit：Max number of search results ，默认值25
    --no-trunc：禁止截断输出
```

* 指定列出收藏数不小于指定值的镜像

  ```
  docker search -f stars=30 tomcat
  ```

  ![image-20211224145701156](https://gitee.com/jobim/blogimage/raw/master/img/20211224145701.png)

* 限制搜索输出个数

  ```sql
  docker search redis --limit 5
  ```

| **NAME**        | 镜像仓库源的名称                              |
| --------------- | --------------------------------------------- |
| **DESCRIPTION** | 镜像描述                                      |
| **STARS**       | 类似 Github 里面的 star，表示点赞、喜欢的数量 |
| **OFFICIAL**    | 是否为docker 官方发布的镜像                   |
| **AUTOMATED**   | 自动构建                                      |

**3、下载镜像**

```
docker pull 镜像名字[:TAG]，如果不写tag，默认是latest(最新版)
```

* **下载最新tomcat**

  ![image-20211224150329603](https://gitee.com/jobim/blogimage/raw/master/img/20211224150329.png)

* **下载Mysql5.7**

  ```
  docker pull mysql:5.7
  ```

**4、删除镜像**

```
删除指定镜像：docker rmi -f 镜像id
删除多个镜像：docker rmi -f 镜像id 镜像id 镜像id
删除全部镜像：docker rmi -f $(docker images -aq)
```

![image-20211228162505979](https://gitee.com/jobim/blogimage/raw/master/img/20211228162506.png)



**5、提交镜像**

docker commit 提交容器副本使之称为一个新的镜像

**`docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]`**

![image-20211230154423227](https://gitee.com/jobim/blogimage/raw/master/img/20211230154423.png)



## 3.3 容器命令

**1、新建并启动容器**

```
docker run [OPTIONS] IMAGE [COMMAND][ARG]
```

* **OPTIONS说明(常用)** :
  * --name="容器新名字":为容器指定一个名称;
  * -d:后台运行容器，并返回容器ID， 也即启动守护式容器;
  * -i:以交互模式运行容器，通常与-t同时使用;
  * -t:为容器重新分配一个伪输入终端，通常与-i同时使用;
  * -P:随机端口映射;
  * -p:指定端口映射，有以下四种格式
    * ip:hostPort:containerPort
    * ip::containerPort
    * hostPort:containerPort
    * containerPort

* 创建一个容器，使用镜像centos  ，容器命名为mycentos1228

![image-20211228171252431](https://gitee.com/jobim/blogimage/raw/master/img/20211228171252.png)



**2、列出当前所有正在运行的容器**

```
docker ps [OPTIONS]
```

* OPTIONS说明(常用) :
  * -a:列出当前所有正在运行的容器+历史上运行过的
  * -|:显示最近创建的容器。
  * -n:显示最近n个创建的容器。
  * -q:静默模式，只显示容器编号。
  * --no-trunc:不截断输出。

* 显示最近2个创建的容器

  ![image-20211228172911882](https://gitee.com/jobim/blogimage/raw/master/img/20211228172911.png)

**3、退出容器**

* `exit`，直接停止容器并退出

* `Crtl + Q + P` ，不停止容器退出

**4、删除容器**

* `docker rm 容器id` ，删除指定的容器，不能删除正在运行的容器，如果要强制删除 `rm -f`

* `docker rm -f $(docker ps -aq)`  ，删除全部容器

**5、启动和停止容器**

* `docker start 容器id ` ，启动容器

* `docker restart 容器id`，重启容器

* `docker stop 容器id` ，停止当前正在运行的容器

* `docker kill 容器id` ，强制停止容器





## 3.4 其他常用命令



**1、在后台启动容器**

```
docker run -d centos
```

注意：docker后台运行时，**必须要有一个前台进程，如果docker容器发现没有运行的应用，会自动停止**。

![image-20211229094604260](https://gitee.com/jobim/blogimage/raw/master/img/20211229094604.png)

**2、查看容器的日志**

```
docker logs -tf  --tail  n(每次显示日志的行数)  容器id
```

* -t 是加入时间戳

* -f 跟随最新的日志打印
* --tail 数字显示最后多少条

![image-20211229094536355](https://gitee.com/jobim/blogimage/raw/master/img/20211229094536.png)

**3、查看容器中的进程信息**

```
 docker top 容器id
```

![image-20211229094722180](https://gitee.com/jobim/blogimage/raw/master/img/20211229094722.png)

**4、查看镜像的元数据**

```
docker inspect  容器id
```

![image-20211229094734242](https://gitee.com/jobim/blogimage/raw/master/img/20211229094734.png)

**5、进入当前正在运行的容器**

* 在使用 -d 参数时，容器启动后会进入后台。此时想要进入容器，可以通过以下指令进入

* **`docker attach 容器id`** ，直接进入容器启动命令的终端，不会启动新的进程

* **`docker exec -it 容器ID bashShell`**，是在容器中打开新的终端，并且可以启动新的进程

**6、把容器内的文件拷贝到主机**

```
docker cp 容器id:容器内要拷贝的文件路径   拷贝到主机的路径
```

![image-20211229095154010](https://gitee.com/jobim/blogimage/raw/master/img/20211229095154.png)



# 四、Dockerfile

## 4.1 DockerFile简介

Dockerfile是用来构建Docker镜像的构建文件，是由一系列命令和参数构成的脚本。

**构建三步骤：**

- 编写Dockerfile文件：必须符合file规范
- docker build：`docker build -f /mydocker/Dockerfile -t mrlinxi/centos .` 通过docker build获得一个自定义的镜像

- docker run

文件长什么样？下面就是我们使用的centos的Dockerfile

```dockerfile
FROM scratch
ADD centos-7-x86_64-docker.tar.xz /

LABEL \
    org.label-schema.schema-version="1.0" \
    org.label-schema.name="CentOS Base Image" \
    org.label-schema.vendor="CentOS" \
    org.label-schema.license="GPLv2" \
    org.label-schema.build-date="20201113" \
    org.opencontainers.image.title="CentOS Base Image" \
    org.opencontainers.image.vendor="CentOS" \
    org.opencontainers.image.licenses="GPL-2.0-only" \
    org.opencontainers.image.created="2020-11-13 00:00:00+00:00"
# default cmd
CMD ["/bin/bash"]
```



**Dockerfile内容基础知识：**

- 每条**保留字指令**都必须**为大写字母**且后面要**跟随至少一个参数**
- 指令按照从上到下，顺序执行

- #表示注释
- 每条指令都会创建一个新的镜像层，并对镜像进行提交



**Docker执行Dockerfile的大致流程：**

- （1）docker从基础镜像运行一个容器
- （2）执行一条指令并对容器作出修改

- （3）执行类似docker commit的操作提交一个新的镜像层
- （4）docker再基于刚提交的镜像运行一个新容器
- （5）执行dockerfile中的下一条指令直到所有指令都执行完成



## 4.2 DockerFile体系结构(保留字指令)

**`FROM`**：基础镜像，当前新镜像是基于哪个镜像的。基于什么镜像进行修改；

**`MAINTAINER`**：镜像维护者的姓名和邮箱地址；

**`RUN`**：容器构建时需要运行的命令；

**`EXPOSE`**：当前容器对外暴露出的端口；

**`WORKDIR`**：指定在创建容器后，终端默认登陆的进来工作目录，一个落脚点，没写默认根目录`/`；

**`ENV`**：用来在构建镜像过程中设置环境变量；

* 例如：`ENV MY_PATH /usr/mytest` 这个环境变量可以在后续的任何RUN指令中使用，这就如同在命令前面指定了环境变量前缀一样；也可以在其它指令中直接使用这些环境变量。

* 比如：`WORKDIR $MY_PATH`

**`ADD`**：将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包；

**`COPY`**：类似ADD，拷贝文件和目录到镜像中。将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置；  `COPY src dest`   `COPY ["src", "dest"]`

* **ADD**跟**COPY**的区别在于ADD在复制后会自动解压缩和处理URL，而COPY仅仅进行复制。

**`VOLUME`**：容器数据卷，用于数据保存和持久化工作；

**`CMD`**：指一个容器启动时要运行的命令；Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换（后面案例会具体说明）

![img](https://cdn.nlark.com/yuque/0/2021/png/22423156/1638330806027-f7d631f5-99cd-4831-aea3-e9367ddf9d84.png)

**`ENTRYPOINT`**：指定一个容器启动时要运行的命令；ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数；

* **CMD**与**ENTRYPOINT**的区别是CMD存在多个时只有最后一个生效以及CMD会被docker run之后的参数替换；而ENTRYPOINT是追加命令。

**`ONBUILD`**：当构建一个被继承的Dockerfile时运行命令，父镜像在被子继承后父镜像的onbuild被触发



## 4.3 案例

### 4.3.1 Base镜像(scratch)

Docker Hub 中 99% 的镜像都是通过在 base 镜像中安装和配置需要的软件构建出来的。

### 4.3.2 自定义镜像mycentos

* Hub默认CentOS镜像什么情况：

  ![image-20211231101129310](https://gitee.com/jobim/blogimage/raw/master/img/20211231101129.png)

**自定义mycentos目的使我们自己的镜像具备如下：**

* 登陆后的默认路径
* vim编辑器
* 查看网络配置ifconfig支持

**1、编写自定义镜像的Dockerfile**

我们在宿主机的`/mydocker`文件夹下，新建一个Dockerfile：`vi Dockerfile`，写入下面的内容

```dockerfile
FROM centos
MAINTAINER mrlinxi<mrzhme@vip.qq.com>
 
ENV MYPATH /usr/local
WORKDIR $MYPATH
 
RUN yum -y install vim
RUN yum -y install net-tools
 
EXPOSE 80
 
CMD echo $MYPATH
CMD echo "success--------------ok"
CMD /bin/bash
```

**2、构建自定义镜像——docker build**

```shell
docker build -f /mydocker/Dockerfile -t mycentos:1.3 .
```

build语句最后面一个**`.`**表示当前目录。

![image-20211231103213531](https://gitee.com/jobim/blogimage/raw/master/img/20211231103213.png)

**3、运行自定义镜像——docker run**

`docker run -it mycentos:0.1`

![image-20211231103940646](https://gitee.com/jobim/blogimage/raw/master/img/20211231103940.png)

默认目录是/usr/locl，可以看到我们自己的新镜像已经支持vim/ifconfig命令，拓展成功

### 4.3.3 CMD/ENTRYPOINT 镜像案例

CMD/ENTRYPOINT都是指定一个容器启动时要运行的命令

**CMD镜像案例：**

Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换。

这里以tomcat为例，tomcat的dockerfile最后一句是`CMD ["catalina.sh", "run"]`

正常我们启动tomcat的命令是：`docker run -it -p 主机端口:8080 tomcat` 

现在我们执行这样一句命令：`docker run -it -p 8888:8080 tomcat ls -l`

这样就相当于在tomcat的dockerfile后面又加了一句`CMD ls -l`，因此会覆盖掉之前的语句。

![image-20211231104808236](https://gitee.com/jobim/blogimage/raw/master/img/20211231104808.png)

此时tomcat并没有运行，只是查看了默认路径下的文件。

![image-20211231104824570](https://gitee.com/jobim/blogimage/raw/master/img/20211231104824.png)

**ENTRYPOINT镜像案例：**

docker run 之后的参数会被当做参数传递给 ENTRYPOINT，之后形成新的命令组合



## 4.4 自定义镜像Tomcat9

**1、创建目录**

```shell
mkdir /mydocker/tomcat9
```

在该目录下新建touch.txt文件

**2、将jdk和tomcat安装的压缩包拷贝进上述目录**

![image-20211231105921264](https://gitee.com/jobim/blogimage/raw/master/img/20211231105921.png)

**3、在tomcat9目录下新建Dockerfile文件**

```dockerfile
FROM         centos
MAINTAINER    mrzhme<mrzhme@vip.qq.com>
#把宿主机当前上下文的c.txt拷贝到容器/usr/local/路径下
COPY c.txt /usr/local/cincontainer.txt
#把java与tomcat添加到容器中
ADD jdk-8u301-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-9.0.56.tar.gz /usr/local/
#安装vim编辑器
RUN yum -y install vim
#设置工作访问时候的WORKDIR路径，登录落脚点
ENV MYPATH /usr/local
WORKDIR $MYPATH
#配置java与tomcat环境变量
ENV JAVA_HOME /usr/local/jdk1.8.0_301
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.56
ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.56
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
#容器运行时监听的端口
EXPOSE  8080
#启动时运行tomcat
# ENTRYPOINT ["/usr/local/apache-tomcat-9.0.56/bin/startup.sh" ]
# CMD ["/usr/local/apache-tomcat-9.0.56/bin/catalina.sh","run"]
CMD /usr/local/apache-tomcat-9.0.56/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.56/bin/logs/catalina.out
```

![image-20211231111040509](https://gitee.com/jobim/blogimage/raw/master/img/20211231111040.png)



**4、构建镜像：`docker build -t zbtomcat9 .`**

**注意：**这里为什么没有加 -f 和 Dockerfile 的路径？Dockerfile的标准文件名就是Dockerfile，当当前目录下用于构建镜像的Dockerfile的文件名是标准文件名时，可以省略-f+路径。这时Docker会直接读取当前目录下名为Dockerfile的文件进行镜像的构建。

![image-20211231145514028](https://gitee.com/jobim/blogimage/raw/master/img/20211231145514.png)

**5、创建容器并启动**

```
docker run -d -p 9080:8080 --name myt9 -v /zzyyuse/mydockerfile/tomcat9/test:/usr/local/apache-tomcat-9.0.56/webapps/test -v /zzyyuse/mydockerfile/tomcat9/tomcat9logs/:/usr/local/apache-tomcat-9.0.56/logs --privileged=true zbtomcat9
```

![image-20211231150112551](https://gitee.com/jobim/blogimage/raw/master/img/20211231150112.png)

可以访问tomcat的界面

![image-20211231112348204](https://gitee.com/jobim/blogimage/raw/master/img/20211231112348.png)



**6、结合前述的容器卷将测试的web服务test发布**

```shell
cd /mydocker/tomcat9/test
mkdir WEB-INF  
```

* 在test目录下创建a.jsp文件，在WEB-INF下创建web.xml文件

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    id="WebApp_ID" version="2.5">
    
    <display-name>test</display-name>
   
  </web-app>
  ```

* a.jsp：

  ```html
  <%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
  <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
  <html>
    <head>
      <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
      <title>Insert title here</title>
    </head>
    <body>
      -----------welcome------------
      <%="i am in docker tomcat self "%>
      <br>
      <br>
      <% System.out.println("=============docker tomcat self");%>
    </body>
  </html>
  ```

  

  ![image-20211231173546269](https://gitee.com/jobim/blogimage/raw/master/img/20211231173546.png)

* 查看日志信息

  ![image-20211231153918479](https://gitee.com/jobim/blogimage/raw/master/img/20211231153918.png)





# 五、Docker常用安装

## 5.1 总体步骤

搜索镜像->拉取镜像->查看镜像->启动镜像->停止容器->移除容器

`docker search xxx` -> `docker pull xxx:TAG` -> `docker images xxx` -> `docker run [-itd -p port:port] [--name yyy] xxx:TAG` -> `docker stop 容器ID/yyy` -> `docker rm [-f] yyy`



## 7.2 安装mysql

```shell
docker pull mysql:5.7

docker run -p 3306:3306 --name mysql \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=10086 \
-d mysql:5.7
```

命令说明：

* `-p 12345:3306`：将主机的3306端口映射到docker容器的3306端口。

* `--name mysql`：运行服务名字

* `-v /mydata/mysql/conf:/etc/mysql` ：将主机/mydata/mysql/conf目录，挂载到容器的/etc/mysql目录

* `-v /mydata/mysql/data:/var/lib/mysql`：将主机的/mydata/mysql/data目录，挂载到容器的/var/lib/mysql目录
* `-v /mydata/mysql/log:/var/log/mysql`：将主机的/mydata/mysql/log目录，挂载到容器的/var/log/mysql目录
* `-e MYSQL_ROOT_PASSWORD=10086`：初始化 root 用户的密码。
* `-d mysql:5.7 `: 后台程序运行mysql5.7



![image-20211231175503486](https://gitee.com/jobim/blogimage/raw/master/img/20211231175503.png)



![image-20211231175840246](https://gitee.com/jobim/blogimage/raw/master/img/20211231175840.png)



## 7.3 安装redis 

**使用`docker pull 镜像名称`拉取镜像**

```javascript
docker pull redis //不指定版本号，默认拉取最新。
docker pull redis:6.0.8
```

**拉取完镜像后，使用docker images查看已经拉取的镜像**

```javascript
docker images
```

**在运行之前对redis进行一些配置**

* redis.conf的配置文件可以在 http://download.redis.io/redis-stable/redis.conf 上下载

  ```
  使用 mkdir /usr/local/docker 在宿主机上创建存放docker目录
  vi /usr/local/docker/redis.conf 在docker中创建redis的配置文件redis.conf
  将下载好的redis.conf文件替换或将内容复制到自己创建的配置文件中
  
  然后修改配置
  
  bind 127.0.0.1  //127.0.0.1 限制只能本机访问 将其改为0.0.0.0
  
  protected-mode no # 默认yes，开启保护模式，限制为本地访问
  
  daemonize no 默认no，改为yes意为以守护进程方式启动，yes会使配置文件方式启动redis失败（一开启就退出）
  ```

**运行指定镜像**

```javascript
docker run -itd -p 16379:6379 -v /mydocker/myredis/data:/data -v /mydocker/myredis/conf/redis.conf:/etc/redis/redis.conf  redis:latest redis-server /etc/redis/redis.conf --requirepass "password"

-d 以守护线程的方式运行（后台运行）
-i 以交互模式运行容器
-t 为容器重新分配一个伪输入终端 ，通常与 -i 同时使用；
-p 映射容器服务的 6379 端口到宿主机的 6379 端口。外部可以直接通过宿主机ip:6379 访问到 Redis 的服务。

-v /mydocker/myredis/data:/data  //把redis持久化的数据挂载到宿主机内，做数据备份

-v /mydocker/myredis/conf/redis.conf:/etc/redis/redis.conf //把宿主机配置好的redis.conf挂载到容器内的指定位置

redis-server /etc/redis/redis.conf  //使redis按照redis.conf的配置启动

--requirepass "password"

--appendonly yes //redis启动后数据持久化
```

![image-20220409134903387](https://gitee.com/jobim/blogimage/raw/master/img/202204091349475.png)



**测试redis-cli连接上来**

```
docker exec -it 1b52df719983 redis-cli
```

![image-20220409135106251](https://gitee.com/jobim/blogimage/raw/master/img/202204091351290.png)



参考博客：[Docker基础 · 语雀 ](https://www.yuque.com/mrlinxi/pxvr4g/polyyw#FW7dE)





