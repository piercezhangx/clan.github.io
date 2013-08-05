---
layout: default
title: howto remove incomplete arp entry
---

arp -d 无法删除 incomplete 的 表项，只能通过如下的命令（假设删除设备 eth1 上的表项）
<pre>
 # ip link set dev eth1 arp off &amp;&amp; ip link set dev eth1 arp on
</pre>

{{ page.date | date_to_string }}
