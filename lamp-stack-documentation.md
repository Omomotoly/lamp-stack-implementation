# WEB STACK IMPLEMENTATION (LAMP STACK) IN AWS

## Introduction
The LAMP stack remains one of the most widely adopted, open-source foundations for engineering and deploying dynamic web applications. The framework is composed of four distinct layers working in tandem:

* Linux: The secure and robust open-source operating system acting as the infrastructure's base layer.

* Apache: The reliable web server responsible for processing incoming client requests and serving web assets.

* MySQL: The relational database management system (RDBMS) used for structured data storage and management.

* PHP: The server-side scripting language engineered to handle business logic, database communication, and dynamic content generation

## Step 0: Prerequisites
1\. An Ubuntu 26.04 LTS (HVM) operating system was spun up on a t3.micro EC2 instance. This virtual machine was deployed via the AWS console within the eu-north-1 region.

![Screenshot](images/creating-ec2.png)
![Screenshot](images/ec2-details.png)

2\. Created SSH key pair named my-ec2-key to access the instance on port 22

3\. Inbound firewall policies were defined within the AWS Security Group using the following rules:

* Allow traffic on port 22 (SSH) with source from any IP address. This is opened by default.
* Allow traffic on port 80 (HTTP) with source from anywhere on the internet. 
* Allow traffic on port 443 (HTTPS) with source from anywhere on the internet. 

![Screenshot](images/security-rule.png)

4\. The default VPC and Subnet was used for the networking configuration.

![Screenshot](images/networking.png)

5\. The private ssh key that got downloaded was located, permission was changed for the private key file and then used to connect to the instance by running

```
chmod 400 my-ec2-key.pem
```
```
ssh -i /my-ec2-key.pem ubuntu@16.170.246.115
```
Where username=ubuntu and public ip address=16.170.246.115

![Screenshot](images/ssh-access.png)

## Step 1 - Install Apache and Update the Firewall

1\. Update and upgrade list of packages in package manager
```
sudo apt update
sudo apt upgrade -y
```
![Screenshot](images/ec2-update.png)

![Screenshot](images/ec2-upgrade.png)

2\. Run apache2 package installation
```
sudo apt install apache2 -y
```

![Screenshot](images/install-apache.png)

3\. Enable and verify that apache is running as a service on the OS.
```
sudo systemctl enable apache2
sudo systemctl status apache2
```

If it green and running, then apache2 is correctly installed 

![Screenshot](images/check-status.png)

4\. The server is running and can be accessed locally in the ubuntu shell by running the command below:

```
curl http://localhost:80
OR
curl http://127.0.0.1:80
```

![Screenshot](images/default-page-curl.png)

5\. To confirm the Apache HTTP server can successfully process requests, input the instance's public IP address as the URL in a web browser.

http://16.170.246.115:80

![Screenshot](images/default-page-browser.png)

This shows that the web server is correctly installed and it is accessible throuh the firewall.

6\. Another way to retrieve the public ip address other than check the aws console

```
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```

The terminal did not output the public IP address after running the command above. To resolve the issue, the following navigation was made from the EC2 instance page on the AWS console

* Actions > Instance Settings > Modify instance metadata options.
* Then change the IMDSv2 from Required to Optional.

![Screenshot](images/imds-option.png)

The command was run again, this time there was no error with the public IP address displayed.

```
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```

![Screenshot](images/pub-ip-curl.png)


## Step 2 - Install MySQL

1\. Install a relational database (RDB)

For data storage and management, MySQL was installed as the relational database engine for the PHP environment.

```
sudo apt install mysql-server
```

![Screenshot](images/install-mysql.png)

When prompted, install was confirmed by typing y and then Enter.

2\. Enable and verify that mysql is running with the commands below

```
sudo systemctl enable --now mysql
sudo systemctl status mysql
```

![Screenshot](images/check-mysql-status.png)

3\. Log in to mysql console

```
sudo mysql
```

This opens the MySQL server with full admin permissions because sudo lets you log in as the root user.

4\. Set a password for root user using mysql_native_password as default authentication method.

Here, the user's password was defined as "Admin123"

```
ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'Admin123';
```

![Screenshot](images/access-mysql-shell.png)

Exit the MySQL shell

```
exit
```

5\. Run an Interactive script to secure MySQL
 
The security script comes pre-installed with mysql. This script removes some insecure settings and lock down access to the database system.

```
sudo mysql_secure_installation
```

![Screenshot](images/secure-mysql-1.png)

![Screenshot](images/secure-mysql-2.png)

Regardless of whether the VALIDATION PASSWORD PLUGIN is set up, the server will ask to select and confirm a password for MySQL root user.

6\. After changing root user password, log in to MySQL console.

