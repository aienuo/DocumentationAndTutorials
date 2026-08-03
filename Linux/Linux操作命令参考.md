# Linux操作命令参考 #

> 实际操作机器为 `CentOS-8.2` 如果没有 `CentOS` 请 [参考在线安装CentOS-8.2](在线安装CentOS-8.2.2004-x86_64.md)

## 零一. 查看Linux系统信息 ##

#### 1. 显示机器的处理器架构 ####

##### ① #####

```shell
arch
```

##### ② #####

```shell
uname -m
```

#### 2. 显示正在使用的内核版本 ####

```shell
uname -r
```

#### 3. 显示硬件系统部件 - (SMBIOS / DMI) ####

```shell
dmidecode -q
```

#### 4. 罗列一个磁盘的架构特性 ####

```shell
hdparm -i /dev/hda
```

#### 5. 在磁盘上执行测试性读取操作 ####

```shell
hdparm -tT /dev/sda
```

#### 6. 显示CPU info的信息 ####

```shell
cat /proc/cpuinfo
```

#### 7. 显示中断 ####

```shell
cat /proc/interrupts
```

#### 8. 校验内存使用 ####

```shell
cat /proc/meminfo
```

#### 9. 显示哪些swap被使用 ####

```shell
cat /proc/swaps
```

#### 10. 显示内核的版本 ####

```shell
cat /proc/version
```

#### 12. 显示网络适配器及统计 ####

```shell
cat /proc/net/dev
```

#### 13. 显示已加载的文件系统 ####

```shell
cat /proc/mounts
```

#### 14. 罗列 PCI 设备 ####

```shell
lspci -tv
```

#### 15. 显示 USB 设备 ####

```shell
lsusb -tv
```

## 零二. Date 显示系统日期 ##

#### 1. 显示2021年的日历表（阳历） ####

```shell
cal 2021
```

#### 2. 设置日期和时间 - 月日时分年.秒 ####

```shell
date 111012002021.00
```

#### 3. 将时间修改保存到 BIOS ####

```shell
clock -w
```

## 零三. 关机（关闭、重启、注销） ##

#### 1. 关闭 ####

##### ① #####

```shell
shutdown -h now 
```

##### ② #####

```shell

init 0 
```

##### ③ #####

```shell
telinit 0
```

##### ④ 按预定时间关闭系统 #####

```shell
shutdown -h hours:minutes &
```

##### ⑤ 取消按预定时间关闭系统 #####

```shell
shutdown -c
```

#### 2. 重启 ####

##### ① #####

```shell
shutdown -r now 
```

##### ② #####

```shell
reboot
```

#### 3. 注销 ####

```shell
logout
```

## 零四. 用户和群组 ##

#### 1. 创建一个新用户组 ####

```shell
groupadd group_name
```

#### 2. 删除一个用户组 ####

```shell
groupdel group_name
```

#### 3. 重命名一个用户组 ####

```shell
groupmod -n new_group_name old_group_name
```

#### 4. 创建一个属于 `admin` 用户组的用户 ####

```shell
useradd -c "Name Surname " -g admin -d /home/user1 -s /bin/bash user1
```

#### 5. 创建一个新用户 ####

```shell
useradd user1
```

#### 6. 删除一个用户（`-r` 排除主目录） ####

```shell
userdel -r user1
```

#### 7. 修改用户属性 ####

```shell
usermod -c "User FTP" -g system -d /ftp/user1 -s /bin/nologin user1
```

#### 8. 修改口令 ####

```shell
passwd
```

#### 9. 修改一个用户的口令 (只允许`root`执行) ####

```shell
passwd user1
```

#### 10. 设置用户口令的失效期限 ####

```shell
chage -E 2021-12-31 user
```

#### 11. 检查 `/etc/passwd` 的文件格式和语法修正以及存在的用户 ####

```shell
pwck
```

#### 12. 检查 `/etc/passwd` 的文件格式和语法修正以及存在的群组 ####

```shell
grpck
```

#### 13. 登陆进一个新的群组以改变新创建文件的预设群组 ####

```shell
newgrp group_name
```

## 零五. 磁盘空间相关 ##

#### 1. 显示已经挂载的分区列表 ####

```shell
df -h
```

#### 2. 以尺寸大小排列文件和目录 ####

```shell
ls -lSr |more
```

