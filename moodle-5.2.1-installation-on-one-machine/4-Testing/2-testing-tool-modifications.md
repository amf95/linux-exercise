

#### In `before_run_setup.sh`:

> Modified condition so that it will accept `mariadb` option in `webserver_config.properties`.

**In `Database backup.` section:**
```bash
elif [ "$dbtype" == "mysqli" ]; then
```

```bash
# Added mariadb in the condition.
elif [ "$dbtype" == "mysqli" ] || [ "$dbtype" == "mariadb" ]; then
```

---

> **Error**: Can't reach website full name.

> Caused by two things:
> 1-You should have two domains one for testing Moodle(testing-moodle.mans.edu.eg) and another one for the result(testing-moodle-result.mans.edu.eg).

> 2- Testing script not following redirects. 

**Before:**
```bash
${curlcmd} --silent "$siteindex" | \
```

**Added `-L`:**
```bash
${curlcmd} --silent -L "$siteindex" | \
```

---

> In Generate courses section.

> Error: Could not open input file: admin/tool/generator/cli/maketestsite.php

> Error: The test site can not be generated, check that the site is correctly installed

**Relpace:**
```bash
testplancommand=${phpcmd}' admin/tool/generator/cli/maketestplan.php \
```

> Added `public` to the path.

**With:**
```bash
testplancommand=${phpcmd}' public/admin/tool/generator/cli/maketestplan.php \
```

---

> Error: mariadb-dump: Got error: 1044: "Access denied for user 'moodle_test_user'@'localhost' to database 'moodle_test_db'" when using LOCK TABLES

**Solved by granting lock and permission to the moodle_test_user:**
```sql
mariadb -e "GRANT LOCK TABLES ON moodle_test_db.* TO 'moodle_test_user'@'localhost';"
mariadb -e "FLUSH PRIVILEGES;"
```

---

> Commented `checkout_branch $beforebranchrepository 'before' $beforebranch` -> That Stopped git from switching to different version.

> Stopped --allow-unstable so that it won't use dev versions.

**Before:**
```bash
# Upgrading moodle, although we are not sure that base and before branch are different.
echo "Checking out Moodle from repo: $beforebranchrepository, ref: $beforebranch"
checkout_branch $beforebranchrepository 'before' $beforebranch
echo "Upgrading Moodle ($basecommit) to $(git rev-parse before/$beforebranch)"
${phpcmd} admin/cli/upgrade.php \
    --non-interactive \
    --allow-unstable \
    > /dev/null || \
    throw_error "Moodle can not be upgraded to $beforebranch"
```

**After:**
```bash
# Upgrading moodle, although we are not sure that base and before branch are different.
echo "Checking out Moodle from repo: $beforebranchrepository, ref: $beforebranch"
#checkout_branch $beforebranchrepository 'before' $beforebranch
#echo "Upgrading Moodle ($basecommit) to $(git rev-parse before/$beforebranch)"
${phpcmd} admin/cli/upgrade.php \
    --non-interactive \
#    --allow-unstable \
    > /dev/null || \
    throw_error "Moodle can not be upgraded to $beforebranch"
```

---
#### In  `defaults.properties`:

> Use `mariadb` and `mariadb-dump`.

**Before:**
```bash
mysqlcmd='mysql'
mysqldumpcmd='mysqldump'
```

**After:**
```bash
mysqlcmd='mariadb'
mysqldumpcmd='mariadb-dump'
```

---
#### In `set_moodle_site.php`:

> **Error**: `/moodle/lib/adminlib.php` doesn't exist error.

**Before:**
```php
require_once(__DIR__ . '/moodle/lib/adminlib.php');
```

> Added `public` to the path.

**After:**
```php
require_once(__DIR__ . '/moodle/public/lib/adminlib.php');
```

---
#### In `lib/lib.sh`:

> **Error**: save_moodle_site_data() should only be called after cd to moodle/.

```bash
if [ ! -f "version.php" ]; then
```

> Added `public` to the path.

**After:**
```bash
if [ ! -f "public/version.php" ]; then
```

---
#### In `after_run_setup.sh`:

>  Allowed the script to use `mariadb`.

```bash
elif [ "$dbtype" == "mysqli" ]; then
```

**After:**
```bash
elif [ "$dbtype" == "mysqli" ] || [ "$dbtype" == "mariadb" ]; then
```

---
#### In `test_runner.sh`:

> **Error**: Script was exiting on jmeter warnings.

**Before:**
```bash
for errorkey in ERROR WARN; do
```

> Removed `WARN`.

**After:**
```bash
for errorkey in ERROR; do
```

---