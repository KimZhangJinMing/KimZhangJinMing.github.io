[TOC]



### 1. Docker简介

Docker是一种虚拟化技术，类比于虚拟机。虚拟机需要模拟硬件还有很多不常用的软件等，而Docker是共用宿主机的内核，需要什么软件就引入相应的容器，而且各个容器之间是隔离的，互不影响也能通信。`Dockre是比虚拟机更加轻量的虚拟技术。`



#### 1.1 基本概念

> 镜像image

Docker镜像就好比是一个模板，可以通过这个模板来创建容器服务(可以创建多个容器)。

比如，tomcat镜像 -> run -> tomcat容器（提供服务）。

> 容器Container

Docker利用容器技术，独立运行一个或一组应用。

容器是通过镜像来创建的。

> 仓库Repository

仓库就是存放镜像的地方。分为公有仓库和私有仓库。

DockerHub(国外)，阿里云(国内，配置镜像加速)



#### 1.2 安装Docker

1. 查看环境

查看Linux版本，系统内核要求3.10以上：

```shell
uname -a
```

```json
Linux ming 3.10.0-514.26.2.el7.x86_64 #1 SMP Tue Jul 4 15:04:05 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
```

查看系统信息：

```shell
cat /etc/os-release
```

```json
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"
```

2. 安装Docker

> 如果已经安装过docker，卸载已经安装的版本

```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

> 设置Docker的仓库

```shell
# 1. 下载yum-utils(其中包含了yum-config-manager)
sudo yum install -y yum-utils
# 2. 设置阿里的docker仓库地址
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

> 安装Docker

```shell
# 清除缓存
yum makecache fast
# 下载docker
sudo yum install docker-ce docker-ce-cli containerd.io
```

> 查看Docker版本信息

```shell
docker version
```

出现以下结果，说明Docker安装成功。

```shell
Client: Docker Engine - Community
 Version:           20.10.1
 API version:       1.41
 Go version:        go1.13.15
 Git commit:        831ebea
 Built:             Tue Dec 15 04:37:17 2020
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true
```

> 启动docker

```shell
sudo systemctl start docker
```

> 运行hello-world示例镜像

```shell
sudo docker run hello-world
```

![image-docker hello-world](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210101205537190.png)



#### 1.3 阿里云镜像加速器

登录阿里云，找到`容器镜像服务`，镜像加速器。

![image-阿里云镜像加速器](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210101213032540.png)



#### 1.4 docerk run 流程

![image-docker run运行流程](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210101214614552.png)



#### 1.5 底层原理

> Docker是怎么工作的？

Docker是一个Client-Server结构的系统，Docker的守护进程运行在服务器上，客户端通过Socket访问，

Docker Server接收到Docker Client的指令，就会执行这个命令。

![image-Docker工作原理](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210101213937951.png)



### 2.镜像的基本命令

