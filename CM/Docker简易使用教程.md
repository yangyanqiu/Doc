# Docker简易使用教程



## 1. 简介

Docker 是一个开源的应用容器引擎，基于 Go语言并遵从Apache 2.0协议开源。 

Docker 可以让开发者打包应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口, 更重要的是容器性能开销极低。一个完整的Docker由以下几个部分组成：

1. Docker Client客户端
2. Docker Daemon守护进程
3. Docker Image镜像
4. Docker Container容器

Docker 使用客户端-服务器 (C/S) 架构模式。Docker Daemon做为Server端，用户不能直接与Server端通信，Docker Client为用户提供一系列可智行的命令与之通信。在客户端通过镜像创建容器，容器与镜像的关系类似于面向对象编程中的对象与类，我们必须使用镜像创建容器，然后在容器中处理事务。

Docker可以帮助我们完成如下工作：

1. 标准化应用发布，docker容器包含了运行环境和可执行程序，可以跨平台和主机使用
2. 节约时间，快速部署和启动
3. 节约成本，以前一个虚拟机至少需要几个G的磁盘空间，docker容器可以减少到MB级
4. .....

## 2. Docker 安装

Docker CE(社区版)在Ubuntu上安装，要求Ubuntu是在64位以下版本：

- Bionic 18.04 (LTS)
- Xenial 16.04 (LTS)
- Trusty 14.04 (LTS)

 Docker CE(社区版)的安装有三种方法：使用远程仓库安装，下载安装包安装，用安装脚本安装；推荐使用第一种方法安装，接下来使用Ubuntu 16.04来讲解安装。

### Method 1. 使用远程仓库安装

第一次安装Docker CE要先在电脑上设置Docker仓库，然后才能安装。

1. 设置Docker仓库

   a. Update the `apt` package index:

   ```
   $ sudo apt-get update
   ```

   b. 安装一下包使apt用过HTTPS使用远程仓库

   ```
   $ sudo apt-get install \
       apt-transport-https \
       ca-certificates \
       curl \
       software-properties-common
   ```

   c. 添加Docker官方的GPG key

   ```
   $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   ```

    d. 验证你已经拥有指纹9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88的key
   ```
    $sudo apt-key fingerprint 0EBFCD88
   ```

   e. 设置稳定版本的远程仓库，不同的Ubuntu xxxxx需要设置不同arch的值：x86_64/amd64的arch=amd64，armhf 的arch=armhf，IBM Power (ppc64le)的arch=ppc64el，IBM Z (s390x)的arch=s390x，下面事例为x86_64/amd64的命令：

   ```
   $ sudo add-apt-repository \
      "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
      stable"
   ```

2. 安装Docker CE

   a. Update the `apt` package index.

   ```
   $ sudo apt-get update
   ```

   b. 安装Docker CE最新的稳定版本：

   ```
   $ sudo apt-get install docker-ce
   ```

   如果想指定安装某个版本，则使用下面命令安装：

   ```
   #列出仓库中的可用版本
   $ apt-cache madison docker-ce
   
   docker-ce | 18.09.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
   
   #安装指定版本，通过docker-ce=<VERSION>来指定版本，例如安装18.09.0~ce-0~ubuntu版本：
   $ sudo apt-get install docker-ce=18.09.0~ce-0~ubuntu
   ```

3. 测试安装成功

      ```
      $ docker -v
      Docker version 18.06.0-ce, build 0ffa825
      ```

4. 使用非root用户执行docker命令要加sudo，如果不想加sudo需要将自己使用的用户添加到docker组：

   ```
     sudo usermod -aG docker your-user
   ```

   **注意：添加user到docker组中，需要注销或者重启电脑后才生效。**

### Method 2.下载安装包安装

在官网网址 <https://download.docker.com/linux/ubuntu/dists/>的pool/stable中下载.deb安装包，根据自己的Ubuntu版本选择amd64, armhf, ppc64el或者s390x的安装包安装：

```
$ sudo dpkg -i /path/to/package.deb
```

### Method 3. 用安装脚本安装

使用Docker提供的安装脚本get-docker.sh进行安装，这脚本需要root或者sudo 执行，并且会安装最新的稳定版本：

