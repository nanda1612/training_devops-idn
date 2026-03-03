# SonarQube 

## Prerequisite SonarQube Server
### Requirement 
Sonarqube: versi sonarqube-9.9.1<br>
Java: versi 17<br>
Postgresql: versi 15<br>

### Spesifikasi Server
Operating System: Ubuntu<br>
CPU: 4 Core<br>
Memory: 8 GB<br>
Disk: 30 GB<br>

### Java Installation
Install package java
```
apt install openjdk-17-jdk openjdk-17-jre -y
sudo update-alternatives --config java
```

### Database Installation
Install package postgresql
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get -y install postgresql
```
### Tuning
Tuning system sonarqube
```
vim /etc/sysctl.d/99-sonarqube.conf
```
edit
```
vm.max_map_count=524288
fs.file-max=131072
```
Tuning limit
```
ulimit -n 131072
ulimit -u 8192
```
Tuning limit permanently
```
vim /etc/security/limits.d/99-sonarqube.conf
```
edit
```
sonarqube   -   nofile   131072
sonarqube   -   nproc    8192
```
Load tuning configuration
```
sysctl --system
```

## Database

### Setup Database
Login to user postgres
```
su - postgres
```

Create new user
```
createuser sonar
```

create database and permission
```
psql
ALTER USER sonar WITH ENCRYPTED password 'sonar';
CREATE DATABASE sonarqube OWNER sonar;
grant all privileges on DATABASE sonarqube to sonar;
```

Verfikasi database and permission
```
\l
\du
```

Exit database
```
\q
```

exit from user postgres
```
exit
```

## Install Sonar Server
### Setup User
Create group sonar dan user sonar
```
groupadd sonar
useradd -s /bin/bash -d /opt/sonarqube -g sonar sonar
```

### Download Sonarqube
Download and extract sonarqube package
```
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.1.69595.zip
sudo apt install unzip -y
unzip sonarqube-9.9.1.69595.zip
```

Move sonarqube package and change ownership
```
mv sonarqube-9.9.1.69595 /opt/sonarqube
chown -R sonar:sonar /opt/sonarqube 
```

Create link sonarqube server
```
ln -s /opt/sonarqube/bin/linux-x86-64/sonar.sh /usr/bin/
```

### Config Sonarqube
Config sonar properties to use database and specific user
```
vim /opt/sonarqube/conf/sonar.properties
```

edit
```
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube
```

Config sonar shell script to spesific user
```
vim /opt/sonarqube/bin/linux-x86-64/sonar.sh
```

edit
```
RUN_AS_USER=sonar
```

### Start Sonarqube
Login to user sonar
```
sudo su sonar
```

Access folder sonarqube
```
cd /opt/sonarqube/bin/linux-x86-64/
```

Sonar shell script
```
./sonar.sh start
./sonar.sh status
./sonar.sh stop
```

Logging
```
tail -f /opt/sonarqube/logs/sonar.log
```

### Configure systemd service
Configure systemd to manage automatic sonar service
```
vim /etc/systemd/system/sonar.service
```

edit
```
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```

Manage service sonar
```
systemctl start sonar
systemctl enable sonar
systemctl status sonar
```

### Access Sonarqube
Access dashboard sonar server
```
http://server:9000
```


# Install Using Docker

### Install Docker
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

### Setup Database

```
su - postgres
psql

drop database sonarqube;
ALTER USER sonar WITH ENCRYPTED password 'sonar';
CREATE DATABASE sonarqube OWNER sonar;
grant all privileges on DATABASE sonarqube to sonar;
```
```
nano /etc/postgresql/18/main/postgresql.conf
```
Change this config:
```
listen_addresses = '*'
```
Buka hba postgresql
```
nano /etc/postgresql/18/main/pg_hba.conf
```
tambahkan di baris paling bawah
```
host    all             all             172.17.0.0/16            md5
```

Restart PostgreSQL
```
systemctl restart postgresql
```
### Deploy container
```
docker run -d --name sonarqube \
    -p 9000:9000 \
    -e SONAR_JDBC_URL=jdbc:postgresql://172.23.x.x/sonarqube \
    -e SONAR_JDBC_USERNAME=sonar \
    -e SONAR_JDBC_PASSWORD=sonar \
    -v sonarqube_data:/opt/sonarqube/data \
    -v sonarqube_extensions:/opt/sonarqube/extensions \
    -v sonarqube_logs:/opt/sonarqube/logs \
    --restart=always \
    sonarqube:community
```

### Access Sonarqube
Access dashboard sonar server
```
http://172.23.x.x:9000
```
