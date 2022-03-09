# Centos8 环境下 Nginx-Rtmp 安装配置 #

### 注意看我的标题！！！！我这是针对 1.20.2 版本 ###

> 偶数版本-稳定版本（两个稳定版本之间的跨越时间越长、补丁发布频率越少），奇数版本-开发、测试版本

## 一、检查本地是否安装 ##

### 1、检查 ###

```shell
rpm -qa | grep nginx
```

### 2、卸载 ###

```shell
rpm -e 已经存在的Nginx全名
```

#### 切换到下载目录下 ####

```shell
cd /usr/local/
```

### 3、下载 ###

```shell
wget https://nginx.org/download/nginx-1.20.2.tar.gz
```

### 4、解压（找到压缩文件） ###

```shell
tar -zxvf nginx-1.20.2.tar.gz  -C /usr/local/
```

## 二、Nginx 调优 ##

> 修改源代码文件

### 1、隐藏版本信息 ###

```shell
vim /usr/local/nginx-1.20.2/src/core/nginx.h
```

#### 按一下键盘字母`i`进行编辑 ####

#### 修改以下内容（13行 显示的版本号） ####

```shell
#define NGINX_VERSION      "996"
```

#### 修改以下内容（14行 显示的软件名） ####

```shell
#define NGINX_VER          "learning/" NGINX_VERSION
```

#### 修改以下内容（22行 显示的软件名） ####

```shell
#define NGINX_VAR          "learning"
```

#### 按一下`esc`键 退出编辑 ####

#### `:wq` 保存退出 ####

### 2、隐藏软件名 ###

```shell
vim /usr/local/nginx-1.20.2/src/http/ngx_http_header_filter_module.c
```

#### 按一下键盘字母`i`进行编辑 ####

#### 修改以下内容（49行 修改为想要显示的软件名） ####

```shell
static u_char ngx_http_server_string[] = "Server: learning" CRLF;
```

#### 按一下`esc`键 退出编辑 ####

#### `:wq` 保存退出 ####

### 3、定义对外展示内容 ###

```shell
vim /usr/local/nginx-1.20.2/src/http/ngx_http_special_response.c
```

#### 按一下键盘字母`i`进行编辑 ####

#### 修改以下内容（22行 此行定义对外展示的内容） ####

```shell
"<hr><center>" NGINX_VER "(learning)</center>" CRLF
```

#### 修改以下内容（36行 此行定义对外展示的软件名） ####

```shell
"<hr><center>learning</center>" CRLF
```

#### 按一下`esc`键 退出编辑 ####

#### `:wq` 保存退出 ####

> 配置Linux系统参数

### 4、修改文件打开数限制 ###

> 操作系统默认单进程最大打开文件数为1024

```shell
vim /etc/security/limits.conf
```

#### 按一下键盘字母`i`进行编辑 ####

#### 新增以下内容 ####

> *号表示所用用户

```shell
* soft nofile 65535
* hard nofile 65535
```

#### 按一下`esc`键 退出编辑 ####

#### `:wq` 保存退出 ####

### 5、设置 login ###

```shell
vim /etc/pam.d/login
```

#### 按一下键盘字母`i`进行编辑 ####

#### 新增以下内容 ####

> `/lib/security/pam_limits.so` 这个路径根据实际情况填写，32位系统是 `/lib` 下 64位系统是 `/lib64` 下

```shell
session    required     /lib/security/pam_limits.so
```

#### 按一下`esc`键 退出编辑 ####

#### `:wq` 保存退出 ####

### 6、设置 DNS缓存 ###

> 开启 NSCD 服务，缓存 DNS，提高域名解析响应速度

```shell
vim /etc/resolv.conf
```

#### 按一下键盘字母`i`进行编辑 ####

#### 新增以下内容 ####

```shell
ststemetl start nscd.service    
ststemetl enable nscd.service
```

#### 按一下`esc`键 退出编辑 ####

#### `:wq` 保存退出 ####

## 三、下载 Rtmp 插件

### 1、安装 Git ### 

#### 切换到下载目录下 ####

```shell
cd /usr/local/
```

#### 下载安装 ####

```shell
yum -y install git
```

### 2、下载 Rtmp ###

```shell
git clone https://github.com/arut/nginx-rtmp-module.git
```

## 三、编译安装 ##

### 1、下载依赖 ###

> 缺少 xxxlibrany，缺少.c.h文件 安装开发组包 xxx-devel

```shell
yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel pcre pcre-devel
```

### 2、切换到解压目录下 ###

```shell
cd /usr/local/nginx-1.20.2
```

### 3.配置 RTMP 实现 RTMP 直播推流效果 ###