1. 下载安装脚本

   ```
   $ curl -fsSL https://get.docker.com -o get-docker.sh
   ```

2. 赋予脚本可执行权限

   ```
   $ sudo chmod 755 get-docker.sh
   ```

3. 执行安装脚本，下载安装最新的Docker CE稳定版本

   ```
   $ sudo sh get-docker.sh
   ```
## 3. 在服务器上使用Docker

### Docker命令简易介绍

1. 使用ssh登录到192.168.3.7服务器

2. 要使用镜像创建容器，然后在容器中完成我们的编译等工作，可以把容器看做虚拟机，使用docker --help查看docker的用法。

3. 查看本地docker镜像

      ```
      $ docker images
      REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
      Android7.x-8.x/ubuntu16.04   v1.0                d3869011650e        6 days ago          871MB
      Android5.x-6.x/ubuntu14.04   v1.0                4f04bd3a57db        6 days ago          681MB
      Android2.3-4.4/ubuntu12.04   v1.0                b308ea3730a8        6 days ago          965MB
      
      ```

      各个选项说明:

      ​	REPOSITORY：表示镜像的仓库源
      ​	TAG：镜像的标签
      ​	IMAGE ID：镜像ID
      ​	CREATED：镜像创建时间
      ​	SIZE：镜像大小

      同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本，如ubuntu仓库源里，有15.10、14.04等多个不同的版本，我们使用 REPOSITORY:TAG 来定义不同的镜像。

4. 使用镜像运行容器

   第一次使用容器需要先创建，以后就可以直接运行容器了， 服务器192.168.3.7已经有3个可以直接使用的镜像，分别安装了不同的编译环境，分别用来创建不同编译环境的容器：

   ​	a. For Android 2.3(Gingerbread) to 4.4(KitKat):
   ```
   $ docker run --name <YourContainerName> -v 本地路径:容器路径 Android2.3-4.4/ubuntu12.04:v1.0
   ```

   说明:

   ​	--name:接容器名字，给自己的容器命名，方便以后使用

   ​	-v: 将本地路映射到容器的路径，把代码下载到服务器，映射到容器中编译

   ​	Android2.3-4.4/ubuntu12.04:用于创建容器的镜像名字，此镜像是ubuntu12.04的系统，安装了Sun Java development kit(JDK 1.6)

   ​	v1.0:是镜像的标签，相当于版本，镜像名字:标签可以用于标识一个镜像

   ​	b. For Android 5.x(Lollipop) to 6.x(Marshmallow):

    

   ```
   $ docker run --name <YourContainerName>  -ti Android5.x-6.x/ubuntu14.04:v1.0 /bin/bash
   ```

   ​	c. For Android 7.x(Nougat) to 8.x(Oreo)

   ​	

   ```
   $ docker run --name <YourContainerName> -ti Android7.x-8.x/ubuntu16.04:v1.0 /bin/bash
   ```

   使用ssh登录到192.168.3.7服务器

   要使用镜像创建容器，然后在容器中完成我们的编译等工作，可以把容器看做虚拟机，使用docker --help查看docker的用法。使用docker run --help查看使用说明

5. 进入容器

      ​	docker exec是进入容器最简单好用的方法:

      ```
      $ docker exec -t -i <YourContainerName> /bin/bash
      ```

      如果想要在容器中使用中文，使用docker exec进入容器时设置 LANG=C.UTF-8, 例：

      ​	docker exec -it myUbuntu env LANG=C.UTF-8 /bin/bash

6. 退出容器

   ```
   $ exit
   ```

7. 停止容器

   ```
   $ docker stop <YourContainerName>
   ```

8. 重启容器
     已经停止的容器，我们可以使用命令 docker start 来启动:

       ```
       $ docker start <YourContainerName>
       ```

9. 查看正在运行的容器

         $ docker ps
         CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS              PORTS                                                                                  NAMES
         af7f06d06224        svn:v1.0            "/root/startSvn.sh"   24 hours ago        Up 4 hours          0.0.0.0:83->80/tcp      
     CONTAINER ID，NAMES都可以用来标识一个容器。

     如果加选项-a, docker ps -a可以查看本地的所有容器。

      

