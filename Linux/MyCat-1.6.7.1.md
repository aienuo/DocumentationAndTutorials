# Contos7环境下MyCat-1.6.7.1安装配置 #
## 一、下载Mycat ##
	wget http://dl.mycat.io/1.6.7.1/Mycat-server-1.6.7.1-release-20190627191042-linux.tar.gz
## 二、解压操作文件 ##
### 1、解压 ###
	tar -zxvf Mycat-server-1.6.7.1-release-20190627191042-linux.tar.gz -C /usr/local/
## 三、创建用户和组 ##
### 1、创建组 ###
	groupadd mycat
### 2、添加用户 ###
	adduser -r -g mycat mycat
### 3、修改mycat目录所属的用户和组为mycat用户 ###
	chown -R mycat.mycat /usr/local/mycat/
## 四、配置文件配置 ##
### 切换到Mycat的 `conf` 目录下 ###
	cd /usr/local/mycat/conf/
##### `schema`标签内的内容大小写问题：必须统一一致 #####
### 1、`schema.xml` 文件 ###
	vim /schema.xml
##### 注意修改 `schema` `dataNode` `dataHost`标签内的内容 #####
	<?xml version="1.0"?>
	<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
	<mycat:schema xmlns:mycat="http://io.mycat/">

		<!-- 逻辑库 -->
		<schema name="propertyw" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1"></schema>
		<!--
			checkSQLschema ： 检查数据库名称
			sqlMaxLimit ： 避免造成过多输出，当SQL语句指定limit的大小不受该属性约束
			dataNode ： 该属性用于绑定逻辑库到某个具体的 database（数据库）上
		-->
		<schema name="anbao" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn2"></schema>

		<!-- 节点 -->
		<dataNode name="dn1" dataHost="localhost" database="propertyw" />
		<!--
			name ：定义数据节点的名字，这个名字需要是唯一的
			dataHost ：该属性用于定义该分片属于哪个数据库实例的，属性值是引用 dataHost 标签上定义的 name 属性。
			database ：该属性用于定义该分片属性哪个具体数据库实例上的具体库，因为这里使用两个纬度来定义分片，就是：实例+具体的库。因为每个库上建立的表和表结构是一样的。所以这样做就可以轻松的对表进行水平拆分
		-->
		<dataNode name="dn1" dataHost="localhost" database="anbao" />
		
		<!-- 主机 -->
		<!-- 
			name ：唯一标识 dataHost 标签，供上层的标签使用。
			maxCon ：指定每个读写实例连接池的最大连接
			minCon ：指定每个读写实例连接池的最小连接，初始化连接池的大小。
			balance ：负载均衡类型
					1. balance="0", 不开启读写分离机制，所有读操作都发送到当前可用的 writeHost 上。
					2. balance="1"，全部的 readHost 与 stand by writeHost 参与 select 语句的负载均衡，简单的说，当双主双从模式(M1->S1，M2->S2，并且 M1 与 M2 互为主备)，正常情况下，M2,S1,S2 都参与 select 语句的负载均衡。
					3. balance="2"，所有读操作都随机的在 writeHost、readhost 上分发。
					4. balance="3"，所有读请求随机的分发到 wiriterHost 对应的 readhost 执行，writerHost 不负担读压力，
					 注意 balance=3 只在 1.4 及其以后版本有，1.3 没有。
			writeType ：负载均衡类型
					-1 表示不自动切换。
					1 默认值，自动切换。
					2 基于 MySQL 主从同步的状态决定是否切换。
			dbType ：指定后端连接的数据库类型
			dbDriver ：指定连接后端数据库使用的 Driver，目前可选的值有 native 和 JDBC。
			switchType ： -1 表示不自动切换
					1 默认值，自动切换
					2 基于 MySQL 主从同步的状态决定是否切换，心跳语句为 show slave status
					3 基于 MySQL galary cluster 的切换机制（适合集群）（1.4.1），心跳语句为 show status like ‘wsrep%’
			slaveThreshold ：

		-->
		<dataHost name="localhost" maxCon="1000" minCon="10" balance="0" writeType="0" dbType="mysql" dbDriver="native" switchType="2" slaveThreshold="100">
			<!-- 这个标签内指明用于和后端数据库进行心跳检查的语句 -->
			<heartbeat>SHOW SLAVE STATUS</heartbeat>
			<!-- 写实例 can have multi write hosts -->
			<writeHost host="hostM1" url="172.30.3.21:3306" user="root" password="root">
				<!-- 读实例 can have multi read hosts -->
				<readHost host="hostS1" url="172.30.3.22:3306" user="root" password="root" />
			</writeHost>
			<writeHost host="hostM2" url="172.30.3.22:3306" user="root" password="root">
				<!-- can have multi read hosts -->
				<readHost host="hostS2" url="172.30.3.21:3306" user="root" password="root" />
			</writeHost>
		</dataHost>
	</mycat:schema>
