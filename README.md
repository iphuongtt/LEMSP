## Update System and install repolist
1. `sudo yum update`
2. `sudo yum install epel-release -y`
3. `sudo yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm -y`
4. `sudo yum repolist`
## Install Nginx
5. `sudo yum install nginx -y`
6. `sudo systemctl enable nginx`
7. `sudo systemctl start nginx`
## Configure and reload Firewall
8. `sudo firewall-cmd --permanent --zone=public --add-service=http`
9. `sudo firewall-cmd --permanent --zone=public --add-service=https`
10. `sudo firewall-cmd --reload`
11. Open browser and check connection
## Install Php 7
12. `sudo yum install yum-utils -y`
13. `sudo yum-config-manager --enable remi-php72`
14. `sudo yum --enablerepo=remi,remi-php71 install php-fpm php-common -y`
15. `sudo yum --enablerepo=remi,remi-php71 install php-opcache php-pecl-apcu php-cli php-pear php-pdo php-mysqlnd php-pgsql php-pecl-mongodb php-pecl-redis php-pecl-memcache php-pecl-memcached php-gd php-mbstring php-mcrypt php-xml php-pecl-zip -y`
16. `sudo systemctl enable php-fpm`
17. `sudo systemctl start php-fpm`
18. `sudo vi /etc/php.ini`
```
Set cgi.fix_pathinfo=0
```
19. `sudo vi /etc/php-fpm.d/www.conf`
```
user = apache to user = nginx
group = apache to group = nginx
listen.owner = nobody to listen.owner = nginx
listen.group = nobody to listen.group = nginx
listen = /var/run/php-fpm/php-fpm.sock
```
20. `sudo systemctl restart php-fpm`
## Configure nginx
21. `sudo vi /etc/nginx/nginx.conf`
```
include /etc/nginx/conf.d/*.conf;
server {
	include /etc/nginx/default.d/*.conf;
}
```
22. `sudo vi /etc/nginx/conf.d/default.conf`
```
server {
    listen   80;
    server_name  your_server_ip;    # note that these lines are originally from the "location /" block
    root   /var/www/html;
    index index.php index.html index.htm;    
    location / {
        try_files $uri $uri/ =404;
    }
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /var/www/html;
    }    
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```
23. `sudo systemctl restart nginx`
## Install MS ODBC
24. `cd ~`
25. `curl -O "ftp://ftp.unixodbc.org/pub/unixODBC/unixODBC-2.3.7.tar.gz"`
26. `tar -xz -f unixODBC-2.3.7.tar.gz`
27. `cd unixODBC-2.3.7/`
28. `sudo yum install gcc -y`
29. `./configure --prefix=/usr --libdir=/usr/lib64 --sysconfdir=/etc --enable-gui=no --enable-drivers=no --enable-iconv --with-iconv-char-enc=UTF8 --with-iconv-ucode-enc=UTF16LE`
30. `sudo make`
31. `sudo make install`
32. `cd /usr/lib64/`
33. `sudo ln -s libodbccr.so.2 libodbccr.so.1`
34. `sudo ln -s libodbcinst.so.2 libodbcinst.so.1`
35. `sudo ln -s libodbc.so.2 libodbc.so.1`
36. `ls -l /usr/lib64/libodbc*`
37. `odbc_config --version --longodbcversion --cflags --ulen --libs --odbcinstini --odbcini`
38. `isql --version`
39. `cd ~`
40. open this link on browser [https://www.microsoft.com/en-us/download/details.aspx?id=36437](https://www.microsoft.com/en-us/download/details.aspx?id=36437) and download driver
41. `curl -O "https://www.microsoft.com/en-us/download/confirmation.aspx?id=36434&6B49FDFB-8E5B-4B07-BC31-15695C5A2143=1"`
42. `tar -xz -f msodbcsql-11.0.2270.0.tar.gz`
43. `cd msodbcsql-11.0.2270.0`
44. `sudo ./install.sh install --accept-license --force`
45. `ls -l /opt/microsoft/msodbcsql/lib64/`
46. `dltest /opt/microsoft/msodbcsql/lib64/libmsodbcsql-11.0.so.2270.0 SQLGetInstalledDrivers`
47. `cd /home`
48. `sudo vi template_dsn.ini`
```
[MSODBC]
Driver = ODBC Driver 11 for SQL Server
Description = MS ODBC
Trace = No
Server = 192.168.1.192
```
49. `sudo odbcinst -i -s -f template_dsn.ini -l`
50. `cat /etc/odbc.ini`
51. `isql -v MSODBC test test`
## Install Composer
52. `sudo yum install php-odbc -y`
53. `php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"`
54. `sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer`
55. `composer`
## Install Laravel
56.  `cd /var/www/html/`
57. `composer create-project --prefer-dist laravel/laravel@5.7 blog`
## Setup vhost
58. `sudo vi /etc/nginx/conf.d/blog.conf`
```
server {
    listen   80;
    server_name myblog.com;
    root   /var/www/html/blog/public;
    index index.php index.html index.htm;
    location / {
        try_files $uri $uri/ /index.php?$args;
    }
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /var/www/html/blog/public;
    }
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
    location ~ /\.ht {
                deny all;
    }
}
```
59. `cd /var/www/html/blog`
60. `sudo chown -R nginx:nginx`
61. `sudo chmod -R 775 storage/`
62. `setenforce 0`
63. `ip addr`
64. add new line with this string to host file: `192.168.1.27 myblog.com`
65. `php artisan key:generate`
66. Enable Protocol TCP/IP of SQL Server and Set Port to 1433
66. `sudo vi config/database.php`
```
'sqlsrv' => [
            'driver' => 'sqlsrv',
            'host' => env('DB_HOST', '192.168.1.192'),
            'port' => env('DB_PORT', '1433'),
            'database' => env('DB_DATABASE', 'test'),
            'username' => env('DB_USERNAME', 'test'),
            'password' => env('DB_PASSWORD', 'test'),
            'charset' => 'utf8',
            'prefix' => '',
			'odbc_datasource_name' => 'Driver={ODBC Driver 11 for SQL Server};Server=192.168.1.192;Database=test',
    	    'odbc' => true,
        ],
```
67. `vi .env`
```
DB_CONNECTION=sqlsrv
DB_HOST=192.168.1.192
DB_PORT=1433
DB_DATABASE=test
DB_USERNAME=test
DB_PASSWORD=test
```
68. `php artisan migrate`
