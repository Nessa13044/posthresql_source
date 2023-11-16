https://nestcode.co/en/blog/how-to-install-postgresql-14-on-ubuntu-20-04-from-source

# STEP 1: Download Source from PostgreSQL website
wget https://ftp.postgresql.org/pub/source/v14.0/postgresql-14.0.tar.gz

# STEP 2: Install Prerequisite Packages
sudo apt-get update
sudo apt install gcc
sudo apt install libreadline-dev
sudo apt install zlib1g-dev

# STEP 3: Server Preparation

#Create a group "dba"

- sudo groupadd -g 54321 dba

#Create a user "postgres" in group "dba"

- sudo useradd -u 54321 -g dba postgres

#Giving password to user "postgres" (Let use Pgs123456 as password)

- sudo passwd postgres

Next, we have to create a home directory of the user postgres as the following command

#Create home directory of user postgres

- sudo mkdir /home/postgres

- sudo chown -R postgres:dba /home/postgres/

#Setting permissions 

- sudo chmod 755 /home/postgres

#initialization

- sudo cp -a /etc/skel/. /home/postgres

#change from default from shell to bash

- sudo usermod -s /bin/bash postgres

#allow postgres to use sudo

- sudo usermod -aG sudo postgres

#checking sudo permission

- groups postgres

#switching to user postgres

- su - postgres

Create data directory

#Create directory for postgresql software to be installed on

- sudo mkdir -p /app/postgres/product/14/db_1/

- sudo chown -R postgres:dba /app/postgres/product/14/db_1/

#Create directory for data store

- sudo mkdir -p /data01

- sudo chown -R postgres:dba /data01

Create Archive directory

- sudo mkdir -p /reco

- sudo chown -R postgres:dba /reco

Create Backup director

- sudo mkdir -p /backup

- sudo chown -R postgres:dba /backup

# STEP 4: User "postgres" Profile Preparation
su - postgres
sudo vi ~/.profile

#!/bin/sh
# The script sets environment variables helpful for PostgreSQL

export PGHOME=/app/postgres/product/14/db_1

export PATH=$PGHOME/bin:$PATH

export PGDATA=/data01

export PGDATABASE=postgres

export PGUSER=postgres

export PGPORT=5432

export PGLOCALEDIR=$PGHOME/share/locale

export MANPATH=$MANPATH:$PGHOME/share/man


#Reload the configuration
- source ~/.profile
# STEP 5: Installation of PostgreSQL 14
- tar -zxvf postgresql-14.0.tar.gz
- cd postgresql-14.0
#checking possible issue by installing this PostgreSQL 14
- ./configure --prefix=/app/postgres/product/14/db_1/
#install make command if it's not yet install on your server
- sudo apt install make
#now let's start installing PostgreSQL 14
- make
- make world
- make install

- make install-docs
- make install-world
- make -C src/bin install
- make -C src/include install
- make -C src/interfaces install
- make -C doc install

It is where your data is really stored by using command

- $PGHOME/bin/initdb -D $PGDATA

$PGHOME and $PGDATA are the parameters you have been defined in postgres user profile. 

# STEP 6: Configuration of PostgreSQL Service

sudo vi /etc/systemd/system/postgresql.service

[Unit]

Description=PostgreSQL database server

After=network.target

[Service]

Type=forking

##Add a service section and set the max number of open files

LimitNOFILE=9999999

User=postgres

#Group=postgres

#Maximum number of seconds pg_ctl will wait for postgres to start.  Note that

#PGSTARTTIMEOUT should be less than TimeoutSec value.

Environment=PGSTARTTIMEOUT=270

Environment=PGDATA=/data01

PIDFILE=/data01/postmaster.pid

ExecStart=/app/postgres/product/14/db_1/bin/pg_ctl start -w -D "/data01" -l "/data01/log/startup.log"

ExecStop=/app/postgres/product/14/db_1/bin/pg_ctl stop -m fast -w -D "/data01"

ExecReload=/app/postgres/product/14/db_1/bin/pg_ctl reload -D "/data01"

#Give a reasonable amount of time for the server to start up/shut down.

#Ideally, the timeout for starting PostgreSQL server should be handled more

#nicely by pg_ctl in ExecStart, so keep its timeout smaller than this value.

TimeoutSec=300

[Install]

WantedBy=multi-user.target

# Create a log file in order to start up the PostgreSQL service.

cd /data01

sudo mkdir log

cd /data01/log

sudo vi startup.log

sudo chown -R postgres:dba startup.log

- boot
sudo systemctl enable postgresql

sudo systemctl start postgresql

sudo systemctl status postgresql

# STEP 7: Final Check


![image](https://github.com/Nessa13044/posthresql_source/assets/114730329/ed4a7da2-7a83-47f4-b605-5e1ceec00c85)
