---
layout: default
title: 使用 rrdcached 降低 IO 压力
---

在RRDtool 的数据文件较多时，磁盘IO会给系统带来一定的压力。rrdcached 即是解决
该问题的一个方法。相应的代码修改也非常简单，只需要设置 RRDCACHED_ADDRESS 环境
变量或者修改相应的更新方法增加一个参数 "-l daemon_address"即可。不过需要注意的
是，如果在每次数据更新（update方法）后接着有其他的操作（如graph等）则rrdcached
起不到应有的作用。原因是rrdcached仅对update方法才会有cache，其他方法在执行之前
都会先发送FLUSH命令，导致数据直接更新到文件，即清空cache。因此，rrdcache不适用
于在数据更新（update）之后马上执行graph等操作的场景。

{{ page.date | date_to_string }}
