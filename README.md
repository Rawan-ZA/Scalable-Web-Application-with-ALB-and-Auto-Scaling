# Scalable-Web-Application-with-ALB-and-Auto-Scaling

## 📌 Project Description

This is the **final project** completed at the end of an **AWS Solutions 
Architect ** training course. It demonstrates the practical application of core 
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
