# Refinedstorage Docker Build Toolchain

## Requirements
* Ubuntu/Debian/CentOS
* Docker-CE
* docker-compose

### Install requirements

Docker-CE
```
wget -qO- https://get.docker.com/ | sh
```
Docker Compose
```
sudo curl -L https://github.com/docker/compose/releases/download/1.21.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
```

## First time setup

Checkout this git repository, any location will do.
First we have to start jenkins and run the setup in the interface

```
cd refinedstorage-docker
docker-compose up --build jenkins
```

Monitor the output and search for the following line and copy the password generated.

```
Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:
```

Goto http://yourserver:8080 and follow the setup instructions (choose default recommended plugins) and create and admin account.

### After steps
After the setup is complete login into jenkins.
We also need to install the following plugins:

 * Job Cacher plugin
 * embeddable-build-status
 
Goto `Manage Jenkins > Manage Plugins`

Click on the tab `Available` and select the checkboxes of the plugins required, click on `Install without restart`

### Slave Setup
Goto `Manage Jenkins > Manage Nodes` and click `New Node`

```
Name: slave
Permanent Agent: yes
```
Press `Save`

```
Remote root directory: /home/jenkins/agent
```
Keep everything else default, press `Save`

Now goto the `Manage nodes` page again, select `slave`.
Copy the secret hash from the java parameters.

```
cp env-example .env
```
Edit the `.env` file and fill in the `JENKINS_SECRET` (without quotes)

Now everything is prepared and we're able to start all the containers normally. Stop the currently running
docker-compose by sending `^c`.

Start all the containers by running:
```
docker-compose up -d
```

## Usage

All `docker-compose` commands should only be run in the directory containing `docker-compose.yml`

### Starting and Stopping

Start and detach:
```
docker-compose up -d
```

Stop:
```
docker-compose down
```

### Debugging

Start in the foreground:
``` 
docker-compose up
```

Attach to the logfiles:
```
docker-compose logs -f
```

Access a container:
`docker-compose exec <vmname> <command>`
 
```
docker-compose exec jenkins bash
docker-compose exec slave bash
```

### Updating

### Backup and restore

## Nginx configuration
```
server {
  listen 80; listen [::]:80;
  server_name jenkins.mydomain.nl;

  location /.well-known/acme-challenge/ {
    root /var/www;
  }

  location / {
    return 301 https://$host$request_uri;
  }
}

server {
  listen 443 ssl http2;  listen [::]:443 ssl http2;
  server_name jenkins.mydomain.nl;

  ssl on;
  ssl_certificate      /etc/letsencrypt/live/jenkins.mydomain.nl/fullchain.pem;
  ssl_certificate_key  /etc/letsencrypt/live/jenkins.mydomain.nl/privkey.pem;

  ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256';
  ssl_protocols TLSv1.2;
  ssl_prefer_server_ciphers on;
  ssl_session_cache shared:SSL:10m;

  add_header Strict-Transport-Security "max-age=63072000;";
  ssl_stapling on;
  ssl_stapling_verify on;

  client_max_body_size 0;

  location /errorpages/ {
    alias /var/www/errorpages/;
  }

  location / {
    error_page 502 =502 /errorpages/jenkins_offline.html;
    proxy_intercept_errors on;
    proxy_pass http://localhost:8080;
    proxy_set_header Host $http_host;
    proxy_http_version 1.1;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;
  }
}
```
