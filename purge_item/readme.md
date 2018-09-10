
## 准备
cd到当前目录下

```
nginx -p `pwd` -c nginx.conf -t
```
查看nginx.conf， 配置一个source站点，一个cache站点


启动
```
nginx -p `pwd` -c nginx.conf
```

测试源站
```
curl 127.0.0.1:10010/upload.txt -i
```

## 测试

测试cache站点
```
curl 127.0.0.1:10009/upload.txt -i
HTTP/1.1 200 OK
Server: openresty/1.13.6.1
X-Proxy-Cache: MISS
Accept-Ranges: bytes

upload txt content%

## 没有cache

curl 127.0.0.1:10009/upload.txt -i
HTTP/1.1 200 OK
Server: openresty/1.13.6.1
X-Proxy-Cache: HIT
Accept-Ranges: bytes

upload txt content%

## 已经被cache
```

查看cache文件
```
ls /tmp/cache1/0/c1/13d60411fbe40eb7ab53b698331d4c10
```

执行删除
```
curl -X PURGE 127.0.0.1:10009/upload.txt -i
HTTP/1.1 200 OK
Server: openresty/1.13.6.1
Date: Mon, 10 Sep 2018 09:38:48 GMT
Content-Type: text/plain
Transfer-Encoding: chunked
Connection: keep-alive

OK
```

查看cache文件目录
```
ls /tmp/cache1/0/c1

```

重新请求文件
```
$ curl 127.0.0.1:10009/upload.txt -i
HTTP/1.1 200 OK
Server: openresty/1.13.6.1
...
X-Proxy-Cache: MISS
Accept-Ranges: bytes

upload txt content%

$ curl 127.0.0.1:10009/upload.txt -i
HTTP/1.1 200 OK
...
X-Proxy-Cache: HIT
Accept-Ranges: bytes

upload txt content%

$ ls /tmp/cache1/0/c1/13d60411fbe40eb7ab53b698331d4c10
/tmp/cache1/0/c1/13d60411fbe40eb7ab53b698331d4c10
```

