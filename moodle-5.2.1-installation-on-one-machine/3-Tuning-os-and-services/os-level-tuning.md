#### References:

https://cubepath.com/docs/Performance%20Optimization/linux-performance-optimization-sysctl-and-kernel-parameters

https://fasterdata.es.net/host-tuning/linux/

https://oneuptime.com/blog/post/2026-01-07-ubuntu-network-performance-tuning/view

---

> These configs are suited to a **high-traffic web server, proxy, or load balancer** with 16 GB+ RAM. On a desktop or small VPS.

---
#### Show any value of a service in systemd:

```bash
systemctl show <SERVICE_NAME> --property=<PROPERTY_NAME>
```

```bash
systemctl show apache2 --property=LimitNOFILE
```
#### Extend the limits for the services in `systemd`:

> These limits are related to the number of files the service can open as every thing on linux is a file including sockets/connections and processes/workers.

> **`LimitNOFILE=1048576`**: Sets the maximum number of open file descriptors (hard limit, and soft too if no separate soft value is given) for the service's processes to about 1 million.

> **`LimitNOFILESoft=1048576`**: Sets only the soft limit on open file descriptors to 1,048,576, which is the value processes actually run with unless they raise it themselves up to the hard limit.

> **`LimitNPROC=202066`**: Caps the number of processes/threads the service's user can have at once to 202,066.

```bash
systemctl edit apache2
```

**Add to the top:**
```bash
[Service]
LimitNOFILE=1048576
LimitNOFILESoft=1048576
LimitNPROC=202066
```

```bash
systemctl daemon-reload
systemctl restart apache2
```

```bash
systemctl edit mariadb
```

**Add to the top:**
```bash
[Service]
LimitNOFILE=1048576
LimitNOFILESoft=1048576
LimitNPROC=202066
```

```bash
systemctl daemon-reload
systemctl restart mariadb
```


```bash
systemctl edit redis-server
```

**Add to the top:**
```bash
[Service]
LimitNOFILE=1048576
LimitNPROC=202066
```

```bash
systemctl daemon-reload
systemctl restart redis-server
```

```bash
systemctl edit php8.4-fpm
```

**Add to the top:**
```bash
[Service]
LimitNOFILE=1048576
LimitNOFILESoft=1048576
LimitNPROC=202066
```

```bash
systemctl daemon-reload
systemctl restart php8.4-fpm 
```

---
---

> **WARNING**: Watch out for memory buffers as it might over load your system memory and crash your application.

> These `TCP` configs are for a VM with single network interface card.

**To Show any value: Attribute**
```bash
sysctl <ATTRIBUE_NAME>
```

**EXP:**
```bash
sysctl vm.swappiness
```

**Output:**
```bash
vm.swappiness = 60
```


```bash
vim /etc/sysctl.d/99-moodle-perf.conf
```

```bash
# Virtual Memory & System Tweaks
vm.swappiness = 10
vm.min_free_kbytes = 262144
fs.file-max = 4194304
fs.inotify.max_user_watches = 524288
# vm.dirty_expire_centisecs = 6000
vm.dirty_ratio = 10
vm.dirty_background_ratio = 5


# TCP Buffer Size Optimization
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.core.rmem_default = 1048576
net.core.wmem_default = 1048576
net.ipv4.tcp_rmem = 4096 131072 16777216
net.ipv4.tcp_wmem = 4096 131072 16777216
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_timestamps = 1
net.ipv4.tcp_sack = 1

# Connection Backlog Tuning
net.ipv4.tcp_max_syn_backlog = 65536
net.core.somaxconn = 65536
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_max_orphans = 262144

# TIME_WAIT & Outgoing Connections Port Optimization
net.ipv4.tcp_tw_reuse = 1
net.ipv4.ip_local_port_range = 1024 65535
net.ipv4.tcp_fin_timeout = 30


# Additional Network Optimizations
net.core.netdev_max_backlog = 65536
net.core.netdev_budget = 600
net.core.netdev_budget_usecs = 6000
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.neigh.default.gc_thresh1 = 4096
net.ipv4.neigh.default.gc_thresh2 = 8192
net.ipv4.neigh.default.gc_thresh3 = 16384
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_probes = 5
net.ipv4.tcp_keepalive_intvl = 15
net.ipv4.tcp_slow_start_after_idle = 0

# BBR Congestion Control
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr

```

**Apply:**
```bash
sysctl -p
sysctl --system
```