```shell
./configure --prefix=/usr/local/nginx --user=nginx --group=nginx --pid-path=/usr/local/nginx/log/nginx.pid --error-log-path=/usr/local/nginx/log/error.log --http-log-path=/usr/local/nginx/log/access.log --with-http_gzip_static_module --http-client-body-temp-path=/usr/local/nginx/temp/client --http-proxy-temp-path=/usr/local/nginx/temp/proxy --http-fastcgi-temp-path=/usr/local/nginx/temp/fastcgi --http-uwsgi-temp-path=/usr/local/nginx/temp/uwsgi --http-scgi-temp-path=/usr/local/nginx/temp/scgi --add-module=/usr/local/nginx-rtmp-module --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_gzip_static_module --with-http_addition_module --with-http_flv_module --with-http_mp4_module --with-http_stub_status_module --with-pcre --with-http_sub_module
```

### 4、执行编译操作 ###

> -j8 多线程操作，加速编译

```shell
make -j8
```

### 5、执行安装操作 ###

```shell
make install -j8
```

### 6、创建临时文件存放路径 ###

#### 进入安装路径 ####

```shell
cd /usr/local/nginx
```

#### 创建 一级 ####

```shell
mkdir temp
```

#### 进入 一级 路径 ####

```shell
cd /usr/local/nginx/temp
```

#### 创建 二级 ####

```shell
mkdir client proxy fastcgi uwsgi scgi
```

### 7、创建 用户，用户组 ###

#### 用户组 ####

```shell
groupadd nginx_group
```

#### 用户 ####

```shell
useradd -g nginx_group nginx
```

## 四、Nginx 运行维护 ##

### 进入安装路径 ###

```shell
cd /usr/local/nginx
```

### 启动 ###

```shell
./sbin/nginx
```

### 查看是否有 Nginx ###

```shell
ps -ef | grep nginx
```

### 重载 ###

```shell
./sbin/nginx -s reload
```

### 关闭 ###

```shell
./sbin/nginx -s stop
```

#### 优雅关闭（当请求被处理完成之后才关闭） ###

```shell
./sbin/nginx -s quit
```

## 五、防火墙端口开放 ##

##### 1、查看防火墙状态 #####

```shell
firewall-cmd --state
```

##### 2、关闭防火墙 #####

```shell
systemctl stop firewalld.service
```

##### 3、禁止防火墙开机自启 #####

```shell
systemctl disable firewalld.service
```

##### 4、查看所有已开放的临时端口 #####

```shell
firewall-cmd --list-ports
```

##### 5、开启防火墙 #####

```shell
systemctl start firewalld.service
```

##### 6、Add 添加开放端口 #####

```shell
firewall-cmd --permanent --zone=public --add-port=80/tcp
```

##### 7、Reload 重新加载 #####

```shell
firewall-cmd --reload
```

##### 8、检查是否生效 ######

```shell
firewall-cmd --zone=public --query-port=80/tcp
```

##### 9、删除开放端口 ######

```shell
firewall-cmd --zone=public --remove-port=8088/tcp --permanent
```

## 六、开机自启配置 ##

### 1、在系统服务目录里创建`nginx.service`文件 ###

```shell
vim /usr/lib/systemd/system/nginx.service
```

#### 按一下键盘字母`i`进行编辑 ####

### 2、输入以下内容 ###

