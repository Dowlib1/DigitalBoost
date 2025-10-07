# DigitalBoost
WordPress website for digital Boost.


# AWS WordPress High Availability Project Documentation

## Overview
This document outlines the architecture, setup steps, security measures, and demonstration for deploying a highly available WordPress site on AWS.

---

## 1. VPC Setup

**Description:**  
Created VPC with CIDR block `10.0.0.0/16`, public/private subnets in two AZs for HA.

**Screenshot:**  
![VPC Setup](<insert-your-screenshot-link-here>)

---

## 2. Public & Private Subnets with NAT Gateway

**Description:**  
Enabled NAT Gateway for secure internet access from private subnets.

**Screenshot:**  
![NAT Gateway](<insert-your-screenshot-link-here>)

---

## 3. Security Groups

**Description:**  
Configured Security Groups for ALB, EC2, RDS, and EFS following least privilege principles.

**Screenshot:**  
![Security Groups](<insert-your-screenshot-link-here>)

---

## 4. MySQL RDS Setup

**Description:**  
Deployed MySQL RDS in private subnets; secured access via SGs.

**Screenshot:**  
![RDS Setup](<insert-your-screenshot-link-here>)

---

## 5. EFS Setup

**Description:**  
Provisioned EFS with mount targets in both AZs; mounted on EC2 instances.

**Screenshot:**  
![EFS Setup](<insert-your-screenshot-link-here>)

---

## 6. EC2 Launch Template & UserData

**Description:**  
Configured EC2 launch template with UserData to install Apache, PHP, mount EFS, and prepare for WordPress.

**Screenshot:**  
![Launch Template](<insert-your-screenshot-link-here>)

---

## 7. Application Load Balancer

**Description:**  
Set up ALB to distribute traffic among EC2s in multiple AZs.

**Screenshot:**  
![ALB Setup](<insert-your-screenshot-link-here>)

---

## 8. Auto Scaling Group

**Description:**  
Configured ASG to dynamically scale EC2s based on CPU usage.

**Screenshot:**  
![Auto Scaling Group](<insert-your-screenshot-link-here>)

---

## 9. Route 53 Setup

**Description:**  
Registered domain and pointed DNS to ALB for easy access.

**Screenshot:**  
![Route 53](<insert-your-screenshot-link-here>)

---

## 10. WordPress Installation & Demo

**Description:**  
Installed WordPress, connected to RDS, and demonstrated scaling by simulating traffic.

**Screenshot:**  
![WordPress Demo](<insert-your-screenshot-link-here>)

---

## Security Measures

- Private subnets for EC2 and RDS.
- Security Groups restrict traffic by type and source.
- NAT Gateway for outbound internet access from private resources.
- EFS for shared, secure file storage.
- ALB for DDoS resilience and SSL termination.
- Route 53 for DNS management.

---

## References

- [UserData Script](https://github.com/dareyio/script-2/blob/54e2cc7525c41259e5d522d0c454fc50a8a2c295/UserData_ALB_ASG.txt)
- AWS Documentation

---

## Additional Notes

- [Add any challenges, optimizations, or observations here]

