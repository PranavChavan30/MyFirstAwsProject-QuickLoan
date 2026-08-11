### QuickLoan AWS Project Deployment Guide

###  Infrastructure Setup

* **VPC:** Custom VPC ('project-vpc' with 10.20.0.0/16)
* **Subnets:** 3 Public subnets and 1 Private subnet
* **Route Tables:** Public Route Table connected with 2 Public subnets
* **Gateways:** Internet Gateway (IGW) & NAT Gateway (NGW)
* **Security:** Security Groups configured in VPC for Web and DB
* **Instances:** 

  * Jump-server (Amazon Linux 2023 / amazon6.1)
  * App-server (Amazon Linux 2023 / amazon6.1)
  * Database-server (Amazon Linux 2023 / amazon6.1)

###  Step-by-Step Deployment Commands

### 1. App-Server Configuration

bash

yum update -y
yum install -y nginx
systemctl start nginx
chkconfig nginx on

# Verification: Enter publicip:80 in the browser to cross check Nginx

yum install -y php8.2 php-fpm php-mysqlnd php-pdo php-mbstring 
php -v
systemctl start php-fpm
chkconfig php-fpm on
systemctl restart nginx

Use code with caution.

*Note: Transferred includes, nginx, and public folders from laptop to App Server via WinSCP.* 

bash

mv /home/ec2-user/includes /usr/share/nginx/html
mv /home/ec2-user/public /usr/share/nginx/html
chown -R nginx:nginx /usr/share/nginx/html/public
chmod -R 755 /usr/share/nginx/html/public

Use code with caution.

### 2. AWS S3 Bucket Setup

* Created S3 bucket named: aws-project-bucket-27022026
* Uploaded images/ folder present inside the public folder.
* Updated asset paths in configuration files:

bash

vim /usr/share/nginx/html/public/index.html
# Replaced loanapp332211.s3.eu-west-2.amazonaws.com with aws-project-bucket-27022026.s3.us-east-2.amazonaws.com

vim /usr/share/nginx/html/public/apply.php
# Updated Logo Source: <img src="https://aws-project-bucket-27022026.s3.us-east-2.amazonaws.com/images/quickloan_logo.png" alt="QuickLoan Logo">

Use code with caution.

### 3. Database Connection & Table Schema

bash

cd /usr/share/nginx/html/includes
vim db_connect.php
# Replaced endpoint, database username & database password

Use code with caution.

Connecting App Server to DB Server via SSH: 

bash

ssh -i db-keypair.pem ec2-user@<private_ip_of_database_server>
yum install mariadb105 -y
mysql -h <endpoint> -u <username> -p<password>

Use code with caution.

Database SQL Queries: 

sql

CREATE DATABASE quickloan_db;
USE quickloan_db;
CREATE TABLE applications (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    contact VARCHAR(20) NOT NULL,
    email VARCHAR(255) NOT NULL,
    loan_type VARCHAR(50) NOT NULL
);

Use code with caution.

### 4. Cloud Administration, AMI & Auto Scaling Setup

* **AMI Creation:** Created a Custom Amazon Machine Image (AMI) from the configured App-Server to preserve Nginx, PHP, and application code configuration.
* **Launch Template:** Built a reusable template utilizing the custom AMI for standardized instance launches.
* **Target Group & ALB:** Configured Target Groups and deployed an AWS Application Load Balancer (ALB) for dynamic load distribution.
* **Auto Scaling Group (ASG):** Bound the custom AMI template with ASG to automate scale-up and scale-down processes based on active traffic load.

### 5. DNS Mapping & Nginx Reverse Proxy (No-IP)

* Configured A record on noip.com: loan.ddns.net pointing to pub-ip

bash

ssh -i app-keypair.pem ec2-user@<private_ip_of_app_server>
vim /home/ec2-user/nginx/quickloan.conf
# Updated line: server_name quickloan.hopto.org;

mv /home/ec2-user/nginx/quickloan.conf /etc/nginx/conf.d
nginx -t
systemctl restart nginx
systemctl reload nginx

Use code with caution.
