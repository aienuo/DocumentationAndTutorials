# Kettke批量同步数据 

## 一、结构介绍：

1. ### 1无过滤条件 
   1. `GetTableName.ktr` 
   	   ```
	   从Table.txt获取表名
	   ```
   2. `Multi-tableFullDataAcquisition.kjb`
   	   ```
	   任务入口
	   ```
   3. `readme.md` 
	   ```
	   简介
	   ```
   4. `SynchronousData.ktr` 
   	   ```
	   数据库操作
	   ```
   5. `Table.txt` 
   	   ```
	   表名存放文件
	   ```
	
## 二、注意事项

1. ### 注意维护 `SynchronousData.ktr`  的DB连接
   1. `OracleInPut` 
   	   ```
	   接受数据的数据库
	   ```
   2. `OracleOutPut`
   	   ```
	   数据来源服务器
	   ```

2. ### 注意维护 `SynchronousData.ktr`  的节点
   1. `执行SQL脚本` 
   	   ```
	   维护SQL语句，这里是清除接受数据数据库的表内容
	   ```
   2. `表输入`
   	   ```
	   维护SQL语句，注意维护检索条件；注意维护记录数量限制参数；这里是获取数据来源
	   ```
   3. `表输出` 
	   ```
	   数据写入，注意维护 目标模式、目标表、提交记录数量
	   ```
	
3. ### 注意维护 `GetTableName.ktr`
	```
	字段名称 要求要与 Table.txt 的第一行保持一致
	```

4. ### 注意维护 `Table.txt` 
	```
	第一行是字段名称，从第二行开始是具体的表名称
	```