# Windows 操作系统下 手动添加jar包进入本地仓库 #
## 内容如下： ##
### 1、黑窗口进入Maven的bin的文件夹下： ### 
```
cd : d\
```
### 2、运行如下代码： ### 
```
mvn install:install-file -Dfile=[jar包的位置] -DgroupId=[pom文件的groupId标签] -DartifactId={pom文件的artifactId标签} -Dversion=[pom文件的version标签] -Dpackaging=jar -DgeneratePom=true -DcreateChecksum=true
```
### 例如： ###
```
mvn install:install-file -Dfile=C:\Users\liuxin\Desktop\jbarcode-0.2.8.jar -DgroupId=org.jbarcode -DartifactId=jbarcode -Dversion=0.2.8 -Dpackaging=jar -DgeneratePom=true -DcreateChecksum=true
```
#### pom文件: ###
```
<dependency>
	<groupId>com.artofsolving</groupId>
	<artifactId>jodconverter</artifactId>
	<version>2.2.2</version>
</dependency>
```