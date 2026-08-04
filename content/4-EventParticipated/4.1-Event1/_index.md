---
title: "FCAJ Community Day 2026"
date: 2026-07-11
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# Summary Report: FCAJ Community Day 2026

**Date:** July 11, 2026  
**Location:** 26th Floor, Bitexco Financial Tower, Ho Chi Minh City  
**Role:** Attendee  

### Event Overview
The FCAJ Community Day 2026 was a vibrant gathering of cloud enthusiasts, engineers, and AWS experts. As a participant, I had the opportunity to absorb practical knowledge, witness live demonstrations of AWS security tools, and understand the pathways to becoming a certified AWS professional.

### Key Highlights

#### 1. AWS Cloud & Solutions Architect Mini-Hackathon Finale
The first few hours of the event featured the thrilling finale of a mini-competition where self-formed teams battled over complex AWS Cloud and Solutions Architect trivia and scenario-based questions. 
- **What I learned:** Watching the teams dissect architecture problems was highly educational. I learned how to approach high-availability designs, cost-optimization strategies, and trade-offs between serverless (AWS Lambda) and containerized (Amazon ECS) workloads. Hearing the rationale behind their answers deeply reinforced my understanding of the AWS Well-Architected Framework.

#### 2. Deep Dive: AWS Security & Threat Detection Agents
The next session was a deep dive into AWS Security services, specifically focusing on the mechanisms of security agents (such as the AWS Systems Manager Agent and Amazon Inspector) operating within cloud environments.
- **How it works:** The speaker explained how the SSM agent facilitates secure, cross-platform remote management without needing open inbound ports. Furthermore, it acts as the foundation for Amazon Inspector to continuously scan EC2 instances and container images for software vulnerabilities and unintended network exposure.
- **Live Demo:** The speaker conducted a live demonstration showing a simulated compromised EC2 instance. We watched in real-time as Amazon GuardDuty detected the anomalous behavior, and Amazon Inspector identified the exact CVE (Common Vulnerabilities and Exposures) package that caused the breach. The agent then automatically triggered an AWS Lambda function to isolate the instance via Security Group modifications. This hands-on demo was a spectacular showcase of automated incident response.

#### 3. Pathway to AWS Certified Cloud Practitioner
The final segment of the event was dedicated to the **AWS Certified Cloud Practitioner (CLF-C02)** certification.
- **Exam Structure:** The speaker broke down the exam domains, highlighting the focus on Cloud Concepts, Security and Compliance, Cloud Technology and Services, and Billing and Pricing. 
- **Value of the Certification:** It was highly recommended as the foundational stepping stone for anyone entering the AWS ecosystem. It proves a robust, overarching understanding of the AWS Cloud platform, which is highly valued by employers and serves as a prerequisite mindset for the more advanced Associate and Professional level certifications.

### Event Experience
Attending the FCAJ Community Day 2026 at the Bitexco Financial Tower was an eye-opening experience. It bridged the gap between theoretical cloud concepts and real-world security operations. The live security demo particularly inspired me to implement automated vulnerability scanning in my own Pedix project.
