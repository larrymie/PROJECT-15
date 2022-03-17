# PROJECT-15
**PROJECT 15 AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

**Implementation guide 

Created a VPC. IMAGE01
 ![01](https://user-images.githubusercontent.com/91284177/158612376-60e11bbf-8403-46c2-86ef-4026a8a3bc4b.png)
 Created 8 different subnets across the two availability zones. IMAGE02
 ![02](https://user-images.githubusercontent.com/91284177/158617360-0fcc51ed-f219-4f17-9369-22962e22897d.png)
 Created an Internet Gateway (IGW)for the public subnets to have access to the internet. IMAGE 03
  ![03](https://user-images.githubusercontent.com/91284177/158619292-10c76593-a9ca-496f-b887-a26977f7852d.png)
 Created route table for the public subnet to use.(Public Route Table). assigned it to the subnet where the resources would be. 
 Created a route in the public route table and point to the Internet gateway. Associated the public subnets to the created route table.
 IMAGE 04  
 ![04](https://user-images.githubusercontent.com/91284177/158626907-a125f606-bdd4-48c0-8b3f-a29be2c164a0.png)

Created a NAT Gateway so that servers in the private subnet can reach the internet too. for example download NGINX. update etc (Outbound). this was created in the public subnest as indicated in the diagram. you might ctreate another Nat gateway in the second availability zone. **this might not be advised for financial reasons. however when the network is shutdown it would result in a single point of failure without access to the internet.  IMAGE 05
![05](https://user-images.githubusercontent.com/91284177/158631620-65805ac6-abf9-4a02-9d4c-7f7ea97ea9e3.png)

Created route table for the private subnet to use (Private Route Table). Created a route in the created route table and point to the NAT Gateway. also, associated 2 Nginx private subnets and 2 business private subnets (for compute only) to the private route table. IMAGE 06
![06](https://user-images.githubusercontent.com/91284177/158636662-9cdad684-34cb-4069-8dc3-3d88f47217d7.png)
Created security group for Bastion. Allowing all DevOps engineers to connect over SSH to the Bastion server. IMAGE 07
![07](https://user-images.githubusercontent.com/91284177/158648237-228aac37-5098-4783-bccc-6dd35246b82d.png)
Created security group and allow the entire world to talk to the ALB. port 80 and 443 were opened for the entire world to talk to the servers. IMAGE 08
![08](https://user-images.githubusercontent.com/91284177/158648271-ccdcac8b-4f55-4219-89fb-f4ee6abe8ff9.png)
Created security group and allow the ALB to talk to the Nginx proxy server, also allow developers to access the bastion server via SSH. IMAGES 9, 10.
![10](https://user-images.githubusercontent.com/91284177/158831380-5dbf1304-67fb-4891-81ea-ecf0ec9b10a4.png)
Created an External facing Application Load Balancer (ALB), also created a Listener (port 80) and target group IMAGE 11
![11](https://user-images.githubusercontent.com/91284177/158844344-5c01acee-62c8-48dc-b192-c97e934240bc.png)
Created a Launch Template for nginx, Ec2 instance automatically follows this template to scale in and out (Use a redhat based AMI) and also Created an auto scaling group ASG with specifications to route traffic to any of the available heathy server. IMAGES 12 & 13
![12](https://user-images.githubusercontent.com/91284177/158852007-719fe25a-c386-4ad9-8253-4d9b6bdc0e48.png)
![13](https://user-images.githubusercontent.com/91284177/158852052-a041fd2e-259b-47f9-a9c6-95b11c2b4171.png)



18. Create a Launch Template for Bastion 
19. Create ASG for Bastion
20. Connect to Bastion server launched in the Public Subnet
21. Connect to the nginx server launched in the Private Subnet
22. create a userdata.sh script to bring up nginx 

sudo yum update -y
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-7.rpm
sudo yum install -y nginx git
sudo systemctl restart nginx

20. Ensure that the nginx Target group is healthy
    1.  Check the security of the instance and ensure it allows port 80
21. Create Route53 entry. Point the DNS to the ALB
22. Create security group for internal ALB. (Alow traffic from Nginx proxy)
23. Create Security group for Wordpress site (Allow traffic from internal ALB)
24. Create Security group for Tooling site (Allow traffic from internal ALB)
25. Create target group for Wordpress site
26. Create target group for tooling site
27. Create internal ALB and configure listeners with Host header rules for both wordpress and tooling sites




Annex
ssh-add --apple-use-keychain ~/.ssh/devops.cer35.177.96.172
