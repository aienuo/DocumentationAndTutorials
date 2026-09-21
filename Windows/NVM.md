# Windows 环境下 NVM 使用与配置 #

[node.js手动下载地址](https://nodejs.org/download/release/)

## 常用命令： ##

### 1、查看 NVM 版本 ###

```shell
nvm version
```

### 2、查看本地已安装的 node.js 版本 ###

```shell
nvm list 
```

```shell
nvm list installed
```

### 3、显示所有可以下载的 node.js 版本 ###

```shell
nvm list available
```

### 4、查看当前 node.js 所在操作系统位数 ###

```shell
nvm arch
```

### 5、开启 node.js 版本管理 ###

```shell
nvm on
```

### 6、关闭 node.js 版本管理 ###

```shell
  nvm off
```

### 7、安装指定版本的 node.js ###

> 有可能会出现无权限安装的问题，如果遇到此问题，请以管理员身份运行

```shell
nvm install 版本号
```

示例：

* 安装 14.5.0 版本 node.js

```shell
nvm install 14.5.0
```

* 安装 32 位的 14.5.0 版本 node.js

```shell
nvm install 14.5.0 32
```

* 安装最新版本 node.js

```shell
nvm install latest
```

### 8、卸载指定版本的 node.js ###

```shell
nvm uninstall 版本号
```

示例：

* 卸载 14.5.0 版本 node.js

```shell
nvm uninstall 14.5.0
```

### 9、使用指定版本的 node.js ###

```shell
nvm use 版本号
```

示例：

* 使用 14.5.0 版本 node.js

```shell
nvm use 14.5.0
```

* 使用 32 位的 14.5.0 版本 node.js

```shell
nvm use 14.5.0 32
```

### 10、查看正在使用的 node.js 版本 ###

```shell
nvm current
```