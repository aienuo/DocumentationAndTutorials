
## Bat脚本创建共享文件夹

```bat
chcp 65001

@echo off

set folder="D:\LiuXin\ProjectSharedFolder"

echo 正在开启系统的网络发现和局域网文件共享防火墙权限。。。。。。
rem 先开启系统的网络发现和局域网文件共享防火墙权限
netsh advfirewall firewall set rule group="文件和打印机共享" new enable=yes >nul
netsh advfirewall firewall set rule group="网络发现" new enable=yes >nul
netsh firewall set service type = fileandprint mode = enable scope = subnet >nul

reg add "HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\Lsa" /v "LimitBlankPasswordUse" /t REG_DWORD /d "00000000" /f >nul
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa" /v "LimitBlankPasswordUse" /t REG_DWORD /d "00000000" /f >nul

echo 查看有没有 ProjectSharedFolder 文件夹，有则继续，没有创建一个。。。。。。
rem 查看有没有 ProjectSharedFolder 文件夹，有则继续，没有创建一个
if not exist %folder% md %folder% >nul

echo 创建共享名为 ProjectSharedFolder 且路径为 %folder% 的文件夹共享。。。。。。
rem 创建共享名为 ProjectSharedFolder 且路径为 %folder% 的文件夹共享
net share "ProjectSharedFolder"=%folder% /unlimited /cache:no /GRANT:Everyone,FULL >nul

echo 调用 cacls 命令给 %folder% 文件增加可编辑的everyone所有共享权限。。。。。。
rem 调用 cacls 命令给 %folder% 文件增加可编辑的everyone所有共享权限
echo cacls %folder% /p everyone:f /t >nul

rem 打开共享的文件夹并提示共享文件夹创建成功
start \\%Computername%\ProjectSharedFolder >nul

# 下面是删除代码

# @echo off
# rem 删除共享的文件夹命令
# net share %folder% /y /del

pause
exit

# Bat脚本创建共享文件夹
```