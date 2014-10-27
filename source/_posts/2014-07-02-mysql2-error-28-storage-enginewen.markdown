---
layout: post
title: "Mysql2::Error: Got error 28 from storage engine问题解决"
date: 2014-07-02 10:35:17 +0800
comments: true
categories: 
- Linux
- MySQL

---

### 问题原因
今天发现MySQL数据库中没有写入新数据，于是看了下Rails后端的日志，发现报了一个`Error: Mysql2::Error: Got error 28 from storage engine: SHOW FULL FIELDS FROM plays`错误。网上搜了一通说是空间不足造成的。
<!--more-->

使用 `df -h` 命令，结果如下：

```
root@windstill:/# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/windstill-root
                       28G   28G    0G   0% /
none                   16G  220K   16G   1% /dev
none                   16G     0   16G   0% /dev/shm
none                   16G   52K   16G   1% /var/run
none                   16G     0   16G   0% /var/lock
/dev/mapper/windstill-data_backup
                       28G   16G   11G  59% /data_backup
/dev/mapper/windstill-mysql
                       65G   15G   47G  24% /mysql
/dev/mapper/windstill-opt
                       28G   15G   12G  55% /opt
/dev/mapper/windstill-redis
                       28G   17G  9.8G  63% /redis
/dev/mapper/windstill-prod
                       28G   13G   14G  50% /prod
/dev/mapper/windstill-mongodb
                       65G   50G   12G  82% /mongodb
                       
```    

可以看出 `/mysql` 空间还是很足的，还有47G空间可用，那是什么原因呢。随着深入了解，原来MySQL查询的时候会生成一些临时的表，在默认的配置里，这些表会被创建在 `/` 挂载点（目录）中。再结合上述结果中，我的 `/` 已经没有可用空间了，那这就是问题所在了。    


### 解决方法
既然知道问题的原因是 `/` 空间不足，那么解决方法就是清理 `/` 无用的文件。
使用 `ls -alh` 命令，结果如下：

```
-rw-r--r--  1 nobody   root     15G 2014-07-02 11:04 nginx.access.log
-rw-r--r--  1 nobody   root      2G 2014-07-02 11:04 nginx_assets.error.log
```
发现nginx的log文件都太大，于是使用 `rm` 命令把他们一一删除了。但是另外的一个小问题出现了，我再次使用 `df -h` 命令发现 `/` 可用空间仍然是0。

后来了解到，在Linux或者Unix系统中，通过rm或者文件管理器删除文件将会从文件系统的目录结构上解除链接(unlink)。然而如果文件是被打开的（有一个进程正在使用），那么进程将仍然可以读取该文件，磁盘空间也一直被占用。而我删除的是Nginx的log文件，删除的时候文件应该正在被使用。

通过 `lsof |grep deleted` 命令可以获得一个已经被删除但是仍然被应用程序占用的文件列表，如下所示：            

```
root@windstill:/# lsof |grep deleted
nginx      6316   nobody    3w      REG              251,4 15966896128    1471723 /tmp/nginx.access.log (deleted)
nginx      6316   nobody    4w      REG              251,4  2815307776    1471730 /tmp/nginx.error.log (deleted)
```

从输出结果可以看到log文件还在被使用，未被释放空间。一种方法是kill掉Nginx进程，让os自动回收磁盘空间。但这种方法显然不好，会影响线上应用。索性Nginx有个重新打开日志文件的方法：  
```
nginx -s reopen
```

至此，所有问题都解决了。

###参考文章
* [Got error 28 from storage engine (disk full) but my disk are not full](http://serverfault.com/questions/399068/got-error-28-from-storage-engine-disk-full-but-my-disk-are-not-full)
* [linux删除文件后没有释放空间](http://blog.csdn.net/wyzxg/article/details/4971843)
* [Nginx在Linux下的启动、停止和重加载](http://blog.csdn.net/dbanote/article/details/17169759)
