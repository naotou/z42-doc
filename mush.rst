==================================================
蘑菇碎碎念
==================================================

vim黑科技
-----------------------

1. 遇到gbk乱码囧木办
   
    ::

        set encoding=utf-8
        set fenc=cp936
        set fileencodings=cp936,ucs-bom,utf-8
        if v:lang =~? '^\(zh\)\|\(ja\)\|\(ko\)'
            set ambiwidth=double
        endif
        set nobomb

#. 粘贴后格式错乱怎么办

   有的时候，在插入模式下从系统粘贴板粘贴文本到vim中会出现缩进异常的情况，为了解决这种问题，在粘贴前应该设置vim为粘贴模式并在粘贴完成后取消粘贴模式

   `:set paste`

   `:set nopaste`

vim插件及使用
-----------------------

1. syntastic 在错误之间跳转
   
    :lnext 跳到下一个

    :lprev 跳到上一个

#. 使用pyflakes进行语法检查 

    :SyntasticCheck pyflakes

iptables
-----------------------

1. 列出所有规则

   `iptables -nvL  -t nat --line-number`

   列出nat表的所有规则并显示行号

#. 删除

   `iptables -t nat -D DOCKER 13`

   删除nat表DOCKER链的第13行的规则

#. 用iptables给Docker添加端口映射 

   `iptables -t nat -A DOCKER --in-interface \!docker0 -p tcp --dport 6666 -j DNAT --to 172.17.0.5:6666`

   docker会在系统中创建一个叫docker0的网卡，本例中172.17.0.5就是docker0的IP地址

linux命令
-----------------------

ssh客户端配置文件
^^^^^^^^^^^^^^^^^^^^^^^

当主机较多的时候，不方便记住所有的IP、用户、端口以及密码，为了解决这个问题我们可以使用一个ssh的配置文件来记录这些服务器。

常用的配置有

    ::

        Host 主机别名
        HostName 主机地址
        User 登陆用户名
        Port 端口号
        IdentityFile 公钥 

在~/.ssh/目录下创建一个config文件，在config中写入相应的配置后就可以使用 `ssh \<主机别名\>` 直接连接服务器了

开发服务器环境介绍
-----------------------

开发服务器上通过使用docker来为每人提供一个独立的开发环境，通过主机上的nginx来将每人的域名分别通过反代指向他的docker。
我们使用了一个数据卷容器充当数据库文件目录，启动ssh供登陆开发.


添加一个新的开发docker
^^^^^^^^^^^^^^^^^^^^^^^

1. 启动一个数据卷容器
 
   `docker run -d -v /data --name \<your name\>_data pevc/data echo data_only for database`

#. 启动一个开发容器

   `docker run -d -i -p 9005:80 -p 10005:22 -p 8005:8000  --volumes-from \<your name\>_data --name \<your name\>_42web mush/ac /usr/sbin/sshd -D -f /etc/ssh/sshd_config`

   需要注意端口号，run之前先看下别人用了哪些端口了，一般就将端口号加一就行了。

#. 配置dns和主机的nginx反向代理

   在/etc/dnsmasq.conf解析你要使用的域名。

   ::

        address=/mushapi.info/192.168.10.169
        address=/*.mushapi.info/192.168.10.169

   在/etc/nginx/conf.d中加入你的反向代理配置。

   ::

        server {
            listen 80;
            server_name mushapi.info *.mushapi.info;
            location / {
                    proxy_pass http://127.0.0.1:9000;
                    proxy_set_header Host $host;
                    proxy_set_header X-Real-IP $remote_addr;
                    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
        } 
