# Centos7 环境下 Haproxy-1.5.18 安装配置 #
### 注意看我的标题！！！！我这是针对7.0.96版本 ###
## 一、检查本地是否安装 ##
### 1、检查 ###
```
rpm -qa | grep xxx
```
### 2、卸载 ###
```
rpm -e xxx
``` 
### 3、下载 ###
```
wget https://src.fedoraproject.org/repo/pkgs/haproxy/haproxy-1.5.18.tar.gz/md5/21d35f114583ef731bc96af05b46c75a/haproxy-1.5.18.tar.gz
```
## 二、解压、操作文件 ##
### 解压（找到压缩文件） ###
	tar -zxvf haproxy-1.5.18.tar.gz -C /usr/local/
#### 切换到解压目录下 ####
```	
cd /usr/local/
```	
#### 查看是否剪切成功 ####
```	
ll
```
#### 将当前文件夹授权给 haproxy ####
##### 创建haproxy组 #####
```
groupadd haproxy
```	
##### 创建haproxy用户添加到haproxy组 #####
```
useradd -g haproxy haproxy
```
##### 授权 #####
```
chown -R haproxy:haproxy /usr/local/haproxy-1.5.18/
```
## 三、配置环境变量 ###
### 1、配置 Haproxy ###
#### ① 修改配置文件 ####
```
vim /etc/haproxy/haproxy.cfg
```
##### 按一下键盘字母`i`进行编辑 #####
#### ② 输入以下内容： ####
``` 
#默认的defaults模块以上不动,以下部分替换成如下内容,两台haproxy配置一致.
 
listen mysql_proxy
		bind 0.0.0.0:3306   	#绑定端口 如果haproxy与mysql在同一台机器上，这个端口要修改，否则无法绑定
		mode tcp
 
		balance source    	#定义负载均衡算法
 
		server mysqldb1 192.168.20.139:3306 weight 1  check  inter 1s rise 2 fall 2        		#master mysql
		server mysqldb2 192.168.20.138:3306 weight 2  check  inter 1s rise 2 fall 2 backup         	#slave mysql
 
listen stats						#监控 可视化监控
		mode http
		bind 0.0.0.0:8888                    	#web监控登录端口
		stats enable
		stats uri /dbs                        	#we监控端登录地址http:ip:8888/dbs
		stats realm haproxy\ statistics
		stats auth admin:admin                	#web监控登端录用户和密码
```
##### 按一下`esc`键 退出编辑 #####
##### `:wq` 保存退出 #####
> 若无法启动，查看服务日志后发现是selinux阻挡或防火墙阻挡绑定端口
```
firewall-cmd --add-port=8888/tcp --zone=public --permanent

firewall-cmd --reload

semanage port -a -t mysqld_port_t -p tcp 8888

setsebool -P haproxy_connect_any 1
```
#### ③ 修改立即生效： ####
```
source /etc/haproxy/haproxy.cfg
```
### 2、配置修改日志系统 ###
#### ① 修改配置文件 ####
```
vim /etc/rsyslog.conf
```
##### 按一下键盘字母`i`进行编辑 #####
#### ② 输入以下内容： ####
```
#在centos6.x系统中，系统日志的配置文件原来的/etc/syslog.conf已经变为/etc/rsyslog.conf
 
###Provides UDP syslog reception               //去掉下面两行注释，开启UDP监听
 
$ModLoad imudp
$UDPServerRun 514
 
local2.* /var/log/haproxy.log           #添加此行
```
##### 按一下`esc`键 退出编辑 #####
##### `:wq` 保存退出 #####
#### ③ 修改立即生效： ####
```
source /etc/rsyslog.conf
```
### 3、配置修改 `/etc/sysconfig/syslog` ###
#### ① 修改配置文件 ####
```
vim /etc/sysconfig/rsyslog
```
##### 按一下键盘字母`i`进行编辑 #####
#### ② 输入以下内容： ####
```
SYSLOGD_OPTIONS="-c 2 -r -m 0"
```
> 注释：-c 2 使用兼容模式，默认是 -c 5；-r 开启远程日志; -m 0标记时间戳。单位是分钟，为0时，表示禁用该功能
##### 按一下`esc`键 退出编辑 #####
##### `:wq` 保存退出 #####
#### ③ 修改立即生效： ####
```
source /etc/sysconfig/rsyslog
```
## 四、查看是否生效 ##
### 重启rsyslog ###
```
systemctl restart rsyslog
```
### 启动Haproxy ###
```
/etc/init.d/haproxy start
```
> 如果在/etc/init.d/下没有haproxy的脚本，
> 解决方案：
1. ` /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg` 
2. 将以下脚本保存为 `/etc/init.d/haproxy`
<br/>start  reload  restart 等都可以通过该脚本直接调用 脚本：

```
#!/bin/bash

# Source function library.

if [ -f /etc/init.d/functions ]; then

  . /etc/init.d/functions

elif [ -f /etc/rc.d/init.d/functions ] ; then

  . /etc/rc.d/init.d/functions

else

  exit 0

fi

CONF_FILE="/etc/haproxy/haproxy.cfg"

HAPROXY_BINARY="/usr/local/sbin/haproxy"

PID_FILE="/var/run/haproxy.pid"

# Source networking configuration.

. /etc/sysconfig/network

# Check that networking is up.

[ ${NETWORKING} = "no" ] && exit 0
[ -f ${CONF_FILE} ] || exit 1
RETVAL=0

start() {

  $HAPROXY_BINARY -c -q -f $CONF_FILE

  if [ $? -ne 0 ]; then

    echo "Errors found in configuration file."

    return 1

  fi

  echo -n "Starting HAproxy: "

  daemon $HAPROXY_BINARY -D -f $CONF_FILE -p $PID_FILE

  RETVAL=$?

  echo

  [ $RETVAL -eq 0 ] && touch /var/lock/subsys/haproxy

  return $RETVAL

}

stop() {

  echo -n "Shutting down HAproxy: "

  killproc haproxy -USR1

  RETVAL=$?

  echo

  [ $RETVAL -eq 0 ] && rm -f /var/lock/subsys/haproxy

  [ $RETVAL -eq 0 ] && rm -f $PID_FILE

  return $RETVAL

}

restart() {

  $HAPROXY_BINARY -c -q -f $CONF_FILE

  if [ $? -ne 0 ]; then

    echo "Errors found in configuration file, check it with 'haproxy check'."

    return 1

  fi

  stop

  start

}

check() {

  $HAPROXY_BINARY -c -q -V -f $CONF_FILE

}

rhstatus() {

  pid=$(pidof haproxy)

  if [ -z "$pid" ]; then

    echo "HAProxy is stopped."

    exit 3

  fi

  status haproxy

}

condrestart() {

  [ -e /var/lock/subsys/haproxy ] && restart || :

}

# See how we were called.

case "$1" in

  start)

    start

    ;;

  stop)

    stop

    ;;

  restart)

    restart

    ;;

  reload)

    restart

    ;;

  condrestart)

    condrestart

    ;;

  status)

    rhstatus

    ;;

  check)

    check

    ;;

  *)

    echo $"Usage: haproxy {start|stop|restart|reload|condrestart|status|check}"

    RETVAL=1

esac

exit $RETVAL

```
### 打开网页查看Haproxy监控MySQL的情况  
	http://xxx.xxx.xxx.xxx:8888/dbs （两台服务器的ip都可以访问）
> 与/etc/haproxy/haproxy.cfg配置的可视化监控关联 用户名和密码在
`stats auth` 配置