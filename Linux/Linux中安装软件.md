

# CentOS 7 安装 jdk11

## 安装包准备

jdk11下载地址：[Java Downloads | Oracle](https://www.oracle.com/java/technologies/downloads/#java11)

下载后上传到服务器中即可：

![image-20220626114708429](https://blog.zhaobincode.cn/blogimages/202206261147595.png)

## 具体安装步骤

* **解压文件到指定目录：**

  ```bash
  tar -zxvf jdk-11.0.15.1_linux-x64_bin.tar.gz -C /opt/environment
  ```

  ![image-20220626115016845](https://blog.zhaobincode.cn/blogimages/202206261150875.png)

* **查看解压结果：**

  ![image-20220626115053749](https://blog.zhaobincode.cn/blogimages/202206261150798.png)

## 配置环境变量

```bash
vi /etc/profile
```

**将以下配置内容添加到文件最下方**

```bash
export JAVA_HOME=/opt/environment/jdk-11.0.15.1
export JRE_HOME=\$JAVA_HOME/
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

JAVA_HOME：环境变量它指向*jdk*的安装目录，*Eclipse/NetBeans/Tomcat*等软件就是通过搜索*JAVA_HOME*变量来找到并使用安装好的jdk

![image-20220626113510992](https://blog.zhaobincode.cn/blogimages/202206261135045.png)

* **使配置文件立即生效**

  ```bash
  source /etc/profile
  ```

* **验证是否成功：**

  ```bash
  java -version
  ```

  ![image-20220626115310334](https://blog.zhaobincode.cn/blogimages/202206261153376.png)







