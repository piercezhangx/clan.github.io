---
layout: default
title: 配置 Chrome 使用 SOCKS5 代理
---

创建 socks5.pac 文件如下：

<pre>
function FindProxyForURL(url, host)
{
    return "SOCKS5 localhost:1080";
}
</pre>

设置代理服务器配置 URL 为上面的文件。

{{ page.date | date_to_string }}