#### Virtual Memory & System:

> a **stall** is a temporary delay or pause in the execution of an instruction inside the processor.

> On small VPS memory < `16GB`, lower the memory-related values (`min_free_kbytes`).

| Setting                            | What it does                                                                                                    | Range                         | Recommended                                                                         |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------- | ----------------------------- | ----------------------------------------------------------------------------------- |
| `vm.swappiness = 10`               | How eagerly the kernel swaps RAM to disk. Lower means it keeps apps in RAM.                                     | 0-100 (default 60)            | Desktop: 10-30. General server: 10-30. Database/high-traffic: 1-10                  |
| `vm.min_free_kbytes = 262144`      | RAM (here 256 MB) the kernel keeps free to avoid stalls under pressure.                                         | ~1024 to ~5% of RAM           | Low-RAM: 65536. General: 128 MB. High-traffic (16 GB+ RAM): 256 MB+                 |
| `fs.file-max = 4194304`            | System-wide limit on open files and sockets.                                                                    | 100k to millions              | Desktop: 500k. General: 1M. High-traffic: 2-4M+                                     |
| `fs.inotify.max_user_watches`      | Number of files one user can watch for changes (IDEs, sync tools).                                              | 8192 to 1M+ (each uses ~1 KB) | Desktop/dev: 524288. Server: 65536-262144                                           |
| `vm.dirty_expire_centisecs = 6000` | How long (60 s) modified data sits in RAM before it must be written to disk, higher number = better performance | 100-6000+ (default 3000)      | Desktop: 3000. Server: 3000-6000. Safer for data in unstable environments: lower it |
| `vm.dirty_ratio = 10`              | When dirty data hits this % of RAM, apps are forced to wait while it is written to disk.                        | 1-100 (default 20)            | Low-RAM: 10-15. Server: 10-20. Fast SSD/NVMe: 10                                    |
| `vm.dirty_background_ratio = 5`    | When dirty data hits this %, background writing starts. It must be lower than `dirty_ratio`.                    | 0-100 (default 10)            | Half of `dirty_ratio` (5 is good)                                                   |

> `Dirty data`: is data that has been changed in RAM but not yet written to disk.

> `vm.dirty_expire_centisecs = 6000`: if the machine went off for any reason within that time you might lose the data in the ram for that time.

#### TCP Buffers:

> `backlog`: anything waiting to be achieved or processed(connections waiting in this case).

> `SYN`: synchronize.

> `ACK`: acknowledge.

> **`rmem_default` / `wmem_default` at 1 MB** can waste RAM if you have tens of thousands of sockets.

> On small VPS memory < `16GB`, lower the memory-related values (buffers).

| Setting                                     | What it does                                                                        | Range                   | Recommended                                                                                  |
| ------------------------------------------- | ----------------------------------------------------------------------------------- | ----------------------- | -------------------------------------------------------------------------------------------- |
| `rmem_max` / `wmem_max = 16 MB`             | Maximum receive(r)/send(w) buffer any socket can request.                           | Bytes (default ~212 KB) | 1 Gbps: 16 MB. 10 Gbps or long distance: 32-128 MB                                           |
| `rmem_default` / `wmem_default = 1 MB`      | Starting buffer for each socket.                                                    | Bytes (default ~212 KB) | Many connections: 256 KB (1 MB × 100k sockets uses a lot of RAM). Few fast connections: 1 MB |
| `tcp_rmem` / `tcp_wmem` (min, default, max) | TCP buffer sizes that auto-grow between min and max.                                | Three byte values       | `4096 131072 16777216` is a good general choice. Raise the max for fast or long links        |
| `tcp_window_scaling = 1`                    | Allows windows over 64 KB, needed for fast links.                                   | 0/1                     | Always 1                                                                                     |
| `tcp_timestamps = 1`                        | Adds timestamps for better RTT measurement and protection against old packets.      | 0/1                     | 1 (0 saves a little CPU/bandwidth but is rarely worth it)                                    |
| `tcp_sack = 1`                              | Lets the receiver report exactly which packets were lost, so only those are resent. | 0/1                     | Always 1                                                                                     |

> `rmem_max` / `wmem_max = 16 MB`: watch out for the memory as this is maximum size per tcp connection -> doesn't mean each connection is going to take that size.

