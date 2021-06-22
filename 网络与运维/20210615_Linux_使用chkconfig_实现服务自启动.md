# Linux 使用chkconfig 实现服务自启动

_2021-06-15_ _15:44_

**查看服务**

```shell
chkconfig --list
```

**自定义服务**

在`/etc/init.d/`目录下，添加自定义的脚本文件。

脚本文件有个要注意的地方，需要在文件开头定义如下内容。如果没有，可能会报这个错误：`service ${service_name} does not support chkconfig`

```shell
# chkconfig: 2345 91 89
# description: ${service_name}
```

第一行表示 chkconfig的规则，2345 启动级别，91 启动优先级，89 关闭优先级（优先级 0-100，值越高优先级越低）

举个例子子，如下为笔者配置的elasticsearch 自启脚本。P.S. 这里使用es 用户来启动elasticsearch。

```shell
#! /bin/bash
# elasticsearch
# chkconfig: 2345 91 89
# description: elasticsearch
su - es -c "source /etc/profile && /usr/es/elasticsearch-6.5.4/bin/elasticsearch -d"
```

最后，给这个脚本添加执行权限：`chmod +x elasticsearch`

**添加服务**

```shell
chkconfig --add ${service_name}	
```

如果如上已经配置了2345的启动级别，则默认添加之后就设置了自启动。

**关闭服务自启动**

```shell
chkconfig ${service_name} off
```

默认level 是2345，可以使用`--level` 指定level

**启动服务自启动**

```shell
chkconfig ${service_name} on
```

默认level 是2345，可以使用`--level` 指定level



