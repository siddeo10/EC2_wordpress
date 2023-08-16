This is instructions to install Wordpress on Ubuntu 22.04

Topics Covered:
- Update package index on Linux Ubuntu 22.04LTS
- Install PHP
- Check the installed PHP version
- Download Wordpress
- Give permissions to Wordpress folder
- Create new database in MySQL
- Add a new MySQL database user with full privileges
- Give full privileges to newly added users in MySQL DB
- Configure NGINX
- Install Wordpress by installer
- Install SSL certificates to secure website
- Test Wordpress website

Wordpress install
- Update packages:
sudo apt update && sudo apt upgrade -y

- Install PHP:
sudo apt install -y nginx php-dom php-simplexml php-ssh2 php-xml php-xmlreader php-curl php-exif php-ftp php-gd php-iconv php-imagick php-json php-mbstring php-posix php-sockets php-tokenizer php-fpm php-mysql php-gmp php-intl php-cli

- Check version
php --version

Configure PHP:
Path:  sudo nano /etc/php/*/fpm/php.ini

upload_max_filesize = 200M
post_max_filesize = 500M
memory_limit = 512M
cgi.fix_pathinfo = 0
max_execution_time = 360

Save the file, start and enable PHP-FPM.
sudo systemctl restart php*-fpm.service

Verify if the service is running:
systemctl status php*-fpm.service
========================================
Download WordPress
wget https://wordpress.org/latest.tar.gz


Extract the file and move it to the /var/www directory
tar -xvzf latest.tar.gz
sudo mv wordpress /var/www/wordpress

Correct permissions for the directory:
sudo chown -R www-data:www-data /var/www/wordpress/
sudo chmod -R 755 /var/www/wordpress/

=========================================
Database

CREATE DATABASE dbname;
CREATE USER 'username’@‘localhost' IDENTIFIED WITH mysql_native_password BY ‘yourpassword;
GRANT ALL ON dbanme.* TO 'username’@‘localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT

=========================================
Configure Nginx Web server 
Path: sudo nano /etc/nginx/site-enabled/wordpress

server {
    listen 80;
    listen [::]:80;
    server_name  example.com www.example.com;
    root /var/www/wordpress;
    index  index.php index.html index.htm;
    access_log /var/log/nginx/wpress_access.log;
    error_log /var/log/nginx/wpress_error.log;

    client_max_body_size 100M;
    autoindex off;
    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
         include snippets/fastcgi-php.conf;
         fastcgi_pass unix:/var/run/php/php-fpm.sock;
         fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
         include fastcgi_params;
    }
}

Note - Remember to replace example.com with your own domain name and /var/run/php/php7.4-fpm.sock with /var/run/php/php8.0-fpm.sock for PHP 8.0. Save the file and exit.

Check the syntax of the file:
sudo nginx -t

Restart:
sudo systemctl restart nginx

=========================================
Access the WordPress Web Installer
Install Wordpress
=========================================
Step 1 — Installing Certbot
sudo snap install core; sudo snap refresh core

Remove if ready installed
sudo apt remove certbot

sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot

- Obtain certificate
sudo certbot --nginx -d example.com -d www.example.com

Step 3 — Allowing HTTPS Through the Firewall
sudo ufw status

Step 5 — Verifying Certbot Auto-Renewal
sudo systemctl status snap.certbot.renew.service



Optimizing an Nginx server configuration can significantly enhance website performance by implementing various techniques such as caching, gzip compression, and more. Below is a sample Nginx configuration that incorporates these optimizations. Please note that you should tailor these settings to your specific website's needs and test thoroughly before deploying them to a production environment.

use following configuration in /etc/nginx/sites-enabled

http {
    # Enable gzip compression
    gzip on;
    gzip_comp_level 5;
    gzip_min_length 256;
    gzip_proxied any;
    gzip_types text/plain text/css application/json application/javascript application/xml;
    gzip_vary on;

    # Enable browser cache for static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
        expires 1y;
        add_header Cache-Control "public, max-age=31536000";
    }

    # Set default cache path and options
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m;
    proxy_temp_path /var/tmp;

    # Enable caching for specific locations
    location / {
        proxy_pass http://backend_server;  # Replace with your backend server's URL
        proxy_set_header Host $host;
        proxy_cache my_cache;
        proxy_cache_valid 200 302 10m;  # Cache successful responses for 10 minutes
        proxy_cache_valid 404 1m;       # Cache 404 responses for 1 minute
        proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
        add_header X-Cached $upstream_cache_status;
    }


#Github actions workflow for automated deployment

use following actions.yaml


name: Push-Wordpress-website-to AWS EC2 server through github actions 

# Trigger deployment only on push to main branch
on:
  push:
    branches:
      - main

jobs:
  deploy:
    name: Deploy to EC2 on master branch push
    runs-on: ubuntu-latest
      
    steps:

      - name: Checkout the files
        uses: actions/checkout@v2

      - name: Deploy to Server 1
        uses: easingthemes/ssh-deploy@main
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}  #store your ssh key in your repository github actions secret
          REMOTE_HOST: 52.5.75.28          #your ec2 public ip (i have assigned elastic ip)
          REMOTE_USER: ubuntu              #user
          TARGET: /var/www/html/wordpress





