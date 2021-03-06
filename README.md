## **Ubuntu 20.04 Server**

## Setup & Hardening

<details>
  <summary>Basic</summary>

## First start
```sh
apt update
apt upgrade

ufw allow OpenSSH
ufw enable
```

## Sudo User
```sh
adduser username
usermod -aG sudo username

(when key auth is enabled, copy key)
rsync --archive --chown=username:username ~/.ssh /home/username
```

## Key Auth Only
```sh
nano /etc/ssh/sshd_config

PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM yes

(if only sudo user should be allowed, WARNING: test login with sudo user before disabling)
PermitRootLogin no

service ssh restart
```

## Timezone
```sh
timedatectl set-timezone Europe/Berlin
```
</details>

<details>
  <summary>Hardening</summary>
  
## OpenSSH
### General
```sh
nano /etc/ssh/sshd_config
Add or change following lines if key auth is enabled:

HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ed25519_key

Protocol 2
LoginGraceTime 20
PermitRootLogin without-password
StrictModes yes
MaxAuthTries 3

PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM yes

AllowAgentForwarding no
AllowTcpForwarding no
X11Forwarding no
```

### Create new and more secure host keys

```sh
rm /etc/ssh/ssh_host_*

ssh-keygen -t rsa -b 4096 -f /etc/ssh/ssh_host_rsa_key -N ""

ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N ""
```

### Small Diffie-Hellman removal
```sh
awk '$5 >= 3071' /etc/ssh/moduli > /etc/ssh/moduli.safe

mv /etc/ssh/moduli.safe /etc/ssh/moduli
```

### Only secure and unexploitable Algorithms
```sh
nano /etc/ssh/sshd_config.d/ssh-hardening.conf
Add to the file:

KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512,diffie-hellman-group-exchange-sha256
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,umac-128-etm@openssh.com
HostKeyAlgorithms ssh-ed25519,ssh-ed25519-cert-v01@openssh.com,sk-ssh-ed25519@openssh.com,sk-ssh-ed25519-cert-v01@openssh.com,rsa-sha2-256,rsa-sha2-512,rsa-sha2-256-cert-v01@openssh.com,rsa-sha2-512-cert-v01@openssh.com
```
1. Test with: sshd -T
2. Restart with: service ssh restart

</details>

<details>
  <summary>Fail2ban</summary>
  
## Installation
  
  ```sh
apt install fail2ban

cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
nano /etc/fail2ban/jail.local

Change these lines:
[sshd]
enabled = true

bantime = 24h
findtime = 1h
maxretry = 3

service fail2ban restart
  ```
  
</details>

## Database

<details>
  <summary>MySQL</summary>

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
</details>
<details>
  <summary>MongoDB</summary>
    
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
</details>

## Webserver

<details>
  <summary>Nginx</summary>

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
</details>

<details>
  <summary>LetsEncrypt</summary>
    
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
</details>

## Languages

<details>
  <summary>Java</summary>
    
## Java
```sh
apt install default-jre
```
</details>

<details>
  <summary>PHP</summary>
    
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
server {
    listen 80;
    listen [::]:80;
    
    server_name example.com www.example.com;
    root /var/www/example.com;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }
}
```
</details>

## Software

<details>
  <summary>Mailcow Dockerized</summary>
    
## Mailcow Dockerized
Type | Host | Value
--- | --- | ---
A Record | mail | Server-IP
CNAME Record | autoconfig | mail.example.com
CNAME Record | autodiscover | mail.example.com
TXT Record | @ | v=spf1 mx ~all
TXT Record | _dmarc | v=DMARC1; p=reject; rua=mailto:mailauth-reports@example.com
MX Record | @ | mail.example.com (Priority 10)
PTR (Reverse DNS) | Server-IP | mail.example.com

### Mailcow Installation
```sh
apt install curl nano git apt-transport-https ca-certificates gnupg2 software-properties-common -y

wget -q https://download.docker.com/linux/ubuntu/gpg -O- | apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

apt update
apt install docker-ce docker-ce-cli -y

curl -L https://github.com/docker/compose/releases/download/$(curl -Ls https://www.servercow.de/docker-compose/latest.php)/docker-compose-$(uname -s)-$(uname -m) > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

cd /opt
git clone https://github.com/mailcow/mailcow-dockerized
cd mailcow-dockerized
./generate_config.sh

