---
layout: default
title: "在 Linux 上 mount window 共享出错： mount error(12): Cannot allocate memory"
---

Windows 系统为 Win7。解决办法如下，修改注册表：

修改 “HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management\LargeSystemCache” 为 “1″.

修改 “HKLM\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters\Size” 为 “3″.

然后重启服务 “server”。

{{ page.date | date_to_string }}
