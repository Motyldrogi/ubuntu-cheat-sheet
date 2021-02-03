# ubuntu-cheat-sheet

Some Ubuntu 20.04 setup commands

## Basic Conf
apt update<br>
apt upgrade<br>
adduser username<br>
usermod -aG sudo username<br>
ufw allow OpenSSH<br>
ufw enable

## MySQL
apt install mysql-server<br>
mysql_secure_installation

mysql<br>
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';<br><br>
GRANT CREATE, ALTER, DROP, INSERT, UPDATE, DELETE, SELECT, REFERENCES, RELOAD on &ast;.&ast; TO 'username'@'localhost' WITH GRANT OPTION;<br><br>
FLUSH PRIVILEGES;<br>
exit

systemctl enable mysql

### MySQL Test
mysqladmin -p -u username version

## MongoDB
curl -fsSL https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -<br><br>
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list<br><br>
apt update<br>
apt install mongodb-org<br>
systemctl start mongod.service<br>
systemctl enable mongod

### MongoDB Test
mongo --eval 'db.runCommand({ connectionStatus: 1 })'

### MongoDB Secure
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

security:<br>
  authorization: "enabled"

systemctl restart mongod

## Nginx
apt install nginx
ufw allow 'Nginx Full'
systemctl enable nginx

### Enable Nginx Conf
ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/

## Java
apt install default-jre

## Timezone
timedatectl set-timezone Europe/Berlin

## Journal
journalctl -u service -n 300


## Nginx Conf Folder
server {
  listen 80;
  listen [::]:80;

  root /var/www/example.com;
  server_name example.com www.example.com;
	index index.html index.htm;
}

### Redirect to Https and Non-www
if ($host = www.example.com) {
  return 301 https://example.com$request_uri;
}


## Nginx Conf Reverse Proxy
server {<br>
        listen 80;<br>
        listen [::]:80;<br>
        server_name sub.example.com;<br><br>
        
   location / {<br>
             proxy_pass http://127.0.0.1:8080;<br>
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;<br>
             proxy_set_header X-Forwarded-Proto $scheme;<br>
             proxy_set_header X-Forwarded-Port $server_port;<br>
    }<br>
}



## LetsEncrypt
apt install certbot python3-certbot-nginx
certbot --nginx -d example.com -d www.example.com

### Test LetsEncrypt
systemctl status certbot.timer
certbot renew --dry-run


## PHP
apt-get install php-fpm php-mysql

### PHP Secure
nano /etc/php/7.4/fpm/php.ini

Change:
;cgi.fix_pathinfo=1
to
cgi.fix_pathinfo=0

systemctl restart php7.4-fpm

### PHP Nginx
location / {
  try_files $uri $uri/ =404;
}

location ~ \.php$ {
  include snippets/fastcgi-php.conf;
  fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
}
