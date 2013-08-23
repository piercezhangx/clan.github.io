---
layout: default
title: 使用 rsync 进行文件备份
---

数据备份的重要性是不言而喻的，但具体涉及到备份方案的选择和执行则受到磁盘空间
和备份软件等因素的制约，尤其是对个人用户或预算不是那么充足的企业。

企业级的备份软件抛开价格的因素，还会有配置复杂，私有的备份格式等问题。备份
的保存也需要相应的存储空间保障。本文介绍一种利用
[rsync](http://www.samba.org/rsync/) 来实现数据备份的方案。

在具体的配置和命令之前，先明确下该方案期望达到的目标：

* 备份准确性，例如文件的元数据信息（创建时间，属主，权限等）
* 备份速度，最好是增量备份
* 备份空间的需求尽可能紧凑
* 历史备份可恢复

需要备份的客体以及相应的备份策略描述如下：

* 操作系统，包括系统软件以及系统配置。对操作系统，从运维的角度出发，我的建议
是对同一类型的系统（从发行版本和业务的角度区分）单独维护一个模板，所有的新
系统的安装都从模板中选择最合适的作为蓝本来进行进一步的定制。如果做到这一点，
则在对这些模板系统做好备份之后，线上系统只需要备份相关的用户数据和系统特有
的配置即可。
* 用户软件及用户数据。用户软件定义为模板没有或不能提供的软件，一般需要自己单独
编译或维护。从备份方便的角度，建议将用户软件和数据都放在 /home 目录下，这样会
方便后续数据备份时的脚本编写。
* 各种数据库数据。数据库由于文件格式一般为二进制格式，直接文件拷贝时文件的内容
不一定处在一致的状态，从而导致不完整或异常的备份。此外，像 mongodb 由于 journal
的存在以及其他特性，其数据库文件往往比实际的文件大很多，直接备份文件也不太
可取。因此，对数据库数据的备份，一般利用数据库的 dump 工具先将数据文件转储成
独立的文件后再来进行文件的备份。需要说明的是，这种方式仅仅针对离线和非实时的
备份而言，实时的备份一般采用除主从热备外，此外为防止数据被误删除后还能恢复，
往往还需要做更多的工作，如备份 MySQL 的 binlog，MongoDB 做延迟复制集等。

本文覆盖的备份对象仅包含修改相对不那么频繁的文件。接下来介绍具体的命令。

由于备份的对象仅限于文件，备份可以简单理解为文件的拷贝，rsync 正好是文件拷
贝的利器。结合其丰富的命令选项，rsync 可以完整的备份文件的几乎所有的信息（目前
已知不能处理的信息是通过 chattr 设置的属性）。常用的选项为

`rsync -havrAEHXi -n --numeric-ids --delete --stats --progress [SRC] [DST]`

其中最重要的选项是 a （归档）。取决于程序版本，并不是所有的选项都能正常使用，
如果碰到问题，则需要结合对应版本的 man 文档去掉相关的参数。**注意：在执行具体
的操作之前，建议加上 -n 参数确认操作和预期符合，以避免数据丢失。尤其是用到了
--delete 选项时。**

SRC 和 DST 的写法请参考 man 文档，需要强调的是，如果同步的对象是目录，则强烈
建议在目录名后加上 ‘/’，以避免出现非预期的结果。

在通过网络执行 rsync 传输文件时有两种方式，一种是通过远程 shell （ssh，rsh
等），另一种是通过 rsync 守护进程方式。由于远程 shell 一般需要配置认证，出
于安全的考虑以及其他的一些因素，本文不推荐这种方式，下面重点描述后一种方式
的相关配置。rsyncd 的启动请参考相应发行版本的文档，下面是一个配置文件示例：

<pre>
# /etc/rsyncd.conf

# See rsync(1) and rsyncd.conf(5) man pages for help

# This line is required by the /etc/init.d/rsyncd script
pid file = /var/run/rsyncd.pid
use chroot = yes
read only = yes

[bak-sys]
    uid = root
    gid = root
    path = /mnt/os/
    comment = OS Backup
    numeric ids = yes
    read only = yes
    list = no
    filter = : .os-filter
    hosts allow = ip.address.of.backup.server

[bak-data]
    uid = root
    gid = root
    path = /mnt/os/
    comment = DATA Backup
    numeric ids = yes
    read only = yes
    list = no
    filter = : .data-filter
    auth users = bak
    exclude = /etc/rsync-secret
    secrets file = /etc/rsync-secret
    hosts allow = ip.address.of.backup.server
</pre>

在这个配置文件中，定义了两个 rsync 模块。/mnt/os/ 是对应 / 文件系统的新的挂
载点，通过 `mount --bind / /mnt/os` 实现，其目录是让 rsync 拷贝文件时仅仅对
真正在存储介质上存在的文件进行拷贝，忽略哪些在系统运行时才会存在的文件，如
/proc， /sys 等目录下的文件。如果系统有多个分区，如 /home 等，相应的也需要将
这些目录同步挂载在 /mnt/os 的对应目录下。也可以编辑 /etc/fstab 实现系统启动
时自动挂载，如下面 /etc/fstab 文件的部分内容：

<pre>
LABEL=/         /            ext3    noatime,errors=remount-ro   0       1
LABEL=/home     /home        ext4    noatime,errors=remount-ro   0       1
/               /mnt/os      ext3    bind                        0       0
/home           /mnt/os/home ext4    bind                        0       0
</pre>

filter 参数需要解析下，该参数定义了文件同时的过滤规则。详细的说明参考
`man rsync` 的 FILTER RULES 部分。在这里，“filter = : .os-filter” 表示
rsync 同步每个子目录时会参考该目录下的 .os-filter 文件来确定文件过滤规则。
例如 /.os-filter 的文件内容为：

<pre>
- /.git
- /etc/blkid.tab*
- /etc/mtab
- /etc/service/*/supervise/
+ /root/.bash_profile
+ /root/.bashrc
+ /root/.local/
+ /root/.profile
+ /root/.screenrc
+ /root/.ssh/
+ /root/.ssh/authorized_keys
- /root/.ssh/*
+ /root/.subversion/
+ /root/.tmux.conf
+ /root/.toprc
+ /root/.vimrc
- /root/*
- /tmp/*
</pre>

则系统在同步时会忽略 /.git 等文件或文件夹。如果仔细看该文件和 /root/
目录相关的规则，则会明白规则的顺序很重要，需要仔细测试体会。

在 bak-data 模块中还定义了 ‘auth users = bak’ 和
‘secrets file = /etc/rsync-secret’，这两个选项可以对 rsync 模块增加一个简单的
认证。其中 rsync-secret 文件的内容为 ‘bak:pass’，相应的同步命令如下：

`RSYNC_PASSWORD=pass rsync -hvarAEHXi --delete --stats --progress --numeric-ids rsync://bak@dst-ip/bak-data dstdir/`

‘hosts allow’ 配置控制允许访问该模块的 IP 地址。

通过仔细的调整 filter 规则即可实现对目标系统的数据文件的快速同步备份。

至此，在本文开始提到的备份目标的前两个都可以通过 rsync 来实现。如果抛开第
三个目标（备份空间需求尽量紧凑），则最后一个目标可以通过每个备份周期一个文
件夹的方式来实现。在文件的改变很频繁时这也是合理的方案。但考虑到大部分文件
（尤其是系统相关）的变动不会那么频繁，备份空间的这个需求也是有一定道理的，
但具体的实现则需要借助另外的手段，如文件系统快照等功能。留待之后再来说明。

{{ page.date | date_to_string }}
