# AWS Cloud Practitioner — Hands-On Projects

Personal learning projects built while studying for the **AWS Certified Cloud Practitioner (CLF-C02)** exam.
Region: `ap-southeast-1` (Singapore) | Environment: AWS Free Tier

---

## Project 1 — Static Website on Amazon S3

A static website deployed to S3 using the AWS CLI, with public bucket policy, versioning, and static website hosting enabled.

### Architecture

```
Internet Users
      ↓
[ S3 Bucket — Static Website Hosting ]
  index.html + error.html
```

### What was practiced

- Creating an IAM user with least-privilege permissions (S3 only)
- Creating and configuring an S3 bucket in ap-southeast-1
- Writing and applying an S3 Bucket Policy (JSON) for public read access
- Enabling Static Website Hosting
- Deploying files using the AWS CLI (`aws s3 sync`)
- Enabling S3 Versioning and testing rollback to a previous version

### Deploy commands

```bash
# Configure CLI with IAM user credentials
aws configure

# Upload files
aws s3 sync . s3://your-bucket-name/

# Verify upload
aws s3 ls s3://your-bucket-name/
```

### Key concepts demonstrated

| Concept | Detail |
|---|---|
| Least Privilege | IAM user with AmazonS3FullAccess only — no EC2, no IAM |
| Bucket Policy | `s3:GetObject` allowed for `Principal: *` to enable public access |
| Block Public Access | Disabled to allow public website hosting |
| Versioning | Enabled — delete marker added on "delete", old version recoverable |

---

## Project 2 — EC2 Web Server with Application Load Balancer

A highly available web application running across two EC2 instances behind an Application Load Balancer, with an Auto Scaling Group.

### Architecture

```
Internet Users
      ↓
[ Application Load Balancer ]   ← single DNS endpoint
     /              \
[EC2 Instance 1]  [EC2 Instance 2]   ← Target Group
 "Server 1"        "Server 2"
      ↑
Auto Scaling Group (Min=1, Desired=2, Max=4)
```

### What was practiced

- Creating Security Groups with SG-to-SG referencing (EC2 only accepts traffic from ALB SG)
- Launching EC2 instances with User Data scripts to auto-install Apache httpd
- Creating a Target Group and registering instances
- Creating an Application Load Balancer (Layer 7)
- Demonstrating health checks — stopping one EC2 reroutes traffic automatically
- Creating an Auto Scaling Group with Target Tracking policy (CPU at 50%)

### EC2 User Data script used

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from EC2 - Server 1</h1><p>Instance ID: $(curl -s http://169.254.169.254/latest/meta-data/instance-id)</p>" > /var/www/html/index.html
```

### Key concepts demonstrated

| Concept | Detail |
|---|---|
| Security Group chaining | EC2-SG source set to ALB-SG ID — no direct internet access to EC2 |
| Health Checks | ALB polls `/` every 30s — unhealthy instances removed from rotation |
| High Availability | Two instances across AZs — one failure does not affect the site |
| Auto Scaling | Target Tracking keeps average CPU at 50%, scales in/out automatically |

---

## Skills Covered

`IAM` `S3` `EC2` `EBS` `ELB` `ASG` `Security Groups` `AWS CLI` `Static Website Hosting` `Versioning` `Bucket Policy`

---

## Study Context

Built during a 75-day Cloud/DevOps self-study plan (started 2026-05-20).
Following Stephane Maarek's AWS Cloud Practitioner course on Udemy + Linux Foundation LFS101 on edX.
Target certification: **AWS Certified Cloud Practitioner (CLF-C02)**
