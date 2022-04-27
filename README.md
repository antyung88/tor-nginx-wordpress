# tor-nginx-wordpress
How To Install WordPress in tor

# Prerequisites
- Ubuntu 20.04 (Recommended)

# Nginx

```
sudo apt update
sudo apt install tor -y
sudo apt install nginx -y
```
```
sudo nano /etc/nginx/nginx.conf 
```

```
...
server_names_hash_bucket_size 125;
server_tokens off;
...
```

Reload Nginx

```
sudo nginx -t
sudo systemctl restart nginx
```

Proceed to configure a Hidden Service within Tor, using your editor open /etc/tor/torrc

```
sudo nano /etc/tor/torrc
```

```
HiddenServiceDir /var/lib/tor/nginx/
HiddenServicePort 80 127.0.0.1:80
```

Reload Tor, to generate your .onion address.

```
sudo systemctl reload tor
sudo cat /var/lib/tor/nginx/hostname
```

# WordPress

```
sudo apt install php7.4-fpm php7.4-common php7.4-mysql php7.4-gmp php7.4-curl php7.4-intl php7.4-mbstring php7.4-xmlrpc php7.4-gd php7.4-xml php7.4-cli php7.4-zip -y
```

```
sudo apt-get install mariadb-server mariadb-client -y
```

```
sudo mysql
CREATE DATABASE exampledb;
CREATE USER 'exampleuser'@'localhost' IDENTIFIED BY 'examplepass';
GRANT ALL ON exampledb.* TO 'exampleuser'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```

Install WordPress

```
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz
sudo mv wordpress /var/www/html/wordpress
```

Change ownership & permission for WordPress:

```
sudo chown -R www-data:www-data /var/www/wordpress/
sudo chmod -R 755 /var/www/wordpress/
```

```
sudo rm /etc/nginx/sites-available/default
sudo rm /etc/nginx/sites-enabled/default 
```

```
sudo nano /etc/nginx/sites-available/wordpress
```

```
server {
    listen 80;
    listen [::]:80;
    root /var/www/wordpress;
    index  index.php index.html index.htm;

    autoindex off;
    location / {
        try_files $uri $uri/ /index.php?$args;
    }
  
    location ~ \.php$ {
         include snippets/fastcgi-php.conf;
         fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
         fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
         include fastcgi_params;
         client_max_body_size 100M;
    }
}
```

```
sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/wordpress
```

```
sudo systemctl restart nginx
```
