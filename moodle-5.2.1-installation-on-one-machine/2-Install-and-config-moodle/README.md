# Final Result:

![](./assets/Pasted%20image%2020260617214415.png)

---
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

---
# **First switch to `root` user:**

```bash
sudo su - root
```

---
# Setup Credentials And Variables:

> Every coming step depends on these variables.

> These variable will be stored in `~/.env_vars` and auto loaded when your current user logs in.

> Change the values of these variables according to your situation and needs.

> `$(hostname -I | awk '{print $1}')` command to get the IP of the machine.

```bash
vim ~/.env_vars
```

**Add the following:**
```bash

export MOODLE_DATA_PATH=/srv/moodle
export MOODLE_DATA_FOLDER=${MOODLE_DATA_PATH}/moodledata
export MOODLE_CODE_PATH=/var/www/html
export MOODLE_CODE_FOLDER=${MOODLE_CODE_PATH}/moodle
export COMPOSER_CACHE_FOLDER=${MOODLE_DATA_PATH}/.cache/composer

export MOODLE_VERSION_TAG=v5.2.1
export PHP_VERSION=8.4


# Apache2:
export MOODLE_APACHE_CONFIG_FILE=my-moodle-site.com.conf
export APACHE_LOGS_FOLDER=/var/log/apache2
export MOODLE_APACHE_ROOT_DOCUMENT=${MOODLE_CODE_FOLDER}/public
# PROTOCOL=https://
export PROTOCOL=http://
# export SERVER_ADDRESS=e-learning.mans.edu.eg
export SERVER_ADDRESS=$(hostname -I | awk '{print $1}')
export WEBSITE_PORT=80
export WEBSITE_SECURE_PORT=443
export APACHE_SSL_PRIVATE_KEY=/etc/ssl/private/private_key.key
export APACHE_SSL_PUBLIC_KEY=/etc/ssl/certs/public_key.crt
export APACHE_SSL_CA_FULL_CHAIN=/etc/ssl/certs/CA_fullChain.crt

# Redis:
export REDIS_PASSWORD=<STRONG_PASSWORD>
export REDIS_SOCKET=/run/redis/redis-server.sock
export REDIS_PORT=6379

export LANGUAGE=en


# Site Name:
export SITE_LONG_NAME="Mansourah University FCI."
export SITE_SHORT_NAME="Mans. F.C.I."


# Admin:
export MOODLE_ADMIN_USER_NAME=admin
export MOODLE_ADMIN_PASSWORD=<STRONG_PASSWORD>
export MOODLE_ADMIN_EMAIL=admin@example.com


# Database:
export MOODLE_DATABASE_USER=moodle_user
export MOODLE_DATABASE_USER_PASSWORD=<STRONG_PASSWORD>
export MOODLE_DATABASE_NAME=moodle_db
export MOODLE_DATABASE_TYPE=mariadb
export MOODLE_DATABASE_SOCKET=/run/mysqld/mysqld.sock
export MOODLE_DATABASE_HOST=localhost
export MOODLE_DATABASE_PORT=3306
export DATABASE_PREFEX=mdl_

```

> Save and exit -> `Esc` -> `:wq!`.

> Replace `<STRONG_PASSWORD>` for `MOODLE_DATABASE_USER_PASSWORD`, `MOODLE_ADMIN_PASSWORD` and  `REDIS_PASSWORD`.

**Hint:** Generate a password for each one `MOODLE_DATABASE_USER_PASSWORD`, `MOODLE_ADMIN_PASSWORD` and  `REDIS_PASSWORD`:
```bash
openssl rand -base64 32 | tr '+/=\\' '_'
openssl rand -base64 32 | tr '+/=\\' '_'
openssl rand -base64 32 | tr '+/=\\' '_'
```

#### Load variables on new sessions:

> **Hint**: In `vim` Press `shift` + `g` to go to the bottom, then press `o` to insert in a new line.

**Load when you login or run bash locally:**
```bash
vim ~/.bashrc
```

**Add to the end:**
```bash
# Load custom variables
if [ -f ~/.env_vars ]; then
    source ~/.env_vars
fi
```

> Save and exit -> `Esc` -> `:wq!`.

**Load for ssh sessions and local logins:**
```bash
vim ~/.bash_profile
```

**Add to the end:**
```bash
# /root/.bash_profile
if [ -f ~/.bashrc ]; then
    source ~/.bashrc
fi
```

> Save and exit -> `Esc` -> `:wq!`.

**Load now:**
```bash
source ~/.bashrc
```

#### Print passwords:

**Run:**
```bash
# Print Passwords:
echo -e "\033[31mNote: Please Save This Password.\033[0m" # red text
echo -e "\033[34mMoodle Admin Password Is:\033[0m" # blue text
echo -e "\033[32m${MOODLE_ADMIN_PASSWORD}\033[0m" # green text
echo -e "\033[34mDatabase Password Is:\033[0m" # blue text
echo -e "\033[32m${MOODLE_DATABASE_USER_PASSWORD}\033[0m" # green text
echo -e "\033[34mRedis Password Is:\033[0m" # blue text
echo -e "\033[32m${REDIS_PASSWORD}\033[0m" # green text
```

**Output should look something like this:**
![](./assets/Pasted%20image%2020260624001328.png)

---
---
# Database Installation:

