这个脚本限制比较大

* proxy_cache_key 中 `$scheme$proxy_host` 要设置为 ""
* 操作系统要是 linux

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
▶ tree /tmp/cache1
/tmp/cache1
├── 1
│   └── 9b
│       └── 1a93233708000b0df0fbd74ea1e199b1
├── 7
│   └── 92
│       └── 87277ffa7dd67f24d964e6e2ae147927
└── e
    └── 5b
        └── 22f6cbe1f23cee38cfa17af0881575be
```


测试purge,每测一种情况都要先恢复到cache上面的状态

### purge 目录

```
▶ curl -X PURGE '127.0.0.1:10009/test/.*' -i
HTTP/1.1 200 OK
...
X-Purged-Count: 4

OK
```


### purge 文件匹配

```
curl -X PURGE '127.0.0.1:10009/test/upload.*' -i
```


```
curl -X PURGE '127.0.0.1:10009/test/.*txt' -i
```
