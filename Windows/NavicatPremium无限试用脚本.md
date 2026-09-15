# Navicat Premium 无限试用脚本 

#### 该脚本是 Windows 脚本，不支持 Mac 和 Linux

> 亲测支持 Navicat Premium 15 \ Navicat Premium 16 \ Navicat Premium 17

## 使用方式：

### 在 Navicat 安装目录下 新建 `.bat` 脚本，编辑输入如下内容

```shell
chcp 65001

@echo off

set dn=Info
set dn2=ShellFolder
set rp=HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium
set rp2=HKEY_CURRENT_USER\Software\Classes\CLSID

echo 【HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium\Update】 开始清除！
reg delete HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium\Update /f
echo 【HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium\Update】 清除完成！

echo.

echo 【HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium\Registration[Version And Language]】 开始清除！
echo 正在搜寻中。。。。。。
for /f %%i in ('"reg query "%rp%" /s | findstr /L Registration"') do (
	echo 正在清除：【 %%i 】
    reg delete %%i /va /f
)
echo 【HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium\Registration[Version And Language]】 清除完成！

echo.

echo 【Info and ShellFolder under HKEY_CURRENT_USER\Software\Classes\CLSID】 开始清除！
echo 正在搜寻中。。。。。。

for /f "tokens=*" %%a in ('reg query "%rp2%"') do (
	echo echo 正在清除：【 %%a 】
	for /f "tokens=*" %%l in ('reg query "%%a" /f "%dn%" /s /e ^|findstr /i "%dn%"') do (
		echo 正在清除：【 %%l 】
		reg delete %%a /f
	)
	for /f "tokens=*" %%l in ('reg query "%%a" /f "%dn2%" /s /e ^|findstr /i "%dn2%"') do (
		echo 正在清除：【 %%l 】
		reg delete %%a /f
	)
)
echo 【Info and ShellFolder under HKEY_CURRENT_USER\Software\Classes\CLSID】 清除完成！

echo.

pause
exit
```

### 保存之后 以管理员身份运行 `.bat` 脚本。可无限续杯14天试用

# Navicat Premium 连接达梦数据库

* 若需要连接达梦数据库，你必须先在你的系统中安装所需达梦 ODBC 驱动程序。请根据以下安装步骤操作

## Windows

### 1、下载驱动器

[Windows 版本 ODBC 驱动程序包 下载地址](https://dn.navicat.com/drivers/dameng_odbc_win.zip)

### 2、解压缩包

* 将下载的 `.zip` 文件内容解压到你的 `Navicat Premium` 本地安装目录
   
### 3、运行安装脚本

* 定位到你刚解压的驱动器文件所在的路径（如，`D:\Program Files\PremiumSoft\Navicat Premium 17\dameng_odbc_win`）

* 以管理员权限运行 `install_odbc.bat` 文件以开始安装文件

### 4、确认安装路径

* 该脚本将提示你输入安装目标。你可以仅按 `Enter` 键即可接受默认位置（解压文件的目录）

### 5、重启 Navicat 测试


## Linux

### 1、下载驱动器

[Linux 版本 ODBC 驱动程序包 下载地址](https://dn.navicat.com/drivers/dameng_odbc_linux.tar.gz)

### 2、解压缩包

* 将下载的 `.tar.gz` 文件内容解压到你的 `Navicat Premium` 所在目录（如，`/usr/local`）

```shell
tar -xzf dameng_odbc_linux.tar.gz -C /usr/local
```

### 3、运行安装脚本

* 进入 `install_odbc.sh` 所在本地目录

```shell
cd /usr/local/dameng_odbc_linux
```

* 给安装脚本 `install_odbc.sh` 添加执行权限

```shell
chmod +x install_odbc.sh
```

* 使用 sudo 以管理员权限运行脚本

```shell
sudo ./install_odbc.sh
```

### 4、确认安装路径

* 该脚本将提示你输入安装目标。你可以仅按 `Enter` 键即可接受默认位置（解压文件的目录）

### 5、重启 Navicat 测试

```shell
./navicat17-premium-cs-x86_64.AppImage
```