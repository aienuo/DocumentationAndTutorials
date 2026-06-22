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
