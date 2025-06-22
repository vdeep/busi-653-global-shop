# busi-653-global-shop

Steps to setup the AWS Resources

## 1. VPC

1. Select `Create VPC` from the VPC dashboard.
2. Choose `VPC and more` in the "Resources to create" option.
3. Enter "GlobalShop" for the name tag.
4. Choose 1 Availability Zones. Select "us-east-1a".
5. Choose 1 for Number of public and private subnets.
6. Change IPv4 CIDR:
7. Public subnet: 10.0.0.0/24
8. Private subnet: 10.0.1.0/24
9. For NAT Gateways, select "In 1 AZ"
10. Press `Create VPC`

## 2. Security Group

1. Select `Create Security Group` from the Security Group panel.
2. Enter "GlobalShop" for the security group name.
3. Enter "Allow HTTP and SSH Access" for the description.
4. Select the "GlobalShop-vpc" from the VPC dropdown.
5. Add following inbound rules:
6. Type: HTTP, Source: Anywhere-IPv4, Description: Allow HTTP Traffic
7. Type: SSH, Source: Anywhere-IPv4, Description: Allow SSH Traffic
8. Press "Create security group".

## 3. Create EC2 Launch Template

1. Go to EC2 -> Launch Templates -> Create launch template
2. Enter "GlobalShop-LT" for the template name
3. Select "Amazon Linux 2023 kernel-6.1" for the AMI
4. Select "t2.micro" for the Instance type
5. Create a new key pair -> Enter "GlobalShop-key-pair" for the name. Save the pem file.
6. Select the "GlobalShop" security group
7. Add the following user data to setup the instance:

  ```
  #!/bin/bash
  yum update -y
  yum install -y httpd git
  systemctl start httpd
  systemctl enable httpd

  # Clean the default html directory
  rm -rf /var/www/html/*

  # Clone your GitHub repository
  git clone https://github.com/vdeep/busi-653-global-shop.git /var/www/html

  # Set permissions
  chown -R apache:apache /var/www/html
  chmod -R 755 /var/www/html
  ```

8. Press "Create Launch Template"

## 4. Create Application Load Balander

1. Go to EC2 -> Load Balancers -> Create Load Balancer -> Choose Application Load Balancer
2. Name: GlobalShop-ALB
3. Scheme: Internet-facing
4. Select the 2 public subnets created above
5. Security Group: Allow HTTP (80)
6. Target Group: Create new with following settings
7. Type: Instance
8. Protocol: HTTP
9. Health checks: /
10. Name: GlobalShop-TG
11. Create the Application Load Balancer

## 5. Configure Auto Scaling Group

1. Go to EC2 -> Auto Scaling Group -> Create
2. Name: GlobalShop-ASG
3. Launch template: use the one created above (GlobalShop-LT)
4. VPC: Select GlobalShop-VPC, use both private subnets
5. Load Balancer
6. Select "GlobalShop-ALB"
7. Use "GlobalShop-TG" Target group
8. Scaling policy:
9. Desired: 2
10. Min: 1
11. Max: 4
12. Add policy: Scale out at 50% CPU Utilization

## 6. Enable Monitoring

1. Go to CloudWatch -> Alarms -> Create Alarm
2. Metric: EC2 -> Auto Scaling -> Group Metrics -> GroupAverageCPUUtilization
3. Threshold: Greater Than - 50%
4. Notification: Create a new topic and add subscribe selecting that topic and your email.

## 7. Test the site

1. Go to ALB
2. Copy the Public DNS for the "GlobalShop-ALB"
3. Prefix the Public DNS with "http://" and open in a new tab. The site should be visible.

## 8. Clean up

1. Terminate ASG
2. Terminate EC2 instances
3. Delete ALB
4. Delete Launch Template
5. Delete VPC and related subnets, route tables, Internet Gateway, and NAT Gateway.
