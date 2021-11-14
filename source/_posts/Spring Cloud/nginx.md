### linux安装nginx

1. 安装nginx所需的依赖

```shell
yum -y install gcc pcre-devel zlib-devel openssl openssl-devel
```

2. 在/usr/local/下创建nginx目录

3. [https://nginx.org/download/]()下载tar.gz包，并解压tar -zxvf xxxx.tar.gz

4. 配置./configure --prefix=/usr/local/nginx

5. 执行make && make install

6. 切换到/usr/local/nginx/sbin，执行nginx -t

如果出现以下，则说明安装成功。

```java
[root@ming sbin]# ./nginx -t
nginx: the configuration file /usr/local/nginx//conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx//conf/nginx.conf test is successful
```

7. 运行nginx,执行./nginx

如果出现以下界面，说明安装成功。

![image-安装成功](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201228003249059.png)

8. 如果打不开，检查防火墙是不是没开放80端口。

```shell
firewall-cmd --query-port=80/tcp
# 开放端口，-permanent  表示永久生效，没有此参数重启后失效
firewall-cmd --add-port=80/tcp --permanent
#重启防火墙
systemctl restart firewalld
```

9. 配置nginx开机自启动

```shell
vi /etc/rc.d/rc.local
```

 添加以下内容

```shell
# nginx开机自启动
/usr/local/nginx/sbin/nginx
```