#### 3. 估算目录 `dir1` 已经使用的磁盘空间 ####

```shell
du -sh dir1
```

#### 4. 以容量大小为依据依次显示文件和目录的大小 ####

```shell
du -sk * | sort -rn
```

#### 5. 以大小为依据依次显示已安装的rpm包所使用的空间（`fedora`, `redhat`类系统） s####

```shell
rpm -q -a --qf '%10{SIZE}t%{NAME}n' | sort -k1,1n
```

#### 6. 以大小为依据显示已安装的deb包所使用的空间（`ubuntu`, `debian`类系统)） ####

```shell
dpkg-query -W -f='${Installed-Size;10}t${Package}n' | sort -k1,1n
```

## 零六. 挂载文件系统 ##

#### 1. 挂载一个叫做hda2的盘 - 确定目录 `/mnt/hda2` 已经存在 ####

```shell
mount /dev/hda2 /mnt/hda2
```

#### 2. 卸载一个叫做hda2的盘 - 先从挂载点 `/mnt/hda2` 退出 ####

```shell
umount /dev/hda2
```

#### 3. 当设备繁忙时强制卸载 ####

```shell
fuser -km /mnt/hda2
```

#### 4. 运行卸载操作而不写入 `/etc/mtab` 文件- 当文件为只读或当磁盘写满时非常有用 ####

```shell
umount -n /mnt/hda2
```

#### 5. 挂载一个软盘 ####

```shell
mount /dev/fd0 /mnt/floppy
```

#### 6. 挂载一个`cdrom`或`dvdrom` ####

```shell
mount /dev/cdrom /mnt/cdrom
```

#### 7. 挂载一个`cdrw`或`dvdrom` ####

```shell
mount /dev/hdc /mnt/cdrecorder
```

#### 8. 挂载一个`cdrw`或`dvdrom` ####

```shell
mount /dev/hdb /mnt/cdrecorder
```

#### 9. 挂载一个文件或ISO镜像文件 ####

```shell
mount -o loop file.iso /mnt/cdrom
```

#### 10. 挂载一个Windows FAT32文件系统 ####

```shell
mount -t vfat /dev/hda5 /mnt/hda5
```

#### 11. 挂载一个usb 捷盘或闪存设备 ####

```shell
mount /dev/sda1 /mnt/usbdisk
```

#### 12. 挂载一个windows网络共享 ####

```shell
mount -t smbfs -o username=user,password=pass //WinClient/share /mnt/share
```

## 零七. 文件和目录 ##

#### 1. 进入 `/home` 目录 ####

```shell
cd /home
```

#### 2. 返回上一级目录 ####

```shell
cd ..
```

#### 3. 返回上两级目录 ####

```shell
cd ../..
```

#### 4. 进入个人的主目录 ####

```shell
cd
```

#### 5. 进入指定人的主目录 ####

```shell
cd ~user1
```

#### 6. 返回上次所在的目录 ####

```shell
cd -
```

#### 7. 显示工作路径 ####

```shell
pwd
```

#### 8. 查看目录中的文件 ####

```shell
ls
```

#### 9. 查看目录中的文件 ####

```shell
ls -F
```

#### 10. 查看目录中的文件 ####

```shell
ls -f
```

#### 11. 显示文件和目录的详细资料 ####

```shell
ls -l
```

#### 12. 显示隐藏文件 ####

```shell
ls -a
```

#### 13. 显示包含数字的文件名和目录名 ####

```shell
ls *[0-9]*
```

#### 14. 显示文件和目录由根目录开始的树形结构 ####

##### ① #####

```shell
tree
```

##### ② #####

```shell
lstree
```

#### 15. 创建一个叫做 `dir1` 的目录 ####

```shell
mkdir dir1
```

#### 16. 同时创建两个目录 ####

```shell
mkdir dir1 dir2
```

#### 17. 创建一个目录树 ####

```shell
mkdir -p /tmp/dir1/dir2
```

#### 18. 删除一个叫做 `file1` 的文件 ####

```shell
rm -f file1
```

#### 19. 删除一个叫做 `dir1` 的目录 ####

```shell
rmdir dir1
```

#### 20. 删除一个叫做 `dir1` 的目录并同时删除其内容 ####

```shell
rm -rf dir1
```

#### 21. 同时删除两个目录及它们的内容 ####

```shell
rm -rf dir1 dir2
```

