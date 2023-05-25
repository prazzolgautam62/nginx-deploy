
# Deploy Laravel Apps on Server (Nginx)  
Go to all projects folder
```bash
cd /var/www
```
Clone the repository
```bash
git clone https://github.com/prazzolgautam62/example_repo.git
```
Switch to the repo folder
```bash
cd example_repo
```
Install all the dependencies using composer
```bash
composer install
```
Copy the example env file and make the required configuration changes in the .env file
```bash
cp .env.example .env
```
Generate a new application key
```bash
php artisan key:generate
```

Run the database migrations (Set the database connection in .env before migrating)
```bash
php artisan migrate
```

Now we need to give the web server user write access to the storage and cache folders, where Laravel stores application-generated files:
```bash
sudo chown -R root:www-data storage
sudo chown -R root:www-data bootstrap/cache
```
```bash
sudo chmod -R 775 storage
sudo chmod -R 775 bootstrap/cache
```
The application files are now in order, but we still need to configure nginx to serve the content. To do this, we’ll create a new virtual host configuration file at /etc/nginx/sites-available:
```bash
sudo nano /etc/nginx/sites-available/example.com
```
Add the configuration
```bash

server {
        listen 80;
        listen [::]:80;

         root /var/www/example_repo/public;

        # Add index.php to the list if you are using PHP
        index index.php index.html index.htm index.nginx-debian.html;

        client_max_body_size 200M;
        server_name example.com;
        server_name www.example.com;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ /index.php?$query_string;
        }
         location ~ \.php$ {
                include snippets/fastcgi-php.conf;

                # With php-fpm (or other unix sockets):
                fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
                fastcgi_read_timeout 180;
                # With php-cgi (or other tcp sockets):
                # fastcgi_pass 127.0.0.1:9000;
        }

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #       deny all;
        #}
}
```

To activate the new virtual host configuration file, create a symbolic link to example.com in sites-enabled
```bash
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
```
To confirm that the configuration doesn’t contain any syntax errors, you can use:
```bash
sudo nginx -t
```
Output
```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
To apply the changes, reload nginx with:
```bash
service nginx restart
nginx -s reload
```
Now go to your browser and access the application using the server’s domain name or IP address, as defined by the server_name directive in your configuration file:
```bash
http://server_domain_or_IP
```
