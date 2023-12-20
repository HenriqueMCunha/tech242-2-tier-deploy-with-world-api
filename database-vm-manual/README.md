## Script for DB VM

```


#!/bin/bash

sudo apt update -y

sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y

#Assign value to password
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password root'
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password root'

#Create environment variables
export DB_USER="root"
export DB_PASS="root"
DB_NAME="world"
SQL_FILE="/home/ubuntu/world.sql"
sudo apt-get update

# Install MySQL
sudo DEBIAN_FRONTEND=noninteractive apt install -y mysql-server

sudo systemctl start mysql

sudo systemctl enable mysql

# get sql script file from github

wget https://raw.githubusercontent.com/HenriqueMCunha/tech242-tier-2-deployment-code/main/world.sql

# Create database world if it does not exist
mysql -u"${DB_USER}" -p"${DB_PASS}" < "${SQL_FILE}"

#Create backup for mysqld.cnf
sudo cp /etc/mysql/mysql.conf.d/mysqld.cnf /etc/mysql/mysql.conf.d/mysqld.cnf.backup

#configure bind address

if grep -q 'bind-address            = 0.0.0.0' /etc/mysql/mysql.conf.d/mysqld.cnf; then
        echo " Bind address already configured."
else
        echo "Configuring bind address..."
        sudo sed -i 's/bind-address\s*=\s*127.0.0.1/bind-address = 0.0.0.0/' /etc/mysql/mysql.conf.d/mysqld.cnf
fi

```