#### 22. 重命名/移动 一个目录 ####

```shell
mv dir1 new_dir
```

#### 23. 复制一个文件 ####

```shell
cp file1 file2
```

#### 24. 复制一个目录下的所有文件到当前工作目录 ####

```shell
cp dir/* . 
```

#### 25. 复制一个目录到当前工作目录 ####

```shell
cp -a /tmp/dir1 .
```

#### 26. 复制一个目录 ####

```shell
cp -a dir1 dir2
```

#### 27. 创建一个指向文件或目录的软链接 ####

```shell
ln -s file1 lnk1
```

#### 28. 创建一个指向文件或目录的物理链接 ####

```shell
ln file1 lnk1
```

#### 29. 修改一个文件或目录的时间戳 - (YYMMDDhhmm) ####

```shell
touch -t 2114470000 file1
```

#### 30. 将文件的 MIME 类型输出为文本 ####

```shell
file file1 
```

#### 31. 列出已知的编码 ####

```shell
iconv -l
```

#### 32. 通过假设它以 fromEncoding 编码并将其转换为 toEncoding，从给定的输入文件创建一个新文件 ####

```shell
iconv -f fromEncoding -t toEncoding inputFile > outputFile
```

#### 33. 批量调整当前目录中的文件并将它们发送到缩略图目录（需要从 Imagemagick 转换） ####

```shell
find . -maxdepth 1 -name *.jpg -print -exec convert "{}" -resize 80x60 "thumbs/{}" \;
```

## 零八. 文件搜索 ##

#### 1. 从 `/` 开始进入根文件系统搜索文件和目录 ####

```shell
find / -name file1
```

#### 2. 搜索属于用户 `user1` 的文件和目录 ####

```shell
find / -user user1
```

#### 3. 在目录 `/home/user1` 中搜索带有 `.bin` 结尾的文件 ####

```shell
find /home/user1 -name \*.bin
```

#### 4. 搜索在过去100天内未被使用过的执行文件 ####

```shell
find /usr/bin -type f -atime +100
```

#### 5. 搜索在10天内被创建或者修改过的文件 ####

```shell
find /usr/bin -type f -mtime -10
```

#### 6. 搜索以 `.rpm` 结尾的文件并定义其权限 ####

```shell
find / -name \*.rpm -exec chmod 755 '{}' \;
```

#### 7. 搜索以 `.rpm` 结尾的文件，忽略光驱、捷盘等可移动设备  ####

```shell
find / -xdev -name \*.rpm
```

#### 8. 寻找以 `.ps` 结尾的文件 - 先运行 `updatedb` 命令 ####

```shell
locate \*.ps
```

#### 9. 显示一个二进制文件、源码或man的位置 ####

```shell
whereis halt
```

#### 10. 显示一个二进制文件或可执行文件的完整路径 ####

```shell
which halt
```

## 零九. 文件权限 ##

> 使用 "+" 设置权限，使用 "-" 用于取消

#### 1. 显示权限 ####

```shell
ls -lh
```

#### 2. 将终端划分成5栏显示 ####

```shell
ls /tmp | pr -T5 -W$COLUMNS
```

#### 3. 设置目录的所有人(u)、群组(g)以及其他人(o)以读（r ）、写(w)和执行(x)的权限 ####

```shell
chmod ugo+rwx directory1
```

#### 4. 删除群组(g)与其他人(o)对目录的读写执行权限 ####

```shell
chmod go-rwx directory1 
```

#### 5. 改变一个文件的所有人属性 ####

```shell
chown user1 file1
```

#### 6. 改变一个目录的所有人属性并同时改变改目录下所有文件的属性 ####

```shell
chown -R user1 directory1
```

#### 7. 改变文件的群组 ####

```shell
chgrp group1 file1 
```

#### 8. 改变一个文件的所有人和群组属性 ####

```shell
chown user1:group1 file1
```

#### 9. 罗列一个系统中所有使用了SUID控制的文件 ####

```shell
find / -perm -u+s
```

#### 10. 设置一个二进制文件的 SUID 位 - 运行该文件的用户也被赋予和所有者同样的权限 ####

```shell
chmod u+s /bin/file1 
```

#### 11. 禁用一个二进制文件的 SUID位 ####

```shell
chmod u-s /bin/file1
```

#### 12. 设置一个目录的SGID 位 - 类似SUID ，不过这是针对目录的 ####

