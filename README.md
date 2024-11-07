![Alt text](/3._Host_a_Dynamic_Web_App_on_AWS.png)

---

# Dynamic Website Deployment on AWS

This project demonstrates a dynamic website deployment on AWS, using multiple AWS services for an efficient, secure, and scalable infrastructure. The resources configured provide high availability, fault tolerance, and secure internet access to both public and private subnets, utilizing DevOps practices for automated deployment and updates.

## Project Architecture

The architecture spans across two availability zones for enhanced reliability. The following resources were configured:

1. **Virtual Private Cloud (VPC)** - Configured with public and private subnets across two availability zones.
2. **Internet Gateway** - Enables internet connectivity for instances within the VPC.
3. **Security Groups** - Acts as a network firewall to control access.
4. **Availability Zones** - Utilized two zones to increase system reliability and fault tolerance.
5. **Public Subnets** - Hosts NAT Gateway and Application Load Balancer.
6. **EC2 Instance Connect Endpoint** - Ensures secure access to assets in both public and private subnets.
7. **Web Servers (EC2 Instances)** - Hosted within private subnets for enhanced security.
8. **NAT Gateway** - Allows instances in private subnets to access the internet.
9. **Application Load Balancer** - Distributes web traffic evenly across an auto-scaling group of EC2 instances.
10. **Auto Scaling Group** - Automatically manages EC2 instances to ensure availability, scalability, and fault tolerance.
11. **Certificate Manager** - Secures application communication.
12. **Simple Notification Service (SNS)** - Provides alerts for activities within the Auto Scaling Group.
13. **Route 53** - Manages domain registration and DNS settings.
14. **Amazon S3** - Stores application code.

## Prerequisites

- AWS CLI configured with appropriate IAM permissions.
- EC2 instances within private subnets to ensure a secure environment.
- Set up domain registration and DNS records in Route 53.

## Getting Started

1. Clone the repository containing the project reference diagram and deployment scripts.
2. Deploy the architecture using the provided CloudFormation templates or manual configurations.

### Deployment Scripts

The following scripts were used to set up and maintain the environment:

#### Server Update and Flyway Installation Script

This script updates the server, installs Flyway for database migrations, and performs SQL migrations.

```bash
# Update all packages
sudo yum update -y

# Download and extract Flyway
sudo wget -qO- https://download.red-gate.com/maven/release/com/redgate/flyway/flyway-commandline/10.9.1/flyway-commandline-10.9.1-linux-x64.tar.gz | tar -xvz

# Create a symbolic link to make Flyway accessible globally
sudo ln -s $(pwd)/flyway-10.9.1/flyway /usr/local/bin

# Create the SQL directory for migrations
sudo mkdir sql

# Download the migration SQL script from AWS S3
sudo aws s3 cp "$S3_URI" sql/

# Run Flyway migration
flyway -url=jdbc:mysql://"$RDS_ENDPOINT":3306/"$RDS_DB_NAME" \
  -user="$RDS_DB_USERNAME" \
  -password="$RDS_DB_PASSWORD" \
  -locations=filesystem:sql \
  migrate
```

#### Web Server and Application Setup Script

This script configures the Apache server, installs PHP and MySQL, and deploys application code from S3.

```bash
# Interpreter Directive
#!/bin/bash

# Update server packages
sudo yum update -y
sudo dnf upgrade --releasever=2023.6.20241031

# Install Apache server
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

# Install PHP and extensions
sudo dnf install -y \
php \
php-pdo \
php-openssl \
php-mbstring \
php-exif \
php-fileinfo \
php-xml \
php-ctype \
php-json \
php-tokenizer \
php-curl \
php-cli \
php-fpm \
php-mysqlnd \
php-bcmath \
php-gd \
php-cgi \
php-gettext \
php-intl \
php-zip

# Install MySQL
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server

# Start MySQL
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Enable Apache mod_rewrite
sudo sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf

# Define S3 bucket variable
S3_BUCKET_NAME=obens-sql-files

# Sync application code from S3 bucket
sudo aws s3 sync s3://"$S3_BUCKET_NAME" /var/www/html

# Navigate to web directory and extract application code
cd /var/www/html
sudo unzip shopwise.zip
sudo cp -R shopwise/. /var/www/html/
sudo rm -rf shopwise shopwise.zip

# Set permissions
sudo chmod -R 777 /var/www/html
sudo chmod -R 777 storage/

# Edit environment file for database credentials
sudo vi .env

# Restart Apache server
sudo service httpd restart
```

## Monitoring and Alerts

The setup includes Amazon SNS to send notifications for Auto Scaling events, ensuring prompt alerts regarding any changes in instance count or status.

## Security and Compliance

Using security best practices, all instances are within private subnets, with access controlled via security groups and EC2 Instance Connect Endpoint. SSL certificates from AWS Certificate Manager secure application communications.
--- 
