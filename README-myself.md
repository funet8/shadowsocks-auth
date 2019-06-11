
# 带有用户和流量统计的 shadowsocks

[github地址](https://github.com/Lupino/shadowsocks-auth)

[fork到自己的仓库](https://github.com/funet8/shadowsocks-auth)

当提供 shadowsocks 服务时，可以通过一个端口和用户的模式，服务于多个用户，而不必启用多个服务。 流量统计可以实时的查看用户的流量，并带有流量限制。
配套的 gui 客户端 [https://github.com/Lupino/shadowsocks-gui](https://github.com/Lupino/shadowsocks-gui)

# 安装golang和redis

```
# cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core) 
操作系统版本为centos7.6 64位

# yum install golang
# go version
go version go1.11.5 linux/amd64
go命令已经加入到了/usr/bin中，不用在加入环境变量中了
# whereis go
go: /usr/bin/go
```

#安装redis

```
yum install redis
systemctl restart redis
vi /etc/redis.conf 
```


## build

```
git clone https://github.com/Lupino/shadowsocks-auth.git
cd shadowsocks-auth
make
```

实际操作：
```
[root@VM_16_8_centos ~]# git clone https://github.com/Lupino/shadowsocks-auth.git
Cloning into 'shadowsocks-auth'...
remote: Enumerating objects: 163, done.
remote: Total 163 (delta 0), reused 0 (delta 0), pack-reused 163
Receiving objects: 100% (163/163), 37.82 KiB | 0 bytes/s, done.
Resolving deltas: 100% (70/70), done.
[root@VM_16_8_centos ~]# cd shadowsocks-auth
[root@VM_16_8_centos shadowsocks-auth]# ls
config.json  LICENSE.txt  local  Makefile  README.md  server
[root@VM_16_8_centos shadowsocks-auth]# make
cd server;go get -v -d; go build -o /bin/server
/bin/sh: go: command not found
/bin/sh: go: command not found
make: *** [all] Error 127

安装  yum install golang

```


## server

```
$(GOPATH)/bin/server -p 8388 -redis=127.0.0.1:6379

redis-cli
redis 127.0.0.1:6379> set ss:user:lupino "{\"name\": \"lupino\", \"password\": \"lup12345\", \"method\": \"aes-256-cfb\", \"limit\": 10737418240}"
redis 127.0.0.1:6379> keys "*"
 1) "ss:flow:lupino:2014:7:31"
 2) "ss:flow:lupino:2014:7:29"
 3) "ss:user:lupino"
 4) "ss:flow:lupino"

# redis keys 说明
ss:flow:[username] sorted set 用户当前的总流量
ss:flow:[username]:year:month:day sorted set 用户当天的流量
ss:user:[username] json 用户的信息
@name: 用户名
@password: 密码
@method: 加密方式
@limit: 最大限制流量

```

实际操作：
```
[root@VM_16_8_centos shadowsocks-auth]# $(GOPATH)/bin/server -p 8388 -redis=127.0.0.1:6379
-bash: GOPATH: command not found
2019/05/28 09:46:16 server listening port 8388 ...
[root@VM_16_8_centos ~]# go version
go version go1.11.5 linux/amd64
[root@VM_16_8_centos ~]# whereis go
go: /usr/bin/go

/bin/server -p 8388 -redis=127.0.0.1:6379

```


[server 端使用脚本](https://gist.github.com/Lupino/d0609ab79d873b2c1015)

## local

修改 config.json

```
{
    "server":"23.226.79.100",
    "server_port":8388,
    "local_port":1080,
    "password":"lupino:lup12345",  # 用户名:密码
    "method": "aes-256-cfb",
    "timeout": 600
}

$(GOPATH)/bin/local -c config.json
```
