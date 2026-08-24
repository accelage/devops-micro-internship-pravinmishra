# Assignment 5 — Deploy a Highly Available Two-Tier Application on AWS (VPC + ALB + ASG + Multi-AZ RDS)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will design and deploy a highly available two-tier web application on AWS: highly available networking across two Availability Zones, an Application Load Balancer, an Auto Scaling Group for the web tier, and a private Multi-AZ RDS database. You must prove high availability with real failure tests.

---

# Task 1 — Create HA Networking (VPC + 4 Subnets + IGW + NAT + Route Tables)

## Goal

Build a VPC (10.0.0.0/16) with two public and two private subnets across two Availability Zones, an Internet Gateway, a NAT Gateway, and the matching public/private route tables.

### Evidence

#### Screenshot 1 — VPC details showing CIDR 10.0.0.0/16

![](screenshots/Ass5sc1.JPG)

---

#### Screenshot 2 — Subnets list showing four subnets and their Availability Zones

![](screenshots/Ass5sc2.JPG)

---

#### Screenshot 3 — Public route table showing the Internet Gateway route and both public-subnet associations

![](screenshots/Ass5sc3.JPG)

---

#### Screenshot 4 — Private route table showing the NAT Gateway route and both private-subnet associations

![](screenshots/Ass5sc4.JPG)

---

#### Screenshot 5 — NAT Gateway status showing Available and the Elastic IP

![](screenshots/Ass5sc5.JPG)

---

# Task 2 — Create Security Groups (ALB, EC2, RDS) with Least Privilege

## Goal

Create `ha-alb-sg` (HTTP public), `ha-web-sg` (HTTP only from `ha-alb-sg`, SSH from your IP), and `ha-db-sg` (database port only from `ha-web-sg`).

### Evidence

#### Screenshot 6 — ALB Security Group inbound rules

![](screenshots/Ass5sc6.JPG)

---

#### Screenshot 7 — EC2 Security Group inbound rules showing the ALB Security Group reference and SSH from your IP

![](screenshots/Ass5sc7.JPG)

---

#### Screenshot 8 — RDS Security Group inbound rule showing the database port allowed only from the EC2 Security Group

![](screenshots/Ass5sc8.JPG)

---

# Task 3 — Deploy Database Tier (RDS Multi-AZ in Private Subnets)

## Goal

Launch a private, Multi-AZ RDS database (MySQL or PostgreSQL) using the private DB Subnet Group and `ha-db-sg`.

### Evidence

#### Screenshot 9 — RDS summary showing Multi-AZ = Yes and Publicly accessible = No

![](screenshots/Ass5sc9.JPG)

---

#### Screenshot 10 — RDS connectivity section showing the DB Subnet Group and Security Group

![](screenshots/Ass5sc10.JPG)

---

# Task 4 — Build a Launch Template (User Data Installs App + Connects to DB)

## Goal

Create a Launch Template whose user data installs the web-server runtime, deploys the application, configures the database connection, and starts the required services.

### Evidence

#### Screenshot 11 — Launch Template details showing that user data exists, including a visible snippet

![](screenshots/Ass5sc11.JPG)

---

#### Screenshot 12 — A running instance created from the template showing that the application responds on port 80 through a local test or browser using its public IP

![](screenshots/Ass5sc12.JPG)

---

# Task 5 — Create an Application Load Balancer (ALB) Across 2 Public Subnets

## Goal

Create an internet-facing ALB across both public subnets with an HTTP listener and a healthy instance target group.

### Evidence

#### Screenshot 13 — ALB details showing two public subnets in two Availability Zones

![](screenshots/Ass5sc13.JPG)

---

#### Screenshot 14 — Target group showing at least one healthy target

![](screenshots/Ass5sc14.JPG)

---

# Task 6 — Create Auto Scaling Group (ASG) in 2 Public Subnets

## Goal

Create an Auto Scaling Group from the Launch Template across both public subnets, with desired capacity 2, minimum 2, and maximum 4, registered to the ALB target group.

### Evidence

#### Screenshot 15 — Auto Scaling Group showing desired, minimum, and maximum capacity and the selected subnet Availability Zones

![](screenshots/Ass5sc15.JPG)

---

#### Screenshot 16 — EC2 instances list showing two running instances in different Availability Zones

![](screenshots/Ass5sc16.JPG)

---

# Task 7 — Configure App to Use RDS + Validate Read/Write

## Goal

Confirm the application communicates with the RDS database through the ALB DNS name with at least one read and one write operation.

### Evidence

#### Screenshot 17 — Browser showing the application loaded through the ALB DNS name with the URL visible

![](screenshots/Ass5sc17.JPG)

---

#### Screenshot 18 — Proof of a database write through a UI message or database query output

![](screenshots/Ass5sc18.JPG)

---

# Task 8 — High Availability Tests (Must Do Both)

## Goal

Test A: terminate one web instance and confirm the Auto Scaling Group replaces it automatically without interrupting the ALB.

Test B: simulate an Availability Zone impact (stop, detach, or reduce desired capacity in one AZ) and confirm the application stays available.

### Evidence

#### Screenshot 19 — EC2 showing the terminated instance and the newly launched instance; timestamps are helpful

