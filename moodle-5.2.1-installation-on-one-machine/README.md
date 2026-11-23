> This tutorial was meant to install `Moodle`  on `VPS` or `on-premises`.

> Make sure my tuning and parameters' values suit your setup.

> Change passwords, usernames, database name and files' locations if needed for security reasons.

---
# Final Result:

![](./assets/Pasted%20image%2020260617214415.png)

---
# Summary and Version Control:

| Software     | Version         | Port         |
| ------------ | --------------- | ------------ |
| OS           | Ubuntu 24.04    | UFW: 80, 443 |
| Database     | MariaDB 12.3    | Socket, 3306 |
| Front        | Apche2 2.4.58   | 80, 443      |
| Back         | PHP/PHP-FPM 8.4 | Socket, fcgi |
| Memory Cache | Redis 7.0.15    | Socket, 6379 |
| Code         | Moodle v5.2.1   | 80, 443      |

> In this exercise you will learn how to setup LAMP stack.

> Make sure your server has access to the internet.

> Reference guide: https://docs.moodle.org/502/en/Step-by-step_Installation_Guide_for_Ubuntu

> Active moodle branches https://github.com/moodle/moodle/branches/active

> Moodle website: https://moodle.org/

> Tuning OS and services will be in a separate file.

> If you are in production environment,you should have enough space in the /tmp or / because some tasks require free temporary space, 50G of free space is good enough for most cases.

> Most of the cpu usage is gonna be by php-fpm, and most of the ram usage is gonna be by the database cache.

> In this exercise the entire app in on one machine, to achieve high availability use clustering at the hypervisor level with `ceph` or whatever method you prefer.

---
# Hardware Recommendations For The VM:

> Assuming that the website is new and the database is almost empty.

> When users upload more data and the database expands you will need more disk.

> I totally recommend that you make the OS: `/`, database: `/var/lib/mysql` and moodledata: `/srv/moodle/moodledata` in separate mount points as the database and moodledata expand and you might get in a situation where the machine is stuck and the app can't function anymore because there is no more free space.

> Make sure you use LVM for your disks as it makes it easier to maintain and expand in the future.

> CPU generation shouldn't be older than 5 years.

> In a perfect scenario with `on-premises` setups `Logical CPUs` = `Virtual CPUs`.

| Size            | vCPUs  | RAM       | Disk             | Network         | Concurrent Users      |
| --------------- | ------ | --------- | ---------------- | --------------- | --------------------- |
| **Extra Small** | 1-2    | 1-2 GB    | 20-40 GB SSD     | 100 Mbps-1 Gbps | 10 concurrent users   |
| **Small**       | 2-4    | 2-4 GB    | 20-40 GB SSD     | 100 Mbps-1 Gbps | 100 concurrent users  |
| **Medium**      | 4-8    | 6-8 GB    | 40-80 GB SSD     | 1 Gbps          | 500 concurrent users  |
| **Large**       | 8-16   | 16-32 GB  | 80-160 GB NVMe   | 1-2 Gbps        | 1000 concurrent users |
| **Extra Large** | 16-32+ | 32-64+ GB | 160-500+ GB NVMe | 2-10 Gbps       | 5000 concurrent users |

#### Recommended Disk Layout:

> Swap in Ubuntu 24.04 and above is a file in `/` and no longer a separate partition.

> You can use separate disks for each mount point for mobility, or just one disk with multiple partitions.

|             | Mount Point            | Size   | Note                                                                            |
| ----------- | ---------------------- | ------ | ------------------------------------------------------------------------------- |
| OS          | `/`                    | 60 GB  | Minimum                                                                         |
| Database    | /var/lib/mysql         | 100 GB | You can start 10 GB if you want                                                 |
| Moodle Data | /srv/moodle/moodledata | 500 GB | Unless you will allow students to upload assignments then 10 GB is good enough. |
