<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
fufix is now known as mailcow!

![mailcow](https://www.debinux.de/256.png)

- [mailcow](#mailcow)
- [Introduction](#introduction)
- [Before You Begin](#before-you-begin)
- [Installation](#installation)
- [Upgrade](#upgrade)
- [SSL certificate](#ssl-certificate)
- [Fetch mail with getmail4](#getmail4)
- [Uninstall](#uninstall)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

mailcow
=====

mailcow is a mail server suite based on Dovecot, Postfix and other open source software, that provides a modern Web UI for administration.
In future versions mailcow will provide Cal- and CardDAV support.

For **Debian and Debian based distributions**. 

This script is permanently **tested on Debian stable (8.x)**.
Debian Squeeze is not supported.

Ubuntu 14.04 is fine but not recommended.

**Please see this album on imgur.com for screenshots -> http://imgur.com/a/x5IST#0**

# Introduction
A summary of what software is installed with which features enabled.

**General setup**
* System environment adjustments (hostname, timzone, etc.)
* Automatically generated passwords with high complexity
* Self-signed SSL certificate for all installed and supporting services
* Nginx or Apache2 installation (+PHP5-FPM)
* MySQL or MariaDB database backend, remote database support
* DNS-Checks (PTR, A-Record, SPF etc.)
* Learn ham and spam, [Heinlein Support](https://www.heinlein-support.de/) SA rules included
* Fail2ban brute force protection
* A "mailcow control center" via browser: Add domains, mailboxes, aliases and more
* Tagged mail like "username+tag@domain.tld" will be moved to folder "tag"
* Advanced ClamAV filters (ClamAV can be turned off, quarantined items can be downloaded)

**Postfix**
* Postscreen activated
* Submission port (587/TCP), TLS-only
* SMTPS (465/TCP)
* The default restrictions used are a good compromise between blocking spam and avoiding false-positives
* Change recipient restrictions in control center
* Blacklist senders in control center
* Incoming and outgoing spam protection
* VirusTotal Uploader for incoming mail
* SSL based on BetterCrypto
* OpenDKIM, manage signatures in control center

**Dovecot**
* Default mailboxes to subscribe to automatically (Inbox, Sent, Drafts, Trash, Junk, Archive - "SPECIAL-USE" tags)
* Sieve/ManageSieve
* Public folder support via control center
* per-user ACL
* Shared Namespace (per-user seen-flag)
* Global sieve filter: Move mail marked as spam into "Junk"
* (IMAP) Quotas
* LMTP service for Postfix virtual transport
* SSL based on BetterCrypto

**Roundcube**
* ManageSieve support (w/ vacation)
* Users can change password
* Attachment reminder (multiple locales)
* Zip-download marked messages

# Before You Begin
- **Please remove any web- and mail services** running on your server. I recommend using a clean Debian minimal installation.
Remember to purge Debians default MTA Exim4:
```
apt-get purge exim4*
``` 

- If there is any firewall, unblock the following ports for incoming connections:

| Service               | Protocol | Port |
| -------------------   |:--------:|:-----|
| Postfix Submission    | TCP      | 587  |
| Postfix SMTPS         | TCP      | 465  |
| Postfix SMTP          | TCP      | 25   |
| Dovecot IMAP          | TCP      | 143  |
| Dovecot IMAPS         | TCP      | 993  |
| Dovecot ManageSieve   | TCP      | 4190 |
| HTTPS                 | TCP      | 443  |
| HTTP (301+autoconfig) | TCP      | 80   |

- Next it is important that you **do not use Google DNS** or another public DNS which is known to be blocked by DNS-based Blackhole List (DNSBL) providers.

# Installation
**Please run all commands as root**

**Download a stable release**

Download mailcow to whichever directory (using ~/build here).
Replace "v0.x" with the tag of the latest release: https://github.com/andryyy/mailcow/releases/latest
```
mkdir ~/build ; cd ~/build
wget -O - https://github.com/andryyy/mailcow/archive/v0.x.tar.gz | tar xfz -
cd mailcow-*
```

**Now edit the file "configuration" to fit your needs!**
```
nano mailcow.config
```

* **sys_hostname** - Hostname without domain
* **sys_domain** - Domain name. "$sys_hostname.$sys_domain" equals to FQDN.
* **sys_timezone** - The timezone must be definied in a valid format (Europe/Berlin, America/New_York etc.)
* **conf_httpd** - Select wether to use Nginx ("nginx") or Apache2 ("apache2"). Nginx is default.
* **my_dbhost** - ADVANCED: Leave as-is ("localhost") for a local database installation. Anything but "localhost" or "127.0.0.1" is recognized as a remote installation.
* **my_usemariadb** - Use MariaDB instead of MySQL. Only valid for local databases. Installer stops when MariaDB is detected, but MySQL selected - and vice versa.
* **my_mailcowdb, my_mailcowuser, my_mailcowpass** - SQL database name, username and password for use with Postfix. **You can use the default values.**
* **my_rcdb, my_rcuser, my_rcpass** - SQL database name, username and password for Roundcube. **You can use the default values.**
* **my_rootpw** - SQL root password is generated automatically by default. You can define a complex password here if you want to. *Set to your current root password to use an existing SQL instance*.
* **mailcow_admin_user and mailcow_admin_pass** - mailcow administrator. Password policy: minimum length 8 chars, must contain uppercase and lowercase letters and at least 2 digits. **You can use the default values**.
* **"cert-" vars** - Used for the self-signed certificate. CN will be the servers FQDN. "cert_country" must be a vaild two letter country code.
* **inst_debug** - Sets Bash mode -x
* An unattended installation is possible, but not recommended ("inst_unattended")

**Empty configuration values are invalid!**

You are ready to start the script:
```
./install.sh
```
Just be patient and confirm every step by pressing [ENTER] or [CTRL-C] to interrupt the installation.
If you run into problems, try to locate the error with "inst_debug" enabled in your configuration.
Please contact me when you need help or found a bug.

More debugging is about to come. Though everything should work as intended.

After the installation, visit your dashboard @ **https://hostname.domain.tld**, use the logged credentials in `./installer.log`

Remember to create an alias- or a mailbox for Postmaster. ;-)

# Upgrade
**Please run all commands as root**

Upgrade is supported since mailcow v0.7.x. From v0.9 on you do not need the file `installer.log` from a previous installation.

The mailcow configuration file will not be read, so there is no need to adjust it in any way before upgrading.

To start the upgrade, run the following command:
```
./install.sh -u
```

# SSL certificate
The SSL certificate is located at `/etc/ssl/mail/mail.{key,crt}`.
You can replace it by just copying over your own files. 
Services effected and necessary to restart are `postfix`, `dovecot` and `nginx` (Nginx if installed, Apache2 does not have a body size limitation configured).

# Getmail4
Install getmail4:
```
apt-get install getmail4
```
Create a getmail4 profile (as root):
```
mkdir ~/.getmail/
nano ~/.getmail/profile1
```
Example profile (do not forget to set me@my-mailcow.tld):
```
[retriever]
# Use SimpleIMAPSSLRetriever for IMAPS, SimpleIMAPRetriever for IMAP etc.
# Available retrievers: SimpleIMAPRetriever, SimpleIMAPSSLRetriever, SimplePOP3SSLRetriever, SimplePOP3Retriever
type = SimpleIMAPSSLRetriever
server = imap.googlemail.com
#server = imap-mail.outlook.com
username = MYUSERNAME
password = MYPASSWORD
mailboxes = ("[Gmail]/All Mail",)
#mailboxes = ("[Gmail]/Alle Nachrichten",)
#mailboxes = ("INBOX", "INBOX.spam",)
port = 993

[destination]
type = MDA_external
path = /usr/lib/dovecot/deliver
# Remember to change the destination address
arguments = ("-e", "-f", "%(sender)", "-d", "me@my-mailcow.tld")
user = vmail
group = vmail

[options]
# Email header will not be altered
received = false
delivered_to = false
# Only fetch missing mails
read_all = false
verbose = 2
message_log = /root/.getmail/deliver.log
message_log_verbose = true
```
This profile delivers directly to Dovecot and does not need to pass a SMTPd process.

You can add a cronjob to run getmail every 10 minutes:
```
crontab -e
```
Last line:
```
*/10 * * * * /usr/bin/getmail -r profile1 --quiet
```
Run manually:
```
/usr/bin/getmail -r profile1
```
You can add an unlimited amount of profiles.

# Uninstall
Run `bash misc/purge.sh` from within mailcow directory to remove mailcow main components.

Your web server + web root, MySQL server + databases as well as your mail directory (/var/vmail) will **not** be removed (>= v0.9).

Please open and review the script before running it!

**You can perform a FULL WIPE** by appending `--all`:

```bash misc/purge.sh --all```

This WILL purge sensible data like your web root, databases + MySQL installation, mail directory and more... 
