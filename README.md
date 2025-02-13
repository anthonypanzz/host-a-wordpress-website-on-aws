![Alt text](Host_a_WordPress_Website_on_AWS.png)

# WordPress Website Deployment on AWS

## Project Overview
This project demonstrates the deployment of a WordPress website on AWS using various AWS services to ensure high availability, security, and scalability. The reference architecture, along with the scripts used for deployment, is available in the project's GitHub repository.
---
## Architecture Components
1. **Virtual Private Cloud (VPC):** Configured with public and private subnets across two Availability Zones.
![Screenshot 2025-02-07 052036](https://github.com/user-attachments/assets/cbbb8386-cb5c-4d7c-b13d-f9dcbd8ef15c)
---
2. **Internet Gateway:** Enables connectivity between the VPC instances and the Internet.
---
3. **Security Groups:** Implemented as a firewall mechanism to regulate traffic.
![Screenshot 2025-02-07 052308](https://github.com/user-attachments/assets/0dfab377-90f2-493a-a6f0-03ec5a658a4b)
---
4. **Availability Zones:** Utilized for system reliability and fault tolerance.
![Screenshot 2025-02-07 052638](https://github.com/user-attachments/assets/dc2cb05d-355c-44c0-bb81-33e1db58c34e)
---
5. **Public Subnets:** Assigned for infrastructure components like the NAT Gateway and Application Load Balancer.
![Screenshot 2025-02-07 052109](https://github.com/user-attachments/assets/9afaf842-95b0-4d97-95d0-b008d5350c51)
---
6. **EC2 Instance Connect Endpoint:** Allows secure access to instances in both public and private subnets.
![Screenshot 2025-02-07 052337](https://github.com/user-attachments/assets/061a19b9-7fcb-4d74-b596-9c66a91cc7fa)
---
7. **Private Subnets:** Hosts web servers (EC2 instances) for enhanced security.
![Screenshot 2025-02-07 052109](https://github.com/user-attachments/assets/3dc008d6-c7dc-41d7-913a-f04caeaf91d5)
---
8. **NAT Gateway:** Enables instances in private subnets to access the Internet securely.
![Screenshot 2025-02-07 052252](https://github.com/user-attachments/assets/97ef38da-73d0-45d1-a34f-6e754ab44527)
---
9. **EC2 Instances:** Hosts the WordPress website.
![Screenshot 2025-02-07 052638](https://github.com/user-attachments/assets/23f06815-adcf-429e-869a-14a2fa0673b2)
---
10. **Application Load Balancer (ALB):** Distributes web traffic evenly to an Auto Scaling Group of EC2 instances.
![Screenshot 2025-02-07 052604](https://github.com/user-attachments/assets/57be8ea1-63e1-44bd-a7a6-934334f310d5)
![Screenshot 2025-02-07 053115](https://github.com/user-attachments/assets/56b3c95d-9531-496a-83f6-d69f7d6420ee)
---
11. **Auto Scaling Group (ASG):** Manages EC2 instances automatically for scalability and availability.
![Screenshot 2025-02-07 052621](https://github.com/user-attachments/assets/79724042-aa59-41db-b508-255e46c52eef)
---
12. **GitHub:** Used for version control and collaboration on web files.
---
13. **AWS Certificate Manager:** Secures application communications with SSL/TLS.
![Screenshot 2025-02-07 052738](https://github.com/user-attachments/assets/998f3502-73f7-4d5a-bf27-0e609e08c458)
---
14. **AWS Simple Notification Service (SNS):** Sends alerts about Auto Scaling Group activities.
![Screenshot 2025-02-07 045343](https://github.com/user-attachments/assets/787721b8-b788-4e27-a2ce-9541f178bba4)
---
15. **Route 53:** Handles domain name registration and DNS records.
![Screenshot 2025-02-07 052716](https://github.com/user-attachments/assets/491d8439-668f-4901-bd02-51d3b414c780)
---
16. **Elastic File System (EFS):** Provides a shared file system for web files.
![Screenshot 2025-02-07 052450](https://github.com/user-attachments/assets/1eb34795-0fa3-48bf-bce6-3edf3cc2f4cb)
---
17. **Relational Database Service (RDS):** Manages the database for WordPress.
![Screenshot 2025-02-07 052535](https://github.com/user-attachments/assets/12371a85-edd1-47c0-a93c-64e33293e52d)
![Screenshot 2025-02-04 002108](https://github.com/user-attachments/assets/82c68688-659c-42cf-80e4-f12a7a79d640)
![Screenshot 2025-02-04 002003](https://github.com/user-attachments/assets/67b29e8a-44b9-4ec7-ad5e-aaeca90d406d)
![Screenshot 2025-02-07 051836](https://github.com/user-attachments/assets/9b64f831-76af-48f9-aae9-102d75a1f5d7)
![Screenshot 2025-02-07 051903](https://github.com/user-attachments/assets/06dfc20a-47e2-4153-b5d8-c5e3afc11781)
![Screenshot 2025-02-07 051914](https://github.com/user-attachments/assets/3776df59-81fd-4e7d-b3d3-fdbcc9943c4a)
---
## Installation and Deployment
### WordPress Installation Script
The following script is used to set up WordPress on an EC2 instance:
```bash
sudo su
sudo yum update -y
sudo mkdir -p /var/www/html
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-1.amazonaws.com
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd
sudo dnf install -y \
php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd \
php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath \
php-ctype php-fileinfo php-openssl php-pdo php-tokenizer
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server
sudo systemctl start mysqld
sudo systemctl enable mysqld
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
sudo vi /var/www/html/wp-config.php
sudo service httpd restart
```

### Auto Scaling Group Launch Template Script
The following script is used for launching EC2 instances within an Auto Scaling Group:
```bash
#!/bin/bash
sudo yum update -y
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd
sudo dnf install -y \
php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd \
php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath \
php-ctype php-fileinfo php-openssl php-pdo php-tokenizer
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server
sudo systemctl start mysqld
sudo systemctl enable mysqld
EFS_DNS_NAME=fs-02d3268559aa2a318.efs.us-east-1.amazonaws.com
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
mount -a
chown apache:apache -R /var/www/html
sudo service httpd restart
```

## Usage
After setting up the infrastructure and executing the scripts, navigate to the registered domain in a web browser to access the WordPress website.

## Conclusion
This project effectively demonstrates the use of AWS services to host a highly available, secure, and scalable WordPress website. The implementation leverages best practices for networking, security, and automation to ensure seamless operation.

For more details, refer to the project's GitHub repository.


