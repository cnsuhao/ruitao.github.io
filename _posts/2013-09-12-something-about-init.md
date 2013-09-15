---
layout: post
title: init的那点事
---

# {{ page.title }}

## init的各种开源实现

在[http://www.ruanyifeng.com/blog/2013/08/linux_boot_process.html](Linux的启动流程) 里面，其实 `/sbin/init` 是有各种开源实现的，比如很久以前（其实也不算久啦）的 `sysvinit`，到现在的各大发行版用的 [http://upstart.ubuntu.com](upstart)、 [http://www.freedesktop.org/wiki/Software/systemd/](systemd)

简单介绍一下upstart的NB之处，先看看upstart官方介绍特性:

* Tasks and Services are started and stopped by events
* Events are generated as tasks and services are started and stopped
* Events may be received from any other process on the system
* Services may be respawned if they die unexpectedly
* Supervision and respawning of daemons which separate from their parent process
* Communication with the init daemon over D-Bus
* User services, which users can start and stop themselves

`event-based` 是它最大的亮点，比如，我们可以只在系统检测到 `iPod` 连接到计算机的时时候才启动 `podcast` 服务
更详细的信息，可以认真看下 [http://archive09.linux.com/articles/57213]()

各种init实现之前的对比，可以参看Gentoo Wiki的 [http://wiki.gentoo.org/wiki/Talk:Comparison_of_init_systems](Comparison of init systems)

## 编写upstart脚本运行程序

安装就不用说了，直接上实例
这是一个典型的upstart配置脚本，存放在 `/etc/init/godaemon.conf` (假设我们的服务叫godaemon)
我们用来跑一些本身没有实现守护进程模式的（或者懒得实现）

```bash
# Ubuntu upstart file at /etc/init/godaemon.conf

limit nofile 65532
kill timeout 300 # wait 300s between SIGTERM and SIGKILL.

start on runlevel [2345]
stop on runlevel [06]

pre-start script
    mkdir -p /var/log/godaemon
    chmod 777 /var/log/godaemon
end script

respawn
respawn limit 5 30

chdir /data/bin/godaemon
setuid www-data
setgid adm
console log

script
    exec ./godaemon
end script

post-start script
    PID=`pidof godaemon`
    echo $PID > /var/run/godaemon.pid
end script

post-stop script
    rm -f /var/run/godaemon.pid
end script
```
