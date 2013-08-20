---
layout: default
title: SSH OTP 认证
---

在[前一篇]({% post_url 2012-05-15-ssh-pubkey %}) blog
的最后提到了采用 OTP + 密码作为 SSH 的认证方案以保证安全性，这里就简单描述
一下实现方案。我们采用的软件是
[Google Authenticator](http://code.google.com/p/google-authenticator/)，
其客户端实现支持 Android，iOS，BlackBerry 系统。

在服务器上执行下面的操作：

<pre>
$ git clone https://code.google.com/p/google-authenticator/
$ cd google-authenticator/libpam/
$ make
</pre>

然后分别拷贝 <em>pam_google_authenticator.so</em> 和 <em>google-authenticator</em>
到 /lib/security/ 和 /usr/bin/ 目录。之后编辑 /etc/pam.d/sshd 文件，在该文件的
最前面插入一行：
<pre>
auth       required     pam_google_authenticator.so
</pre>

至此服务端的配置完成。需要注意的是在这个操作之后如果用户没有做公钥认证的配置
会导致用户无法登陆系统。解决办法有多种：基于安全起见可以强制用户在管理员的协
助下配置公钥认证方式登录系统；或者在上面的配置项后设置
pam_google_authenticator.so 的选项为 nullok，这样用户在没有做 OTP 相关的设置
时可以先用密码登录系统来进行相应的操作。

如果 SSH 服务配置了只允许公钥认证，则需要修改 /etc/ssh/sshd_config 文件的配置
项为：
<pre>
ChallengeResponseAuthentication yes
UsePAM yes
</pre>
并重启 SSH 服务以支持 OTP 和密码认证。

客户端需要每个用户登录到系统之后进行相关的配置。用户可以直接输入
google-authenticator 后按提示操作即可，或者直接用下面的命令：
<pre>
$ google-authenticator -l test@knownsec.com -t -d -r 3 -R 30 -w 3
</pre>

google-authenticator -h 能看到可用的参数。上面命令用到的参数解释如下：
- -l 设置移动设备上的程序对应的 OTP 码的标签，便于区分不同的应用。</li>
- -t 设置采用基于时间的验证。相应的也可以 -c 参数采用基于计数的验证。</li>
- -d 禁止重用基于时间的验证码</li>
- -r 限制登录频率</li>
- -R 设置登录频率限制的时间间隔</li>
- -w 设置时间窗的大小，主要在移动设备的时间不是准确同步的情况下比较有用</li>

如果系统安装了 qrencode 软件，google-authenticator 可以直接输出一个二维码供
移动设备扫描录入（见下图），否则用户需要根据其输出来手工进行相应的设置。

<a href="/image/google-authenticator.png"><img src="/image/google-authenticator.png" title="google-authenticator" width="300" height="252"/></a>

剩下的就是验证测试，祝顺利!

P.S. 由于系统配置以及软件版本等的不同，本文中提到的配置并不能覆盖所有的环境。
准确的配置需要参考官方文档并结合实际的系统进行调整。

P.P.S. 显然，本文中提到的 OTP 认证并不局限于 SSH，其他任何支持 PAM 认证方式的
软件也都可以使用。

{{ page.date | date_to_string }}
