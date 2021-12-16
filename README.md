## TOC
* [Linux and Web server setup under GCE](#linux-and-web-server-setup-under-gce)
* [MySQL](#mysql)
* [Elasticsearch](#elasticsearch)
* [Magento installation](#magento-installation)
* [Magento extension module installation](#magento-extension-module-installation)
  * [3rd-party extensions](#3rd-party-extensions) 
* [CLI - Command line interface](#command-line-interface)
* [Reference](#reference)


## Network overview
![](https://raw.githubusercontent.com/MRLIVING/magento/master/doc/img/overview_network.PNG)

## Linux and Web server setup under GCE
###
* `2vCPU` with `8GB` memory
* `Ubuntu 20.04 LTS` with `200 GB` capacity 

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
### Configuration for production mode
* [Virtual memory](https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html#vm-max-map-count)
* elasticsearch.yaml
  ```
  network.host: 0.0.0.0
  discovery.seed_hosts: []
  cluster.initial_master_nodes: []
  ```
  
### [Configure Commerce and Magento to use Elasticsearch](https://devdocs.magento.com/guides/v2.4/config-guide/elasticsearch/configure-magento.html)
```
Stores > Settings > Configuration > Catalog > Catalog > Catalog Search

Elasticsearch Server Hostname
Elasticsearch Server Port
Elasticsearch Index Prefix
```
  
### [Setup Chinese Analyzer](https://github.com/MRLIVING/M2-ESIKAnalyzer)

### Check and Start ES demon
#### Check ES demon
```
sudo su -l elk
jps

// You should see the message below
${PID} Elasticsearch
```

#### Start ES demon if you haven't seen the process via the [JPS](https://docs.oracle.com/javase/7/docs/technotes/tools/share/jps.html) command
```
sudo su -l elk
cd elasticsearch-7.6.2/
./bin/elasticsearch -d

# check ES alive
curl localhost:9200/_state
```

## [Magento installation](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/zip_install.html)
* Add an user as [magento file system owner](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/file-sys-perms-over.html#magento-file-system-owner)
  ```
  sudo adduser <magento_user>
  sudo mkdir -p /var/www/html/magento2
  sudo chown -R <magento_user>:<magento_user> magento2
  ```

* Get the Magento package
  * [Build versions of compressed archive](https://magento.com/tech-resources/download), under **Archive (zip/tar)** section.
    * [Extract the software on your server](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/zip_install.html#zip-extract)
      ```            
      sudo cp <src path>/magento-ce-2.4.3-p1-2021-09-23-07-56-34.zip /var/www/html/magento2/      
      
      sudo su -l <web server docroot>
      cd /var/www/html/magento2
      unzip magento-ce-2.4.3-p1-2021-09-23-07-56-34.zip
      ```
      
  * or [Composer](https://devdocs.magento.com/guides/v2.4/install-gde/composer.html#get-the-metapackage)  
    * [Install Latest Composer](https://getcomposer.org/download/)    
    
    * Create a new Composer project (get the Magento software metapackage)
      ```
      sudo su -l <web server docroot>
      cd /var/www/html/magento2      
      composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition=2.4.3-p1 .
      ```      
      * [Get your authentication keys](https://devdocs.magento.com/guides/v2.3/install-gde/prereq/connect-auth.html)

    * Deprecated, Install Composer v1.x
      ```
      curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer
      composer self-update 1.10.16    # downgrade it back to version 1.x due to a recent incompatibility issue with one of the packages.
      ```     


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

### [Deploy static view files](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-static-view.html)
TODO ...

### [Install sample data after Magento](https://devdocs.magento.com/guides/v2.4/install-gde/install/sample-data-after-magento.html)
* [Deploy Sample Data from GitHub Repository](https://github.com/magento/magento2-sample-data#deploy-sample-data-from-github-repository)


## Magento extension module installation
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

* [Magento 2 Currency Formatter extension](https://www.mageplaza.com/magento-2-currency-formatter/)  
  ```
  composer require mageplaza/module-currency-formatter
  php bin/magento setup:upgrade
  php bin/magento setup:static-content:deploy
  ```

* [Order Editor](https://support.mageworx.com/manuals/order-editor/#requirements-and-installation)
  * prerequisites  
    ```
    composer require matomo/device-detector  
    magento setup:upgrade
    ```
 
  * download & unzip the package to ${M2_Base}/app/code/MageWorx

  * enable modules  
    ```
    ./bin/magento module:enable MageWorx_Info MageWorx_OrdersBase MageWorx_OrderEditor
    ./bin/magento setup:upgrade
    ```

* [Magento 2 PDF Customizer](https://www.magezon.com/magento-2-pdf-customizer.html)
  * prerequisites 
    * [mPDF](https://github.com/mpdf/mpdf) 
      ```      
      composer require mpdf/mpdf
      ```

* [Facebook Business Extension](https://marketplace.magento.com/facebook-facebook-for-magento2.html)
  * [install guide](https://marketplace.magento.com/media/catalog/product/facebook-facebook-for-magento2-1-4-2-ce/installation_guides.pdf)
  * 注意事項: 記得要照安裝手冊中的把 php-business-sdk 裝起來。
  
## Command line interface

* `magento -h`

* `magento admin:user:create`, create an new admin user, see [Create or edit an administrator](https://devdocs.magento.com/guides/v2.4/install-gde/install/cli/install-cli-subcommands-admin.html#create-or-edit-an-administrator)

### [About application modes](https://devdocs.magento.com/guides/v2.4/config-guide/bootstrap/magento-modes.html)
* `magento deploy:mode:show`, see [Set the Magento mode](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-mode.html)
  * `magento deploy:mode:set ${mode}`, where ${mode} can be either `default`, `developer`, or `production`.

* `magento setup:di:compile`, see [Code compiler](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-compiler.html)

* `magento maintenance:status`, see [Enable or disable maintenance mode](https://devdocs.magento.com/guides/v2.4/install-gde/install/cli/install-cli-subcommands-maint.html)
  *  `magento maintenance:disable`

* `magento config:show`, see [Display the value of configuration settings](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-config-mgmt-set.html#config-cli-config-show)
   * `magento config:show | grep 'base_url'`
  
* `magento setup:store-config:set`, see [Configure the store](https://devdocs.magento.com/guides/v2.4/install-gde/install/cli/install-cli-subcommands-store.html#instgde-cli-storeconfig).
   * `magento setup:store-config:set --base-url="http://${Host-IP}/"`   
   * `magento setup:store-config:set --base-url-secure="https://${Host-IP}/"`
   * `magento config:set admin/url/use_custom 0`, revert to the default 

* [Export the configuration](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-config-mgmt-export.html)  
  First, please backup `app/etc/config.php` `and app/etc/env.php`; otherwise, the original files will be overwritten.  
  `bin/magento app:config:dump`

* [list all enabled/disabled modules](https://devdocs.magento.com/guides/v2.4/install-gde/install/cli/install-cli-subcommands-enable.html#instgde-cli-subcommands-status)  
  `magento module:status`
  
* [Enable or disable modules](https://devdocs.magento.com/guides/v2.4/install-gde/install/cli/install-cli-subcommands-enable.html)  
  `magento module:disable Magento_TwoFactorAuth`
  
* [Uninstall a module](https://devdocs.magento.com/guides/v2.4/install-gde/install/cli/install-cli-uninstall-mods.html#instgde-cli-uninst-mod-uninst)  
  `bin/magento module:uninstall ${ModuleName}`  
  , where `${ModuleName}` specifies the module name in `<VendorName>_<ModuleName>` format
 
* `bin/magento cache:status`,  see [View the cache status](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-cache.html#view-the-cache-status)

### [Clear directories during development](https://devdocs.magento.com/guides/v2.4/howdoi/php/php_clear-dirs.html)
* `bin/magento c:f`, cache:flush see [Clean and flush cache types](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-cache.html#config-cli-subcommands-cache-clean)

* `./bin/magento indexer:status`, see [View indexer status](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-index.html#view-indexer-status)
* `./bin/magento indexer:reindex customer_grid`, see [Manage the indexers](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-index.html)

* `bin/magento catalog:image:resize`, see [Resize catalog images](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/themes/theme-images.html#resize-catalog-images)


## [Upgrade Magento](https://devdocs.magento.com/guides/v2.4/comp-mgr/cli/cli-upgrade.html#manage-packages)
### Install the Composer update plugin
`composer require magento/composer-root-update-plugin=~1.0 --no-update`
TODO ...


## [Web API](https://devdocs.magento.com/guides/v2.4/get-started/bk-get-started-api.html)
* [Token-based authentication](https://devdocs.magento.com/guides/v2.4/get-started/authentication/gs-authentication-token.html)
TODO ...


## Setup GCP CDN (Alpha)
### Change the base URLs for `Static View Files` and `User Media Files`
* Disable [Static content signing](https://devdocs.magento.com/guides/v2.4/config-guide/cache/static-content-signing.html)
  * see [ref](https://magento.stackexchange.com/questions/167278/where-do-i-point-my-secure-base-url-for-static-view-files-for-cdn-in-magento-2)

* Pre-generate all necessary resizes `magento catalog:image:resize` 
  * see [Regenerate catalog cache images issues](https://magento.stackexchange.com/questions/175224/regenerate-catalog-cache-images-issues)

* [Using a Custom Admin URL](https://docs.magento.com/user-guide/stores/store-urls-custom-admin.html)
* 

## [Variables and configuration paths](https://devdocs.magento.com/guides/v2.4/config-guide/prod/config-reference-sens.html#general-category-sensitive-and-system-specific-paths)


## [phpMyAdmin for MYSQL](https://github.com/MRLIVING/magento/wiki/phpMyAdmin-on-GCE-with-Cloud-MySQL)


## Reference
* [Magento 2 Developer Guide](https://devdocs.magento.com/)
* [How to Create a Module in Magento 2](https://devdocs.magento.com/videos/fundamentals/create-a-new-module)
  * [How to Create Controller](https://www.mageplaza.com/magento-2-module-development/how-to-create-controllers-magento-2.html)
  * [Create View: Block, Layouts, Templates](https://www.mageplaza.com/magento-2-module-development/view-block-layout-template-magento-2.html#step-3-create-block)
  * [Return a JSON Response from Controller](https://meetanshi.com/blog/return-json-response-from-controller-in-magento-2/)
* [Custom Web API for Magento 2](https://inchoo.net/magento-2/magento-2-custom-api/)
* [Layout instructions](https://devdocs.magento.com/guides/v2.3/frontend-dev-guide/layouts/xml-instructions.html)
* [How to use Plugin, Preference to rewrite Block, Model, Controller, Helper in Magento 2](https://www.mageplaza.com/devdocs/how-use-plugin-preference-rewrite-block-model-controller-helper-magento-2.html)
* [Plugin - Interceptor](https://www.mageplaza.com/magento-2-module-development/magento-2-plugin-interceptor.html)
* [Virtual Types, Types, Preferences: Magento 2 Design Patterns](https://magently.com/blog/magento-2-design-patterns-preferences-virtual-types/)
  * [What is the difference between type and virtualType](https://magento.stackexchange.com/questions/33103/what-is-the-difference-between-type-and-virtualtype)

### Extension
* [How To Install/Uninstall Magento 2 Extensions (Detailed Examples)](https://bsscommerce.com/blog/how-to-install-extension-in-magento-2/#I_Install_Magento_2_Extension)
* [Facebook Business Extension](https://marketplace.magento.com/facebook-facebook-for-magento2.html)
* [Magezon Page Builder extension, community](https://www.magezon.com/magezon-page-builder-for-magento-2.html)
* [Amasty - Shipping Rules](https://amasty.com/shipping-rules-for-magento-2.html)
* payment getway  
  * [Stripe Payments](https://marketplace.magento.com/stripe-stripe-payments.html)
  * [PayU module for Magento 2 version 2.4](https://github.com/PayU-EMEA/plugin_magento_24)  

### Theme
* [Porto | Ultimate Responsive Magento Theme](https://themeforest.net/item/porto-ultimate-responsive-magento-theme/9725864)
* [Learning Page Builder](https://docs.magento.com/user-guide/cms/page-builder-learn.html)

### Connector, [evaluation](https://docs.google.com/spreadsheets/d/1i0igXkdmvaEnmVIAXKGaDJ4-dELUSeVPWHXuPT6NaEg/edit?usp=sharing)
* [SAP Business One Connect, i95Dev](https://marketplace.magento.com/i95dev-i95devsapconnect.html)
* [SAP Business One Integration Add-on for Magento 2, Firebear](https://firebearstudio.com/sap-business-one-integration-add-on-for-magento-2.html)
* [APPSeCONNECT, insync](https://www.appseconnect.com/sap-business-one-and-magento-integration/)




