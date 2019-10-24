### yum安装keepalived  125-128
> keepalived与haproxy配置在相同的机器上

` yum -y install keepalived `

```
wget http://www.keepalived.org/software/keepalived-1.3.5.tar.gz
```

配置keepalived
```
vim /etc/keepalived/keepalived.conf
```

```

! Configuration File for keepalived
 
global_defs {          
notification_email {          # 发现异常后邮件通知管理员
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 172.0.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}
 
vrrp_script chk_haproxy {
    script "/etc/keepalived/chk.sh"     # 检查haproxy的脚本
    interval 2                          # 每两秒检查一次
}
 
vrrp_instance VI_1 {
    state BACKUP                        # 定义为BACKUP节点
    nopreempt                           # 开启不抢占，另一个不写
    interface eth0                      # 与网卡绑定
    virtual_router_id 51
    priority 100          # 开启了不抢占，所以此处优先级必须高于另一台，另一个写99
    advert_int 1
 
    authentication {        # 用于两台keepalived做相互的连通
        auth_type PASS      # 用于检测对方状态
        auth_pass 1111
    }
 
    virtual_ipaddress {
        192.168.20.140                  # 配置VIP
    }
 
    track_script {
        chk_haproxy                     # 调用检查脚本
    }
 
    notify_backup "/etc/init.d/haproxy restart"
    notify_fault "/etc/init.d/haproxy stop"
 
}
```
> ==**注意:Master和Backup不同的地方只有nopreempt和priority两处.**==

> **此处两台主机均配置为BACKUP，因此哪台先运行keepalived，VIP就在哪台上**

### 在两台机器上创建chk.sh文件

` vim /etc/keepalived `

```
#!/bin/bash
if [ $(ps -C haproxy --no-header | wc -l) -eq 0 ]; then
       /etc/init.d/keepalived stop
fi
```
`chmod +x /etc/keepalived/chk.sh` 增加脚本的执行模式

#### 两台haproxy启动keepalived
`/etc/init.d/keepalievd start` 
> 如果该脚本没有，处理方式同haproxy
下载安装包 [https://www.keepalived.org/download.html](https://www.keepalived.org/download.html)
    解压后在haproxy文件夹下找到etc/init.d/haproxy脚本复制到/etc/init.d/目录下，可以直接调用了。==（推荐）==  start  reload  restart 都可以通过该脚本直接调用

### 测试mysql负载均衡

即根据haproxy反向代理相应的策略用VIP访问MySQL.

#### 关闭其中一台MySQL依然可以连接,

#### 关闭其中一台haproxy同样可以连接;

#### 即搭建成功.
   
    
> tcpdump -nn -i ens33 vrrp # 抓包查看    没弄明白怎么用，todo吧

---

## 生产部署时遇到的问题
#### 无法绑定端口，防火墙开端口问题
> https://blog.csdn.net/s_p_j/article/details/80979450
> https://blog.csdn.net/s_p_j/article/details/80979450

#### selinux拦截
1. keepalived  检测无响应后无法自杀

` ausearch -c 'haproxy' --raw | audit2allow -M my-haproxy`

` semodule -i my-haproxy.pp`
2. 其他
> https://blog.51cto.com/victor2016/2096841?utm_source=oschina-app
> https://blog.csdn.net/zs12344444/article/details/8197235

#### 设置开机启动
> systemctl enable haproxy

#### 查看各类服务日志
> journalctl -ex
> https://www.linuxidc.com/Linux/2018-10/154964.htm

#### 查看系统日志
> tail -f /var/log/messages
> journalctl -ex







参考：
> https://blog.csdn.net/Doudou_Mylove/article/details/82888380