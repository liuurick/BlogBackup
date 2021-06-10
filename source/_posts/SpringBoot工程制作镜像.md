---
title: SpringBoot工程制作镜像
date: 2021-01-06 17:07:17
tags: Docker
categories: [后端,分布式/微服务,虚拟化/容器化]
---



## 1.准备工作

### 1.1 准备 Docker

我这里以 CentOS7 为例来给大家演示。

首先需要在 CentOS7 上安装好 Docker，这个安装方式网上很多，我就不多说了，我自己去年写过一个 Docker 入门教程，大家可以在公众号后台回复 `Docker` 获取教程下载地址。

Docker 安装成功之后，我们首先需要修改 Docker 配置，开启允许远程访问 Docker 的功能，开启方式很简单，修改 `/usr/lib/systemd/system/docker.service` 文件，加入如下内容：

```
-H tcp://0.0.0.0:2375  -H unix:///var/run/docker.sock
```

如下图：

![image-20210529084409955](/images/2021052801.png)

配置完成后，保存退出，然后重启 Docker：

```bash
systemctl daemon-reload
service docker restart
```

Docker 重启成功之后，Docker 的准备工作就算是 OK 了。

1.2 准备 IDEA

IDEA 上的准备工作，主要是安装一个 Docker 插件，点击 `File->Settings->Plugins->Browse Repositories` 

安装完成之后就可以配置Docker了。

![image-20210529085916787](/images/2021052802.png)

## 2 通过jar包构建镜像

### 2.1 SpringBoot项目打成jar包

- 打包配置pom

  ```xml
  <build>
      <plugins>
          <plugin>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-maven-plugin</artifactId>
          </plugin>
      </plugins>
  </build>
  ```

- 使用maven打包并运行、访问工程

  ```shell
  maven install
  ```

### 2.2 创建Docker镜像

在linux上新建一个目录，将上一步的jar包拷贝到Linux服务器，准备创建镜像

```bash
cd /home
mkdir icoding
```

测试jar包是否可以运行，执行:java -jar

```bash
java -jar http-demo-1.0-SNAPSHOT.jar
```

![image-20210106165935824](/images/2021010606.png)

访问：http://192.168.60.129:10000/get/user/1

在`http-demo-1.0-SNAPSHOT.jar`所在文件夹位置编写`Dockerfile`文件

