# WordPress on AWS with EC2, EFS, RDS Database — Complete Setup Guide

---

## Project Overview

This guide demonstrates how to deploy a WordPress website on Amazon Web Services (AWS) using EC2 for hosting and RDS for database management. The walkthrough includes security best practices, database connectivity, and web server optimization for a robust, scalable solution.

---

## Architecture Diagram

> **Space for Screenshot:**  
> `screenshots/architecture-diagram.png`

The deployment consists of:

- **EC2 Instance**: Amazon Linux 2 hosting Apache and PHP
- **RDS Database**: Managed MySQL for WordPress data
- **Security Groups**: Network access controls for EC2 and RDS
- **VPC**: Virtual Private Cloud for network isolation

---

## Prerequisites

- AWS Account with permissions for EC2, RDS, VPC
- Basic Linux command line knowledge
- Familiarity with web servers and databases

---

## Step 1: EC2 Instance Setup

### 1.1 Launch EC2 Instance

- Go to the **EC2 Dashboard** in AWS Console
- Click **"Launch Instance"**
- Configure:
  - **AMI**: Amazon Linux 2
  - **Instance Type**: t2.micro (Free Tier eligible)
  - **Key Pair**: Create or select existing
  - **Security Group**: Configure as below

> **Space for Screenshot:**  
> `screenshots/ec2-launch-config.png`

### 1.2 Security Group Configuration

| Type   | Protocol | Port Range | Source      | Description       |
|--------|----------|------------|-------------|-------------------|
| SSH    | TCP      | 22         | 0.0.0.0/0   | SSH access        |
| HTTP   | TCP      | 80         | 0.0.0.0/0   | Web traffic       |
| HTTPS  | TCP      | 443        | 0.0.0.0/0   | Secure web traffic|

> **Space for Screenshot:**  
> `screenshots/security-group-rules.png`

### 1.3 Connect to EC2 Instance

```sh
ssh -i "your-key.pem" ec2-user@your-ec2-public-ip
sudo yum update -y
```

> **Space for Screenshot:**  
> `screenshots/ssh-connection.png`

---

## Step 2: Install and Configure Web Server

### 2.1 Install Apache, PHP, and Required Modules

```sh
sudo yum install -y httpd
sudo yum install -y php php-mysqlnd php-gd php-xml php-mbstring php-json php-curl
sudo systemctl start httpd
sudo systemctl enable httpd
sudo systemctl status httpd
```

> **Space for Screenshot:**  
> `screenshots/apache-installation.png`

### 2.2 Test Web Server

```sh
echo "<h1>HTML Test - Working!</h1>" | sudo tee /var/www/html/test.html
echo "<?php echo '<h1>PHP Test - Working!</h1>'; ?>" | sudo tee /var/www/html/test.php
sudo chown apache:apache /var/www/html/test.*
sudo chmod 644 /var/www/html/test.*
```
Test in browser:

- [http://your-ec2-ip/test.html](http://your-ec2-ip/test.html)
- [http://your-ec2-ip/test.php](http://your-ec2-ip/test.php)

> **Space for Screenshot:**  
> `screenshots/web-server-test.png`

---

## Step 3: RDS Database Setup

### 3.1 Create RDS Instance

- Go to **RDS Dashboard**
- Click **"Create database"**
- Engine: **MySQL 8.0.35**
- Template: **Free tier**
- DB Instance Identifier: `database-1`
- Master Username: `admin`
- Master Password: `your-secure-password`
- DB Instance Class: `db.t3.micro`
- Storage: **20 GB** (General Purpose SSD)
- VPC: Default
- Subnet group: Default
- Public Access: Yes (for demo)
- Security Group: Create new

> **Space for Screenshot:**  
> `screenshots/rds-configuration.png`

### 3.2 RDS Security Group

| Type         | Protocol | Port Range | Source (EC2 SG ID) | Description                  |
|--------------|----------|------------|--------------------|------------------------------|
| MySQL/Aurora | TCP      | 3306       | EC2-Security-Group | Database access from EC2     |

> **Space for Screenshot:**  
> `screenshots/database-creation.png`

### 3.3 Create WordPress Database and User

```sh
mysql -h database-1.cafsc6kai733.us-east-1.rds.amazonaws.com -u admin -p

CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'%' IDENTIFIED BY 'WordPress123!SecurePass';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'%';
FLUSH PRIVILEGES;
EXIT;
```

---

## Step 4: Test Database Connectivity

### 4.1 Create Database Test Script

```php
<?php
$connection = mysqli_connect('database-1.cafsc6kai733.us-east-1.rds.amazonaws.com', 'wpuser', 'WordPress123!SecurePass', 'wordpress');
if (!$connection) {
    echo '<h1 style="color: red;">Database connection failed: ' . mysqli_connect_error() . '</h1>';
} else {
    echo '<h1 style="color: green;">Database connection successful!</h1>';
    echo '<p>Connected to: ' . mysqli_get_host_info($connection) . '</p>';
    mysqli_close($connection);
}
?>
```
Save as `/var/www/html/db-test.php`  
Set permissions:

```sh
sudo chown apache:apache /var/www/html/db-test.php
sudo chmod 644 /var/www/html/db-test.php
```

### 4.2 Test Database Connection

- Visit: `http://your-ec2-ip/db-test.php`

> **Space for Screenshot:**  
> `screenshots/database-creation.png`

---

## Step 5: WordPress Installation

### 5.1 Download and Extract WordPress

```sh
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/
sudo chown -R apache:apache /var/www/html/
sudo chmod -R 755 /var/www/html/
```

> **Space for Screenshot:**  
> `screenshots/wordpress-installation.png`

### 5.2 Configure WordPress

Create `wp-config.php`:

```php
<?php
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wpuser' );
define( 'DB_PASSWORD', 'WordPress123!SecurePass' );
define( 'DB_HOST', 'database-1.cafsc6kai733.us-east-1.rds.amazonaws.com' );
define( 'DB_CHARSET', 'utf8mb4' );
define( 'DB_COLLATE', '' );

<?php echo file_get_contents('https://api.wordpress.org/secret-key/1.1/salt/'); ?>

$table_prefix = 'wp_';
define( 'WP_DEBUG', false );
if ( ! defined( 'ABSPATH' ) ) {
    define( 'ABSPATH', __DIR__ . '/' );
}
require_once ABSPATH . 'wp-settings.php';
?>
```
Set permissions:

```sh
sudo chown apache:apache /var/www/html/wp-config.php
sudo chmod 644 /var/www/html/wp-config.php
```

---

## Step 6: Troubleshooting and Optimization

### 6.1 Common Issues

**Filesystem Corruption**  
_Circular directory references causing infinite loops_

```sh
sudo mv /var/www/html /var/www/html_backup
sudo mkdir /var/www/html
sudo cp -r /tmp/wordpress/* /var/www/html/
```

**PHP Processing Errors**  
_"Primary script unknown" errors_

```sh
sudo yum install -y php
sudo systemctl restart httpd
```

> **Space for Screenshot:**  
> `screenshots/troubleshooting.png`

### 6.2 Performance Optimization

Append to Apache config for performance and security:

```apacheconf
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 15

Header always set X-Content-Type-Options nosniff
Header always set X-Frame-Options DENY
Header always set X-XSS-Protection "1; mode=block"
```

Restart Apache:

```sh
sudo systemctl restart httpd
```

---

## Step 7: Final Testing and Verification

### 7.1 Comprehensive Testing

```sh
echo "=== Testing HTML ==="
curl http://localhost/test.html

echo "=== Testing PHP ==="
curl http://localhost/test.php

echo "=== Testing Database Connection ==="
curl http://localhost/db-test.php

echo "=== Testing WordPress ==="
curl -I http://localhost/
```

> **Space for Screenshot:**  
> `screenshots/final-testing.png`

### 7.2 WordPress Setup Completion

- Visit: `http://your-ec2-ip/`
- Complete WordPress installation:
  - **Site Title**: Your WordPress Site
  - **Username**: admin
  - **Password**: Use a strong password
  - **Email**: your-email@example.com

Login to WordPress admin panel and customize your site.

> **Space for Screenshot:**  
> `screenshots/wordpress-dashboard.png`

---

## Step 8: Security Considerations

### 8.1 Security Best Practices

- **Database**: Separate user, strong password, private subnet in production
- **Web Server**: Regular updates, secure file permissions, security headers
- **Network**: Restrictive security groups, SSH keys, HTTPS ready

### 8.2 Monitoring and Maintenance

```sh
sudo tail -f /var/log/httpd/access_log
sudo tail -f /var/log/httpd/error_log
sudo yum update -y
sudo systemctl status httpd
sudo systemctl status mysqld
```

---

## Project Structure

```
wordpress-aws-project/
├── README.md
├── screenshots/
│   ├── architecture-diagram.png
│   ├── ec2-launch-config.png
│   ├── security-group-rules.png
│   ├── ssh-connection.png
│   ├── apache-installation.png
│   ├── web-server-test.png
│   ├── rds-configuration.png
│   ├── database-creation.png
│   ├── wordpress-installation.png
│   └── final-testing.png
├── scripts/
│   ├── install-lamp.sh
│   ├── setup-database.sh
│   └── deploy-wordpress.sh
└── config/
    ├── httpd.conf
    └── wp-config-template.php
```

---

## Lessons Learned

- **Filesystem Management**: Avoid directory corruption with atomic operations
- **Service Dependencies**: Understand Apache, PHP, and database interplay
- **Security Groups**: Correct network configuration is critical
- **Troubleshooting**: Use a systematic approach
- **Documentation**: Stepwise documentation aids reproducibility

---

## Future Enhancements

- SSL/TLS: Use Let's Encrypt or AWS Certificate Manager
- Load Balancing: Application Load Balancer for HA
- Auto Scaling: Configure for dynamic scaling
- Backups: Automate EC2 and RDS backups
- Monitoring: Integrate with CloudWatch
- CDN: Use CloudFront for performance
- IaC: Migrate to CloudFormation/Terraform

---

## Cost Optimization

- **EC2**: t2.micro (Free Tier)
- **RDS**: db.t3.micro (Free Tier)
- **Storage**: Minimal allocation
- **Data Transfer**: Optimize for low costs

---

## Conclusion

This guide demonstrates a successful WordPress deployment on AWS, with secure database connectivity, optimized web server, and a scalable cloud architecture.

- ✅ Functional WordPress website
- ✅ Secure database connectivity
- ✅ Proper security group configuration
- ✅ Troubleshooting steps
- ✅ Performance optimization
- ✅ Scalable architecture

---

> **Live Site:**  
> `http://your-ec2-public-ip/`

---