```shell
chmod g+s /home/public
```

#### 13. 禁用一个目录的 SGID 位 ####

```shell
chmod g-s /home/public
```

#### 14. 设置一个文件的 STIKY 位 - 只允许合法所有人删除文件 ####

```shell
chmod o+t /home/public
```

#### 15. 禁用一个目录的 STIKY 位 ####

```shell
chmod o-t /home/public
```

## 一零. 文件的特殊属性 ##

> 使用 "+" 设置权限，使用 "-" 用于取消

#### 1. 只允许以追加方式读写文件 ####

```shell
chattr +a file1
```

#### 2. 允许这个文件能被内核自动压缩/解压 ####

```shell
chattr +c file1
```

#### 3. 在进行文件系统备份时，dump程序将忽略这个文件 ####

```shell
chattr +d file1
```

#### 4. 设置成不可变的文件，不能被删除、修改、重命名或者链接 ####

```shell
chattr +i file1
```

#### 5. 允许一个文件被安全地删除 ####

```shell
chattr +s file1
```

#### 6. 一旦应用程序对这个文件执行了写操作，使系统立刻把修改的结果写到磁盘 ####

```shell
chattr +S file1
```

#### 7. 若文件被删除，系统会允许你在以后恢复这个被删除的文件 ####

```shell
chattr +u file1
```

#### 8. 显示特殊的属性 ####

```shell
lsattr
```

## 一一. 打包和压缩文件 ##

#### 1. 解压一个叫做 `xxx.bz2` 的文件 ####

```shell
bunzip2 xxx.bz2
```

#### 2. 压缩一个叫做 `xxx` 的文件 ####

```shell
bzip2 xxx
```

#### 3. 解压一个叫做 `xxx.gz`  的文件 ####

```shell
gunzip xxx.gz
```

#### 4. 压缩一个叫做 `xxx` 的文件 ####

```shell
gzip xxx
```

#### 5. 最大程度压缩 `xxx` 文件 ####

```shell
gzip -9 xxx
```

#### 6. 创建一个叫做 `file1.rar` 的包 ####

```shell
rar a file1.rar test_file
```

#### 7. 同时压缩 `file1`, `file2` 以及目录 `dir1` ####

```shell
rar a file1.rar file1 file2 dir1
```

#### 8. 解压rar包 ####

```shell
rar x file1.rar
```

#### 9. 解压rar包 ####

```shell
unrar x file1.rar
```

#### 10. 创建一个非压缩的 tarball ####

```shell
tar -cvf archive.tar file1
```

#### 11. 创建一个包含了 `file1`, `file2` 以及 `dir1`的档案文件 ####

```shell
tar -cvf archive.tar file1 file2 dir1
```

#### 12. 显示一个包中的内容 ####

```shell
tar -tf archive.tar
```

#### 13. 释放一个包 ####

```shell
tar -xvf archive.tar
```

#### 14. 将压缩包释放到 /tmp目录下 ####

```shell
tar -xvf archive.tar -C /tmp
```

#### 15. 创建一个bzip2格式的压缩包 ####

```shell
tar -cvfj archive.tar.bz2 dir1
```

#### 16. 解压一个bzip2格式的压缩包 ####

```shell
tar -jxvf archive.tar.bz2
```

#### 17. 创建一个gzip格式的压缩包 ####

```shell
tar -cvfz archive.tar.gz dir1
```

#### 18. 解压一个gzip格式的压缩包 ####

```shell
tar -zxvf archive.tar.gz
```

#### 19. 创建一个zip格式的压缩包 ####

```shell
zip file1.zip file1
```

#### 20. 将几个文件和目录同时压缩成一个zip格式的压缩包 ####

```shell
zip -r file1.zip file1 file2 dir1
```

#### 21. 解压一个zip格式压缩包 ####

```shell
unzip file1.zip
```

## 一二. RPM包（Fedora, Redhat及类似系统） ##

#### 1. 安装一个rpm包 ####

```shell
rpm -ivh package.rpm 
```

#### 2. 安装一个rpm包而忽略依赖关系警告 ####

```shell
rpm -ivh --nodeeps package.rpm
```

#### 3. 更新一个rpm包但不改变其配置文件 ####

```shell
rpm -U package.rpm
```

#### 4. 更新一个确定已经安装的rpm包 ####

