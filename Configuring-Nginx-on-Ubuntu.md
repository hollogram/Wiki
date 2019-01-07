* change the group ownership of the **storage** and **bootstrap/cache** directories to www-data
> sudo chgrp -R www-data storage bootstrap/cache
* grant all permissions, including write and execute, to the group
> sudo chmod -R ug+rwx storage bootstrap/cache
* adjust site config in nginx
  * change document root to public
  * change server name
  * adjust php cgi path

```Nginx
server {
    listen 80;
    listen [::]:80;

    . . .

    root /var/www/html/public;
    index index.php index.html index.htm index.nginx-debian.html;

    server_name example.com www.example.com;

    location / {
        try_files $uri /index.php?$query_string;
    }
	
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        # With php-cgi (or other tcp sockets):
        # fastcgi_pass 127.0.0.1:9000;
    }

    location ~ /\.ht {
        deny all;
    }

    . . .
}
```

* check config for errors
> sudo nginx -t
* reload Nginx to take the changes into account
> sudo systemctl reload nginx