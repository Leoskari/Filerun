# Filerun 20220519 with Ubuntu 20.04 LTS LAMPS
### Installing Apache and Updating the Firewall

    sudo apt update && sudo apt upgrade -y
    sudo apt install apache2
    sudo ufw allow in "Apache"
    sudo ufw allow in "OpenSSH"
    sudo ufw enable
    sudo ufw status

### Installing MySQL

    sudo apt install mysql-server
    sudo mysql
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password by 'MYNEWPASSWORD';
    exit
    sudo mysql_secure_installation
    exit

### Setting Up FileRun's Database

    mysql -u root -p
    CREATE DATABASE filerun;
    CREATE USER 'filerun'@'localhost' IDENTIFIED BY 'YOUR-DB-PASSWORD';
    GRANT ALL ON filerun.* TO 'filerun'@'localhost';
    FLUSH PRIVILEGES;
    exit

### Installing PHP

    sudo apt install php libapache2-mod-php php-mysql
    php -v
    sudo vim /var/www/html/info.php
    
And add:  
    
    <?php
    phpinfo();
 
Go to:

- http://server_domain_or_IP/info.php

### Configuring PHP

    sudo apt-get install php-mbstring php-zip php-curl php-gd php-ldap php-xml php-imagick php-mysql

One last module which needs manual install is ionCube:

    sudo wget https://downloads.ioncube.com/loader_downloads/ioncube_loaders_lin_x86-64.tar.gz
    sudo tar -xzf ioncube_loaders_lin_x86-64.tar.gz -C /usr/lib/php
    sudo vim /etc/php/7.4/apache2/conf.d/00-ioncube.ini

add:

    zend_extension = /usr/lib/php/ioncube/ioncube_loader_lin_7.4.so

and: 

    sudo vim /etc/php/7.4/apache2/conf.d/filerun.ini

add: 

    expose_php              = Off
    error_reporting         = E_ALL & ~E_NOTICE
    display_errors          = Off
    display_startup_errors  = Off
    log_errors              = On
    error_log               = "/home/user/php_error.log"
    ignore_repeated_errors  = Off
    allow_url_fopen         = On
    allow_url_include       = Off
    variables_order         = "GPCS"
    allow_webdav_methods    = On
    memory_limit            = 128M
    max_execution_time      = 300
    output_buffering        = Off
    output_handler          = ""
    zlib.output_compression = Off
    zlib.output_handler     = ""
    safe_mode               = Off
    register_globals        = Off
    magic_quotes_gpc        = Off
    upload_max_filesize     = 20M
    post_max_size           = 20M
    enable_dl               = Off
    disable_functions       = ""
    disable_classes         = ""
    session.save_handler     = files
    session.use_cookies      = 1
    session.use_only_cookies = 1
    session.auto_start       = 0
    session.cookie_lifetime  = 0
    session.cookie_httponly  = 1
    date.timezone            = "Europe/Helsinki"

 - Note: You can find the latest FileRun recommended PHP settings here: https://docs.filerun.com/php_configuration

        sudo systemctl restart apache2

### Installing FileRun

    cd /var/www/html/
    sudo rm index.html
    sudo rm info.php
    sudo wget -O FileRun.zip https://filerun.com/download-latest-ubuntu-20
    sudo apt-get install unzip
    sudo unzip FileRun.zip
    sudo chown -R www-data:www-data /var/www/html/

### Securing the FileRun installation

The permissions of the FileRun application files should not allow PHP (or any other web server application) to make changes to them:

    sudo chown -R root:root /var/www/html

The system/data FileRun folder is the only folder where PHP needs write access.

    sudo chown -R www-data:www-data /var/www/html/system/data
    mysql -u root -p

 and:
 
    REVOKE ALTER, DROP ON filerun.* FROM 'filerun'@'localhost';
    FLUSH PRIVILEGES;
    exit

- Note: You will need to add these permissions back before you will be installing any FileRun software update in the future. To do that, connect again to the database server and runt the following commands:

        GRANT  ALTER, DROP ON filerun.* TO 'filerun'@'localhost';
        FLUSH PRIVILEGES;
        exit;

### Home folder and Unraid share

    sudo mkdir /files
    sudo chown www-data:www-data /files
    sudo vim /etc/fstab

add:

    unraid        /unraid            9p         trans=virtio,version=9p2000.L,_netdev,rw 0 0

and:

    sudo mount -a

### Installing ImageMagick and FFmpeg

    sudo apt-get install imagemagick
    sudo apt-get install ffmpeg

### Additional Apache Configuration
#### Enabling mod_rewrite

    sudo a2enmod rewrite
    sudo apachectl restart

#### Enabling .htaccess

    sudo vim /etc/apache2/apache2.conf

 add:
 
    <Directory /var/www/>
            Options Indexes FollowSymLinks
            AllowOverride All
            Require all granted
    </Directory>

Then:

    sudo systemctl restart apache2

### Grouping notifications

You can configure FileRun to send all the notifications for a certain time period in a single e-mail message. This helps preventing FileRun from sending hundreds of e-mail messages at a time, when users are uploading many files in a short time span.

To enable this you need to uncheck the option “Instant email notifications” available in “Control Panel”

    sudo su
    cd
    sudo cp /etc/php/7.4/apache2/conf.d/00-ioncube.ini /etc/php/7.4/cli/conf.d/
    vim auto_email.sh

 add:
 
    #bin/bash
    /usr/bin/php /var/www/html/cron/email_notifications.php filerun.leoskari.site

and:  

    crontab -e
    * 12 * * * bash /root/auto_email.sh

and:

    chmod +x auto_email.sh

### Server timezone

    timedatectl
    sudo timedatectl set-timezone Europe/Helsinki
 
### Apache tika index

- https://docs.filerun.com/file_indexing#install_apache_tika_command_line_mode

- https://www.apache.org/dyn/closer.lua/tika/2.6.0/tika-app-2.6.0.jar
    
        java -version
        sudo apt install openjdk-11-jre-headless
 
 - Control panel → Searching → Apache tika

add patch at control panel -> /home/leo//tika-app-2.6.0.jar
and test settings

        sudo su
        crontab -e
    
- Every day at 12:00

        * 12 * * * php /var/www/html/cron/process_search_index_queue.php

### Plugins
#### AES Crypt

    cd /home/leo/
    wget http://www.aescrypt.com/download/v3/linux/AESCrypt-GUI-3.11-Linux-x86_64-Install.gz
    gzip -d AESCrypt-GUI-3.11-Linux-x86_64-Install.gz
    chmod +x AESCrypt-GUI-3.11-Linux-x86_64-Install
    sudo ./AESCrypt-GUI-3.11-Linux-x86_64-Install

- Where do you want to install AES Crypt for Linux? [/usr/share/aescrypt] /user/bin/aescrypt