[官方参考文档](https://docs.docker.com/engine/reference/commandline/docker/)

```shell
# 显示docker版本
docker version
# 显示docker的系统信息，包括镜像和容器的数量
docker info
# 帮助命令
docker 命令 --help
```

#### 2.1 查看镜像

```shell
# 1. 查看所有镜像
docker images -a
# 2. 查看镜像的id
docker images -q
```

#### 2.2 搜索镜像

搜索镜像可以使用DockerHub搜索，也可以使用命令进行搜索。

```shell
# 1. 搜索mysql的镜像
docker search mysql
# 2. 搜索stars大于9000的本版
docker search mysql --filter=stars=9000
```

#### 2.3 下载镜像

```shell
# 1. 不指定版本，默认下载最新版本latest
docker pull mysql
# 2. 下载指定版本，这里的版本号一定要是DockerHub上有的版本
docker pull mysql:5.7
```

既然知道了真实地址，docker pull mysql:5.7 相当于 docker pull docker.io/library/mysql:5.7

![image-docker pull mysql:5.7](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210101225333094.png)



#### 2.4 删除镜像

```shell
# 1.通过镜像名称删除，-f 强制删除
docker rmi -f mysql
# 2. 通过id删除
docker rmi -f f07dfa83b528
# 3. 删除所有的镜像
docker rmi -f $(docker images -aq)
```

![image-删除镜像](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210101225852498.png)



### 3.容器的基本命令

#### 3.1 创建容器并启动

```shell
# 1. 以交互模式启动centos容器，交互使用bash命令行
docker run -it centos /bin/bash
# 也可以使用images id 启动容器
docker run -it 300e315adb2f /bin/bash

# 参数解释
--name=""  # 指定容器名字
-d         # 后台方式运行
-it        # 使用交互方式运行，进入容器查看内容
-p         # 指定容器的端口，有以下几种形式
	- 主机端口：容器端口（常用）
	- ip:主机端口：容器端口
	- 容器端口
-P         # 随机指定端口

```

```shell
# 进入容器后,主机名称变成了容器id
[root@8434890d1132 /]# 

[root@ming local]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
41a27a712b32   centos    "/bin/bash"   14 seconds ago   Up 13 seconds             funny_wright

```



#### 3.2 查看容器

```shell
# 1. 查看正在运行的容器
docker ps 
# 2. 查看正在运行的容器和历史运行的容器
docker ps -a

# 参数解释
-n=1 # 显示最近创建的容器
-q   # 只显示容器id
```

#### 3.3 退出容器

```shell
# 1. 退出容器并关闭
[root@8434890d1132 /]# exit
# 2. 退出容器，不关闭
CTRL + P + Q 
```

#### 3.4 删除容器

删除容器使用rm，删除镜像使用rmi。(i = image)

```shell
# 1. 指定容器id删除
docker rm -f 41a27a712b32
# 2. 删除所有容器，指定参数-f强制删除，如果容器正在运行，也能删除
docker rm -f $(docker ps -aq)
# 3. 删除所有容器，如果容器正在运行，会报错
docker ps -aq|xargs docker rm
```

#### 3.5 启动和停止容器

```shell
# 1. 启动容器
docker start 容器id
# 2. 重启容器
docker restart 容器id
# 3. 停止容器
docker stop 容器id
# 4. 容器在运行，可以使用kill强制停止容器
docker kill 容器id
```



### 4.常用其他命令

#### 4.1 后台启动容器

```shell
docker run -d 镜像名称
```

```shell
# 问题：后台方式启动容器后,docker ps查看容器,发现刚启动的容器停止了?
docker容器使用后台运行，就必须要有一个前台进程，docker发现没有应用，就会自动停止.
```

#### 4.2 查看日志

```shell
# 查看容器7194f420dccf最近10行的日志信息
docker logs -tf --tail 10 7194f420dccf

# 参数解释
-f # 显示最新的日志
-t # 显示时间戳
-tail n # 显示最后n行
```

为了方便查看日志，启动容器的时候让它运行脚本：

```shell
# 后台方式启动，每隔1s输出hello,world，方便查看日志信息
docker run -d centos /bin/sh -c "while true;do echo hello,world;sleep 1;done"

[root@ming local]# docker logs -tf --tail 10 7194f420dccf 
2021-01-02T04:59:49.064172640Z hello,world
2021-01-02T04:59:50.065917036Z hello,world
2021-01-02T04:59:51.067497230Z hello,world
2021-01-02T04:59:52.069279085Z hello,world
2021-01-02T04:59:53.070906928Z hello,world
2021-01-02T04:59:54.072698176Z hello,world
2021-01-02T04:59:55.074312881Z hello,world
2021-01-02T04:59:56.075940314Z hello,world
2021-01-02T04:59:57.077579824Z hello,world
2021-01-02T04:59:58.083724580Z hello,world
2021-01-02T04:59:59.085284061Z hello,world
2021-01-02T05:00:00.086883423Z hello,world
2021-01-02T05:00:01.088530900Z hello,world
```

#### 4.3 查看容器中进程的信息

```shell
# 1. 查看容器中进程信息
docker top 7194f420dccf
```

#### 4.4 查看镜像的元数据

```shell
docker inspect 容器id
```

```json
[
    {	
        # 容器的完整id，默认只是截取前面几位
        "Id": "7194f420dccf83fc219103ba4b8fe704583cbe52a8e0ad53bc5ee7c788ece631",
        "Created": "2021-01-02T04:54:22.996950933Z", # 容器创建时间
        "Path": "/bin/sh",
        "Args": [ # 启动容器时使用的参数
            "-c",
            "while true;do echo hello,world;sleep 1;done"
        ],
        "State": { # 容器状态
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 19341,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-01-02T04:54:23.420056968Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55", # 镜像信息
        "ResolvConfPath": "/var/lib/docker/containers/7194f420dccf83fc219103ba4b8fe704583cbe52a8e0ad53bc5ee7c788ece631/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/7194f420dccf83fc219103ba4b8fe704583cbe52a8e0ad53bc5ee7c788ece631/hostname",
        "HostsPath": "/var/lib/docker/containers/7194f420dccf83fc219103ba4b8fe704583cbe52a8e0ad53bc5ee7c788ece631/hosts",
        "LogPath": "/var/lib/docker/containers/7194f420dccf83fc219103ba4b8fe704583cbe52a8e0ad53bc5ee7c788ece631/7194f420dccf83fc219103ba4b8fe704583cbe52a8e0ad53bc5ee7c788ece631-json.log",
        "Name": "/quizzical_brown",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "host",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/d76624dcc42355cece3b8a39ba99dcf0b1714c80f5c28fc7ff80cf778a635932-init/diff:/var/lib/docker/overlay2/d5c1df794c28e4c26a5118b7df6f110d8cd67fd4b2184e3f0509513bde2686ac/diff",
                "MergedDir": "/var/lib/docker/overlay2/d76624dcc42355cece3b8a39ba99dcf0b1714c80f5c28fc7ff80cf778a635932/merged",
                "UpperDir": "/var/lib/docker/overlay2/d76624dcc42355cece3b8a39ba99dcf0b1714c80f5c28fc7ff80cf778a635932/diff",
                "WorkDir": "/var/lib/docker/overlay2/d76624dcc42355cece3b8a39ba99dcf0b1714c80f5c28fc7ff80cf778a635932/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "7194f420dccf",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "while true;do echo hello,world;sleep 1;done"
            ],
            "Image": "centos",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20201204",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "ea030739e7cabd90101a0040969de447c6ba176267e502f5a5558b19f2a4d813",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/ea030739e7ca",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "2cef83333f52c17f883782c62909d4878761bb98238aa45939039609e2d1ccfc",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "1cd4dcbb6871cbcce8292729579d2a62dee582c9b7cd3810298c1d69bc1de4cb",
                    "EndpointID": "2cef83333f52c17f883782c62909d4878761bb98238aa45939039609e2d1ccfc",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

#### 4.5 进入当前正在运行的容器

```shell
# 1. 进入当前正在运行的容器进行操作，不会启动新的进程
docker attach 4db8b5f683c1
# 2. 进入当前正在运行的容器进行操作，会重新启动一个进程
docker exec -it 4db8b5f683c1 /bin/bash
```

虽然是不同的进程进行操作，但容器内的资源还是共享的。

首先使用docker exec -it 4db8b5f683c1 /bin/bash进入容器：

![image-docker attach](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210102132444427.png)

然后使用docker attach 4db8b5f683c1进入容器:并没有启动一个新的/bin/bash进程。

![image-docker exec](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210102132515791.png)

#### 4.6 从容器内拷贝文件到主机上

```shell
# 即使容器当前不在运行，也可以拷贝出文件
docker cp 容器id:容器内路径  当前主机路径
```

```shell
[root@ming local]# docker cp 4db8b5f683c1:/aaa.txt /home/  # 将容器内/aaa.txt 拷贝到当前主机 /home/ 目录下
[root@ming local]# ls /home
aaa.txt
```



### 5.命令小结

![docker命令图谱](https://upload-images.jianshu.io/upload_images/12842279-f5d4c22882f4a649.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)



### 6. 小实战

#### 6.1 部署nginx

```shell
# 1. 下载nginx
docker pull nginx

# 2. 查看镜像
[root@ming ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        latest    ae2feff98a0c   2 weeks ago   133MB
centos       latest    300e315adb2f   3 weeks ago   209MB

# 3. 启动容器，将主机的3344端口映射到容器内的80端口
[root@ming ~]# docker run -p 3344:80 -it nginx  /bin/bash
root@6b29c7751037:/# 

# 4.查看nginx安装目录，版本信息
root@6b29c7751037:/# whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx
root@6b29c7751037:/# cd /usr/sbin
root@6b29c7751037:/usr/sbin# ./nginx -version
nginx version: nginx/1.19.6

# 5.启动nginx
root@6b29c7751037:/usr/sbin# ./nginx

```

外网访问[http://47.96.224.198:3344/]()，容器内启动nginx成功。

![image-docker部署nginx](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210102143249340.png)

#### 6.2 部署tomcat 

```shell
# 1. 查看镜像
[root@ming ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
tomcat       latest    feba8d001e3f   2 weeks ago   649MB
nginx        latest    ae2feff98a0c   2 weeks ago   133MB
centos       latest    300e315adb2f   3 weeks ago   209MB

# 2. 启动容器，将主机的3355端口映射到容器内的80端口
[root@ming ~]# docker run -p 3355:8080 -it tomcat /bin/bash

# 3. 启动tomcat
root@5e2e3cb50a29:/usr/local/tomcat/bin# ./startup.sh 

# 4. 查看进程，tomcat已经启动
root@5e2e3cb50a29:/usr/local/tomcat/bin# ps -ef|grep tomcat
root        15     1  1 07:45 pts/0    00:00:02 /usr/local/openjdk-11/bin/java -Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dignore.endorsed.dirs= -classpath /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/tomcat -Dcatalina.home=/usr/local/tomcat -Djava.io.tmpdir=/usr/local/tomcat/temp org.apache.catalina.startup.Bootstrap start
root        45     1  0 07:47 pts/0    00:00:00 grep tomcat

```

外网访问[http://47.96.224.198:3355/](),出现以下界面：

![image-访问tomcat](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210102154829545.png)

这是因为阿里云镜像默认是下载最小的镜像，只下载了必要的组件。

```shell
root@5e2e3cb50a29:/usr/local/tomcat# ls  
BUILDING.txt	 LICENSE  README.md	 RUNNING.txt  conf  logs	    temp     webapps.dist
CONTRIBUTING.md  NOTICE   RELEASE-NOTES  bin	      lib   native-jni-lib  webapps  work
root@5e2e3cb50a29:/usr/local/tomcat# ls webapps  # webapps目录是空的
root@5e2e3cb50a29:/usr/local/tomcat# ls webapps.dist/   # webapps.dist目录才是原本的webapps目录
ROOT  docs  examples  host-manager  manager
root@5e2e3cb50a29:/usr/local/tomcat# mv webapps webapps.backup
root@5e2e3cb50a29:/usr/local/tomcat# mv webapps.dist webapps  # 将webapps.dist目录改成webapps目录
root@5e2e3cb50a29:/usr/local/tomcat# ls
BUILDING.txt	 LICENSE  README.md	 RUNNING.txt  conf  logs	    temp     webapps.backup
CONTRIBUTING.md  NOTICE   RELEASE-NOTES  bin	      lib   native-jni-lib  webapps  work
root@5e2e3cb50a29:/usr/local/tomcat# ls webapps
ROOT  docs  examples  host-manager  manager

```

再次访问[[http://47.96.224.198:3355/](),这次就可以正常访问了。

![image-访问docker中的tomcat](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210102193605035.png)



### 7.提交镜像

从仓库下载下来的镜像都是只读的，每当我们操作镜像，就会在原来镜像的上面加一层我们自己的操作，我们可以将它保存起来，下次直接使用我们操作过的镜像。

```shell
# 1. 查看镜像
[root@ming /]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED       STATUS       PORTS                    NAMES
89b24a4b7084   tomcat    "/bin/bash"   4 hours ago   Up 4 hours   0.0.0.0:3366->8080/tcp   infallible_lewin

# 2. 将我们操作过的镜像commit成mytomcat
[root@ming /]# docker commit -m 'tomcat add webapps' -a 'xiaoming' 89b24a4b7084 mytomcat
sha256:42d7b6dea11ce283033c6015765bff176ffbdad71105c02048e4e7352a5239f4

# 3. 查看镜像
[root@ming /]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
mytomcat     latest    42d7b6dea11c   10 seconds ago   654MB  # 我们自己生成的镜像
tomcat       latest    feba8d001e3f   2 weeks ago      649MB
nginx        latest    ae2feff98a0c   2 weeks ago      133MB
centos       latest    300e315adb2f   3 weeks ago      209MB
```

```shell
# 将镜像89b24a4b7084提交，生成我们自己的镜像mytomcat
docker commit -m 'tomcat add webapps' -a 'xiaoming' 89b24a4b7084 mytomcat

# 参数解释
-m  # 提交的信息,类似于git commit
-a  # 作者

```



### 8.容器数据卷

我们在容器内操作，数据保存在容器内，这是一件很`危险`的事情，假如不小心删除了容器，容器内的数据就全部丢失了。

有没有一种技术可以将容器内的数据同步到宿主机器上，将容器内的目录和宿主机的目录进行绑定，无论修改容器内的目录，还是修改宿主机的目录，都能自动同步数据，完成数据的持久化呢？ 这就是数据卷的技术。

使用以下命令完成数据卷的挂载：

```shell
docker run -v 宿主机的目录:容器内的目录
```

```shell
# 1. 进入容器，进行目录挂载
docker run -p 3355:8080 -v /home/test/:/home -it mytomcat /bin/bash

# 2. 查看镜像状态，查看挂载信息
docker inspect 7feb5a932a8b
"Mounts": [
            {
                "Type": "bind",
                "Source": "/home/test",
                "Destination": "/home",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],

# 3. 进入到容器内,/home目录下创建文件test.java
root@7feb5a932a8b:/home# ls
test.java

# 4. 查看宿主机器/home/test目录下是否同步了test.java文件
[root@ming test]# pwd
/home/test
[root@ming test]# ls
test.java

# 5.在宿主机上修改test.java文件，修改其内容
[root@ming test]# vim test.java 
[root@ming test]# cat test.java 
hello,world

# 6.查看容器中/home目录下的test.java文件
root@7feb5a932a8b:/home# cat test.java 
hello,world

# 结论：使用数据卷技术，可以将容器内的目录和宿主机器上的目录进行同步，从而达到数据持久化的目的。
```



#### 8.1 部署mysql

```shell
# 1. 直接使用docker run 下载mysql，如果本地不存在mysql的镜像，会从远程仓库中拉取
docker run -p 3307:3306 -v /home/mysql/data:/var/lib/mysql -v /home/mysql/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=123456 --name mysql5.7 mysql:5.7

# 参数解释
-e  # 设置环境
MYSQL_ROOT_PASSWORD # mysql root用户的密码
--name  # 容器的名称

# 2.使用navicat连接数据库，并创建数据库test
host:47.96.224.198:3307 
username:root
password:123456

# 3. 查看宿主机上/home/mysql/data目录下有没有test数据库
[root@ming data]# pwd
/home/mysql/data
[root@ming data]# ll |grep test
drwxr-x--- 2 systemd-bus-proxy input     4096 Jan  2 20:14 test  # test数据库,说明容器内的目录和宿主机的目录已经同步

# 4. 删除容器
docker rm -f a312a3e56f40 

# 5.数据持久化了，即使删除容器，数据也不会丢失
[root@ming data]# ll | grep test
drwxr-x--- 2 systemd-bus-proxy input     4096 Jan  2 20:14 test
```



#### 8.2 具名挂载和匿名挂载

之前使用的`docker run -v 宿主机的目录:容器内的目录`是指定目录的挂载方式。

除此之外，还有具名挂载和匿名挂载。

> 具名挂载

具名挂载通过`docker run -v 卷名：容器内的目录`进行挂载。

```shell
# 1. 具名挂载的方式启动nginx
docker run -d -p 3344:80 -v juming_nginx:/etc/nginx nginx

# 2. 查看数据卷
[root@ming /]# docker volume ls
DRIVER    VOLUME NAME
local     juming_nginx

# 3. 查看juming_nginx的详细信息
[root@ming /]# docker volume inspect juming_nginx
[
    {
        "CreatedAt": "2021-01-02T20:37:12+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/juming_nginx/_data",  # 挂载的路径
        "Name": "juming_nginx",
        "Options": null,
        "Scope": "local"
    }
]

# 4. 到挂载的路径下查看，nginx的配置文件目录已经挂载到了/var/lib/docker/volumes/juming_nginx/_data目录下
[root@ming _data]# pwd
/var/lib/docker/volumes/juming_nginx/_data
[root@ming _data]# ls
conf.d          koi-utf  mime.types  nginx.conf   uwsgi_params
fastcgi_params  koi-win  modules     scgi_params  win-utf

```

> 匿名挂载

匿名挂载通过`docker run -v 容器内路径`进行挂载。

```shell
# 1. 匿名挂载的方式启动nginx
docker run -d -p 3344:80 -v /etc/nginx.conf nginx

# 2. 查看数据卷,是一串字符串
[root@ming _data]# docker volume ls
DRIVER    VOLUME NAME
local     ffb564010a743ee8d607d1fd35bdf54fbef705881cf9c2625c431e83f626c532

# 3. 查看挂载的详细信息
[root@ming _data]# docker volume inspect ffb564010a743ee8d607d1fd35bdf54fbef705881cf9c2625c431e83f626c532
[
    {
        "CreatedAt": "2021-01-02T20:42:50+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/ffb564010a743ee8d607d1fd35bdf54fbef705881cf9c2625c431e83f626c532/_data",  # 挂载的目录
        "Name": "ffb564010a743ee8d607d1fd35bdf54fbef705881cf9c2625c431e83f626c532",
        "Options": null,
        "Scope": "local"
    }
]

# 4.到挂载的目录查看，发现目录是空的
[root@ming _data]# pwd
/var/lib/docker/volumes/2909b1711dfe40c72036dc8d950d2da653a6b88563acbb0b1e49932469e1247d/_data
[root@ming _data]# ls

```

我们发现，如果不是以指定目录的方式进行挂载，挂载的目录默认是在`/var/lib/docker/volumes/卷名/_data/`目录下。



#### 8.3 权限

相对于容器内而言，默认情况下，文件是可读可写的。

```shell
# 1. 启动nginx容器并进入容器
docker run -p 3344:80 -v juming_nginx:/etc/nginx.conf -it nginx /bin/bash

# 2. 到/etc/目录下，查看nginx.conf的权限
drwxr-xr-x 3 root root    4096 Jan  2 12:37 nginx.conf  # 可读可写的权限
```

我们可以在启动容器的时候，指定容器内的文件权限：

```shell
# 1. ro = read only ,在容器内只读，不能修改
docker run -p 3344:80 -v juming_nginx:/etc/nginx.conf:ro -it nginx /bin/bash

# 2. rw = read write,在容器内可读，可写
docker run -p 3344:80 -v juming_nginx:/etc/nginx.conf:ro -it nginx /bin/bash
```

 

#### 8.4 多个容器挂载同一个目录

使用--volume-for指定一个容器，同步这个容器的数据。就相当于，继承一个容器，自动同步它的数据。

```shell
# 1. 启动容器centos1,挂载容器内的目录/home/centos1
docker run -it --name centos1 -v centos1:/home/centos1/ centos /bin/bash

# 2. 在centos1容器的/home/centos1目录下创建centos1.txt文件
[root@69a8809a9e6f centos1]# pwd
/home/centos1
[root@69a8809a9e6f centos1]# ls
centos1.txt

# 3. 查看宿主机上centos1的挂载目录
[root@ming _data]# pwd
/var/lib/docker/volumes/centos1/_data
[root@ming _data]# ls
centos1.txt

# 4. 启动容器centos2，centos2继承自centos1
docker run -it --name centos2 --volumes-from centos1 centos /bin/bash

# 5. 进入到centos2的/home/centos1目录下
[root@a5afec9346bd centos1]# pwd
/home/centos1
[root@a5afec9346bd centos1]# ls
centos1.txt

# 在centos2的目录下，可以看到centos1的目录

# 6. centos2容器中,在centos1目录中创建centos2.txt
[root@a5afec9346bd centos2]# pwd
/home/centos1
[root@a5afec9346bd centos2]# ls
centos1.txt centos2.txt


# 7. 进入到centos1容器中，查看/home/centos1目录下是否多了centos2.txt
[root@06e27ea51ee0 centos1]# ls  # centos1的目录下,存在centos2.txt，说明centos2创建的文件自动同步到centos1
centos1.txt centos2.txt

# 8. 查看centos2的挂载目录
docker inspect centos2
"Mounts": [
            {
                "Type": "volume",
                "Name": "centos1",
                "Source": "/var/lib/docker/volumes/centos1/_data",
                "Destination": "/home/centos1",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],

# 9. 查看centos1的挂载目录
docker inspect centos1
"Mounts": [
            {
                "Type": "volume",
                "Name": "centos1",
                "Source": "/var/lib/docker/volumes/centos1/_data",
                "Destination": "/home/centos1",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],

# 结论：centos1和centos2挂载的是同一个目录，连name都是相同的centos1。
# 这意味着，我们查看centos2的详细数据卷信息，也是使用docker volume inspect centos1

```

接下来，我们创建一个新的容器centos3，让它--volumes-from centos2,它能不能自动同步centos2创建的目录centos2呢？

```shell
# 1. 启动centos3容器
docker run -it --name centos3 --volumes-from centos2 centos /bin/bash

# 2. 查看/home目录下是否存在centos2目录
[root@96b772d61de0 home]# ls  # 说明centos3不能同步centos2创建的目录
centos1

# 3. 查看centos3挂载的目录信息
docker inspect centos3
"Mounts": [
            {
                "Type": "volume",
                "Name": "centos1",
                "Source": "/var/lib/docker/volumes/centos1/_data",
                "Destination": "/home/centos1",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],

# centos3虽然继承自centos2，但是centos2是继承自centos1，挂载的只是centos1的目录，所以centos3不能同步centos2创建的目录。
```



### 9.Dockfile

Dockfile是面向开发的，我们以后要发布项目，就需要编写dockfile文件。

#### 9.1 制作一个Dockefile

dockerfile就是一个脚本文件，里面的脚本用来创建镜像。

新建一个目录/home/docker_volume_test，创建dockerfile1的脚本文件。

需要注意的几点：

* 每个保留关键字（指令）都必须是大写字母
* 执行从上到下顺序执行
* 每一个指令都会创建提交一个新的镜像层，并提交
* 必须使用双引号，单引号不行

```shell
FROM centos

# 匿名挂载
VOLUME ["volume1","volume2"]

CMD /bin/bash
```

写好脚本文件后，使用`docker build`命令生成镜像。

需要注意的有2点：

* -t 生成的镜像名称不能带/
* 最后一个参数指定生成路径，.代表当前目录
* 如果文件命名为Dockerfile，在docker build的时候可以不指定-f参数

```shell
# 1. 利用当前目录下的dockerfile文件在当前目录下生成xiaoming/centos的镜像
[root@ming docker_volume_test]# docker build -f dockerfile1 -t xiaoming/centos .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM centos
 ---> 300e315adb2f
Step 2/4 : VOLUME ["volume1","volume2"]
 ---> Running in 7d9097ed198e
Removing intermediate container 7d9097ed198e
 ---> 8f3d72c8bccc
Step 3/4 : CMD echo '------end------'
 ---> Running in 27f56b2fe602
Removing intermediate container 27f56b2fe602
 ---> bdcf8bb3ee27
Step 4/4 : CMD /bin/bash
 ---> Running in e9c64f298af8
Removing intermediate container e9c64f298af8
 ---> 7fc575cbe035
Successfully built 7fc575cbe035
Successfully tagged xiaoming/centos:latest


# 参数解释
-f # 指定的dockfile文件
-t # target,生成的目录镜像

# 2. 查看镜像
[root@ming docker_volume_test]# docker images
REPOSITORY        TAG       IMAGE ID       CREATED          SIZE
xiaoming/centos   latest    7fc575cbe035   21 seconds ago   209MB  # 利用dockerfile生成的镜像文件

# 3. 启动容器
[root@ming docker_volume_test]# docker run -it xiaoming/centos /bin/bash
[root@40476da59260 /]# ls -l
drwxr-xr-x  2 root root 4096 Jan  3 07:14 volume1 # 生成镜像的时候自动挂载的volume1目录
drwxr-xr-x  2 root root 4096 Jan  3 07:14 volume2 # 生成镜像的时候自动挂载的volume2目录

# 4. 在容器内的volume1目录中创建文件volume1.txt
[root@40476da59260 volume1]# pwd
/volume1
[root@40476da59260 volume1]# ls
volume1.txt

# 5. 查看挂载信息
docker inspect 40476da59260  # 容器id
"Mounts": [
            {
                "Type": "volume",
                "Name": "188f9cd826889b4497db0df774a3c493666b808bce0cffbadc90ae20901fda49",
                "Source": "/var/lib/docker/volumes/188f9cd826889b4497db0df774a3c493666b808bce0cffbadc90ae20901fda49/_data",
                "Destination": "volume1",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "9ecc05996ab6035f6fee907b130473f8ad9eba1244635a98bbf3e5dbde8747a8",
                "Source": "/var/lib/docker/volumes/9ecc05996ab6035f6fee907b130473f8ad9eba1244635a98bbf3e5dbde8747a8/_data",
                "Destination": "volume2",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],

# 6. 到挂载目录下查看是否同步了容器的文件
[root@ming _data]# pwd
/var/lib/docker/volumes/188f9cd826889b4497db0df774a3c493666b808bce0cffbadc90ae20901fda49/_data
[root@ming _data]# ls
volume1.txt  # 容器内创建的文件已经同步，说明挂载成功。

```



#### 9.2 Dockfile指令

| 命令       | 描述                                             |
| ---------- | ------------------------------------------------ |
| FROM       | 指定基础镜像，一切从这里开始构建                 |
| MAINTAINER | 指定维护者信息（姓名+邮箱）                      |
| RUN        | 镜像构建的时候需要运行的命令                     |
| ADD        | 添加内容，会自动解压                             |
| WORKDIR    | 镜像的工作目录                                   |
| VOLUME     | 设置卷，挂载主机目录                             |
| EXPOSE     | 暴露端口配置                                     |
| CMD        | 指定容器启动的时候运行的命令，只有最后一个会生效 |
| ENTRYPOINT | 指定这个容器启动的时候运行的命令，可以追加命令   |
| ONBUILD    | 当构建一个被继承Dockfile就会触发指令             |
| COPY       | 类似ADD,将文件拷贝到镜像中                       |
| ENV        | 构建的时候设置环境变量                           |



#### 9.3 CMD与ENTRYPOINT的区别

我们编写一个Dockerfile文件，创建添加了vim的centos：

```shell
# 基于centos镜像构建
FROM centos
# 维护者
MAINTAINER "ming<11498895@qq.com>"
# 构建镜像，也就是执行docker build的时候，下载vim命令
RUN yum -y install vim
# 设置工作目录，容器启动的时候就在这个工作目录下
ENV MAINDIR /home
WORKDIR $MAINDIR
# 启动容器时，执行的命令
CMD ["ls","-l"]

```

```shell
# 执行docker build 生成镜像
docker build -f centos_dockfile -t centos:0.1 .

# 这里我们指定了生成镜像的版本号，在启动的时候，也需要指定镜像的版本号才能启动
```

```shell
# 1. 启动容器，注意这里不指定使用/bin/bash启动，才会执行dockerfile中的cmd命令
[root@ming docker_volume_test]# docker run centos:0.1
total 0

# 容器启动的时候执行了ls -l命令

# 2. 做个测试，在启动容器的时候使其执行ls -l /命令
[root@ming docker_volume_test]# docker run centos:0.1 /
docker: Error response from daemon: OCI runtime create failed: container_linux.go:370: starting container process caused: exec: "/": permission denied: unknown.

# 这时候，产生了一个错误，这是由于docker run时指定的/命令替换了dockfile中CMD指定指定的ls -l命令，而/命令不是一个正确的命令，就报错了
```

接下来，我们在dockerfile文件中使用ENTRYPOINT指令替换CMD指令：

```shell
FROM centos
MAINTAINER "ming<11498895@qq.com>"

RUN yum -y install vim

ENV MAINDIR /home
WORKDIR $MAINDIR

ENTRYPOINT ["ls","-l"]
```

```shell
# 1. 我们直接启动容器，执行的是ls -l 命令
[root@ming docker_volume_test]# docker run centos:0.1
total 0

# 2. 我们在启动容器的时候追加命令，使其执行ls -l /命令
[root@ming docker_volume_test]# docker run centos:0.1 /
total 48
lrwxrwxrwx  1 root root    7 Nov  3 15:22 bin -> usr/bin
drwxr-xr-x  5 root root  340 Jan  3 09:57 dev
drwxr-xr-x  1 root root 4096 Jan  3 09:57 etc
drwxr-xr-x  2 root root 4096 Nov  3 15:22 home
lrwxrwxrwx  1 root root    7 Nov  3 15:22 lib -> usr/lib
lrwxrwxrwx  1 root root    9 Nov  3 15:22 lib64 -> usr/lib64
drwx------  2 root root 4096 Dec  4 17:37 lost+found
drwxr-xr-x  2 root root 4096 Nov  3 15:22 media
drwxr-xr-x  2 root root 4096 Nov  3 15:22 mnt
drwxr-xr-x  2 root root 4096 Nov  3 15:22 opt
dr-xr-xr-x 90 root root    0 Jan  3 09:57 proc
dr-xr-x---  1 root root 4096 Jan  3 09:55 root
drwxr-xr-x 11 root root 4096 Dec  4 17:37 run
lrwxrwxrwx  1 root root    8 Nov  3 15:22 sbin -> usr/sbin
drwxr-xr-x  2 root root 4096 Nov  3 15:22 srv
dr-xr-xr-x 13 root root    0 Jan  3 09:57 sys
drwxrwxrwt  7 root root 4096 Dec  4 17:37 tmp
drwxr-xr-x  1 root root 4096 Dec  4 17:37 usr
drwxr-xr-x  1 root root 4096 Dec  4 17:37 var

# 结论：如果使用ENTRYPOINT,我们在docker run时指定的命令是会追加到原来命令的后面的
```



#### 9.4 查看构建过程

```shell
# 1. 使用docker history 镜像id  可以查看一个镜像构建的过程
[root@ming ~]# docker history 7e97f5b8161a
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
7e97f5b8161a   3 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "sh /…   0B        
4744060107e4   3 minutes ago   /bin/sh -c #(nop)  EXPOSE 8080                  0B        
cd04af7c483b   3 minutes ago   /bin/sh -c #(nop) WORKDIR /usr/local/tomcat     0B        
c7b335498d11   3 minutes ago   /bin/sh -c #(nop)  ENV MAINPATH=/usr/local/t…   0B        
d07929bdbbaa   3 minutes ago   /bin/sh -c #(nop)  ENV PATH=/usr/local/sbin:…   0B        
f3fc451eee02   3 minutes ago   /bin/sh -c #(nop)  ENV CATALINA_BASH=/usr/lo…   0B        
3498f6fce614   3 minutes ago   /bin/sh -c #(nop)  ENV CATALINA_HOME=/usr/lo…   0B        
978f9d791d9c   3 minutes ago   /bin/sh -c #(nop)  ENV CLASSPATH=.:/usr/loca…   0B        
92e9bc253a9e   3 minutes ago   /bin/sh -c #(nop)  ENV JAVA_HOME=/usr/local/…   0B        
8773f4df5b08   3 minutes ago   /bin/sh -c cd /usr/local/java && yum -y inst…   349MB     
90e3f7678077   4 minutes ago   /bin/sh -c #(nop) ADD file:a1bfef71355927262…   113MB     
f4149d27d5d7   4 minutes ago   /bin/sh -c #(nop) ADD file:f10b183415fb392e3…   14.6MB    
256d743023b7   4 minutes ago   /bin/sh -c yum -y install rpm                   1.83MB    
1bbdc376780a   4 minutes ago   /bin/sh -c yum -y install vim                   58MB      
c1b496dc6b45   5 minutes ago   /bin/sh -c #(nop)  MAINTAINER xiaoming<11498…   0B        
300e315adb2f   3 weeks ago     /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
<missing>      3 weeks ago     /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B        
<missing>      3 weeks ago     /bin/sh -c #(nop) ADD file:bd7a2aed6ede423b7…   209MB 
```



#### 9.5 制作tomcat的Dockfile

> 准备tomcat，JDK的压缩包,编写Dockerfile

```shell
# 基于centos
FROM centos
# 作者信息
MAINTAINER xiaoming<1149895826@qq.com>
# 下载vim
RUN yum -y install vim
RUN yum -y install rpm
# 将当前目录下的压缩包发送到服务器，ADD指令会自动解压
ADD apache-tomcat-8.5.61.tar.gz /usr/local/tomcat/
# 将当前目录下的JDK rpm包发送到服务器
ADD jdk-8u271-linux-x64.rpm /usr/local/java/
# 下载rpm包，安装JDK
RUN cd /usr/local/java && yum -y install /usr/local/java/jdk-8u271-linux-x64.rpm && yum install which -y
# 设置环境变量,这里非常重要，稍微有一点错误，容器就启动不起来
ENV JAVA_HOME /usr/java/jdk1.8.0_271-amd64
ENV CLASSPATH .:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/tomcat/apache-tomcat-8.5.61
ENV CATALINA_BASH /usr/local/tomcat/apache-tomcat-8.5.61
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
# 设置容器启动时的工作目录
ENV MAINPATH /usr/local/tomcat
WORKDIR $MAINPATH
# 暴露容器的端口
EXPOSE 8080
# 容器启动时执行的命令：启动tomcat，并打印启动日志
CMD sh /usr/local/tomcat/apache-tomcat-8.5.61/bin/startup.sh && tail -f /usr/local/tomcat/apache-tomcat-8.5.61/logs/catalina.out
```

> docker build 生成镜像文件

```shell
# 1. 由于文件命名是Dockerfile,无需指定-f参数
docker build -t diy_tomcat .

# 2. 后台启动容器,将tomcat的webapps和日志目录挂载到宿主机,启动后自动打印启动日志
[root@ming diy_tomcat]# docker run -p 3344:8080 -v /home/diy_tomcat/webapps/:/usr/local/tomcat/apache-tomcat-8.5.61/webapps/ -v /home/diy_tomcat/logs/:/usr/local/tomcat/apache-tomcat-8.5.61/logs/ diy_tomcat 

# 3. 查看容器,容器正在运行
[root@ming diy_tomcat]# docker ps
CONTAINER ID   IMAGE        COMMAND                  CREATED              STATUS              PORTS                    NAMES
3134796152c3   diy_tomcat   "/bin/sh -c 'sh /usr…"   About a minute ago   Up About a minute   0.0.0.0:3344->8080/tcp   heuristic_kare

# 4.外网访问http://47.96.224.198:3344/,访问正常

# 5. 查看宿主机挂载的目录是否挂载成功
[root@ming webapps]# pwd
/home/diy_tomcat/webapps
[root@ming webapps]# ls
balance  dev  docs  examples  host-manager  manager  ROOT

# 以后，我们只需要将项目部署在宿主机的webapps目录下就可以啦
```





```shell
echo "1">/proc/sys/vm/drop_caches

```



### 10.发布自己的镜像

```shell
# 1. 首先需要创建DockerHub的账号

# 2. 在命令行中先登录账户
[root@ming ~]# docker login -u jinming8
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

# 3. 将要发布的镜像tag成 用户名/镜像名，这一步至关重要
[root@ming ~]# docker tag nginx jinming8/nginx:0.1

# 4. 执行push命令，发布到Dockhub
[root@ming ~]# docker push jinming8/nginx:0.1
The push refers to repository [docker.io/jinming8/nginx]
4eaf0ea085df: Mounted from library/nginx 
2c7498eef94a: Mounted from library/nginx 
7d2b207c2679: Mounted from library/nginx 
5c4e5adc71a8: Mounted from library/nginx 
87c8a1d8f54f: Mounted from library/mysql 
0.1: digest: sha256:13e4551010728646aa7e1b1ac5313e04cf75d051fa441396832fcd6d600b5e71 size: 1362   # 发布成功

# 5. 退出登录
[root@ming ~]# docker logout
Removing login credentials for https://index.docker.io/v1/
```



### 11. Docker网络

#### 11.1 问题1：宿主机和容器能不能ping通？  ✔

```shell
# 1. 查看宿主机器的ip addr
[root@ming ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:16:3f:00:fd:98 brd ff:ff:ff:ff:ff:ff
    inet 172.16.47.149/20 brd 172.16.47.255 scope global dynamic eth0
       valid_lft 314901004sec preferred_lft 314901004sec
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN 
    link/ether 02:42:e7:4f:27:e0 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever

# 宿主机器上有3个网卡：
# 1. lo:本地回环地址
# 2. eth0:阿里云内网，ip=172.16.47.149
# 3. docker0:docker的网卡，只要我们安装了docker就会有这个网卡

# 2. 启动一个tomcat容器
[root@ming ~]# docker run -it --name tomcat1  tomcat /bin/bash

# 3. 进入容器内查看ip
root@1826a8cb702d:/usr/local/tomcat# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
332: eth0@if333: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

# 容器内有2个网卡：
# 1. lo:本地回环地址
# 2. 332: eth0@if333:docker为容器分配的网卡，ip=172.17.0.2

# 4. 在容器内ping宿主机器的ip 172.16.47.149
root@1826a8cb702d:/usr/local/tomcat# ping 172.16.47.149
PING 172.16.47.149 (172.16.47.149) 56(84) bytes of data.
64 bytes from 172.16.47.149: icmp_seq=1 ttl=64 time=0.079 ms
64 bytes from 172.16.47.149: icmp_seq=2 ttl=64 time=0.075 ms
64 bytes from 172.16.47.149: icmp_seq=3 ttl=64 time=0.077 ms
64 bytes from 172.16.47.149: icmp_seq=4 ttl=64 time=0.077 ms


# 结论1：在容器内是可以ping通宿主机器地址的

# 5. CTRL+P+Q退出容器，在宿主机查看ip addr
[root@ming ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:16:3f:00:fd:98 brd ff:ff:ff:ff:ff:ff
    inet 172.16.47.149/20 brd 172.16.47.255 scope global dynamic eth0
       valid_lft 314900722sec preferred_lft 314900722sec
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:e7:4f:27:e0 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
333: veth740bb55@if332: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP 
    link/ether 06:b3:9f:71:29:93 brd ff:ff:ff:ff:ff:ff link-netnsid 0

# 相比与启动容器tomcat之前，宿主机多了一个网卡333: veth740bb55@if332，而且我们发现，它与容器的内的网卡332: eth0@if333形成了一对。

# 6. 在宿主机内ping容器地址172.17.0.2
[root@ming ~]# ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.059 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.066 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.064 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.084 ms

# 结论2:宿主机是可以ping通容器内的ip的
```

小结：

* 安装了docker之后，会默认分配一个网卡docker0
* 我们每启动一个容器，docker就会给容器分配一个ip，使用的是桥接模式，veth-pair技术(一对虚拟设备接口，在这里，一端连着容器，一端连着docker0)

#### 11.2 问题2：两个容器直接能否直接ping通？ ✔

```shell
# 1.再次启动一个容器tomcat2，查看ip地址
[root@ming ~]# docker run -it --name=tomcat2 tomcat /bin/bash
root@9d1e83b5c4d6:/usr/local/tomcat# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
334: eth0@if335: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
       
# 容器内也是有2个网卡，一个本机回环地址，一个docker分配的网卡，tomcat2的ip=172.17.0.3

# 2. tomcat2 能否ping通 tomcat1 ？ 
root@9d1e83b5c4d6:/usr/local/tomcat# ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.099 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.087 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.085 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.089 ms

# 结果肯定是可以的，因为从ip地址可以看出，它们属于同一个网段。

# 3. 进入到tomcat1容器，ping tomcat2的ip
[root@ming ~]# docker attach 1826a8cb702d
root@1826a8cb702d:/usr/local/tomcat# ping 172.17.0.3
PING 172.17.0.3 (172.17.0.3) 56(84) bytes of data.
64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.075 ms
64 bytes from 172.17.0.3: icmp_seq=2 ttl=64 time=0.090 ms
64 bytes from 172.17.0.3: icmp_seq=3 ttl=64 time=0.085 ms
64 bytes from 172.17.0.3: icmp_seq=4 ttl=64 time=0.089 ms

# 再次验证。

# 结论：两个容器之间也是可以相互ping通的。虽然两个容器直接能ping通，但两个容器间却不是直接相连接的。它们通过veth-pair与docker0相连，docker0这个桥接起了作用

```

#### 11.3 能否通过容器的name来连接？ ×(--link ✔)

```shell
# 1. 进入到tomcat1容器，直接ping tomcat2容器的名字
root@1826a8cb702d:/usr/local/tomcat# ping tomcat2
ping: tomcat2: Name or service not known

# 不能直接通过容器名称来连接

# 2. 重新启动容器tomcat3，指定--link 连接tomcat2
[root@ming ~]# docker run -it --name tomcat3 --link tomcat2 tomcat /bin/bash 
root@0ded9ce6d825:/usr/local/tomcat# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
338: eth0@if339: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

# 注意，这里tomcat3的ip=172.17.0.2，好像和tomcat1的ip相同，但是我这里是把tomcat1容器退出了的

# 3. 在tomcat3容器内直接ping tomcat2
root@0ded9ce6d825:/usr/local/tomcat# ping tomcat2
PING tomcat2 (172.17.0.3) 56(84) bytes of data.
64 bytes from tomcat2 (172.17.0.3): icmp_seq=1 ttl=64 time=0.094 ms
64 bytes from tomcat2 (172.17.0.3): icmp_seq=2 ttl=64 time=0.084 ms
64 bytes from tomcat2 (172.17.0.3): icmp_seq=3 ttl=64 time=0.085 ms
64 bytes from tomcat2 (172.17.0.3): icmp_seq=4 ttl=64 time=0.084 ms

# 结论1:在启动容器的时候，使用--link 参数指定容器，可以与指定的容器通过名称连接

# 4. 进入容器tomcat2，ping tomcat3
[root@ming ~]# docker attach 9d1e83b5c4d6
root@9d1e83b5c4d6:/usr/local/tomcat# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
334: eth0@if335: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
root@9d1e83b5c4d6:/usr/local/tomcat# ping tomcat3
ping: tomcat3: Name or service not known

# 结论2:tomcat2是不能ping通tomcat3的，说明tomcat3容器启动的时候指定的--link参数是单向的。

# 5. 进入tomcat3容器，查看/etc/hosts文件
[root@ming ~]# docker attach 0ded9ce6d825
root@0ded9ce6d825:/usr/local/tomcat# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.3	tomcat2 9d1e83b5c4d6
172.17.0.2	0ded9ce6d825

# 结论3：我们惊奇的发现，所谓的tomcat3启动时指定的--link参数，只是修改了tomcat的hosts文件，当tomcat3访问tomcat2的时候，跳转到172.17.0.3(也就是tomcat的ip地址)
```

虽然--link能使用容器名称来连接，但这种做法是过时的，更好的方法是使用自定义网络。



#### 11.4 自定义网络

> 查看网络信息

```shell
# 1. 查看网络信息
[root@ming ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
1cd4dcbb6871   bridge    bridge    local
937119faf618   host      host      local
5f780b566b6c   none      null      local

# docker有3种网络模式：
# 1.bridge:桥接模式，docker默认的网络模式
# 2.host:与宿主机器共享网络
# 3.none:不使用网络

# 2. 查看某个网络模式的详细信息
docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "1cd4dcbb6871cbcce8292729579d2a62dee582c9b7cd3810298c1d69bc1de4cb",
        "Created": "2021-01-01T21:28:47.705629496+08:00",
        "Scope": "local",
        "Driver": "bridge",   # 桥接
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",  # 子网
                    "Gateway": "172.17.0.1"     # 网关
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
            "0ded9ce6d825fac539ca1190591a162f9437a10a5b85a75f10c2631431748686": {
                "Name": "tomcat3",
                "EndpointID": "82d9154362a6a3e96e6ca9e52b550682a8eb7f79d99d19809116c6b44d67bd8f",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16", # ip
                "IPv6Address": ""
            },
            "9d1e83b5c4d6f1d1c569d7df4d24df17be392f9d72c33b4107ea4f8e1686bb16": {
                "Name": "tomcat2",
                "EndpointID": "354c25db100af0a53d982fd893a52fee7f5b4d8f533ce9901de24bf9c191dc93",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]

```

> 自定义一个网络

```shell
# 1. 创建网络，指定网关和子网掩码
[root@ming ~]# docker network create --gateway 192.168.0.1 --subnet 192.168.0.0/16 mynet 

# 2. 查看创建的网络
[root@ming ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
1cd4dcbb6871   bridge    bridge    local
937119faf618   host      host      local
805e98b29181   mynet     bridge    local
5f780b566b6c   none      null      local

# 3. 启动容器的时候指定网络
[root@ming ~]# docker run -it --name tomcat1 --net mynet tomcat /bin/bash
root@a8d9dd3ce963:/usr/local/tomcat# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
341: eth0@if342: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:c0:a8:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.0.2/16 brd 192.168.255.255 scope global eth0
       valid_lft forever preferred_lft forever
       
 # 容器启动后的ip=192.168.0.2,是我们自定义的网络mynet
 
 # 4. 查看mynet的详细信息
 
[root@ming ~]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "805e98b29181a303950c9f3267587e6ea27c2df3728a0723e4f16acb43add392",
        "Created": "2021-01-05T19:12:36.296644818+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16", # 我们设置的子网
                    "Gateway": "192.168.0.1"    # 我们设置的网关
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
        "Containers": {  # 此网络内的容器信息
            "a8d9dd3ce9633968957a5d1487bbf85716985f97ed6cf4f2b9a78c47d2c3f67d": {
                "Name": "tomcat1",
                "EndpointID": "f1bbab5f9d330f62f17ef2076dd09d00a124d14d5ce8c3658d8e6459215c4e19",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",  # tomcat1容器的ip
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]

# 5. 启动容器tomcat2，查看ip，并通过容器名称连接tomcat1
[root@ming ~]# docker run -it --name tomcat2 --net mynet tomcat /bin/bash
root@bf9e41e968ae:/usr/local/tomcat# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
343: eth0@if344: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:c0:a8:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.0.3/16 brd 192.168.255.255 scope global eth0
       valid_lft forever preferred_lft forever
root@bf9e41e968ae:/usr/local/tomcat# ping tomcat1
PING tomcat1 (192.168.0.2) 56(84) bytes of data.
64 bytes from tomcat1.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.101 ms
64 bytes from tomcat1.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.083 ms
64 bytes from tomcat1.mynet (192.168.0.2): icmp_seq=3 ttl=64 time=0.098 ms
64 bytes from tomcat1.mynet (192.168.0.2): icmp_seq=4 ttl=64 time=0.088 ms

# 结论1:通过自定义网络是能通过容器名称进行连接的。
```

#### 11.5 网络互联

上面tomcat1和tomcat2已经连接在mynet网络下了，那么连接在docker0的容器，能否访问mynet网络下的容器呢？

```shell
# 1. 使用默认的docker0启动一个容器tomcat3
[root@ming ~]# docker run -it --name tomcat3 tomcat /bin/bash
root@bac489a3beb3:/usr/local/tomcat# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
345: eth0@if346: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

# 2. 使用tomcat3连接mynet网络下的tomcat2
root@bac489a3beb3:/usr/local/tomcat# ping tomcat2
ping: tomcat2: Name or service not known
root@bac489a3beb3:/usr/local/tomcat# ping 192.168.0.3
PING 192.168.0.3 (192.168.0.3) 56(84) bytes of data.

# 无论是通过ip，还是通过容器名称，都无法连接。
# mynet和docker0的网段不同

# 3. 使用docker network connect让docker0下的tomcat3容器连接到mynet网络
[root@ming ~]# docker network connect mynet tomcat3

# 4. 查看mynet网络的详细信息
[root@ming ~]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "805e98b29181a303950c9f3267587e6ea27c2df3728a0723e4f16acb43add392",
        "Created": "2021-01-05T19:12:36.296644818+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
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
            "a8d9dd3ce9633968957a5d1487bbf85716985f97ed6cf4f2b9a78c47d2c3f67d": {
                "Name": "tomcat1",
                "EndpointID": "f1bbab5f9d330f62f17ef2076dd09d00a124d14d5ce8c3658d8e6459215c4e19",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            },
            "bf9e41e968aeccc9ccd0ecc3efc60c1d38b550ea783dbc530b68eba9b7952d5c": {
                "Name": "tomcat2",
                "EndpointID": "f016fc90361535f9e08c0ad4dfd92bc630c83a88704680231e33a4bbd201f15a",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            },
            "f1933b52fa5559c60890e2a223cb118ff2290a5f0ff4966d1cffac28c280c439": {
                "Name": "tomcat3",  # mynet网络下已经有了tomcat3的容器
                "EndpointID": "48ce720242696f70d6ae7051a8754d56b30ae2d3bc08b82761822eeca25ddebf",
                "MacAddress": "02:42:c0:a8:00:04",
                "IPv4Address": "192.168.0.4/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]

# 5. 查看tomcat3容器的ip
root@f1933b52fa55:/usr/local/tomcat# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
347: eth0@if348: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
349: eth1@if350: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:c0:a8:00:04 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.0.4/16 brd 192.168.255.255 scope global eth1
       valid_lft forever preferred_lft forever
       
 # 可以看到，tomcat有2个ip地址
 #          172.17.0.2  docker0网卡的
 #          192.168.0.4  mynet网卡的
```



