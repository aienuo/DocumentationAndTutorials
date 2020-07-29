# 使用CentOS-8.2.2004-x86_64-boot.iso网络安装CentOS

> 本文将介绍如何通过网络安装CentOS-8

## 下载CentOS-8的boot文件

[下载地址一](https://mirrors.aliyun.com/centos/8.2.2004/isos/x86_64/CentOS-8.2.2004-x86_64-boot.iso)

[下载地址二](https://mirrors.163.com/centos/8.2.2004/isos/x86_64/CentOS-8.2.2004-x86_64-boot.iso)

> 文件大小很亲民，才600多M，别说一张DVD了，就是CD也能刻下。

## 进行安装

> 如果是非VMwareWorkstation用户请越过第一步

#### 第一步： 新建一个虚拟机

```text
1.选择文件下的新建虚拟机
2.选择自定义，并点击下一步
```

![Alt](image/1.png)

```text
3.点击下一步
```

![Alt](image/2.png)

```text
4.选择稍后安装操作系统，并点击下一步
```

![Alt](image/3.png)

```text
5.选择Linux(L)，版本下拉选择CentOS 8 64 位，并点击下一步
```

![Alt](image/4.png)
	
```text
6.自定义一个虚拟机名称，调整位置，并点击下一步
```

![Alt](image/5.png)

```text
7.设置好虚拟机处理器，并点击下一步
```

![Alt](image/6.png)
	
```text
8.设置好虚拟机内存，并点击下一步
```

![Alt](image/7.png)

```text
9.设置网络类型为 使用桥接网络，并点击下一步
```

![Alt](image/8.png)

```text
10.设置I/O控制器类型，并点击下一步
```

![Alt](image/9.png)

```text
11.设置磁盘类型，并点击下一步
```

![Alt](image/10.png)

```text
12.选择磁盘，并点击下一步
```

![Alt](image/11.png)

```text
13.设置磁盘容量，并点击下一步
```

![Alt](image/12.png)

```text
14.指定磁盘文件，并点击下一步
```

![Alt](image/13.png)

```text
15.选择完成
```

![Alt](image/14.png)

#### 第二步： 安装操作系统

```text
16.点击编辑虚拟机，选择CD/DVD(IDE)调整刚下载的xxx-boot.iso文件，点击确定
```

![Alt](image/15.png)

```text
17.开启此虚拟机，出现如下窗口选择第一个并按回车键等待加载完毕
```

![Alt](image/16.png)

```text
18.出现如下窗口调整系统语言，选择完毕后点击蓝色按钮
```

![Alt](image/17.png)

```text
19.出现如下窗口，根据提示先调整  装目的地
```

![Alt](image/18.png)


```text
20.注意存储配置  选择自动，点击左上角完成
```

![Alt](image/19.png)


```text
21.点击 右上角 打开，再点击右下角配置
```

![Alt](image/20.png)


```text
22.注意查看如下两个配置项
```

![Alt](image/21.png)
![Alt](image/22.png)


```text
23.配置安装源，注意选择并在后面红线处手动输入如网址（两个选一个，大小写敏感）

https://mirrors.aliyun.com/centos/8.2.2004/BaseOS/x86_64/os/

https://mirrors.163.com/centos/8.2.2004/BaseOS/x86_64/os/

```

![Alt](image/23.png)

```text
24.点击完成，出现下图所示（注意：一般很快就会出现文字滚动，超过一分钟就是地址不正确，重启并重复步骤17-23）
```

![Alt](image/24.png)
![Alt](image/25.png)

```text
25.选件选择，选择服务器，点击完成
```

![Alt](image/26.png)

```text
26.出现如下页面内容，点击开始安装
```

![Alt](image/27.png)

```text
26.设置根密码，等待安装完成
```

![Alt](image/28.png)
![Alt](image/29.png)
![Alt](image/30.png)


### 配置内网地址

```text
26.配置内网地址

cd /etc/sysconfig/network-scripts/

ll

vi ifcfg-ens33

注意按照下图修改

按一下`esc`键 退出编辑

:wq 保存退出


service network restart 重启网络

 或者  systemctl restart network


```

![Alt](image/31.png)


```text
vim /etc/sysconfig/network-scripts/ifcfg-ens33 


TYPE=Ethernet					#网络类型：因特网
PROXY_METHOD=none				#代理方式：关闭状态
BROWSER_ONLY=no					#只是浏览器：否
BOOTPROTO=dhcp 改成 static      #网卡的引导协议：DHCP[中文名称: 动态主机配置协议],static：静态ip
DEFROUTE=yes					# 默认路由：是,不明白的可以百度关键词 `默认路由` 
IPV4_FAILURE_FATAL=no			# 是否开启IPV4致命错误检测：否
IPV6INIT=yes					# IPV6是否自动初始化: 是[不会有任何影响, 现在还没用到IPV6]
IPV6_AUTOCONF=yes				# IPV6是否自动配置：是[不会有任何影响, 现在还没用到IPV6]
IPV6_DEFROUTE=yes				# IPV6是否可以为默认路由：是[不会有任何影响, 现在还没用到IPV6]
IPV6_FAILURE_FATAL=no			# 是否开启IPV6致命错误检测：否
IPV6_ADDR_GEN_MODE=stable-privacy	# IPV6地址生成模型：stable-privacy [这只一种生成IPV6的策略]
NAME=ens33						# 网卡物理设备名称
UUID=b1081c3a-9c31-41c2-81ea-937945a3f6ee		# 通用唯一识别码, 每一个网卡都会有, 不能重复, 否则两台linux只有一台网卡可用
DEVICE=ens33					# 网卡设备名称, 必须和 `NAME` 值一样
ONBOOT=no 改成 yes	            # 是否开机启动， 要想网卡开机就启动或通过 `systemctl restart network`控制网卡,必须设置为 `yes` 
#以下为新添值
IPADDR=192.168.1.101			#设置本机固定ip地址【0-255】
NETMASK=255.255.255.0			#子网掩码
GATEWAY=192.168.1.1				#默认网关                

```

### 配置自定义登录欢迎语

```text
26.如下内容，编辑即可
```

![Alt](image/32.png)
