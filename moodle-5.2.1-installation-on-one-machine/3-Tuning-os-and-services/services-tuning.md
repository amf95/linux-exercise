
> Tuning a system is not a one time thing, its a journey of fine tuning every single parameter.

> The parameters here are just a way of starting.

> These configs were created for a system with 64GB Memory and 24 cores vCPU.

> **Restart** or **Reload** the service in using `systemctl`.

---
# Apache:

> Configs to handle 10,000 requests per second.

> **WARNING**: Watch out for memory buffers as it might over load your system memory and crash your application.

> Rule of thump: The content served is 70-80% static and 20-30% dynamic when we combine  `Apache` + `PHP-FPM` or any backend.

> This ratio changes depending on the application.

> If `static`:`dynamic` is `4:1`  ratio, therefor  `MaxRequestWorkers = pm.max_children * 4`.

> Most of the extra `apache workers` are for handling `keepalive` and serving static content like images, css, html, ... 

> For and app like `moodle` `MaxRequestWorkers = pm.max_children * 1.25` by practice as `moodle` is a `php` heavy app.

```bash
vim /etc/apache2/mods-enabled/mpm_event.conf
```

```bash
StartServers            20
ServerLimit             157
MinSpareThreads         100
MaxSpareThreads         300
# ThreadLimit             400
ThreadsPerChild         64
MaxRequestWorkers       10048
ListenBacklog           8192
MaxConnectionsPerChild  10000
```

> **IMPORTANT**: `MaxRequestWorkers` = `ServerLimit` * `ThreadsPerChild` -> otherwise `Apache` will load the default configs which is `MaxRequestWorkers` = `150`.

> These attributes are called `directives`.

> `ThreadsPerChild` <=> `worker`.

> **`ServerLimit`**: The maximum number of `Child Processes` that the `Apache Server` will create, if not set default value is `16`.

> **`StartServers`**: The number of `Child Processes` that the `Apache Server` will create on Start-Up.

> **`MinSpareThreads`**: The Minimum Number of Threads that must exist(Mostly at Start-Up).

> **`MaxSpareThreads`**: The Maximum Number of Threads that the Server will keep running.

> **`ThreadLimit`**: Must be >= `ThreadsPerChild` (usually with 3-4 times).

> **`ThreadsPerChild`**: The Number of Threads that each Process will create.

> **`MaxRequestWorkers`**: Maximum number of connections that the Server will accept simultaneously.

> **`MaxConnectionsPerChild`**: The Maximum Number of Connections that each `Child Processes` will accept. This determines the Time of Life of a `Child Process`(How many connections can a `Child Processes` handle before being killed an replaced with a new one to free memory), default is `0` which means unlimited, 1000–10000 is recommended by practice.

---

**Apache security:**
```bash
vim /etc/apache2/conf-enabled/security.conf
```

```bash
ServerTokens Prod
ServerSignature Off
```

> `ServerTokens Prod`: makes Apache's `Server` HTTP header show only "Apache" with no version/OS/module details, reducing information leaked to attackers.

> `ServerSignature Off`: removes the Apache version/server info footer that appears on auto-generated pages like 404/error pages and directory listings.

---

**Timeouts for your site:**
```bash
vim /etc/apache2/sites-enabled/my-site.conf
```

**In your `VirtualHost` configs:**
```bash
    # all relevant timeouts 
    Timeout 600
    ProxyTimeout 600

    # FastCGI/PHP-FPM: 
    FcgidIOTimeout 600
    FcgidConnectTimeout 600
    FcgidBusyTimeout 600
```

> When you have timeout errors in apache logs `Timeout`, `ProxyTimeout` and `FcgidIOTimeout` are usually one of the causes along with php's and database timeouts.

> `Timeout`: how long Apache waits for I/O (reading a request or sending a response) before giving up(at the client side).

> `ProxyTimeout`: how long Apache's general proxy module (mod_proxy, reverse-proxying to a backend/upstream) waits for that backend to respond before timing out.

>`FcgidIOTimeout`: how long Apache waits for I/O activity (data being sent/received) from a FastCGI (mod_fcgid) process before timing out.

> `FcgidConnectTimeout`: how long Apache waits to establish the initial connection to the FastCGI process.

> `FcgidBusyTimeout`: how long a FastCGI request is allowed to run while "busy" (actively processing) before being killed as hung.

---
---
# PHP:

> **WARNING**: Watch out for memory buffers as it might over load your system memory and crash your application.

> `max_execution_time`  `request_terminate_timeout` controls the timeout of tasks, `600s` -> `10m` is good for most cases, but for tasks like exporting large CSV files and reports or uploading large CSV file with a lot of rows you can extend the timeout to `7200s` -> `2h`.

```bash
vim /etc/php/8.4/fpm/pool.d/www.conf
```

