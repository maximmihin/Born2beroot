# Born2beroot

## About

Born2beRoot is an introduction to virtualization and system administration. The purpose here is to create a virtual machine which is a server, with the LST version of Debian, implementing strict rules, and minimum required services.

For detailed information, refer to the **[subject of this project](https://github.com/maximmihin/Born2beroot/blob/master/en.subject.pdf)**

## Table of Contents

1. [UFW](#ufw)
2. [SSH](#ssh)
3. [Apparmor](#apparmor)
4. [Sudo](#sudo)
5. [Passwords policy](#passwords-policy)
6. [Bash script-monitoring](#Bash-script-monitoring)
7. [WordPress](#WordPress)

## UFW

### Requirements:

> You have to configure your operating system with the UFW firewall and thus leave only port 4242 open.

Step 0:

```bash
apt update
```

Install UFW

```bash
apt install ufw -y
```

Enable Firewall

```bash
ufw enable
```

Allow incoming connection using Port 4242 (for ssh)

```bash
ufw allow 4242
```

Check UFW status

```bash
ufw status verbose
```



## SSH

### Requirements:

> A SSH service will be running on port 4242 only. For security reasons, it must not be possible to connect using SSH as root.

Install ssh

```bash
apt install openssh-server
```

Start ssh

```bash
systemctl start sshd
```

Change default Port to 4242 and denied connection SSH as root

```bash
vi /etc/ssh/sshd_config
```

Change:

```
13 #Port 22
32 #PermitRootLogin prohibit-password
```

to

```
13 Port 4242
32 PermitRootLogin no
```

Restart ssh

```bash
service ssh restart
```

Check status

```bash
systemctl status sshd
```

Now we can connect

```bash
ssh <user_name>@<ip_addres> -p <port>
```

```bash
ssh gradagas@localhost -p 4242
```



## Apparmor

Requirements:

> <...> AppArmor for Debian must be running at startup too.

install apparmor utils and profiles

```bash
apt install apparmor-utils apparmor-profiles -y
```

Check apparmor

```bash
apparmor_status
```



## Sudo

Requirements:

> - Authentication using sudo has to be limited to 3 attempts in the event of an incorrect password.
> - A custom message of your choice has to be displayed if an error due to a wrong password occurs when using sudo.
> - Each action using sudo has to be archived, both inputs and outputs. The log file has to be saved in the /var/log/sudo/ folder.
> - The TTY mode has to be enabled for security reasons.
> - For security reasons too, the paths that can be used by sudo must be restricted. Example: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin

Install sudo

```bash
apt install sudo
```

Add user to the sudoers group

```bash
usermod -aG sudo <user_name>
```

or

```bash
adduser <username> sudo
```

Check users group

```bash
cat /etc/group
```

Configuring sudo.

```bash
EDITOR=vim /usr/sbin/visudo
```

Add the below lines to the file

```bash
Defaults	passwd_tries=3
Defaults	badpass_message="<your_origin_badpass_message>"
Defaults	logfile=/var/log/sudo
Defaults	iolog_dir=/var/log/sudo
Defaults	log_input
Defaults	log_output
Defaults	requiretty
Defaults	secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
```



## Passwords policy

### Password Age policy

Requirements:

> - Password has to expire every 30 days.
> - The minimum number of days allowed before the modification of a password will be set to 2.
> - The user has to receive a warning message 7 days before their password expires.
>

```bash
vim /etc/login.defs
```

Change below lines to the file

```bash
160 PASS_MAX_DAYS   99999
161 PASS_MIN_DAYS   0
162 PASS_WARN_AGE   7
```

to

```bash
160 PASS_MAX_DAYS   30
161 PASS_MIN_DAYS   2
162 PASS_WARN_AGE   7
```

These rules will apply **only** to new users. To change age policy, existing users use:

To change PASS_MAX_DAYS

```bash
chage -M <num_days> <user_name>
```

To change PASS_MIN_DAYS

```bash
chage -m <num_days> <user_name>
```

Check

```bash
chage -l <user_name>
```



#### Password Strength policy

Requirements:

> - Your password must be at least 10 characters long. It must contain an uppercase letter and a number. Also, it must not contain more than 3 consecutive identical characters. 
> - The password must not include the name of the user.
> - The following rule does not apply to the root password: The password must have at least 7 characters that are not part of the former password.
> - Of course, your root password has to comply with this policy.
>



Install the *libpam-pwquality* package.

```bash
apt install libpam-pwquality -y
```

Edit config file

```bash
vim /etc/security/pwquality.conf
```

Desired settings:

```bash
difok = 7
minlen = 10
dcredit = -1
ucredit = -1
maxrepeat = 3
usercheck = 1
enforce_for_root = 1
```

## Bash script-monitoring

Requirements:

> At server startup, the script will display some information (listed below) on all terminals every 10 minutes (take a look at wall). The banner is optional. No error must be visible.
>
> Your script must always be able to display the following information:
>
> - The architecture of your operating system and its kernel version.
> - The number of physical processors.
> - The number of virtual processors.
> - The current available RAM on your server and its utilization rate as a percentage.
> - The current available memory on your server and its utilization rate as a percentage.
> - The current utilization rate of your processors as a percentage.
> - The date and time of the last reboot.
> - Whether LVM is active or not.
> - The number of active connections.
> - The number of users using the server.
> - The IPv4 address of your server and its MAC (Media Access Control) address.
> - The number of commands executed with the sudo program.

I found a good reference [here](https://github.com/HEADLIGHTER/Born2BeRoot-42/blob/main/monitoring.sh). For this work need to install net-tools package.

```bash
apt install net-tools
```

Make monitoring.sh (reference [here](https://github.com/HEADLIGHTER/Born2BeRoot-42/blob/main/monitoring.sh))

```bash
vim /usr/local/bin/monitoring.sh
```

Make the file executable

```bash
chmod +x /usr/local/bin/monitoring.sh
```

Add script to cron

```bash
crontab -e
```

Add to the opened file-config line below

```bash
*/10 * * * * root /usr/local/bin/monitoring.sh
```

Check script work

```bash
grep CRON /var/log/syslog
```



## Wordpress

Requirements:

> Set up a functional WordPress website with the following services: lighttpd, MariaDB, and PHP.

### PHP

Install PHP

```bash
apt install php7.4 php7.4-fpm php7.4-mysql php7.4-cli php7.4-cgi php7.4-curl php7.4-xml -y
```

### lighttpd

Install lighttpd

```bash
apt install lighttpd -y
```

Configure lighttpd

```bash
lighttpd-enable-mod fastcgi
```

```bash
lighttpd-enable-mod fastcgi-php
```

```bash
lighty-enable-mod accesslog
```

Enable rewrite in Lighttpd

```bash
vim /etc/lighttpd/lighttpd.conf
```

Add line below to `server.modules`

```bash
"mod_rewrite",
```

### MariaDB

Install MariaDB

```bash
apt install mariadb-server mariadb-client -y
```

```bash
mysql_secure_installation
```

Make new database

```bash
mysql -u root -p
```

```SQL
CREATE DATABASE wpdb;
```

```SQL
CREATE USER 'wpdbuser'@'localhost' IDENTIFIED BY 'new_password_here';
```

```sql
GRANT ALL ON wpdb.* TO 'wpdbuser'@'localhost' IDENTIFIED BY 'user_password_here' WITH GRANT OPTION;
```

```SQL
FLUSH PRIVILEGES;
```

```sql
EXIT;
```

### Wordpress

Install Wordpress

```bash
cd /tmp/ && wget http://wordpress.org/latest.tar.gz
```

```bash
tar -xzvf latest.tar.gz
```

```bash
cp -R wordpress/* /var/www/html
```

```bash
rm -rf /var/www/html/*.index.html
```

```bash
cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
```

Change file-config

```bash
vim /var/www/html/wp-config.php
```

Desired settings:

```php
define( 'DB_NAME', 'wpdb' );
define( 'DB_USER', 'wpdbuser' );
define( 'DB_PASSWORD', 'user_password_here' );
```

Сontinue install

```bash
chown -R www-data:www-data /var/www/html/
```

```bash
chmod -R 755 /var/www/html/
```

```bash
systemctl restart lighttpd.service
```

```bash
ufw allow 80
```

Check: go to http://localhost/