```shell
rpm -F package.rpm
```

#### 5. 删除一个rpm包 ####

```shell
rpm -e package_name.rpm
```

#### 6. 显示系统中所有已经安装的rpm包 ####

```shell
rpm -qa
```

#### 7. 显示所有名称中包含 "httpd" 字样的rpm包 ####

```shell
rpm -qa | grep httpd
```

#### 8. 获取一个已安装包的特殊信息 ####

```shell
rpm -qi package_name
```

#### 9. 显示一个组件的rpm包 ####

```shell
rpm -qg "System Environment/Daemons"
```

#### 10. 显示一个已经安装的rpm包提供的文件列表 ####

```shell
rpm -ql package_name
```

#### 11. 显示一个已经安装的rpm包提供的配置文件列表 ####

```shell
rpm -qc package_name
```

#### 12. 显示与一个rpm包存在依赖关系的列表 ####

```shell
rpm -q package_name --whatrequires
```

#### 13. 显示一个rpm包所占的体积 ####

```shell
rpm -q package_name --whatprovides
```

#### 14. 显示在安装/删除期间所执行的脚本 ####

```shell
rpm -q package_name --scripts
```

#### 15. 显示一个rpm包的修改历史 ####

```shell
rpm -q package_name --changelog
```

#### 16. 确认所给的文件由哪个rpm包所提供 ####

```shell
rpm -qf /etc/httpd/conf/httpd.conf
```

#### 17. 显示由一个尚未安装的rpm包提供的文件列表 ####

```shell
rpm -qp package.rpm -l
```

#### 18. 导入公钥数字证书 ####

```shell
rpm --import /media/cdrom/RPM-GPG-KEY
```

#### 19. 确认一个rpm包的完整性 ####

```shell
rpm --checksig package.rpm
```

#### 20. 确认已安装的所有rpm包的完整性 ####

```shell
rpm -qa gpg-pubkey
```

#### 21. 检查文件尺寸、 许可、类型、所有者、群组、MD5检查以及最后修改时间 ####

```shell
rpm -V package_name
```

#### 22. 检查系统中所有已安装的rpm包- 小心使用 ####

```shell
rpm -Va
```

#### 23. 确认一个rpm包还未安装 ####

```shell
rpm -Vp package.rpm
```

#### 24. 从一个rpm包运行可执行文件 ####

```shell
rpm2cpio package.rpm | cpio --extract --make-directories *bin*
```

#### 25. 从一个rpm源码安装一个构建好的包 ####

```shell
rpm -ivh /usr/src/redhat/RPMS/`arch`/package.rpm
```

#### 26. 从一个rpm源码构建一个 rpm 包 ####

```shell
rpmbuild --rebuild package_name.src.rpm
```

## 一三. YUM（Fedora, RedHat及类似系统） ##

#### 1. 下载并安装一个rpm包 ####

```shell
yum install package_name
```

#### 2. 将安装一个rpm包，使用你自己的软件仓库为你解决所有依赖关系 ####

```shell
yum localinstall package_name.rpm
```

#### 3. 更新当前系统中所有安装的rpm包 ####

```shell
yum update package_name.rpm
```

#### 4. 更新一个rpm包 ####

```shell
yum update package_name
```

#### 5. 删除一个rpm包 ####

```shell
yum remove package_name
```

#### 6. 列出当前系统中安装的所有包 ####

```shell
yum list
```

#### 7. 在rpm仓库中搜寻软件包 ####

```shell
yum search package_name
```

#### 8. 清理rpm缓存删除下载的包 ####

```shell
yum clean packages
```

#### 9. 删除所有头文件 ####

```shell
yum clean headers
```

#### 10. 删除所有缓存的包和头文件 ####

```shell
yum clean all
```

## 一四. DEB包 (Debian, Ubuntu及类似系统) ##

#### 1. 安装/更新一个 deb 包 ####

```shell
dpkg -i package.deb
```

#### 2. 从系统删除一个 deb 包 ####

```shell
dpkg -r package_name
```

#### 3. 显示系统中所有已经安装的 deb 包 ####

```shell
dpkg -l
```

#### 4. 显示所有名称中包含 "httpd" 字样的deb包 ####

```shell
dpkg -l | grep httpd
```

#### 5. 获得已经安装在系统中一个特殊包的信息 ####

```shell
dpkg -s package_name
```

