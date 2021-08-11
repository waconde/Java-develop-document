# centos 创建systemd 
摘要： - Centos7开机第一个程序从init完全换成了systemd这种启动方式，同centos 5 6已经是实质差别。systemd是靠管理unit的方式来控制开机服务，开机级别等功能。 在/usr/lib/systemd/system目录下包含了各种unit文件，有service后缀的服务unit，有target后缀的开机级别unit等，这里临时介绍关于service后缀的文件。

Centos7开机第一个程序从init完全换成了systemd这种启动方式，同centos 5 6已经是实质差别。systemd是靠管理unit的方式来控制开机服务，开机级别等功能。

在/usr/lib/systemd/system目录下包含了各种unit文件，有service后缀的服务unit，有target后缀的开机级别unit等，这里临时介绍关于service后缀的文件。因为systemd在开机要想执行自启动，都是通过这些*.service 的unit控制的

具体流程

在/usr/lib/systemd/system目录下新建一个 service-name.service的文件
以apache的httpd.service的unit为例解释


```
[Unit]
#定义描述
Description=The Apache HTTP Server 
#指定了在systemd在执行完那些target之后再启动该服务
After=network.target remote-fs.target nss-lookup.target
 
[Service]
#定义Service 的运行type，一般是forking，就是后台运行
Type=notify
Environment=LANG=C
#以下定义systemctl start |stop |reload *.service  的每个执行方法，具体命令#需要写绝对路径
ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
# Send SIGWINCH for graceful stop
KillSignal=SIGWINCH
KillMode=mixed
#创建私有的内存临时空间
PrivateTmp=true
 
[Install]
WantedBy=multi-user.target

```
