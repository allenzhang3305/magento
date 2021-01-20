## TOC
* [Linux and Web server setup under GCE](#linux-and-web-server-setup-under-gce)
* [MySQL](#mysql)
* [Magento installation](#magento-installation)
* [Magento extension module installation ](#magento-extension-module-installation)
* [CLI](#command-line)


### Network overview
![](https://raw.githubusercontent.com/MRLIVING/magento/master/doc/img/overview_network.PNG)

## Linux and Web server setup under GCE
### Time zone and ntp
```
apt-get update
dpkg-reconfigure tzdata
apt-get install ntp
```

### Unzip
```
apt-get install unzip
```

### Nginx
```
apt-get -y install nginx
```

#### Optional, enable default PHP interpreter  
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

### [PHP & modules (extensions)](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/php-settings.html#verify-installed-extensions)
```
apt-get update
apt-get -y install php7.4-fpm php7.4-cli
```

```
apt-get install php7.4-fpm php7.4-common php7.4-mysql php7.4-gmp php7.4-curl php7.4-intl php7.4-mbstring php7.4-xmlrpc php7.4-gd php7.4-xml php7.4-cli php7.4-zip php7.4-soap php7.4-bcmath
```

### [Check PHP settings](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/php-settings.html#check-php-settings)

### Check PHP/Nginx services
#### Restart services
```
systemctl restart php7.4-fpm.service
systemctl restart nginx.service 
```

#### Verify PHP is installed
* we have tested on `PHP v7.4.1/v7.4.3`.
* [/var/www/html/phpinfo.php](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/optional.html#install-optional-phpinfo)
```
<?php
// Show all information, defaults to INFO_ALL
phpinfo();
?>
```

* open `http://${HOST}/phpinfo.php` with a browser 

## MySQL
### [Installing MySQL](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/mysql.html#instgde-prereq-mysql-ubuntu)
* MySQL `v8.0`
* Character set `utf8mb4`
* Collation `utf8mb4_0900_ai_ci`
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
* [OpenJDK](https://openjdk.java.net/)
  * [support matrix](https://www.elastic.co/support/matrix#matrix_jvm)
```
sudo apt-get update
sudo apt-get install openjdk-8-jdk
```

* [download ES](https://www.elastic.co/downloads/past-releases/elasticsearch-7-6-2)
```
adduser elk
sudo su -l elk
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.6.2-linux-x86_64.tar.gz
```

* [install ES from archive](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html)
```
tar -xvf elasticsearch-7.6.2-linux-x86_64.tar.gz
```

## [Magento installation](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/zip_install.html)
* Add an user as [magento file system owner](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/file-sys-perms-over.html#magento-file-system-owner)
  ```
  sudo adduser <magento_user>
  sudo mkdir -p /var/www/html/magento2
  ```

* Get the Magento package
  * [Compressed archive](https://magento.com/tech-resources/download), under **Archive (zip/tar)** section.
    * [Extract the software on your server](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/zip_install.html#zip-extract)
      ```            
      sudo cp <src path>/magento-ce-2.4.1-2020-09-25-04-08-02.zip /var/www/html/magento2/
      sudo chown -R <magento_user>:<magento_user> magento2
      
      sudo su -l <web server docroot>
      cd /var/www/html/magento2
      unzip magento-ce-2.4.1-2020-09-25-04-08-02.zip
      ```
      
  * or [Composer](https://devdocs.magento.com/guides/v2.4/install-gde/composer.html#get-the-metapackage)
    * Install Composer
      ```
      curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer
      composer self-update 1.10.16    # downgrade it back to version 1.x due to a recent incompatibility issue with one of the packages.
      ```
    
    * Create a new Composer project (get the Magento software metapackage)
      ```
      sudo su -l <web server docroot>
      cd /var/www/html/magento2
      composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition .
      ```      
      * [Get your authentication keys](https://devdocs.magento.com/guides/v2.3/install-gde/prereq/connect-auth.html)
         


* [Set ownership and permissions](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/file-system-perms.html#perms-private)
```
sudo usermod -a -G www-data <magento_user>

su -l <magento_user>
cd <magento_root>
find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +
chown -R :www-data .
chmod u+x bin/magento
```

* Config the [Server Block](https://www.nginx.com/resources/wiki/start/topics/examples/server_blocks/#two-server-blocks-serving-static-files) of Nginx
  * [nginx server configuration](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/nginx.html#configure-nginx-ubuntu)     
    * `/etc/nginx/sites-available/magento`
    ```
    upstream fastcgi_backend {
        server  unix:/run/php/php7.4-fpm.sock;
    }

    server {
        listen 80;
    #    server_name www.magento-dev.com;
        set $MAGE_ROOT /var/www/html/magento2;
        include /var/www/html/magento2/nginx.conf.sample;
    }
    ```
    
  * `ln -s /etc/nginx/sites-available/magento /etc/nginx/sites-enabled`
  
  * `rm /etc/nginx/sites-enabled/default`

### [Sample localhost installations](https://devdocs.magento.com/guides/v2.4/install-gde/install/cli/install-cli-install.html#install-cli-example)
```
./bin/magento setup:install --base-url=http://${m2_host}/ --db-host=${db_host} --db-name=${db_name} --db-user=${db_user} --db-password=${dp_pass} --admin-firstname=Magento --admin-lastname=User --admin-email=user@example.com --admin-user=${admin_name} --admin-password=${admin_pass} --language=zh_Hant_TW --currency=TWD --timezone=Asia/Taipei --use-rewrites=1
```

#### [Installer help commands](https://devdocs.magento.com/guides/v2.4/install-gde/install/cli/install-cli-install.html#instgde-cli-help-cmds)
* `magento info:language:list`
* `magento info:currency:list`
* `magento info:timezone:list`

### [Install sample data after Magento](https://devdocs.magento.com/guides/v2.4/install-gde/install/sample-data-after-magento.html)
* [Deploy Sample Data from GitHub Repository](https://github.com/magento/magento2-sample-data#deploy-sample-data-from-github-repository)


## Magento extension module installation 
TODO...
### Composer
* add MageWorx composer repository to your Magento composer.json file  
  `composer config ${vender repository name} composer ${url of composer repository}`
* `composer require ${mageworx-module-composer-code}`

* `./bin/magento module:enable MageWorx_Info MageWorx_OrdersBase MageWorx_OrderEditor`

* `./bin/magento setup:upgrade`

### Copy extension folder and files
TODO...

### 3rd-party extensions
* [Simple Chinese Language Pack](https://marketplace.magento.com/sunflowerbiz-magento-2-chinese-language-pack.html)
  * unzip the zip package into `/${M2_ROOT}/app/i18n/Sunflowerbiz/zh_hans_cn/`
  * `magento setup:upgrade`
  * enable in front-end
    * backend console `STORES/ Configuration/ GENERAL/ General/ Locale Options/ Locale/ Chinese (Simplified Han, China)` and click `Save Config`
  * enable in backend
    * backend console `SYSTEM/ ALL USERS/ click an user/ Interface Locale/ Chinese (Simplified Han, China)`

* [The Most Popular SMTP for Magento 2](https://www.mageplaza.com/magento-2-smtp/)
  * [installation guide](https://www.mageplaza.com/install-magento-2-extension/#smtp)
```
composer require mageplaza/module-smtp
php bin/magento setup:upgrade
php bin/magento setup:static-content:deploy
```

* [Magento 2 PDF Customizer](https://www.magezon.com/magento-2-pdf-customizer.html)
  * prerequisites 
    * [mPDF](https://github.com/mpdf/mpdf) 
      ```
      cd /var/www/
      composer require mpdf/mpdf
      ```

## Setup GCP CDN (Alpha)
### Change the base URLs for `Static View Files` and `User Media Files`
* Disable [Static content signing](https://devdocs.magento.com/guides/v2.4/config-guide/cache/static-content-signing.html)
  * see [ref](https://magento.stackexchange.com/questions/167278/where-do-i-point-my-secure-base-url-for-static-view-files-for-cdn-in-magento-2)

* Pre-generate all necessary resizes `magento catalog:image:resize` 
  * see [Regenerate catalog cache images issues](https://magento.stackexchange.com/questions/175224/regenerate-catalog-cache-images-issues)

## Command-line

* `magento -h`

* `magento module:disable Magento_TwoFactorAuth`

* `magento deploy:mode:show`, see [Set the Magento mode](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-mode.html)
  * `magento deploy:mode:set ${mode}`, where ${mode} can be either `default`, `developer`, or `production`.

* `magento config:show`, see [Display the value of configuration settings](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-config-mgmt-set.html#config-cli-config-show)

* `magento setup:store-config:set`, see [Configure the store](https://devdocs.magento.com/guides/v2.4/install-gde/install/cli/install-cli-subcommands-store.html#instgde-cli-storeconfig).
   * `magento setup:store-config:set --base-url="http://${Host-IP}/"`   

* [list all enabled/disabled modules](https://devdocs.magento.com/guides/v2.4/install-gde/install/cli/install-cli-subcommands-enable.html#instgde-cli-subcommands-status)  
  `magento module:status`
  
* [Enable or disable modules](https://devdocs.magento.com/guides/v2.4/install-gde/install/cli/install-cli-subcommands-enable.html)  
  `magento module:disable Magento_TwoFactorAuth`
  
* `bin/magento cache:status`,  see [View the cache status](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-cache.html#view-the-cache-status)

* `bin/magento c:f`, cache:flush see [Clean and flush cache types](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-cache.html#config-cli-subcommands-cache-clean)

* `bin/magento catalog:image:resize`, see [Resize catalog images](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/themes/theme-images.html#resize-catalog-images)

## [phpMyAdmin for MYSQL](https://github.com/MRLIVING/magento/wiki/phpMyAdmin-on-GCE-with-Cloud-MySQL)

## Reference
* [magento helpful resources](https://devdocs.magento.com/guides/v2.4/install-gde/install-resources-parent.html)
* [Magezon Page Builder extension, community](https://www.magezon.com/magezon-page-builder-for-magento-2.html)

### Connector, [evaluation](https://docs.google.com/spreadsheets/d/1i0igXkdmvaEnmVIAXKGaDJ4-dELUSeVPWHXuPT6NaEg/edit?usp=sharing)
* [SAP Business One Connect, i95Dev](https://marketplace.magento.com/i95dev-i95devsapconnect.html)
* [SAP Business One Integration Add-on for Magento 2, Firebear](https://firebearstudio.com/sap-business-one-integration-add-on-for-magento-2.html)
* [APPSeCONNECT, insync](https://www.appseconnect.com/sap-business-one-and-magento-integration/)

### Extension installation
* [How To Install/Uninstall Magento 2 Extensions (Detailed Examples)](https://bsscommerce.com/blog/how-to-install-extension-in-magento-2/#I_Install_Magento_2_Extension)

### Theme
* [Porto | Ultimate Responsive Magento Theme](https://themeforest.net/item/porto-ultimate-responsive-magento-theme/9725864)