[Install MariaDB Official Guide]([https://mariadb.org/download/?t=repo-config&d=24.04+%22noble%22&v=12.3&r_m=liquidtelecom](https://mariadb.org/download/?t=repo-config&d=24.04+%22noble%22&v=12.3&r_m=liquidtelecom))

[MariaDB Master Slave Replication](https://mariadb.com/docs/server/ha-and-performance/standard-replication/setting-up-replication)

> **Note**: Internet connection required for this step.

> **IMPORTANT**: Have a remote storage where you can dump the database daily and a replica database on a different machine as a backup for your data in case you are not using `ceph` or any other HA setup architecture.

---
#### Import the MariaDB repository key on your Ubuntu system:

```bash
apt install apt-transport-https curl -y
mkdir -p /etc/apt/keyrings
curl -o /etc/apt/keyrings/mariadb-keyring.asc 'https://mariadb.org/mariadb_release_signing_key.pgp'
```

#### Create `MariaDB 12.3` repository for `apt` package manager:

```bash
cat >> /etc/apt/sources.list.d/mariadb.sources << EOF
# MariaDB 12.3 repository list - created 2026-06-05 20:39 UTC
# https://mariadb.org/download/
X-Repolib-Name: MariaDB
Types: deb
# deb.mariadb.org is a dynamic mirror if your preferred mirror goes offline. See https://mariadb.org/mirrorbits/ for details.
# URIs: https://deb.mariadb.org/12.3/ubuntu
URIs: https://mariadb.mirror.liquidtelecom.com/repo/12.3/ubuntu
Suites: noble
Components: main main/debug
Signed-By: /etc/apt/keyrings/mariadb-keyring.asc
EOF
```

**Install the database:**
```bash
apt update
DEBIAN_FRONTEND=noninteractive apt install mariadb-server mariadb-client -y
```

> `DEBIAN_FRONTEND=noninteractive` makes you avoid this question:

![](./assets/Pasted%20image%2020260614224912.png)

**Check status:**
```bash
systemctl status --no-pager mariadb
```

> Should be `active (running)`.

**Check version:**
```bash
mariadb --version
```

**Output:**
```bash
mariadb from 12.3.2-MariaDB, client 15.2 for debian-linux-gnu (x86_64) using  EditLine wrapper
```

---
#### Secure MariaDB:

```bash
mariadb-secure-installation
```

| #   | Question                              | Answer                   | Explanation                                                                                                                                                                             |
| --- | ------------------------------------- | ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Current password for root             | _Press Enter_ (if fresh) | If root user has no password(default) press enter.                                                                                                                                      |
| 2   | Switch to unix_socket authentication? | `y`                      | `root` user can only connect to the database server locally using `unix_socket` instead of a password, if `n` that means root can connect remotely with a password which is not secure. |
| 3   | Change root password?                 | `y`                      | In case you answer step `2` and `5` with `n` this changes the password for the root user for the database server only both locally and remotely.                                        |
| 4   | Remove anonymous users?               | `y`                      | Fresh MariaDB installs and creates an anonymous user that lets anyone log in without credentials.                                                                                       |
| 5   | Disallow root login remotely?         | `y`                      | Never let root login remotely.                                                                                                                                                          |
| 6   | Remove test database?                 | `y`                      | Remove unnecessary default database.                                                                                                                                                    |
| 7   | Reload privilege tables now?          | `y`                      | Changes above take effect immediately without restarting MariaDB.                                                                                                                       |

---
#### Create the `${MOODLE_DATABASE_USER}` and `${MOODLE_DATABASE_NAME}` moodle app will use:

> By default if you use `localhost` mariadb uses local unix socket, but if you use `127.0.0.1` mariadb uses TCP internal network.

**Create a new database with the `utf8mb4` character set and `utf8mb4_unicode_ci` collation:**
```bash
mariadb -e "CREATE DATABASE ${MOODLE_DATABASE_NAME} DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

**Show databases:**
```bash
mariadb -e "show databases;"
```

**Output:**
```bash
+--------------------+
| Database           |
+--------------------+
| information_schema |
| moodle_db          |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

**Create `${MOODLE_DATABASE_USER}` for moodle:**
```bash
mariadb -e "CREATE USER '${MOODLE_DATABASE_USER}'@'${MOODLE_DATABASE_HOST}' IDENTIFIED BY '${MOODLE_DATABASE_USER_PASSWORD}';"
```

**Show `${MOODLE_DATABASE_USER}` in database:**
```bash
mariadb -e "SELECT User, Host FROM mysql.user;"
```

**Output:**
```bash
+-------------+-----------+
| User        | Host      |
+-------------+-----------+
| mariadb.sys | localhost |
| moodle_user | localhost |
| mysql       | localhost |
| root        | localhost |
+-------------+-----------+
```

**Grant privileges on the database to `${MOODLE_DATABASE_USER}` to allow access locally:**
```bash
mariadb -e "GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, CREATE TEMPORARY TABLES, DROP, INDEX, ALTER ON ${MOODLE_DATABASE_NAME}.* TO '${MOODLE_DATABASE_USER}'@'${MOODLE_DATABASE_HOST}';"
```

**Make everything take effect:**
```bash
mariadb -e "FLUSH PRIVILEGES;"
```

**Check that `${MOODLE_DATABASE_USER}` can access `${MOODLE_DATABASE_NAME}` from `${MOODLE_DATABASE_HOST}`:**
```bash
mariadb -e "SELECT User, Host, Db FROM mysql.db ORDER BY User;"
```

```bash
+-------------+-----------+-----------+
| User        | Host      | Db        |
+-------------+-----------+-----------+
| moodle_user | localhost | moodle_db |
+-------------+-----------+-----------+
```

**Test connection for `root` user:**
```bash
mariadb -u root -p --socket=/run/mysqld/mysqld.sock
```

> Just press enter without password.

**Output:**
```bash
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 22
Server version: 12.3.2-MariaDB-ubu2404 mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> exit
Bye
```

**Restrict access to socket:**
```bash
chmod 770 /run/mysqld/mysqld.sock
```

#### Add `www-data` user to group `mysql`:

> So that it can access the database throw its local unix socket.

```bash
usermod -aG mysql www-data
```

**Test connection for `${MOODLE_DATABASE_USER}` user:**
```bash
mariadb -u ${MOODLE_DATABASE_USER} --password=${MOODLE_DATABASE_USER_PASSWORD}  --socket=/run/mysqld/mysqld.sock
```

**Output:**
```bash
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 23
Server version: 12.3.2-MariaDB-ubu2404 mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> exit
Bye
```

---
---
# Install Necessary Packages:

> **Note**: Internet connection required for this step.

**Add php repository then update:**
```bash
apt install -y software-properties-common ca-certificates lsb-release apt-transport-https
add-apt-repository ppa:ondrej/php -y
apt update
```

**Install required packages:**
```bash
apt install -y php${PHP_VERSION}-fpm php${PHP_VERSION}-cli php${PHP_VERSION}-curl php${PHP_VERSION}-zip php${PHP_VERSION}-gd php${PHP_VERSION}-xml php${PHP_VERSION}-intl  php${PHP_VERSION}-mbstring php${PHP_VERSION}-xmlrpc php${PHP_VERSION}-soap php${PHP_VERSION}-bcmath php${PHP_VERSION}-exif php${PHP_VERSION}-ldap php${PHP_VERSION}-mysql php${PHP_VERSION}-redis ufw vim graphviz aspell git clamav ghostscript composer libapache2-mod-fcgid apache2 redis-server redis-tools
```

> Will take sometime.

> `php*`: PHP packages needed by moodle.

> `graphviz`: For Moodle quiz/report diagrams.

> `aspell`: Used by applications to check spelling in various languages.

> `clamav`: Open-source antivirus engine. Used to scan uploaded files, email attachments, and storage for malware.

> `ghostscript`: Processes PostScript (PS) and PDF files. Can convert PDFs, generate thumbnails, merge PDFs, and render pages as images.

> `composer`: Dependency/package manager for PHP. Similar to npm for Node.js or pip for Python. Used to install PHP libraries defined in composer.json.

**Check PHP version:**
```bash
php -v
```

**Output:**
```bash
PHP 8.4.22 (cli) (built: Jun  6 2026 06:34:49) (NTS)
Copyright (c) The PHP Group
Built by Ubuntu
Zend Engine v4.4.22, Copyright (c) Zend Technologies
    with Zend OPcache v8.4.22, Copyright (c), by Zend Technologies
```

---
---
# Clone Moodle From Github:

> **Note**: Internet connection required for this step.
#### Clone:

```bash
git clone --branch ${MOODLE_VERSION_TAG} --single-branch --depth 1 https://github.com/moodle/moodle.git ${MOODLE_CODE_FOLDER}
```

> Clone a specific moodle version to `${MOODLE_CODE_FOLDER}`.

**Output:**
```bash
Cloning into '/var/www/html/moodle'...
remote: Enumerating objects: 71381, done.
remote: Counting objects: 100% (71381/71381), done.
remote: Compressing objects: 100% (34700/34700), done.
remote: Total 71381 (delta 33530), reused 68342 (delta 33019), pack-reused 0 (from 0)
Receiving objects: 100% (71381/71381), 94.72 MiB | 1.73 MiB/s, done.
Resolving deltas: 100% (33530/33530), done.
Note: switching to '63e16b757ca8fee05b672a27c23ee27cc8f9fabb'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

Updating files: 100% (61552/61552), done.
```

#### Create a backup so that you wont have to download it again:

```bash
cp -R ${MOODLE_CODE_FOLDER} ${MOODLE_CODE_FOLDER}-backup
```

**Take a look:**
```bash
ls -al ${MOODLE_CODE_PATH}
```

**Output:**
```bash
total 28
drwxr-xr-x  4 root root  4096 Jun 17 18:17 .
drwxr-xr-x  3 root root  4096 Jun 17 18:14 ..
-rw-r--r--  1 root root 10671 Jun 17 18:15 index.html
drwxr-xr-x 12 root root  4096 Jun 17 18:16 moodle
drwxr-xr-x 12 root root  4096 Jun 17 18:17 moodle-backup
```

---
---
# Create necessary Directories:

```bash
mkdir -p  ${MOODLE_DATA_FOLDER} ${COMPOSER_CACHE_FOLDER}
```

---
---
# Change Permissions and Ownership:

```bash

# Set Permissions:
find ${MOODLE_CODE_FOLDER} -type d -exec chmod 755 {} \;
find ${MOODLE_CODE_FOLDER} -type f -exec chmod 644 {} \;

find ${MOODLE_DATA_FOLDER} -type d -exec chmod 700 {} \;
find ${MOODLE_DATA_FOLDER} -type f -exec chmod 600 {} \;

chmod -R 750 ${COMPOSER_CACHE_FOLDER}

# Change ownership to www-data user and group:
chown -R www-data:www-data ${MOODLE_CODE_FOLDER} ${MOODLE_DATA_PATH}

```

> Will take sometime.

---
---
# PHP Configs:

> **Logs**: `/var/log/php${PHP_VERSION}-fpm.log`, `/var/log/php${PHP_VERSION}-fpm.log.1`

> Any Change should be applied to both `/etc/php/${PHP_VERSION}/fpm/php.ini` and `/etc/php/${PHP_VERSION}/cli/php.ini`.

---
#### Setup some moodle packages with compressor:

```bash
sudo -u www-data COMPOSER_CACHE_DIR=${COMPOSER_CACHE_FOLDER} composer update --working-dir=${MOODLE_CODE_FOLDER}

sudo -u www-data COMPOSER_CACHE_DIR=${COMPOSER_CACHE_FOLDER} composer install --no-dev --classmap-authoritative --working-dir=${MOODLE_CODE_FOLDER}
```

**Output looks something like this:**
```bash
...
  - Installing psr/http-server-handler (1.0.2): Extracting archive
  - Installing psr/http-server-middleware (1.0.2): Extracting archive
  - Installing nikic/fast-route (v1.3.0): Extracting archive
  - Installing slim/slim (4.13.0): Extracting archive
  - Installing spatie/php-cloneable (1.0.2): Extracting archive
Generating optimized autoload files
17 packages you are using are looking for funding.
Use the `composer fund` command to find out more!
```

#### Make sure `php${PHP_VERSION}-fpm` is working: (Optional)

```bash
sudo systemctl status --no-pager php${PHP_VERSION}-fpm
```

> Should be `active (running)`.

**Enable and start if needed:**
```bash
sudo systemctl enable --now php${PHP_VERSION}-fpm
```

#### Recommended minimum php configs for moodle:

**Just copy past in terminal:** modify later in the php tuning page.
```bash

MAX_INPUT_VARS="15000" # 5000 minimum
POST_MAX="2048M"
UPLOAD_MAX="2048M"
FILE_UPLOADS="On" # On or Off
PHP_INI_FPM="/etc/php/${PHP_VERSION}/fpm/php.ini"
PHP_INI_CLI="/etc/php/${PHP_VERSION}/cli/php.ini"

# For /etc/php/${PHP_VERSION}/fpm/php.ini
sudo sed -i "s/^;*max_input_vars = .*/max_input_vars = ${MAX_INPUT_VARS}/" "$PHP_INI_FPM"
sudo sed -i "s/^post_max_size = .*/post_max_size = ${POST_MAX}/" "$PHP_INI_FPM"
sudo sed -i "s/^file_uploads = .*/file_uploads = ${FILE_UPLOADS}/" "$PHP_INI_FPM"
sudo sed -i "s/^upload_max_filesize = .*/upload_max_filesize = ${UPLOAD_MAX}/" "$PHP_INI_FPM"


# For /etc/php/${PHP_VERSION}/cli/php.ini
sudo sed -i "s/^;*max_input_vars = .*/max_input_vars = ${MAX_INPUT_VARS}/" "$PHP_INI_CLI"
sudo sed -i "s/^post_max_size = .*/post_max_size = ${POST_MAX}/" "$PHP_INI_CLI"
sudo sed -i "s/^file_uploads = .*/file_uploads = ${FILE_UPLOADS}/" "$PHP_INI_CLI"
sudo sed -i "s/^upload_max_filesize = .*/upload_max_filesize = ${UPLOAD_MAX}/" "$PHP_INI_CLI"

systemctl restart php${PHP_VERSION}-fpm.service

```

**Take a look:**
```bash
echo -e "\033[34mIn '/etc/php/${PHP_VERSION}/fpm/php.ini':\033[0m"
cat /etc/php/${PHP_VERSION}/fpm/php.ini | grep -E "max_input_vars =|post_max_size =|file_uploads =|upload_max_filesize ="
echo ""
echo -e "\033[32mIn '/etc/php/${PHP_VERSION}/cli/php.ini':\033[0m"
cat /etc/php/${PHP_VERSION}/cli/php.ini | grep -E "max_input_vars =|post_max_size =|file_uploads =|upload_max_filesize ="
```

**Output should look something like this:**
![](./assets/Pasted%20image%2020260623125124.png)

> **Important**: Edit variables in both `/etc/php/${PHP_VERSION}/fpm/php.ini` and `/etc/php/${PHP_VERSION}/cli/php.ini`.

> `max_input_vars`: number of fields in a quiz or grades sheet ...., 5000 for small ones, 10000 for large, 20000 for massive question banks and other things that need many fields.

> If `max_input_vars` is smaller than needed you will experience truncation while uploading a sheet with many fields for example.

> `upload_max_filesize`, `post_max_size` controls the upper size limit of files that can be uploaded to moodle.

> `upload_max_filesize` if = 0 that means upload is disabled from web GUI.

> `post_max_size` if = 0 that means upload is disabled.

> `file_uploads = Off` or `On` disable or enable file upload at all.

or 
#### Manually edit php variables: `upload_max_filesize`, `post_max_size`, `file_upload`, `max_input_vars` in the following files:

```bash
vim /etc/php/${PHP_VERSION}/fpm/php.ini
```

```bash
vim /etc/php/${PHP_VERSION}/cli/php.ini
```

---
#### Enable `cron.php`: (IMPORTANT)

> Calls the cron.php in the moodle admin directory to run every minute. 

```bash
echo "* * * * * /usr/bin/php ${MOODLE_CODE_FOLDER}/admin/cli/cron.php >/dev/null" | sudo crontab -u www-data -
```

**Show cron jobs for www-data user:**
```bash
sudo -u www-data crontab -l
```

**Output:**
```bash
* * * * * /usr/bin/php /var/www/html/moodle/admin/cli/cron.php >/dev/null
```

> Runs every minute.

---
#### Configure moodle:

```bash

sudo -u www-data /usr/bin/php${PHP_VERSION} ${MOODLE_CODE_FOLDER}/admin/cli/install.php \
--non-interactive \
--agree-license \
--allow-unstable \
--lang="${LANGUAGE}" \
--wwwroot="${PROTOCOL}${SERVER_ADDRESS}" \
--dataroot="${MOODLE_DATA_FOLDER}" \
--fullname="${SITE_LONG_NAME}" \
--shortname="${SITE_SHORT_NAME}" \
--adminuser="${MOODLE_ADMIN_USER_NAME}" \
--adminpass="${MOODLE_ADMIN_PASSWORD}" \
--adminemail="${MOODLE_ADMIN_EMAIL}" \
--summary="" \
--dbtype="${MOODLE_DATABASE_TYPE}" \
--dbhost="${MOODLE_DATABASE_HOST}" \
--dbname="${MOODLE_DATABASE_NAME}" \
--dbuser="${MOODLE_DATABASE_USER}" \
--dbpass=${MOODLE_DATABASE_USER_PASSWORD} \
--dbsocket=${MOODLE_DATABASE_SOCKET} \
--dbport=${MOODLE_DATABASE_PORT}

```

**See all options:**
```bash
sudo -u www-data /usr/bin/php ${MOODLE_CODE_FOLDER}/admin/cli/install.php --help
```

> If your are using a `load balancer` or `NAT` change `--wwwroot` value to match the address or the IP that you will be typing in the web-browser, EXP: `https://e-learning.masn.edu.eg`, `http://e-learning.masn.edu.eg`, `https://46.58.71.31`, `http://46.58.71.31`. 

> If you use a custom port to access moodle server add it to the `--wwwroot`, EXP: `https://e-learning.masn.edu.eg:9999`, `http://e-learning.masn.edu.eg:9999`, `https://46.58.71.31:9999`, `http://46.58.71.31:9999`.

**Output looks something like this:**
```bash
...
-->factor_totp
++ Success (0.03 seconds) ++
-->factor_webauthn
++ Success (0.03 seconds) ++
-->upgrade_noncore()
++ Success (0.88 seconds) ++
Installation completed successfully.
```

**Take a look at the configs:**
```bash
cat ${MOODLE_CODE_FOLDER}/config.php
```

**Output:**
```bash
<?php  // Moodle configuration file

unset($CFG);
global $CFG;
$CFG = new stdClass();

$CFG->dbtype    = 'mariadb';
$CFG->dblibrary = 'native';
$CFG->dbhost    = 'localhost';
$CFG->dbname    = 'moodle_db';
$CFG->dbuser    = 'moodle_user';
$CFG->dbpass    = 'a15My-lz1wHBXNiVadBIcc7IyujlMQR0gehb3FWuH9U_';
$CFG->prefix    = 'mdl_';
$CFG->dboptions = array (
  'dbpersist' => 0,
  'dbport' => 3306,
  'dbsocket' => '/run/mysqld/mysqld.sock',
  'dbcollation' => 'utf8mb4_unicode_ci',
);

$CFG->wwwroot   = 'http://192.168.1.120';
$CFG->dataroot  = '/var/moodle/moodledata';
$CFG->admin     = 'admin';

$CFG->directorypermissions = 02777;

require_once(__DIR__ . '/lib/setup.php');

// There is no php closing tag in this file,
// it is intentional because it prevents trailing whitespace problems!
```

---
---
# Change Permissions and Ownership:

> Just in case that you used any other user than `www-data` by mistake.

```bash

# Set Permissions:
find ${MOODLE_CODE_FOLDER} -type d -exec chmod 755 {} \;
find ${MOODLE_CODE_FOLDER} -type f -exec chmod 644 {} \;

find ${MOODLE_DATA_FOLDER} -type d -exec chmod 700 {} \;
find ${MOODLE_DATA_FOLDER} -type f -exec chmod 600 {} \;

chmod -R 750 ${COMPOSER_CACHE_FOLDER}

# Change ownership to www-data user and group:
chown -R www-data:www-data ${MOODLE_CODE_FOLDER} ${MOODLE_DATA_PATH}

```

> Will take sometime.

---
---
# Apache Configs:

#### Make sure `apache2` is working: (Optional)

```bash
systemctl status --no-pager apache2
```

> Should be `active (running)`.

**Enable and start apache if needed:**
```bash
systemctl enable --now apache2
```

---
#### Enable required `apache2` modules for `PHP-FPM` and `rewriting`:

```bash
a2enmod proxy_fcgi setenvif rewrite
a2enconf php${PHP_VERSION}-fpm
systemctl restart apache2
```

**Output:**
```bash
systemctl restart apache2
Considering dependency proxy for proxy_fcgi:
Enabling module proxy.
Enabling module proxy_fcgi.
Module setenvif already enabled
Enabling module rewrite.
To activate the new configuration, you need to run:
  systemctl restart apache2
Enabling conf php8.4-fpm.
To activate the new configuration, you need to run:
  systemctl reload apache2
```

---
## Create site configuration file in `/etc/apache2/sites-available`:

### (Option_1) No SSL/TLS -> Not secure (just for testing):

#### Make sure ssl mod is disabled:

```bash
a2dismod ssl
```

**Output:**
```bash
Module ssl already disabled
```

#### Create the actual site configuration file:

> Comment `ServerAlias` if you are going to use IP.

**Just copy past in the terminal:**
```html
tee /etc/apache2/sites-available/${MOODLE_APACHE_CONFIG_FILE} > /dev/null <<EOF
<VirtualHost *:${WEBSITE_PORT}>
    # Domain name without http://
    # EXP: Domain Name: mans.edu.eg or IP: 192.168.1.120
    ServerName ${SERVER_ADDRESS}
    ServerAlias www.${SERVER_ADDRESS}

    DocumentRoot ${MOODLE_APACHE_ROOT_DOCUMENT}
    
    <Directory ${MOODLE_CODE_FOLDER}>
        Options FollowSymLinks
        AllowOverride None
        Require all granted
        DirectoryIndex index.php index.html

        # Enable fallback routing for URLs not matching files/directories
        FallbackResource /r.php
    </Directory>

    # PHP-FPM ${PHP_VERSION} FastCGI handler via socket
    <FilesMatch "\.php$">
        SetHandler "proxy:unix:/run/php/php${PHP_VERSION}-fpm.sock|fcgi://localhost/"
    </FilesMatch>

    # Log files
    ErrorLog ${APACHE_LOGS_FOLDER}/error.log
    CustomLog ${APACHE_LOGS_FOLDER}/access.log combined
    
    # all relevant timeouts 
    Timeout 300
    ProxyTimeout 300

    # FastCGI/PHP-FPM: 
    FcgidIOTimeout 300
    FcgidConnectTimeout 300
    FcgidBusyTimeout 300
</VirtualHost>
EOF
```

**Take a look at the `${MOODLE_APACHE_CONFIG_FILE`} file:**
```bash
cat /etc/apache2/sites-available/${MOODLE_APACHE_CONFIG_FILE}
```

**Output should look something like this:**
```html
<VirtualHost *:80>
    # Domain name without http://
    # EXP: Domain Name: mans.edu.eg or IP: 192.168.1.120
    ServerName 192.168.1.120
    ServerAlias www.192.168.1.120

    DocumentRoot /var/www/html/moodle/public
    
    <Directory /var/www/html/moodle>
        Options FollowSymLinks
        AllowOverride None
        Require all granted
        DirectoryIndex index.php index.html

        # Enable fallback routing for URLs not matching files/directories
        FallbackResource /r.php
    </Directory>

    # PHP-FPM 8.4 FastCGI handler via socket
    <FilesMatch "\.php$">
        SetHandler "proxy:unix:/run/php/php8.4-fpm.sock|fcgi://localhost/"
    </FilesMatch>

    # Log files
    ErrorLog /var/log/apache2/error.log
    CustomLog /var/log/apache2/access.log combined
    
    # all relevant timeouts 
    Timeout 600
    ProxyTimeout 600

    # FastCGI/PHP-FPM: 
    FcgidIOTimeout 600
    FcgidConnectTimeout 600
    FcgidBusyTimeout 600
</VirtualHost>
```
### **End of: (Option_1)**

---
### (Option_2) With SSL/TLS enabled:

[Add Apache SSL Guide Link](https://httpd.apache.org/docs/current/ssl/ssl_howto.html)

> **IMPORTANT**: Make sure `SSLCertificateKeyFile`, `SSLCertificateFile` and `SSLCertificateChainFile` matching certificate files are in their location as mentioned in `~/.env_vars`.
#### **IMPORTANT**: Enable apache2 over ssl: 

```bash
a2enmod ssl
```

#### Create the actual site configuration file:

> Comment `ServerAlias` if you are going to use IP.

> HTTP port:`80` requests are redirected to HTTPS port:`443`.

**Just copy past in the terminal:**
```html
tee /etc/apache2/sites-available/${MOODLE_APACHE_CONFIG_FILE} > /dev/null <<EOF
<VirtualHost *:${WEBSITE_PORT}>
    # Domain name without http://
    # EXP: mans.edu.eg
    ServerName ${SERVER_ADDRESS}
    Redirect / https://${SERVER_ADDRESS}
</VirtualHost>

<VirtualHost *:${WEBSITE_SECURE_PORT}>
    # Domain name without http://
    # EXP: Domain Name: mans.edu.eg or IP: 192.168.1.120
    ServerName ${SERVER_ADDRESS}
    ServerAlias www.${SERVER_ADDRESS}

    DocumentRoot ${MOODLE_APACHE_ROOT_DOCUMENT}
    
    <Directory ${MOODLE_CODE_FOLDER}>
        Options FollowSymLinks
        AllowOverride None
        Require all granted
        DirectoryIndex index.php index.html

        # Enable fallback routing for URLs not matching files/directories
        FallbackResource /r.php
    </Directory>

    # PHP-FPM ${PHP_VERSION} FastCGI handler via socket
    <FilesMatch "\.php$">
        SetHandler "proxy:unix:/run/php/php${PHP_VERSION}-fpm.sock|fcgi://localhost/"
    </FilesMatch>

    # Log files
    ErrorLog ${APACHE_LOGS_FOLDER}/error.log
    CustomLog ${APACHE_LOGS_FOLDER}/access.log combined
    
    # all relevant timeouts 
    Timeout 600
    ProxyTimeout 600

    # FastCGI/PHP-FPM: 
    FcgidIOTimeout 600
    FcgidConnectTimeout 600
    FcgidBusyTimeout 600
    
	SSLEngine on
    # IMPORTANT: Make sure you have these files in these locations.
    SSLCertificateKeyFile ${APACHE_SSL_PRIVATE_KEY}
    SSLCertificateFile ${APACHE_SSL_PUBLIC_KEY}
    SSLCertificateChainFile ${APACHE_SSL_CA_FULL_CHAIN}
</VirtualHost>
EOF
```

**Take a look at the `${MOODLE_APACHE_CONFIG_FILE`} file:**
```bash
cat /etc/apache2/sites-available/${MOODLE_APACHE_CONFIG_FILE}
```

**Output should look something like this:**
```HTML
<VirtualHost *:80>
    # Domain name without http://
    # EXP: mans.edu.eg
    ServerName e-learning.mans.edu.eg
    Redirect / https://e-learning.mans.edu.eg
</VirtualHost>

<VirtualHost *:443>
    # Domain name without http://
    # EXP: Domain Name: mans.edu.eg or IP: 192.168.1.120
    ServerName e-learning.mans.edu.eg
    ServerAlias www.e-learning.mans.edu.eg

    DocumentRoot /var/www/html/moodle/public
    
    <Directory /var/www/html/moodle>
        Options FollowSymLinks
        AllowOverride None
        Require all granted
        DirectoryIndex index.php index.html

        # Enable fallback routing for URLs not matching files/directories
        FallbackResource /r.php
    </Directory>

    # PHP-FPM 8.4 FastCGI handler via socket
    <FilesMatch "\.php$">
        SetHandler "proxy:unix:/run/php/php8.4-fpm.sock|fcgi://localhost/"
    </FilesMatch>

    # Log files
    ErrorLog /var/log/apache2/error.log
    CustomLog /var/log/apache2/access.log combined
    
    # all relevant timeouts 
    Timeout 600
    ProxyTimeout 600

    # FastCGI/PHP-FPM: 
    FcgidIOTimeout 600
    FcgidConnectTimeout 600
    FcgidBusyTimeout 600
    
    SSLEngine on
    # IMPORTANT: Make sure you have these files in these locations.
    SSLCertificateKeyFile /etc/ssl/private/private_key.key
    SSLCertificateFile /etc/ssl/certs/public_key.crt
</VirtualHost>
```

> `SSLCertificateFile /etc/ssl/certs/public_key.crt` should include the full chain too.
### **End of: (Option_2)**

---
#### Enable `${MOODLE_APACHE_CONFIG_FILE}` website and disable `000-default.conf`:

```bash
# Enable the new Moodle site and disable the default site if desired
a2ensite ${MOODLE_APACHE_CONFIG_FILE}
a2dissite 000-default.conf
# Restart Apache to apply all changes
systemctl restart apache2
```

#### Remove default page: (Optional)

```bash
rm /var/www/html/index.html
```

---
#### Check that configurations are correct: (Optional)

```bash
sudo apache2ctl configtest
```

> Should be with no warnings. 

**Output:**
```output
Syntax OK
```

---
---
# Test the website:

> EXP: http://192.168.1.120

> In your **browser** type the **url** or **IP** you used in `--wwwroot="${PROTOCOL}${SERVER_ADDRESS}"` -> **EXP**: `https://e-learning.masn.edu.eg`, `http://e-learning.masn.edu.eg`, `https://46.58.71.31`, `http://46.58.71.31`, `https://e-learning.masn.edu.eg:9999`, `http://e-learning.masn.edu.eg:9999`, `https://46.58.71.31:9999`, `http://46.58.71.31:9999`. 

> For some reason login doesn't work from the first time.

![](./assets/Pasted%20image%2020260617214415.png)


![](./assets/Pasted%20image%2020260617112340.png)


![](./assets/Pasted%20image%2020260617214835.png)

---
---
# Redis Cache:

**Edit `redis-server` Configs:**
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

maxclients 50000
```

> Replace `<STRONG_PASSWORD>`.

**Validate configs:**
```bash
redis-server /etc/redis/redis.conf
```

> If output is empty then you are doing fine.

#### Restart Redis to take effect:

```bash
systemctl restart redis
```

```bash
systemctl status redis
```

> Should be `active (running)`.

#### Add `www-data` user to group `redis`:

> So that `www-data` can access redis throw its unix socket.

```bash
usermod -aG redis www-data
```

> `PHP-FPM` is running with user `www-data` -> we are giving it access to the `redis` `unix socket` that has permission of `770` -> `u:rwx, g:rwx, o:---`.

#### Test connection for `www-data`:

**`/run/redis/redis-server.sock`:**
```bash
sudo -u www-data redis-cli -s ${REDIS_SOCKET}
```

**Output:**
```bash
redis /run/redis/redis-server.sock>
```

#### Test password:

**Command is `auth <STRONG_PASSWORD>`:**
```bash
redis /run/redis/redis-server.sock> auth <STRONG_PASSWORD>
```

> Replace `<STRONG_PASSWORD>` with the password you already set in `/etc/redis/redis.conf` -> `requirepass <STRONG_PASSWORD>`.

**Output:**
```bash

OK
```

#### Show keys: (after test password auth step)

> This step won't show keys only after the next step which is integrating `redis` with `Moodle`.

**Command is `KEYS *`:**
```bash
redis /run/redis/redis-server.sock> KEYS * 
```

**Output should look something like this:**
```bash
1) "mdl_php_session_588f8096d6b01ba6dd8e288b0c742198"
 2) "mdl_muc_38fe949f8d2b52b4bb84994b3f90c15e"
 3) "mdl_muc_50fe7cfcb51cf3b8be5c56a9b0be6c7c"
 4) "mdl_muc_929bec7dd951df9aa832e8b1fece960a"
 5) "mdl_muc_999eeb29a680d85c9f4d0d8839a3156e"
 6) "mdl_muc_333b05a93763134d16ff7006d711b5aa"
 7) "mdl_muc_1b088c226629b3fa247ff7ef67dad266"
 8) "mdl_muc_8f371215eb636856fc0cb92b3526ed93"
 9) "mdl_muc_ed7fa70b0d7a330a6aadaebe7bba1cf1"
