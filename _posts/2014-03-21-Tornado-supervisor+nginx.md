---
layout: post
title: "Tornado + Supervisord + Nginx 配置"
description: "简要记录如何在centos6上配置Nginx反向代理Tornado程序，同时使用supervisor配合守护进程。"
category: linux
tags:
- tornado
- nginx
- supervisor
---
{% include JB/setup %}

在项目中用到了tornado来写一个简单web程序，同时为了更加方便地管理，使用supervisor来守护管理tornado进程。
另外使用nginx作为tornado多进程多端口的反向代理，同时可以达到均衡负载的作用。下面简要记录配置过程。

### Tornado
参考此文件，使用Tornado提供的options，可以解析命参数，到时候在后面加 --port=8001 就能
按照指定端口执行，方便后面多进程。

    import tornado.httpserver
    import tornado.ioloop
    import tornado.options
    import tornado.web
    
    from tornado.options import define, options
    define("port", default=8000, help="run on the given port", type=int)
    
    class IndexHandler(tornado.web.RequestHandler):
        def get(self):
            greeting = self.get_argument('greeting', 'Hello')
            self.write(greeting + ', friendly user!')
    
    if __name__ == "__main__":
        tornado.options.parse_command_line()
        app = tornado.web.Application(handlers=[(r"/", IndexHandler)])
        http_server = tornado.httpserver.HTTPServer(app,xheaders=True) 
        http_server.listen(options.port)
        tornado.ioloop.IOLoop.instance().start()
        
### Supervisord
使用pip安装，安装好之后默认是没有配置文件的，需要自行添加，位置是在 `/etc/supervisord.conf` 可以使用
` echo_supervisord_conf >/etc/supervisord.conf` 生成默认配置，不过里面都是注释掉的。
配置内容修改参考下面，下面配置成Tornado开了四个进程，端口为 `8001-8004`

    [group:strapp]
    programs=strategy
    
    [program:strategy]
    command=python /home/strategydev/www/index.py --port=80%(process_num)02d
    directory=/home/strategydev/www
    process_name = %(program_name)s%(process_num)d
    autorestart=true
    redirect_stderr=true
    stdout_logfile=/home/strategydev/var/log/tornado.log
    stdout_logfile_maxbytes=500MB
    stdout_logfile_backups=50
    stderr_logfile=/home/strategydev/var/log/tornado.log
    loglevel=info
    numprocs = 4
    numprocs_start = 1
    
然后启动supervisord,使用控制台查看运行状况，如显示无误即可：

    [root@Backend_Server etc]# supervisorctl status
    strapp:strategy1                 RUNNING    pid 12528, uptime 0:02:14
    strapp:strategy2                 RUNNING    pid 12529, uptime 0:02:14
    strapp:strategy3                 RUNNING    pid 12530, uptime 0:02:14
    strapp:strategy4                 RUNNING    pid 12527, uptime 0:02:14

###Nginx
需要添加 upstream 并配反向代理

    upstream tornadoes {
    server 127.0.0.1:8001;
    server 127.0.0.1:8001;
    server 127.0.0.1:8002;
    server 127.0.0.1:8004;
    }
    
    server {
    listen 8888;
    server_name www.example.org *.example.org;


    location / {
        proxy_pass_header Server;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_pass http://tornadoes;
    }

配置完成重启nginx即可。

另外应该用iptable屏蔽8001-8004端口。

1.允许本地所有端口：

    -A INPUT -i lo -j ACCEPT

2.屏蔽外网访问8001～8002端口：

    -A INPUT -p tcp  --dport 8001:8004 -j REJECT

###开机启动
将下面保存为`/etc/rc.d/init.d/supervisord` 

    #!/bin/sh
    #
    # /etc/rc.d/init.d/supervisord
    #
    # Supervisor is a client/server system that
    # allows its users to monitor and control a
    # number of processes on UNIX-like operating
    # systems.
    #
    # chkconfig: - 64 36
    # description: Supervisor Server
    # processname: supervisord
     
    # Source init functions
    . /etc/rc.d/init.d/functions
     
    prog="supervisord"
     
    prefix="/usr/"
    exec_prefix="${prefix}"
    prog_bin="${exec_prefix}/bin/supervisord"
    PIDFILE="/var/run/$prog.pid"
     
    start()
    {
            echo -n $"Starting $prog: "
            daemon $prog_bin --pidfile $PIDFILE
            retval=$?
            echo
            [ $retval -eq 0 ] && [ -f $PIDFILE ]
            return $retval
    }
     
    stop()
    {
            echo -n $"Shutting down $prog: "
            [ -f $PIDFILE ] && killproc $prog || success $"$prog shutdown"
            echo
    }
     
    case "$1" in
     
      start)
        start
      ;;
     
      stop)
        stop
      ;;
     
      status)
            status $prog
      ;;
     
      restart)
        stop
        start
      ;;
     
      *)
        echo "Usage: $0 {start|stop|restart|status}"
      ;;
     
    esac


然后 

    chmod +x /etc/rc.d/init.d/supervisord
    chkconfig --add supervisord #加为服务

运行ntsysv，选中supervisord启动系统时跟着启动。


### 参考

- [xiangjian]( http://blog.xiangjian.info/2011/08/deploy_tornado_with_supervisor_nginx.html)
- [serholiu](http://serholiu.com/tornado-nginx-supervisord)
- [keakon](http://www.keakon.net/2012/12/17/%E7%94%9F%E4%BA%A7%E7%8E%AF%E5%A2%83%E4%B8%8B%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E5%9C%B0%E9%87%8D%E5%90%AFTornado)
- [pythoner](http://demo.pythoner.com/itt2zh/ch8.html)
- [au92](http://www.au92.com/archives/tornado-get-remote-ip-address-complement.html)


### 2014-03-22补充一键脚本

    #/bin/bash
    # 2014-03-22
    yum install python-pip
    yum install python-devel
    yum install nginx
    pip install tornado
    pip install supervisor

    wget -O /var/www/index.py https://gist.githubusercontent.com/gracece/21e5719b234929799eeb/raw/index.py
    wget -O /etc/supervisord.conf https://gist.githubusercontent.com/gracece/21e5719b234929799eeb/raw/supervisord.conf
    wget -O /etc/nginx/conf.d/tornado.conf https://gist.githubusercontent.com/gracece/21e5719b234929799eeb/raw/tornado.conf 

    supervisord
    /etc/init.d/nginx restart
    supervisorctl reload all
    supervisorctl status

    iptables -A INPUT -i lo -j ACCEPT
    iptables -A INPUT -p tcp  --dport 8001:8004 -j REJECT
    /etc/init.d/iptables save

    wget -O /etc/rc.d/init.d/supervisord https://gist.githubusercontent.com/gracece/21e5719b234929799eeb/raw/supervisord
    chmod +x /etc/rc.d/init.d/supervisord
    chkconfig --add supervisord
    ntsysv

以上为粗放型安装Tornado+supervisor+nginx 脚本，适用于centos6,其他系统可能无法正常运行，默认端口为8888,nginx反向代理8001-8004 四个进程，
默认py文件为/var/www/index.py ，如果需要更改，请修改/etc/supervisord.conf 里对应文件夹即可， 然后运行 supervisorctl reload all 重启进程。
