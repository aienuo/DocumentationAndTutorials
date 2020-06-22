# SpringBoot Jar包 秒部署到Linux服务器运行

[参考](https://blog.csdn.net/whh18254122507/article/details/78011713?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)


## 项目重启的脚本，写个start.sh 的脚本，注意脚本和jar包同级目录

```text

#!/bin/sh
RESOURCE_NAME=项目名称
 
tpid=`ps -ef|grep $RESOURCE_NAME|grep -v grep|grep -v kill|awk '{print $2}'`
if [ ${tpid} ]; then
echo 'Stop Process...'
kill -15 $tpid
fi
sleep 5
tpid=`ps -ef|grep $RESOURCE_NAME|grep -v grep|grep -v kill|awk '{print $2}'`
if [ ${tpid} ]; then
echo 'Kill Process!'
kill -9 $tpid
else
echo 'Stop Success!'
fi
 
tpid=`ps -ef|grep $RESOURCE_NAME|grep -v grep|grep -v kill|awk '{print $2}'`
if [ ${tpid} ]; then
    echo 'App is running.'
else
    echo 'App is NOT running.'
fi
 
rm -f tpid
#nohup 意思是不挂断运行命令,当账户退出或终端关闭时,程序仍然运行
#加&表示一直后台运行，不加表示临时运行，关闭窗口项目即停止运行
nohup java -jar ./$RESOURCE_NAME --spring.profiles.active=pro &
echo $! > tpid
echo Start Success!

```

## 查看防火墙状态:

```text
sudo systemctl status firewalld 
```

## 开启firewall:
```text
service firewalld start
```

## 停止firewall:
```text
systemctl stop firewalld.service
```

## 查询端口是否开放:
```text
firewall-cmd --query-port=8080/tcp
```

## 开放8080端口:
```text
firewall-cmd --permanent --add-port=8080/tcp
```

## 移除端口:
```text
firewall-cmd --permanent --remove-port=8080/tcp
```

## 重启防火墙(修改配置后要重启防火墙!!!):
```text
firewall-cmd --reload
```