10) "mdl_php_user_2"
11) "mdl_muc_26bb96d1df16a1c8b3326075d0d80fc3"
12) "mdl_muc_d7dbf14ad7624c3afae5ebefa15246d8"
13) "mdl_muc_e76d7d94612882280115adcdc37b14c2"
14) "mdl_muc_f63f2a7821f91e1c6276c15d9449e1f7"
15) "mdl_muc_3680502b1870f722155ddf6bae6505f1"
16) "mdl_muc_92c18184d874d4bbcebbf87940d76b8b"
17) "mdl_muc_5924e41929987a52d335b6d11191f849"
18) "mdl_muc_91c7f8386233fca3d945441025a315ab"
19) "mdl_muc_b5d07a007fad4fe35f784d9797cf300f"
20) "mdl_muc_50fe7cfcb51cf3b8be5c56a9b0be6c7c_ttl"
21) "mdl_muc_346f17378d0993bb74e0fb3a5000fdd0"
22) "mdl_muc_f63f2a7821f91e1c6276c15d9449e1f7_ttl"
23) "mdl_muc_e76d7d94612882280115adcdc37b14c2_ttl"
24) "mdl_muc_28ef3342b805bd623882d4360dedd27c"
25) "mdl_muc_6962e25b7883aa8af0008da2082a3935"
26) "mdl_muc_7c480f7a77f4ddab97dc34862bd5b524"