#### 6. 显示系统中已经安装的一个deb包所提供的文件列表 ####

```shell
dpkg -L package_name
```

#### 7. 显示尚未安装的一个包所提供的文件列表 ####

```shell
dpkg --contents package.deb
```

#### 8. 确认所给的文件由哪个deb包提供 ####

```shell
dpkg -S /bin/ping
```

## 一五. APT软件工具 (Debian, Ubuntu及类似系统) ##

#### 1. 安装/更新一个 deb 包 ####

```shell
apt-get install package_name
```

#### 2. 从光盘安装/更新一个 deb 包 ####

```shell
apt-cdrom install package_name
```

#### 3. 升级列表中的软件包 ####

```shell
apt-get update
```

#### 4. 升级所有已安装的软件 ####

```shell
apt-get upgrade
```

#### 5. 从系统删除一个deb包 ####

```shell
apt-get remove package_name
```

#### 6. 确认依赖的软件仓库正确 ####

```shell
apt-get check
```

#### 7. 从下载的软件包中清理缓存 ####

```shell
apt-get clean
```

#### 8. 返回包含所要搜索字符串的软件包名称 ####

```shell
apt-cache search searched-package
```

## 一六. 查看文件内容 ##

#### 1. 从第一个字节开始正向查看文件的内容 ####

```shell
cat file1
```

#### 2. 从最后一行开始反向查看一个文件的内容 ####

```shell
tac file1
```

#### 3. 查看一个长文件的内容 ####

```shell
more file1
```

#### 4. 类似于 `more` 命令，但是它允许在文件中和正向操作一样的反向操作 ####

```shell
less file1
```

#### 5. 查看一个文件的前两行 ####

```shell
head -2 file1
```

#### 6. 查看一个文件的最后两行 ####

```shell
tail -2 file1
```

#### 7. 实时查看被添加到一个文件中的内容 ####

```shell
tail -f /var/log/messages
```

## 一七. 文本处理 ##

#### 1.  ####

```shell
cat file1 file2 ... | command <> file1_in.txt_or_file1_out.txt general syntax for text manipulation using PIPE, STDIN and STDOUT

cat file1 | command( sed, grep, awk, grep, etc...) > result.txt # 合并一个文件的详细说明文本，并将简介写入一个新文件中
cat file1 | command( sed, grep, awk, grep, etc...) >> result.txt # 合并一个文件的详细说明文本，并将简介写入一个已有的文件中

grep Aug /var/log/messages     # 在文件 `/var/log/messages`中查找关键词"Aug"
grep ^Aug /var/log/messages    # 在文件 `/var/log/messages`中查找以"Aug"开始的词汇
grep [0-9] /var/log/messages   # 选择 `/var/log/messages` 文件中所有包含数字的行
grep Aug -R /var/log/*         # 在目录 `/var/log` 及随后的目录中搜索字符串"Aug"

sed 's/string1/string2/g' example.txt # 将 `example.txt` 文件中的 "string1" 替换成 "string2"
sed '/^$/d' example.txt           # 从 `example.txt` 文件中删除所有空白行
sed '/ *#/d; /^$/d' example.txt   # 从 `example.txt` 文件中删除所有注释和空白行
echo 'esempio' | tr '[:lower:]' '[:upper:]'    # 合并上下单元格内容
sed -e '1d' result.txt          # 从文件 `example.txt` 中排除第一行
sed -n '/string1/p'            # 查看只包含词汇 "string1" 的行
sed -e 's/ *$//' example.txt    # 删除每一行最后的空白字符
sed -e 's/string1//g' example.txt  # 从文档中只删除词汇 "string1" 并保留剩余全部
sed -n '1,5p;5q' example.txt     # 查看从第一行到第5行内容
sed -n '5p;5q' example.txt       # 查看第5行
sed -e 's/00*/0/g' example.txt   # 用单个零替换多个零

```

#### 2.  ####

```shell

```

#### 19. 标示文件的行数 ####

```shell
cat -n file1
```

#### 20. 删除 `example.txt` 文件中的所有偶数行 ####

```shell
cat example.txt | awk 'NR%2==1'
```

#### 21. 查看一行第一栏 ####

```shell
echo a b c | awk '{print $1}'
```

#### 22. 查看一行的第一和第三栏 ####

```shell
echo a b c | awk '{print $1,$3}'
```