```php
pm = dynamic
pm.max_children = 2000
pm.start_servers = 100
pm.min_spare_servers = 100
pm.max_spare_servers = 101
pm.max_requests = 500 # just uncomment -> 500

request_terminate_timeout = 1200 # 20 mins
listen.backlog = 65536
```

> `pm = dynamic`: dynamically increase and decrease the number of children(workers).

> `child` <=> `procces` <=> `worker`.

> `pm.max_children`: max number of children that could be spawn, golden ration is child per user, or child per 1.5 users for a moment in time.

> `pm.start_servers`: number of children to start with when the php-fpm service is starting.

> `pm.min_spare_servers`: minimum number of workers to keep on standby even if not needed.

> `pm.max_spare_servers`: maximum number of workers to keep on standby, should be higher that `pm.min_spare_servers`.

> `pm.max_requests`: max number of requests a child can handle before it's process is terminated and new one is started, used to protect from memory leaks.

> `request_terminate_timeout`: max time a request can take, it kills the process responding to the request and cuts the connection with the client just for this request(not the entire session), client might get `502 Bad Gateway` or `504 Gateway Timeout`.

> `listen.backlog`: max number of the OS-level TCP/socket backlog queue, In other words how many pending incoming connections can queue up waiting for an FPM worker to accept them before the OS starts refusing new connections.

---

```bash
vim /etc/php/8.4/fpm/php.ini
```

```php
max_execution_time = 600
memory_limit = 1024M
# memory_limit = 2048M # For larger moodle tasks.


max_input_vars = 15000
post_max_size = 2048M
file_uploads = On
upload_max_filesize = 2048M
```

> `max_execution_time`: how long a single PHP script can run before PHP itself kills it.

> `memory_limit`: how much memory a single PHP script can use before it's fatally killed with an "allowed memory size exhausted" error, exporting and importing large csv files might need up to 2048MB of memory.

> `max_input_vars`: number of allowed input variables (form fields, array/nested fields) per request before PHP silently drops the excess.

> `post_max_size`: caps the total size of POST data (form fields + uploaded files combined) a request can send, can also be controlled from the website `Site administration` -> `General` -> `Security` -> `Maximum uploaded file size`, default is the same as your configs.

> `file_uploads`: enables file uploads via HTTP POST at all, if off, no uploads work regardless of other settings.

> `upload_max_filesize`: max size of any single uploaded file, must be ≤ `post_max_size` to actually take effect.

---

```bash
vim /etc/php/8.4/cli/php.ini
```

```php
max_execution_time = 7200
memory_limit = 1024M

max_input_vars = 15000
post_max_size = 2048M
file_uploads = On
upload_max_filesize = 2048M
```

> Same as `fpm` configs but for commands run from the terminal or a script.

---

**PHP Opcache:
```bash
vim  /etc/php/8.4/mods-available/opcache.ini
```

```bash
opcache.enable = 1
opcache.memory_consumption=512
opcache.max_accelerated_files=32768
opcache.interned_strings_buffer=64
; Required for Moodle -> https://docs.moodle.org/502/en/OPcache
opcache.use_cwd = 1
opcache.validate_timestamps = 1
opcache.save_comments = 1
opcache.enable_file_override = 0
```

> `Opcache` caches .php scripts in memory(RAM) for faster execution next time, also shares memory pages which reduces the memory consumption radically.

> `opcache.memory_consumption`: size dedicated in memory for opcache, `128mb` for small apps, `256mb` for mid size like moodle.

> `opcache.max_accelerated_files`: number of .php scripts that can be loaded in memory, should exceed your total file count.

> `opcache.interned_strings_buffer`: size dedicated in memory for deduplicating/storing repeated strings across scripts, saving memory.

> `opcache.use_cwd = 1`: caches files keyed by their full path including current working directory, so identically-named files in different dirs don't collide.

> `opcache.validate_timestamps = 1`: checks each file's mtime before serving it from cache, so edited PHP files are picked up automatically (safer but slightly slower than disabling this).

> `opcache.save_comments = 1`: keeps docblock comments in cached code, needed because Moodle (and many frameworks) use `/** @annotations */` for reflection-based logic.

> `opcache.enable_file_override = 0`: disables an optimization that skips `file_exists()`/`is_file()` checks via OPcache, keeping those calls accurate (Moodle needs this correctness).

---
---
# MariaDB:

