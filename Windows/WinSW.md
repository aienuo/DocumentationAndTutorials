# Windows 环境下 WinSW 安装配置 #

### 注意看我的标题！！！！我这是针对 WinSW v2.12.0 版本 ###

## 下载地址：

> 下载之后直接放到想要注册成服务的应用的安装目录下

[WinSW](https://github.com/winsw/winsw/releases/download/v2.12.0/WinSW-x64.exe)

## 配置文件：

[配置文件参考](sample.xml)

## WinSW 命令

|    命令     | 描述            |          使用示例           |
|:---------:|:--------------|:-----------------------:|
|  install  | 安装服务          |  WinSW-x64.exe install  |
| uninstall | 卸载服务          | WinSW-x64.exe uninstall |
|   start   | 启动服务          |   WinSW-x64.exe start   |
|   stop    | 停止服务          |   WinSW-x64.exe stop    |
|  restart  | 重启服务          |  WinSW-x64.exe restart  |
|  status   | 检查服务状态        |  WinSW-x64.exe status   |
|  refresh  | 刷新服务属性而不是重新安装 |  WinSW-x64.exe refresh  |

## 示例 

[1、FPR Windows 客户端 注册为服务 自启动](FRP-Client.md)