```shell
[Unit]
Description=nginx
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

#### 按一下`esc`键 退出编辑 ####

#### `:wq` 保存退出 ####

### 3、设置开机自启动 ###

```shell
systemctl enable nginx.service
```

### 4、重启计算机 ###

```shell
shutdown -r now
```

### 5、查看 Nginx 状态 ###

```shell
systemctl status nginx.service
```

### 6、查看是否有 Nginx ###

```shell
ps -ef | grep nginx
```

## 七、配置 ##

### 编辑默认配置文件 ###

#### 查看CPU核数（记下来留作备用） ####

```shell
grep -c processor /proc/cpuinfo
```

#### 编辑 `nginx.conf` 配置文件 ####

```shell
vim nginx.conf
```

#### 按一下键盘字母`i`进行编辑 ####

#### 修改为以下内容： ####

```shell
# 1、指定Nginx服务的用户和用户组
user nginx nginx;
# 2、设置 Worker 进程数
# 工作进程数指令；指令值有两种类型，分别为数字和 auto。指令值为 auto 时，Nginx 会根据 CPU 的内核数生成等数量的工作进程。
# worker_processes auto;
# 工作进程 CPU 绑定指令；指令值除了可以是 CPU 掩码外，还可以是 auto。当指令值为 auto 时，Nginx 会自动进行 CPU 绑定。
# worker_cpu_affinity auto;
# #2核CPU的配置
worker_processes  2;
worker_cpu_affinity 01 10;
# #4核CPU的配置
# worker_processes  4;
# worker_cpu_affinity 0001 0010 0100 1000;
# #8核CPU的配置
# worker_processes 8;
# worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 1000000;
# 3、工作进程最大打开文件数，可设置为优化后的文件打开数限制
worker_rlimit_nofile 65535;
# 4、事件处理
events {
    # 在Linux下，Nginx使用 epoll 的I/O多路复用模型（支持的事件模型有 select、poll、kqueue、epoll、/dev/poll、eventport。）
    use epoll;
    # Linux 系统下，因为每个网络连接都将打开一个文件描述符，Nginx 可处理的并发连接数受限于操作系统的最大打开文件数，同时所有工作进程的并发数也受 worker_rlimit_nofile 指令值的限制。
    # 进程的最大连接数受 Linux 系统进程的最大打开文件数限制，只有修改最大打开文件数限制之后，worker_connections 才能生效
    # Nginx总并发连接数 = worker_processes * worker_connections。总数保持在 3w 左右即可。
    worker_connections 15000;
}
# 5、RTMP
rtmp {
	server {
		listen 1935; 
		chunk_size 4000;
		
		# 直播链接 测试  推流地址 ： rtmp://IP:1935/e-learning
        application e-learning {
			# 开启实时
			live on;
			# 这个参数把直播服务器改造成实时回放服务器。
			hls on;
			# 对视频切片进行保护，这样就不会产生马赛克了。
			wait_key on;
			# 切片视频文件存放位置。
			hls_path /usr/local/nginx/html/e-learning;
			# 每个视频切片的时长。
			hls_fragment 10s;
			# 总共可以回看的事件，这里设置的是1分钟。
			hls_playlist_length 60s;
			# 连续模式。
			hls_continuous on;
			# 对多余的切片进行删除。
			hls_cleanup on;
			# 嵌套模式。
			hls_nested on;
        }
        
        # 直播链接 1  推流地址 ： rtmp://IP:1935/e-learning/live_1
        application e-learning/live_1 {
			# 开启实时
			live on;
			# 这个参数把直播服务器改造成实时回放服务器。
			hls on;
			# 对视频切片进行保护，这样就不会产生马赛克了。
			wait_key on;
			# 切片视频文件存放位置。
			hls_path /usr/local/nginx/html/e-learning/live_1;
			# 每个视频切片的时长。
			hls_fragment 10s;
			# 总共可以回看的事件，这里设置的是1分钟。
			hls_playlist_length 60s;
			# 连续模式。
			hls_continuous on;
			# 对多余的切片进行删除。
			hls_cleanup off;
			# 嵌套模式。
			hls_nested on;
        }
        
        # 直播链接 2  推流地址 ： rtmp://IP:1935/e-learning/live_2
        application e-learning/live_2 {
			# 开启实时
			live on;
			# 这个参数把直播服务器改造成实时回放服务器。
			hls on;
			# 对视频切片进行保护，这样就不会产生马赛克了。
			wait_key on;
			# 切片视频文件存放位置。
			hls_path /usr/local/nginx/html/e-learning/live_2;
			# 每个视频切片的时长。
			hls_fragment 10s;
			# 总共可以回看的事件，这里设置的是1分钟。
			hls_playlist_length 60s;
			# 连续模式。
			hls_continuous on;
			# 对多余的切片进行删除。
			hls_cleanup off;
			# 嵌套模式。
			hls_nested on;
        }
    }
}
# 5、HTTP 核心配置指令
http {
    include mime.types;
    # 配置在 http 区块，默认是 512kb ，一般设置为 cpu 一级缓存的 4-5 倍，一级缓存大小可以用 lscpu 命令查看
    server_names_hash_bucket_size 512;
    # 默认类型
    default_type application/octet-stream;
    # 默认编码格式
    charset utf-8;
    # 日志格式设置为Json格式，方便ELK日志分析，注意“使用日志中时间替换掉@timestamp默认的时间”配置要一致
    log_format main_json '{"status":"$status",'
                     '"log_date":"$time_iso8601",'
                     '"client_ip":"$remote_addr",'
                     '"request":"$request",'
                     '"request_time":"$request_time",'
                     '"size":"$body_bytes_sent",'
                     '"upstream_addr":"$upstream_addr",'
                     '"upstream_status":"$upstream_status",'
                     '"upstream_time":"$upstream_response_time",'
                     '"url":"$uri",'
                     '"http_referrer":"$http_referer",'
                     '"http_x_forwarded_for":"$http_x_forwarded_for",'
                     '"http_user_agent":"$http_user_agent"}';
    # 日志存放地址 从Nginx 0.7.6版本开始 access_log 的路径配置可以包含变量，我们可以利用这个特性来实现日志分割。
    map $time_iso8601 $logdate {
      '~^(?<ymd>\d{4}-\d{2}-\d{2})' $ymd;
      default '';
    }
    access_log  /usr/local/nginx/log/access_json_$logdate.log  main_json;
    # 每次日志写入都会打开和关闭该文件。但是，由于常用文件的描述符可以存储在缓存中，因此在 open_log_file_cache 指令的参数指定的时间内，可以继续写入旧文件
    open_log_file_cache max=1000 inactive=20s valid=1m min_uses=2;
    # 开启文件的高效传输模式
    sendfile on;
    # 激活 TCP_CORK socket 选择
    tcp_nopush on;
    # 数据在传输的过程中不进缓存
    tcp_nodelay on;
    # 对指定的浏览器关闭保持连接机制，如果指令值为 none，则对所有浏览器开启保持连接机制
    keepalive_disable none;
    # 同一 TCP 连接可复用的最大 HTTP 请求数，超过该数值后，TCP 连接将被关闭
    keepalive_requests 1000;
    # 设置客户端连接保持会话的超时时间，超过这个时间服务器会关闭该连接
    keepalive_timeout 65s;
    # 设置读取客户端请求头数据的超时时间，如果超时客户端还没有发送完整的 header 数据，服务器将返回 "Request time out (408)" 错误
    client_header_timeout 15;
    # 设置读取客户端请求主体数据的超时时间，如果超时客户端还没有发送完整的主体数据，服务器将返回 "Request time out (408)" 错误
    client_body_timeout 15;
    # 指定响应客户端的超时时间，如果超过这个时间，客户端没有任何活动，Nginx 将会关闭连接
    send_timeout 25;
    # 设置客户端最大的请求主体大小为10M，超过了此配置项，客户端会收到 413 错误，即请求的条目过大
    client_max_body_size 10m;
    client_body_buffer_size 10m;
    # 压缩功能
    gzip  on;
    # 允许压缩的对象的最小字节
    gzip_min_length 1k;
    # 低版本IE禁用Gzip压缩
    gzip_disable "MSIE [1-6]\.";
    # 压缩缓冲区大小，表示申请4个单位为16k的内存作为压缩结果的缓存
    gzip_buffers 4 32k;
    # 压缩版本，用于设置识别HTTP协议版本
    gzip_http_version 1.1;
    # 压缩级别，1级压缩比最小但处理速度最快，9级压缩比最高但处理速度最慢。级别越大压缩比越高，但浪费的CPU资源也越多
    gzip_comp_level 9;
    # 允许压缩的媒体类型
    gzip_types text/plain text/css text/javascript text/xml application/json application/javascript application/x-javascript application/font-woff application/xml;
    # 该选项可以让前端的缓存服务器缓存经过gzip压缩的页面，例如用代理服务器缓存经过Nginx压缩的数据
    gzip_vary on;
    # 用于设置共享内存区域，addr 是共享内存区域的名称，10m 表示共享内存区域的大小
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    # 引入所有 server 配置文件
    include /usr/local/nginx/conf/server/*.conf;
    # 500 错误页面
    error_page 500 502 503 504 /50x.html;
    # 404 错误页面
    error_page 404 /404.html;
    server {
        # Nginx 默认端口
        listen 80;
        # 默认访问资源
        location / {
            # 填写你的你的站点目录
            root html;
            # 网站首页
            index index.html index.htm;
            # 限制单个IP的并发连接数为 1
            limit_conn addr 1;
        }
        # 防止跨域问题
        location ~* \.(m3u8|ts)$ {
            # 防止跨域问题
            add_header 'Access-Control-Allow-Origin' '*';
            add_header 'Access-Control-Allow-Credentials' 'true';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
            add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type'; 
        }
        
        # 缓存的对象
        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|js|css|ico)$ {
            # 缓存的时间，30天 当用户第一次访问这些内容时，会把这些内容存储在用户浏览器本地，这样用户第二次及以后继续访问该网站时，浏览器会检查加载已经缓存在用户浏览器本地的内容，就不会去服务器下载了，直到缓存的内容过期或被清除为止
            expires 30d;
            # 不记录访问日志
            access_log off;
        }
        
        location /stat {
            rtmp_stat all;
            rtmp_stat_stylesheet stat.xsl;
        }
    
        location /stat.xsl {
            root /usr/local/nginx-rtmp-module/;
        }
    }
}
```