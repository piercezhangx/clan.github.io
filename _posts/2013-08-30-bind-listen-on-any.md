---
layout: default
title: BIND 监听任意地址的问题解决 (interface-interval)
---

BIND 及时在配置为 'listen-on { any; };' 时也不会 bind 到 IPv4 的 0.0.0.0 地址
上监听请求，但对 IPv6 （'listen-on-v6 { any; }; '）则会 bind 到 '::' 上。这么做
的原因未知。但至少在 IPv4 的情况下这个特性会导致一些问题，尤其是在系统 IP 地址
有变化时（如 IP 地址的增加/删除，VPN 接口的出现等），其现象是向指定的（新增）
地址发起请求时得不到回应。BIND 的配置指令 interface-interval 即是用来解决该
问题的。其文档如下：

<pre>
 interface-interval minutes;

 interface-interval defines the time in MINUTES when scan all interfaces on
 the server and will begin to listen on new interfaces (assuming they are not
 prevented by a listen-on option) and stop listening on interfaces which no
 longer exist. The default is 60 (1 hour), if specified as 0 NO interface scan
 will be performed. The maximum value is 40320 (28 days). This option may only
 be specified in a 'global' options statement.
</pre>

该指令控制扫描网络接口 IP 地址变化的时间间隔，其时间单位为分钟。因此对 IP 地
址会有变化的系统最好配置该选项为合适的值，例如通过 VRRP 协议决定 VIP 的系统，
或有 VPN 客户端的系统，...

{{ page.date | date_to_string }}