![](screenshots/Ass5sc19.JPG)
![](screenshots/Ass5sc19b.JPG)

---

#### Screenshot 20 — Target group showing healthy targets after replacement

![](screenshots/Ass5sc20.JPG)
![](screenshots/Ass5sc20b.JPG)

---

#### Screenshot 21 — Evidence that an instance was removed, detached, placed in Standby, or stopped in one Availability Zone

![](screenshots/Ass5sc21.JPG)

---

#### Screenshot 22 — Browser showing that the ALB DNS endpoint still works during the change

![](screenshots/Ass5sc22.JPG)

---

# Task 9 — Architecture and Test-Results Summary

## Goal

Summarize the VPC/subnet layout, the ALB and Auto Scaling Group setup, the private Multi-AZ RDS setup, and the results of both high-availability tests.

### Evidence

#### Screenshot 23 — A simple architecture diagram, which may be hand-drawn, or an AWS console overview showing the components

![](screenshots/two-tier_1HA.jpg)

---

### Notes

Summarize the VPC and subnets across the two Availability Zones.

Write your answer here.

Summarize the ALB and Auto Scaling Group setup.

Write your answer here.

Summarize the private Multi-AZ RDS setup.

Write your answer here.

Summarize the results of both high-availability tests.

1. VPC and Subnets Across Two Availability Zones

The application was deployed inside a custom VPC (10.0.0.0/16) spanning two Availability Zones to provide network-level redundancy. Each Availability Zone contains a public and private subnet. The public subnets (10.0.1.0/24 and 10.0.2.0/24) host the internet-facing ALB and web-tier EC2 instances, while the private subnets (10.0.11.0/24 and 10.0.12.0/24) are reserved for the database tier. An Internet Gateway provides internet connectivity to the public subnets, while a NAT Gateway provides outbound connectivity for private resources. This design reduces single points of failure and provides a foundation for high availability.

2. ALB and Auto Scaling Group Setup

An internet-facing Application Load Balancer (ALB) was deployed across both public subnets to provide a single, stable endpoint for users and distribute incoming HTTP traffic across healthy web servers. The web tier was managed through an Auto Scaling Group (ASG) using a Launch Template. The ASG maintained a minimum and desired capacity of 2 instances, distributed across the two Availability Zones, with the ability to scale up to 4 instances. ELB health checks allow unhealthy instances to be detected and replaced automatically, improving application availability and resilience.

3. Private Multi-AZ RDS Setup

The database tier was deployed using Amazon RDS with Multi-AZ enabled across the two private subnets. The database was configured with public access disabled, ensuring that it could not be accessed directly from the internet. Connectivity was restricted through the ha-db-sg Security Group so that only the web-tier EC2 instances could communicate with the database on the required database port. Multi-AZ provides automatic database failover capability, helping maintain database availability in the event of an infrastructure failure in the primary Availability Zone.

4. Results of the High-Availability Tests

Test A — Instance Failure: One web-tier EC2 instance was terminated intentionally. The Auto Scaling Group detected the reduction in capacity and automatically launched a replacement instance. The ALB continued routing traffic to healthy targets, demonstrating automated recovery and web-tier resilience.

Test B — Availability Zone Impact: One web instance in an Availability Zone was stopped/removed as part of the failure simulation. The remaining healthy instance in the other Availability Zone continued serving requests through the ALB. The application remained accessible through the ALB DNS endpoint, demonstrating that the architecture could continue operating despite the simulated Availability Zone impact.

Overall result: The tests demonstrated that the architecture was designed not only to deploy successfully, but to remain available, detect failures, recover automatically, and continue serving users when components fail.

---

# LinkedIn Post (Required)

## Goal

Publish a LinkedIn post about the high-availability build, including the ALB URL (or a redacted screenshot), three to five lines on what you built and how you tested high availability, and one proof screenshot.

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/topedavids_aws-aws-cloudengineering-share-7496621924006092801-1Skk/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAAySvXcBSksEGgTHjx1oRy7rOmDlzNAFmEA

---

#### Screenshot of LinkedIn post

![](screenshots/ass5_linkedin.JPG)

---

# Submission Instructions

- Add all required screenshots in your submission
- Do not expose passwords, connection strings, private keys, or account IDs

---

# Completion Checklist

- [ ] Task 1: VPC, four subnets, IGW, NAT Gateway, and route tables created (Screenshots 1–5)
- [ ] Task 2: Least-privilege ALB, EC2, and RDS security groups created (Screenshots 6–8)
- [ ] Task 3: Private Multi-AZ RDS created (Screenshots 9–10)
- [ ] Task 4: Self-configuring Launch Template created and tested (Screenshots 11–12)
- [ ] Task 5: ALB created across both public subnets (Screenshots 13–14)
- [ ] Task 6: Auto Scaling Group running two instances across two AZs (Screenshots 15–16)
- [ ] Task 7: Application verified through the ALB with a database read and write (Screenshots 17–18)
- [ ] Task 8: Both high-availability tests completed (Screenshots 19–22)
- [ ] Task 9: Architecture and test-results summary completed (Screenshot 23 & Notes)
- [ ] LinkedIn post published and URL submitted
- [ ] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*