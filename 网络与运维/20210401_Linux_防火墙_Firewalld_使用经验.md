# Linux 防火墙 Firewalld 使用过的经验

_2021-04-01_ _15:46_ 

目前使用经验适用于CentOS 7+，Redhat 7+

有些系统没有默认安装Firewalld，可以使用yum进行安装：`yum install -y firewalld`

**Firewalld 状态查询**
```
[root@Genl-gm-4885 ~]# firewall-cmd --state
not running
```

**Firewalld 启用，关闭等等命令**
```
systemctl start firewalld   //启动
systemctl stop firewalld    //停用
systemctl enable firewalld  //设置Firewalld自启动
systemctl disable firewalld //设置Firewalld不自启动
```

PS. 查看系统自启动服务
```
systemctl list-unit-files|grep enabled
```

Firewalld 使用了zone的一个概念，通过设置zone 可以快速设置防火墙的规则。  
使用`firewall-cmd --get-zones` 获取zone列表
```
block dmz drop external home internal public trusted work
```

默认的zone 是public。

Firewalld 的连接规则有三类：  
1. drop：抛弃连接请求
2. reject：拒绝连接请求
3. accept：接受连接请求

public 的默认连接规则是default，应该是reject（或者reject）待确认

**使用rich rule 添加防火墙规则**
```
firewall-cmd --zone=public --add-rich-rule 'rule family="ipv4" source address="172.255.92.199" accept' --permanent  //添加 permanent，表明永久有效，但需要reload
firewall-cmd --reload   //reload 规则
```

**查看当前规则**
```
firewall-cmd --list-all
```

举个例子，假设要禁用 172.253.0.0/16 网段，但允许172.253.32.21 访问。

理所当然的这么调整，发现不行；调换顺序也不行。
```
firewall-cmd --zone=public --add-rich-rule 'rule family="ipv4" source address="172.253.0.0/16" reject' --permanent
firewall-cmd --zone=public --add-rich-rule 'rule family="ipv4" source address="172.253.32.21" accept' --permanent
```

翻了N久的bing和百度，没找到任何说法。后来发现了NOT 语法！  
如下这番，就可以了！！

```
firewall-cmd --zone=public --add-rich-rule 'rule family="ipv4" source NOT address="172.253.0.0/16" accept' --permanent
firewall-cmd --zone=public --add-rich-rule 'rule family="ipv4" source address="172.253.32.21" accept' --permanent
```

PS.一些字段掩码的规则

172.0.0.0/8：代表 172.0.0.1 - 172.255.255.255  
172.23.0.0/16：代表 172.23.0.1 - 172.23.255.255  
172.23.15.0/24：代表 172.23.15.1 - 172.23.15.255  