```

#### Exit from redis-cli:

**Command is `exit`:**
```bash
redis /run/redis/redis-server.sock> exit
```

### Integrating `redis` with `Moodle`:
#### Use redis for caching sessions:

```bash
vim ${MOODLE_CODE_FOLDER}/config.php
```

> **IMPORTANT:** Add these before `require_once(__DIR__ . '/lib/setup.php');`.

**Add the following as shown in the image:**
```bash

// Redis session handler:
$CFG->session_handler_class = '\core\session\redis';
$CFG->session_redis_host = '/run/redis/redis-server.sock';
$CFG->session_redis_prefix = 'mdl_php_';
$CFG->session_redis_port = 0; # Must be 0 to endable sockets and disable TCP.
$CFG->session_redis_serializer_use_igbinary = true;
$CFG->session_redis_auth = '<STRONG_PASSWORD>';  # your requirepass in redis.conf

```

> Replace `<STRONG_PASSWORD>` -> you have already added it in redis.conf -> `requirepass <STRONG_PASSWORD>`.

**Example snapshot:** take a look at the red and yellow squares.
![](./assets/Pasted%20image%2020260624115452.png)

**Your file should look something like this:**
```bash
<?php  // Moodle configuration file

unset($CFG);
global $CFG;
$CFG = new stdClass();

