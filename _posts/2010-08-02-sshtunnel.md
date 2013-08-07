---
layout: default
title: 配置 SSH 做代理或端口映射
---

1. 作 socks 代理

   <pre>ssh -D [bind_address:]port username@remote.host.com</pre>

2. 作端口映射

    本机端口port被映射到远端host的hostport端口
    <pre>ssh -L [bind_address:]port:host:hostport</pre>

    远端端口port被映射到本地host的hostport端口
    <pre>ssh -R [bind_address:]port:host:hostport</pre>

{{ page.date | date_to_string }}