#### 23. 合并两个文件或两栏的内容 ####

```shell
paste file1 file2
```

#### 24. 合并两个文件或两栏的内容，中间用"+"区分 ####

```shell
paste -d '+' file1 file2
```

#### 25. 排序两个文件的内容 ####

```shell
sort file1 file2
```

#### 26. 取出两个文件的并集(重复的行只保留一份) ####

```shell
sort file1 file2 | uniq
```

#### 27. 删除交集，留下其他的行 ####

```shell
sort file1 file2 | uniq -u
```

#### 28. 取出两个文件的交集(只留下同时存在于两个文件中的文件) ####

```shell
sort file1 file2 | uniq -d
```

#### 29. 比较两个文件的内容只删除 `file1` 所包含的内容 ####

```shell
comm -1 file1 file2
```

#### 30. 比较两个文件的内容只删除 `file2` 所包含的内容 ####

```shell
comm -2 file1 file2
```

#### 31. 比较两个文件的内容只删除两个文件共有的部分 ####

```shell
comm -3 file1 file2
```

## 十八. 字符设置和文件格式转换 ##

#### 1. 将一个文本文件的格式从 `MSDOS` 转换成 `UNIX` ####

```shell
dos2unix filedos.txt fileunix.txt
```

#### 2. 将一个文本文件的格式从 `UNIX` 转换成 `MSDOS` ####

```shell
unix2dos fileunix.txt filedos.txt
```

#### 3. 将一个文本文件转换成 `html` ####

```shell
recode ..HTML < page.txt > page.html
```

#### 4. 显示所有允许的转换格式 ####

```shell
recode -l | more
```

## 十九. 文件系统分析 ##

#### 1. 检查磁盘hda1上的坏磁块 ####

```shell
badblocks -v /dev/hda1
```

#### 2. 修复/检查hda1磁盘上linux文件系统的完整性 ####

```shell
fsck /dev/hda1
```

#### 3. 修复/检查hda1磁盘上ext2文件系统的完整性 ####

```shell
fsck.ext2 /dev/hda1
```

#### 4. 修复/检查hda1磁盘上ext2文件系统的完整性 ####

```shell
e2fsck /dev/hda1
```

#### 5. 修复/检查hda1磁盘上ext3文件系统的完整性 ####

```shell
e2fsck -j /dev/hda1
```

#### 6. 修复/检查hda1磁盘上ext3文件系统的完整性 ####

```shell
fsck.ext3 /dev/hda1
```

#### 7. 修复/检查hda1磁盘上fat文件系统的完整性 ####

```shell
fsck.vfat /dev/hda1
```

#### 8. 修复/检查hda1磁盘上dos文件系统的完整性 ####

```shell
fsck.msdos /dev/hda1
```

#### 9. 修复/检查hda1磁盘上dos文件系统的完整性 ####

```shell
dosfsck /dev/hda1
```

## 二零. 初始化一个文件系统 ##

#### 1. 在hda1分区创建一个文件系统 ####

```shell
mkfs /dev/hda1
```

#### 2. 在hda1分区创建一个linux ext2的文件系统 ####

```shell
mke2fs /dev/hda1
```

#### 3. 在hda1分区创建一个linux ext3(日志型)的文件系统 ####

```shell
mke2fs -j /dev/hda1
```

#### 4. 创建一个 FAT32 文件系统 ####

```shell
mkfs -t vfat 32 -F /dev/hda1
```

#### 5. 格式化一个软盘 ####

```shell
fdformat -n /dev/fd0
```

#### 6. 创建一个swap文件系统 ####

```shell
mkswap /dev/hda3
```

## 二一. SWAP文件系统 ##

#### 1. 创建一个swap文件系统 ####

```shell
mkswap /dev/hda3
```

#### 2. 启用一个新的swap文件系统 ####

```shell
swapon /dev/hda3
```

#### 3. 启用两个swap分区 ####

```shell
swapon /dev/hda2 /dev/hdb3
```

## 二二. 备份 ##

#### 1. 制作一个 `/home` 目录的完整全量备份 ####

> 备份整个 `/home` 文件系统的所有文件（无论是否变化）

```shell
dump -0aj -f /tmp/home0.bak /home
```

#### 2. 制作一个 `/home` 目录的增量备份（一级） ####

> 创建一个增量备份，备份自最近一次级别 0（或更低级别） 备份以来，所有发生过变动的文件

