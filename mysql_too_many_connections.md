MySQL Error: Too Many Connections
=================================

Recently, I had a customer calling about a problem connecting to the MySQL service running on his server. The error message he got was “Error: Too many connections“, which means that the max_connections variables has been reached. Without disrupting the MySQL service, the only solution was to reset max_connections while the MySQL service was running.

Solution
========
Before you can reset the maximum connection, install the `GDB` utility:

```bash
# yum install gdb
```

Once you have installed the `GDB` utility, issue the following command to set the max_connections variable to 500 (or a number of your choice):

```bash
gdb -p $(cat /var/run/mysqld/mysqld.pid) -ex "set max_connections=500" -batch
```

The max_connections variable is now set to 500, thus allowing more users to connect.

I would like to thank my colleague Marcin for providing me with this solution, I hope you find this helpful.
