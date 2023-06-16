# AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY

 In this project, i built a secure infrastructure inside **AWS VPC (Virtual Private Cloud)** network for a company that uses **WordPress CMS** for its main business website, and a Tooling Website for their DevOps team.

 As part of the company’s desire for improved security and performance, a decision has been made to use a **reverse proxy** technology from **NGINX** to achieve this.

 Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below, i ensured that infrastructure for both websites, WordPress and Tooling, is resilient to Web Server’s failures, can accomodate to increased traffic and, at the same time, has reasonable cost.

![Secured INfrastructure](./images/Secure%20Infrastructure%20Architeture%20for%202%20Companies%20Website.png)

*Create a free domain name for your fictitious company any domain registrar of your choice.*

*Create a hosted zone in AWS, and map it to your.*

## SET UP A VIRTUAL PRIVATE NETWORK (VPC)

1. Create a VPC

2. Create subnets as shown in the architecture

3. Create a route table and associate it with public subnets

4. Create a route table and associate it with private subnets

5. Create an Internet Gateway

6. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accisble from the Internet)

7. Create 3 Elastic IPs

8. Create a Nat Gateway and assign one of the Elastic IPs (Note: The other 2 will be used by Bastion hosts)

9. Create a Security Group for:

- External Application Load Balancer: Access to ALB will be available from the Internet

- Nginx Servers: Access to Nginx should only be allowed from the external Application Load balancer (ALB), and the bastion server (for use by the engineer).

- Bastion Servers: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address.

- Internal Application Load Balancer: Access to int ALB should be alowed from only the nginx webservers and bastion servers

- Webservers (Wordpress and Tooling): Access to website Webservers should only be allowed from the internal ALB.

- Data Layer: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully desinged – only webservers and engineers that need to query data should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.

### Proceed With Compute Resources

You will need to set up and configure compute resources inside your VPC. The recources related to compute are:

- EC2 Instances
- Launch Templates
- Target Groups
- Autoscaling Groups
- TLS Certificates
- Application Load Balancers (ALB)

### Set Up Compute Resources for **Nginx**

Provision EC2 Instances for Nginx

1. Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) in any 2 Availability Zones (AZ) in any AWS Region (it is recommended to use the Region that is closest to your customers).

2. Ensure that it has the following software installed:

```
python

ntp

net-tools

vim

wget

telnet

epel-release

htop
```

3. Create an AMI out of the EC2 instance

- Prepare Launch Template For Nginx (One Per Subnet)

 1. Make use of the AMI to set up a launch template

 2. Ensure the Instances are launched into a private subnet

 3. Assign appropriate security group

 4. Configure Userdata to update yum package repository and install nginx

- Configure Target Groups

 1. Select Instances as the target type

 2. Ensure the protocol HTTPS on secure TLS port 443

 3. Ensure that the health check path is /healthstatus

 4. Register Nginx Instances as targets

 5. Ensure that health check passes for the target group

- Configure Autoscaling For Nginx

 1. Select the right launch template

 2. Select the VPC

 3. Select both private subnets

 4. Enable Application Load Balancer for the AutoScalingGroup (ASG)

 5. Select the target group you created before

 6. Ensure that you have health checks for both EC2 and ALB

 7. The desired capacity is 2

 8. Minimum capacity is 2

 9. Maximum capacity is 4

 10. Set scale out if CPU utilization reaches 90%

 11. Ensure there is an SNS topic to send scaling notifications

### Set Up Compute Resources for **Bastion**

Provision the EC2 Instances for Bastion

1. Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) per each Availability Zone in the same Region and same AZ where you created Nginx server

2. Ensure that it has the following software installed

```
python
ntp
net-tools
vim
wget
telnet
epel-release
htop
```

3. Associate an Elastic IP with each of the Bastion EC2 Instances

4. Create an AMI out of the EC2 instance

- Prepare Launch Template For Bastion (One per subnet)

1. Make use of the AMI to set up a launch template

2. Ensure the Instances are launched into a public subnet

3. Assign appropriate security group

4. Configure Userdata to update yum package repository and install Ansible and git

- Configure Target Groups

1. Select Instances as the target type

2. Ensure the protocol is TCP on port 22

3. Register Bastion Instances as targets

4. Ensure that health check passes for the target group

- Configure Autoscaling For Bastion

1. Select the right launch template

2. Select the VPC

3. Select both public subnets

4. Enable Application Load Balancer for the AutoScalingGroup (ASG)

5. Select the target group you created before

6. Ensure that you have health checks for both EC2 and ALB

7. The desired capacity is 2

8. Minimum capacity is 2

9. Maximum capacity is 3

10. Set scale out if CPU utilization reaches 90%

11. Ensure there is an SNS topic to send scaling notifications


### Set Up Compute Resources for **Webservers**

Provision the EC2 Instances for Webservers

Now, you will need to create 2 separate launch templates for both the WordPress and Tooling websites

1. Create an EC2 Instance (Centos) each for WordPress and Tooling websites per Availability Zone (in the same Region).

2. Ensure that it has the following software installed

```
python
ntp
net-tools
vim
wget
telnet
epel-release
htop
php
```

3. Create an AMI out of the EC2 instance

- Prepare Launch Template For Webservers (One per subnet)

1. Make use of the AMI to set up a launch template

2. Ensure the Instances are launched into a public subnet

