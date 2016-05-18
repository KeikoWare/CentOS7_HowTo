# Installation notes on OwnCloud

## Install CentOS 7
Install CentOS 7 minimal installation

 - Set root password
 - Make administrator account

And login as administrator.

Disable SElinux:
```
$ sudo  vi /etc/sysconfig/selinux
[[i]]
    SELINUX=disabled
[[ESC]]
:w
:q
```

Enable http through firewall:
```
$ sudo firewall-cmd --permanent --add-service=http
$ sudo firewall-cmd --permanent --add-service=https
$ sudo firewall-cmd --reload
```
CURL needs to be updated due to a security issue.
Fix will come with CentOS 7.1.3, so until that comes out you need to do the following:
```
$ sudo rpm -Uvh http://www.city-fan.org/ftp/contrib/yum-repo/rhel7/x86_64/city-fan.org-release-1-13.rhel7.noarch.rpm
$ sudo yum -y install libcurl php-curl
```

## Install dependencies
```
$ sudo yum -y install httpd mariadb mariadb-server php-fpm php-cli php-gd php-mcrypt php-mysql php-pear php-xml bzip2 vim
$ sudo systemctl start mariadb
$ sudo mysql_secure_installation
    current root password is [blank] - just press [[ENTER]]
    Change the root password? [Y/n] Y -> yourSecretPassword 
    Remove anonymous users? [Y/n] Y
    Disallow root login remotely? [Y/n] Y
    Remove test database and access to it? [Y/n] Y
    Reload privilege tables now? [Y/n] Y
```

Configure PHP-FPM (FastCGI Process Manager)
Make sure that the configuration is as following (default):
```
$ sudo vi /etc/php-fpm.d/www.conf
[[i]]
    listen = 127.0.0.1:9000
    user = apache
    group = apache
[[ESC]]
:w
:q

$ sudo mkdir -p /var/lib/php/session
$ sudo chown apache:apache -R /var/lib/php/session/
$ sudo systemctl start php-fpm

$ sudo systemctl start httpd
```

## Create database for ownCloud 
```
$ mysql -u root -p
> CREATE DATABASE owncloud_db;
> CREATE USER owncloud_usr@localhost IDENTIFIED BY 'anotherSecretPassword';
> GRANT ALL PRIVILEGES ON owncloud_db.* TO owncloud_usr IDENTIFIED BY 'anotherSecretPassword';
> FLUSH PROVILEGES;
> QUIT;
```

## Install ownCloud
First create /etc/yum.repos.d/owncloud.repo:
```
$ sudo vi /etc/yum.repos.d/owncloud.repo 
[[i]]
    [isv_ownCloud_community]
    name=Latest stable community release of ownCloud (CentOS_CentOS-7)
    type=rpm-md
    baseurl=http://download.opensuse.org/repositories/isv:/ownCloud:/community/CentOS_CentOS-7/
    gpgcheck=1
    gpgkey=http://download.opensuse.org/repositories/isv:/ownCloud:/community/CentOS_CentOS-7/repodata/repomd.xml.key
    enabled=1

[[ESC]]
:w
:q
```

After creating the file, type:
```
$ sudo yum install -y owncloud-config-apache owncloud-server owncloud
```
Answer yes to all questions during installation 
```
$ sudo chown -R apache:apache /var/www/html/owncloud/
```

## Finish of the installation
Finish of the installation from the web GUI

Browse to http://[the_ip_of_your_server]/owncloud 
```
administrator: admin
password: thirdSecretPassword
```
Choose: mariadb
```
db_user: owncloud_usr 
db_pass: anotherSecretPassword 
db_name: owncloud_db
db_host: localhost 
```
You ar e now logged in for the first time.

In the left side top menu undernieth the little cloud icon, press the + sign called "Apps"
The new menu on the left pane, you choose "slået fra" / "disabled"
Find the app called "LDAP user and group backend" and enable it 

In the top right menu change to the "Admin" menu.


find the LDAP section (newly added based on the enabling of the LDAP app).
Configure it appropriate to your own LDAP settings. Example:
```
Server: ldap://ldapserver.domain.org
Port: 389
Bruger Dn:
Kodeord:
Base DN: dc=domain,dc=org
```

On the pane "Users" choose "posixAccount"
On the pane Goups choose "Edit LDAP query" and insert the text: objectclass=posixGroup

now create folders that matches the groups in your LDAP, and you have yourself a organisational file sharing platform, based on http.

Credits goes to:
https://www.howtoforge.com/tutorial/how-to-install-owncloud-8-with-nginx-and-mariadb-on-centos-7/

