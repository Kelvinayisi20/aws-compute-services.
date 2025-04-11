# aws-compute-services.

AWS EC2 Auto Scaling Setup with Load Balancer and Apache Web Server
This guide outlines the step-by-step process I followed to launch an EC2 instance using the AWS Management Console, configure a custom VPC, install Apache, and set up an Auto Scaling Group (ASG) with a Load Balancer. All steps were performed manually using the AWS Console.

Table of Contents
•	Overview
•	VPC and Networking Configuration
•	Security Group Configuration
•	EC2 Launch Template
•	Target Group and Load Balancer
•	Auto Scaling Group Setup
•	Installing Apache Web Server
•	Testing the Setup
•	Challenges Faced
•	Conclusion
•	Screenshots

Overview
The goal of this project was to:
•	Deploy a publicly accessible web server using EC2
•	Use Apache as the web server
•	Ensure high availability and scalability using an Auto Scaling Group
•	Distribute incoming traffic using an Application Load Balancer
•	Configure all resources manually via the AWS Management Console

VPC and Networking Configuration
1.	Create a Custom VPC
o	CIDR block: 12.0.0.0/16
2.	Create Public Subnets
o	Subnet A (e.g., 12.0.1.0/24) in public-demo-1a
o	Subnet B (e.g., 10.0.3.0/24) in public-demo-1b
3.	Create and Attach an Internet Gateway
o	Attach the IGW to your custom VPC
4.	Create a Route Table
o	Add a route for 0.0.0.0/0 pointing to the IGW
o	Associate the route table with both subnets to make them public

Security Group Configuration
1.	Create a Security Group for Web Servers
o	Inbound Rules:
	HTTP (Port 80) — from 0.0.0.0/0
	SSH (Port 22) — from My IP only for secure access
o	Outbound Rules:
	Allow all traffic (default)

EC2 Launch Template
1.	Create a Launch Template
o	Base AMI: Ubuntu
o	Attach the custom security group
o	Add the following User Data script:
#!/bin/bash
yes | sudo apt update
yes | sudo apt install apache2
echo "<h1>Server Details</h1><p><strong>Hostname:</strong> $(hostname)</p><p><strong>IP Address:</strong> $(hostname -I | cut -d' ' -f1)</p>" > /var/www/html/index.html
sudo systemctl restart apache2

Target Group and Load Balancer
1.	Create a Target Group
o	Type: Instance
o	Protocol: HTTP
o	Port: 80
o	Health check path: /
2.	Create an Application Load Balancer (ALB)
o	Type: Internet-facing
o	Attach it to the two public subnets (different public subnets)
o	Assign the security group that allows HTTP traffic
o	Configure listener on port 80 to forward requests to the target group

Auto Scaling Group Setup
1.	Create the Auto Scaling Group
o	Use the previously created Launch Template
o	Select the two public subnets (different public subnets)
o	Attach to the ALB’s target group
2.	Capacity Configuration
o	Desired capacity: 2
o	Minimum: 1
o	Maximum: 3
3.	Scaling Policy
o	Used manual/static scaling for initial setup

Installing Apache Web Server
Apache is installed automatically through the user data script in the Launch Template. This ensures every new instance added by the ASG is pre-configured to serve a simple HTML page.

Testing the Setup
•	Visited the Load Balancer’s DNS name via browser
•	Confirmed that the HTML page was rendered correctly
•	Terminated one instance to test Auto Scaling
•	Verified that a new instance was automatically launched and resumed serving the webpage

Challenges Faced
1.	VPC Misconfiguration
o	Forgot to associate subnets with the route table — caused internet access failure
2.	Security Group Rules
o	Initially missed adding HTTP port — couldn’t access web server externally
3.	Health Check Failures
o	Apache didn’t start in time, leading to unhealthy targets — resolved by improving the user data script
4.	ASG Instance Launch Issues
5.	Only selected one AZ — fixed by using subnets across two different public subnets 
6.	Apache Not Running
o	User data script had syntax issues — corrected and tested manually before final deployment

Conclusion
This project helped reinforce key concepts in deploying scalable, highly available architecture using EC2, Load Balancers, and Auto Scaling Groups — all done manually through the AWS Console. The result is a robust setup capable of handling increased load and maintaining uptime even if instances fail.