$CFG->dbtype    = 'mariadb';
$CFG->dblibrary = 'native';
$CFG->dbhost    = 'localhost';
$CFG->dbname    = 'moodle_db';
$CFG->dbuser    = 'moodle_user';
$CFG->dbpass    = 'a15My-lz1wH46878adBIcc7IyujlMQR0gehb3FWuH9U_';
$CFG->prefix    = 'mdl_';
$CFG->dboptions = array (
  'dbpersist' => 0,
  'dbport' => 3306,
  'dbsocket' => '/run/mysqld/mysqld.sock',
  'dbcollation' => 'utf8mb4_unicode_ci',
);


# EXP: $CFG->wwwroot   = 'http://my-moodle.com';
# EXP: $CFG->wwwroot   = 'https://my-moodle.com';
# EXP: $CFG->wwwroot   = 'http://IP';
$CFG->wwwroot   = 'http://192.168.1.120';
$CFG->dataroot  = '/srv/moodle/moodledata';
$CFG->admin     = 'admin';

$CFG->directorypermissions = 02777;

// Redis session handler:
$CFG->session_handler_class = '\core\session\redis';
$CFG->session_redis_host = '/run/redis/redis-server.sock';
$CFG->session_redis_prefix = 'mdl_php_';
$CFG->session_redis_port = 0; # Must be 0 to endable sockets and disable TCP.
$CFG->session_redis_serializer_use_igbinary = true;
$CFG->session_redis_auth = 'a15My-lz1wHBXNiVa4892c7R0gehb3FWuH9UIyujlMQ_';  # your requirepass in redis.conf

