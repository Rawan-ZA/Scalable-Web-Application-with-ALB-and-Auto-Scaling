# Scalable-Web-Application-with-ALB-and-Auto-Scaling

## 📌 Project Description

This is the **final project** completed at the end of an **AWS Solutions 
Architect** training course. It demonstrates the practical application of core 
AWS Solutions Architecture concepts by designing and deploying a 
**production-grade, highly available and Scalable web application** infrastructure on AWS.

The project focuses on applying real-world Well-Architected Framework principles:
- **Reliability** — surviving the failure of an entire Availability Zone without downtime
- **Performance Efficiency** — automatically scaling compute based on real traffic load
- **Security** — protecting the application from common web attacks (OWASP Top 10) 
  and eliminating direct SSH exposure
- **Operational Excellence** — monitoring and alerting on infrastructure health

The solution is built entirely inside a custom-designed **Amazon VPC**, spanning 
**two Availability Zones**, with EC2 instances running behind an Application Load 
Balancer and Auto Scaling Group, backed by a Multi-AZ RDS database for automatic 
failover.

This project was completed as part of the course requirements to demonstrate 
hands-on proficiency in designing highly available, secure, and scalable AWS 
architectures.

## 🏗️ Solution Architecture Diagram

![Architecture Diagram](./## 🏗️ Solution Architecture Diagram

![Architecture Diagram](./aws-scalable-web-application-architecture.png.drawio.png)

### Architecture Summary

The infrastructure is deployed across a single AWS Region, spanning **two 
Availability Zones** for high availability. Each Availability Zone contains:

- A **Public Subnet** hosting a NAT Gateway (for outbound internet access from 
  private resources)
- A **Private Subnet** hosting the application EC2 instance (behind the ALB and 
  managed by the Auto Scaling Group)
- A **Private Subnet** hosting the RDS database instance (Primary in AZ-1, 
  Standby in AZ-2, synchronously replicated)

### Traffic Flow

| Flow | Path |
|---|---|
| **User Request** | User → Route 53 → CloudFront → WAF (associated with ALB) → ALB → Auto Scaling Group → EC2 (Private Subnet) |
| **Database Replication** | RDS Primary (AZ-1) → Sync → RDS Standby (AZ-2) |
| **Admin Access** | Administrator → AWS Systems Manager Session Manager → EC2 (no SSH, no bastion host) |
| **Outbound Internet (patches/updates)** | EC2 (Private Subnet) → NAT Gateway → Internet Gateway |
| **Monitoring & Alerting** | EC2 / ALB / RDS → CloudWatch → SNS → Email/SMS Notification |)

### Architecture Summary

The infrastructure is deployed across a single AWS Region, spanning **two 
Availability Zones** for high availability. Each Availability Zone contains:

- A **Public Subnet** hosting a NAT Gateway (for outbound internet access from 
  private resources)
- A **Private Subnet** hosting the application EC2 instance (behind the ALB and 
  managed by the Auto Scaling Group)
- A **Private Subnet** hosting the RDS database instance (Primary in AZ-1, 
  Standby in AZ-2, synchronously replicated)

### Traffic Flow

| Flow | Path |
|---|---|
| **User Request** | User → Route 53 → CloudFront → WAF (associated with ALB) → ALB → Auto Scaling Group → EC2 (Private Subnet) |
| **Database Replication** | RDS Primary (AZ-1) → Sync → RDS Standby (AZ-2) |
| **Admin Access** | Administrator → AWS Systems Manager Session Manager → EC2 (no SSH, no bastion host) |
| **Outbound Internet (patches/updates)** | EC2 (Private Subnet) → NAT Gateway → Internet Gateway |
| **Monitoring & Alerting** | EC2 / ALB / RDS → CloudWatch → SNS → Email/SMS Notification |

## ☁️ AWS Services Used

### 1. Amazon VPC (Virtual Private Cloud)
A private, isolated network within AWS, designed with **Public and Private 
Subnets distributed across 2 Availability Zones**. Public Subnets host resources 
that need direct internet connectivity (such as the NAT Gateway), while Private 
Subnets host sensitive resources (EC2, RDS) completely isolated from direct 
internet access.

### 2. Internet Gateway
The gateway that allows resources in the Public Subnet to communicate with the 
internet in both directions (inbound and outbound).

### 3. NAT Gateway
Allows resources in the **Private Subnets** (such as EC2 instances) to make 
outbound requests to the internet (updates, downloading packages, etc.) without 
having a public IP or being exposed to inbound connections from outside. A 
separate NAT Gateway is deployed in each AZ to ensure High Availability.

### 4. Amazon EC2 + Auto Scaling Group (ASG)
The servers that actually run the application, located in the Private Subnets. 
The **Auto Scaling Group** monitors load (CPU utilization) and automatically 
increases or decreases the number of instances using a **target tracking 
scaling policy**, ensuring consistent performance regardless of user traffic 
fluctuations.

### 5. Application Load Balancer (ALB)
Distributes incoming traffic across EC2 instances in both AZs based on health 
checks, operating at **Layer 7** (HTTP/HTTPS), which allows for advanced 
routing rules if needed in the future.

### 6. AWS WAF (Web Application Firewall)
Associated directly with the ALB, filtering malicious requests before they 
reach the application, using **Managed Rule Groups** that cover the 
**OWASP Top 10** (such as SQL Injection and XSS).

### 7. Amazon CloudFront
A Content Delivery Network (CDN) that caches static assets at edge locations 
geographically closer to users, reducing latency and lowering the load on the 
backend.

### 8. Amazon RDS (Multi-AZ)
The managed database (MySQL/PostgreSQL), deployed in **Multi-AZ mode**: a 
Primary instance handles reads/writes, while a Standby instance in the second 
AZ stays continuously synchronized via **synchronous replication**. If the 
Primary fails, AWS performs an **automatic failover** to the Standby with no 
manual intervention required.

### 9. Amazon Route 53
The DNS service, routing user requests to the ALB endpoint via an **Alias 
Record**, combined with **Health Checks** to ensure traffic only reaches 
healthy resources.

### 10. AWS Systems Manager – Session Manager
A secure alternative to the traditional SSH + Bastion Host approach for 
accessing EC2 instances. No ports are exposed to the internet, and access is 
controlled entirely through IAM permissions, significantly reducing the 
attack surface.

### 11. Amazon CloudWatch
Collects metrics and logs from all resources (EC2, ALB, RDS), enabling 
**Dashboards** and **Alarms** that trigger when a specific metric (such as 
CPU utilization or error rate) crosses a defined threshold.

### 12. Amazon SNS (Simple Notification Service)
Connected to CloudWatch Alarms, sending real-time notifications (Email/SMS) 
to the operations team whenever an issue occurs, enabling a fast response.

### 13. Security Groups & NACLs
Two complementary layers of network-level protection:
- **Security Groups**: operate at the instance level and are **Stateful** 
  (automatically allow return traffic for any permitted request)
- **NACLs**: operate at the subnet level and are **Stateless** (inbound and 
  outbound rules must be defined separately), providing an additional layer 
  of protection in case a Security Group is misconfigured
