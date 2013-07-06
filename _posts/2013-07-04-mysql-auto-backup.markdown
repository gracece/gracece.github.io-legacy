---
author: admin
comments: true
date: 2013-07-04 13:01:25+00:00
layout: post
slug: mysql-auto-backup
title: mysql 数据库定时备份
description: 网站再小，数据也是无价的，做好数据的备份工作是对自己负责更是对用户负责。这里给出使用crontab定时执行备份脚本并备份到 `dropbox`的方法。
wordpress_id: 878
categories:
- linux
tags:
- mysql
- dropbox
---


网站再小，数据也是无价的，做好数据的备份工作是对自己负责更是对用户负责。这里给出使用crontab定时执行备份脚本并备份到 `dropbox`的方法。



`crontab`主要用于安排一些周期性进行的任务，可以灵活安排执行的时间频度，比如每周、每日、3点到6点每两小时一次等。 每一个用户都有自己的一个 crontab任务文件，放在`/var/spool/cron/crontabs`，当其中任务执行条件满足时系统就会 自动执行相关任务。crontab的任务形如：

    
      * * * * * command
  


分别代表分钟、小时、日、月、星期以及最后的命令。通过crontab命令我们可以添加新的任务或查看任务列表，具体的使用方法有下面几种：

    
    crontab [-u user] file  使用指定的文件覆盖crontab，谨慎使用。
    -e  (edit user's crontab) 使用编辑器编辑crontab文件，编辑器可选vim或nano等。
    -l  (list user's crontab) 列出当前用户的crontab文件内容。
    -r  (delete user's crontab) 删除
   


更详细的使用方法还请参考man手册，这里就不再赘述。当然，还有更偷懒的方法，在`/etc`下面的cron.hourly，cron.daily，cron.weekly，cron.weekly，这4个目录下脚本的执行周期 分别是每小时，每天，每周，每月执行的脚本。根据自己的需要，将要运行的脚本直接放置到对应 的目录下即可，系统会自动为你完成任务。

数据库备份使用 `mysqldump`来实现,编写`mysql_backup.sh` 如下：

    
    #!/bin/bash
    exdir='/root/Dropbox/myWebsite/sql/';
    extfilyWebsitee='m_'`date '+%F'`'.sql';
    echo `date '+%F_%R '`"Starting "$extfile;
    cd $exdir
    mysqldump --opt upload -u user -ppassword | gzip>$exdir$extfile.gz
    


用chmod将该脚本改为可执行，然后在crontab文件中加入

    
    * 1 * * * /usr/share/grace/mysql_backup.sh >/usr/share/grace/mysql_backup_log.txt
    50 0 * * * /usr/sbin/dropbox.py start 
    5 1 * * * /usr/sbin/dropbox.py stop
  


表示在每日1点执行相关脚本，并将输出保存到日志文件中。同时结合dropbox提供的脚本来控制dropbox的启动与停止，小内存的VPS 还是不要把dropbox一直挂在后台，能省则省吧。这样就能够实现每日备份数据库到dropbox。
