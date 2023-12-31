# Script for DB VM Automated

## First Try 

## User Root not being properly created, will have to update this script


```


#!/bin/bash

sudo apt update -y

sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y

#Assign value to password
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password root'
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password root'

#Create environment variables
export DB_USER=root
export DB_PASS=root
DB_NAME=world
SQL_FILE=/home/ubuntu/world.sql
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


### Blockers

When automating the database script, I found a particular blocker derived from the last command in the script to create a root user and grant it all privileges.

  #### Before changes

  * sudo mysql -u root -proot -e "CREATE USER 'root'@'%' IDENTIFIED BY 'root'; GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION; FLUSH PRIVILEGES;"

The reason for that was the fact that the command I had previously did not specify whether the user existed or not, which was leading to an error 503 in browser.

  #### After Changes

  * sudo mysql -u root -proot -e "CREATE USER IF NOT EXISTS 'root'@'%' IDENTIFIED BY 'root'; GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION; FLUSH PRIVILEGES;"


## Second script

```

#!/bin/bash

sudo apt update -y

sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y

sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password root'
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password root'

export DB_USER=root
export DB_PASS=root
DB_NAME=world
SQL_FILE=/home/ubuntu/world.sql
sudo apt-get update
sudo DEBIAN_FRONTEND=noninteractive apt install -y mysql-server

sudo systemctl start mysql

sudo systemctl enable mysql

wget https://raw.githubusercontent.com/HenriqueMCunha/tech242-tier-2-deployment-code/main/world.sql

mysql -u"${DB_USER}" -p"${DB_PASS}" < "${SQL_FILE}"

sudo cp /etc/mysql/mysql.conf.d/mysqld.cnf /etc/mysql/mysql.conf.d/mysqld.cnf.backup

#configure bind address

if grep -q 'bind-address            = 0.0.0.0' /etc/mysql/mysql.conf.d/mysqld.cnf; then
        echo " Bind address already configured."
else
        echo "Configuring bind address..."
        sudo sed -i 's/bind-address\s*=\s*127.0.0.1/bind-address = 0.0.0.0/' /etc/mysql/mysql.conf.d/mysqld.cnf
fi

sudo mysql -u root -proot -e "CREATE USER IF NOT EXISTS 'root'@'%' IDENTIFIED BY 'root'; GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION; FLUSH PRIVILEGES;"

```