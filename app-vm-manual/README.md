# Script APP VM Manual


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
 
# recompile the app
cd repo
cd WorldProject
 
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