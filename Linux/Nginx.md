# Centos8 环境下 Nginx 安装配置 #
### 注意看我的标题！！！！我这是针对 1.20.2 版本 ###
> 偶数版本-稳定版本（两个稳定版本之间的跨越时间越长、补丁发布频率越少），奇数版本-开发、测试版本
## 一、检查本地是否安装 ##
### 1、检查 ###
    rpm -qa | grep nginx
### 2、卸载 ###
    rpm -e 已经存在的Nginx全名	
#### 切换到下载目录下 ####
	cd /usr/local/
### 3、下载 ###
	wget https://nginx.org/download/nginx-1.20.2.tar.gz
### 4、解压（找到压缩文件） ###
	tar -zxvf nginx-1.20.2.tar.gz  -C /usr/local/
## 一、Nginx 调优 ##
> 修改源代码文件
### 1、隐藏版本信息 ###
    vim /usr/local/nginx-1.20.2/src/core/nginx.h
#### 按一下键盘字母`i`进行编辑 ####
#### 修改以下内容（13行 显示的版本号） ####
    #define NGINX_VERSION      "996"
#### 修改以下内容（14行 显示的软件名） ####
    #define NGINX_VER          "www.lau.xin/" NGINX_VERSION
#### 修改以下内容（22行 显示的软件名） ####
    #define NGINX_VAR          "www.lau.xin"
#### 按一下`esc`键 退出编辑 ####
#### `:wq` 保存退出 ####
### 2、隐藏软件名 ###
    vim /usr/local/nginx-1.20.2/src/http/ngx_http_header_filter_module.c
#### 按一下键盘字母`i`进行编辑 ####
#### 修改以下内容（49行 修改为想要显示的软件名） ####
    static u_char ngx_http_server_string[] = "Server: www.lau.xin" CRLF;
#### 按一下`esc`键 退出编辑 ####
#### `:wq` 保存退出 ####
### 3、定义对外展示内容 ###
    vim /usr/local/nginx-1.20.2/src/http/ngx_http_special_response.c
#### 按一下键盘字母`i`进行编辑 ####
#### 修改以下内容（22行 此行定义对外展示的内容） ####
    "<hr><center>" NGINX_VER "(www.lau.xin)</center>" CRLF
#### 修改以下内容（36行 此行定义对外展示的软件名） ####
    "<hr><center>www.lau.xin</center>" CRLF
#### 按一下`esc`键 退出编辑 ####
#### `:wq` 保存退出 ####
> 配置Linux系统参数
### 4、修改文件打开数限制 ###
> 操作系统默认单进程最大打开文件数为1024

    vim /etc/security/limits.conf
#### 按一下键盘字母`i`进行编辑 ####
#### 新增以下内容 ####
>  *号表示所用用户

    * soft nofile 65535
    * hard nofile 65535
#### 按一下`esc`键 退出编辑 ####
#### `:wq` 保存退出 ####
### 5、设置 login ###
    vim /etc/pam.d/login
#### 按一下键盘字母`i`进行编辑 ####
#### 新增以下内容 ####
> /lib/security/pam_limits.so 这个路径根据实际情况填写，32位系统是/lib下 64位系统是/lin64下

    session    required     /lib/security/pam_limits.so
#### 按一下`esc`键 退出编辑 ####
#### `:wq` 保存退出 ####
### 6、设置 DNS缓存 ###
> 开启 NSCD 服务，缓存 DNS，提高域名解析响应速度

    vim /etc/resolv.conf
#### 按一下键盘字母`i`进行编辑 ####
#### 新增以下内容 ####
    ststemetl start nscd.service    
    ststemetl enable nscd.service
## 二、编译安装 ##
### 1、下载依赖 ###
> 缺少 xxxlibrany，缺少.c.h文件 安装开发组包  xxx-devel

	yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel pcre pcre-devel
### 2、切换到解压目录下 ###
	cd /usr/local/nginx-1.20.2
### 3、运行./configure 进行初始化配置 [参考](image/configure.md) ###
	./configure --prefix=/usr/local/nginx --user=nginx --group=nginx --pid-path=/usr/local/nginx/log/nginx.pid --error-log-path=/usr/local/nginx/log/error.log --http-log-path=/usr/local/nginx/log/access.log --with-http_gzip_static_module --http-client-body-temp-path=/usr/local/nginx/temp/client --http-proxy-temp-path=/usr/local/nginx/temp/proxy --http-fastcgi-temp-path=/usr/local/nginx/temp/fastcgi --http-uwsgi-temp-path=/usr/local/nginx/temp/uwsgi --http-scgi-temp-path=/usr/local/nginx/temp/scgi --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_gzip_static_module --with-http_addition_module --with-http_flv_module --with-http_mp4_module --with-http_stub_status_module --with-pcre --wuth--http_sub_module
### 4、执行编译操作 ###
> -j8 多线程操作，加速编译

	make -j8
