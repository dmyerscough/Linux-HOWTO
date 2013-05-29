Recovering files from /proc
===========================

A while back, we encountered a hacker who deleted the Apache log files to hide his tracks. However, knowing that the Apache process had not been restarted, I was able to recover the file from the /proc filesystem.

In order to recover deleted files from a process that is currently running, you need to get the PID (Process ID) that is writing the file. In the example shown below, we get the parent PID of the Apache web server since the parent PID writes the Apache web server logs.

```bash
# ps aux | grep -i httpd
root      4417  0.2  1.2 174376  3204 ?        Ss   19:24   0:00 /usr/sbin/httpd
apache    4419  0.0  0.8 174508  2204 ?        S    19:24   0:00 /usr/sbin/httpd
apache    4420  0.0  0.8 174508  2204 ?        S    19:24   0:00 /usr/sbin/httpd
…
…
```

The parent PID for the Apache web server is 4417, and from this you can navigate to the /proc filesystem and retrieve the deleted file as shown below.

```bash
# cd /proc/4417/fd
 
# ls -al
l-wx------ 1 root root 64 Jul 17 19:24 2 -> /var/log/httpd/error_log (deleted)
```

The `/proc/[PID]/fd` directory contains all the files that the program has opened and not released. The above output shows that the file “2″ is a symbolic link to the deleted Apache logs. To recover the file, you can do a simple copy to a destination where you would like the file restored to.

```bash
# cp 2 ~/error_log
```
