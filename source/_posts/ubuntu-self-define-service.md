title: Ubuntu中自定义服务
tags: [Shell,Ubuntu]
date: 2015-08-10 15:18:29
description: 在Ubuntu中定义一些服务，可以对该服务进行全局控制
---

在Ubuntu中定义一些服务，可以对该服务进行全局控制，比较常见的就是用sudo service `servcie_name` start/stop/restart 进行服务的启停

# 在/etc/init.d/目录下面定义控制服务的脚本

```bash
service_name="service_name"
service_script_path="service_script_path"

start()
{
    echo "$service_name service start"
    chmod a+x $service_script_path && nohup ./$service_script_path 
}

stop()
{
    echo "$service_name service stop"
    server_process_id=`ps -aux | grep $service_name | cut -d " " -f 6`
    echo "server_process_id="server_process_id
    kill -9 $server_process_id
}

case $1 in
start)
    start
    ;;
stop)
    stop
    ;;
restart)
    echo "resert the $service_name"
    stop
    start
    ;;
*)
    echo "undefine operation"
    ;;
esac
```

<font color="#FF0000">版权声明：本文为博主原创文章，转载请注明出处</font>