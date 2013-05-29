Configuring Google Authenticator
================================

Download and install Google Authenticator using the `yum` command. The Google Authenticator package can be retrieved from the EPEL YUM repository:

```bash
$ google-authenticator 
https://www.google.com/chart?chs=200x200&amp;chld=M|0&amp;cht=qr&amp;chl=otpauth://totp/damian@kdc.sfdc.net%3Fsecret%3DPYC5HWHYIL24GGOL
Your new secret key is: PYC5HWHYIL24GGOL
Your verification code is 569556
Your emergency scratch codes are:
  88831003
  40953378
  67507388
  81171796
  28823080
 
Do you want me to update your "~/.google_authenticator" file (y/n) y
 
Do you want to disallow multiple uses of the same authentication
token? This restricts you to one login about every 30s, but it increases
your chances to notice or even prevent man-in-the-middle attacks (y/n) y
 
By default, tokens are good for 30 seconds and in order to compensate for
possible time-skew between the client and the server, we allow an extra
token before and after the current time. If you experience problems with poor
time synchronization, you can increase the window from its default
size of 1:30min to about 4min. Do you want to do so (y/n) y
 
If the computer that you are logging into isn't hardened against brute-force
login attempts, you can enable rate-limiting for the authentication module.
By default, this limits attackers to no more than 3 login attempts every 30s.
Do you want to enable rate-limiting (y/n) y
```

To retrieve the QR code with the provided Google URL, first download the Google Authenticator application to either your iPhone, Android or Blackberry to capture the QR code, which can then be used to generate authentication tokens.

Configure OpenSSH Daemon
========================

Modify the /etc/pam.d/sshd file to work with the authentication tokens:

```bash
# cat /etc/pam.d/sshd
#%PAM-1.0
auth       required     pam_sepermit.so
auth       include      password-auth
 
auth required pam_google_authenticator.so
 
account    required     pam_nologin.so
account    include      password-auth
password   include      password-auth
```

When you have updated the `/etc/pam.d/sshd` configuration file, modify your global OpenSSH configuration file to permit challenge response authentication. The OpenSSH configuration file needs the following directive added to the `/etc/ssh/sshd_config` file:

	ChallengeResponseAuthentication yes

Once you have enabled the challenge response authentication, you can SSH into your server and you will be asked for a Google authentication token and the password:

```bash
$ ssh damian@192.168.0.12
Verification code: 
Password:
```

Now you have two factor authentication configured on the OpenSSH daemon, you can easily adjust this for other services such as OpenVPN.