require_once(__DIR__ . '/lib/setup.php');

// There is no php closing tag in this file,
// it is intentional because it prevents trailing whitespace problems!

```
#### IMPORTANT: Delete sessions on the `moodledata` directory:

```bash
rm -rf ${MOODLE_DATA_FOLDER}/sessions/*
```

> You might run into errors like moodle thinking that you are doing session forging if your don't delete sessions in `${MOODLE_DATA_FOLDER}/sessions/`.

> Deleting the files in `${MOODLE_DATA_FOLDER}/sessions` will also help you with the next step that will tell you if sessions have been moved to redis or not.

#### Restart  `php${PHP_VERSION}-fpm.service`:(Optional)

```bash
systemctl restart redis php${PHP_VERSION}-fpm.service
```

#### Make sure redis is being used for sessions:

**Login with any account on moodle:**
![](./assets/Pasted%20image%2020260617214415.png)

#### Check if there are any files in `${MOODLE_DATA_FOLDER}/sessions`:

```bash
ls -al ${MOODLE_DATA_FOLDER}/sessions
```

**Output should look something like this:** Empty.
```bash
drwxrwsrwx  2 www-data www-data 4096 Jun 23 13:35 .
drwxrwsrwx 13 www-data www-data 4096 Jun 22 17:00 ..
```

> You shouldn't see any files here while some users are being logged in, otherwise redis is not being used for sessions.

#### Use redis for Moodle Universal Cache(MUC):

> **Hint**: MUC configs are stored in `${MOODLE_DATA_FOLDER}/muc/config.php`.

> On the website: Go to `Site administration` -> `Plugins` -> `Caching` -> `Configuration` -> Under `Cache administration` `Installed cache stores` -> `Redis` -> Click `Add Instance`

![](./assets/Pasted%20image%2020260624123712-cut.png)

**Table of values:**

| Key            | Value                                                                |
| -------------- | -------------------------------------------------------------------- |
| Server(s)      | `/run/redis/redis-server.sock`                                       |
| Store name     | `Redis-MUC-Cache`                                                    |
| Password       | password you already set in `/etc/redis/redis.conf` -> `requirepass` |
| key prefix     | `mdl_muc_`                                                           |
| Use serializer | `Igbinary`                                                           |
**Add the values as mentioned in the table above:**
![](./assets/Pasted%20image%2020260624123549.png)

> Then scroll down and click `Save`.

**Page should look like this:**
![](./assets/Pasted%20image%2020260624123712.png)

> Scroll all the way down in  `Site administration` -> `Plugins` -> `Caching` -> `Configuration`

> Click `Edit mappings` -> Change it to the following.

**Before:**
![](./assets/Pasted%20image%2020260624124039.png)

**Before:**
![](./assets/Pasted%20image%2020260624124344.png)

**After:**
![](./assets/Pasted%20image%2020260624124246.png)

> Click `Save changes`.

**This is how it is supposed to look after you save and scroll all the way down:**
![](./assets/Pasted%20image%2020260624124622.png)

#### Add redis to `Test servers` for testing load:

> On the website: Go to `Site administration` -> `Plugins` -> `Caching` -> `Redis`.

**Add the following credentials then scroll down and click `Save changes`:**

| Key                  | Value                                                                |
| -------------------- | -------------------------------------------------------------------- |
| Test server          | `/run/redis/redis-server.sock`                                       |
| Test server password | password you already set in `/etc/redis/redis.conf` -> `requirepass` |

**Snapshot:**
![](./assets/Pasted%20image%2020260624122524.png)

#### Test Performance:

> On the website: Go to `Site administration` -> `Plugins` -> `Caching` -> `Test performance`

**`Redis` -> `Tested`:**
![](./assets/Pasted%20image%2020260624123304.png)

---
---
# Clamav antivirus:

[Clamav Explained Video](https://www.youtube.com/watch?v=cOikRy7P1_g)
#### Update viruses database:

```bash
systemctl stop clamav-freshclam
```

```bash
freshclam
```

```bash
systemctl enable --now clamav-freshclam
```

#### Manual Scan(Optional):

```bash
clamscan -r -i --log=~/clamav-scan-moodle-code.log <PATH>
```

> `~/clamav-scan-moodle-code.log`: any path and name.
#### Enable `Clamav` for `Moodle`:

> `Site Administration` -> `Plugins` -> `Antivirus plugins` -> `Manage antivirus plugins`

> Click on the eye icon under `Enable`.

**Before:**
![](./assets/Pasted%20image%2020260804115450.png)

**After:**
![](./assets/Pasted%20image%2020260804115515.png)

> Scroll down and click `Save changes`.

---
---
# See what is happening on the server:

> `Site administration` -> `Reports` -> `Live Logs`

---
---
# **Machine Firewall Configs:**

```bash
ufw allow 80 comment "http for everybody"
```

```bash
ufw allow 443 comment "https for everybody"
```

**If your device has dynamic IP:** more common if you are using `VPS`.
```bash
ufw limit <SSH_PORT>/tcp comment "SSH PORT Limited."
```

**If your device has static IP:** more common if your setup is `on-premises`.
```bash
ufw allow from <YOUR_IP> to any port <SSH_PORT> comment "SSH PORT For Ahmed Fawzy"
```

> Replace `<YOUR_IP>` with the IP of the machine you are using to ssh into the server.

> Replace `<SSH_PORT>`.

---
#### **Enable firewall:**

```bash
ufw enable
```

> Type `y`.

**Output:**
```bash
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

---
#### Show ufw rules numbered:

```bash
ufw status numbered
```

**Should look something like this:**
```bash
Status: active

[ 1] 80                 ALLOW IN    Anywhere         # http to everybody
[ 2] 443                ALLOW IN    Anywhere         # https to everybody
[ 3] 5348/tcp           ALLOW IN    192.168.1.16     # SSH PORT For Ahmed Fawzy
```

> Custom SSH port.

---
#### Delete a rule: (Optional)

```bash
ufw delete <NUMBER>
```

> Type `y`.

---
---
# Helping Commands:

**Upgrade Moodle site's database schema and plugins:** 
```bash
php${PHP_VERSION} ${MOODLE_CODE_FOLDER}/admin/cli/upgrade.php
```

> Helpful during migration and upgrade.

**Purge caches command:**
```bash
php${PHP_VERSION} ${MOODLE_CODE_FOLDER}/admin/cli/purge_caches.php
```

> Helpful when you can't access the website `purge all caches`.

---

**Get database size:**
```bash
# Get databse size in memory:
mariadb -e "SELECT ROUND(SUM(data_length + index_length) / 1024 / 1024 / 1024, 2) AS total_size_gb \
FROM information_schema.tables \
WHERE engine = 'InnoDB';"
```

```bash
mariadb -e "SELECT ROUND(SUM(data_length) / 1024 / 1024 / 1024, 2) AS total_size_gb \
FROM information_schema.tables \
WHERE engine = 'InnoDB';"
```

---

**Get database configs:**
```bash
mariadb -e "SHOW VARIABLES LIKE 'innodb_buffer_pool_size';"
```

---

**Get ldap info:**
```bash
mariadb -e "SELECT * FROM mdl_config_plugins WHERE plugin = 'auth_ldap';"
```

---

**Convert Manually added users to ldap authentication:**

**Except for `admin` users:** -> `AND id>2;`
```sql
mariadb -e "UPDATE mdl_user SET auth='ldap' WHERE auth='manual' AND id>2;"
```

**Make `admin` user manual:** If you converted the admin to ldap by mistake.
```sql
mariadb -e "UPDATE mdl_user SET auth='manual' WHERE username = 'admin';"
```

**Count users  who are logging manually:**
```sql
mariadb -e "select count(*) from mdl_user where auth='manual';"
```

# Extras:


**Edit/Enable/Disable upload:**
```bash
MAX_INPUT_VARS="5000"
POST_MAX="500M"
UPLOAD_MAX="500M"
FILE_UPLOADS="On"
php_ini_fpm="/etc/php/8.4/fpm/php.ini"
php_ini_cli="/etc/php/8.4/cli/php.ini"

# For /etc/php/8.3/fpm/php.ini
sudo sed -i "s/^max_input_vars = .*/max_input_vars = ${MAX_INPUT_VARS}/" "$php_ini_fpm"
sudo sed -i "s/^post_max_size = .*/post_max_size = ${POST_MAX}/" "$php_ini_fpm"
sudo sed -i "s/^file_uploads = .*/file_uploads = ${FILE_UPLOADS}/" "$php_ini_fpm"
sudo sed -i "s/^upload_max_filesize = .*/upload_max_filesize = ${UPLOAD_MAX}/" "$php_ini_fpm"


# For /etc/php/8.3/cli/php.ini
sudo sed -i "s/^max_input_vars = .*/max_input_vars = ${MAX_INPUT_VARS}/" "$php_ini_cli"
sudo sed -i "s/^post_max_size = .*/post_max_size = ${POST_MAX}/" "$php_ini_cli"
sudo sed -i "s/^file_uploads = .*/file_uploads = ${FILE_UPLOADS}/" "$php_ini_cli"
sudo sed -i "s/^upload_max_filesize = .*/upload_max_filesize = ${UPLOAD_MAX}/" "$php_ini_cli"

systemctl restart apache2.service php8.3-fpm.service
```

**Disable uploading files:**
```bash
FILE_UPLOADS="On"
php_ini_fpm="/etc/php/8.4/fpm/php.ini"
php_ini_cli="/etc/php/8.4/cli/php.ini"

sudo sed -i "s/^file_uploads = .*/file_uploads = ${FILE_UPLOADS}/" "$php_ini_fpm"
sudo sed -i "s/^file_uploads = .*/file_uploads = ${FILE_UPLOADS}/" "$php_ini_cli"
```