### 2、`server.xml` 文件 ###
	vim /server.xml
##### 注意修改 `firewall` `user` 标签内的内容 #####
	<?xml version="1.0" encoding="UTF-8"?>
	<!-- - - Licensed under the Apache License, Version 2.0 (the "License"); 
		- you may not use this file except in compliance with the License. - You 
		may obtain a copy of the License at - - http://www.apache.org/licenses/LICENSE-2.0 
		- - Unless required by applicable law or agreed to in writing, software - 
		distributed under the License is distributed on an "AS IS" BASIS, - WITHOUT 
		WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. - See the 
		License for the specific language governing permissions and - limitations 
		under the License. -->
	<!DOCTYPE mycat:server SYSTEM "server.dtd">
	<mycat:server xmlns:mycat="http://io.mycat/">
		<system>
			<!-- 字符集 -->
			<property name="charset">utf8</property>
			<!-- 处理线程数量，默认是CPU数量 -->
			<!-- <property name="processors">1</property> -->
			<!-- 0为需要密码登陆、1为不需要密码登陆 ,默认为0，设置为1则需要指定默认账户-->
			<!-- 每次读取流的数量，默认4096 -->
			<!-- <property name="processorBufferChunk">40960</property> -->
			<!-- 创建共享Buffer需要占用的空间大小，processorBufferChunk*processors*100 -->
			<!-- <property name="processorBufferPool">409600</property> -->
			<!-- 默认为type 0: DirectByteBufferPool | type 1 ByteBufferArena | type 2 NettyBufferPool -->
			<property name="processorBufferPoolType">0</property>
			<!-- 二级共享buffer是processorBufferPool的百分比，这里设置的是百分比。 -->
			<property name="processorBufferLocalPercent">100</property>
			<!-- 全局ID生成方式。(0:为本地文件方式，1:为数据库方式；2:为时间戳序列方式；3:为ZK生成ID；4:为ZK递增ID生成。 -->
			<property name="sequnceHandlerType">2</property>
			<!-- 1为开启mysql压缩协议 -->
			<!-- <property name="useCompression">1</property> -->
			<!-- 指定 Mysql 协议中的报文头长度。默认 4。 -->
			<property name="packetHeaderSize">4</property>
			<!-- 指定 Mysql 协议可以携带的数据最大长度。默认 16M。 -->
			<!-- <property name="maxPacketSize">16M</property> -->
			<!-- 指定连接的空闲超时时间。某连接在发起空闲检查下，发现距离上次使用超过了空闲时间，那么这个连接会被回收，就是被直接的关闭掉。默认 30 分钟，单位毫秒 -->
			<property name="idleTimeout">1800000</property>
			<!-- 前端连接的初始化事务隔离级别，只在初始化的时候使用，后续会根据客户端传递过来的属性对后端数据库连接进行同步。默认为 REPEATED_READ，设置值为数字默认 3。 READ_UNCOMMITTED = 1; READ_COMMITTED = 2; REPEATED_READ = 3; SERIALIZABLE = 4; -->
			<property name="txIsolation">3</property>
			<!-- SQL 执行超时的时间，Mycat 会检查连接上最后一次执行 SQL 的时间，若超过这个时间则会直接关闭这连接。默认时间为 300 秒，单位秒 -->
			<property name="sqlExecuteTimeout">300</property>
			<!-- 清理 NIOProcessor 上前后端空闲、超时和关闭连接的间隔时间。默认是 1 秒，单位毫秒。-->
			<property name="processorCheckPeriod">1000</property>
			<!-- 对后端连接进行空闲、超时检查的时间间隔，默认是 300 秒，单位毫秒。-->
			<property name="dataNodeIdleCheckPeriod">300000</property>
			<!-- 对后端所有读、写库发起心跳的间隔时间，默认是 10 秒，单位毫秒。-->
			<property name="dataNodeHeartbeatPeriod">10000</property>
			<!-- mycat 服务监听的 IP 地址，默认值为 0.0.0.0。-->
			<!-- <property name="bindIp">0.0.0.0</property> -->
			<!-- 定义 mycat 的使用端口，默认值为 8066。-->
			<property name="serverPort">8066</property>
			<!-- 定义 mycat 的管理端口，默认值为 9066。-->
			<property name="managerPort">9066</property>
			<!-- 设置模拟的MySQL版本号 -->
			<!-- <property name="fakeMySQLVersion">5.7.26-log</property> -->
			<!-- 是否开启实时统计 1为开启、0为关闭 -->
			<property name="useSqlStat">0</property>
			<!-- 是否开启全局表一致性检测 1为开启、0为关闭 -->
			<property name="useGlobleTableCheck">0</property>
			<!--分布式事务开关，0为不过滤分布式事务，1为过滤分布式事务（如果分布式事务内只涉及全局表，则不过滤），2为不过滤分布式事务,但是记录分布式事务日志-->
			<property name="handleDistributedTransactions">0</property>
			<!-- 默认是65535。 64K 用于sql解析时最大文本长度 -->
			<property name="maxStringLiteralLength">65535</property>
			
			<property name="nonePasswordLogin">0</property>
			<property name="useHandshakeV10">1</property>
			
			<!--<property name="sequnceHandlerPattern">(?:(\s*next\s+value\s+for\s*MYCATSEQ_(\w+))(,|\)|\s)*)+</property>-->
			<!--必须带有MYCATSEQ_或者 mycatseq_进入序列匹配流程 注意MYCATSEQ_有空格的情况-->
			<property name="sequnceHandlerPattern">(?:(\s*next\s+value\s+for\s*MYCATSEQ_(\w+))(,|\)|\s)*)+</property>
			<!-- 子查询中存在关联查询的情况下,检查关联字段中是否有分片字段 .默认 false -->
			<property name="subqueryRelationshipCheck">false</property>
			
			<!-- <property name="processorExecutor">32</property> -->
			<!-- <property name="backSocketNoDelay">1</property> -->
			<!-- <property name="frontSocketNoDelay">1</property> -->
			<!-- <property name="frontWriteQueueSize">4096</property> -->
			
			
			<!-- off heap for merge/order/group/limit      1开启   0关闭 -->
			<property name="useOffHeapForMerge">0</property>

			<!-- 单位为m -->
			<property name="memoryPageSize">64k</property>

			<!-- 单位为k -->
			<property name="spillsFileBufferSize">1k</property>

			<property name="useStreamOutput">0</property>

			<!-- 单位为m -->
			<property name="systemReserveMemorySize">384m</property>


			<!--是否采用zookeeper协调切换  -->
			<property name="useZKSwitch">false</property>

			<!-- XA Recovery Log日志路径 -->
			<!--<property name="XARecoveryLogBaseDir">./</property>-->

			<!-- XA Recovery Log日志名称 -->
			<!--<property name="XARecoveryLogBaseName">tmlog</property>-->
			<!--如果为 true的话 严格遵守隔离级别,不会在仅仅只有select语句的时候在事务中切换连接-->
			<property name="strictTxIsolation">false</property>
			
			<property name="useZKSwitch">true</property>
			
		</system>
		
		<!-- 全局SQL防火墙设置, 就是在网络层对请求的地址进行限制，主要是从安全角度来保证Mycat不被匿名IP进行访问-->
		<!--白名单可以使用通配符%或着*-->
		<!--例如<host host="127.0.0.*" user="root"/>-->
		<!--例如<host host="127.0.*" user="root"/>-->
		<!--例如<host host="127.*" user="root"/>-->
		<!--例如<host host="1*7.*" user="root"/>-->
		<!--这些配置情况下对于127.0.0.1都能以root账户登录-->
		<firewall>
		   <whitehost>
			  <host host="172.*" user="root"/>
			  <host host="127.*" user="root"/>
			  <host host="172.30.3.21" user="17230321"/>
			  <host host="172.30.3.22" user="17230322"/>
		   </whitehost>
		   <blacklist check="false">
		   </blacklist>
		</firewall>

		<user name="root" defaultAccount="true">
			<property name="password">root</property>
			<property name="schemas">propertyw,anbao</property>
			<!-- <property name="schemas">SYS</property> -->
			<!-- 表级 DML 权限设置 -->
			<!-- dml权限顺序 insert,update,select,delete -->
			<!-- 
			<privileges check="false">
				<schema name="DATABASE_TEST" dml="1111" >
					<table name="SYS_SAMPLES" dml="1111"></table>
				</schema>
			</privileges>
			 -->
		</user>
		
		<user name="17230321">
			<property name="password">17230321</property>
			<property name="schemas">propertyw,anbao</property>
			<property name="readOnly">true</property>
		</user>
		
		<user name="17230322">
			<property name="password">17230322</property>
			<property name="schemas">propertyw,anbao</property>
			<property name="readOnly">true</property>
		</user>
		
	</mycat:server>
