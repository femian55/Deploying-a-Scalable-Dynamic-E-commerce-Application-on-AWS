# Hosting a Dynamic E-commerce Web Application on AWS

## Project Overview

This project aims to create a highly available, scalable, and fault-tolerant e-commerce web application using AWS services. The infrastructure is designed to leverage multiple availability zones (AZs) for high availability and fault tolerance.

## Architecture Overview

1. **VPC with Public and Private Subnets**: 
   - Deployed in 2 availability zones for redundancy and high availability.
   - Public subnets contain resources accessible from the internet.
   - Private subnets contain sensitive resources not directly accessible from the internet.

2. **Internet Gateway**: 
   - Allows communication between instances in the VPC and the internet.

3. **NAT Gateway**: 
   - Enables instances in the private subnets to access the internet for updates and patches.

4. **Bastion Host**: 
   - Deployed in the public subnet to provide secure access to instances in private subnets.

5. **Application Load Balancer (ALB)**: 
   - Distributes incoming web traffic across multiple EC2 instances in different AZs.

6. **EC2 Instance Connect Endpoint**: 
   - Facilitates SSH access to instances without the need for a public IP.

7. **EC2 Instances**: 
   - Host the dynamic e-commerce website.
   - Configured in an Auto Scaling Group to ensure scalability and high availability.

8. **MySQL RDS Database**: 
   - Provides a managed relational database service for the application data.

9. **Flyway**: 
   - Used for database migrations to manage schema changes.

10. **Amazon S3**: 
    - Stores static assets for the website.
    - Used for hosting the website files which EC2 instances download.

11. **IAM Roles**: 
    - Provide necessary permissions for EC2 instances to interact with other AWS services, such as downloading files from S3.

12. **Route 53**: 
    - Manages domain name registration and DNS records for the web application.

## Deployment Steps

1. **Clone the Repository**
   ```sh
   git clone https://github.com/your-repo/your-project.git
   cd your-project
   ```

2. **Set Up AWS CLI and Configure Credentials**
   ```sh
   aws configure
   ```

3. **Deploy the CloudFormation Stack**
   - Update the parameters in the `cloudformation-template.yaml` file as needed.
   - Deploy the stack using AWS CLI.
   ```sh
   aws cloudformation create-stack --stack-name ecommerce-app --template-body file://cloudformation-template.yaml --parameters ParameterKey=KeyName,ParameterValue=your-key-name
   ```

4. **Database Migration with Flyway**
   - Configure Flyway with your RDS endpoint and credentials.
   - Run the migration.
   ```sh
   flyway -url=jdbc:mysql://<RDS-ENDPOINT>:3306/dbname -user=username -password=password migrate
   ```

5. **Upload Website Files to S3**
   ```sh
   aws s3 cp website/ s3://your-bucket-name/ --recursive
   ```

6. **Deploy the Application**
   - Connect to the bastion host and then to the EC2 instances.
   - Install the necessary software and deploy the application.
   ```sh
   ssh -i your-key.pem ec2-user@bastion-host-public-ip
   ssh -i your-key.pem ec2-user@private-instance-ip
   ```

7. **Create an AMI from the EC2 Instance**
   ```sh
   aws ec2 create-image --instance-id i-xxxxxxxx --name "EcommerceApp-AMI"
   ```

8. **Configure Auto Scaling Group**
   - Use the created AMI to configure the Auto Scaling Group.

## EC2 Instance Setup Script

```bash
#!/bin/bash

# Update all packages on the server
sudo yum update -y

# Install Apache web server, enable it to start on boot, and start the server immediately
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# Install PHP and necessary extensions
sudo dnf install -y php php-pdo php-openssl php-mbstring php-exif php-fileinfo php-xml php-ctype php-json php-tokenizer php-curl php-cli php-fpm php-mysqlnd php-bcmath php-gd php-cgi php-gettext php-intl php-zip

# Install MySQL version 8
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Enable 'mod_rewrite' module in Apache
sudo sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf

# Set environment variable for S3 bucket name
S3_BUCKET_NAME=webapp-bucket

# Download contents of the specified S3 bucket to the '/var/www/html' directory
sudo aws s3 sync s3://"$S3_BUCKET_NAME" /var/www/html

# Change directory to '/var/www/html'
cd /var/www/html

# Extract the contents of the application code zip file
sudo unzip shopwise.zip

# Recursively copy all files, including hidden ones, from the 'shopwise' directory to the '/var/www/html/'
sudo cp -R shopwise/. /var/www/html/

# Permanently delete the 'shopwise' directory and the 'shopwise.zip' file
sudo rm -rf shopwise shopwise.zip

# Set permissions 777 for the '/var/www/html' directory and the 'storage/' directory
sudo chmod -R 777 /var/www/html
sudo chmod -R 777 storage/

# Edit the .env file to add your database credentials 
sudo vi .env

# Restart the Apache server
sudo service httpd restart
```

## Conclusion

This project demonstrates the deployment of a dynamic e-commerce web application using a variety of AWS services to ensure high availability, scalability, and security. For detailed instructions and scripts, please refer to the respective files in this repository.

