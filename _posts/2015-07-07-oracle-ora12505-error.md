---
layout: post
title:  "Tomcat服务器启动报ora-12505错误"
keywords: "oracle"
description: "Tomcat服务器上部署war包时报ora-12505拒绝侦听错误，但是用PL/SQL工具能连接数据库"
category: oracle
tags: oracle
---

####解决程序运行报TNS:listener does not currently know of SID given in connect descriptor错误，但是用工具能链接数据库的问题

上午在Tomcat服务器上部署生产war包时，报ora-12505错误，监听器不识别连接中给的SID,错误信息如下:

	2015-07-07 12:20:10,261 ERROR [com.alibaba.druid.pool.DruidDataSource]  - init datasource error
	java.sql.SQLException: Listener refused the connection with the following error:
	ORA-12505, TNS:listener does not currently know of SID given in connect descriptor
	The Connection descriptor used by the client was:
	10.100.6.104:1521:PRPS

		at oracle.jdbc.driver.DatabaseError.throwSqlException(DatabaseError.java:111)
		at oracle.jdbc.driver.DatabaseError.throwSqlException(DatabaseError.java:260)
		at oracle.jdbc.driver.T4CConnection.logon(T4CConnection.java:386)
		at oracle.jdbc.driver.PhysicalConnection.<init>(PhysicalConnection.java:419)
		at oracle.jdbc.driver.T4CConnection.<init>(T4CConnection.java:164)
		at oracle.jdbc.driver.T4CDriverExtension.getConnection(T4CDriverExtension.java:34)
		at oracle.jdbc.driver.OracleDriver.connect(OracleDriver.java:752)
		at com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:146)
		at com.alibaba.druid.filter.stat.StatFilter.connection_connect(StatFilter.java:211)
		at com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:140)



在`jdbc.properties`文件中 配置的`url`如下:

	jdbc.url=jdbc:oracle:thin:@10.100.6.104:1521:PRPS

将url改为

	jdbc.url=jdbc:oracle:thin:@(description=(address=(protocol=tcp)(port=1521)(host=10.100.6.104))(connect_data=(service_name=PRPS)))

后问题解决。

其中链接地址结构为：

结构为:

	description
		address
		       protocol
		       host
		       port
		connect_data
		       service_name
	}