docker-compose pull
docker-compose up -d
```

### Mailcow Config
1. Navigate to mail.example.com
2. Login with admin:moohoo
3. Edit Administrator details and change password
4. Click Configuration in top menu > Mail Setup
5. Domains > Add domain and restart SOGo
6. Mailboxes > Add mailbox


### DKIM Config
1. Configuration > ARC/DKIM keys
2. Put in domain in input field
3. DKIM key length > 2048 bits
4. Click "Add"
5. Copy whole key starting with v=DKIM1;k=rsa;t=s;s=email;p=
6. Add following DNS Record

Type | Host | Value
--- | --- | ---
TXT Record | dkim._domainkey | v=DKIM1;k=rsa;t=s;s=email;p= [...]

7. Test the rating on mail-tester.com to not get added to blocklists

### Third Party Use
Type | Protocol | Hostname | Port | SSL | Auth
--- | --- | --- | --- | --- | ---
Incoming | IMAP | mail.example.com | 993 | SSL/TLS | Mailbox User
Incoming | POP3 | mail.example.com | 995 | SSL/TLS | Mailbox User
Outgoing | SMTP | mail.example.com | 587 | STARTTLS | Mailbox User
</details>
<details>
  <summary>Nextcloud</summary>

## Nextcloud with Nginx
### PHP Preconf
```sh
apt install php-imagick php7.4-common php7.4-mysql php7.4-fpm php7.4-gd php7.4-json php7.4-curl  php7.4-zip php7.4-xml php7.4-mbstring php7.4-bz2 php7.4-intl php7.4-bcmath php7.4-gmp

nano /etc/php/7.4/fpm/php.ini
memory_limit = 512M
upload_max_filesize = 512M
max_input_time = 1800
max_execution_time = 1800

nano /etc/php/7.4/fpm/pool.d/www.conf
clear_env = no

systemctl restart php7.4-fpm
```
### MySQL Database
```sql
mysql
CREATE DATABASE nextcloud;
CREATE USER nextcloud@localhost IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'localhost';
FLUSH PRIVILEGES;
exit
```

### Install Nextcloud
```sh
apt install wget unzip zip -y
cd /var/www/
wget https://download.nextcloud.com/server/releases/latest.zip
unzip latest.zip
sudo chown -R www-data:www-data /var/www/nextcloud
rm latest.zip

mkdir /usr/share/nginx/nextcloud-data
chown www-data:www-data /usr/share/nginx/nextcloud-data -R
```

### Nginx
[NGINX CONFIG](cloud.example.com.txt)
```sh
nano /etc/nginx/sites-available/cloud.example.com

(put in config from above and change server_name and root folder)

ln -s /etc/nginx/sites-available/cloud.example.com /etc/nginx/sites-enabled/
nginx -t 
systemctl restart nginx

certbot --nginx --redirect --staple-ocsp -d cloud.example.com
```

### HSTS
```sh
nano /etc/nginx/sites-available/cloud.example.com

listen 443 ssl http2; # managed by Certbot

(under listen and ssl) >
add_header Strict-Transport-Security "max-age=31536000" always;

nginx -t
systemctl reload nginx
```

```sh
ufw allow http
ufw allow https
```

### Finish Installation
1. Navigate to cloud.example.com
2. Input Admin Credentials
3. Folder: /usr/share/nginx/nextcloud-data
4. Previously created sql credentials
5. Untick the box at the bottom that recommends the installation of additional apps
6. Click "Finish Setup"

### Fix Missing Indexes
```sh
cd /var/www/nextcloud/
sudo -u www-data php occ db:add-missing-indices
```

</details>

## Other

<details>
  <summary>Commands</summary>
    
## Journal
```sh
journalctl -u service -n 500
```

## List open ports
```sh
netstat -tulpn
```

## Show Services
```sh
service --status-all
```
</details>

<details>
  <summary>Systemd Service</summary>
  
## Restartable Service
1. nano /etc/systemd/system/name.service
```ruby
[Unit]
Description=<description>
After=syslog.target

[Service]
User=<username>
ExecStart=/usr/bin/java -jar /home/username/server.jar
Restart=always
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=your-service
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```
2. systemctl enable name.service 
3. systemctl start name.service 
</details>
