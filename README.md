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
cd refiendstorage-docker
docker-compose up --build jenkins
```

Monitor the output and search for the following line and copy the password generated.

```
Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:
```

Goto http://yourserver:8080 and follow the setup instructions (choose default recommended plugins) and create and admin account.

After the setup is complete login into jenkins.

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

Now goto the Manage nodes page again, select slave.
Copy the secret hash from the java parameters.

```
cp env-example .env
```
Edit the `.env` file and fill in the `JENKINS_SECRET` (without quotes)