3. Assign appropriate security group

4. Configure Userdata to update yum package repository and install **wordpress** (Only required on the WordPress launch template)

- Configure Target Groups

 1. Select Instances as the target type

 2. Ensure the protocol HTTPS on secure TLS port 443

 3. Ensure that the health check path is /healthstatus

 4. Register webservers Instances as targets

 5. Ensure that health check passes for the target group

- Configure Autoscaling For Webserver

 1. Select the right launch template

 2. Select the VPC

 3. Select both private subnets

 4. Enable Application Load Balancer for the AutoScalingGroup (ASG)

 5. Select the target group you created before

 6. Ensure that you have health checks for both EC2 and ALB

 7. The desired capacity is 2

 8. Minimum capacity is 2

 9. Maximum capacity is 4

 10. Set scale out if CPU utilization reaches 90%

 11. Ensure there is an SNS topic to send scaling notifications

### TLS Certificates From Amazon Certificate Manager (ACM)

You will need TLS certificates to handle secured connectivity to your Application Load Balancers (ALB).

1. Navigate to AWS ACM

2. Request a public wildcard certificate for the domain name you registered

3. Use DNS to validate the domain name

4. Tag the resource










### **CONFIGURE APPLICATION LOAD BALANCER (ALB)**

**Application Load Balancer To Route Traffic To NGINX**

Nginx EC2 Instances will have configurations that accepts incoming traffic only from Load Balancers. No request should go directly to Nginx servers. With this kind of setup, we will benefit from intelligent routing of requests from the ALB to Nginx servers across the 2 Availability Zones. We will also be able to offload SSL/TLS certificates on the ALB instead of Nginx. Therefore, Nginx will be able to perform faster since it will not require extra compute resources to valifate certificates for every request.

1. Create an Internet facing ALB

Ensure that it listens on HTTPS protocol (TCP port 443)

Ensure the ALB is created within the appropriate VPC | AZ | Subnets

Choose the Certificate from ACM

Select Security Group

Select Nginx Instances as the target group

Application Load Balancer To Route Traffic To Web Servers

Since the webservers are configured for auto-scaling, there is going to be a problem if servers get dynamically scaled out or in. Nginx will not know about the new IP addresses, or the ones that get removed. Hence, Nginx will not know where to direct the traffic.

To solve this problem, we must use a load balancer. But this time, it will be an internal load balancer. Not Internet facing since the webservers are within a private subnet, and we do not want direct access to them.

Create an Internal ALB

Ensure that it listens on HTTPS protocol (TCP port 443)

Ensure the ALB is created within the appropriate VPC | AZ | Subnets

Choose the Certificate from ACM

Select Security Group

Select webserver Instances as the target group

Ensure that health check passes for the target group

**NOTE:** This process must be repeated for both **WordPress* and **Tooling websites.**

Setup EFS

Amazon Elastic File System (Amazon EFS) provides a simple, scalable, fully managed elastic Network File System (NFS) for use with AWS Cloud services and on-premises resources. In this project, we will utulize EFS service and mount filesystems on both Nginx and Webservers to store data.

Create an EFS filesystem

Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer

Associate the Security groups created earlier for data layer.

Create an EFS access point. (Give it a name and leave all other settings as default)

Setup RDS

Pre-requisite: Create a KMS key from Key Management Service (KMS) to be used to encrypt the database instance.

Amazon Relational Database Service (Amazon RDS) is a managed distributed relational database service by Amazon Web Services. This web service running in the cloud designed to simplify setup, operations, maintenance & scaling of relational databases.

To ensure that yout databases are highly available and also have failover support in case one availability zone fails, we will configure a multi-AZ set up of RDS MySQL database instance. In our case, since we are only using 2 AZs, we can only failover to one, but the same concept applies to 3 Availability Zones. 


To configure RDS, follow steps below:

Create a subnet group and add 2 private subnets (data Layer)

Create an RDS Instance for mysql 8.*.*

To satisfy our architectural diagram, you will need to select either Dev/Test or Production Sample Template. But to minimize AWS cost, you can select the Do not create a standby instance option under Availability & durability sample template (The production template will enable Multi-AZ deployment)

Configure other settings accordingly (For test purposes, most of the default settings are good to go). In the real world, you will need to size the database appropriately. You will need to get some information about the usage. If it is a highly transactional database that grows at 10GB weekly, you must bear that in mind while configuring the initial storage allocation, storage autoscaling, and maximum storage threshold.

Configure VPC and security (ensure the database is not available from the Internet)

Configure backups and retention

Encrypt the database using the KMS key created earlier

Enable CloudWatch monitoring and export Error and Slow Query logs (for production, also include Audit)

Note This service is an expensinve one. Ensure to review the monthly cost before creating. (DO NOT LEAVE ANY SERVICE RUNNING FOR LONG)

Configuring DNS with Route53

Earlier in this project you registered a domain and configured a hosted zone in Route53. But that is not all that needs to be done as far as DNS configuration is concerned.

You need to ensure that the main domain for the WordPress website can be reached, and the subdomain for Tooling website can also be reached using a browser.

Create other records such as CNAME, alias and A records.

NOTE: You can use either CNAME or alias records to achieve the same thing. 

Create an alias record for the root domain and direct its traffic to the ALB DNS name.

Create an alias record for tooling.<yourdomain>.com and direct its traffic to the ALB DNS name.