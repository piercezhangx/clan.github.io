---
layout: default
title: Sort /etc/passwd, /etc/group based on uid/gid
---
<pre>
sort -n -t ':' -k3 /etc/passwd
</pre>

{{ page.date | date_to_string }}
