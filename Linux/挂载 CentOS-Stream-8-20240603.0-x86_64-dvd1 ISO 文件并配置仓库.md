# 配置 CentOS Stream 8 的本地 YUM 仓库 #

### 🧳 上传 ISO 镜像文件

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： scp 【本地文件地址】 【服务器账号】@【服务器IP地址】:/usr/local/

```shell
scp D:/Documents/CentOS-Stream-8-20240603.0-x86_64-dvd1.iso root@100.110.111.106:/usr/local/
```

### 📀 挂载 ISO 镜像文件

1. **登录服务器**

```shell
ssh root@100.110.111.106
```

2. **创建挂载点**

* 首先，创建一个用于挂载 ISO 文件的目录。

```shell
sudo mkdir -p /mnt/cdrom
```

3. **执行挂载**

* 使用 `mount` 命令将 ISO 文件挂载到刚才创建的目录。请将 `/usr/local/CentOS-Stream-8-20240603.0-x86_64-dvd1.iso` 替换为你存放 ISO 文件的实际路径。

```bash
sudo mount -o loop /usr/local/CentOS-Stream-8-20240603.0-x86_64-dvd1.iso /mnt/cdrom
```
	
* 成功挂载后，系统通常不会有输出信息。你可以通过 `ls /mnt/cdrom` 命令查看是否出现了 `BaseOS` 和 `AppStream` 等目录来确认挂载成功。

### ⚙️ 配置本地 YUM 仓库

1. **备份原有配置**

* 进入 YUM 仓库配置目录，并备份原有的 `.repo` 文件，以防后续需要恢复。

```bash
cd /etc/yum.repos.d/
```

```shell
sudo mkdir backup && sudo mv *.repo backup/
```

2. **创建新的仓库文件**

* 创建一个新的仓库配置文件，例如 `local.repo`。

```bash
sudo vi local.repo
```

3. **写入仓库配置**

* 将以下内容写入 `local.repo` 文件中。CentOS 8 采用了双仓库结构，因此需要分别配置 `BaseOS` 和 `AppStream`。

```ini
[BaseOS]
name=CentOS Stream 8 - BaseOS
baseurl=file:///mnt/cdrom/BaseOS
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial

[AppStream]
name=CentOS Stream 8 - AppStream
baseurl=file:///mnt/cdrom/AppStream
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
```

*   `baseurl`: 指向你挂载 ISO 的路径，并分别指向 `BaseOS` 和 `AppStream` 子目录。
*   `enabled=1`: 启用该仓库。
*   `gpgcheck=1`: 启用 GPG 签名检查，确保软件包的安全性。
*   `gpgkey`: 指定 GPG 公钥的路径，该密钥文件通常位于 ISO 镜像内部。

4.  **导入 GPG 公钥**

* 为了使 GPG 签名检查生效，需要先从 ISO 中导入公钥。

```bash
sudo rpm --import /mnt/cdrom/RPM-GPG-KEY-centosofficial
```

### ✅ 验证配置

* 完成以上步骤后，清理并重建 YUM 缓存，然后检查仓库列表以验证配置是否成功。

```bash
sudo yum clean all
```

```shell
sudo yum makecache
```

```shell
sudo yum repolist
```

* 如果配置正确，`yum repolist` 命令的输出中应该会显示 `BaseOS` 和 `AppStream` 两个仓库的信息。











```bash
yum install gcc-c++
yum install bison
yum install flex
yum install readline-devel
yum install zlib-devel
yum install openssl-devel
yum install libicu-devel
yum install tcl-devel
yum install perl-devel
yum install python3-devel
yum install libxml2-devel
yum install libxslt-devel
yum install openldap-devel
yum install docbook-dtds
yum install docbook-style-xsl
yum install xmlto
yum install fop
yum install pam-devel
yum install gcc
yum install make
yum install lz4-devel
yum install libuuid-devel
yum install systemd-devel
```