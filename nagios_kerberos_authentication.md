Installing mod_auth_kerb
========================

Install the Apache module to provide Kerberos authentication by running the following command:

```bash
# yum install mod_auth_kerb
```

After installing the module, check that it is loaded into Apache by listing all the modules on the Apache web server:

```bash
# httpd -D DUMP_MODULES 2>&1 | grep -i kerb
 auth_kerb_module (shared)
```

When you install the `mod_auth_kerb` module using the `yum` command, you should automatically have a configuration file placed in `/etc/httpd/conf.d/auth_kerb.conf`. This configuration file will automatically cause the module to be loaded and provide a sample configuration for configuring `mod_auth_kerb`.

Configuring Nagios
==================

I will assume you already have Nagios setup and running, I do plan to write future guides on Nagios but for the time being you will need to do this on your own. If you installed Nagios using the CentOS or Red Hat RPM’s, a configuration file is already generated (`/etc/httpd/conf.d/nagios.conf`) within this configuration file, and you will need to add the following directives:

	ScriptAlias /nagios/cgi-bin/ "/usr/lib64/nagios/cgi-bin/"
	
	<Directory "/usr/lib64/nagios/cgi-bin/">
		Options ExecCGI
		AllowOverride None
		Order allow,deny
		Allow from all
	
		AuthName             "Kerberos Authentication"
		AuthType             Kerberos
		Krb5KeyTab           /etc/krb5.keytab
		KrbAuthRealms        UNSUPPORTED.COM
		KrbMethodNegotiate   On
		KrbMethodK5Passwd    On
		require              valid-user
	
		Alias /nagios "/usr/share/nagios/html"
	
	</Directory> 

	<Directory "/usr/share/nagios/html">
		Options None
		AllowOverride None
		Order allow,deny
		Allow from all
	
		AuthName             "Kerberos Authentication"
		AuthType             Kerberos
		Krb5KeyTab           /etc/krb5.keytab
		KrbAuthRealms        UNSUPPORTED.COM
		KrbMethodNegotiate   On
		KrbMethodK5Passwd    On
		require              valid-user
	</Directory>

The key directives that you need to modify here are the `Krb5KeyTab` which you generated on your Kerberos server, and `KrbAuthRealms` which should be the Kerberos realm you have configured in your environment. The Kerberos keytab that you specify must be readable by the Apache webserver; otherwise, you will hit permission errors similar to the following:

	[Sun Nov 11 21:32:29 2012] [error] [client 192.168.84.1] krb5_get_init_creds_password() failed: Decrypt integrity check failed
	[Sun Nov 11 21:32:34 2012] [error] [client 192.168.84.1] failed to verify krb5 credentials: Permission denied
	[Sun Nov 11 21:36:25 2012] [error] [client 192.168.84.1] failed to verify krb5 credentials: Permission denied
	[Sun Nov 11 21:36:38 2012] [error] [client 192.168.84.1] failed to verify krb5 credentials: Permission denied

Once you have configured Nagios to use Kerberos authentication, your users will be permitted to login to Nagios. However, your users will not be permitted to manage any of the services/alerts or view the “Process Info” tab. To enable your Nagios users to perform these tasks, add them to the cgi.cfg configuration file by specifying the user followed by the realm they are in:

	authorized_for_all_service_commands=nagiosadmin,jason@UNSUPPORTED.COM
	authorized_for_all_host_commands=nagiosadmin,jason@UNSUPPORTED.COM

When you have added the user to the Nagios cgi.cfg configuration file, you will not need to reload Apache or the Nagios process, and you should now have Nagios successfully authenticating against Kerberos.
