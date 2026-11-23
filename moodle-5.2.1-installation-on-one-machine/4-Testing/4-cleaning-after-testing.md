
> **WARNING**: Please Please be extra careful not to delete the main DB and take backups before you do this.

---

**Drop the test database:**
```bash
mariadb -e "DROP database test_db;"
mariadb -e "DROP USER 'test_user'@'localhost';"
```

> Assuming your test database name is `test_db` and test user is `test_user`.

---

**Disable the test web-pages:**
```bash 
a2dissite testing-moodle.mans.edu.eg.conf
a2dissite testing-moodle-result.mans.edu.eg.conf

# Restart Apache to apply all changes or reload it.
systemctl restart apache2
```

> Assuming the file names are `testing-moodle.mans.edu.eg.conf` and `testing-moodle-result.mans.edu.eg.conf`.