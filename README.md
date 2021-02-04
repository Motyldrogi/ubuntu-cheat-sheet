# **Ubuntu 20.04 Setup Commands**

## Basic Conf
```sh
apt update
apt upgrade

adduser usernam
usermod -aG sudo username

ufw allow OpenSSH
ufw enable
```
## MySQL
```sh
apt install mysql-server
mysql_secure_installation

mysql
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
GRANT CREATE, ALTER, DROP, INSERT, UPDATE, DELETE, SELECT, REFERENCES, RELOAD on *.* TO 'username'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
exit

systemctl enable mysql
```
### MySQL Test
```sh
mysqladmin -p -u username version
```
## MongoDB
```sh
curl -fsSL https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
apt update
apt install mongodb-org

systemctl start mongod.service
systemctl enable mongod
```
### MongoDB Test
```sh
mongo --eval 'db.runCommand({ connectionStatus: 1 })'
```
### MongoDB Secure
```sh
mongo
use admin

db.createUser(
    {
        user: "username",
        pwd: passwordPrompt(),
        roles: [ { role: "root", db: "admin" } ]
    }
)

exit

nano /etc/mongod.conf

security:
  authorization: "enabled"

systemctl restart mongod
```
## Nginx
```sh
apt install nginx
ufw allow 'Nginx Full'
systemctl enable nginx
```
### Enable Nginx Conf
```sh
ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
```
## Java
```sh
apt install default-jre
```
## Timezone
```sh
timedatectl set-timezone Europe/Berlin
```
## Journal
```sh
journalctl -u service -n 300
```

## Nginx Conf Folder
```sh
server {
    listen 80;
    listen [::]:80;

    root /var/www/example.com;
    server_name example.com www.example.com;
    index index.html index.htm;
}
```
## Nginx Conf Reverse Proxy
```sh
server {
    listen 80;
    listen [::]:80;
    server_name sub.example.com;
        
    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Port $server_port;
    }
}
```
### Redirect to Https and Non-www
```sh
if ($host = www.example.com) {
    return 301 https://example.com$request_uri;
}
```

## LetsEncrypt
```sh
apt install certbot python3-certbot-nginx

certbot --nginx -d example.com -d www.example.com
```
### Test LetsEncrypt
```sh
systemctl status certbot.timer
certbot renew --dry-run
```

## PHP
```sh
apt-get install php-fpm php-mysql
```
### PHP Secure
```sh
nano /etc/php/7.4/fpm/php.ini

Change
;cgi.fix_pathinfo=1
to
cgi.fix_pathinfo=0

systemctl restart php7.4-fpm
```
### PHP Nginx
```sh
location / {
    try_files $uri $uri/ =404;
}

location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
}
```