> `tcp_window_scaling = 1`: A TCP "window" is how much data a sender can have in flight (sent but not yet acknowledged) before it has to stop and wait for the receiver to confirm.
> Original TCP stores the window size in a 16-bit field, so it can't exceed **64 KB** (65535 bytes). `tcp_window_scaling = 1` turns on an extension that multiplies that number, allowing windows up to about 1 GB.

#### Connection Backlog:

> `backlog`: anything waiting to be achieved or processed(connections waiting in this case).

> `SYN`: synchronize.

> `ACK`: acknowledge.

> On small VPS memory < `16GB`, lower the memory-related values (`max_orphans`).

| Setting                       | What it does                                                                                     | Range             | Recommended                                                                           |
| ----------------------------- | ------------------------------------------------------------------------------------------------ | ----------------- | ------------------------------------------------------------------------------------- |
| `tcp_max_syn_backlog = 65536` | Queue of half-open (SYN received) connections.                                                   | 128-65536+        | Desktop: default. Server: 4096-8192. High-traffic: 65536                              |
| `somaxconn = 65536`           | Maximum queue of fully established connections waiting for the app to accept them.               | 128-65535         | Server: 4096. High-traffic: 65535. The app must also raise its own `listen()` backlog |
| `tcp_syncookies = 1`          | Protection against SYN flood attacks.                                                            | 0/1/2             | 1                                                                                     |
| `tcp_synack_retries = 2`      | How many times the server resends SYN-ACK. Lower means half-open connections are dropped faster. | 1-255 (default 5) | Server: 2-3. Unreliable networks: 4-5                                                 |
| `tcp_syn_retries = 2`         | How many times a client retries the SYN. At 2, a failed connect gives up after about 7 s.        | 1-255 (default 6) | Internal/fast networks: 2-3. Internet or mobile clients: 4-6                          |
| `tcp_max_orphans = 262144`    | Maximum sockets not attached to any process (closing connections).                               | 4096-millions     | Low-RAM: 16384-65536. High-traffic: 262144+                                           |

#### TIME_WAIT & Outgoing Connections Port Optimization:

> Matters if DB/Redis is remote.

> When you have multiple proccesses/workers on your app machine wanting to connect to the database, redis or whatever(outgoing connectionis), each outgoing connection is assigned and internal port (default 32768-60999 -> 28,000 ports) so that when the DB responds the OS can tell which process should get that response, EXP: `web-server-IP:41822 → DB-IP:3306`.

> Here we are setting the tcp connection to use same port for any other process after the current one is finished to avoid closing and reopening connections -> slower performance.

> Extended the range of these internal IPs to avoid The connection attempt fails with "Cannot assign requested address", in case you need more than 28,000 ports, which is not that common.

| Setting                                     | What it does                                                                                                                                              | Range              | Recommended                                                                            |
| ------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------ | -------------------------------------------------------------------------------------- |
| `net.ipv4.tcp_tw_reuse = 1`                 | Reuses TIME_WAIT ports for new outgoing connections.<br>0 = off, 1 = on, 2 = loopback only.<br>**Benefit:** no port exhaustion with a remote DB or Redis. | 0/1/2              | **DB/Redis on another machine:** 1<br>**Everything on one machine:** default is fine   |
| `net.ipv4.ip_local_port_range = 1024 65535` | Ports available for outgoing connections.<br>**Benefit:** more simultaneous outgoing connections.                                                         | 1024-65535         | **Remote DB/Redis:** full range<br>**Single machine:** default (32768-60999) is fine   |
| `net.ipv4.tcp_fin_timeout = 30`             | Seconds a closing connection waits in FIN-WAIT-2.<br>**Benefit:** frees sockets and memory faster.                                                        | 1-120 (default 60) | **Moodle at 10k users:** 30<br>**Avoid below 10:** it can cause problems on slow links |

#### Additional Network Optimizations:

> `netdev`: Network Device.

> `usecs`: (u)micro (secs)Seconds.

> `netdev_budget_usecs`: Time limit for one round of processing incoming network packets per CPU core, before the kernel takes a break to let the rest of the system run.

> **`rp_filter = 1`** can silently drop traffic if you use multiple interfaces, VPNs, or asymmetric routing. Change it to 2 in those cases.

