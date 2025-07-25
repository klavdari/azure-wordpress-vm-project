# Deploying a WordPress Site on an Azure Linux VM

## Project Description


This project demonstrates the process of provisioning a cloud infrastructure from scratch on Microsoft Azure. I deployed a complete, functional WordPress blog by setting up a Linux virtual machine, configuring its network security, and installing a full LAMP stack (Linux, Apache, MySQL, PHP).

## Technology Used

- **Cloud Provider** : Microsoft Azure
- **Core Infastructure** : Azure Virtual Machines, Virtual Networks (VNet), Network Security Groups (NSGs), Public IP Addresses
- **Operating System** : Linux (Ubuntu 22.04 LTS)
- **Web Stack** : Apache2 (Web Server), MySQL (Database), PHP
- **Application** : WordPress (Content Management System)
- **Security** : Firewall rule configuration (SSH, HTTP, HTTPS), user/password administration
- **DNS & Domain Management**
- **SSL/TLS Security** : Let's Encrypt, Certbot
- **Web Server Configuration** : Apache Virtual Hosts
- **Platform as a Service (PaaS)** : Azure Database for MySQL
- **Cloud Architecture** : Multi-tier application design
- **Data Migration** : Exporting and importing a database ```mysqldump```
- **High availability & Load Balancing**
- **Scalability**
- **Shared Storage**

## Architecture Diagram
Here is the architecture of the final deployed application. It outlines the flow of traffic from the end-user through the Azure network components to the virtual machine hosting the WordPress site.

![Project Architecture Diagram](assets/MyWordPressApp-Fourth-Phase.jpg)

## Installation Steps

- **Provisioned the Virtual Machine** : Created an Ubuntu 22.04 VM in the Azure Portal, configured with a B1s size and attached a public IP.
- **Configured Networking** : Set up a Network Security Group with inbound rules to allow traffic on port 22 (SSH), 80 (HTTP), and 443 (HTTPS).
- **Installed LAMP Stack** : Connected to the VM via SSH and ran the following commands to install the web server, database, and PHP
   ```bash
    sudo apt update
    sudo apt install apache2
    sudo apt install mysql-server
   ```
- **Secure MySQL and Create Database**
   ```bash
    sudo mysql_secure_installation
    sudo mysql
   ```
   ```sql
   CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
   CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password';
   GRANT ALL ON wordpress.* TO 'wordpressuser'@'localhost';
   FLUSH PRIVILEGES;
   EXIT;
   ```
- **Download and Configure WordPress**
  ```bash
  cd /tmp
  curl -O https://wordpress.org/latest.tar.gz
  tar xzvf latest.tar.gz
  sudo cp -a /tmp/wordpress/. /var/www/html/
  ```
- **Set Permissions and Clean Up**
  ```bash
  sudo chown -R www-data:www-data /var/www/html/
  sudo rm /var/www/html/index.html
  ```
- **Edit WordPress Configuration**
  ```bash
  sudo nano /var/www/html/wp-config.php
  ```
  *We have to edit the php file with the database credentials*
- **Restart Web Server**
  ```bash
  sudo systemctl restart apache2
  ```
- **Securing the Site with a Custom Domain and SSL**
  - *Configured DNS* : Created two "A" records in my domain name registrar's control panel to point ![krisel.xyz](krisel.xyz) and![www.krisel.xyz](www.krisel.xyz) to the server public IP address.
  - *Created Apache Virtual Host*
    - To make Apache aware of the new domain, I created the following configuration file:
      ```bash
      sudo nano /etc/apache2/sites-available/krisel.xyz.conf
      ```
      - *Virtual Host file*
        ```Apache
        <VirtualHost *:80>
        ServerAdmin webmaster@localhost
        ServerName krisel.xyz
        ServerAlias www.krisel.xyz
        DocumentRoot /var/www/html
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>

## Modernizing the Architecture with a Managed Database
To improve scalability and reliability, the application was refactored into a multi-tier architecture. The database was migrated from the local MySQL server on the VM to a fully managed **Azure Database for MySQL** PaaS.

**Steps included** :
   - Provisioning a new Azure Database for MySQL flexible server
   - Configuring network firewall rules to allow a secure connection from the VM
   - Performing a database migration by exporting the original data with ```mysqldump``` and importing it into the new managed database
   - Reconfiguring the WordPress ```wp-config.php``` file to point to the new database endpoint and require an SSL connection

## Creating the second VM
   - Creating a script ```setup-webserver.sh``` that will update the server, install Apache and PHP and remove the default home page.
     ```bash
     #!/bin/bash
     sudo apt update
     sudo apt install apache2 php libapache2-mod-php php-mysql -y
     sudo rm /var/www/html/index.html```
   - We follow the steps as we did for the first VM and we insert the custom script at the end

## High Availability and Load Balancing
To ensure the application is resilient and can handle more traffic, an Azure Load Balancer was deployed and configured with the following components:
   - Frontend IP Configuration: A new static public IP was created and assigned to the load balancer to act as the single entry point for all web traffic.
   - Backend Pool: Both virtual machines (MyWordPressVM and MyWordPressVM2) were added to the backend pool, making them available to receive traffic from the load balancer.
   - Health Probe: An HTTP health probe was configured to monitor port 80 on the VMs, allowing the load balancer to automatically detect if a server is unhealthy and stop sending traffic to it.
   - Load Balancing Rule: A rule was created to forward all incoming traffic on port 80 (HTTP) to the VMs in the backend pool.

## Final Result
![WordPress site](assets/wordpress-live-domain-name.PNG)
