### 1.查看网页源码

直接在curl命令后加上网址，就可以看到网页源码。我们以网址[www.sina.com]()为例.

直接使用curl相当于发送get请求，这就相当与wget命令。

```shell
curl www.sina.com
```

```html
<html>
<head><title>301 Moved Permanently</title></head>
<body bgcolor="white">
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>

```

```shell
wget www.sina.com
```

wget会直接把文件下载到当前目录，而curl命令需要使用-o属性保存。

以下命令将[www.sina.com]()的源代码保存为index.html。

```shell
curl -o index.html www.sina.com
```

直接使用-O可以不指定文件的名称，直接使用远程文件的名称。但如果远程文件没有名称，会报以下错误：

```txt
curl: Remote file name has no length!
```



### 2.自动跳转

上面我们访问[www.sina.com]()的时候，返回的html是错误页面。这是因为www.sina.com新的网址是[www.sina.com.cn]().

在使用curl的时候，使用-L参数可以自动跳转。

```shell
curl -L www.sina.com
```



### 3.显示响应头信息

使用-i参数能显示response头信息，连带着网页一起。

```shell
curl -i www.sina.com
```

```txt
HTTP/1.1 301 Moved Permanently
Server: nginx
Date: Wed, 30 Dec 2020 07:38:19 GMT
Content-Type: text/html
Content-Length: 178
Connection: keep-alive
Location: http://www.sina.com.cn/
Expires: Wed, 30 Dec 2020 07:38:55 GMT
Cache-Control: max-age=120
X-Via-SSL: ssl.23.sinag1.qxg.lb.sinanode.com
Edge-Copy-Time: 1609313896730
Age: 84
Via: https/1.1 ctc.guangzhou.union.182 (ApacheTrafficServer/6.2.1 [cRs f ]), https/1.1 ctc.qingdao.union.63 (ApacheTrafficServer/6.2.1 [cRs f ])
X-Via-Edge: 1609313899518c6e0602ff105f98c4141b37e
X-Cache: HIT.63
X-Via-CDN: f=edge,s=ctc.qingdao.union.46.nb.sinaedge.com,c=47.96.224.198;f=Edge,s=ctc.qingdao.union.63,c=140.249.5.46

<html>
<head><title>301 Moved Permanently</title></head>
<body bgcolor="white">
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

如果只想显示头信息，不想显示网页，使用-I参数。

```shell
curl -I www.sina.com
```



### 4.显示通信过程

-v 参数可以显示一次http通信的整个过程，包括端口连接、 request头信息、response头信息。

```shell
* About to connect() to www.sina.com port 80 (#0)
*   Trying 112.90.6.240...
* Connected to www.sina.com (112.90.6.240) port 80 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: www.sina.com
> Accept: */*
> 
< HTTP/1.1 301 Moved Permanently
< Server: nginx
< Date: Wed, 30 Dec 2020 07:43:51 GMT
< Content-Type: text/html
< Content-Length: 178
< Connection: keep-alive
< Location: http://www.sina.com.cn/
< Expires: Wed, 30 Dec 2020 07:45:33 GMT
< Cache-Control: max-age=120
< X-Via-SSL: ssl.95.sinag1.qxg.lb.sinanode.com
< Edge-Copy-Time: 1609314213341
< Age: 18
< Via: https/1.1 cnc.guangzhou.union.55 (ApacheTrafficServer/6.2.1 [cRs f ])
< X-Cache: HIT.70
< X-Via-CDN: f=edge,s=cnc.guangzhou.union.57.nb.sinaedge.com,c=47.96.224.198;f=Edge,s=cnc.guangzhou.union.55,c=112.90.6.74
< X-Via-Edge: 1609314231193c6e0602ff0065a7029a4230b
< 
<html>
<head><title>301 Moved Permanently</title></head>
<body bgcolor="white">
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
* Connection #0 to host www.sina.com left intact

```

如果想查看更加详细的信息，使用下面的命令将结果输处到out.txt文件：

```shell
curl --trace out.txt www.sina.com
```



### 5.发送表单信息

发送表单信息有GET和POST两种方法。GET方法相对简单，只要把数据附在网址后面就行。

```shell
curl example.com/form.cgi?data=xxx
```

POST方法必须把数据和网址分开，curl就要用到--data参数。

```shell
curl -X POST --data "data=xxx" example.com/form.cgi
```

如果你的数据没有经过表单编码，还可以让curl为你编码，参数是`--data-urlencode`。

```shell
curl -X POST --data-urlencode "date=April 1" example.com/form.cgi
```



### 6.HTTP动词

curl默认的HTTP动词是GET，使用`-X`参数可以支持其他动词。

```shell
curl -X DELETE www.example.com
```



### 7.文件上传

假定文件上传的表单是下面这样：

```html
<form method="POST" enctype='multipart/form-data' action="upload.cgi">
    <input type=file name=upload>
    <input type=submit name=press value="OK">
</form>
```

你可以用curl这样上传文件：

```shell
curl --form upload=@localfilename --form press=OK [URL]
```



### 8.reference字段

有时你需要在http request头信息中，提供一个referer字段，表示你是从哪里跳转过来的。

```shell
curl --referer http://www.example.com http://www.example.com
```



### 9.User-Agent

这个字段是用来表示客户端的设备信息。服务器有时会根据这个字段，针对不同设备，返回不同格式的网页，比如手机版和桌面版.

```shell
curl --user-agent "[User Agent]" [URL]
```



### 10.Cookie

使用`--cookie`参数，可以让curl发送cookie。

```shell
curl --cookie "name=xxx" www.example.com
```

至于具体的cookie的值，可以从http response头信息的`Set-Cookie`字段中得到。

`-c cookie-file`可以保存服务器返回的cookie到文件，`-b cookie-file`可以使用这个文件作为cookie信息，进行后续的请求。

```shell
curl -c cookies http://example.com
curl -b cookies http://example.com
```



### 11.增加头信息

有时需要在http request之中，自行增加一个头信息。`--header`参数就可以起到这个作用。

```shell
curl --header "Content-Type:application/json" http://example.com
```



### 12.HTTP认证

有些网域需要HTTP认证，这时curl需要用到`--user`参数。

```shell
curl --user name:password example.com
```