### 3、`wrapper.conf` 文件 ###
	vim /wrapper.conf
##### 注意修改 `wrapper.java.command` 配置项
	#********************************************************************
	# Wrapper Properties
	#********************************************************************
	# Java Application
	wrapper.java.command=C:/Program Files/Java/jdk1.8.0_73/bin/java
	wrapper.working.dir=..

	# Java Main class.  This class must implement the WrapperListener interface
	#  or guarantee that the WrapperManager class is initialized.  Helper
	#  classes are provided to do this for you.  See the Integration section
	#  of the documentation for details.
	wrapper.java.mainclass=org.tanukisoftware.wrapper.WrapperSimpleApp
	set.default.REPO_DIR=lib
	set.APP_BASE=.

	# Java Classpath (include wrapper.jar)  Add class path elements as
	#  needed starting from 1
	wrapper.java.classpath.1=lib/wrapper.jar
	wrapper.java.classpath.2=conf
	wrapper.java.classpath.3=%REPO_DIR%/*

	# Java Library Path (location of Wrapper.DLL or libwrapper.so)
	wrapper.java.library.path.1=lib

	# Java Additional Parameters
	#wrapper.java.additional.1=
	wrapper.java.additional.1=-DMYCAT_HOME=.
	wrapper.java.additional.2=-server
	#wrapper.java.additional.3=-XX:MaxPermSize=64M
	wrapper.java.additional.4=-XX:+AggressiveOpts
	wrapper.java.additional.5=-XX:MaxDirectMemorySize=2G
	wrapper.java.additional.6=-Dcom.sun.management.jmxremote
	wrapper.java.additional.7=-Dcom.sun.management.jmxremote.port=1984
	wrapper.java.additional.8=-Dcom.sun.management.jmxremote.authenticate=false
	wrapper.java.additional.9=-Dcom.sun.management.jmxremote.ssl=false
	wrapper.java.additional.10=-Xmx2G
	wrapper.java.additional.11=-Xms512M

	# Initial Java Heap Size (in MB)
	#wrapper.java.initmemory=3
	wrapper.java.initmemory=30

	# Maximum Java Heap Size (in MB)
	#wrapper.java.maxmemory=64
	wrapper.java.maxmemory=640

	# Application parameters.  Add parameters as needed starting from 1
	wrapper.app.parameter.1=io.mycat.MycatStartup
	wrapper.app.parameter.2=start

	#********************************************************************
	# Wrapper Logging Properties
	#********************************************************************
	# Format of output for the console.  (See docs for formats)
	wrapper.console.format=PM

	# Log Level for console output.  (See docs for log levels)
	wrapper.console.loglevel=INFO

	# Log file to use for wrapper output logging.
	wrapper.logfile=logs/wrapper.log

	# Format of output for the log file.  (See docs for formats)
	wrapper.logfile.format=LPTM

	# Log Level for log file output.  (See docs for log levels)
	wrapper.logfile.loglevel=INFO

	# Maximum size that the log file will be allowed to grow to before
	#  the log is rolled. Size is specified in bytes.  The default value
	#  of 0, disables log rolling.  May abbreviate with the 'k' (kb) or
	#  'm' (mb) suffix.  For example: 10m = 10 megabytes.
	wrapper.logfile.maxsize=512m

	# Maximum number of rolled log files which will be allowed before old
	#  files are deleted.  The default value of 0 implies no limit.
	wrapper.logfile.maxfiles=30

	# Log Level for sys/event log output.  (See docs for log levels)
	wrapper.syslog.loglevel=NONE

	#********************************************************************
	# Wrapper Windows Properties
	#********************************************************************
	# Title to use when running as a console
	wrapper.console.title=Mycat-server

	#********************************************************************
	# Wrapper Windows NT/2000/XP Service Properties
	#********************************************************************
	# WARNING - Do not modify any of these properties when an application
	#  using this configuration file has been installed as a service.
	#  Please uninstall the service before modifying this section.  The
	#  service can then be reinstalled.

	# Name of the service
	wrapper.ntservice.name=mycat

	# Display name of the service
	wrapper.ntservice.displayname=Mycat-server

	# Description of the service
	wrapper.ntservice.description=The project of Mycat-server

	# Service dependencies.  Add dependencies as needed starting from 1
	wrapper.ntservice.dependency.1=

	# Mode in which the service is installed.  AUTO_START or DEMAND_START
	wrapper.ntservice.starttype=AUTO_START

	# Allow the service to interact with the desktop.
	wrapper.ntservice.interactive=false

	wrapper.startup.timeout=3600

	wrapper.ping.timeout=3600
	configuration.directory.in.classpath.first=conf
### 4、`log4j2.xml` 文件 ###
	vim /log4j2.xml
##### 调整日志输出级别 #####
	<?xml version="1.0" encoding="UTF-8"?>
	<!-- DEBUG -->
	<Configuration status="WARN">
	<!-- <Configuration status="DEBUG"> -->
		<Appenders>
			<Console name="Console" target="SYSTEM_OUT">
				<PatternLayout pattern="%d [%-5p][%t] %m %throwable{full} (%C:%F:%L) %n"/>
			</Console>

			<RollingFile name="RollingFile" fileName="../logs/mycat.log"
						 filePattern="logs/$${date:yyyy-MM}/mycat-%d{MM-dd}-%i.log.gz">
				<PatternLayout>
					<Pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %5p [%t] (%l) - %m%n</Pattern>
				</PatternLayout>
				<Policies>
					<OnStartupTriggeringPolicy/>
					<SizeBasedTriggeringPolicy size="250 MB"/>
					<TimeBasedTriggeringPolicy/>
				</Policies>
			</RollingFile>
		</Appenders>
		<Loggers>
			<!--<AsyncLogger name="io.mycat" level="info" includeLocation="true" additivity="false">-->
				<!--<AppenderRef ref="Console"/>-->
				<!--<AppenderRef ref="RollingFile"/>-->
			<!--</AsyncLogger>-->
			<asyncRoot level="debug" includeLocation="true">

				<AppenderRef ref="Console" />
				<AppenderRef ref="RollingFile"/>

			</asyncRoot>
		</Loggers>
	</Configuration>
### 5、保存退出 ###
#### 按一下 `Esc` 键
	:wq
## 五、启动Mycat服务 ##
### 1、切换到Mycat的bin目录下 ###
	cd /usr/local/mycat/bin/
### 2、查看启动状态 ###
	./mycat status
### 3、启动服务 ###
	./mycat start
### 4、重启服务 ###
	./mycat restart
### 5、停止服务 ###
	./mycat stop

## 六、防火墙端口开放 ##
### 1、查看防火墙状态 ###
	firewall-cmd --state
### 2、关闭防火墙 ###
	systemctl stop firewalld.service
### 3、禁止防火墙开机自启 ###
	systemctl disable firewalld.service
### 4、开启防火墙 ###
	systemctl start firewalld.service
### 5、Add 添加开放端口 ###
	firewall-cmd --permanent --zone=public --add-port=8066/tcp
### 6、Reload 重新加载 ###
	firewall-cmd --reload
### 7、检查是否生效 ####
	firewall-cmd --zone=public --query-port=8066/tcp