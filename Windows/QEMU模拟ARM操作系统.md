# `Win10 X86`  环境下 模拟 `ARM版麒麟V11` #

### 注意看我的标题！！！！我这是针对 X86 版本的 Win10 模拟 ARM版麒麟V11

## 一、准备安装包 ##

* [`QEMU`版本列表](https://qemu.weilnetz.de/w64/)

* [Kylin-Server-V11-2503-Release-General-20250715-ARM64 下载地址](https://iso.kylinos.cn/web_pungi/download/cdn/WMe7UrQI2mx0LY1nPfbSTCv83sypjioa/Kylin-Server-V11-2503-Release-General-20250715-ARM64.iso)

## 二、手动安装 `QEMU` 工具 ##

### 安装 ###

* 1、双击下载的 `.exe` 安装包

* 2、选择安装语言：`English`

* 3、选择安装路径：`D:\Documents\ARM_OS\qemu`（注意根据自己实际情况调整路径，再次强调：无空格全英文）

* 4、组件选择：保持默认（全选）

* 5、完成安装

### 验证 ###

> 注意根据自己实际情况调整路径

* 1、 能否模拟 `ARM` 指令集

```cmd
D:\Documents\ARM_OS\qemu\qemu-system-aarch64.exe --version
```

* 2、固件文件是否存在

```cmd
dir D:\Documents\ARM_OS\qemu\share\edk2-aarch64-code.fd
```

## 三、配置虚拟机文件 ##

> 注意根据自己实际情况调整路径

* 1、创建磁盘文件

```cmd
mkdir D:\Documents\ARM_OS\QEMU_Kylin\Kylin_V11_01
```

```cmd
D:\Documents\ARM_OS\qemu\qemu-img create -f qcow2 D:\Documents\ARM_OS\QEMU_Kylin\Kylin_V11_01\system.img 80G
```

| 命令       | 说明                                  |
|:-----------|:--------------------------------------|
| `create`   | 创建磁盘                              |
| `-f qcow2` | 动态分配格式（用多少占多少，最大80G） |
| `80G`      | 最大容量                              |

* 2、验证磁盘文件

```cmd
D:\Documents\ARM_OS\qemu\qemu-img info D:\Documents\ARM_OS\QEMU_Kylin\Kylin_V11_01\system.img
```

## 四、安装银河麒麟系统 ##

* 1、创建安装程序启动脚本

> 在 `D:\Documents\ARM_OS` 下 创建 `create_os.bat` 输入如下内容(注意根据自己实际情况调整变量)

```bat
@echo off
chcp 65001 >nul 2>&1
:: 设置变量
set QEMU_DIR=D:\Documents\ARM_OS\qemu
set VM_DIR=D:\Documents\ARM_OS\QEMU_Kylin\Kylin_V11_01
set BIOS_DIR=D:\Documents\ARM_OS\qemu\share
set ISO_DIR=D:\Documents\ARM_OS\

:: 自动创建虚拟机目录
if not exist "%VM_DIR%" (
    echo [0/3] 正在创建虚拟机目录 %VM_DIR% ...
    mkdir "%VM_DIR%"
)

:: 检查 ISO 文件是否存在
if not exist "%ISO_DIR%\Kylin-Server-V11-2503-Release-General-20250715-ARM64.iso" (
    echo [错误] 找不到 ISO 文件：%ISO_DIR%\Kylin-Server-V11-2503-Release-General-20250715-ARM64.iso
    echo 请将麒麟系统 ISO 文件放到 D:\Documents\ARM_OS\ 目录下
    pause
    exit /b 1
)

:: 检查 BIOS 文件是否存在
if not exist "%BIOS_DIR%\edk2-aarch64-code.fd" (
    echo [错误] 找不到 BIOS 文件：%BIOS_DIR%\edk2-aarch64-code.fd
    pause
    exit /b 1
)

:: 检查虚拟磁盘是否已存在，避免误覆盖
if exist "%VM_DIR%\system.img" (
    echo [提示] 虚拟磁盘已存在：%VM_DIR%\system.img
    set /p confirm=是否覆盖重建？[y/n]: 
    if /i not "%confirm%"=="y" (
        echo [取消] 已跳过镜像创建，直接进入启动流程
    ) else (
        echo [1/4] 正在创建 80G 虚拟磁盘镜像...
        "%QEMU_DIR%\qemu-img.exe" create -f qcow2 "%VM_DIR%\system.img" 80G
        if errorlevel 1 (
            echo [错误] 虚拟磁盘创建失败
            pause
            exit /b 1
        )
    )
) else (
    echo [1/4] 正在创建 80G 虚拟磁盘镜像...
    "%QEMU_DIR%\qemu-img.exe" create -f qcow2 "%VM_DIR%\system.img" 80G
    if errorlevel 1 (
        echo [错误] 虚拟磁盘创建失败
        pause
        exit /b 1
    )
)

echo [2/4] 正在启动虚拟机...
"%QEMU_DIR%\qemu-system-aarch64.exe" ^
  -m 8192 ^
  -cpu cortex-a72 ^
  -smp 8,sockets=4,cores=2 ^
  -M virt ^
  -bios "%BIOS_DIR%\edk2-aarch64-code.fd" ^
  -device VGA ^
  -device nec-usb-xhci ^
  -device usb-mouse ^
  -device usb-kbd ^
  -drive if=none,file="%VM_DIR%\system.img",id=hd0 ^
  -device virtio-blk-device,drive=hd0 ^
  -drive if=none,file="%ISO_DIR%\Kylin-Server-V11-2503-Release-General-20250715-ARM64.iso",id=cdrom,media=cdrom ^
  -device virtio-scsi-device ^
  -device scsi-cd,drive=cdrom ^
  -net nic,model=virtio ^
  -net user,hostfwd=tcp::2222-:22

echo [3/4] 虚拟机已关闭
echo [4/4] 如安装完成，请运行 start_os.bat 启动系统
pause
```

| 参数                              | 含义           | 选型理由                          |
|:----------------------------------|:---------------|:----------------------------------|
| `-m 8192`                         | 8GB 内存       | 麒麟桌面版最低要求 4GB            |
| `-cpu cortex-a72`                 | ARM CPU 模型   | 性能与兼容性平衡点                |
| `-smp 8,sockets=4,cores=2`        | 8 核           | 编译/AI 推理需要多核              |
| `-M virt`                         | ARM 通用虚拟机 | QEMU 官方推荐的 ARM 机型          |
| `-device virtio-blk-device`       | VirtIO 磁盘    | 比模拟 IDE 快 30%+                |
| `-net user,hostfwd=tcp::2222-:22` | 端口转发       | 宿主机 2222 → 虚拟机 22，便于 SSH |


* 2、运行脚本

双击运行 `create_os.bat`

> 部分操作系统会等待很久才会出现安装界面，有的需要必须在5秒内用方向键完成选择，否则会进入桌面预览

* 3、后续步骤与物理机安装一致，按照提示操作即可（安装可能需要很长时间）

* 4、看到 `安装成功，请重启` 提示后，关闭 `QEMU` 窗口

## 五、系统运行 ##

* 1、创建操作系统启动脚本

> 在 `D:\Documents\ARM_OS` 下 创建 `start_os.bat` 输入如下内容(注意根据自己实际情况调整变量)

```bat
@echo off
chcp 65001 >nul
set QEMU_DIR=D:\Documents\ARM_OS\qemu
set VM_DIR=D:\Documents\ARM_OS\QEMU_Kylin\Kylin_V11_01
set BIOS_DIR=D:\Documents\ARM_OS\qemu\share

echo ============================================
echo   银河麒麟 V11 ARM64 虚拟机启动器
echo ============================================
echo.

echo [1/3] 检查虚拟磁盘...
if not exist "%VM_DIR%\system.img" (
    echo 错误：虚拟磁盘不存在，请先运行 create_os.bat 安装银河麒麟系统
    pause
    exit /b 1
)

echo [2/3] 启动虚拟机...
echo     内存: 8GB
echo     CPU:  Cortex-A72 x8
echo     磁盘: %VM_DIR%\system.img
echo     SSH:  127.0.0.1:2222
echo.

"%QEMU_DIR%\qemu-system-aarch64.exe" ^
  -m 8192 ^
  -cpu cortex-a72 ^
  -smp 8,sockets=4,cores=2 ^
  -M virt ^
  -bios "%BIOS_DIR%\edk2-aarch64-code.fd" ^
  -device VGA ^
  -device nec-usb-xhci ^
  -device usb-mouse ^
  -device usb-kbd ^
  -drive if=none,file="%VM_DIR%\system.img",id=hd0 ^
  -device virtio-blk-device,drive=hd0 ^
  -net nic,model=virtio ^
  -net user,hostfwd=tcp::2222-:22

echo.
echo [3/3] 虚拟机已启动
echo     SSH连接命令: ssh -p 2222 root@127.0.0.1
echo     账号：root
echo     密码：kylin@12345
pause
```

* 2、双击 `start_os.bat`，看到登录界面即成功

* 3、启动后在宿主机 `PowerShell` 验证端口转发（检查 `2222` 端口是否监听）

```cmd
netstat -ano | findstr :2222
```

## 六、国产化源配置 ##

> 旨在解决下载慢的痛点

##### 创建脚本文件 #####

```shell
vim /usr/local/setup-sources.sh
```

##### 按一下键盘字母 `i` 进行编辑 #####

##### 输入以下内容： #####

```shell
#!/bin/bash
# setup-sources.sh
# 一键配置麒麟V11 ARM环境的国产化源

set -e

echo "=========================================="
echo "  麒麟V11 ARM64 国产化源配置脚本"
echo "=========================================="
echo ""

# 颜色定义
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# 检查root权限
if [ "$EUID" -ne 0 ]; then 
    echo -e "${RED}错误：请使用 sudo 运行此脚本${NC}"
    exit 1
fi

echo "[1/4] 备份原有配置..."
cp /etc/apt/sources.list /etc/apt/sources.list.bak.$(date +%Y%m%d)
echo -e "${GREEN}✓ APT源已备份${NC}"

echo ""
echo "[2/4] 配置APT清华镜像源..."
cat > /etc/apt/sources.list << 'EOF'
deb https://mirrors.tuna.tsinghua.edu.cn/kylin-ubuntu/v11/ main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/kylin-ubuntu/v11/ main-security restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/kylin-ubuntu/v11/ main-updates restricted universe multiverse
EOF
echo -e "${GREEN}✓ APT源已配置${NC}"

echo ""
echo "[3/4] 更新APT缓存..."
apt clean
apt update -y
echo -e "${GREEN}✓ APT缓存已更新${NC}"

echo ""
echo "[4/4] 配置Pip清华镜像源..."
mkdir -p ~/.pip
cat > ~/.pip/pip.conf << 'EOF'
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn
EOF
echo -e "${GREEN}✓ Pip源已配置${NC}"

echo ""
echo "=========================================="
echo -e "${GREEN}  所有源配置完成！${NC}"
echo "=========================================="
echo ""
echo "验证命令："
echo "  apt update          # 测试APT源"
echo "  pip install requests -v  # 测试Pip源"
echo ""
echo "如配置有误，恢复命令："
echo "  sudo cp /etc/apt/sources.list.bak.* /etc/apt/sources.list"
```

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

##### 赋予执行权限并运行脚本 #####

```shell
chmod +x /usr/local/setup-sources.sh

sudo ./usr/local/setup-sources.sh
```
