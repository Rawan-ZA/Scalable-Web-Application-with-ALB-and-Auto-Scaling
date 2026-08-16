# Scalable-Web-Application-with-ALB-and-Auto-Scaling

## Table of Content
- [Solution Overview](#solution-overview)
- [Architecture Diagram](#architecture-diagram)
- [AWS Services Used](#aws-services-used)
- [Traffic Flow](#traffic-flow)
- [Security Highlights](#security-highlights)


## 📌 Solution Overview

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

![Architecture Diagram](./architecture-diagram.png)

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


## ☁️ AWS Services Used

| Service | Purpose |
|---|---|
| **Amazon VPC** | Isolated network with public/private subnets across 2 AZs |
| **Internet Gateway** | Enables internet access for public subnet resources |
| **NAT Gateway** | Allows private-subnet resources (EC2) to reach the internet without being exposed to inbound traffic |
| **EC2 + Auto Scaling Group** | Runs the application; automatically scales based on CPU utilization (target tracking) |
| **Application Load Balancer (ALB)** | Distributes Layer 7 traffic across healthy EC2 instances in both AZs |
| **AWS WAF** | Associated with the ALB; filters malicious traffic (OWASP Top 10 rules) |
| **Amazon CloudFront** | CDN caching static assets closer to users to reduce latency |
| **Amazon RDS (Multi-AZ)** | Managed database with a synchronously replicated standby for automatic failover |
| **Amazon Route 53** | DNS routing to the ALB via alias record, with health checks |
| **Systems Manager – Session Manager** | Secure, bastion-free, SSH-free access to EC2 instances via IAM |
| **Amazon CloudWatch** | Metrics, dashboards, and alarms for infrastructure health |
| **Amazon SNS** | Sends notifications when CloudWatch alarms are triggered |
| **Security Groups & NACLs** | Instance-level (stateful) and subnet-level (stateless) network access control |

## 🔐 Security Highlights

- **Private Compute & Database Layers** — All EC2 instances and the RDS database 
  are deployed in private subnets, with no direct internet exposure.

- **Defense in Depth (Security Groups + NACLs)** — Security Groups provide 
  stateful, instance-level access control, while NACLs add a stateless, 
  subnet-level layer as a secondary safeguard.

- **Web Application Firewall (WAF)** — Associated with the ALB to filter out 
  common web exploits (SQL Injection, XSS, and other OWASP Top 10 threats) 
  before traffic reaches the application.

- **No SSH, No Bastion Host** — Administrative access to EC2 instances is done 
  exclusively through **AWS Systems Manager Session Manager**, controlled by 
  IAM permissions, with zero open inbound ports.

- **Database Isolation** — RDS is only reachable from the application tier via 
  Security Group rules; it has no public endpoint.

- **Multi-AZ Database Failover** — RDS Multi-AZ ensures the database remains 
  available even if the primary instance or its entire Availability Zone fails.

- **Encrypted Data** — RDS storage and backups are 
  encrypted at rest, and traffic between the client and ALB is encrypted in 
  transit via HTTPS/TLS.
