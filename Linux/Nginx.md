# Centos8 环境下 Nginx 安装配置 #
### 注意看我的标题！！！！我这是针对 1.21.3 版本 ###
## 一、检查本地是否安装 ##
### 1、检查 ###
    rpm -qa | grep nginx
### 2、卸载 ###
    rpm -e 已经存在的Nginx全名	
#### 切换到下载目录下 ####
	cd /usr/local/
### 3、下载 ###
	wget https://nginx.org/download/nginx-1.21.3.tar.gz
### 4、解压（找到压缩文件） ###
	tar -zxvf nginx-1.21.3.tar.gz  -C /usr/local/
## 二、编译安装 ##
### 1、下载依赖 ###
	yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel pcre pcre-devel
### 2、切换到解压目录下 ###
	cd /usr/local/nginx-1.21.3
### 3、运行./configure 进行初始化配置 [参考](image/configure.md) ###
	./configure --prefix=/usr/local/nginx --pid-path=/usr/local/nginx/log/nginx.pid --error-log-path=/usr/local/nginx/log/error.log --http-log-path=/usr/local/nginx/log/access.log --with-http_gzip_static_module --http-client-body-temp-path=/usr/local/nginx/temp/client --http-proxy-temp-path=/usr/local/nginx/temp/proxy --http-fastcgi-temp-path=/usr/local/nginx/temp/fastcgi --http-uwsgi-temp-path=/usr/local/nginx/temp/uwsgi --http-scgi-temp-path=/usr/local/nginx/temp/scgi --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_addition_module --with-http_flv_module --with-http_mp4_module --with-http_stub_status_module
### 4、执行编译操作 ###
	make
### 5、执行安装操作 ###
	make install
### 6、创建临时文件存放路径 ###
#### 进入安装路径 ####
	cd /usr/local/nginx
#### 创建 一级 ####
    mkdir temp
#### 进入 一级 路径 ####
    cd /usr/local/nginx/temp
#### 创建 二级 ####
    mkdir client proxy fastcgi uwsgi scgi
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
### 3、编辑 `nginx.conf` 配置文件
    vim nginx.conf
#### 按一下键盘字母`i`进行编辑 ####
#### 添加以下内容： ####
	# 以上均为 nginx.conf 文件的默认配置，不需要更改，直接在下面添加即可
    # another virtual host using mix of IP-, name-, and port-based configuration
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
        location / {
                # 填写你的你的站点目录
                root html;
                # 网站首页
                index index.html;
            }
        }
    server {
        # Nginx 默认端口
        listen 80;
        # 填写绑定证书的域名
        server_name www.lau.xin;
        # 将 http 转到 https
        rewrite ^ https://$http_host$request_uri? permanent;
    }
    server {
        # Nginx 默认端口
        listen 80;
        # 填写绑定证书的域名
        server_name lau.xin;
        # 将 http 转到 https
        rewrite ^ https://$http_host$request_uri? permanent;
    }
##### 按一下`esc`键 退出编辑 #####
##### `:wq` 保存退出 #####