A command prompt for password was noticed after running the command below.

```
sudo mysql -p
```

![Screenshot](images/access-mysql-with-password.png)

Exit MySQL shell

```
exit
```

## Step 3 - Install PHP

1\. **Install php** With Apache handling web traffic and MySQL storing the data, PHP is installed to process the code and display dynamic content to the user

The following were installed:

* php package
* php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases.
* libapache2-mod-php, to enable Apache to handle PHP files

```
sudo apt install php libapache2-mod-php php-mysql
```

![Screenshot](images/php-install.png)

Confirm the PHP version

```
php -v
```

![Screenshot](images/confirm-php-install.png)

The LAMP stack is now completely installed.

To test the set up with a PHP script, it's best to set up a proper Apache Virtual Host to hold the website files and folders. Virtual host permits multiple websites on a single machine and it won't be noticed by the website users.

## Step 4 - Create a virtual host for the website using Apache

1\. The default directory serving the apache default page is /var/www/html. Create your document directory next to the default one.

Created the directory for projectlamp using "mkdir" command


```
sudo mkdir /var/www/projectlamp
```

Assign the directory ownership with $USER environment variable which references the current system user.

```
sudo chown -R $USER:$USER /var/www/projectlamp
```

2\. Create and open a new configuration file in apache’s “sites-available” directory using vim.

```
sudo vim /etc/apache2/sites-available/projectlamp.conf
```

Paste in the bare-bone configuration below:

```
<VirtualHost *:80>
  ServerName projectlamp
  ServerAlias www.projectlamp
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/projectlamp
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

![Screenshot](images/virtual-host.png)

3\. Show the new file in sites-available

```
sudo ls /etc/apache2/sites-available
```

```
Output:
000-default.conf default-ssl.conf projectlamp.conf
```

![Screenshot](images/ls-sites-available.png)

With the VirtualHost configuration, Apache will serve projectlamp using /var/www/projectlamp as its web root directory.

4\. Enable the new virtual host

```
sudo a2ensite projectlamp
```

![Screenshot](images/enable-virtual-host.png)

5\. Disable apache’s default website.

This is because Apache’s default configuration will overwrite the virtual host if not disabled. This is required if a custom domain is not being used.

```
sudo a2dissite 000-default
```

![Screenshot](images/disable-root-dir.png)


6\. Ensure the configuration does not contain syntax error

The command below was used:

```
sudo apache2ctl configtest
```

![Screenshot](images/check-config-syntax.png)

7\. Reload apache for changes to take effect.

```
sudo systemctl reload apache2
```

![Screenshot](images/reload-apache.png)


8\. The new website is now active but the web root /var/www/projectlamp is still empty. Create an index.html file in this location so to test the virtual host work as expected.


```
sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html
```

9\. Open the website on a browser using the public IP address.

```
http://16.170.246.115:80
```

![Screenshot](images/site-url-ip.png)

10\. Open the website with public dns name (port is optional)

```
http://<public-DNS-name>:80
```

![Screenshot](images/site-url-public-dns.png)


This file can serve as a temporary landing page until an index.php file is created to replace it. Once the PHP file is ready, you must delete or rename index.html, because Apache will load it first by default.


## Step 5 - Enable PHP on the website

By default, Apache looks for index.html before index.php. This is great for putting up a quick maintenance page—just add an index.html file with your message, and visitors will see it first. When you are done, delete or rename index.html to bring your PHP app back online. If you want to change this order permanently, open the /etc/apache2/mods-enabled/dir.conf file and move index.php to the front of the list.

1\. Open the dir.conf file with vim to change the behaviour

```
sudo vim /etc/apache2/mods-enabled/dir.conf
```

```<IfModule mod_dir.c>
  # Change this:
  # DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
  # To this:
  DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

![Screenshot](images/index-php-config.png)

2\. Reload Apache

Apache is reloaded so the changes takes effect.

```
sudo systemctl reload apache2
```

3\. Create a php test script to confirm that Apache is able to handle and process requests for PHP files.

A new index.php file was created inside the custom web root folder.

```
vim /var/www/projectlamp/index.php
```

Add the text below in the index.php file

```
<?php
phpinfo();
```

![Screenshot](images/index-php.png)

4\. Now refresh the page

![Screenshot](images/php-page.png)

This page shows server details from PHP's perspective, which helps with debugging and checking your settings. After verifying the details, you should delete this file because it exposes sensitive data about your PHP setup and the Ubuntu server. You can easily recreate it later if needed.

```
sudo rm /var/www/projectlamp/index.php
```

### Conclusion:

The LAMP stack offers a reliable platform for developing and deploying web apps. By following these steps, you can successfully configure and manage your environment for stable performance.
