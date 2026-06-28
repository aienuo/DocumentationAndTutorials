# Centos8 环境下 Jenkins 安装配置 #
### 注意看我的标题！！！！我这是针对2.318版本 ###

> Jenkins 安装需要JDK环境，如果没有安装JDK请参考安装 [JDK-1.8](JDK-1.8.md)
> Jenkins 运行需要TomCat，如果没有安装TomCat请参考安装 [TomCat](TomCat.md)

## 一、检查本地是否安装 ##
### 1、检查 ###

```shell
rpm -qa | grep Jenkins
```

### 2、卸载 ###

```shell
rpm -e 已经存在的 Jenkins 全名
```

## 二、解压操作文件 ##

### 下载并上传到 /usr/local/apache-tomcat-8.5.72/webapps 文件夹下

#### 关闭TomCat ####

```shell
/usr/local/apache-tomcat-8.5.72/bin/shutdown.sh
```

#### 进入TomCat webapps 文件夹下 ####

```shell
cd /usr/local/apache-tomcat-8.5.72/webapps
```

#### 删除原来的 ROOT 文件夹 ####

```shell
rm -rf ROOT
```

#### 下载地址 ####

```shell

http://mirrors.jenkins.io/war-stable/latest/jenkins.war

wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/war/2.318/jenkins.war
```

#### 重命名 jenkins 文件 ####

```shell
mv jenkins ROOT
```

#### 查看是否成功 ####

```shell
ls -a
ll
```

## 三、重启验证是否生效 ###

### 启动TomCat 或者 重启计算机 ###

#### 启动TomCat ####

```shell
/usr/local/apache-tomcat-8.5.72/bin/startup.sh
```

#### 重启计算机 ####

```shell
shutdown -r now
```

### 查看 TomCat 启动日志 寻找 默认密码

#### 进入TomCat logs 文件夹下 ####

```shell
cd /usr/local/apache-tomcat-8.5.72/logs
```

#### 最后50行内容 ####

```shell
tail -50 catalina
```

#### 查看是否出现如下内容 ####

> 如果出现如下内容说明启动成功，
> 其中类似于 0471dca6b053462fb481bf7599b48f3b 这串字符串就是 admin 的默认密码使用此密码进行登录即可
> 
> 在浏览器内访问TomCat 查看是否出现 Jenkins 登录页


```text
*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

0471dca6b053462fb481bf7599b48f3b

This may also be found at: /root/.jenkins/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************
```


## 四、`Plugin-Installation-Manager-Tool` Jenkins 插件安装管理工具 ##

> Plugin Installation Manager Tool（以下简称 `plugin-manager`）是 Jenkins 官方提供的插件管理命令行工具，用于自动化地下载、安装和管理 Jenkins 插件及其依赖。它取代了 Docker 项目中旧的 `install-plugins.sh` 脚本，是官方 Jenkins Docker 镜像中内置的插件管理方案。

### 1、安装与准备 ###

> 1. 确保已安装 `Java 运行环境`（JRE，最新版本的 Jenkins 已要求 JDK 11 或 17）。
> 2. 下载最新的 [Jenkins.war](https://get.jenkins.io/war-stable/latest/jenkins.war)
> 3. 从 [GitHub Releases](https://github.com/jenkinsci/plugin-installation-manager-tool/releases/latest) 页面下载最新版本的 `JAR` 文件。

### 2、配置 shell 脚本 ###

```shell
#!/bin/bash

# 获取当前脚本所在的目录
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"

# 定义文件和目录的路径，它们都位于脚本同级目录下
JENKINS_WAR_FILE="${SCRIPT_DIR}/jenkins.war"
PLUGINS_TXT_FILE="${SCRIPT_DIR}/plugins.txt"
PLUGIN_DOWNLOAD_DIR="${SCRIPT_DIR}/plugins"
JENKINS_VERSION="2.541.1"

# 执行插件安装命令
java -jar "${SCRIPT_DIR}/jenkins-plugin-manager-*.jar" \
  --war "${JENKINS_WAR_FILE}" \
  --plugin-file "${PLUGINS_TXT_FILE}" \
  --plugins "delivery-pipeline-plugin:1.3.2 deployit-plugin" \
  --plugin-download-directory "${PLUGIN_DOWNLOAD_DIR}" \
  --jenkins-version "${JENKINS_VERSION}" \
  --latest "false" \
  --skip-failed-plugins \
  --mirror "https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/current/"
```

#### 常用参数详解 ####

| 参数                            | 简写     | 说明                         | 默认值                                                                                         |
|:------------------------------|:-------|:---------------------------|:--------------------------------------------------------------------------------------------|
| `--war`                       |        | Jenkins WAR 文件路径           | 必填（部分场景）                                                                                    |
| `--plugin-file`               | `-f`   | 插件列表文件路径（`.txt` 或 `.yaml`） | 可选                                                                                          |
| `--plugins`                   | `-p`   | 直接指定要安装的插件列表               | 可选                                                                                          |
| `--plugin-download-directory` | `-d`   | 插件下载目录                     | Windows: `C:\ProgramData\Jenkins\Reference\Plugins`<br>其他: `/usr/share/jenkins/ref/plugins` |
| `--jenkins-version`           |        | 指定 Jenkins 版本，用于匹配插件兼容版本   | 自动检测                                                                                        |
| `--latest`                    |        | 是否下载最新版本                   | `true`                                                                                      |
| `--no-download`               |        | 仅解析插件列表但不实际下载              | `false`                                                                                     |
| `--skip-failed-plugins`       |        | 跳过下载失败的插件                  | `false`                                                                                     |

### 编制插件列表文件 `plugins.yaml` ###

