# Database User Data VM

# Initial Script

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

## Blockers

* When transfering to user data, I realized there was a problem with the SQL_FILE variable.
* When the world.sql file is created by root that file will not be correct so there was no need to have that variable with the path there.
* Because of that, the database was not being created.
* Once it was removed, the vm was working.

## Final User Data Script

```

#!/bin/bash

sudo apt update -y

sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y

sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password root'
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password root'

export DB_USER=root
export DB_PASS=root
DB_NAME=world
sudo apt-get update
sudo DEBIAN_FRONTEND=noninteractive apt install -y mysql-server

sudo systemctl start mysql

sudo systemctl enable mysql

wget https://raw.githubusercontent.com/HenriqueMCunha/tech242-tier-2-deployment-code/main/world.sql

mysql -u"${DB_USER}" -p"${DB_PASS}" < world.sql

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