```shell
vim Dockerfile

FROM java:8
#VOLUME 指定了临时文件目录/tmp
#其效果是在主机 /var/lib/docker 目录下创建了一个临时文件，并链接到容器的/tmp
VOLUME /tmp
# 将jar包添加到容器中并更名为app.jar
ADD http-demo-0.0.1-SNAPSHOT.jar app.jar
#运行jar包
RUN bash -c 'touch /app.jar'
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

**注释：**

- **FROM**：表示已JDK8为基础镜像制作docker镜像
- **VOLUME**：表示创建 /tmp 目录，并持久化到 Docker 数据文件夹。创建 /tmp 是因为SpringBoot内嵌的Tomcat容器默认使用 /tmp 作为工作目录
- **ADD**：将容器外的 demo-0.0.1-SNAPSHOT.jar 拷贝到容器中，并重命名为 demo.jar
- **RUN**：RUN表示运行后面跟着的命令行命令，-c 表示将后面的内容作为一个字符串来统一执行，bash容器执行 -c 后面的命令，在 / 目录下创建一个 demo.jar 文件。需要注意的是，这里的 demo.jar 要和上一条ADD命令后面的 demo.jar 命名一致，表示将上一条命令添加到容器里的文件，创建在 / 目录下
- **ENTRYPOINT**：容器启动时运行的命令，相当于我们在命令行中输入java -jar xxxx.jar，为了缩短 Tomcat 的启
  动时间，添加 java.security.egd 的系统属性指向 /dev/urandom 作为 ENTRYPOINT



在Dockerfile文件所在目录创建镜像

```shell
docker build -t http-demo-0.0.1-snapshot .
```

**注释：**

- build：表示制作镜像
- -t：表示给镜像打个标签，相当于 `docker tag 镜像ID 新镜像名:版本号`
-  . ：表示jar包和Dockerfile文件所在位置，如果是当前目录下，也可以用 . 表示



查看镜像

```shell
docker images
```

![image-20210106172228684](/images/2021010607.png)

再次运行项目：

```shell
docker run -d -p 10000:10000 http-demo-0.0.1-snapshot
```

### 2.3 创建启动容器

### 2.4访问界面

### 2.5 停止与删除

## 3 使用maven构建镜像

第二种方法通过maven的`docker-maven-plugin`插件可完成从打包到构建镜像，构建容器等过程

### 3.1 编写pom_docker.xml

```xml
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>1.0.0</version>
    <!--docker镜像相关的配置信息-->
    <configuration>
        <!--镜像名，这里使用工程名-->
        <imageName>${project.artifactId}</imageName>
        <!--Dockerfile文件所在目录-->
        <dockerDirectory>${project.basedir}/src/main/resources</dockerDirectory> <!-- 指定 Dockerfile 路径-->
        <!--TAG，这里用工程版本号-->
        <imageTags>
            <imageTag>${project.version}</imageTag>
        </imageTags>
        <!--构建镜像的配置信息-->
        <resources>
            <resource>
                <targetPath>/</targetPath>
                <directory>${project.build.directory}</directory>
                <include>${project.artifactId}-${project.version}.jar</include>
            </resource>
        </resources>
    </configuration>
</plugin>
```

或者：

```xml
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>1.0.0</version>
    <executions>
        <execution>
            <id>build-image</id>
            <phase>package</phase>
            <goals>
                <goal>build</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <imageName>mall-tiny/${project.artifactId}:${project.version}</imageName>
        <dockerHost>http://192.168.3.101:2375</dockerHost>
        <baseImage>java:8</baseImage>
        <entryPoint>["java", "-jar","/${project.build.finalName}.jar"]</entryPoint>
        <resources>
            <resource>
                <targetPath>/</targetPath>
                <directory>${project.build.directory}</directory>                    <include>${project.build.finalName}.jar</include>
            </resource>
        </resources>
    </configuration>
</plugin>
```

**注意：**

- executions.execution.phase:此处配置了在maven打包应用时构建docker镜像；
- imageName：用于指定镜像名称，mall-tiny是仓库名称，${project.artifactId}为镜像名称，${project.version}为镜像版本号；
- dockerHost：打包后上传到的docker服务器地址；
- baseImage：该应用所依赖的基础镜像，此处为java；
- entryPoint：docker容器启动时执行的命令；
- resources.resource.targetPath：将打包后的资源文件复制到该目录；
- resources.resource.directory：需要复制的文件所在目录，maven打包的应用jar包保存在target目录下面；
- resources.resource.include：需要复制的文件，打包好的应用jar包。

### 3.2 拷贝Dockerfile文件

### 3.3 在IDEA中提交修改的文件

### 3.4 clone最新项目

```bash
git clone http://192.168.60.129:8888/root/http-demo.git
```

### 3.5 打包构建镜像

```shell
#进入工程目录
cd http-demo
# 打包构建镜像
mvn -f pom.xml clean package -DeskipTests docker:build
```

![image-20210106181643885](/images/2021010608.png)

**注意：**

- 建议还是一个pom文件吧

- 这里需要安装一下maven

```bash
yum install maven -y
```

![image-20210106175745867](/images/2021010609.png)

### 3.6 创建启动容器

基于http-demo:1.0-SNAPSHOT镜像创建容器，容器名称为http-demo

```shell
docker run -d -p 10000:10000 http-demo:1.0-SNAPSHOT
```

容器创建成功，可以通过`docker ps -a`命令查看

### 3.7 访问界面

