

> Most of the errors could be found in the **Apache logs**, in this setup you can find it in `/var/log/apache2/error.log` or `/var/log/apache2/error.log.1`.

> **PHP logs**: `/var/log/php8.4-fpm.log`, `/var/log/php8.4-fpm.log.1`

---
### **Error**: Allowed memory size  of `<NUMBER>` bytes exhausted:

> Caused when admin was exporting large `csv` file.

**Here `memory_limit` =  `1073741824` bytes = `1024` MB, but task need more `20480` bytes:**
```bash
[Mon Sep 14 22:59:57.574147 2025] [proxy_fcgi:error] [pid 501502:tid 134864154113728] [client 10.0.0.15:38384] AH01071: Got error 'PHP message: PHP Fatal error:  Allowed memory size of 1073741824 bytes exhausted (tried to allocate 20480 bytes) in /var/www/html/moodle/public/lib/dml/mysqli_native_moodle_database.php on line 1369', referer: https://my-moodle.com/mod/quiz/report.php?id=62976&mode=overview
```

#### **Solution:**

> Give the php scripts higher memory limit.

```bash
vim /etc/php/8.4/fpm/php.ini
```

**Set memory_limit to a higher value that is enough for the php script:**
```bash
# Old value:
# memory_limit = 1024M 

# New value:
memory_limit = 2048M
```

> If you get that error while running a php script from CLI the edit same parameter in `/etc/php/8.4/cli/php.ini` too.

---
---
### **Error**: execution timed out (`NUMBER` sec), terminating:

> Caused when admin was uploading a large `csv` file with thousands of records that needed `2h` to process but the time limit was` 10m`.

> The way i calculate it is `1000` rows per minute.

```bash
[Sat Sep 19 22:53:12.007687 2025] [proxy_fcgi:error] [pid 1465024:tid 134867828315840] [client 10.0.0.15:35996] AH01067: Failed to read FastCGI header, referer: https://my-moodle.com/admin/tool/uploaduser/index.php

[Sat Sep 19 22:53:12.007778 2025] [proxy_fcgi:error] [pid 1465024:tid 134867828315840] (104)Connection reset by peer: [client 10.0.0.15:35996] AH01075: Error dispatching request to : , referer: https://my-moodle.com/admin/tool/uploaduser/index.php

[19-Sep-2025 22:53:11] WARNING: [pool www] child 1496128, script '//var/www/html/moodle/public/admin/tool/uploaduser/index.php' (request: "POST /admin/tool/uploaduser/index.php") execution timed out (777.145510 sec), terminating
```

#### **Solution:**

> Expand the timeout -> request_terminate_timeout = max_execution_time = 7200s -> 2h

```bash
vim /etc/php/8.4/fpm/php.ini
```

```php
# Old value:
# max_execution_time = 600 # 10m

# New value:
max_execution_time = 7200 # 2h
```

```bash
vim /etc/php/8.4/fpm/pool.d/www.conf
```

```php
# Old value:
# request_terminate_timeout = 600 # 10m

# New value:
request_terminate_timeout = 7200 # 2h
```

> Don't Forget to set it back to `600s` as its enough for most tasks in Moodle and without that limit bad plugins with infinite loops or being stuck might exhausted your memory and CPU.

> If you get that error while running a php script from CLI the edit same parameter in `/etc/php/8.4/cli/php.ini` too.

---
---
### **Error**: Database timeout:

> Edit `wait_timeout` to a higher value that is enough.

```bash
vim /etc/mysql/mariadb.conf.d/50-server.cnf
```

```bash
# Old value:
# wait_timeout = 7200 # 2hrs

# New value:
wait_timeout = 28800 # 8hrs is the default value
```