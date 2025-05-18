# VPC3TierJava
## Student Management Web Application – AWS Deployment Guide
## What This App Does

- Manage student records through a simple web interface  
- Supports adding new students, editing details, deleting entries, and listing all students  
- Uses MySQL to store data persistently  
- Dockerized so it runs consistently anywhere  
- Designed to be deployed securely on AWS using a 3-subnet VPC  

---
![](https://github.com/gaurav3972/VPC3TierJava/blob/main/THREE%20TIER%20USING%20JAVA/structure.png)
## Our AWS Setup Explained

We wanted to make sure the app is secure and runs smoothly on AWS, so here’s how we set it up:

- **Public Subnet:** This is where the NAT Gateway and Bastion Host live. The Bastion Host helps us securely connect to our EC2 servers in private subnets.  
- **Private Subnet 1:** This subnet hosts our application server (an EC2 instance running Tomcat and our app).  
- **Private Subnet 2:** This one is for the MySQL database running on AWS RDS, isolated from the internet for security.  

This setup keeps our database safe and makes sure the app can connect to the internet when it needs to (for updates, downloads, etc.) through the NAT Gateway.

---

## Technologies We Used

- Java (JSP & Servlets)  
- MySQL database  
- Apache Tomcat server  
- AWS services: VPC, EC2, RDS, NAT Gateway, Bastion Host  

---
## Detailed Configuration and Setup Explanation

This section provides a comprehensive explanation of the configuration and setup steps required to deploy the Student Management Web Application on AWS, leveraging Java and MySQL within a secure VPC environment.

### 1. AWS VPC and Subnet Configuration

- **VPC Setup**:  
  Create a Virtual Private Cloud (VPC) with a CIDR block (e.g., `10.0.0.0/16`) to logically isolate your AWS resources.
![](https://github.com/gaurav3972/VPC3TierJava/blob/main/THREE%20TIER%20USING%20JAVA/VPC.png)
- **Subnets**:  
  - **Public Subnet** (e.g., `10.0.1.0/24`): Hosts the NAT Gateway and Bastion Host, enabling secure access and internet connectivity for private resources.  
  - **Private Subnet 1 - Application Server** (e.g., `10.0.2.0/24`): Contains the EC2 instance running the Java application server. No direct internet access; outbound traffic is routed via the NAT Gateway.  
  - **Private Subnet 2 - Database Server** (e.g., `10.0.3.0/24`): Contains the RDS MySQL instance, fully isolated from the internet and accessible only from the application subnet.
![](https://github.com/gaurav3972/VPC3TierJava/blob/main/THREE%20TIER%20USING%20JAVA/SUBNETS.png)
- **Internet Gateway and NAT Gateway**:  
  Attach an Internet Gateway to the VPC to enable internet access. Set up a NAT Gateway in the public subnet to allow instances in private subnets to access the internet for updates and patches without exposing them publicly.
![](https://github.com/gaurav3972/VPC3TierJava/blob/main/THREE%20TIER%20USING%20JAVA/PRIVET-RT.png)
- **Route Tables**:  
  Configure route tables so that the public subnet routes internet traffic directly through the Internet Gateway, while private subnets route outbound traffic through the NAT Gateway.

---

### 2. Security Groups and Network ACLs
![](https://github.com/gaurav3972/VPC3TierJava/blob/main/THREE%20TIER%20USING%20JAVA/SG.png)
- **Application Server Security Group**:  
  - Allow inbound HTTP (port 8080) from trusted IPs or load balancers.  
  - Allow outbound traffic to the RDS MySQL port (3306).  
  - Restrict SSH access, permitting only connections from the Bastion Host.

- **Database Server Security Group**:  
  - Allow inbound MySQL (port 3306) traffic exclusively from the Application Server Security Group.  
  - Block all inbound internet traffic.  
  - Outbound traffic is usually unrestricted or limited to necessary AWS services.

- **Bastion Host Security Group**:  
  - Allow inbound SSH (port 22) from your trusted office/home IP addresses only.  
  - Allow outbound SSH to instances in the private subnet.

---

### 3. EC2 Application Server Setup

- **Instance Launch**:  
  Launch an EC2 instance in the private application subnet with an appropriate instance type (e.g., t2.micro).
![](https://github.com/gaurav3972/VPC3TierJava/blob/main/THREE%20TIER%20USING%20JAVA/EC2-INSTANCES.png  )
- **Access**:  
  Use SSH to connect to the EC2 instance by first connecting to the Bastion Host in the public subnet, then SSH into the private EC2 instance.

- **Software Installation**:  
  - Install Java Development Kit (JDK 8 or above).  
  - Install Apache Tomcat server or Docker if using containerized deployment.  
  - Transfer the application WAR file or Docker image.

- **Environment Variables**:  
  Configure the database connection environment variables for the application:

  ```bash
  export DB_HOST=<rds-endpoint>
  export DB_PORT=3306
  export DB_NAME=studentdb
  export DB_USER=admin
  export DB_PASSWORD=yourpassword
## 4. RDS MySQL Database Setup

* Instance Creation:
Create an RDS MySQL instance within the private database subnet group. Choose the desired instance class, storage, and enable Multi-AZ for high availability if needed.

* Security:
Assign a security group that only allows inbound traffic from the application server subnet.

* Database Initialization:
Connect to the RDS instance using a MySQL client and initialize the database schema by running the provided student.sql script: 

    ***mysql -h <rds-endpoint> -u admin -p studentdb < student.sql***
* Backups and Maintenance:
Enable automated backups and snapshots for data recovery. Schedule maintenance windows for updates.
![](https://github.com/gaurav3972/VPC3TierJava/blob/main/THREE%20TIER%20USING%20JAVA/DB.png)
## 5. Docker Containerization (Optional)
* Docker Build:
Use the included Dockerfile to build a container image of the Java application.

`docker build -t student-app`
* Docker Run:
Run the container locally or on the EC2 instance with proper environment variables set.

`docker run -d -p 8080:8080 \`

  `-e DB_HOST=<rds-endpoint> \`

  `-e DB_PORT=3306 \`

  `-e DB_NAME=studentdb \`

  `-e DB_USER=admin \`

 `-e DB_PASSWORD=yourpassword \
  student-app`
  ## output
## 6. Monitoring and Logging
  * AWS CloudWatch Integration:
Configure CloudWatch to collect logs from Tomcat and the EC2 instance.
Set alarms for CPU usage, disk space, and RDS health metrics.

* Log Retention:
Manage log retention policies to optimize storage costs.
## output:
![](https://github.com/gaurav3972/VPC3TierJava/blob/main/THREE%20TIER%20USING%20JAVA/OUPUT%200R%20APPLICATION.png)
## 7. Backup and Recovery Strategy
* RDS Backups:
Use AWS RDS automated backups with retention period.

* Snapshots:
Take manual snapshots before major updates or maintenance.

* EC2 Backups:
Create AMI images of the EC2 instance regularly or before changes.

* Disaster Recovery:
Store backups securely in Amazon S3 with versioning and lifecycle policies.
## linux commands:
![](https://github.com/gaurav3972/VPC3TierJava/blob/main/THREE%20TIER%20USING%20JAVA/COMMANDS%20FOR%20LINUX.png)
## 8. Cost Management Tips
* Use reserved or spot EC2 instances where applicable.

* Optimize storage with S3 Intelligent-Tiering.

* Monitor usage using AWS Cost Explorer and Budgets.

* Schedule EC2 instance downtime during off-hours if possible.

## What We Learned
* How to build a Java web app using JSP and Servlets

* Basics of Docker and containerizing applications

* Setting up and managing AWS infrastructure securely

* Networking concepts like VPCs, subnets, NAT Gateway, and Bastion Host

* Managing databases on AWS RDS

## License
**MIT License**

## Summary
Hey! This is our Student Management Web Application project built as part of our course.
It’s a Java web app that lets you add, update, delete, and view student records.
We used JSP/Servlets for the backend and MySQL for the database.To make things cool, we dockerized the app and set it up to run on AWS securely with a Virtual Private Cloud (VPC) that has three subnets one public and two private.The app server runs in one private subnet, and the database runs in another private subnet, with the public subnet handling NAT Gateway and Bastion Host for security.