# MariaDB - Basic informations and SQL statements

```sql title="Grant root access via socket (no password) and default DB connection with password"
GRANT ALL PRIVILEGES ON *.* TO `root`@`localhost` IDENTIFIED VIA mysql_native_password USING PASSWORD ('my_secret_password') OR unix_socket WITH GRANT OPTION;
```
