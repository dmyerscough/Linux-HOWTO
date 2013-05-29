MySQL Reset Root Password
=========================

I have been asked on many occasions "How can I reset the root users password on MySQL?" and the answer to this question is simple. For the purpose of this article, we’ll take a look at how to do this in CentOS. Although there are many ways to reset the root password in MySQL, a common technique is to skip the grants table with the --skip-grants-table option. But this poses a slight security risk, even when used with --skip-networking as a local user, which can connect to MySQL without a password and expose your data.

Resetting the password securely
===============================
A secure way to reset the MySQL root password is to create an initialization script that contains a SQL statement to reset the root password. The following SQL statement resets the MySQL root password and make MySQL re-read the grants table:

```sql
UPDATE mysql.user SET Password = PASSWORD ("My_New_Password") WHERE User = "root";
FLUSH PRIVILEGES;
```

Once you have created the initialization script, start MySQL with the --init-file option:

```sql
# /usr/libexec/mysqld --user=mysql --init-file=/var/lib/mysql/resetRoot.sql
100718 19:42:06 [Note] /usr/libexec/mysqld: ready for connections.
Version: '5.0.77'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  Source distribution
```

Once MySQL has started, the initialization script runs and resets the root password. After the root password has been reset, you can stop the MySQL daemon by opening another terminal or making a new connection by issuing:

```bash
# mysqladmin shutdown -u root -p
Enter password:
```

When you are prompted for the root password, you can enter the root password from the initialization file and MySQL will be shutdown gracefully.  After it shuts down, you can start the MySQL daemon in the traditional way via the initialization script:

```bash
# service mysqld start
# /etc/init.d/mysqld start
```
