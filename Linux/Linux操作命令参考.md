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
#### 11. 创建一个包含了 'file1', 'file2' 以及 'dir1'的档案文件 ####
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
#### 1.  ####
```shell

```
#### 2.  ####
```shell

```
#### 3.  ####
```shell

```
#### 4.  ####
```shell

```
#### 5.  ####
```shell

```
#### 6.  ####
```shell

```
#### 7.  ####
```shell

```
#### 8.  ####
```shell

```
## 一五. APT软件工具 (Debian, Ubuntu及类似系统) ##
#### 1.  ####
```shell

```
#### 2.  ####
```shell

```
#### 3.  ####
```shell

```
#### 4.  ####
```shell

```
#### 5.  ####
```shell

```
#### 6.  ####
```shell

```
#### 7.  ####
```shell

```
#### 8.  ####
```shell

```
## 一六. 查看文件内容 ##
#### 1.  ####
```shell

```
#### 2.  ####
```shell

```
#### 3.  ####
```shell

```
#### 4.  ####
```shell

```
#### 5.  ####
```shell

```
#### 6.  ####
```shell

```
#### 7.  ####
```shell

```
## 一七. 文本处理 ##
#### 1.  ####
```shell

```
#### 2.  ####
```shell

```
#### 3.  ####
```shell

```
#### 4.  ####
```shell

```
#### 5.  ####
```shell

```
#### 6.  ####
```shell

```
#### 7.  ####
```shell

```
#### 8.  ####
```shell

```
#### 9.  ####
```shell

```
#### 10.  ####
```shell

```
#### 11.  ####
```shell

```
#### 12.  ####
```shell

```
#### 13.  ####
```shell

```
#### 14.  ####
```shell

```
#### 15.  ####
```shell

```
#### 16.  ####
```shell

```
#### 17.  ####
```shell

```
#### 18.  ####
```shell

```
#### 19.  ####
```shell

```
#### 20.  ####
```shell

```
#### 21.  ####
```shell

```
#### 22.  ####
```shell

```
#### 23.  ####
```shell

```
#### 24.  ####
```shell

```
#### 25.  ####
```shell

```
#### 26.  ####
```shell

```
#### 27.  ####
```shell

```
#### 28.  ####
```shell

```
#### 29.  ####
```shell

```
#### 30.  ####
```shell

```






## 二零.  ##
#### 1.  ####
```shell

```
#### 2.  ####
```shell

```
#### 3.  ####
```shell

```
#### 4.  ####
```shell

```
#### 5.  ####
```shell

```
#### 6.  ####
```shell

```
#### 7.  ####
```shell

```
#### 8.  ####
```shell

```
#### 9.  ####
```shell

```
#### 10.  ####
```shell

```
#### 11.  ####
```shell

```
#### 12.  ####
```shell

```
#### 13.  ####
```shell

```
#### 14.  ####
```shell

```
#### 15.  ####
```shell

```
#### 16.  ####
```shell

```
#### 17.  ####
```shell

```
#### 18.  ####
```shell

```
#### 19.  ####
```shell

```
#### 20.  ####
```shell

```
#### 21.  ####
```shell

```
#### 22.  ####
```shell

```
#### 23.  ####
```shell

```
#### 24.  ####
```shell

```
#### 25.  ####
```shell

```
#### 26.  ####
```shell

```
#### 27.  ####
```shell

```
#### 28.  ####
```shell

```
#### 29.  ####
```shell

```
#### 30.  ####
```shell

```