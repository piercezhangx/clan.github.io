---
layout: default
title: fix mysqldump error after upgrade
---

After mysql upgrade, mysqldump may fail w/ following error:

    Error: Couldn't read status information for table general_log ()
    mysqldump: Couldn't execute 'show create table `general_log`': Table 'mysql.general_log' doesn't exist (1146)

Fix it by run mysql_upgrade!

{{ page.date | date_to_string }}
