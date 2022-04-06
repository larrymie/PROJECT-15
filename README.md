# PROJECT-15
**PROJECT 15 AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

**Implementation guide 
I built a securde infrastructure inside AWS VPC (Virtual Private Cloud) network for a Jolan company, that uses WordPress CMS for its main business website, and a Tooling Website ('wordpress.jolan20.click' and 'tooling.jolan20.click')  for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this.

Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below. I ensured that infrastructure for both websites, WordPress and Tooling, is resilient to Web Server’s failures, can accomodate to increased traffic and, at the same time, has reasonable cost.
IMAGE 01 ![1](https://user-images.githubusercontent.com/91284177/161916370-b6a1befd-00fd-4dd6-babe-cdd8337f83e7.png)

I set Up a Virtual Private Network (VPC). I created a VPC IMAGE 02
![02](https://user-images.githubusercontent.com/91284177/161916067-c5692c7b-8093-4c92-8f52-2c1d5fe0f98a.png)

I created subnets as shown in the architecture IMAGE 03
![03](https://user-images.githubusercontent.com/91284177/161917877-99181604-656b-4f42-9033-fe0be04bf114.png)

Created a route table and associate it with public subnets
Created a route table and associate it with private subnets
Created an Internet Gateway
Edited a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accisble from the Internet)
Create an Elastic IPs
Create a Nat Gateway and assigned the of the Elastic IPs
Created a Security Group for:

Nginx Servers: Access to Nginx should only be allowed from a Application Load balancer (ALB). At this point, we have not created a load balancer, therefore we will update the rules later. For now, just create it and put some dummy records as a place holder.
Bastion Servers: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. To get this information, simply go to your terminal and type curl www.canhazip.com
Application Load Balancer: ALB will be available from the Internet
Webservers: Access to Webservers should only be allowed from the Nginx servers. Since we do not have the servers created yet, just put some dummy records as a place holder, we will update it later.
Data Layer: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully desinged – only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint. MAGE 04
![04](https://user-images.githubusercontent.com/91284177/161918960-c4f48f6d-bea0-46b1-b005-91b6053ebd0e.png)

Proceed With Compute Resources
**I set up and configured compute resources inside my VPC. The recources related to compute are:

EC2 Instances IMAGE 05
![05](https://user-images.githubusercontent.com/91284177/161919298-3254fa6b-b45a-4c8b-b00c-1926d0970576.png)
Launched Templates IMAGE 06
![06](https://user-images.githubusercontent.com/91284177/161919594-d1be0ed2-652f-406a-ad9a-76b48be88f0a.png)
Target Groups IMAGE 07
![07](https://user-images.githubusercontent.com/91284177/161919724-3135e657-ba5e-4f1d-adec-0369dc71540c.png)
Autoscaling Groups IMAGE 08
![08](https://user-images.githubusercontent.com/91284177/161919818-9d5ff5f8-8ac3-4282-b8f2-f146dac47cbf.png)

TLS Certificates
Application Load Balancers (ALB) Internet and internal load balancers IMAGE 09
![09](https://user-images.githubusercontent.com/91284177/161920584-5f6d3d41-8f1c-426c-8ac9-4371dd0a1971.png)

**Set Up Compute Resources for Nginx
Provisioned EC2 Instances for Nginx
I created an EC2 Instance based on Red hat Amazon Machine Image (AMI) in any 2 Availability Zones (AZ) in any AWS Region (it is recommended to use the Region that is closest to your customers). I used EC2 instance of T2 family (e.g. t2.micro)

I Ensure that it has the following software installed:

python
ntp
net-tools
vim
wget
telnet
epel-release
htop
I Created an AMI out of the EC2 instances

I Prepared Launch Template For Nginx (One Per Subnet)
I Made use of the AMI to set up a launch template
Ensured the Instances were launched into a public subnet and assigned appropriate security groups
Configure Userdata to update yum package repository and install nginx
Configured Target Groups IMAGE 10
![10](https://user-images.githubusercontent.com/91284177/161921570-e2026bc2-688b-4664-bc66-c4defe24c170.png)
Selected Instances as the target type
Ensure the protocol HTTPS on secure TLS port 443
Ensure that the health check path is /healthstatus
Register Nginx Instances as targets
Ensure that health check passes for the target group IMAGE 11, 12 & 13
![11](https://user-images.githubusercontent.com/91284177/161922101-d4bc703f-c6f7-4142-8930-e1599c90661e.png)
![12](https://user-images.githubusercontent.com/91284177/161922106-0bdd2ab0-6d77-4aba-81af-37af8f654f7c.png)
![13](https://user-images.githubusercontent.com/91284177/161922141-c0b456ba-eabf-42a9-bc49-123cfbd826ac.png)
Configured Autoscaling For Nginx
Selecedt the right launch template
Selected the VPC
Selected both public subnets
Enabled Application Load Balancer for the AutoScalingGroup (ASG)
Selected the target group i created before
Ensured I have health checks for both EC2 and ALB
The desired capacity is 2, minimum capacity is 2 while maximum capacity is 4
Set scale out if CPU utilization reaches 90%
Ensured there is an SNS topic to send scaling notifications
I Set Up Compute Resources for Bastion
Provisioned the EC2 Instances for Bastion
Created an EC2 Instance based on CentOS Amazon Machine Image (AMI) per each Availability Zone in the same Region and same AZ where you created Nginx server
Ensured that it has the following software installed

python
ntp
net-tools
vim
wget
telnet
epel-release
htop
I associated an Elastic IP with each of the Bastion EC2 Instances
Create an AMI out of the EC2 instance
Preparedd Launch Template For Bastion (One per subnet)
Made use of the AMI to set up a launch template
Ensured the Instances were launched into a public subnet
Assigned appropriate security group
Configured Userdata to update yum package repository and instaledl Ansible and git
Configurde Target Groups
Selected Instances as the target type
Ensurde the protocol is TCP on port 22
Registered Bastion Instances as targets
Ensured that health check passed for the target group
Configured Autoscaling For Bastion
Selected the right launch template
Selected the VPC
Selected both public subnets
Enabled Application Load Balancer for the AutoScalingGroup (ASG)
Selected the target group I created before
Ensured that I have health checks for both EC2 and ALB
The desired capacity is 2, the minimum capacity is 2 while the Maximum capacity is 4
Set scale out if CPU utilization reaches 90%
Ensured there is an SNS topic to send scaling notifications
Set Up Compute Resources for Webservers.
Provisioned the EC2 Instances for Webservers
I created 2 separate launch templates for both the WordPress and Tooling websites. IMAGES 14 & 15
![14](https://user-images.githubusercontent.com/91284177/161923922-ad596418-3b28-4c53-8dd8-53729bb0e0a4.png)
![15](https://user-images.githubusercontent.com/91284177/161923934-ce955762-a217-4217-9eea-caa19ccb1b26.png)

Created an EC2 Instance (RED HAT) for WordPress and Tooling websites per Availability Zone (in the same Region).

Ensured that it had the following software installed

python
ntp
net-tools
vim
wget
telnet
epel-release
htop
php
Created an AMI out of the EC2 instance

Prepared Launch Template For Webservers (One per subnet)
Made use of the AMI to set up a launch template
Ensured the Instances are launched into a public subnet
Assigned appropriate security group
Configured Userdata to update yum package repository and install wordpress (Only required on the WordPress launch template)
TLS Certificates From Amazon Certificate Manager (ACM)
I used TLS certificates to handle secured connectivity to my Application Load Balancers (ALB).

Navigated to AWS ACM
Request a public wildcard certificate for the domain name I registered in AWS, Used DNS to validate the domain name and tagged the resource


**CONFIGURE APPLICATION LOAD BALANCER (ALB)
Application Load Balancer To Route Traffic To NGINX
Nginx EC2 Instances will have configurations that accepts incoming traffic only from Load Balancers. No request should go directly to Nginx servers. With this kind of setup, this made sense as it would be  routing requests from the ALB to Nginx servers across the 2 Availability Zones. it will also be able to offload SSL/TLS certificates on the ALB instead of Nginx. Therefore, Nginx will be able to perform faster since it will not require extra compute resources to validate certificates for every request.

Create an Internet facing ALB. 
Ensure that it listens on HTTPS protocol (TCP port 443)
Ensure the ALB is created within the appropriate VPC | AZ | Subnets
Choose the Certificate from ACM
Select Security Group
Select Nginx Instances as the target group
Application Load Balancer To Route Traffic To Web Servers
Since the webservers are configured for auto-scaling, there is going to be a problem if servers get dynamically scalled out or in. Nginx will not know about the new IP addresses, or the ones that get removed. Hence, Nginx will not know where to direct the traffic.

To solve this problem, we must use a load balancer. But this time, it will be an internal load balancer. Not Internet facing since the webservers are within a private subnet, and we do not want direct access to them.

Created an Internal ALB. IMAGE 16
![16](https://user-images.githubusercontent.com/91284177/161925391-53f47b04-970a-4392-b469-e3b4b9935759.png)
 
Ensured that it listens on HTTPS protocol (TCP port 443)
Ensurde the ALB is created within the appropriate VPC | AZ | Subnets
Chose the Certificate from ACM
Selected Security Group
Selecedt webserver Instances as the target group
Ensured that health check passes for the target group
NOTE: I repeated for both WordPress and Tooling websites.

**Setup EFS
Amazon Elastic File System (Amazon EFS) provides a simple, scalable, fully managed elastic Network File System (NFS) for use with AWS Cloud services and on-premises resources. In this project, we will utulize EFS service and mount filesystems on both Nginx and Webservers to store data.

Created an EFS filesystem.
Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer
Associated the Security groups created earlier for data layer.
Created an EFS access point. IMAGE 17
![17](https://user-images.githubusercontent.com/91284177/161926046-19488a90-44bd-427d-838d-d586ca30cd62.png)


**Setup RDS

Pre-requisite: Created a KMS key from Key Management Service (KMS) to be used to encrypt the database instance. IMAGE 18
![18](https://user-images.githubusercontent.com/91284177/161926203-f89d6d87-a46a-4220-be73-65277342185c.png)

**Amazon Relational Database Service (Amazon RDS) is a managed distributed relational database service by Amazon Web Services. This web service running in the cloud designed to simplify setup, operations, maintenans & scaling of relational databases. Without RDS, Database Administrators (DBA) have more work to do, due to RDS, some DBAs have become jobless

To ensure that my databases are highly available and also have failover support in case one availability zone fails, I configured a multi-AZ set up of RDS MySQL database instance. In my case, since I only used 2 AZs, I can only failover to one, but the same concept applies to 3 Availability Zones. I will not consider possible failure of the whole Region, but for this AWS also has a solution – this is a more advanced concept that will be discussed in following projects.

To configure RDS, I followed steps below:

Created a subnet group and add 2 private subnets (data Layer)
Created an RDS Instance for mysql 8.*.*
To satisfy my architectural diagram, I selected either Dev/Test or Production Sample Template. But to minimize AWS cost, I select the do not create a standby instance option under Availability & durability sample template (The production template will enable Multi-AZ deployment)
Configured other settings accordingly (For test purposes, most of the default settings are good to go). In the real world, I will need to size the database appropriately. If it is a highly transactional database that grows at 10GB weekly, I must bear that in mind while configuring the initial storage allocation, storage autoscaling, and maximum storage threshold.
Configured VPC and security (ensure the database is not available from the Internet)
Configured backups and retention
Encrypedt the database using the KMS key created earlier
Enabled CloudWatch monitoring and export Error and Slow Query logs (for production, also include Audit)
Note This service is an expensinve one. Ensure to review the monthly cost before creating. (DO NOT LEAVE ANY SERVICE RUNNING FOR LONG)

**Configuring DNS with Route53
Earlier in this project you registered a free domain with Freenom and configured a hosted zone in Route53. But that is not all that needs to be done as far as DNS configuration is concerned.

I ensured that the main domain for the WordPress website can be reached, and the subdomain for Tooling website can also be reached using a browser.
IMAGES 19, 20 & 21
![19](https://user-images.githubusercontent.com/91284177/161928134-8d78f304-a1f8-451b-87b7-4b2d6716a36d.png)
![20](https://user-images.githubusercontent.com/91284177/161928175-a2649d61-a2bf-4774-a484-4406c66be02f.png)
![21](https://user-images.githubusercontent.com/91284177/161928188-4a5fa65f-9780-4e08-b9c0-ebb1a53721e3.png)