### 5、执行安装操作 ###
	make install -j8
### 6、创建临时文件存放路径 ###
#### 进入安装路径 ####
	cd /usr/local/nginx
#### 创建 一级 ####
    mkdir temp
#### 进入 一级 路径 ####
    cd /usr/local/nginx/temp
#### 创建 二级 ####
    mkdir client proxy fastcgi uwsgi scgi
### 7、创建 用户，用户组 ###
#### 用户组 ####
    groupadd nginx
#### 用户 ####
    useradd -g nginx nginx
## 三、Nginx 运行维护 ##
### 进入安装路径 ###
	cd /usr/local/nginx
### 启动 ###
	./sbin/nginx
### 查看是否有 Nginx ###
	ps -ef | grep nginx
### 重载 ###
	./sbin/nginx -s reload
### 关闭 ###
	./sbin/nginx -s stop
#### 优雅关闭（当请求被处理完成之后才关闭） ###
	./sbin/nginx -s quit
## 四、防火墙端口开放 ##
##### 1、查看防火墙状态 #####
	firewall-cmd --state
##### 2、关闭防火墙 #####
	systemctl stop firewalld.service
##### 3、禁止防火墙开机自启 #####
	systemctl disable firewalld.service
##### 4、查看所有已开放的临时端口 #####
	firewall-cmd --list-ports
##### 5、开启防火墙 #####
	systemctl start firewalld.service
##### 6、Add 添加开放端口 #####
	firewall-cmd --permanent --zone=public --add-port=80/tcp
##### 7、Reload 重新加载 #####
	firewall-cmd --reload
##### 8、检查是否生效 ######
	firewall-cmd --zone=public --query-port=80/tcp
##### 9、删除开放端口 ######
    firewall-cmd --zone=public --remove-port=8088/tcp --permanent
## 五、开机自启配置 ##
### 1、在系统服务目录里创建`nginx.service`文件 ###
    vim /usr/lib/systemd/system/nginx.service
#### 按一下键盘字母`i`进行编辑 ####
### 2、输入以下内容 ###
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
#### 按一下`esc`键 退出编辑 ####
#### `:wq` 保存退出 ####
### 3、设置开机自启动 ###
    systemctl enable nginx.service
### 4、重启计算机 ###
	shutdown -r now
### 5、查看 Nginx 状态 ###
    systemctl status nginx.service
### 6、查看是否有 Nginx ###
	ps -ef | grep nginx
## 六、配置 域名 与 HTTPS ##
### 1、上传 SSL 证书 ###
#### ① 切换到 Nginx 目录下 ####
    cd /usr/local/nginx
#### ② 新建 SSL 证书目录 ####
    mkdir cert
##### 将证书与key上传到此目录 #####
### 2、切换到安装目录的配置文件目录下 ###
	cd /usr/local/nginx/conf/
### 3、编辑默认配置文件 ###
#### 查看CPU核数（记下来留作备用） ####
    grep -c processor /proc/cpuinfo
#### 编辑 `nginx.conf` 配置文件 ####
    vim nginx.conf
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
}
```
### 4、配置 server 配置 ###
#### 创建 server 文件夹 ####
    mkdir server
#### 进入 server 文件夹 ####
    cd /usr/local/nginx/conf/server/
#### 创建 自定义的 server 配置文件 ####
    vim demo.conf
#### 按一下键盘字母`i`进行编辑 ####
#### 添加以下内容： ####
```shell
# HTTPS
server {
    # HTTPS 默认端口
    listen 443 ssl;
    # 填写绑定证书的域名
    server_name www.lau.xin;
    # 填写你的证书所在的位置
    ssl_certificate /usr/local/nginx/cert/lauxin.pem;
    # 填写你的key所在的位置
    ssl_certificate_key /usr/local/nginx/cert/lauxin.key;
    # 会话超时
    ssl_session_timeout 5m;
    # 按照这个协议配置
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    # 按照这个套件配置
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
    # 默认访问资源
    location / {
        # 填写你的你的站点目录
        root html;
        # 网站首页
        index index.html index.htm;
        # 限制单个IP的并发连接数为 1
        limit_conn addr 1;
    }
    # 缓存的对象
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|js|css)$ {
        # 缓存的时间，30天 当用户第一次访问这些内容时，会把这些内容存储在用户浏览器本地，这样用户第二次及以后继续访问该网站时，浏览器会检查加载已经缓存在用户浏览器本地的内容，就不会去服务器下载了，直到缓存的内容过期或被清除为止
        expires 30d;
        # 不记录访问日志
        access_log off;
    }
}
server {
    # Nginx 默认端口
    listen 80;
    # 填写绑定证书的域名
    server_name lau.xin;
    # 将 http 转到 https
    rewrite ^ https://$http_host$request_uri? permanent;
}
```
##### 按一下`esc`键 退出编辑 #####
##### `:wq` 保存退出 #####
