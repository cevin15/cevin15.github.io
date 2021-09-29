# Oracle JDBC连接的几种配置写法

_2019-10-24_ _20:08_

今天上线项目的时候，发现Oracle的数据库连接一直有问题，报了这么一个错：
```shell
ORA-12505, TNS:listener does not currently know of SID given in connect descriptor
```

笔者的数据库写法是：`jdbc:oracle:thin:@172.253.34.154:1521:eas`

这很奇怪，用客户端工具连接是没错的呀。纠结了好一会，突然灵光一闪：客户端工具配置的时候，用的是服务，而不是sid。难道服务跟sid的配置是不一样的？

赶紧查下资料，果然！

如果用的是服务名，配置是这样的：`jdbc:oracle:thin:@//172.253.34.154:1521/eas`  
而如果是sid，则配置是这样的：`jdbc:oracle:thin:@172.253.34.154:1521:eas`

更新，问题修复！

