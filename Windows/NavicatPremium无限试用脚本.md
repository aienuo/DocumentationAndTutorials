# Navicat Premium 无限试用脚本 

#### 该脚本是 Windows 脚本，不支持 Mac 和 Linux

> 亲测支持 Navicat Premium 16 （16.0.9 其他版本自测）

## 使用方式：

### 在 Navicat 安装目录下 新建 `.bat` 脚本，编辑输入如下内容

```shell
@echo off
set dn=Info
set dn2=ShellFolder
set rp=HKEY_CURRENT_USER\Software\Classes\CLSID
:: reg delete HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium\Registration14XCS /f  %针对<strong><font color="#FF0000">navicat</font></strong>15%
reg delete HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium\Registration16XCS /f
reg delete HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium\Update /f
echo finding.....

for /f "tokens=*" %%a in ('reg query "%rp%"') do (
 echo %%a
for /f "tokens=*" %%l in ('reg query "%%a" /f "%dn%" /s /e ^|findstr /i "%dn%"') do (
  echo deleteing: %%a
  reg delete %%a /f
)
for /f "tokens=*" %%l in ('reg query "%%a" /f "%dn2%" /s /e ^|findstr /i "%dn2%"') do (
  echo deleteing: %%a
  reg delete %%a /f
)
)
echo re trial done!
  
pause
exit
```

### 保存之后 以管理员身份运行 `.bat` 脚本。可无限续杯14天试用