> Better to use [MySQLTuner](https://github.com/major/MySQLTuner-perl) during a real pressure, these parameters are basic configs.

> Most important parameters are: `innodb_buffer_pool_size`, `innodb_log_file_size`, `max_connections`, `wait_timeout`.

> **WARNING**: Watch out for memory buffers as it might over load your system memory and crash your application.

> `innodb_buffer_pool_size`: how much RAM InnoDB uses to cache table/index data in memory, the single biggest factor in DB performance (typically set to 60–80% of available RAM on a dedicated DB server).

> `innodb_log_file_size`: sets the size of InnoDB's on-disk redo log, a bigger log lets more writes accumulate before forcing a slow flush of changed data to disk (better write performance), at the cost of longer crash-recovery time.

> `max_connections`: the maximum number of simultaneous client connections MariaDB will accept before rejecting new ones with "too many connections", should be higher than `pm.max_children`.

> `wait_timeout`: how long (in seconds) MariaDB keeps an idle `non-interactive` connection (from a script/app) open before closing it.

> `interactive_timeout`: same as `wait_timeout` but applies to `interactive` connections (user connected via the `mysql` CLI client).

```bash
vim /etc/mysql/mariadb.conf.d/50-server.cnf
```

```bash

# CONNECTIONS
max_connections = 2500
thread_cache_size = 128
wait_timeout = 28800 # 8hrs, 2 hours is good for most tasks.
interactive_timeout = 28800 # 8hrs
skip_name_resolve = 1

# TABLE / METADATA CACHE
table_open_cache = 8000
table_definition_cache = 4000
open_files_limit = 65535

# INNODB MEMORY
innodb_buffer_pool_size = 8G # adjust according to the size of your database and available RAM.
innodb_buffer_pool_instances = 8
innodb_log_buffer_size = 128M

# INNODB LOGGING
innodb_log_file_size = 2G
#innodb_flush_log_at_trx_commit: 2 will lose data in the last 1s in case of failier but faster, 1 every commit is written to the disk safer but slower.
innodb_flush_log_at_trx_commit = 2

# INNODB BUFFER POOL WARMUP
innodb_buffer_pool_dump_at_shutdown = 1
innodb_buffer_pool_load_at_startup = 1
innodb_buffer_pool_dump_pct = 75

# INNODB IO / STORAGE
innodb_flush_method = O_DIRECT
innodb_file_per_table = 1
# innodb_flush_neighbors: 0 for ssd nvme
innodb_flush_neighbors = 1

# IO TUNING
# innodb_io_capacity: for raid HDD, for just HDD 100 400, for ssd nmve 2000 4000.
innodb_io_capacity = 500
innodb_io_capacity_max = 1000
innodb_read_io_threads = 8
innodb_write_io_threads = 8

# OPTIMIZER / STATS
innodb_stats_on_metadata = 0

# DISABLE FEATURES THAT SCALE BADLY
query_cache_type = 0
query_cache_size = 0
innodb_adaptive_hash_index = OFF

# SAFE PER-CONNECTION BUFFERS
read_buffer_size = 2M
read_rnd_buffer_size = 1M
sort_buffer_size = 2M
join_buffer_size = 2M

# TEMP TABLES
tmp_table_size = 128M
max_heap_table_size = 128M

# PACKET SIZE
max_allowed_packet = 512M

```



---
---
# Redis:

> **WARNING**: Watch out for memory buffers as it might over load your system memory and crash your application.

```bash
vim  /etc/redis/redis.conf
```

**Make sure you have the following configs:**
```bash

# Edit:
supervised systemd # manage start, stop, restart and status usnig systemd

# default: 6379, We will use unixsocket
port 0
requirepass <STRONG_PASSWORD> # add to moodle config.php
unixsocket /run/redis/redis-server.sock
unixsocketperm 770 # u:rwx, g:rwx, o:---

maxmemory 16384mb                 # max memory redis can use.
maxmemory-policy volatile-lru  # Evict MUC least-recently-used keys when full but not session keys.

# when memory is full, key deleting is done in the background
lazyfree-lazy-eviction yes
lazyfree-lazy-expire yes

loglevel notice

# --------------------------------------------------- #

# save 3600 1 300 100 60 10000
# ----------------------------------------
# 1. SNAPSHOTTING (RDB)
# ----------------------------------------
# Save the DB if:
# - 15 mins have passed and at least 1 key changed
# - 5 mins have passed and at least 10 keys changed
# - 1 min has passed and at least 10,000 keys changed
# Save Time(s) Keys
save 900 1
save 300 10
save 60 10000

# Stop accepting writes if RDB snapshots fail (safer to keep enabled)
stop-writes-on-bgsave-error yes

# Compress string objects using LZF compression (saves disk space)
rdbcompression yes

# CRITICAL: Turn on AOF to prevent session loss on unexpected crashes
appendonly yes

# The name of the append-only file
appendfilename "appendonly.aof"

# How often to sync to disk. 
# 'everysec' is the best balance of speed and data safety.
appendfsync everysec

# Don't sync AOF to disk while an RDB snapshot is running 
# This prevents major disk I/O bottlenecks
no-appendfsync-on-rewrite yes

# Automatically rewrite the AOF file when it grows too large
auto-aof-rewrite-percentage 100 # only rewrite when file reaches %100
auto-aof-rewrite-min-size 64mb # rewrite when file is this size

# Maximum number of simultaneous client connections Redis will accept before refusing new ones.
maxclients 50000
```

---
---
