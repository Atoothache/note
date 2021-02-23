redis的启动方式
1.直接启动
 进入redis根目录，执行命令:
 \#加上‘&’号使redis以后台程序方式运行

```shell
.``/redis-server` `&
```

 2.通过指定配置文件启动
 可以为redis服务启动指定配置文件，例如配置为/etc/redis/6379.conf
 进入redis根目录，输入命令：

```shell
.``/redis-server` `/etc/redis/6379``.conf
```

 \#如果更改了端口，使用`redis-cli`客户端连接时，也需要指定端口，例如：

```powershell
redis-cli -p 6380
```

**#检查后台进程是否正在运行**

ps -ef |grep redis

ps aux | grep redis

**#检测6379端口是否在监听**

netstat -lntp | grep 6379

**#使用配置文件启动redis服务**

./redis-server /etc/redis/redis.conf

**#使用`redis-cli`客户端检测连接是否正常**

./redis-cli -h 127.0.0.1 -p 6379  （登陆客户端）

auth (password) （密码验证）
cluster info （集群状态）

**#查看redis密码**

可查看 redis 安装根目录下的配置文件：redis-conf 中 requirepass 后面的内容。

启动redis：  /etc/init.d/redis start

关闭redis：  redis-cli shutdown