10. 把容器提交成镜像

      把已有容器提交成镜像:

        ```
        $ docker commit -m "commit message" -a "author" containerID imageName:TAG
        ```

      -m: 提交说明   
      -a: 提交者
      containerID: 容器的ID，使用docker ps可以查看
      imageName:想要生成的镜像的名称
      TAG:要生成镜像的标签

11. 保存镜像

          ```
          $ docker save <imageName>:<TAG> > xxx.tar.gz
          ```
        
        将镜像imageName:TAG保存为文件xxx.tar.gz

12. 生成镜像
      ​    ```
      ​    $ docker commit -m "commit message" -a "author" containerID imageName:TAG
      ​    ```

        -m: 提交说明
        
        -a: 提交者
        
        containerID: 容器的ID，使用docker ps可以查看
        
        imageName:想要生成的镜像的名称
        
        TAG:要生成镜像的标签

13. 加载镜像

          ``` 
          $ docker load -i xxx.tar.gz
          ```
        
        加载了镜像xxx.tar.gz之后，可以使用docker images查看镜像

14. 服务器与容器之前拷贝文件

          a. 使用docker拷贝文件到容器中:
          
          ```
          $ docker cp file <YourContainerName>:<containerPath>
          ```
          
          b. 使用docker把容器中的文件拷贝到本地
          
          ```
          $ docker cp <YourContainerName>:<containerFile> <本地路径>
          ```

### 服务器中容器编译实例

此例子为账号yangyanqiu第一次登录到192.168.3.7服务器，下载ivi-android-4.4.3的代码，并在容器中编译 autolink_6dl-eng。

1. 使用自己账号登录的编译服务器

```
$ ssh yangyanqiu@192.168.3.7
```

2. 第一次登录，讲自己的账户添加到docker组，不添加每次使用docker命令都需要使用sudo

```
$sudo usermod -aG docker yangyanqiu
```
3. 退出重新登录进来使用docker命令就不用再加sudo

```
$ exit
$ ssh yangyanqiu@192.168.3.7
```
4. 把自己本地的公钥拷贝到服务器上自己的主目录

```
$scp -r xxx@192.xx.xx.xx:~/.ssh ~/
```

5. 挂载samba中我自己可用的目录到服务器，方便映射到docker容器拷贝存放自己需要的编译生成文件

```
$ mkdir MySambaShare
$ sudo mount -t cifs -o username=yangyanqiu,password=xxxxx,rw,uid=1960,gid=1001 //10.8.38.111/yangyanqiu MySambaShare
```

6. 下载代码ivi-android-4.4.3.git

```
$ mkdir -p code/android4.4
$ cd code/android4.4
$ git clone git@192.168.3.249:android-group/ivi-android-4.4.3.git
$ git checktout -b IMX6DL-KX63 origin/IMX6DL-KX63
```

7. 创建并运行用于自己编译android4.4的容器（如果容器已经创建运行容器使用命令：docker start 容器名）

```
#容器名字： yanqiu_android2.3-4.4
#映射路径/home/devel/yangyanqiu/MySambaShare， /home/devel/yangyanqiu/code/android4.4映射到容器中，以后每次start容器都会自动映射
$ docker run --name yanqiu_android2.3-4.4 -v /home/devel/yangyanqiu/MySambaShare:/root/samba -v /home/devel/yangyanqiu/code/android4.4/:/root/android4.4/ -dti Android2.3-4.4/ubuntu12.04:v1.0
```
8. 修改代码
   ....

9. 进入容器编译代码

```
#env LANG=C.UTF-8设置支持查看中文字符
$ docker exec -ti yanqiu_android2.3-4.4 env LANG=C.UTF-8 /bin/bash
$ cd /root/android4.4/ivi-android-4.4.3
$ source build/envsetup.sh
$ lunch autolink_6dl-eng
$ make installclean 
$ make 
```

10. 拷贝需要的编译生成文件到samba服务器，这样就可以在自己本地获取，使用

```
$ cp out/xx/xxx /root/samba/
```
