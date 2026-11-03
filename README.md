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

![VPC resource map](screenshots/01-vpc-resource-map.png)
![Subnets](screenshots/02-subnets.png)
![Internet Gateway](screenshots/03-internet-gateway.png)
![NAT Gateway](screenshots/04-nat-gateway.png)
![Route tables](screenshots/05-route-tables.png)

### 2. Security Groups and EC2 Instances
Created separate security groups for web and database tiers, then launched the Web/Application server, Jump server and Database server.

![Security groups](screenshots/06-security-groups.png)
![EC2 instances](screenshots/07-ec2-instances.png)

### 3. S3 and RDS
Stored loan product images in S3 and created an RDS MySQL instance in the private subnets.

![S3 bucket](screenshots/08-s3-images.png)
![RDS database](screenshots/09-rds.png)

### 4. Route 53 and Domain
Created a public hosted zone and updated the GoDaddy nameservers to the Route 53 nameservers.

![Route 53 hosted zone](screenshots/10-route53.png)
![GoDaddy nameservers](screenshots/11-godaddy-nameservers.png)

### 5. Web Server Setup (Nginx)
Transferred application files with WinSCP, installed Nginx, added a custom `quickloan.conf` server block, validated with `nginx -t`, and reloaded the service.

![WinSCP transfer](screenshots/12-winscp.png)
![Nginx default page](screenshots/13-nginx-welcome.png)
![Nginx config](screenshots/14-nginx-config.png)
![QuickLoan site](screenshots/15-quickloan-site.png)

### 6. Application and Database Verification
Loan application form submits data into the `applications` table in RDS MySQL.

![Apply form](screenshots/16-apply-form.png)
![Database records](screenshots/17-database-records.png)

### 7. Application Load Balancer
Placed the web servers behind an ALB and mapped the domain to it.

![ALB DNS](screenshots/18-alb.png)
![ALB with domain](screenshots/19-alb-domain.png)

### 8. AMI, Launch Template and Auto Scaling
Created an AMI (`QL-image-web`) from the configured web server, a Launch Template, and an Auto Scaling Group (desired 2, min 2, max 5). Web Server 2 and 3 were created automatically by the ASG.

![AMI](screenshots/20-ami.png)
![Launch template](screenshots/21-launch-template.png)
![Auto Scaling Group](screenshots/22-asg.png)
![ASG-created instances](screenshots/23-asg-instances.png)

### 9. Monitoring and Alerts
- **Scale-out alarm:** CPU > 80% triggers adding instances
- **Scale-in alarm:** CPU < 20% triggers removing instances
- **SNS topic** sends email notifications on alarm state changes
- **CloudWatch dashboard** shows CPU utilization

![SNS topic](screenshots/24-sns.png)
![CloudWatch alarms](screenshots/25-alarms.png)
![Scale-in alarm](screenshots/26-scalein-alarm.png)
![Scale-out alarm](screenshots/27-scaleout-alarm.png)
![CPU dashboard](screenshots/28-dashboard.png)
![Alert email](screenshots/29-alert-email.png)
![Dashboard at low CPU](screenshots/32-dashboard-idle.png)
![Scale-in alert email](screenshots/30-scalein-email.png)
![Instances terminating after scale-in](screenshots/31-instances-terminating.png)

## Auto Scaling Test
Generated CPU load on a web server; CPU rose above 80%, the scale-out alarm fired, and an email alert was received. When load dropped, the scale-in alarm terminated the extra instances.

## Key Learnings
- Designing a VPC with public and private tiers and controlling traffic with route tables and security groups
- Keeping the database private and reaching it only through a jump server
- Building a reusable AMI and Launch Template for consistent scaling
- Configuring CloudWatch alarms and SNS to automate scaling and notify on events

## Author
**Surajsinh Pawar**: Aspiring Cloud & DevOps Engineer
