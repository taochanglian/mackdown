1.在redis安装目录中，有一个utils文件夹，有一个redis_init_script脚本。将redis_init_script脚本copy到linux系统的/etc/init.d目录中，将redis_init_script重命名为redis_6379,6379使我们希望这个redis实例监听的端口号。
```shell
[root@vm-linux-161 redis-3.2.11]# pwd
/usr/local/redis-3.2.11
[root@vm-linux-161 redis-3.2.11]# cp utils/redis_init_script /etc/init.d
[root@vm-linux-161 redis-3.2.11]# cd /etc/init.d/
[root@vm-linux-161 init.d]# ls
activity-analyzer  ambari-agent  ambari-server  functions  hst  hst-gateway  netconsole  network  README  redis_init_script  vmware-tools
[root@vm-linux-161 init.d]# mv redis_init_script redis_6379
[root@vm-linux-161 init.d]# ls
activity-analyzer  ambari-agent  ambari-server  functions  hst  hst-gateway  netconsole  network  README  redis_6379  vmware-tools
```

2.修改redis_6379脚本的第六行的REDISPORT，设置为相同的端口号(默认就是6379)
```shell
[root@vm-linux-161 init.d]# cat redis_6379 
#!/bin/sh
#
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.

REDISPORT=6379
...
...
...
```
可以看到，默认就是6379.当然，如果你的redis需要使用其他端口号，那么就需要修改它。

3.创建两个文件夹:/etc/redis (存放redis的配置文件)，/var/redis/6379 (存放redis的持久化文件)
```shell
[root@vm-linux-161 ~]# mkdir -p /etc/redis
[root@vm-linux-161 ~]# mkdir -p /var/redis/6379
```

4.修改redis的配置文件(在redis安装目录中的redis.conf)，copy到/etc/redis文件夹中。修改内容如下：
daemonize	yes    让redis以daemon进行运行
pidfile		/var/run/redis_6379.pid    设置redis的pid文件位置
port		6379    设置redis的监听端口号
dir         /var/redis/6379    设置持久化文件的存储位置 

```shell
[root@vm-linux-161 ~]# cd /usr/local/redis-3.2.11
[root@vm-linux-161 redis-3.2.11]# ls
00-RELEASENOTES  BUGS  CONTRIBUTING  COPYING  deps  INSTALL  Makefile  MANIFESTO  README.md  redis.conf  runtest  runtest-cluster  runtest-sentinel  sentinel.conf  src  tests  utils
[root@vm-linux-161 redis-3.2.11]# cp redis.conf /etc/redis/
[root@vm-linux-161 redis-3.2.11]# cd /etc/redis/
[root@vm-linux-161 redis]# ls
redis.conf
[root@vm-linux-161 redis]# vi redis.conf
```
修改内容较多，也可以先down到本地，通过文本编辑器编辑。
```file
# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
daemonize yes

# Creating a pid file is best effort: if Redis is not able to create it
# nothing bad happens, the server will start and run normally.
pidfile /var/run/redis_6379.pid

# Accept connections on the specified port, default is 6379 (IANA #815344).
# If port 0 is specified Redis will not listen on a TCP socket.
port 6379


# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
dir /var/redis/6379

```
最后，重命名为6379.conf
```shell
[root@vm-linux-161 redis]# pwd
/etc/redis
[root@vm-linux-161 redis]# ls
redis.conf
[root@vm-linux-161 redis]# mv redis.conf 6379.conf
[root@vm-linux-161 redis]# ls
6379.conf
```

5. 启动redis
进入/etc/init.d文件夹，给redis_6379文件赋权，然后启动它
```shell
[root@vm-linux-161 ~]# cd /etc/init.d/
[root@vm-linux-161 init.d]# ls
activity-analyzer  ambari-agent  ambari-server  functions  hst  hst-gateway  netconsole  network  README  redis_6379  vmware-tools
[root@vm-linux-161 init.d]# chmod 777 redis_6379 
[root@vm-linux-161 init.d]# ./redis_6379 start
Starting Redis server...
```

确认是否启动：
```shell
[root@vm-linux-161 init.d]# ps -ef|grep redis
root     10214     1  0 17:03 ?        00:00:00 /usr/local/bin/redis-server 127.0.0.1:6379
root     10362  2115  0 17:03 pts/1    00:00:00 grep --color=auto redis
```

6.让redis自动启动
在/etc/init.d 文件夹中，修改redis_6379文件，在最上面，加入两行注释,加入的内容如下：
```text
#chkconfig:		2345  90  10
#description:	Redis is a persistent key-value database

```
使用chkconfig命令，让脚本开机启动
chkconfig redis_6379 on