```shell
dump -1aj -f /tmp/home1.bak /home

```

#### 3. 还原备份 ####

```shell
restore -rf /tmp/home0.bak
```

> 如果你只是想验证备份包里的内容，或者只找回某个丢失的配置文件

```shell
restore -if /tmp/home0.bak
```

#### 4.  ####

```shell
rsync -rogpav --delete /home /tmp    #同步两边的目录
rsync -rogpav -e ssh --delete /home ip_address:/tmp           #通过SSH通道rsync
rsync -az -e ssh --delete ip_addr:/home/public /home/local    #通过ssh和压缩将一个远程目录同步到本地目录
rsync -az -e ssh --delete /home/local ip_addr:/home/public    #通过ssh和压缩将本地目录同步到远程目录

dd bs=1M if=/dev/hda | gzip | ssh user@ip_addr 'dd of=hda.gz'  #通过ssh在远程主机上执行一次备份本地磁盘的操作
dd if=/dev/sda of=/tmp/file1 #备份磁盘内容到一个文件
tar -Puf backup.tar /home/user # 执行一次对 '/home/user' 目录的交互式备份操作
( cd /tmp/local/ && tar c . ) | ssh -C user@ip_addr 'cd /home/share/ && tar x -p' #通过ssh在远程目录中复制一个目录内容
( tar c /home ) | ssh -C user@ip_addr 'cd /home/backup-home && tar x -p' #通过ssh在远程目录中复制一个本地目录
tar cf - . | (cd /tmp/backup ; tar xf - ) #本地将一个目录复制到另一个地方，保留原有权限及链接

find /home/user1 -name '*.txt' | xargs cp -av --target-directory=/home/backup/ --parents #从一个目录查找并复制所有以 `.txt` 结尾的文件到另一个目录
find /var/log -name '*.log' | tar cv --files-from=- | bzip2 > log.tar.bz2 #查找所有以 `.log` 结尾的文件并做成一个bzip包

dd if=/dev/hda of=/dev/fd0 bs=512 count=1 #做一个将 MBR (Master Boot Record)内容复制到软盘的动作
dd if=/dev/fd0 of=/dev/hda bs=512 count=1 #从已经保存到软盘的备份中恢复MBR内容
```

## 二三. 网络 - （以太网和WIFI无线） ##

#### 1. 显示所有以太网卡的配置 ####

```shell
ifconfig
```

#### 2. 显示 `eth0` 以太网卡的配置 ####

```shell
ifconfig eth0
```

#### 3. 启用一个 `eth0` 网络设备 ####

```shell
ifup eth0
```

#### 4. 禁用一个 `eth0` 网络设备 ####

```shell
ifdown eth0
```

#### 5. 控制 `eth0` IP地址 ####

```shell
ifconfig eth0 192.168.1.1 netmask 255.255.255.0
```

#### 6. 设置 `eth0` 成混杂模式以嗅探数据包 (sniffing) ####

```shell
ifconfig eth0 promisc
```

#### 7. 以 dhcp模式 启用 `eth0` ####

```shell
dhclient eth0
```

#### 8. 查看路由表 ####

```shell
route -n
```

#### 9. 配置默认网关 ####

```shell
route add -net 0/0 gw IP_Gateway
```

#### 10. 配置静态路由到达网络 `192.168.0.0/16` ####

```shell
route add -net 192.168.0.0 netmask 255.255.0.0 gw 192.168.1.1
```

#### 11. 删除静态路由 ####

```shell
route del 0/0 gw IP_gateway
```

#### 12. 查看机器名 ####

```shell
hostname
```

#### 13. 把一个主机名解析到一个网际地址或把一个网际地址解析到一个主机名 ####

```shell
host www.example.com
```

#### 14. 用于查询DNS的记录，查看域名解析是否正常，在网络故障的时候用来诊断网络问题 ####

```shell
nslookup www.example.com
```

#### 15. 查看网卡信息 ####

```shell
ip link show
```

#### 16. 用于查看、管理介质的网络接口的状态 ####

```shell
mii-tool
```

#### 17. 用于查询和设置网卡配置 ####

```shell
ethtool
```

#### 18. 用于显示TCP/UDP的状态信息 ####

```shell
netstat -tupl
```

#### 19. 显示所有http协议的流量 ####

```shell
tcpdump tcp port 80
```
