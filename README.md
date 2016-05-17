# Installation notes on OwnCloud

## Install CentOS 7
Install CentOS 7 minimal installation

 - Set root password
 - Make administrator account

Disable SElinux:
```
$ sudo  vi /etc/sysconfig/selinux
...
SELINUX=disabled
...
```

Enable http through firewall:
```
$ sudo firewall-cmd --permanent --add-service=http
$ sudo firewall-cmd --permanent --add-service=https
$ sudo firewall-cmd --reload
```
CURL SKAL OPDATERES INDTIL CentOS 7.1.3 er frigivet
```
sudo rpm -Uvh http://www.city-fan.org/ftp/contrib/yum-repo/rhel7/x86_64/city-fan.org-release-1-13.rhel7.noarch.rpm
sudo yum -y install libcurl php-curl
```

## Install dependencies
```
$ sudo yum -y install httpd mariadb mariadb-server php-fpm php-cli php-gd php-mcrypt php-mysql php-pear php-xml bzip2 vim
$ sudo systemctl start mariadb
$ sudo mysql_secure_installation
    current root password is [blank] - just press [[ENTER]]
    Change the root password? [Y/n] Y -> 3grise+1ulv
    Remove anonymous users? [Y/n] Y
    Disallow root login remotely? [Y/n] Y
    Remove test database and access to it? [Y/n] Y
    Reload privilege tables now? [Y/n] Y
```

Configure PHP-FPM (FastCGI Process Manager)
```
$ sudo vim /etc/php-fpm.d/www.conf
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

## CREATE DATABASE FOR OWNCLOUD 
```
$ mysql -u root -p
> CREATE DATABASE owncloud_db;
> CREATE USER owncloud_usr@localhost IDENTIFIED BY 'secretpassword';
> GRANT ALL PRIVILEGES ON owncloud_db.* TO owncloud_usr IDENTIFIED BY 'secretpassword';
> FLUSH PROVILEGES;
> QUIT;
```

## INSTALL OWNCLOUD:
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
´´´
administrator: admin
password: yourSecretPassword
´´´
Choose: mariadb
```
db_user: owncloud_usr 
db_pass: secretpassword 
db_name: owncloud_db
db_host: localhost 
```
You ar e now logged in for the first time.

I menuen under det lille sky ikon i øverste venstre hjørne trykker du på plustegnet "Apps"
i ventremenuen vælger du "slået fra"
Find tilføjelsen "LDAP user and group backend" og aktiver den.

Skift til Admin siden i menuen øverst til højre.

Find afsnittet med LDAP og indtast følgende
Server: ldap://ldap1.govcert.dk
Port: 389
Bruger Dn:
Kodeord:
Base DN: dc=govcert,dc=dk

På fanen "Brugere" vælges "PosixAccount"
På fanen "Grupper" vælges "Rediger LDAP forespørgsel"
Indtast objectclass=posixGroup

Opret gruppe mapper - grupper i LDAP kan findes med kommandoen
ldapsearch -H ldap://ldap1.govcert.dk -b "dc=govcert,dc=dk" -s sub -x "(objectclass=posixGroup)" cn

Hovedparten af denne vejledning er taget fra:
https://www.howtoforge.com/tutorial/how-to-install-owncloud-8-with-nginx-and-mariadb-on-centos-7/

