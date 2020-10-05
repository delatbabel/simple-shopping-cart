# simple-shopping-cart
Simple PHP shopping cart which I use for performance testing

I got this code from https://phppot.com/php/simple-php-shopping-cart/ and then made some
modifications for use in my class on web application performance testing.

Please feel free to contact the original developer for freelance work.

If you want any further information about my course on performance testing and tuning
for web applications then contact me via email -- del@babel.com.au

## Setup Instructions

These are the setup steps that I used on a CentOS 8 virtual machine running under Oracle VirtualBox on an Ubuntu Linux host.

```
# Install some packages on top of base
dnf -y update
dnf -y install psmisc sysstat
dnf -y install sysstat wget bind-utils zip unzip rsync openssh-clients mlocate net-tools psmisc telnet
chmod +x /etc/cron.daily/mlocate

# Turn off SELinux Enforcing Mode
cat /etc/selinux/config | sed 's/=enforcing/=permissive/g' > /tmp/selinux.tmp
/bin/mv /tmp/selinux.tmp /etc/selinux/config
setenforce permissive
getenforce

# Set up for PHP applications
dnf -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
dnf -y install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
dnf module reset php
dnf module install php:remi-7.3
dnf config-manager --set-enabled PowerTools

dnf -y install httpd
dnf -y install php mariadb
dnf -y install php-tidy
dnf -y install php-mysql
dnf -y install php-gd
dnf -y install php-imap
dnf -y install php-mbstring
dnf -y install php-odbc
dnf -y install php-intl

dnf -y install php-pecl-zendopcache
dnf -y install php-pecl-apcu
dnf -y install php-pecl-igbinary
dnf -y install php-pecl-imagick
dnf -y install php-pecl-memcache
dnf -y install php-pecl-memcached
dnf -y install php-pecl-xdebug
dnf -y install memcached
dnf -y install phpmyadmin

# MariaDB Install
dnf -y install mariadb-server

# Set up PHPMyAdmin
# DO NOT DO THIS ON A PRODUCTION SYSTEM -- THIS IS ONLY FOR TESTING
cd /etc/httpd/conf.d/
vi phpMyAdmin.conf
# Change the permissions of the first Directory section to "Require all granted" 

# Enable XDebug and Memcache for sessions
cd /etc/php-fpm.d/
vi www.conf
# Include these lines
#; for xdebug / kcachegrind
#php_flag[xdebug.profiler_enable] = on
#php_value[xdebug.profiler_output_dir] = "/tmp"
# Also adjust these values (one of the two pairs) for setting session handler and save path
#php_value[session.save_handler] = memcached
#php_value[session.save_path]    = "localhost:11211"
#;php_value[session.save_handler] = files
#;php_value[session.save_path]    = /var/lib/php/session

# Enable Services
systemctl enable memcached.service
systemctl enable httpd.service
systemctl enable mariadb.service
systemctl start memcached
systemctl start httpd
systemctl start mariadb

# MariaDB Setup
mysql_secure_installation

# Firewall Setup
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
firewall-cmd --list-all

# Install simple shopping cart code
cd /var/www/html
unzip simple-php-shopping-cart.zip
cd simple-php-shopping-cart/
mv * ..
cd ..
rm -rf simple-php-shopping-cart

# Setup simplecart Database
echo "grant all on simplecart.* to simpleuser@localhost identified by 'simplepass';" | mysql -u root -p
mysql -u simpleuser --password="simplepass" simplecart < tblproduct.sql
```
