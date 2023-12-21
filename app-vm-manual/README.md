# Script APP VM Manual


## Planning and Manual Testing

### Planning

At first it was necessary to plan what course of action I was going to take, for that I used comments in a notepad which I will highlight below:

```
#!/bin/bash
#Update and Upgrade VM

# Install Maven

# Install java 17

# Install apache and configure reverse proxy

# Set environment variables (not necessary at this point in time, but I felt like it facilitated the process)

# Clone repository from github

# If connected to database, run maven (recompile code first)



```

### Testing

Once planning was done, the commands were set and tested manually before recording them to my notepad (if working).

#### Blockers (Testing)

* Ran into a problem with recompilation of target folder and kept testing until realizing the problem was not having the DB_HOST environment variable.

### Assemble Script to get ready for automation

```
#!/bin/bash
 
# Update VM
sudo apt update -y
echo "Update Done!"
echo ""
 
# Upgrade VM
sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y
echo "Update Done!"
echo ""
 
# install maven
sudo DEBIAN_FRONTEND=noninteractive apt install maven -y
echo "Maven Install Done!"
echo ""
 
# install JDK (java) 17
sudo DEBIAN_FRONTEND=noninteractive apt install openjdk-17-jdk -y
echo "Installed JDK Done!"
echo ""
 
# Set environment variables
export DB_USER=root
export DB_PASS=root
export DB_HOST=jdbc:mysql://172.31.39.111:3306/world
export DB_PORT=3306
 
# clone from git
rm -rf repo
git clone https://github.com/HenriqueMCunha/tech242-tier-2-deployment-code.git repo
 
# move into repository
cd repo/WorldProject
 
output=$(mysql -h 172.31.39.111 -u root -proot -e "USE world; SELECT 1" 2>&1)
 
if [ $? -eq 0 ]; then
    echo "Connected Successfully"
    mvn clean package spring-boot:start
else
    echo "Failed to connect"
    echo $output
fi
 
# Install apache2
sudo DEBIAN_FRONTEND=noninteractive apt install apache2 -y
 
# Start apache and enable
sudo systemctl start apache2
sudo systemctl enable apache2
 
# Enable mods
sudo a2enmod proxy
sudo a2enmod proxy_http
 
# Change file
if grep -q 'ProxyPass / http://localhost:5000/' /etc/apache2/sites-available/000-default.conf; then
    # The string exists, so nothing to do
    echo "Reverse proxy already configured."
else
    echo "Configuring reverse proxy..."
    sudo sed -i '/DocumentRoot \/var\/www\/html/ a\ProxyPreserveHost On\nProxyPass \/ http:\/\/localhost:5000\/\nProxyPassReverse \/ http:\/\/localhost:5000\/\n' /etc/apache2/sites-available/000-default.conf
fi
 
# Restart apache2
sudo systemctl reload apache2

sudo systemctl restart apache2
```