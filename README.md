# QuickLoan: Highly Available 3-Tier Web Application on AWS

A loan-application website (QuickLoan) deployed on AWS using a custom VPC, public/private subnets, Nginx on EC2, MySQL on RDS, an Application Load Balancer, Auto Scaling, and CloudWatch + SNS alerting. The domain is managed with Route 53.

## Architecture

![QuickLoan AWS architecture diagram](architecture-diagram.png)

## AWS Services Used

| Area | Service / Resource |
|---|---|
| Networking | VPC `10.10.0.0/16`, 9 subnets across multiple AZs, Internet Gateway, NAT Gateway, Public/Private route tables |
| Security | Security groups (`Web-QL-sg`, `DB-QL-sg`), Jump (bastion) server, DB in private subnet |
| Compute | EC2 (t3.micro), Amazon Linux 2023, Nginx + PHP |
| Database | Amazon RDS (MySQL), database `quickloan_db`, table `applications` |
| Storage | Amazon S3 (static images for loan products) |
| DNS | Route 53 hosted zone, GoDaddy nameservers pointed to Route 53 |
| High availability | Application Load Balancer, AMI, Launch Template, Auto Scaling Group (2-5 instances) |
| Monitoring | CloudWatch alarms and dashboard, SNS topic with email subscription |

## Implementation Steps

### 1. VPC and Networking
Created `QuickLoan-Project-VPC` with public subnets (Web-1, Web-2, Jump) and private subnets (DB / RDS). Attached an Internet Gateway for public subnets and a NAT Gateway so private resources can reach the internet outbound. Separate `Public-RT` and `Private-RT` route tables.

<img src="Screenshots/01-vpc-resource-map.png.png" alt="VPC resource map" width="800">

<img src="Screenshots/02-subnets.png.png" alt="Subnets" width="800">

<img src="Screenshots/03-internet-gateway.png.png" alt="Internet Gateway" width="800">

<img src="Screenshots/04-nat-gateway.png.png" alt="NAT Gateway" width="800">

<img src="Screenshots/05-route-tables.png.png" alt="Route tables" width="800">

### 2. Security Groups and EC2 Instances
Created separate security groups for web and database tiers, then launched the Web/Application server, Jump server and Database server.

<img src="Screenshots/06-security-groups.png.png" alt="Security groups" width="800">

<img src="Screenshots/07-ec2-instances.png.png" alt="EC2 instances" width="800">

### 3. S3 and RDS
Stored loan product images in S3 and created an RDS MySQL instance in the private subnets.

<img src="Screenshots/08-s3-images.png.png" alt="S3 bucket" width="800">

<img src="Screenshots/09-rds.png.png" alt="RDS database" width="800">

### 4. Route 53 and Domain
Created a public hosted zone and updated the GoDaddy nameservers to the Route 53 nameservers.

<img src="Screenshots/10-route53.png.png" alt="Route 53 hosted zone" width="800">

<img src="Screenshots/11-godaddy-nameservers.png.png" alt="GoDaddy nameservers" width="800">

### 5. Web Server Setup (Nginx)
Transferred application files with WinSCP, installed Nginx, added a custom `quickloan.conf` server block, validated with `nginx -t`, and reloaded the service.

<img src="Screenshots/12-winscp.png.png" alt="WinSCP transfer" width="800">

<img src="Screenshots/13-nginx-welcome.png.png" alt="Nginx default page" width="800">

<img src="Screenshots/14-nginx-config.png.png" alt="Nginx config" width="800">

<img src="Screenshots/15-quickloan-site.png.png" alt="QuickLoan site" width="800">

### 6. Application and Database Verification
Loan application form submits data into the `applications` table in RDS MySQL.

<img src="Screenshots/16-apply-form.png.png" alt="Apply form" width="800">

<img src="Screenshots/17-database-records.png.png" alt="Database records" width="800">

### 7. Application Load Balancer
Placed the web servers behind an ALB and mapped the domain to it.

<img src="Screenshots/18-alb.png.png" alt="ALB DNS" width="800">

<img src="Screenshots/19-alb-domain.png.png" alt="ALB with domain" width="800">

### 8. AMI, Launch Template and Auto Scaling
Created an AMI (`QL-image-web`) from the configured web server, a Launch Template, and an Auto Scaling Group (desired 2, min 2, max 5). Web Server 2 and 3 were created automatically by the ASG.

<img src="Screenshots/20-ami.png.png" alt="AMI" width="800">

<img src="Screenshots/21-launch-template.png.png" alt="Launch template" width="800">

<img src="Screenshots/22-asg.png.png" alt="Auto Scaling Group" width="800">

<img src="Screenshots/23-asg-instances.png.png" alt="ASG-created instances" width="800">

### 9. Monitoring and Alerts
- **Scale-out alarm:** CPU > 80% triggers adding instances
- **Scale-in alarm:** CPU < 20% triggers removing instances
- **SNS topic** sends email notifications on alarm state changes
- **CloudWatch dashboard** shows CPU utilization

<img src="Screenshots/24-sns.png.png" alt="SNS topic" width="800">

<img src="Screenshots/25-alarms.png.png" alt="CloudWatch alarms" width="800">

<img src="Screenshots/26-scalein-alarm.png.png" alt="Scale-in alarm" width="800">

<img src="Screenshots/27-scaleout-alarm.png.png" alt="Scale-out alarm" width="800">

<img src="Screenshots/28-dashboard.png.png" alt="CPU dashboard" width="800">

<img src="Screenshots/29-alert-email.png.png" alt="Alert email" width="800">

<img src="Screenshots/32-dashboard-idle.png.png" alt="Dashboard at low CPU" width="800">

<img src="Screenshots/30-scalein-email.png.png" alt="Scale-in alert email" width="800">

<img src="Screenshots/31-instances-terminating.png.png" alt="Instances terminating after scale-in" width="800">

## Auto Scaling Test
Generated CPU load on a web server; CPU rose above 80%, the scale-out alarm fired, and an email alert was received. When load dropped, the scale-in alarm terminated the extra instances.

## Key Learnings
- Designing a VPC with public and private tiers and controlling traffic with route tables and security groups
- Keeping the database private and reaching it only through a jump server
- Building a reusable AMI and Launch Template for consistent scaling
- Configuring CloudWatch alarms and SNS to automate scaling and notify on events

## Author
**Surajsinh Pawar**: Aspiring Cloud & DevOps Engineer

