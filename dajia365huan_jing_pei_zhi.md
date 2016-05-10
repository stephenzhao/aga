# dajia365环境配置

下面以用户名`linhaote`为例说明。
### 1. 登录开发机
`ssh linhaote@192.168.1.119`

### 2. 配置无密码登录开发机

```sh
cd ~
mkdir .ssh
chmod 700 .ssh
touch .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
vim .ssh/authorized_keys
# 把id_rsa.pub里面的公钥内容粘贴到里面即可
```

### 3. 配置oschina公钥
进入<https://git.oschina.net/profile/sshkeys>，  
然后把公钥内容粘贴到里面，点击添加公钥完成。

### 4. 部署代码
在本地得代码:  
`git clone git@git.oschina.net:youwe/dajia365-api.git`

同步到开发机:  
`rsync -rav -e ssh --exclude='*.git' ~/youwei/dajia365-api linhaote@192.168.1.119:/home/linhaote/`

为了方便开发，可以配置iNotify。
如果你使用sublime开发，可以用sftp插件。

### 5. 配置php-fpm
`vim /usr/local/php/etc/haote.dajia365.com.php-fpm.conf`

一般来说, 每个开发只配置一个单独的php-fpm即可, 自己域名下的服务都指向自己的同一个php-fpm.

```
[global]
pid = run/haote.dajia365.com-php-fpm.pid
log_level = error
error_log = /var/log/haote.dajia365.com-php-fpm-error.log

[haote.dajia365.com]
listen = /tmp/haote.dajia365.com-php-cgi.sock
user = www
group = www
pm = static
pm.max_children = 4
pm.start_servers = 4
pm.min_spare_servers = 4
pm.max_spare_servers = 4
request_terminate_timeout = 0

rlimit_files = 51200

listen.owner = www
listen.group = www
listen.mode = 0666

pm.status_path = /status
ping.path = /ping
ping.response = pong
slowlog = /var/log/haote.dajia365.com-php-fpm-slow.log
request_slowlog_timeout = 1


php_flag[display_errors] = on
```

### 6. 配置nginx
`vim /etc/nginx/vhost/api.haote.dajia.com`  
配置如下：

```
server
{
        listen       80;
        server_name api.haote.dajia365.com;
        index index.php;

        set $path '/home/linhaote/dajia365-api/public';

        root  $path;

        location / {
                root  $path;
                if (!-e $request_filename) {
                        rewrite ^/(.*)  /index.php?$1 last;
                        break;
                }
        }


        location ~ .*\.php$
        {
                #fastcgi_pass   127.0.0.1:9000;
                fastcgi_pass  unix:/tmp/php-cgi.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME  $path$fastcgi_script_name;
                include        fastcgi_params;
        }


        access_log /var/log/haote.dajia365.com-access.log main; #access_log end
        error_log /var/log/haote.dajia365.com-error.log crit; #error_log end
}
```

### 7. 开发机上设置/etc/hosts
配置hosts是为了让dajia365-api项目能够成功访问到dajia365-web提供用户中心服务

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

127.0.0.1 haote.dajia365.com api.haote.dajia365.com
```

### 8. phpwind论坛添加id
访问<http://bbs.dajia.com/youwei-admin.php>,   
使用admin/123456登录后台, 在设置->创始人->客户端管理那里添加自己的客户端.

![Windid客户端配置](http://git.oschina.net/uploads/images/2015/0609/135003_13db1c8e_324502.png "Windid客户端配置")

### 9. 拷贝 dajia365-web/public/sync/windid_client/conf/config.dev.php, 

改名为config.php, 把论坛的clientId和key填到config.php里

```
<?php
return array( 
	'connect' => 'db', 									//db为本地连接  http远程连接  如为db，请同时配置database.php里的数据库设置
	'serverUrl' => 'http://bbs.dajia.com/windid',			//服务端访问地址. 如:http://www.phpwind.net
	'clientId' => '6', 									//该客户端在WindID里的id
	'clientKey' => '8373fae40be6ea1ed92dafae85da8dc6',							//通信密钥，请保持与WindID里的一致
	'charset' => 'utf8',								//客户端使用的字符编码
);
?>
```

### 10. 拷贝 dajia365-web/public/sync/windid_client/conf/database.dev.php, 改名为database.php

### 11. 拷贝 dajia365-web/public/sync/config.inc.dev.php,改名为config.inc.php

### 12. 配置发短信IP
因为开发环境没有静态IP, 所以遇到提示无权限发送短信的时候, 请联系王毅