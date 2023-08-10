# Navicat Premium 无限试用脚本 

#### 该脚本是 Windows 脚本，不支持 Mac 和 Linux

> 亲测支持 Navicat Premium 16 （16.0.9 其他版本自测）

## 使用方式：

### 在 Navicat 安装目录下 新建 `.bat` 脚本，编辑输入如下内容

```shell
@echo off
 
echo Delete HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium\Registration[version and language]
for /f %%i in ('"REG QUERY "HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium" /s | findstr /L Registration"') do (
    reg delete %%i /va /f
)
echo.

echo Delete HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium\Registration[version and language]
for /f %%i in ('"REG QUERY "HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium" /s | findstr /L Registration16XCS"') do (
    reg delete %%i /va /f
)
echo.
 
echo Delete Info folder under HKEY_CURRENT_USER\Software\Classes\CLSID
for /f %%i in ('"REG QUERY "HKEY_CURRENT_USER\Software\Classes\CLSID" /s | findstr /E Info"') do (
    reg delete %%i /va /f
)
echo.
echo Finish
pause
```

### 保存之后 以管理员身份运行 `.bat` 脚本。可无限续杯14天试用
