## TOC
* [Linux and Web server setup under GCE](#linux-and-web-server-setup-under-gce)
* [Install the Magento with the compressed archive](#install-the-magento-with-the-compressed-archive)

## Linux and Web server setup under GCE
### Time zone and ntp
```
apt-get update
dpkg-reconfigure tzdata
apt-get install ntp
```

### Nginx
```
apt-get update
apt-get -y install nginx
```

#### Nginx config
* Optional, enable default PHP interpreter  

Uncomment the partial commands of the PHP section in `/etc/nginx/sites-available/default`.

```
# pass PHP scripts to FastCGI server                                                                       
#                                                                                                          
location ~ \.php$ {                                                                                        
        include snippets/fastcgi-php.conf;                                                                 

        # With php-fpm (or other unix sockets):                                                            
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;                                                    
#       # With php-cgi (or other tcp sockets):                                                             
#       fastcgi_pass 127.0.0.1:9000;                                                                       
}   
```

### PHP & modules (extensions)
```
apt-get update
apt-get -y install php7.4-fpm php7.4-cli
```

```
apt-get install php7.4-pdo php7.4-mysqlnd php7.4-opcache php7.4-xml php7.4-gd php7.4-devel php7.4-mysql php7.4-intl php7.4-mbstring php7.4-bcmath php7.4-json php7.4-iconv php7.4-soap php7.4-curl php7.4-zip
```

### [Check PHP settings](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/php-settings.html#check-php-settings)

### Check PHP/Nginx services
#### Restart services
```
systemctl restart php7.4-fpm.service
systemctl restart nginx.service 
```

#### Verify PHP is installed
* `/var/www/html/phpinfo.php`
```
<?php
// Show all information, defaults to INFO_ALL
phpinfo();
?>
```

* open `http://${HOST}/phpinfo.php` with a browser 

## MySQL
### [Installing MySQL](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/mysql.html#instgde-prereq-mysql-ubuntu)
### install mysql-client
```
apt-get install mysql-client-core-8.0
```
### usage
```
mysql -h {mysql_server_host} -u ${username} -p 
```

### [Configuring the Magento database instance](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/mysql.html#instgde-prereq-mysql-config)
* [GRANT Statement](https://dev.mysql.com/doc/refman/8.0/en/grant.html)

```
create database magento;
create user 'magento'@'<remote web node server ip address> or %' IDENTIFIED BY 'magento';
GRANT ALL ON magento.* TO 'magento'@'<remote web node server ip address> or %';
flush privileges;
```

## Elasticsearch
### [OpenJDK](https://openjdk.java.net/)
* [support matrix](https://www.elastic.co/support/matrix#matrix_jvm)
```
apt-get update
apt-get install openjdk-11-jdk
```

### [download ES](https://www.elastic.co/downloads/past-releases/elasticsearch-7-6-2)
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.6.2-linux-x86_64.tar.gz
```

### [install ES from archive](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html)
```
tar -xvf elasticsearch-7.6.2-linux-x86_64.tar.gz
```

## [Install the Magento with the compressed archive](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/zip_install.html)
* [Config an new virtual host, Nginx](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/nginx.html#configure-nginx-ubuntu)

* [Get the Magento archives](https://magento.com/tech-resources/download)
  * under **Archive (zip/tar)** section
  
* [Extract the software on your server](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/zip_install.html#zip-extract)
```
sudo adduser <magento_user>
cd <web server docroot>        # Ubuntu default: /var/www/html
mkdir magento2
cp magento-xxx.zip magento2
cd magento2
unzip magento-xxx.zip
```

* [Set ownership and permissions](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/file-system-perms.html#perms-private)
```
sudo usermod -a -G www-data <magento_user>

su -l <magento_user>
cd <magento_root>
find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +
chown -R :<web server group> .
chmod u+x bin/magento
```


### [Sample localhost installations](https://devdocs.magento.com/guides/v2.4/install-gde/install/cli/install-cli-install.html#install-cli-example)
```
./bin/magento setup:install --base-url=http://127.0.0.1/magento2/ \
--db-host=${db_host} --db-name=magento --db-user=magento --db-password=${db_pass} \
--admin-firstname=Magento --admin-lastname=User --admin-email=user@example.com \
--admin-user=admin --admin-password=${admin_pass} --language=zh_Hant_TW \
--currency=TWD --timezone=Asia/Taipei --use-rewrites=1 \
--search-engine=elasticsearch7 --elasticsearch-host=${es_host} \
--elasticsearch-port=9200
```


## ~~Composer~~
### [Command-line installation](https://getcomposer.org/download/)
```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '795f976fe0ebd8b75f26a6dd68f78fd3453ce79f32ecb33e7fd087d39bfeb978342fb73ac986cd4f54edd0dc902601dc') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```


## Reference
* [magento helpful resources](https://devdocs.magento.com/guides/v2.4/install-gde/install-resources-parent.html)
* [Magezon Page Builder extension, community](https://www.magezon.com/magezon-page-builder-for-magento-2.html)
### Connector
* [SAP Business One Integration Add-on for Magento 2](https://firebearstudio.com/sap-business-one-integration-add-on-for-magento-2.html)
