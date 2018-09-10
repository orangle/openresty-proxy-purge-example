这个脚本限制比较大

* proxy_cache_key 中 `$scheme$proxy_host` 要设置为 ""
* 操作系统要是 linux

主要原理就是查找nginx cache文件的header中的 `KEY: /test/upload.txt` 信息，通过这个信息来找到文件并删除。应该比较影响性能。

### 准备

启动
```
nginx -p `pwd` -c nginx.conf
```

访问源站
```
curl 127.0.0.1:10010/test/test.txt
curl 127.0.0.1:10010/test/upload.json
curl 127.0.0.1:10010/test/upload.txt
```

### 测试

三个文件访问一遍，确认已经被cache(判断cache一个是从reponse的header，另一个方式是查看缓存目录)

```
curl 127.0.0.1:10009/test/test.txt -i
curl 127.0.0.1:10009/test/upload.json -i
curl 127.0.0.1:10009/test/upload.txt -i
```

查看cache目录
```
$ tree /tmp/cache1/
/tmp/cache1/
├── 1
│   └── 9b
│       └── 1a93233708000b0df0fbd74ea1e199b1
├── 7
│   └── 92
│       └── 87277ffa7dd67f24d964e6e2ae147927
└── e
    └── 5b
        └── 22f6cbe1f23cee38cfa17af0881575be

6 directories, 3 files
```


测试purge,每测一种情况都要先恢复到cache上面的状态

### purge 目录

```
$ curl -X PURGE '127.0.0.1:10009/test/.*' -i
HTTP/1.1 200 OK
Server: openresty/1.11.2.3
Date: Mon, 10 Sep 2018 14:15:57 GMT
Content-Type: text/plain; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
X-Purged-Count: 3

OK

$ tree /tmp/cache1/
/tmp/cache1/
├── 1
│   └── 9b
├── 7
│   └── 92
└── e
    └── 5b
```


### purge 文件匹配

```
$ curl -X PURGE '127.0.0.1:10009/test/upload.*' -i
HTTP/1.1 200 OK
Server: openresty/1.11.2.3
Date: Mon, 10 Sep 2018 14:16:59 GMT
Content-Type: text/plain; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
X-Purged-Count: 2

OK

$ tree /tmp/cache1/
/tmp/cache1/
├── 1
│   └── 9b
├── 7
│   └── 92
└── e
    └── 5b
        └── 22f6cbe1f23cee38cfa17af0881575be
```

按照文件格式

```
$ curl -X PURGE '127.0.0.1:10009/test/.*txt' -i
HTTP/1.1 200 OK
Server: openresty/1.11.2.3
Date: Mon, 10 Sep 2018 14:17:47 GMT
Content-Type: text/plain; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
X-Purged-Count: 2

OK

$ tree /tmp/cache1/
/tmp/cache1/
├── 1
│   └── 9b
│       └── 1a93233708000b0df0fbd74ea1e199b1
├── 7
│   └── 92
└── e
    └── 5b

$ cat /tmp/cache1/1/9b/1a93233708000b0df0fbd74ea1e199b1
X[y[}[7ìý¾«
                 "5b967995-b"
KEY: http://127.0.0.1:10010/test/upload.json
HTTP/1.1 200 OK
Server: openresty/1.11.2.3
Date: Mon, 10 Sep 2018 14:17:36 GMT
Content-Type: text/plain
Content-Length: 11
Last-Modified: Mon, 10 Sep 2018 14:03:01 GMT
Connection: close
ETag: "5b967995-b"
Accept-Ranges: bytes

upload.json
```