| Setting                         | What it does                                                                                       | Range                      | Recommended                                                                           |
| ------------------------------- | -------------------------------------------------------------------------------------------------- | -------------------------- | ------------------------------------------------------------------------------------- |
| `netdev_max_backlog = 65536`    | Packet queue per CPU when the network card is faster than the kernel.                              | 1000-100000                | Server: 5000-16384. 10 Gbps: 65536                                                    |
| `netdev_budget = 600`           | Max packets processed per network polling cycle.                                                   | 300+ (default 300)         | Server: 600. Latency-sensitive: keep default                                          |
| `netdev_budget_usecs = 6000`    | Maximum time (micro socends) for one polling cycle.                                                | 1000+ (default 2000)       | 2000-8000. Lower is better for latency, as cpu needs to give time for other processes |
| `accept_source_route = 0`       | Ignores packets that dictate their own route (a security risk).                                    | 0/1                        | Always 0                                                                              |
| `accept_redirects = 0`          | Ignores ICMP messages telling you to change routes (a spoofing risk).                              | 0/1                        | 0 on servers. Also 0 on desktops unless you need them                                 |
| `rp_filter = 1`                 | Drops packets whose source address couldn't arrive on that interface (anti-spoofing).              | 0/1/2                      | Standard server: 1. Multi-homed or asymmetric routing (VPNs, multiple NICs): 2        |
| `neigh.gc_thresh1/2/3`          | ARP table sizes: below 1 nothing is cleaned, above 2 cleanup is aggressive, and 3 is a hard limit. | 128-65536+                 | Small network: defaults.                                                              |
| `tcp_keepalive_time = 600`      | Seconds of idle time before keepalive probes begin.                                                | 1-2 billion (default 7200) | 300-600 for servers behind NAT or load balancers                                      |
| `tcp_keepalive_probes = 5`      | Number of failed probes before the connection is declared dead.                                    | 1-127 (default 9)          | 3-5                                                                                   |
| `tcp_keepalive_intvl = 15`      | Seconds between probes.                                                                            | 1-255 (default 75)         | 10-30                                                                                 |
| `tcp_slow_start_after_idle = 0` | Keeps the connection speed after idle instead of restarting slowly.                                | 0/1                        | 0 for web servers and persistent connections                                          |

#### Congestion Control & RPS:

| Setting                         | What it does                                                                            | Range                       | Recommended                                                                                                            |
| ------------------------------- | --------------------------------------------------------------------------------------- | --------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `default_qdisc = fq`            | Packet scheduler. `fq` is needed for BBR to pace traffic properly.                      | fq, fq_codel, pfifo_fast... | `fq` with BBR. `fq_codel` for desktops/routers                                                                         |
| `tcp_congestion_control = bbr`  | Algorithm that decides how fast to send. BBR is better on lossy or long-distance links. | cubic, bbr, reno...         | Servers and internet users: `bbr`. Local LAN: `cubic` is fine                                                          |

---
---
#### Disable `transparent_hugepage`:

> `transparent_hugepage`: THP for short, a background memory defragmentation causes unpredictable latency spikes and (especially for Redis's fork-based snapshots) massive memory bloat, bad for DB/cache workloads that need consistent, low-latency memory access.

**Temperately but take effect now:**
```bash
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

---

**Permanent but take effect  after reboot:**
```bash
vim /etc/default/grub
```

**Add to `GRUB_CMDLINE_LINUX=""`:**
```bash
transparent_hugepage=never
```

**Should look something like this on Ubuntu:**
```bash
GRUB_CMDLINE_LINUX="transparent_hugepage=never"
```

**Redhat:**
```bash
GRUB_CMDLINE_LINUX="crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M resume=/dev/mapper/rhel-swap rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap rhgb quiet transparent_hugepage=never"
```

>save and exit.

**Update grup:**
```bash
update-grub
```

**Reboot to take effect:**
```
reboot
```

**Make sure it works:**
```bash
cat /sys/kernel/mm/transparent_hugepage/enabled
cat /sys/kernel/mm/transparent_hugepage/defrag
```

**Output should look something like this:** `[never]`.
```output
always [madvise] never
always defer defer+madvise [madvise] never
```

---
#### System limits for terminal commands:

```bash
vim /etc/security/limits.conf
```

```bash
* soft nofile 1048576
* hard nofile 1048576
* soft nproc 202066
* hard nproc 202066
```

**Check that it took effect:**
```bash
ulimit -n
```

> If it doesn't take effect try setting it temperately:

**Temperately set it:**
```bash
ulimit -n 10240
```

**Check that it took effect:**
```bash
ulimit -n
```

**Output:**
```bash
10240
```

> **Note:** this value will get back to default `1024` once you close that terminal.

---