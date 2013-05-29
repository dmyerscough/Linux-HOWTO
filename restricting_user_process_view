Restricting users process view
==============================

I have always been interested in security and like to see how I can improve the security on Linux systems. As of kernel 3.2.20 [1], you can now restrict users to view only their own processes and no others on the system. You also have the ability to permit specific groups access to view all processes, which can be handy for system administrators. The restriction feature has already been pushed into the kernel for Red Hat Enterprise Linux 6.3 and CentOS.

To restrict users from viewing other processes, you will need to remount the /proc filesystem by issuing the following command:

```bash
# mount -o rw,hidepid=2,remount /proc
```

 - zero (0) sets the /proc filesystem to the default configuration, which allows users to view all processes on the system.
 - one (1) restricts users from accessing the /proc/[PID] directories that they do not own, but they can access their own.
 - two (2) restricts the users from viewing all PIDs, including when the users navigates to the /proc filesystem.

If you would like these settings to persist over a reboot, you will need to modify the /etc/fstab file as shown below.

	proc                    /proc                   proc    defaults,hidepid=2        0 0

With the hidepid option, you can also add the gid option to specify a group ID that is permitted to view all process on the system regardless of the hidepid option. The gid option is handy if you have multiple system administrators on the system and you want to permit them to view all processes on the system for troubleshooting. The example shown below permits one group to view all processes and restrict users that are not in the group.

```bash
# mount -o rw,hidepid=2,gid=700,remount /proc
```
